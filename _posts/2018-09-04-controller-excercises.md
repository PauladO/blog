---
title: "Controller workouts"
layout: post
excerpt: "We have a model, let's say it's a Human model. Now some of our humans may want to lose weight, so they can become a FitnessGroupMember in a gym. Obviously they can join different gyms and different fitness groups. Some of these humans are reported to occasionally hang out with other people, but not all of them do so. It really depends, but if a human is introverted, they generally do not hang out in with strangers."
last_modified_at: 2018-09-04T19:27:01
categories:
  - Refactoring
tags:
  - code
  - refactoring
  - ruby
  - ruby on rails
---

I’m currently working in a 6-7 year old legacy code base, that has never been refactored. It has some very bloated models, some of the controllers contain a lot of business logic, and there is a lot of completely unnecessary
complexity.

So let’s refactor some of that. All variable names have been changed and code is left out. I really don’t like it when people use foo or bar in their examples, because it makes it harder to reason about the code, so instead, I've come up with analogies of our code.

----

We have a model, let's say it's a Human model. Now some of our humans may want to lose weight, so they can become a FitnessGroupMember in a gym. Obviously they can join different gyms and different fitness groups. Some of these humans are reported to occasionally hang out with other people, but not all of them do so. It really depends, but if a human is introverted, they generally do not hang out in with strangers…

I knew all this going in, and I knew that the logic for this was not in a separate FitnessGroupMember model. I assumed it would be in the Human model, since in this particular code base, all of the things humans do are in the Human model. Not the best idea, so this was what I set out to fix.

But looking in the Human model, I only found a few bits and pieces with some logic for fitness group members. But I had a hunch. I quickly found the offending code in a big fat FitnessGroupMember controller. This thing really needed to lose some weight:

```ruby
class FitnesGroupMembersController < ApplicationController

  ...

  def import
   @fitness_group = FitnessGroup(params[:fitness_group_id])
   members = @fitness_group.import(params[:file]) if params[:file]

   if members.nil?
     redirect_to fitness_group_path(@fitness_group), alert: "Can't read excel"
     return
   elsif members.empty?
     redirect_to fitness_group_path(@fitness_group), alert: "Empty excel"
     return
   elsif @fitness_group.gym.max_members and @fitness_group.gym.max_members - @fitness_group.gym.member_count <= (members.count - @fitness_group.gym.members.where(email: members.map{|member| member[:email]}).count)
     notice = "Reached maximum number of gym members"
     notice << " only #{@fitness_group.gym.max_members - @fitness_group.gym.member_count)} places_left" if @fitness_group.gym.max_members - @fitness_group.gym.member_count > 0
     notice << " please expand_fitness_group"
     redirect_to fitness_group_path(@fitness_group), alert: notice
     return
   end

   city = @fitness_group.gym.company.try(:city) if @fitness_group.gym.present? and @fitness_group.gym.company.present?
   city = current_user.city if !city

   count = 0
   members.each do |member|
     fitness_group_member = Human.where(email: member[:email].downcase.strip).first
     fitness_group_member = Human.new(email: member[:email].downcase.strip) if
     fitness_group_member.language = member[:language].to_s.downcase.strip
     fitness_group_member.invited_by = current_user
     ...
     ...
     ...

     if fitness_group_member.new_record?
       fitness_group_member.invite!
    else
      UserMailer.add_fitness_group_member(current_user, fitness_group_member, @fitness_group).deliver
    end

     if fitness_group_member.save
       if @fitness_group.members.where(id: fitness_group_member.id).count == 0
         @fitness_group.members << fitness_group_member
         count += 1
         fitness_group_member.send_schedule
       end
     end
   end

   redirect_to fitness_group_path(@fitness_group), notice:  "fitness group members added: #{count}"
 end

 def create
     @fitness_group = Team.find(params[:fitness_group_id])

     if (!@fitness_group.gym.can_add_members?) and @fitness_group.gym.members.where(email: params[:fitness_group_member][:email].downcase.strip).count < 1
       notice = "Reached max number of members"
       redirect_to fitness_group_path(@fitness_group), alert: notice
       return
     end

     @fitness_group_member = Spotter.where(email: params[:fitness_group_member][:email].downcase.strip).first
     @fitness_group_member = Spotter.new(email: params[:fitness_group_member][:email].downcase.strip) if @fitness_group_member.nil?
     city = @fitness_group.try(:gym).try(:city) || current_user.city
     ...
     ...
     ...

     respond_to do |format|
 			if @fitness_group_member.new_record?
 	      @fitness_group_member.invite!
 			else
 				UserMailer.add_fitness_group_member(current_user, @fitness_group_member, @fitness_group).deliver
 			end

       if @fitness_group_member.save
         unless @fitness_group.members.where(id: @fitness_group_member.id).count > 0
           @fitness_group_member.fitness_groups << @fitness_group
         end

         @fitness_group_member.send_schedule

         flash[:notice] = t('fitness_groups.member_added', name: @fitness_group_member.name)
         format.html { redirect_to(fitness_group_path(@fitness_group)) }
       else
         format.html { render action: 'edit' }
       end
     end
 	end


 ...


end
```
It became clear really quickly, that not only did we need a FitnessGroupMember model. The controller was passing an excel file to the FitnessGroup method, which just parsed it and returned the members. In several places, fitness group members were being configured in the exact same way. Basically, it needed to do run some laps and dry off.

So first I created a MembershipBuilder. You initialize it with the fitness group and your current user, and pass it the hash with attributes (form the form or the excel). It configures the member, and sends the member the correct email. basically everything that needs to be done before saving. sending the emails before saving the fitness_group_members seems a bit stupid. But I was refactoring and not changing behavior

```ruby
class TeamMembershipBuilder
  def initialize(fitness_group, user)
    @fitness_group = fitness_group
    @user = user
  end

  def create_membership(attributes)
    fitness_group_member = set_fitness_group_member_attributes(attributes)
    send_invitation_mails(fitness_group_member)
    fitness_group_member
  end

  protected

  def set_fitness_group_member_attributes(attributes)
    fitness_group_member = self.find_or_initialize_member
    unless fitness_group_member.new_record? or fitness_group_member.is_introverted(attributes[:email].downcase.strip)
      fitness_group_member.hangs_out_with_strangers = true
    end
    fitness_group_member.is_introverted = true
    fitness_group_member.first_name = attributes[:first_name] || fitness_group_member.first_name
    fitness_group_member.last_name = attributes[:last_name] || fitness_group_member.last_name
    fitness_group_member.city = city
    fitness_group_member.language = attributes[:language].to_s.downcase.strip
    fitness_group_member.invited_by = @user

    fitness_group_member
  end

  def city
    city = @fitness_group.gym.try(:city) if @fitness_group.gym.present?
    city = @user.city if !city

    city
  end

  def find_or_initialize_member(email)
    fitness_group_member = Spotter.where(email: email).first
    fitness_group_member = Spotter.new(email: email) if fitness_group_member.nil?

    fitness_group_member
  end

  def send_invitation_mails(fitness_group_member)
    if fitness_group_member.new_record?
      fitness_group_member.invite!
    else
      UserMailer.add_fitness_group_member(@user, fitness_group_member, @fitness_group).deliver
    end
  end
end
```

Then I set up a MembershipImporter. This handles the excel parsing, and turns it into an array of hashes with attributes. It handles the errors, so that the user can easily be redirected if something goes wrong. It can then be called again from the controller. It initializes a MembershipBuilder, and passes the array with attribute hashes, creating the members. Finally it handles saving and counting all the members, and dispatching a schedule to all the members.

```ruby
class MembershipImporter
  attr_accessor :count
  attr_accessor :error

  def initialize(fitness_group)
    @count = 0
    @fitness_group = fitness_group
    @error = nil
  end

  def import(file)
    members_data = parse_excel(file)

    @members = excel_data_to_members(members_data)
  end

  def create_memberships(user)
    builder = MembershipBuilder.new(@fitness_group, user)

    @members.each do |member|
      fitness_group_member = builder.create_membership(member)
      save_and_send_schedule(fitness_group_member)
    end
  end

  def parse_excel(file)
    begin
      members_data = ExcelParser.excel_to_hash(file.path)
    rescue
      @error = "Can't read excel"
      return nil
    end

    if members_data.empty?
      @error = "Empty excel"
    end

    members_data
  end

  def excel_data_to_members(members_data)
    members = []
    members_data.each do |row|
      member = {}
      row.each do |k, v|
        member[:email] = v if k.downcase.include? 'mail'
        member[:first_name] = v if k.downcase.include? 'first'
        member[:last_name] = v if k.downcase.include? 'last'
        member[:language] = v if k.downcase.include? 'language'
      end
      members << member if member[:email]
    end
    members
  end

  def save_and_send_schedule(fitness_group_member)
    if fitness_group_member.save
      if  @fitness_group.members.where(id: fitness_group_member.id).count == 0
        @fitness_group.members << fitness_group_member
        @count += 1
        fitness_group_member.send_schedule
      end
    end
  end
end
```

With these models in place, I could clean up the controller:

```ruby
class FitnesGroupMembersController < ApplicationController

  ...

  def import
    @fitness_group = FitnesGroup.find(params[:fitness_group_id])
    importer = MembershipImporter.new(@fitness_group)
    members = importer.import(params[:file])

    if importer.error
      redirect_to fitness_group_path(@fitness_group), alert: importer.error
    elsif @fitness_group.has_reached_max_member_count?(members)
      redirect_to fitness_group_path(@fitness_group), alert: @fitness_group.has_reached_max_member_count_notice
    else
      importer.create_memberships(current_user)

      redirect_to fitness_group_path(@fitness_group), notice:  "#{importer.count} fitness group members added")
    end
  end

  ...

  def create
    @fitness_group = FitnesGroup.find(params[:fitness_group_id])

    if (!@fitness_group.gym.can_add_members?) and @fitness_group.gym.members.where(email: params[:fitness_group_member][:email].downcase.strip).count < 1
       ... (creating a notice)
      redirect_to fitness_group_path(@fitness_group), alert: notice
      return
    end

    builder = MembershipBuilder.new(@fitness_group, current_user)

    @fitness_group_member = builder.create_membership(params[:fitness_group_member])
    if @fitness_group_member.save
      unless @fitness_group.members.where(id: @fitness_group_member.id).count > 0
        @fitness_group_member.fitness_groups << @fitness_group
      end

      @fitness_group_member.send_schedule

      flash[:notice] = "added #{@fitness_group_member.name}")
      redirect_to(fitness_group_path(@fitness_group))
    else
      render action: 'edit'
    end

  ...

  end
end
```

It’s slimmed down quite a bit. It’s not quite there yet, and I’m not 100% sure about all of the changes. (for instance, I don’t know if it’s wise to let the importer also save the members.) But at least it's a lot DRYer.

---
title: "New private project"
layout: post
excerpt: "A few months ago, there was an [article](https://nos.nl/artikel/2321931-een-tikkie-voor-een-paar-duppies-meer-dan-tienduizend-per-dag.html) in the news about small payments through Tikkie (a Dutch payment request app). Every day, more than 10.000 payment requests of under 2 euro were being made through the app. Mostly by highschool students who make very little money. For them, these small amounts are worth much more than for adults with jobs. This gave me an idea. An app that tracks who spent what among friends would reduce the need for these small payments."
last_modified_at: 2020-05-07T21:45:01
categories:
  - Projects
tags:
  - design
---

_A few months ago, there was an [article](https://nos.nl/artikel/2321931-een-tikkie-voor-een-paar-duppies-meer-dan-tienduizend-per-dag.html) in the news about small payments through Tikkie (a Dutch payment request app). Every day, more than 10.000 payment requests of under 2 euro were being made through the app. Mostly by highschool students who make very little money. For them, these small amounts are worth much more than for adults with jobs. This gave me an idea. An app that tracks who spent what among friends would reduce the need for these small payments. It's an app I would have liked to have, when my college friend group was eating out every day in China. Something to keep track of whose turn it was to pay. Something to keep it more or less even. This led me to my new project: Payback Time (working title?)._

The first step is fleshing out the design. Aside from providing me with some handholds for what it should look like, his helps me think of what I actually want the app to do, and how the UX will work. I used Figma to design some screens. I decided on a simple color scheme, and straight forward UX, with only a few interactive components on each screen.

<img alt="Landing page" src="/assets/images/overview_toggled.png?raw=true" width="300px" />

The overview page would be a little like WhatsApp, with only a list of groups and friends, and a button to add a new group or friend. Under the hood, both the groups and friends will be the same. However, in the interface, a single debt to a single friend is simpler, and I wanted the design to reflect that. For instance, when settling the debt of a single friend, it only shows the amount you need to pay, or the amount you are requesting. When settling within a group, an overview of who pays what to whom is shown.

<img alt="Settling the score with one friend" src="/assets/images/settle_friend.png?raw=true" width="300px" style="display: inline" />
<img alt="Settling the score with one friend" src="/assets/images/settle_group.png?raw=true" width="300px" style="display: inline" />

For MVP (Minimal Viable Product) I want to implement the friend / group views (including adding new amounts of course), previous payments, objecting to an entry (requesting an edit) and settling the score. I still need to figure out a few things, such as how to show outstanding payment requests and resolving those, and where these are displayed without cluttering the interface.

After MVP one of the first features I would implement is notifications. If you're strapped for cash, it's nice if your friend actually knows you'd like them to pay you back. I also think that it would be nice to allow people to say 'never mind about this one' when it comes to settling the score. Some amounts are not worth asking for.

Another idea I had was having amounts settled between groups. Say I owed Janet 5 euro, but in a group, she owes 10 euro and I am owed 5. I'd like to be able to allow the user to clear that debt between the groups. The biggest issue I have with this is that it may make the payments within the group unclear to users. I'm still thinking about a way to solve this. I may give users the option to allow or disallow this in groups, so they can decide whether they want to trust my algorithm. The point of the app is to save people from having to make the calculations for themselves. (It's not a mediation app to keep them from fighting about money though...)

So the next step is chosing a stack. I've settled on React Native, since I have plenty experience with it and it fits the project pretty well. I'm still not 100% decided on the back end. I don't have a lot of time to spare, so it would be wise to go with something I know. I'm considering NestJS. I've worked with it before, it's relatively simple and has decent documentation. I am also considering .Net, because it's a thing I'd like to learn, and Ruby on Rails, because I worked with it for years and my friend, who is joining me in this project, is also very familiar with it.

For now, here is the prototype:

<iframe style="border: none;" width="800" height="900" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Fproto%2F2Et2lglG9SzJJ2Qe2AVhDe%2FPayback-time%3Fnode-id%3D1%253A2%26scaling%3Dscale-down" allowfullscreen></iframe>
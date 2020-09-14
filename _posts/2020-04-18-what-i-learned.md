---
title: "What I learned from bad code"
layout: post
excerpt: "Let me preface this by saying having legacy code isn't bad. Having bad code isn't even bad. However, when the code you wrote 5 minutes ago is just as bad as code you wrote 5 years ago, and just as bad as everything in between, you may have a problem."
last_modified_at: 2020-04-18T16:17:01
categories:
  - General
tags:
  - lessons
  - bad code
  - legacy code
---

_Let me preface this by saying having legacy code isn't bad. Having bad code isn't even bad. However, when the code you wrote 5 minutes ago is just as bad as code you wrote 5 years ago, and you've not done any better in between, you may have a problem._

About three years ago, I switched careers to web developing. Fresh out of bootcamp, I worked with some people who did not seem to care about the quality of their code. Being a total rookie, I didn't quite realize this right away, but it became abundantly clear fairly quickly. The idea of conistently pushing straight to master seemed like a bad idea to me from the start, and I wasn't happy about not getting code reviews, but I was programming, and making a living doing it. So for a while I was happy. But that didn't last very long. The lessons I learned in the time I worked with them are things you hear over and over again as a developer. Things that somehow still are not always lived by. Working in this mess, however, I truly internalized these lessons.

#### The frustration begins
The first bump in the road came when, due to an unfortunate situation, I had to take over virtually all work of maintaining the codebase. I was temporarily the only one really working on the project. After only 2 months and with only a few months of experience as a developer, I had to try to keep the ship from sinking. Luckily, they brought in a second junior developer. A good friend of mine. She made the work a bit more bareable again. At least it was fun. The problem was, while I got the responsibility of keeping the whole house of cards from coming down dumped on me, she never seemed to be allowed to do anything of substance. All in all, not very helpful for my workload.

Meanwhile, I was getting very frustrated with the code. It was a formless blob. Written in a dynamic language (Ruby), but with no tests, and no real architecture, other than what Ruby on Rails dictates. It was a hot mess. For a long time I got by, tiptoeing around code that was unreadable, buggy and tangled. It was not a situation in which I could learn about patterns and best practices. So I started teaching myself.

#### Lesson 1: Never let code rot get out of hand
I was lucky enough to be able to take classes on design patterns and refactoring. Once per month I got to see some decent code. And it felt good. Really good. And then the next day, I would have to return to the horror of the code base. It had been over 6 years, and no refactor had been done. Classes were thousands of lines long, methods sometimes were hundreds of lines. Naming was completely unhelpful in some parts of the code (I'll smack the next person who uses 'data' as a variable name). You know you're in trouble when developers start avoiding big chunks of code. And we were in trouble.

I discussed it with the other junior developer. The others seemed to think everything was still hunkydory, but we were both extremely frustrated. We decided to request some time to refactor the worst of it. We sort of got it. Only it was not allowed to get in the way of producing more features. We took it anyway, even though we knew it would definitely get in the way of building features. We were both at the point of calling it quits if nothing changed, so we figured they'd prefer us working on improving the code rather than having two thirds of their development team leaving.

Step one was assessing the code. Finding places that were most desperately in need of improvement. We did some static code analysis, and were baffled by the results. Parts of the code we had been avoiding were off the charts bad. Where the score should be between 5 and 20, 30 at most, it was sometimes over 400. Duplication was everywhere. Code was simply copy-pasted all over. We split some of the worst, but most used code among ourselves, another developer reluctantly taking on a particularly bad chunk (which he then didn't touch), and set to work. I attempted to refactor a bit of code that I had been working around for a month. I still couldn't figure out what half of it did (seriously, don't name variables 'data'). I ironed out the worst of it, but simply had to leave parts the way they were, because they were so convoluted, it took too much time to figure them out. I vowed to refactor any new code I wrote to something simple and clear from then on (or if I couldn't, at least commented on the way it worked).

#### Lesson 2: If it's a bad practice, it's probably not wise to do
You'd think this would be obvious, but some developers are just really stubborn. Even when the framework made something impossible, one developer decided it was a good idea to put several different variable types into one database field (string, hash or array). He didn't just think "This is kind of a dirty way of doing it, but it's a solution to my problem". He thought it was smart. It was a major source of bugs. Sometimes big parts of the website simply stopped working, because someone decided to change the type of the field halfway through collecting data. Even worse, when updating the framework (Ruby on Rails 3 to 4), we ran into major difficulties.

RoR 4 introduced strong parameters, every parameter had to be typed. It was a real headache to fix. We ended up having to hack in a solution. I felt dirty adding a hack upon a hack. I knew there was no saving it too. There was 6 years of data in the database. I found a solution to the problem of having different types of data that wasn't horrifying, but would require a big migration. I knew I couldn't convince the others to make that kind of sacrifice.

Sometimes there's discussion on whether something is a bad practice or not, that's good. But never will I just ignore it if people say it is.

#### Lesson 3: Keep your framework up to date
Well, at least, up to date-ish. Six years in, we were still working with the version that came out months before they started writing the code. There were many security issues. Not great when you're storing bank account information. The only reason we updated when we did, was that I pushed it through. Reading about the security issues made me uncomfortable. It was hell trying to update. It should have taken a few weeks, but it turned into months. Which leads me to my next lesson.

#### Lesson 4: Write test
Updating the framework wouldn't have been nearly as painful if we had had tests. A lot of other things also hurt less. You can simply write code with much more confidence when there are tests in place. Working without tests, every time I pushed a change, I had a nervewracking minute until I was sure I didn't break the entire website. And I was never totally confident I didn't break something that I couldn't see right away. You can never take away that insecurety entirely, but tests do help.

#### Lesson 5: Always keep learning
Although this is the final lesson in this blog, I learned much more. The problem was, the others didn't seem that interested in learning new things. The code they were writing while I was there was just as messy and convoluted as the code that had been written in the years before. It was like they thought they were finished learning. There were so many things that would have benefitted from learning a new approach. For one, with the amount of data going through the system, using Ruby was probably not the smartest thing to do. If we had created some microservices in a faster language, we'd have been so much better off.

After over a year, I finally had the chance to work in a different codebase. I didn't really know just how bad the first one was, until I saw how much better things could be. I bacame very eager to learn how to do things right, and quickly picked up a better way of working. The general lesson was to stay on top of things and that is what I try to do now.
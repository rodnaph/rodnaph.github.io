---
layout: page
---

I've been using Jenkins since it was Hudson, and in a bunch of different configurations. My first experience was when working with a large client who had a big deployment that automated their entire build cycle through multiple stages. Seeing how awesome this (free!) tool was the company I worked for proceeded to give it a spin, and I think it probably started out running on one of the developers machines for more than a while. Over the next few years it grew to be managed by developers, then by ops, then kind of by everyone...  oh, in single-server and master/slave setups.

The constants I've seen through all this change though seem to be the following...

  * Be it because of plugins or cosmic rays, given a long enough period of time...  all Jenkins instances will break. It's like ultimate Jenga.

  ![Image](/public/dj1.jpg)

  * Jobs don't always pass, and when this happens often the only way to debug things is on the CI server itself.
  * Contention arises through the performance of this shared resource.

Trying to prevent these issues usually has the consequence that arguments about access and ownership arise. Who's responsibility is it to fix it, who should be ensuring it's capacity, and who is allowed to access it? It's because of this that in my current role I'm trying a new way of using Jenkins, by decentralising it to each developer.

Even modest development machines these days probably pack a fair bit of punch, and I'd guess that most of the time that power isn't being used (When we're typing the computer doesn't have to do that much, right?  Well, Eclipse aside of course...). So using a VM (or Docker image) for Jenkins shouldn't be too much of a problem, you probably have one for your development environment already.

I've become a big fan of Ansible over the last 6 months, and have fully configured my Jenkins VM with all the jobs that are needed. This has the following benefits...
New developers can spin up an instance in no time at all.
No need to bother with Jenkins roles and permissions.
Should I balls up my instance, I can easily create a new one.
If I suspect anything funky going on with my jobs, I've full access to investigate (without risking introducing problems for others)
I am free to innovate and experiment with new stuff on my instance (again, without the risk of affecting anyone else)
Knowledge about the system will hopefully be spread more evenly around all the users because of the no-hassle access, and complete ownership.
Decentralisation reduces the risk of Jenkins becoming a single point of failure when shit hits the fan at the wrong time.
I'm currently working in a *very* small team of two, so can't claim I've rolled this out to an organisation and it's brilliant - but it's working nicely enough so far so I just wanted to share my thoughts.

Of course, this wouldn't work for everyone (if you need it to be a high-performance dedicated machine, maybe for long running batch jobs for instance) but I think it could cover a lot of cases. Access to shared resources like package repositories can be configured and provisioned automatically, leaving less (hidden) magic knowledge that suddenly becomes lost.

Tried this yourself? Have a better system? Think I'm an idiot? Let me know :)


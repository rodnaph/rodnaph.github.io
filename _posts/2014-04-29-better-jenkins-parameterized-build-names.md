---
layout: page
---

I use Jenkins to do all kinds of things, and recently I've been using parameterized builds more and more to reduce the number of jobs that I need to maintain. But when each job can do a diverse set of things (for example I have a create-rpm job which has parameters for the project to build a package for, and the repository to put it in) the build list becomes quite meaningless.

![Image](/public/jpn1.png)

When I go back to the build it's impossible to tell which one of these was the stable build of package X that I need to refer to. I googled for options and instantly hit upon the build name plugin, which seemed to do exactly what I needed.

When installed the plugin adds an option to the build environment section which allows you to customize how to construct the build name, for example here's one that adds the git branch that the project was built from.

![Image](/public/jpn2.png)

You can enter normal characters to add text, but of course more usefully you can reference dynamic variables from the build to add some more context to the name that is created.

Also, luckily for me, the plugin supports accessing the build parameters themselves, but you need to go through the environment to get at them. Here's an example from my create-rpm job where I use the name of the repository the package is being built from and the stability of the repository it's being deployed to...

![Image](/public/jpn3.png)

It's then nice and easy to refer back to what jobs actually built, rather than being lost in the numbers.

![Image](/public/jpn4.png)

For once, computers save the day!


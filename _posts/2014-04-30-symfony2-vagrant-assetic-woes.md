---
layout: page
---

One of the projects I work on is a fairly sizeable Symfony2 application, and while it's always been deployed on Linux, until fairly recently I'd been developing it on OSX. This seems to be a fairly standard setup these days.

But I've always thought it would be nicer to be closer to the target environment. Especially when developing against and integrating with the various components the application uses. Examples of these things would be email proxies, metrics and logging backends, and search infrastructure. So I've made the move to using Vagrant, provisioned as our servers are with Ansible - and it's great!

The first downside I hit though was trying to use the Assetic --watch switch to dynamically rebuild my Js/CSS assets as I modified them. The problem I was seeing was that changes to files were regularly taking up to 30 seconds to get picked up and rebuilt, obviously too slow. This is an essential part of my workflow, I change and save very frequently so updating these manually is simply not an option.

I hunted around online for some suggestions to see if I was missing something, found a few good articles, but was already using the techniques they listed as speedups. Those being...

Use NFS for shared folders
Use opcache
Disable xdebug
Mount cache/logs to tmpfs

The next thing I tried was the new rsync support in Vagrant 1.5, but wasn't massively hopefully after reading that it didn't scale particularly well to large projects. And my problems here were twofold...

Vagrant was a little slow to sync
Permissions

The first problem was a bit off putting, but with some suggestions I thought maybe I could optimise my way around it. The permissions issue though was that the Vagrant rsync support copies files to the guest owned by the user vagrant:vagrant, and doesn't seem to be configurable. I already have a mix of users going on with my Ansible provisioning and my applications running under php-fpm. Adding another into this seemed to be quite clumsy, and when coupled to the fact that the rsync support wasn't that fast anyway, I didn't want to pursue this any further.

While I was gnashing and wrangling over all this my pal and top clever boy @BenjaminDavies had taken a look into alternative scanning/dumping solutions, with one of the top options being fswatch. It looked simple so I cloned, compiled and using an example from Stackoverflow gave it a try...

```
fswatch src/My/Bundle/Resources/public "app/console assetic:dump"
```

Turns out this is from an older version of the tool, and the current version doesn't support executing a command when changes are detected. Instead, it prints the path to the changed file to stdout. Ben sends me an updated one-liner and I paste it without blinking (my eyes beginning to bulge at this stage of rage)

```
fswatch src/my/Bundle/Resources/public | while read line; \
  do app/console assetic:dump; done
```

But editing files does nothing.  FML. The problem this time was that I was running this on the host not the guest, and the database configuration details wouldn't work from here. "Database?" I hear you say, but I thought you were dumping assets? It would seem booting a Symfony application which has Doctrine configured will connect to the database on boot.

So...  what now?  Well I'm already a few hours of wasted time in here, so decide to create a new config environment especially for Assetic, which doesn't have any database configuration, saving this in app/config/config_assetic.yml. Even this turned out to be a lot more painful than I'd hoped, but I got there in the end.

```
fswatch src/My/Bundle/Resources/public | while read line; \
  do app/console assetic:dump --env=assetic; done
```

It was a long ride any many thanks to Ben for sticking by me during my dark moments of despair.

Using this with my NFS mounted folder in Vagrant is now acceptably fast. Though not perfect. I may have to visit a better, simpler, solution when my blood pressure returns to normal.

I'm sure there are a million better solutions to this, and I'd love to hear how other people are solving this problem. So please let me know!


---
layout: page
---

Today I ran into some interesting behaviour with Ansible when role dependencies are skipped. Let me set the scene, so I have a route53 role which has a dependency on a java role.

```
# roles/route53/meta/main.yml
---
dependencies:
  - { role: java }
```

I also have a jenkins role which has the same dependency on java. So far so simple. I then have a role which includes both of these as dependencies...

```
# roles/ci-server/meta/main.yml
---
dependencies:
  - { role: route53, when: not is_dev_vm }
  - { role: jenkins }
```

The only interesting bit here is that when provisioning a local VM (not in EC2) it doesn't make sense to perform any Route53 tasks so the role is skipped. I just upgraded the base Vagrant box I'm using, so scrapped my old one and booted up the new version using the Ansible provisioner, only to get this error half way through...

```
TASK: [jenkins | Ensure jenkins Running] ******************** 
failed: [192.168.33.12] => {"failed": true, "item": ""}
msg: bash: /usr/bin/java: No such file or directory
```

WAT? jenkins requires java and there haven't been any errors so it must be installed...  WAT!?!

I checked back through the output, and before my jenkins role I couldn't see any mention of java...  But going back still further (and after some asking on IRC) I realised there was some related output just before my route53 role...

```
TASK: [java | Install Java Package] ******************** 
skipping: [192.168.33.12]
```

Aha! I dived into the docs to see if this was expected behaviour, and found this...
By default, roles can also only be added as a dependency once - if another role also lists it as a dependency it will not be run again.
So because the route53 role was skipped, and the java role was skipped, it will not be run again. Even though it's required by the jenkins role. Gotcha! I understand why this decision probably makes sense most of the time, and maybe my nesting of dependent roles is somewhat unique, but in my instance it's rather unfortunate. I'm not sure if this would be considered a bug.

My workaround is to always include the route53 role, but then do the conditional inside it.

```
# roles/route53/main/tasks.yml
---
- include: install.yml
  when: not is_dev_vm
```

And all is good again.

Update: I raised the issue on the Ansible Github project, stay tuned for the result!

[https://github.com/ansible/ansible/issues/7353](https://github.com/ansible/ansible/issues/7353)


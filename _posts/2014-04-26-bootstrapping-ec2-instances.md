---
layout: page
---

I love AWS, even if for nothing more than the fact that as one commentator put it, it works at all. When provisioning instances with Ansible I like to always start from vanilla distro AMIs (usually CentOS). But this leaves the problem of bootstrapping so I can get the barebones authentication and DNS stuff set up (ie. I can ssh as a non-default sudo user via a DNS name, which I can add to my playbook).

(One way around the DNS issue that I'd like to explore is via dynamic inventories, but I'm not quite ready for that yet...)

So to bootstrapping. While looking into how I might do this I found a handy piece of information buried deep down in a Stackoverflow comment (that I can no longer find) - you can pass a list of host names as the -i argument to ansible-playbook. For example...

```
ansible-playbook -i "hostname," ec2-bootstrap.yml
```

Note the comma at the end, that's actually required to let Ansible know this is a list of hostnames. My  playbook then looks like...

{% highlight yaml linenos %}
# ec2-bootstrap.yml
- hosts: all
  user: root
  roles:
    - sudo-access
    - setup-dns
{% endhighlight %}

This playbook will target all the hosts that I've specified by -i and run the list of roles against them. When its done I'm left with an instance I can add to my inventory for full provisioning, and ssh in to via my standard ops user.

Maybe I could even specify the role via a command line extra-vars option...  so I could do the whole provisioning at once... I will have to try that! No wait, I need the parameters from the inventory, natch...

How do other people handle this, with "gold" AMIs to clone maybe?  I don't fancy that option as it's something to maintain.  And things like this that need maintenance usually don't get it. Please let me know if I'm doing it wrong!


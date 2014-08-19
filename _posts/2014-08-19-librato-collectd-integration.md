---
layout: page
title: Librato Collectd Integration
---

In my day job we use the excellent [Librato Metrics](https://metrics.librato.com) 
service to push all our metrics data to for monitoring and visualisation. We've
been primarily using [StatsD](https://github.com/etsy/statsd/) to collect metrics from our various applications,
and the [AWS integration](https://metrics.librato.com/cloudwatch) for data on 
those services. Something we've been missing though is some server level 
information that isn't provided with [Cloudwatch](http://aws.amazon.com/cloudwatch/) (specifically 
disk space and memory). I've been meaning to try using [collectd](https://collectd.org/) 
to collect this and then push it off to Librato somehow. Then luckily the Librato team announced 
[native integration](http://blog.librato.com/posts/turnkey-server-monitoring-beta)...

Just what we needed!

## Ansible Configuration

Luckily Collectd was available through the package manager on our current
distribution, so I set right to creating an Ansible role for it...

{% highlight yaml %}
# roles/collectd/tasks/main.yml
---

- name: Package Installed
  yum: pkg=collectd state=present
  tags:
      - collectd

- name: Configured
  template: src=collectd.conf.j2 dest=/etc/collectd.conf
            owner=root group=root mode=644
  tags:
      - collectd
  notify:
      - restart collectd

- name: Service Running
  service: name=collectd state=started enabled=true
  tags:
      - collectd
{% endhighlight %}

With the associated handler...

{% highlight yaml %}
# roles/collectd/handlers/main.yml
---

- name: restart collectd
  service: name=collectd state=restarted
{% endhighlight %}

And basic configuration file for testing the setup on my VM...

{% highlight bash %}
# roles/collectd/templates/collectd.conf.j2

Hostname    {% raw %}"{{ inventory_hostname_short }}"{% endraw %}
FQDNLookup   false

LoadPlugin load
LoadPlugin memory
LoadPlugin df
LoadPlugin write_http

<Plugin write_http>
    <URL "https://collectd.librato.com/v1/measurements">
        User {% raw %}"{{ librato_email }}"{% endraw %}
        Password {% raw %}"{{ librato_token }}"{% endraw %}
        Format "JSON"
	</URL>
</Plugin>

Include "/etc/collectd.d"
{% endhighlight %}

Helpfully the configuration is provided when you enable the service integration
in Librato as well:

![Image](/public/lcd1.png)

# Just Works?

After adding the role I quickly reprovisioned my development VM and switched to
Librato to watch the data roll in...  and it did! Some quick testing,
deployment to our test environment for final checking and then it was
straight out to production - thanks Librato!

The collectd plugins are easily configured now through my Ansible scripts, time
to dive in to the data! :)


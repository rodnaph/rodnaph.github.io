---
layout: page
---

I use Mock to build RPMs in a clean chroot'd environment - and it works great! The only problem is the documentation is abysmal. Now, I'm not trying to be one of the people who complains and slags off someone else's project - I'm actually trying to not be that person... but in this case I'm afraid the criticism is absolutely true.

If you need to use Mock then it seems the only place that has any kind of information is the official website. And if you're not already an expert, then like me I expect you'll find yourself pretty lost (though you won't be lost for long as it's only one page, so you'll move on quite quickly from being lost to being completely stuck).

I was quite happily building RPMs with Mock for a PHP application, but build times were slow as the environment was clean each time Composer was having to download every single one of the dependencies. Not ideal. Luckily there's a plugin for Mock (which is enabled by default) called bind_mount which allows mounting host folders inside the chroot which can then act as the durable cache folder for Composer.

Brilliant! Indeed, if you can figure out how to use it... Sorry... I'm trying hard and failing to not be bitchy here, but the only documentation I could find that this feature even existed (after resorting to IRC channels) were two lines of example configuration that comes installed with Mock. I used these...

```
configopts['pluginconf']['bind_mount_enable'] = True
configopts['pluginconf']['bind_mount_opts']['dirs'].append(('/var/cache/composer/', '/var/cache/composer/' ))
```

But alas... watching the builds crawl by it was clear nothing was actually being cached. But no errors, no clues as to why. I suspected it was something to do with the permissions of the folder on the host I was mounting into the chroot. After a lot of trial and error I finally hit upon the solution, here it is...

```
chmod 2755 /var/cache/composer
```

Or in my Ansible role...

{% highlight yaml linenos %}
- name: Composer Cache Directory
  file: path=/var/cache/composer state=directory
        owner=jenkins group=mock mode=2775
{% endhighlight %}

Then the next time I ran a build I could see the dependencies getting written, and after that the build time was cut by about 70%

Yahoooooooooooooo!


---
layout: page
---

I've always wondered this... just why does clearing the cache with Symfony2 take so long, and use so much memory?

```
app/console cache:clear
```

The answer is I still don't really know, but welcome to my adventure on speeding up deployments and reducing memory usage.

I've been told that Symfony loads the application into memory (as it would do running any other task) before clearing the cache folder. Recently I've been running into problems with this actually running out of memory during deployments to my production environments, so switched it to the more brute force...

```
rm -rf app/cache
```

But wondered if this had any side-effects...

![Image](/public/sc1.png)

Afaik I'm not using any cache event listeners, and testing it seemed all good.  As I mentioned in the tweet I also need to now call cache:warmup (which is called by default on cache:clear unless you specify --no-warmup).

Rolling out to production things were better. Clearing the cache during a deployment was now a much quicker process, but I was still occasionally running into out of memory errors on the cache:warmup.

```
Warming up the cache for the prod environment with debug false
ERROR 9 
```

So next I looked into the warmup command...  What I learned was that there are required and optional cache warmers.  The later can be disabled by using the flag --no-optional-warmers. I've found that this significantly speeds up cache warming, and crucially means that I don't suffer from out of memory failures.

Of course, the warming of these caches is only deferred until runtime, so if this would be a problem for you then it might not be an option. But in my case it's something I can deal with.


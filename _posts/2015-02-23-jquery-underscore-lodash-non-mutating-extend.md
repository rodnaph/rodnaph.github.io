---
layout: page
title: jQuery/Underscore/Lodash extend, without the mutation
---

I was writing a little Javascript today where I wanted to scale a bunch of
coordinate data from a source object to a new set of scaled objects.  So I did
something like this...

{% highlight javascript %}
var originalData = JSON.parse(source);
var scaledData = _.extend(
    originalData,
    {objects: _.map(toScaledObject, originalData.objects)}
);
{% endhighlight %}

But surprisingly didn't get the results I expected. Turns out firstly I had the
arguments to _map_ the wrong way around, but before realising this I also
noticed that my original source object was also getting updated. Wat?!?

## Destination Object

After checking the docs I realised both jQuery and Underscore/Lodash refer to
the first object passed to the _extend_ function as the **destination** object.
So while the function also (curiously) returns this destination object, it is in
fact mutated in-place as part of the extend operation.

## The Workaround

The workaround I used was to provide a new object for this destination object,
and then as part of _extend_ map the source object onto this. Like...

{% highlight javascript %}
var originalData = JSON.parse(source);
var scaledData = _extend(
    {},
    originalData,
    {objects: _.map(originalData.objects, toScaledObject)}
);
{% endhighlight %}

Simple enough.  So library writers please try to be immutable by default.


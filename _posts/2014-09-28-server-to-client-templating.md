---
layout: page
---

The PHP application I work on uses [Symfony2](http://symfony.com/) 
with [Twig](http://twig.sensiolabs.org/). Recently we've undergone a redesign
and it was a good chance to consolidate a lot of things that have grown out in
their own directions over the last few years. One of those things involved
listing content, so I created a listing template for other code to extend
(obviously simplified for this example) ...

{% highlight jinja %}
<li>
    {% raw %}<div class="title">{% block title %}{% endblock %}</div>{% endraw %}
    {% raw %}<div class="body">{% block body %}{% endblock %}</div>{% endraw %}
</li>
{% endhighlight %}

This worked great and allowed me to *really* simplify and centralise the
styling of these kinds of elements (of which there are lots!).  But anyway, one
of the places where content was listed was in a Javascript-only component that
did the usual fetching [JSON](http://json.org/) from the server and then rendering
it ting...

So how to use my lovely listing templating here?

I'm using Lodash templating in the client so wanted to get the code there somehow.
I decided to try using script templates...

{% highlight html %}
<script id="tpl-listing" type="text/template">
    {% raw %}{% include 'MyBundle:Foo:listing.html.twig' %}{% endraw %}
</script>
{% endhighlight %}

With the template then including the Lodash bits...

{% highlight jinja %}
{% raw %}{% extends 'MyBundle::listing.html.twig' %}

{% block title %}<%= item.title %>{% endblock %}

{% block body %}
    <ul>
        <% _.each(item.children, function(child) { %>
            <li>
                <%= child.title %>
            </li>
        <% }); %>
    </ul>
{% endblock %}{% endraw %}
{% endhighlight %}

I can then use this in the client...

{% highlight javascript %}
var tpl = $('#tpl-listing').html();
var html = _.template(tpl, {item: foo});

// etc...
{% endhighlight %}

When I first did this I thought it was a bit insane, but I've grown to like it
and haven't run into any problems yet.  If you know of any, or can suggest some
improvements please let me know (using a single language or templating library 
between client and server probably being one of them...)


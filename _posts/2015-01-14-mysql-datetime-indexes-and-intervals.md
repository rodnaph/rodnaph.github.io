---
layout: page
---

Today I ran into an interesting (though afterwards 'doh! of course...') problem
involving a slow MySQL query that should have been a hell of a lot quicker. The
application I work on supports users in many timezones, so when it comes to
query time we always need to adjust system recorded times (which are always
UTC) to allow for the users current timezone.

So for example consider a table...

{% highlight sql linenos %}
CREATE TABLE mytable (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    date_created DATETIME NOT NULL,
    KEY date_created_idx (date_created),
    PRIMARY KEY (id)
);
{% endhighlight %}

And a made-up query against it, where :tzOffset is the users UTC offset in
hours, and :theDate is a datetime we'd like to query against...

{% highlight sql linenos %}
SELECT *
FROM mytable
WHERE date_created + INTERVAL :tzOffset HOUR > :theDate
{% endhighlight %}

Given a relatively small number of rows you'll have no problem with this, but
open the *EXPLAIN* output and you'll notice MySQL ignores the index.

## Change Perspective

I suppose it makes sense that MySQL is now unable to use the index, it's an
index on the *date_created* column, not on the
*date_created_plus_some_timezone_* column. So the fix is to move the date
wrangling to the query parameter instead.

{% highlight sql linenos %}
SELECT *
FROM mytable
WHERE date_created > :theDate - INTERVAL :tzOffset HOUR
{% endhighlight %}

Notice how the sign on the _INTERVAL_ modifier changes as we're moving the date
the other way. You could of course do this calculation on the date before
binding it to the query, but in my case we often need to do the _INTERVAL_
adjustment on the column itself so it's nicer to keep it in one place.

A reminder to always keep one eye on *EXPLAIN*...

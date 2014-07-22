---
layout: page
---

While making what I thought was going to be a two second change to a MySQL
query today I found myself waiting a full 60 seconds for the results when I
first ran it.  Not a good start, let's have a look at an example with all the
cruft removed (background: there are applications, each having many quotes)

{% highlight sql linenos %}
select
    a.id,
    a.name,
    a.etc,
    quotes.total -- find quote total for applications
from applications a
    left join (
        select application_id, count(*) as total
        from quotes
        group by application_id
    ) quotes
    on quotes.application_id = a.id
{% endhighlight %}

Now the reduced query above looks pretty contrived, but it's only because I've
removed all the other joins and selects. In the above example the query could
be written much more concisely (to essentially *be* the derived table) - but in
my real query there are many more joins which mean that aggregating on the
quotes join is not possible.

## Full Table Scan Time

Here's a snippet of the explain output - with the index information removed as 
it was too wide for the page, but it's not important...

| select\_type | table | type | ... | ref | rows | Extra
|------------|-------|------|--------------|-----|------|-------
|	DERIVED  |quotes |	index | ... |		NULL |	451839 |	Using index 

If you're not familiar with MySQLs *EXPLAIN* keyword, the important part to
note is the rather large number of rows that are being scanned to create the
result set. Bugger...

Using derived tables to compute aggregates this way is a technique I've used
quite a bit and never really had a problem with. Usually it's a simple way to
pull in more information to your query without complicating the JOIN situation.

Adding indexes didn't seem to help, at least I couldn't work out any that were 
missing that I could add. The derived table was correct, and if executed alone 
it would indeed scan the entire quotes table as it needs to.

## Derived to subquery

What I imagined would be happening here is that the query plan would be
generated to evaluate outside-in, so selecting all matching applications and
then creating the derived table to calculate quote totals for each. This
doesn't appear to be the case however, as a much larger portion  of the quotes 
table is actually being scanned - though far from all of it.

What I tried (randomly) was to move the derived table to be a subquery in the
SELECT portion of the query (hoping for the result I assumed above).

{% highlight sql linenos %}
select
    a.id,
    ( select count(*) 
      from quotes 
      where quotes.application_id = a.id ) as total
from applications a
    -- etc...
{% endhighlight %}

Here's the new EXPLAIN snippet...

| select\_type | table | type | ... | ref | rows | Extra
|------------|-------|------|--------------|-----|------|-------
|DEPENDENT SUBQUERY |	quotes |ref	| ... | database.a.id |	1 |	Using where

Much better! And more importantly execution time is down from 60 seconds to
about 65 milliseconds. 

## Lesson Learned

There was no particular reason I was using derived tables over this method 
previously, it's just a technique I knew that gave me the results I needed. But
I'm glad I've got another tool to work with when I approach these same
situations in the future.

Of course, if there's something I've missed or a better way to approach this 
please [let me know](https://twitter.com/rodnaph)!


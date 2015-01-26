---
layout: page
title: jQuery UI onChangeMonthYear Event Gotchas
---

Today involved fixing a bug where I have a some date fields that use the
**datepicker** component from jQuery UI, and I've added some helper links next
to them when you can set the year to 3, 6 12, etc months in the future. The bug
reported was that these links only work the _second time_ they are clicked.
Hmmm...

I spent more time than I'd like to admit reading and stepping through the code
to work out what was going on, and just found myself going round in circles.

## onChangeMonthYear

Recently I had added a hook for the event **onChangeMonthYear** so that when
the user changed the month or year from the open datepicker it would default to
the first of that month (as this is what my users usually want to do, and the
out-of-the-box behaviour is to do nothing). Here it is...

{% highlight javascript %}
var onChangeMonthYearHandler = function(year, month, inst)
{
    var picker = $(this);
    var current = picker.datepicker('getDate');

    if (current) {
        current.setDate(1);
        current.setYear(year);
        current.setMonth(month - 1);

        picker.datepicker('setDate', current);
    }
};
{% endhighlight %}

## Gotcha #1

Can you see my first bug?  Well, when setting the value of the datepicker via
my helper links if the date being set is different to the current month/year of
the datepicker then this event gets fired.  The flow being...

* Set date to 6 months in future
* _onChangeMonthYear_ fires
* _onChangeMonthYear_ handler sets the date to the first of the month
* Doh!

Ok, so I guess it's expected and probably useful that this event fires both
when the user changes the month/year via the control and **also** when the date
is changed via the API.  So here's my first fix, checking if the datepicker is
visible...

{% highlight javascript %}
var onChangeMonthYearHandler = function(year, month, inst)
{
    var picker = $(this);
    var current = picker.datepicker('getDate');
    var isVisible = $(inst.dpDiv).is(':visible');

    if (isVisible && current) {
        current.setDate(1);
        current.setYear(year);
        current.setMonth(month - 1);

        picker.datepicker('setDate', current);
    }
};
{% endhighlight %}

## Gotcha #2

I thought this would have fixed it, but no...  I can see that my check as to
wether or not the datepicker is visible is working, and am confident it's
required...  but the date is still being set incorrectly and needs two clicks.

After more debugging and code reading it turns out the bug comes from my usage
of the **getDate** API method. This method does not just return the date from
the datepicker, but in doing so mutates the internal state of the control. The
flow being...

* Set date to 6 months in future
* _onChangeMonthYear_ fires
* _getDate_ called, which resets the date using current data (not applied from new date yet)
* Doh!

Unfortunately there is no mention in the docs that calling any datepicker 
methods from within this handler could lead to undesirable results...  Gah!

Here's my fix...

{% highlight javascript %}
var onChangeMonthYearHandler = function(year, month, inst)
{
    var isVisible = $(inst.dpDiv).is(':visible');

    if (isVisible) {
        var picker = $(this);
        var current = picker.datepicker('getDate');

        if (current) {
            current.setDate(1);
            current.setYear(year);
            current.setMonth(month - 1);

            picker.datepicker('setDate', current);
        }
    }
};
{% endhighlight %}

I'm only trying to set the date to the 1st if the datepicker is open, but also 
only accessing the current date of the control when this is the case.  This
means that when the handler fires after the date has been set programatically
it won't step in and bugger things up.

## Conclusion

My take away from this is a reminder as to the complications and dangers of
managing state, and how immutability makes things safer and less unexpected. I
took a look to see if I could contribute this note to the official docs but
there was no obvious way to do so.

## P.S.

I'm aware of Javascript scope rules and the fact my **picker** and **current**
vars are actually visible to the entire function, but I prefer to ignore
particular gotcha in favour of readability.


---
layout: page
---

Yesterday I spent about half an hour tracking down a bug that had cropped up mid feature development. Everything was working fine...  then for some reason two of my vars appeared to have the same value, when they really shouldn't have. After much "GGrraaah!!" and many "Waaaatt?'s I found the problem...

{% highlight php %}
<?php
protected function foo(
    DateTime $fromTime,
    DateTime $toTime,
    DateTime $fromTime,
    DateTime $toTime
) { ...
{% endhighlight %}

Now reading that it's probably obvious something is not right, but this was the result of either some spurious renaming or copy pasting, I didn't catch it right away. But then I thought about it, shouldn't the compiler be catching this kind of error? PHP is a dynamically typed language, but you'll still get errors about duplicate method/property names, why not parameter names? I cried out to the Twitter...

![Image](/public/dfa1.png)

I guess it's kind of expected, but the fact that it had caused a bug (which I luckily caught before it got any further) was clearly a problem.

This morning fellow elite hacker Ian "Jonko" Jenkins mentioned that he'd tried this in a few other languages and got some of the same result. So how do other languages fair? Let's see, first everyones favourite Javascript...

{% highlight javascript %}
function foo(x, x) {
    return x;
}

foo(1, 2); // 2
{% endhighlight %}

The result is 2, the same as PHP. Javascript is dynamically typed as well, interesting...  How about Python...

{% highlight python %}
def foo(x, x):
  x

print foo(1, 2)
{% endhighlight %}

This time an error!

```
SyntaxError: duplicate argument 'x' in function definition
```

Hoorah for Python. Ruby next...

{% highlight ruby %}
def foo(x, x)
  return x
end

puts foo(1, 2)
{% endhighlight %}

And another error!

```
duplicated argument name
```

Now a personal favourite Clojure, come on do me proud...

{% highlight clojure %}
(defn foo [x x] x)

(foo 1 2)
{% endhighlight %}

Gah, failure, it happily returns 2... Let's try Java...

{% highlight java %}
package rod;

class Test { 
    public function foo(int x, int x) {
        return x; 
    }
}
{% endhighlight %}

And as I suspected it caught the problem, which a lot more information that the other languages that did.

```
variable x is already defined in method foo public function foo(int x, int x) {
```

So given the fact that Python and Ruby passed the "test" quite happily there can't be anything intrinsic about dynamic languages that make this kind of checking impossible to do. Perhaps it's just an afterthought, I mean who in actual reality is going to do this... I guess.

Have any deep insights about why this "functionality" might exist, want to post how your favourite language fairs, just use the comments below. Back to hacking.


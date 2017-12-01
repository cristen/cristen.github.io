---
layout: post
title:  "How to generate a password using python"
date: 2017-12-01 11:57:00 GMT+1
categories: python
---
As an aware web user, you probably know that it is vital to use a different password for all the websites you’re using.
It can be quite boring to make up a new password each time. Fortunately in python it’s quite easy to create one.

Here’s my snippet to do so :

{% highlight python %}
import string
import random

password = ''.join([random.choice(string.ascii_letters + string.digits) for i in range(10)])
print(password)
{% endhighlight %}

For anybody wondering how it works :

string is the module to use if you want to work with letters or digits.
random has a lot of nice methods to randomize things.

Basically the first thing we do here is to concatenate the list of ascii letters and digits.

{% highlight python %}
string.ascii_letters + string.digits
{% endhighlight %}

Then we use the choice(list) method from the random module to pick a letter or a digit from the forementioned list :

{% highlight python %}
random.choice(string.ascii_letters + string.digits)
{% endhighlight %}

This only works once that’s why I use a generator to do this task a certain number of times (here 10 times) :

{% highlight python %}
[random.choice(string.ascii_letters + string.digits) for i in range(10)]
{% endhighlight %}

This creates a list of random digits and letters. Now all we have to do is to join the elements of the list with no space between using :

{% highlight python %}
''.join(list)
{% endhighlight %}

And voilà ! The password is created.

Have fun !

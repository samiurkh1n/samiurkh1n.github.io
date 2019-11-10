---
layout: post
title: const, volatile, mutable for C variables
date: 2019-11-09 00:01:00.000000000 -05:00
type: post
published: true
status: published
---

_Note: This is more of a quick reminder to myself in the future when I forget
what the volatile and mutable keyword is useful for._

```const``` is to make sure that the value of a variable doesn't change. The data
of the type is constant.

```volatile``` is to tell the compiler to reduce optimizations around this variable.
An example would be a global int that gets modified by another thread but, in
the scope of the file, it looks like nothing will edit this variable.

```mutable```, usually applied to private member of a class, keyword allows for
a const member function to modify state.
use case:

{% highlight cpp %}
class Test {
    mutable int data = 0;
public:
    int getData() const {
        if (data == 0) {
            data = 1;
        }
        return data;
    }
};
{% endhighlight %}

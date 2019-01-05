---
layout: post
title: An introduction to overload sets.
date: 2019-01-05 00:01:00.000000000 -05:00
type: post
published: true
status: published
---

_Note: This post was inspired by Titus Winter's talk on Modern C++ API Design given at cppcon 2018. Check
that out [here](https://youtu.be/xTdeZ4MxbKo)._

In this article, I wanted to introduce a concept that has really changed the way I thought about API
design: C++ overload sets.

## API design basics

API design is an important challenge a lot of scientists and engineers face. People put in a lot of effort
to build something that they find value in or they believe others might find value in. But it's usually the
interface that can really impact whether a particular thing gets used or not.

The same principle applies to software: the software for how things are done can be dense. That is why
most of that complexity and density is *abstracted* away so that the software writer can focus on what
matters, which is to use a tool or library to build what they want to. That interface is an API, an
application programming interface.

## The fundamental unit of API design in C++

In principle, an API can be anything that abstracts. A button on a website can abstract away all the code
that sends relevant information over the tubes of the internet. With programming libraies, the item that
tends to abstract things away are functions, which can group together a whole sequence or set of instructions
that a machine should execute. 

However, can we say that functions are the fundamental unit of API design? For instance, let's take a look
at vector push back, a method to push an element into a vector:
```cpp
void vector<T>::push_back(const T&);
void vector<T>::push_back(T&&);  // rvalue argument
``` 
Above, these are both valid functions to push an element into a vector. Is it right to say that they are
distinct elements of an API? Not really! An API is composed of a set of methods to do computations. A method,
however, shouldn't really be reasoned about as a single function. After all, push back should do the same
thing with a const T& or a T&&. 

## Overload Sets

This is the idea behind *overload sets*. Overload sets are the proper unit of API design. Titus Winters
defines overload sets as such:
> An overload set is a set of functions in the same scope in the same namespace such that if one is found, all them will be.
From the C++ standard:
> Functions that are in an overload set are roughly equivalent (that is they roughly do the same thing).

Winters describes a few properties or principles of good overload set design:
1. correctness can be judged without knowing which overload is being used
2. a single comment can describe a full set
3. each element of the set is doing the same thing

### Overload sets in Condit
Over Christmas break, I continued working on Condit, a set of libraries aimed at C++ developers to help them write
more defensive programs. I added a new type in Condit called StatusOr. The idea behind StatusOr is that your
computation can return either a Status or another type. Check out the new library 
[here](https://github.com/samiurkh1n/Condit/blob/master/includes/status_or.h). In designing the interface for
the StatusOr type, I wanted to make sure that a programmer can easily construct a StatusOr by simply returning
a Status or a value from within a function scope and I wanted to explicitly disallow any default constructors.
I also had to decide whether I wanted to allow implicit conversions from one type to another.

Some context on that last problem:
A constructor is allowed to make one implicit conversion of an argument to resolve types. When you mark a constructor
as explicit, you are disallowing the implicit conversion. An example
```cpp
class Test {
public:
	Test(bool s): s_(s) {}
private:
	bool s_;
};

// main function
Test t(0);  // implicit conversion fails if constructor above is marked explicit
```
_Should I have marked the constructor explicit?_

With overload sets, you can better reason about whether you should use the explicit keyword (especially for constructors).
One of the key principles mentioned from before was that correctness can be judged without needing to know which
constructor gets called.
If the user needs to know which constructor is picked, then the constructor needs to be explicit. With StatusOr,
the user didn't need to know so I didn't mark the constructor as explicit.
And just like that, I could move on.

I think reasoning about APIs as a set of overload sets is a pretty useful conceptual tool that more people should use.
It can simplify API design from the API writers point of view: ensuring that all functions in an overload set do the
same thing allows software writers to clearly cut duplication.
It can simplify the interface itself, since the user of the API can trust that the compiler will resolve to the
correct function compile time.

Give Titus Winter's talk a listen and let me know what you think!

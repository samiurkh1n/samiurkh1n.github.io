---
layout: post
title: C++ Value Categories
date: 2020-03-27 00:01:00.000000000 -05:00
type: post
published: true
status: published
---

### "lvalue required as left operand of assignment"? What?

Often times when writing C++, you'll encounter errors about mismatch between
lvalues and something values.

For instance, here is a rather trivial program that might produce that error:

{% highlight cpp %}
#include <iostream>

int main()
{
    int x = 5, y = 5;
    if (x % y = 0)
    {
        std::cout << "Something to print\n";
    }
    return 0;
}
{% endhighlight %}

You'll encounter that wonderfully readable statement: "lvalue required as left
operand of assignment".

For a newcomer to programming, the very first word is already intimidating.
Students will often use trial and error to try and get past the error but that
can often lead to some nasty mistakes later on.

My objective here is to help you, my dear reader, understand what lvalues are in
C++!

### Compiler and programming language design

When you sit down to design a [programming] language, you first need to decide
what it looks like and how to read it. Reading, with English at least, involves
reading left to right, taking in each word or set of words until you get the
fuller picture of what a sentence is trying to say. You then process that sentence
in the larger context of a paragraph or multiple paragraphs and you finally rest
with an understanding of what something has to say.

Writing a programming language is no different, just more structured :o!
In the end a programming language needs words, sentences, paragraphs, and a system
for interpreting it. If you were to write a compiler (a reader for your language),
you'd have to write different stages that (sort of) reflect how you'd read a
normal language like English:

1. Lexer - how do I know when one word or symbol ends and a new one begins?
   Often times, a lexer will break a string of text down into pieces called
   tokens.
   In literature, this might be called "Lexical Analysis"
2. Parser - am I reading a structurally valid sentence?
   A parser will have a grammar, a set of rules that define valid sentence
   structure.
   Remember that stream of tokens made by the lexer? Does the stream I am
   reading physically make sense?
   In literature, this might be called "Syntax Analysis"
3. Semantics - am I reading a sentence that is meaningful?
   Remember that stream of tokens the parser said was structurally sound? Does
   the stream produce a sentence that is meaningful (not some weird set of
   words)?
   In literature, this might be called "Semantic Analysis"

For now, we are just going to focus on the parser, the part of a compiler that
enforces grammar.

Before we go on, I want to introduce you to an idea you might have seen before:
*tradeoffs*. When making any decision, you will have alternatives with certain
benefits and costs. This also applies to programming language design where you
might need to balance how understandable a language is against how many features
a language has. C++ language designers have the same problem and often times,
they erred to let features in first and fix understandability later. This same
tradeoff impacted the value categoires we will later on go to discuss.

### Expressions in C++

In C++, expressions are "a sequence of operators and their operands that specifies
a computation" (from section 5 on expressions in the
[C++ standard]({{site.url}}/assets/cpp-standard.pdf)).

There are a few things we can observe from that definition:
* sequence: this is a collection of objects (in this case operands and operators)
  where repetition is allowed and order matters.
* operands: this is usually some object or quantity that an operation can act upon.
* operators: these act on operands and act like functions. In C++, operators
  include = for assignment, +|-|*|/ for arithmetic, ==|!= for equality comparison
  and so on.
* evaluations: expressions specify some computation and, importantly, produce
  some value. So 2+2 is an expression that, when executed, produces a value 4.
  In C++, expressions produce a value and can also induce side effects.
  For instance, evaluating printf("4") prints the character 4 onto standard
  output (the printing is a side effect of the evaluation).

### The Heart of the Cards: Value Categories Explained

(Remember all that talk about language design? Well it's about to come in handy.)

When designing a language like C++, you will need to define the syntax of
expressions. For instance, if you are trying to define arithmetic expressions,
you would have to document or program that a valid arithmetic expression
will include an operator sandwiched between two operands (like 2 + 2, where
the + operator is between the operands 2 and 2).

<strong><u>Value categories</u></strong> are a way of classifying expressions.
The end goal of classifying expressions is to help define what a valid expression
is! Remember, a language designer wants to balance the features they want in
their language against understandability. Value categories are as much a tool
as it is a concept to help support a set of memory operations (like move semantics,
which we'll cover in a later section) alongside with the expressions most
C++ programmers expect.

In C++, expressions are classified as such:
<div style="text-align:center">
<img id="cppvaluecategories" src="{{site.url}}/assets/images/cppvaluecategories.jpg" width="300"
align="middle" alt="C++ value category tree"><br />
Source: C++ Standard Section 3.10
</div>

Let's dive deeper into what each of these categories contain:
* lvalue := (historically appears on left side of assignment) designates a
  function or an object. Think *locator value*. In C++, an object, generally,
  an instance of a class. Any instantiation optionally has a name, and is required
  to have a value, a type (the type usually defines the size, and memory alignment).
  Also, think *left value* as in left side of assignment.
* rvalue := (historically appears on right side of assignment) an xvalue,
  temporary object, or subobject, or a value not associated with an object.
  Think, *right value* as in right side of assignment.
  An example of a temporary object is here, in ```X(2)``` before being passed to
  ```convert(X)```:

{% highlight cpp %}
class X {...}

Y convert(X);

void f()
{
    Y object = convert(X(2));
}
{% endhighlight %}
* xvalue := refers to an object, usually near the end of its lifetime so that
  the resource may be moved. For example, the result of a function that returns
  a rvalue reference (```int&&```) to an object is an xvalue.
  Think *eXpiring value*.
* prvalue := an rvalue that is not an xvalue, like literals (2,
  true, 2e7, 1.2) or the result of a function call that is not a reference.
  Think *pure rvalue*.
* glvalue := an lvalue or an xvalue

Bjorne Stroustrup, creator of C++, [thinks]({{site.url}}/assets/terminology.pdf)
about each value category as having two independent properties:
1. "has identity" - this means you can distinguish objects with values. For
   instance, in ```std::vector<int> x, y```, you can easily distinguish x and y
   because they would be stored in different locations in memory. x and y, here,
   are lvalues.
2. "can be moved from" - you can't really distinguish objects here. You can't
   take the memory location of this value and distinguish it from another.
   prvalues, like literals and the result of functions that return a value, fall
   here.

Using this, we can put each value category into it's own combination of "having
an identity" and "being movable":
- lvalue := has identity but cannot be moved (ex: you can distinguish the
  variable ```int x``` from ```int y```).
- xvalue := has identity and can be moved (ex: rvalue references, we'll touch
  upon this in the move semantics section below).
- prvalue := has no identity and can be moved (ex: you can't distinguish one
  use of 0 from another).
- glvalues are just anything that has an identity.
- rvalues are anything that can be moved.

### "lvalue required as left operand of assignment"? Let's understand it first.

Let's go back to that error from above. If the compiler is helpful, it'll
point to the problematic section:

{% highlight cpp %}
if (x % y = 0)
{% endhighlight %}

So why is this wrong? It's because ```x % y``` produces an prvalue equal to the
literal 0. In a valid C++ expression, a prvalue cannot be assigned to. Only
glvalues, like lvalues and xvalues, can be assigned to!

A conditional usually has some sort of boolean expression for it's condition.
A boolean expression should have some use of any boolean operator. In this case,
the programmer intended to use ```==``` to compare the result of the expression
```x % y```.

The fix:
{% highlight cpp %}
if (x % y == 0)
{% endhighlight %}

### Move Semantics or: How I Learned to Stop Worrying and Love Value Categories

If you've seen me write "rvalue reference" above and want to understand it,
you're in the right place.

C++ has always erred towards empowering the writer. The C++ working group (CWG)
wants to add capabilities. If a feature or paradigm can fit
well within C++, the working group will try to add it. (Note: this is my
observation and this can be wrong)

This philosophy includes being able to support more precise control over how
memory gets allocated, used, and cleaned up in C++. This is a pretty powerful
feature but, if used improperly, can cause wastage and even programs to break.
That is why you will see game engines other and high performance servers, that
want precise control over memory, more often written in C++. Many machine
learning libraries, like numpy, scikit, will be Python wrappers to C libraries.
But as stated before, if used improperly, things can go wrong. *Very wrong.*

Let's say we have some class that maintains a pretty large resource, like a
giant vector of structs. Now a copy operator or constructor for this class
would do something like "clone the underlying data from the original copy into
my private data members and destroy the original". This copying, however, can
get pretty expensive: it takes time to make a copy and, for some period of time,
there are two large sections of memory that have the same data.

Instead of copying data, it'd be much better if we just said that the new object
is responsible for that data now. If the classes held pointers to that region
of memory, this would be like swapping the pointers so that the original is
pointing to ```nullptr``` and the object being copied to holds the valid pointer
to the region. This is the essence of move semantics, where you are essentially
"moving" the data instead of making a copy!

Say you have two instances of the class, X and Y. Each instance has its own
pointer to a region of memory that the class is responsible for maintaining.
Without move semantics, you would implement a copy constructor by copying the
region of memory maintained by Y into X. With move semantics, you instead just
swap the pointers they have.

But there is one catch left! We wanted to copy Y's value into X. Upon swapping
the pointers, X now holds Y's original value which is great. But now Y also holds
X's value. When you copy into a variable, the original value should get
overwritten! The copy constructor should now clear Y so that we don't have some
kind of misunderstanding. This is where rvalue references come in! Instead of
passing Y into X's constructor as a lvalue reference (which would look
like ```&Y```), we instead pass an rvalue reference (which would
look like ```(&&Y)```. The && denotes an rvalue reference. Because an rvalue
reference is an xvalue (remember "eXpiring value"), the Y variable and it's data
will be marked for destruction.

Now when you declare Y, you can use it as an lvalue reference ```(&Y)``` or an
rvalue reference ```(&&Y)```. You need to call ```std::move(Y)``` to convert it
into an rvalue reference. This is to preserve both the copy constructor. Move
operations and constructors are usually seperate definitions.

Two gotchas to be aware of though:
1. You need to make sure the deletion of the original data has no side effects.
   Test your type's destructor.
2. Don't return a call to ```std::move(x)```. You might be tempted to do this:
   {% highlight cpp %}
   Y Blah(int i)
   {
       Y object;
       data_ = i;
       // do some other things ._.
       return std::move(object);
   }
   {% endhighlight %}
   A compiler can run a return value optimization to reduce how many temporary
   variables it claims. std::move(x) returns an expiring value within that scope.
   That return value might expire before it ever gets returned.

### The objective of this article

You've reached this point. Hopefully, you now
* have a basic understanding of language and compiler design
* understand value categories and how to identify if an expression's value category
* have a starting understanding of move semantics and rvalue references

If I've failed to help you achieve any of the three things above, shoot me an
[email](mailto://samiurkh1n@gmail.com) and tell me why.

If you see a mistake, also shoot me an [email](mailto://samiurkh1n@gmail.com)
so that others reading this and I don't make the same mistake again :o

Thanks, stay safe! PRACTICE SOCIAL DISTANCING >:(

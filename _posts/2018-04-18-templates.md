---
layout: post
title: Templates out of the blue
tags: PRO2
---

In a digression, the professor explained templates. While moderately
advanced, I think they are a useful feature that we will need later, so I think
it is a good moment to explain them.

(Sidenote: most of the article was written on 2018-03-15, but was left as a
draft because I didn't know for sure if templates would be relevant this course.
As I'm about to write the tree article, I realised that they are, so I
decided to finish it and publish it.)

Let's consider the following function:
```c++
int max(int a, int b) {
    if (a > b) return a;
    else return b;
}
```

(Tip: if you ever find yourself writing this, check [`std::max`][std::max]
from `<algorithm>`)

Nothing new here. Now, we want to expand it to doubles. No problem:
```c++
double max(double a, double b) {
    if (a > b) return a;
    else return b;
}
```

(This is a good moment to think why this works and remember the joys of
function overloading)

Now we have a class `Decimal` which implements the ordering operators
(in particular, `>`). Well, let's copy-paste again:
```c++
Decimal max(Decimal a, Decimal b) {
    if (a > b) return a;
    else return b;
}
```

Now's when you hear a thousand programmers screaming avoid the evils of
copy-paste programming. And they're right. We've written the exact same
code three times, only changing the types involved, but not touching the
body of the function otherwise. Also, this only allows `int`, `max` and
`Decimal` to be used, do you want to write the same function for the
gazillion types that could be used with it?

C++ has a feature that deals with this: _templates_. Templates allow you
to define a function with "placeholder" types, which will be substituted
when compiling as needed. Let's see how:

```c++
template <typename T>
T max(T a, T b) {
    if (a > b) return a;
    else return b;
}
```

As always the space in `template <...>` is optional, and `T` can
be named anything you want as long as it is a valid C++ identifier (convention
is to use UpperCamelCase, and is, of course, optional).
Also, you could use this (note that `typename` has become `class`, nothing else
changed):

```c++
template <class T>
T max(T a, T b) {
    if (a > b) return a;
    else return b;
}
```

This is exactly the same, without any hidden exceptions, as strange as this
may seem in C++. It is equivalent, only a matter of personal preference and
code style (but, please, be consistent: it will be easier for everyone).

I prefer `typename`, because `int` and `double` are not classes (they're
primitives), but are nonetheless accepted (they still would with `class T`,
despite not being classes, that's why I prefer `typename T`).

Now the compiler will get this template and do the copy-paste for us, replacing
`T` for whatever type we want. It is important to note that `T` refers to
a single type: if `T = int`, then `int max(int a, int b)`; if `T = double`,
then `double max(double a, double b)`; never `int max(double a, Decimal b)`.

This has a catch: the compiler must be able to get to the definition, so
this has to be in a header file (although some use a special `.tpl` file
which is then `#include`d in the `.hh`). Also, since the compiler must copy and
compile every instantation of the template (to the compiler, `max<int>` and
`max<dobule>` are two completely different and unrelated functions), the
compilation time and the size of the compiled executable may grow dramatically
when abusing the template system. This is nothing to worry about in PRO2,
because this "abuse" of the template system occurs usually when metaprogramming
(something very advanced) or with poorly written template classes (so, please,
don't do horrible things).

Did I say template classes? Yes, I did:

```c++
template <typename T>
class MyPair {
private:
    T first_;
    T second_;
public:
    MyPair(T first, T second);
    
    T first() const;
    void first(T new_first);

    T second() const;
    void second(T new_second);
};
```

Actually, we can implement something like the [`std::pair`][std::pair] of
`<utility>`, with two template arguments:
```c++
template <typename T, typename U>
class MyPair {
private:
    T first_;
    U second_;
public:
    MyPair(T first, U second);
    
    T first() const;
    void first(T new_first);

    U second() const;
    void second(U new_second);
};
```

Now, we can create a pair with `MyPair<int, double>(1, 2.5)`. But what if we
still want to be able to create a pair with the same type as before, i.e.
instead of `MyPair<int, int>(1, 1)` use `MyPair<int>(1, 1)`? Well, similar
to default function arguments, we can provide template arguments, which can
rely on other arguments:

```c++
template <typename T, typename U = T>
class MyPair {
private:
    T first_;
    U second_;
public:
    MyPair(T first, U second);
    
    T first() const;
    void first(T new_first);

    U second() const;
    void second(U new_second);
};
```

Or what if most of our pairs are of `int`s and we want to use `MyPair` instead
of `MyPair<int>`[^1]?

```c++
template <typename T = int, typename U = T>
class MyPair {
private:
    T first_;
    U second_;
public:
    MyPair(T first, U second);
    
    T first() const;
    void first(T new_first);

    U second() const;
    void second(U new_second);
};
```

Hopefully you will have seen that templates are powerful tools of C++. In fact,
they are kind of another language inside C++, and are used to do advanced things
like metaprogramming, which thankfully we don't do on PRO2.

----

Footnotes:

[^1]: Okay, this is a stupid example: if most of ours pairs are of `int`s, we
    should still use `MyPair<int>` or do a `typedef`, for clarity; but there
    are legitimate uses for default template arguments. For example: did you
    know that most STL containers like [`std::vector`][std::vector] actually
    take **two** template arguments? The second one is rarely used, so
    a default which suits well most cases is provided.

[std::max]: http://www.cplusplus.com/reference/algorithm/max/
[std::pair]: http://www.cplusplus.com/reference/utility/pair/
[std::vector]: http://www.cplusplus.com/reference/vector/vector/
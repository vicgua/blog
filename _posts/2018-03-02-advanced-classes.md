---
layout: post
title: "Advanced classes: destructors, copy constructors and operators"
tags: PRO2
---

In [an earlier post][oop-classes] we saw classes, but we left some of the
most advanced things for later. While most of them are not needed for now,
they have been explained in theory class (our teacher is known to digress
sometimes), so I'll write about them anyway:

# Destructors
Let me introduce briefly dynamically-allocated pointers. You may remember
pointers from EC (if you don't they will be explained later on in PRO2), but
for now let's say that there are a special kind of pointers: unlike most
variables, where the compiler is trusted for getting the memory to store
values in them, dynamically-allocated pointers do not rely on the compiler to
allocate the variable, but explicitly ask for memory. A very stupid and
useless example:
```c++
class Example {
public:
    int *x;

    Example() {
        this->x = new int;
    }
};
```

Here we have a problem: since we ask for the memory, we are responsible to
relase it and return it to the system, but we aren't doing so: when an
`Example` goes out of scope (i.e.: is destroyed because we can't use it
anymore), the memory will continue to be reserved, forever (or until the
program exists, usually). If this happens many times, the system may start
running slow and, eventually, run out of memory and promptly die.

To avoid this, we have to run some code every time an `Example` is destroyed.
Meet the destructor. It is the constructor, but its job is the opposite:
while the constructor sets up the class, the destructor tears it down.
Let's see it:

```c++
class Example {
public:
    int *x;

    Example() {
        this->x = new int;
    }

    ~Example() {
        delete this->x;
    }
};
```

Note that:
1. It is called automatically: we can't call it, because it must be run only
    when the instance is gone.
2. Because of (1), it takes no parameters.
3. As the constructor, it does not have a return type (not even `void`) and its
    name must be the same as the class, but preceded by a tilde (`~`).
4. Like the constructor too, it can't be `const` (in case you wanted a destructor
    that can't destroy anything...).
5. Because of (2) and (4), there can only be one destructor.

# Copy constructors
Sometimes, we have a class `T` and we want to do:
```c++
T obj1(/* ... */);
T obj2 = obj1;
// Which is the same as
T obj2(obj1);
```

To do this, we need a "copy constructor", which, in case you
haven't guessed, it copies the original object. The compiler will
create one by default, and usually it will be perfect; but sometimes, our
program need to deal with strange things that don't cope well being copied.
In this case, we will have to create one ourselves, and it must be the following:

```c++
class T {
    // ...
    T(const T& other) {
        // ...
    }
}
```
The parameter doesn't have to be named `other`.

# Overloaded operators

Let's go back to our complex class: (by the way, if you need a complex
class, be sure to check [`std::complex`][std::complex] to avoid reinventing
the wheel)

```c++
class Complex {
public:
    double re, im;
    
    Complex(double re = 0, double im = 0);

    // Alternative constructor
    static Complex from_polar(double module, double phase);
    
    // Getters
    double module() const;
    double phase() const;
    // Setters
    void module(double new_module);
    void phase(double new_phase);

    // Operations
    Complex add(const Complex& other) const;
    Complex subs(const Complex& other) const;
    Complex mult(const Complex& other) const;
    Complex div(const Complex& other) const;

    // I/O
    void read();
    void print() const;
};
```

While it is perfectly functional (if we omit the lack of any implementation,
and just trust that it'll be correct), let's use it. For now, we will write
a program that gets two numbers, adds them and multiplies them by 10:

```c++
int main() {
    Complex a, b;
    a.read();
    b.read();
    a.add(b).mult(Complex(10, 0)).print();
    cout << endl;
}
```

Great, but it's slightly... ugly. Wouldn't you prefer writing something like
this?

```c++
int main() {
    Complex a, b;
    cin >> a >> b;
    cout << (a + b) * 10 << endl;
}
```

Short and easy to understand. How do we do it? First we will do some changes
to the class to allow the `(a + b) * 10`. Replace the "Operations" sections
with this:

```c++
    // Operations
    Complex operator +(const Complex& other) const;
    Complex operator -(const Complex& other) const;
    Complex operator *(const Complex& other) const;
    Complex operator /(const Complex& other) const;
```

&lt;Side-note&gt;

There's a code style debate here, either of the following is correct for the
compiler, and good and bad style depending on who you ask:
- `T operator+(const T& other) const;`
- `T operator +(const T& other) const;`
- `T operator+ (const T& other) const;`
- `T operator + (const T& other) const;`

&lt;/Side-note&gt;

It's not difficult to understand: we're just changing the name of the function
from `add` to `operator +`. The magic is on how C++ handles this `operator`
functions: if it encounters something like `a + b`, it will try to call
`a.operator+(b)`. The same with all other arithmetic operators and other,
stranger ones, like `*` (prefix unary, as pointer dereference, not as
multiplication), `&` (address of), `[]` (access to an element), `()` (call as a
function), etc. You can find a full list in the
[C++ reference][cppref::operators], in case you are interested.

Another side-note: unlike in `int i; i + 3`, where `i.operator+(3)` is invalid,
`Complex a, b; a.operator+(b)` is valid (but not recommended). `a + b` is called
_syntactic sugar_, and during compilation, the compiler _desugars_ it
internally to the full `a.operator+(b)`. The "sugar" is more developer-friendly,
while the "desugared" version is more machine-friendly. If that sounds familiar,
it's because it's (more or less) the same idea behind some macros (especially
the ones found in C and assembly).

Back on to our `Complex` class, let's add the `cin` and `cout` support. Now
there are two ways of doing it. The first is the one we have used in PRO2:
remove the I/O section from the class and add **after** (outside) the class

```c++
istream& operator >>(istream& is, Complex& c);
ostream& operator <<(ostream& os, const Complex& c);
```

`istream` is the Input Stream class, an instance of which is `cin`.

`ostream` is the Output Stream class, an instance of which is `cout`.

This is called a _non-member overload_. Since we can't modify `istream`
(or `ostream`), we have to use a trick: C++ allows us to change, for example,

```c++
class Complex {
    // ...
    Complex operator +(const Complex& other) const;
    // ...
}
```

for

```c++
// (Outside the class)
Complex operator +(const Complex& left, const Complex& right);
```

Usually this is useless: we can just add it inside the class. However,
this is not possible for classes we can't edit, so we add the operators `>>`
and `<<` to `istream` and `ostream` this way.

We have to return `istream&` and `ostream&` so that we can continue using
the stream (like in `cin >> a >> b` or `cout << a << b`).

To see how this works, let's see the "desugaring" of the following:
```c++
Complex c, d;
cin >> c >> d;
cout << c << d; // ' ' and endl removed for simplicity
// becomes
operator>>(operator>>(cin, c), d);
operator<<(operator<<(cout, c), d);
```

Now, let's see the implementation:
```c++
istream& operator >>(istream& is, Complex& c) {
    is >> c.re >> c.im;
    return is;
}

ostream& operator <<(ostream& os, const Complex& c) {
    os << c.re;
    if (c.im >= 0)
        os << " + j" << +c.im; // + is only for symmetry
    else
        os << " - j" << -c.im;
    return os;
}
```

Maybe the most remarkable thing here is `is` and `os`, which behave as `cin`
and `cout`. This is because both are instances of `istream` and `ostream`,
respectively.

I said there was a second way: this one allows you to use private members
of the class. In this case, that property is moot: there are no private members.
However, sometimes there are, and it's interesting to access them directly.
In this case, the implementation is the same, but in the class declaration,
instead of removing I/O, replace it with the following **inside** the class
(and don't add anything outside):

```c++
    // I/O
    friend istream& operator >>(istream& is, Complex& c);
    friend ostream& operator <<(ostream& os, const Complex& c);
```

`friend` is a keyword that means "this function is trusted, and allowed to
access the private members". While most of the time this is something to be
avoided (why make private members that everyone and their dog can access them?),
`istream` and `ostream` overloads are somewhat special cases, where they should
actually be members of the class but aren't because C++.

Finally, let's recap and analyse the following expression:

```c++
cout << (a + b) * 10 << endl;
```

Okay, we know how the `(a + b)` and the `cout <<` parts work, but how
is `10` allowed, when the function for multiplication
(`operator *(const Complex& other)`) only allows a `Complex`? Well, the compiler
is intelligent enough to implicitly transform the number until it can be used
(within reasonable limits), so the expression is transformed to
```c++
(a + b) * Complex(10.0);
```

(It did `int->double->Complex`)

If we wanted to avoid this (in this case, we don't), we would have to add the
`explicit` keyword to the constructor:
```c++
class Complex {
public:
    // ...
    explicit Complex(double re = 0, double im = 0);
    // ...
};
```

Also note that this only affects constructors with just one parameter (in this
case, since the both are optional, this counts as a constructor with 0
parameters (called _default constructor_), 1 and 2).

# Complete code
To finish, I'll leave a copy of the complete `Complex` class:

{% gist e91c2f7e650cb985dee9a2efb579966a complex.hh %}

{% gist e91c2f7e650cb985dee9a2efb579966a complex.cc %}

Notes:
1. I have used a short version for the constructor.
2. I have added a few more operators, like comparison and unary operators,
    like `-a` (e.g.:, for 1+j2 is -1-j2) and `+a` (which returns the number
    unchanged; only for symmetry with `-a`).
3. Because computers have finite precision, given the following code
    ```c++
    Complex a(4, 5);
    Complex b = Complex::from_polar(a.module(), a.phase());
    bool eq = (a.re == b.re) and (a.im == b.im);
    ```
    `eq` may not be true because, since `sqrt(4*4 + 5*5)` is not exact,
    they may differ due to rounding and other floating point issues,
    so instead I compare if the difference between both is less than a certain
    threshold.


[oop-classes]: {{ site.base_url }}{% post_url 2018-02-21-oop-classes %}
[std::complex]: http://www.cplusplus.com/reference/complex/
[cppref::operators]: http://en.cppreference.com/w/cpp/language/operators
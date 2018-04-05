---
layout: post
title: "Object-oriented programming and classes, or structs++"
tags: PRO2
excerpt: >
    Object-oriented programming (OOP for short) is a programming paradigm,
    evolved from imperative programming. Imperative programming is the kind of
    programming we have been doing at PRO1: storing basic types and collections
    in variables and operating with them. Let's see an example:
---

**Note**: Before reading this, you should probably check
[What you should have learned but didn't][What you should have learned].

**Warning**: Dense post. You have been warned.

---

**Object-oriented programming** (OOP for short) is a programming _paradigm_,
evolved from imperative programming. _Imperative programming_ is the kind of
programming we have been doing at PRO1: storing basic types and collections
in variables and operating with them. Let's see an example:

```cpp
struct Complex {
    double re;
    double im;
};

void read(Complex& c) {
    cin >> c.re >> c.im;
}

void print(const Complex& c) {
    cout << c.re;
    if (c.im > 0)
        cout << " + j" << c.im;
    else if (c.im < 0)
        cout << " - j" << -c.im;
    cout << endl;
}

Complex sum(const Complex& a, const Complex& b) {
    Complex sum;
    sum.re = a.re + b.re;
    sum.im = a.im + b.im;
    return sum;
}

int main() {
    Complex a, b;
    read(a);
    read(b);
    Complex c = sum(a, b);
    print(c);
}
```

It's good, we even have nice function names like `read` and `print` instead
of `read_complex`. However, we could make it better by having `Complex` contain
all of the operations. If you are thinking of functions inside structs, you're
not wrong, but we can do better: `struct` is a construct inherited from C. It is
nice, but it is somewhat retrofitted for this purpose. The C++ way of doing this
is with classes:

```c++
class Complex {
public:
    double re;
    double im;

    Complex(); // Constructor
    Complex(double re, double im); // Another constructor
    static Complex read();
    void print() const;
    Complex sum(const Complex& other) const;
};
```

(We will see how to actually implement those functions later)

Here we notice several thins:

# `public` and `private`
`public` and `private` (along with `protected`, which we won't use now) define
who can access the functions and variables (which, as they are inside a class
are called methods and attributes, respectively) in the block.

This example only contains public methods and attributes, so they can be accessed
throughout the program. It it contained private attributes or methods, only the
class itself would be allowed to use them. An example with private attributes
is available later.

# Constructors
When we create a class variable in C++, this variable does not materialize itself out
of thin air, it has to initialize any other variables it contains. Take for
example a `vector<int>` of size 5:it has to allocate memory for storing 5 `int`s
and (depending on the compiler), set them to 0. It must do this as soon as it is
created, because having to call `v.init()` or something like this would be
annoying, and what if a programmer forgets to call it? The program would crash!

Instead, C++ classes are allowed to create a _constructor_, a method
which is called when a new instance of that class is to be created (and only then).

If we don't declare any constructor, the compiler will give us a default one,
which will call the default constructors of every attribute (for numbers, it
will set them to 0, because _basic types_ do not have constructors).

The default constructor of class `C` is `C()`

Our constructors (we have two are called like this):

```c++
Complex a(2, 3); // a = 2 + j3
Complex b(); // b = 0 + j0
Complex c; // c = 0 + j0
```

You may see that `b` and `c` are the same. In fact, `Complex c;` is an alias
for `Complex c();`: when declaring variables, the default (no parameters)
constructor is called, unless you use the syntax `Complex a(2, 3)`, which
will call `Complex(double re, double im)` (`int`s are implicitly converted to
`double`s).

But what would happen if we remove the `Complex()` constructor?

```c++
int main() {
    Complex a(2, 3); // 2 + j3
    Complex b; // ???
}
```

Compile and...

```
complex.cc:18:11: error: no matching function for call to ‘Complex::Complex()’
   Complex b; // ???
           ^
complex.cc:10:3: note: candidate: Complex::Complex(double, double)
   Complex(double re, double im); // Constructor
   ^
```

ERROR! That's why I defined an empty constructor, because in this case it is
useful. In cases where values should be provided (more on that later), a
default constructor may not be a good idea.

Let's note two **important things**:
1. Constructors have no return type, not even `void` (because they are special
    enough to be exempt from "every function has to return something").
2. Constructor MUST be named as the class.

# `static` methods and attributes
There are some methods (and attributes) which belong conceptually to the class,
but they are not tied to any specific instance. `read` is a good example:
it is a function that creates a new `Complex` from the keyboard input.
What's the point in creating a new `Complex` if you have to create one to be
able to call `read`?

Let's create a program that reads a complex from input and adds one imaginary
unit for... whatever reason. Look, it's not easy to create good examples. Anyway,
let's move on:

```c++
int main() {
    Complex orig = Complex::read();
    Complex jone = Complex(0, 1); // j1
    Complex sum = orig.sum(jone);
    sum.print();
}
```

Note the syntax for calling static methods, which is different from non-static
ones. For fans of short and complex (pun not intended) code, here is the same
with one line (interpretation left as an exercise for the reader[^1]):

```c++
int main() {
    Complex::read().sum(Complex(0, 1)).print();
}
```

# (Method) `const`

Another new syntax is the `const` in an unusual place: at the end of the
method declaration. It means that calling this method does not modify the
instance. In effect, if a method is `const`, all the attributes become `const`
for it. And why would I want that? Aside from the "make sure I don't change
anything by mistake", let's see another example: the following is a possible
implementation for `vector<int>`.

(The class is named `vector_int` because the original `vector<int>` uses
_[Something you don't need to know]_ (that's why it has `<int>`), and it's
missing... well, almost everything it need to be useful,
but it is otherwise correct.)

```c++
class vector_int {
public:
    int size() const;
    void push_back(int elem);
};

int my_function(const vector_int& v) {
    int s = v.size(); // Correct
    v.push_back(3); // ERROR!
}
```

As you may have guessed, if your object is constant, you can call its constant
methods (because they are guaranteed not to change the object), but not
non-constant ones, that may alter the object.

Obvious side-note: the constructor can't be const because... well, why would you
want a constructor that isn't allowed to construct anything?

# Writing the methods
So far, we have only been using the prototype of the function: we say what
methods and attributes it has, and act as if they magically worked. Now, I'm
sorry to spoil this moment, but C++ does not telepathically guess what you
want to do, so we have to implement the methods. Like before, I will dump the
code and explain it later. You may try to figure out what it means, before
going straight to the explanation.

```c++
class Complex {
public:
    double re;
    double im;

    Complex() {
        this->re = this->im = 0;
    }

    Complex(double re, double im) {
        this->re = re;
        this->im = im;
    }

    static Complex read() {
        Complex c;
        cin  >> c.re >> c.im;
        return c;
    }

    void print() const {
        cout << this->re;
        if (this->im > 0)
            cout << " + j" << this->im;
        else if (this->im < 0)
            cout << " - j" << -this->im;
        cout << endl;
    }

    Complex sum(const Complex& other) const {
        return Complex(this->re + other.re, this->im + other.im);
    }
};
```

First things first: we had a wonderful specification of the methods the class
had, and now we're messing with it. While _inlining_ the definition of the
methods is sometimes convenient, this quick-and-dirty system will not end well up
with classes bigger than this, and when (if) we do headers, this won't be possible,
so let's separate declaration and definition instead:

```c++
class Complex {
public:
    double re;
    double im;

    Complex(); // Constructor
    Complex(double re, double im); // Constructor
    static Complex read();
    void print() const;
    Complex sum(const Complex& other) const;
};

Complex::Complex() {
    this->re = this->im = 0;
}

Complex::Complex(double re, double im) {
    this->re = re;
    this->im = im;
}

Complex Complex::read() {
    Complex c;
    cin >> c.re >> c.im;
    return c;
}

void Complex::print() const {
    cout << this->re;
    if (this->im > 0)
        cout << " + j" << this->im;
    else if (this->im < 0)
        cout << " - j" << -this->im;
    cout << endl;
}
```

## Defining methods outside of the class declaration
OK, the first thing we see is that we have to use the same syntax as static
methods everywhere. It kind of makes sense: if we are referring to an object,
we use `.`, but if we are referring to the class itself, we use `::`.

## What is `this`? `->`!? (Extra)
Note the conspicuous absence of `this` in the static method `read`. If we recap,
a static method is one that does not act on an instance of the class, so
non-static methods are those that (should) act on an instance. How? With `this`:
`this->re` is the `re` attribute of the instance, so if we call `c.print()`,
the first line of the function is like `cout << c.re;`.

Now, why `->` instead of `.`? Well, `this` is a special kind of variable called
pointer, which we'll study later this course. For now, just trust me (or don't,
but the compiler agrees with me) and use `->`. Tip: if you ever get it wrong,
most compilers will emit an error which says
`(maybe you meant to use ‘->’ ?)` (G++) or
`did you mean to use '->'?` (Clang++) or something like this.

There are two **important side notes** here:
1. `this` is optional if there are no variables named like the instance attribute.
    For example: in the constructor `Complex(double re, double im)`, we must use
    `this->re` and `this->im` because `re` and `im` alone are the parameters of
    the constructor. To avoid having to track when I have and don't have to use
    `this`, I always use it. I don't know what is the official position of the
    teachers of PRO2 about this. [**UPDATE**: In PRO2 professors usually don't
    use `this`, but I don't think it's forbidden or anything like that.]
2. A static method can not use `this`, but a non-static method does not have
    to use it, like it does not have to use all of its parameters. However let's
    remember the following lemma: if you don't use a parameter, you probably
    shouldn't ask for it. Likewise, if you don't use `this` (or implicit `this`,
    see point 1), you

## Repeating `const`
("Repeating" meaning that we have to write it in the declaration and definition)

Given that we already declared `print` as `const`, one would think that we don't
need to repeat it, right? One would be wrong. Due to an advanced form of
overloading, `void f()` and `void f() const` may be declared at the same time,
just like `void f()` and `void f(int)`, so yes, **you have to repeat `const`
when defining the function.**

## Not repeating `static`
Since we have to repeat `const`, one would guess that we have to repeat `static`.
And one would be wrong. Yes, again. One should have learned not to make
assumptions about C++. This time there are two factors for why:

1. Unlike `const`, `static` is not considered overloading, so declaring both
    `static void f()` and `void f()` is an error.
2. `static` is a keyword inherited from C, and it has a very different meaning.
    Yes, C++ has a bad habit of reusing keywords. Anyway, using `static` outside
    a class fiddles with the way the compiler creates the executable, and if you
    do it unknowingly, you'll be given the "Bug of the week" award for the most
    obscure and difficult to debug error message. So don't do it.

# Another example
This is almost all I wanted to write about, but I want to include another
example. This is the same we were show in class, with some modifications,
mostly because the original was in Spanish and, alas, I don't have photographic
memory (nor a copy of the program), but the idea is the same. Also, we were
only shown the declaration, so the implementation is mine.

```c++
class Student {
private:
    int id;
    double grade;
    bool has_grade_;

    static const double MAX_GRADE = 10;
public:
    // Constructor
    Student(int id);
    Student(int id, double grade);

    // Getters
    int get_id() const;
    bool has_grade() const;
    double get_grade() const;

    // Setters
    void set_id(int id);
    void set_grade(double grade);
    void clear_grade();

    // Static getters
    static double get_max_grade();
};

Student::Student(int id) {
    this->id = id;
    this->has_grade_ = false;
    this->grade = 0; // Dummy value
}

Student::Student(int id, double grade) {
    this->id = id;
    if (grade > Student::MAX_GRADE) {
        this->has_grade_ = false;
        this->grade = 0;
    } else {
        this->has_grade_ = true;
        this->grade = grade;
    }
}

int Student::get_id() const {
    return this->id;
}

bool Student::has_grade() const {
    return this->has_grade_;
}

double Student::get_grade() const {
    if (not this->has_grade_)
        return -1; // ERROR
    return this->grade;
}

void Student::set_id(int id) {
    this->id = id;
}

void Student::set_grade(double grade) {
    if (grade > Student::MAX_GRADE)
        return;
    this->has_grade_ = true;
    this->grade = grade;
}

void Student::clear_grade() {
    this->has_grade_ = false;
}

double Student::get_max_grade() {
    return Student::MAX_GRADE;
}

int main() {
    int id;
    cin >> id;
    Student s(id);
    cin >> s.grade; // ERROR!
}
```

## Dummy values
Sometimes, if a grade value isn't correct, I return a -1. The usual behaviour
is to error, but this isn't part of the temary of PRO2, as far as I know,
so I simply return an impossible value.

## Private attributes
Here you can see private attributes. Those are only accessible by the methods
of the class.

So for example, the following wouldn't compile:
```cpp
int main() {
    int id;
    cin >> id;
    Student s(id);
    cin >> s.grade; // ERROR!
}
```
```
test.cc: In function ‘int main()’:
test.cc:8:10: error: ‘double Student::grade’ is private
   double grade;
          ^
test.cc:78:12: error: within this context
   cin >> s.grade; // ERROR!
            ^
```

There can also be private methods, which can only be called inside the class.

## Getters and setters
This shows a recurring pattern in programming: getters and setters. The theory
is this: You have a private variable which stores the actual data, and two
public methods, which can edit its value indirectly. They are usually called
`get_var` and `set_var`, although `has_var`, `is_var` and `clear_var` are also
common for booleans.

This is useful if you want to restrict the access to a variable. In the case of
grades, we only want to show the grade if it is set, and we want to set the
variable `has_grade_` when we set the grade, and clear it when we clear the
grade.

### About underscore in `has_grade_`
As `has_grade` is both the name of a method and an attribute, I have to rename
one of them. If one of them is private, usually this is the one renamed.
A trailing underscore is the usual character used in these situations.

### Other conventions for accessors (Extra)
In other languages, accessors (getters and setters) are built within the
language. Let's consider Python (although many languages like Ruby,
Javascript/Typescript or D implement them in some way):

```python
class Example:
    _var: int  # _ means "private" (kind of)

    @property
    def var(self):
        return self._var

    @x.setter
    def x(self, value):
        self._var = value

# Used as:
e = Example()
e.var = 5
print(e.var)  # -> 5
```

On the other hand, there are Java and Objective-C, for example, that, while
they don't implement at the language level such constructs, they do have
strong conventions to use them:

```java
// Java
class Example {
    private int var;
    public int get_var() {
        return this.var;
    }
    public void set_var(int var) {
        this.var = var;
    }

    public static void main(String[] args) {
        // Used as:
        Example e = new Example();
        e.set_var(5);
        System.out.println(e.get_var()); // -> 5
    }
}
```

However, C++ has neither a built-in method like Python nor a standard convention
like Java. In this document, I've used the Java style:

```c++
class Example {
private:
    int var;
public:
    int get_var() const;
    void set_var(int new_x);
}
```

But other conventions for accessors are also used:

Overload style:
```c++
class Example {
private:
    int var_;
public:
    int var() const;
    void var(int new_x);
}
```

Objective-C style:
```c++
class Example {
private:
    int var_;
public:
    int var() const;
    void setVar(int var);
}
```

You may find any of them, depending on who wrote the code: the STL
(e.g.: `std::vector`) uses mostly the overload style (with some variations, like
references where appropiate), while the Qt library (commonly used for GUIs) uses
Objective-C style.


## Static attributes
A class can have static attributes (which are, more often than not, constants).
This are attributes that are shared amongst all instances. In this case,
the maximum grade any student can have is 10, regardless of the student.

In fact, since it is a constant, there's no need to control the access, so
usually I'd write it directly as public, without the useless `get_max_grade`
method.

---

If you read all the post and reached this line, congratulations, you're
done! Also, congratulations, because if you've understood this, you've
understood all the theory of the first weeks of the course.

However, if you are here only because you scrolled past the article,
the bad news is that information will not be automatically transferred to your
brain. Sorry.

---

Footnotes:

[^1]: In case you haven't guessed, "as an exercice for the reader" translates to
    "I'm too tired to explain it, and most of you already have too much things to
    assimilate, so I won't digress further. Have fun."

[What you should have learned]: {{ site.base_url }}{% post_url 2018-02-20-what-you-should-have-learned %}

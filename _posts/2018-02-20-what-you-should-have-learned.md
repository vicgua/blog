---
layout: post
title: What you should have learned but didn't. Of structs and overloads
tags: PRO1 PRO2
---
Today we did the first class of PRO2. It was... "intense", but had a big problem:
the teacher assumed knowledge that wasn't taught in PRO1, so I am going to
summarize it here to consolidate ideas (and for quick reference near the exam,
I hope):

#### Conventions
Through this blog, unless stated otherwise, fragments of code should be prefixed
with the following code:
```c++
#include <iostream>
// Other includes as needed, like <vector> or <string>

using namespace std; // Following the conventions of PRO1 and PRO2
```

# Structs and their hidden wonders
In PRO1 we did structs, but only in a basic level. The following structures were
normal:
```c++
struct Student {
    int id;
    vector<double> grades;
};

void read_student(Student& s) {
    cin >> s.id;
    int n;
    cin >> n;
    s.grades = vector<double>(n);
    for (int i = 0; i < n; ++i)
    cin >> s.grades[i];
}

void print_student(const Student& s) {
    cout << s.id << " -- ";
    int n = s.grades.size();
    for (int i = 0; i < n; ++i) {
    if (i > 0) cout << ' ';
    cout << s.grades[i];
    }
    cout << endl;
}

int main() {
    Student s;
    read_student(s);
    // Do something
    print_student(s);
}
```

In fact, this is a perfectly acceptable way of reading and printing structs...
In C. However, C++ prefers another way of doing things: reading and printing
from inside the struct itself:
```c++
struct Student {
    int id;
    std::vector<double> grades;

    void read() {
    cin >> id;
    int n;
    cin >> n;
    grades = vector<double>(n);
    for (int i = 0; i < n; ++i)
        cin >> grades[i];
    }

    void print() {
    cout << id << " -- ";
    int n = grades.size();
    for (int i = 0; i < n; ++i) {
        if (i > 0) cout << ' ';
        cout << grades[i];
    }
    cout << endl;
    }
};

int main() {
    Student s;
    s.read();
    // Do something
    s.print();
}
```

Easy things first: we call functions in an struct with `ins.func(...)`, where
`ins` is an instance of the struct (i.e. not `Student`, but a variable of type
`Student`) and `func` the function we want to call.

When a function is inside an `struct`, its _scope_ includes the instance of the
struct. Translation: If the compiler doesn't find the variable declared in the
function, it will try to look it up on the struct. Think of the following loop:
```c++
int sum;
sum = 0; // Let's separate declaration and assignment for clarity
int i;
i = 0;
while (i < 10) {
    int square;
    square = i * i;
    sum = sum + square
    i = i + 1
}
sum = sum + square // INVALID!
```

Here, the while has its own scope, so `square` is not visible outside of the
loop. However, it can access variables of the scope that contain it, so it can
read and change `sum` and `i`.

For the structs, a similar thing happens: the function's (technically, _method_,
because it is inside an struct) scope is inside the struct's scope, so it
can access its variables.

# Function overloading
Another wonderful addition to C++ is _function overloading_, which is a fancy
name for "two different functions with the same name".

## Based on the types of parameters

Imagine the following:

```c++
void print_int(int i) {
    cout << i << endl;
}

void print_double(double x) {
    cout << x << endl;
}
```

(Here the implementation of both functions is basically the same, we will see
later why, but they can be implemented in different ways)

While it is totally correct (and, in fact, the only correct way in C), this is,
again, not the "C++ way" of doing it, because it is boring and redundant having
to say `int i; print_int(i);` and `double x; print_double(x)`: we already know
that `i` and `x` are an `int` and a `double`, there's no need to repeat ourselves:

```c++
void print(int i) {
    cout << i << endl;
}

void print(double x) {
    cout << x << endl;
}
```

Here, if you write `print(var)`, the compiler will figure out which of the
`print`s it should call: if var is an `int`, it will call the former, if
it is a `double` the latter.

In fact, the compiler is already doing this for `cout`, and because of this
the implementation of the functions is the same, otherwise we would have to write
something like `cout_int << i; cout_endl << endl; cout_double << x`..., which
is... let's say "not ideal".

## Based on the number of operators

We should note that overloading also works based on the number of parameters:
```c++
bool divisible(int q, int p) {
    return q % p == 0;
}

bool divisible(int q) {  // Assume that p = 2
    return q % 2 == 0;
}
```

Here, the compiler knows that if you call `divisible(x)`, it will be the second one,
and `divisible(x, y)`, the first one (Assuming `int x, y;`).

In fact, overloaded functions can call the other overloads, so a better way to
write the former is:
```c++
bool divisible(int q, int p) {
    return q % p == 0;
}

bool divisible(int q) {  // Assume that p = 2
    return divisible(q, 2);
}
```

As we are already here, let me comment that this is so common that C++ has
a shortcut for this:

```c++
bool divisible(int q, int p = 2) {
    return q % p == 0;
}
```

(Meaning that if called as `divisible(x)`, `p` will default to 2. It has no
effect if called with `divisible(x, y)`)

Finally, we should note that overloads can differ by both the number and the
type of the variable. The following are all different overloads:
```c++
void f(double);
void f(int);
void f(int, double);
void f(double, int);
// Whatever other combinations you want
```

## Calling overloads
You can check that the following code

```c++
void print(double x) {
    cout << "(double) " << x << endl;
}

int main() {
    int n = 0;
    print(n);
}
```

outputs `(double) 2`. Why? Because of an **implicit conversion**: if a parameter
type does not match the function, it will be "upgraded" or converted if possible
(`int` → `double` is considered an upgrade) to match it. However, what if we
add a new overload?

```c++
void print(double x) {
    cout << "(double) " << x << endl;
}

void print(int x) {
    cout << "(int) " << x << endl;
}
```

But then, the old `main` will print `(int) 2`. We want to get the same behaviour,
so we make the conversion explicit:

```c++
int main() {
    int n = 0;
    print(double(n));
}
```

It will now show `(double) 2`.

## Restrictions
We have seen that you can overload function with different number and/or type of
parameters. Can you also overload functions with different returns? Well...
Yes and no. Yes: overloads are not required to return the same type, so the
following is valid:

```c++
int sum(int x, int y) {
    return x + y;
}

double sum(double x, double y) {
    return x + y;
}
```

Also no: you can not overload based ONLY on the return type. Think of the
following:

```c++
int number() {
    return 42;
}

double number() {
    return 3.14;
}

int main() {
    double a = number();
    int b = number();
    double c = int(number());
    int d = double(number());
    // d is INVALID: Conversion double -> int must be explicit, but let's ignore
    // that for the sake of the example
    cout << a << ' ' << b << ' ' << c << endl;
}
```

What should this program print? The answer is: whatever you want to. `a` and
`b` would be more or less clear: `3.14 42`. And `c`? Should it be `42.0`?
The answer is that it depends on what you meant, and the C++ standard takes the
drastic-but-pragmatic solution: **ban overloads based on the return type**.

----

And this concludes the things that you are expected to know but aren't taught
in PRO1. Have fun with PRO2! /s

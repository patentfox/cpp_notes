# C++ Learnings

## auto

auto ignores references while inferring type.

```cpp
auto i = 10, &r = i;    // r is an int reference
auto x = r;     // inferred type of x is int, not int&
auto &y = i;    // now y is a reference to i
```

auto ignores top level const-ness while deducing type. However, low level
const-ness is preserved.

```cpp
const int i = 0;    // top level const
int j = 0;
// lower level const. Here, p is const, value of j can still be changed via p
int* const p = &j;
```

```cpp
const int ci = 1, &cr = ci;
int& r = ci;    // ERROR: cannot bind int& to const int
auto b = ci;    // top level const is dropped
b = 10;
auto c = cr;    // cr is alias for ci, which has top level const
c = 20;

// d is deduced to be of type const int*, which is a pointer to const int
// d cannot be used to modify ci
auto d = &ci;
*d = 30;        // ERROR: cannot modify d
```

If deduced type should be a top-level const, add const to auto in declaration

```cpp
const auto x = i;
```

Another mind bender

```cpp
const int i = 10;
auto &cr = i, *cp = &i;     // Works, auto is deduced as const int
int a = 20;
auto &b = a, *p = i;        // ERROR
cout << b << " " << i << endl;
```

Line 4 is an error, as `auto` will be normal `int` according to type deduced
for b, and `const int` for `p`. So, effectively, we are asking auto to work as
both int and const int in same statement. Hence it is an error.

```cpp
int i = 10;
const auto &j = i;
j = 20;         // ERROR: can't change i using j
```

---

## decltype

`sum` has whatever type `f()` returns. Note that `f()` is not called.

```cpp
decltype(f()) sum = x;
```

decltype handles top level const-ness and references differently from auto. If
applied to a vairiable, it preserves top-level const-ness and whether the
variable is a reference.

```cpp
int i = 2, &r = i;
decltype(r) s;  // ERROR: an int reference should be initialized;
const int j = 0;
decltype(j) p;  // ERROR: a const int needs to be initialized
```

`decltype` is the only context where a reference is treated differently from the
object to which it refers.

There is difference between application of `decltype` to variable vs expression.
`decltype` when applied to an expression yields a reference, when the expression
is an lvalue.\
Consider int `p = 10;`. Note that `*p` is an expression, `p` is not, it's a
variable.

```cpp
decltype(1+2) x = 10;   // x is of type int
int *p = &x;
int q = 10;
decltype(*p) r = q;     // r is now int&, as *p is an expression and lvalue
cout << ++q << " " << r << endl;    // 11 11

// error, as declype(*p) yields an int reference
decltype(*p) s = 10;  // ERROR
```

When we apply decltype to any variable, we get the type of that variable.
However if we apply parenthesis to the variable, it is treated as an expression.
Since the expression is now an lvalue, we get a reference type.

```cpp
int i = 0;
decltype(i) j;  // OK, j is of type int
decltype((i)) k;    // ERROR: k is of type int&, need an initializer
```

In general, the form `decltype ((variable))` always yields a reference type of
variable, however `decltype(variable)` may or may not be a reference type
depending on type of variable.

---

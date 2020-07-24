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

## string

### Different string constructors/initializers

```cpp
string s1;
string s2 = s1;
string s3(s2);
string s4 = "Hello";
string s5("Hello");
string s6(3, 'c');      // "ccc"
```

`getline` function reads from stream upto and including where it finds a
newline character, and stores it in the string, without newline character.\
So, newline character is consumed from stream, but not stored in string.

```cpp
string line;
while(getline(cin, line)) {
    cout << line << endl;
}
```

Some interesting behavior with string concatentation

```cpp
string s1 = "abc";
string s2 = s1 + "def" + "ghi" + 'l'; // OK
string s3 = "abc" + "def" + s1; // ERROR
```

Initialization of `s3` is invalid as one of the operands to `+` has to be a
string. This is not a problem for `s2`, as `s1 + "def"` yields a temporary
string object which can be valid left operand to + operator.

Be careful while mixing signed and unsigned types in comparison expressions.
This prints `false` (0) as i is converted to `unsigned int` which wraps up to a
very large number.

```cpp
int i = -10;
string::size_type j = 20;
cout << (j > i) << endl;
```

---

## Arrays

Array initialization in 1 dimension

```cpp
int a1[10];  // possibly contains random values
int a2[10] = {};    // All values are 0
int a3[10] = {0};   // All values are 0
int a4[10] = {1,2}; // All values after first 2 are 0
// values starting from index 3 are 3, 4 and 5.
// Rest all are 0, including before index 3
int a5[10] = {[3] = 3,4,5};
```

Array initialization in multiple dimensions

```cpp
int b1[3][4];   // random values
// b2[0][0] is 1, rest all are 0
int b2[3][4] = {1};
// b[0][0] = 1, b[1][0] = 2, rest all are 0
int b3[3][4] = {{1}, {2}};
// b[0][0] = 1, b[1][0] = 2, b[1][1] = 3, rest all are 0
int b4[3][4] = {{1},2,3};
```

Using range based for loop with multidimensional arrays

```cpp
int b[3][4] = {1,2,3,4,5,6,7,8,9,10,11,12};
for(auto row : b) {         // (1) ERROR
    for(auto col : row) {
        cout << col << " ";
    }
    cout << endl;
}
```

The above code does not compile. (1) tries to copy 1D array to row, which ends
up copying the `int*` pointer. `int*` is not iterable using `begin()` method.
Hence, this is an error.\
The fix is simple. Change (1) to `const auto& row : b`

Multidimension array and pointers

```cpp
int b[3][4] = {{1,2,3,4}, {5,6,7,8}, {9,10,11,12}};
int (*p2)[4] = &b[1];       // (1)
cout << p2[1][1] << endl;   // 10
```

The declaration in line (1) reads as: \
p2 is a pointer to an array of 4 ints (inside out starting from variable name)
starting from 5.\
Thus, `p2[1]` is equal to the array `{9,10,11,12}`.\
This is different from below

```cpp
int* p3[4];
p3 = &b[1];     // ERROR
p3[0] = b[1];   // OK
```

Here, p3 is an array of int pointers. Size of array is 4.

---

const_cast can be used to change a const to non const type. However, if the
object was originally const, the result of writing to it after casting away
constness is undefined.

```cpp
const char* str = "Kaustubh";
char* p = const_cast<char*>(str);   // no error
p[1] = '0';     // undefined behavior, gives bus error on my mac
```

---

## Function matching - Overload resolution

This is used to decide which function to call when calling an overloaded
function.\
*Candidate functions* - Those functions which have same name and are visible.\
*Viable functions* - Those candidate functions where number of arguments match,
and types of argument and parameters are exact match, or argument is
convertible to parameter type.\
If the number of viable functions are more than 1, then following rules apply.

Lets say there are 2 viable functions A and B. A will be selected if

1. At least one argument is better match for A as compared to B.
1. There is no argument of B which is a better match than A.

If these conditions are not met, the function call is ambiguous, and a compiler
error will be thrown.\
Better match is the one which doesn't require type conversion. Here type
conversion includes both narrowing and widening conversions.

Examples

```cpp
void f(int a, int b) {      // (1)
    cout << "Inside f(int a, int b)" << endl;
}
void f(int a, double b) {   // (2)
    cout << "Inside f(int a, double b)" << endl;
}
void f(double a, double b) {    //(3)
    cout << "Inside f(double a, double b)" << endl;
}

f(1, 2);     // (1)
f(1.0, 2.0); // (3)
f(1, 2.0);   // (2)
f(1.0, 2);   // ERROR: ambiguous
```

There are further rules to determine which conversions rank above another,
but having your code rely on those arcane rules is not a great practice.

---

## Function pointers

Multiple ways to declare function pointer types

```cpp
double f(int, int);    // assume this function type

int main() {
    using ftype1 = double (int, int);           // (1)
    // With trailing return type, auto is required at beginning
    using ftype2 = auto (int, int) -> double;   // (2)
    typedef double ftype3(int, int);            // (3)
    ftype1* pf = f;

    // Declarations above are easier than below, which works the same way.
     double (*pf2)(int, int) = f;
}
```

All the types defined in (1), (2) and (3) are equivalent.

```cpp
typedef decltype(f) ftype4;
```

This also works, but not in case the function `f` is overloaded.

---

## Assignment and copy constructors

In (1), copy constructor of A is called, not assignment operator. In (2), assigment operator is used.

```cpp
A B::getA() {               // v1
    return this->a;
}
A& B::getA() {              // v2
    return this->a;
}
A obj1 = objB.getA();       // (1)
A obj2;
obj2 = objB.getA();         // (2)

const A& obj3 = objB.getA();   // (3)
A& obj4 = objB.getA();         // (4)
```

In case `getA()` is defined like v1, copy constructor will always be called when
returning the object. Copy ctor is called once for (3). However, if call is made
like (1), copy ctor is still called only once as the returned temporary is
assigned to obj1.\
In case of v2 and (1), `getA()` returns a reference but it is copied to obj1.
Hence copy constructor is called again (only once) here. In case of (3) and
`getA()` defined as in v1, it is error to omit `const`, as plain non const
reference cannot refer to a temporary returned by `getA()` method.\
(4) is valid only with v2. In this case copy ctor is not called.\
If (2) is used with v1, copy constructor is called to create a temporary object
to return by value. Assignment operator is also called.

---

* A friend declaration only specifies access, that is, this function can access
  private variables. It is NOT a substitute for declaration, which is required
  by any users outside the class. It may appear to work in some cases due to ADL
  (*Argument dependent lookup*, later on this), however it will not work always
  like a real declaration.

* Within a class declaration, members which define a type must appear before
  they are used. This has something to do with name lookup (later). This
  restriction is not applicable to data or function members. For example,

```cpp
class Screen {
public:
    using pos = std::string::size_type;
private:
    pos cursor_;
}
```

* If you make a function inline, its definition needs to be visible to compiler.
  That is, the function must be defined in header class itself.

* A `const` member function that returns `*this` should have a return type that
  is a const reference.

* It is possible to overload a function based on its const-ness. A const
  function will only match to a const calling object. However, a non const
  function can call both a const version as well as a non const version, but
  non const version is a better match (among 2 viable functions)

---

## Forward Declaration

A class can be declared with just the class name, without the body.

```cpp
class Screen;
```

This allows the typename `Screen` to be used in limited ways before the whole
class declaration is available. We cannot make an object of `Screen` before the
class is completely defined, but we can declare pointers or references. For the
same reason, a class cannot contain an object of its type, but can contain its
reference or pointer.

A friend declaration inside a class is allowed to be defined as well at same
place. Not sure how this is useful though :)

```cpp
struct X {
    friend void f() { ... }
    X() { f(); }    // ERROR
}
```

However, any use of `f()` is not allowed until the declaration of `f()` is seen
outside the class as well.

```cpp
void f();
struct X {
    friend void f() { ... }
    X() { f(); }    // OK
}
```

Similarly, just declaring `f()` after the class is not enough. Declaration
needs to be seen before its use.

---

When a member function is defined outside of a class, we need to use class name
with scope resolution operator to indicate that the function belongs to the
scope of specified class. Anything in function defintion that occurs after the
name of function is automatically considered to belong to class scope.

```cpp
class A {
public:
    typedef std::vector<string>::size_type index;
    index fun(index aa);
}
A::index A::fun(index aa) { ... }
```

In the above code, the parameter `aa`'s type is not required to be qualified
with `A::`, but the return type has to be qualified, as it occurs before the
function name (which contains `A::`).

For name resolution, class definitions are processed in 2 phases.

* All the member declarations are compiled first.
* Function bodies are compiled only after the entire class has been seen.

Thus, when defining inline functions, we can use data members in the body even
if they are declared later in the class, as member definitions are processed
after declarations.

Order of initialization in constructor initializer list

```cpp
class X {
public:
    int j;
    int i;

    // WARN: field i is uninitialized when used here.
    X(int a) : i(10), j(i * 10) {}
};
```

The order of member initialization in constructor initializer list is always
same as the order in which members are declared in class, NOT the one in which
the appear in member initializer list.\
The above code compiles but initializes `j` with garbage value of `i`.
ALWAYS define the members in initializer list in same order as declared in the
class definition.\
Another useful practice is to only use constructor parameters for
initialization of members.

---

## Implicit conversion

Implicit conversions are allowed for user defined types using single argument
constructors. Only one level of implicit conversion is allowed.

```cpp
class X {
private:
    string str;
public
    X(string s);
}
void fun(X xobj);
```

The above can be called as follows

```cpp
string s = "some string";
fun(s);         // (1) allowed, implicit conversion
fun("abc");     // (2) not allowed
```

(2) is not allowed, as it requires 2 conversions, `char*` to `string` and
`string` to `X`.

We can also prevent these implicit conversions by marking the constructor as
`explicit`.

```cpp
explicit X(string s);
```

---

## Aggregate Class

It is a class whose object can be instantiated directly using a brace
initialization syntax, much like plain a struct.

```cpp
class X {
public:
    int a;
    string b;
};
// Intialization
X xobj = {10, "abc"}
```

A class is an aggregate class if

* It has only public members.
* No user specified constructors.
* No base class.
* No in class initializers.
* No virtual functions.

Generally not a good idea to use it. If a member is added ever to the class,
all the invocations will need to be updated.

---

## Literal class

Literal classes are the ones where their objects can be created at compile time,
and assigned to constexpr variables, i.e, the members are of a literal type.\
This means, that the at least one constructor for a literal class have to be
`constexpr`.\
A constexpr constructor must initialize all the class members. For this to work,
all data members must also be of literal type. The class must not define a
custom destructor.

Declaring a member method as `constexpr` does not implicitly make it a `const`
method as well. This used to be the case in C++11, but was fixed later in C++17.

---

## static keyword

Static data members must not be initialized inside the class. A definition has
to be provided outside the class.

```cpp
class X {
public:
    static string str1 = "abc";         // ERROR
    static const string str2 = "abc";   // ERROR
    static string str3;
    static const string str4;

    static const int i1 = 10;           // OK
    static int i2 = 10;                 // ERROR

    static constexpr int i3;            // ERROR
    static constexpr int i4 = 10;       // OK
};

// in .cpp file
string X::str3 = "abc";
const string X::str4 = "def";
const int X::i1;
constexpr int X::i4;
```

However, we are allowed to provide in-class initializer for static members of
const integral type, for example, `i1`. It is strongly suggested to also define
even the static const integral types outside the class, as simplest cases like
taking the address of `i1` will cause compilation to fail otherwise.

constexpr data members have to be declared as `static`, and be provided with an
in class initializer. We still should define the member ouside class.

static members can exist as incomplete types. This means, that the static member
does not have to be fully defined at the point of its declaration.\
Therefore, static members can be of the same type as their class

```cpp
class X {
    // ...
    static X obj1;
    X* obj2;
    X obj3;         // ERROR, X cannot contain an object of type X
};
```

A static member can even be default argument for a class method. In above class,
it is valid to declare a method such as

```cpp
...
void foo(X = obj);
```

This is not possible for non static members, as there is no implicit object
from which the value can be taken.

---

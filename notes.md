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

## IO classes

IO classes (`istream`, `ostream` and their hierarchy) does not allow to copy or
assign the objects from one to another (deleted copy and assign operators).\
Hence, no method can take an i/ostream object parameter, or return an i/ostream
object. The common way is to pass or return these as ordinary references.
Passing a const reference is generally not useful, as any meaningful opreation
including read or write, requires changing the internal state of i/ostream
object.

### using iostream::iostate

i/ostream objects can have an internal state called `iostate` which indicates
if the stream is good for read/write operations. It can have following states

* `good` - all flags are clear. Equivalent to 0.
* `bad` - generally indicates some serious unrecoverable error, like
  hardware/driver failure.
* `fail` - set when invalid input in encountered, like reading a string when an
  int is expected.
* `eof` - when end of file is encountered. fail bit is also set when eof is
  found.

`cin.fail()` method tests for both bad bit and fail bit. Hence, it is a good
test to see if the stream is in correct state for processing input.

```cpp
while(cin >> i) {
    // ...
}
while(!cin.fail()) {
    // equivalent
}
```

Internally this is implemented by `cin >> i` returning `cin`, and having
i/ostream object overload `bool()` operator, which allows the object to be
treated as a boolean value in valid contexts.

There's a special class of type conversion operators which allows us to do that
with user defined types (*note the lack of return type*).

```cpp
explicit operator bool() {
    // ...
    return true/false;
}
```

This overloaded conversion operator defines a conversion from the enclosing type
to `bool`.

Getting back to i/ostream, we can manualy change the state of a stream using

* `cin.setstate(iostream::ios_base::badbit)` adds `bad_bit` to current state.
* `cin.clear()` clears the current state and sets it to good.
* `cin.clear(iostream::ios_state::eofbit)` clears the current state and sets it
  to `eofbit`.

How to clear a single flag, i.e. just remove the bad state?

```cpp
cin.clear(cin.rdstate() & ~cin.badbit)
```

### stream flushing

By default, `ostream` is buffered, which means that output may be written at a
later time.\
When is the stream flushed?

* When endl or ends are used on stream.
* When cout.flush() is explicitly called.
* If stream manipulator `unitbuf` is used. It causes stream to flush after each
  write. This behavior can be turned off again using `nounitbuf`
* Reading from/writing to a tied stream.

The last point requires more detail. An i/ostream can be tied to at max one
`ostream`. Multiple i/ostreams can tie themselves to same `ostream`. If, on the
stream which ties itself (`s1`) to ostream (`s2`), any read/write oepration
occurs, `s2` will automatically be flushed.

For example, `cin` and `cerr` are tied to `cout`. Whenever we read something
from `cin`, or write something to `cerr`, `cout` is guaranteed to be flushed.

```cpp
cin.tie(&cout);     // for illustration
ostream* current_tie = cin.tie();
// break the existing tie and return it.
ostream* current_tie = cin.tie(nullptr);
```

Some bullet points for file operations

* Cannot open an opened file - puts the `ostream` in bad mode.
* Can open a file either via constructor, or via an open call later.
* File does not need to be closed explicitly. Uses RAII idiom.
* We can close a file, and use same i/ostream object to open another file later.

### File open modes

1. You can find the file open flags in following classes/type aliases
   * ios_base
   * ios
   * ofstream
   * ifstream
1. `in` mode is default for file opened by `ifstream`. `out | trunc` is default
   for `ofstream`.
1. `in | out` opens the file and seeks to begin. Any write will overwrite bytes
   from beginning.
1. `out` or `out|trunc` or `trunc` clobbers the file (deletes all contents)
1. `app` or `out|app` seeks to end before writes (appends)
1. `app|trunc` is invalid, results in file open error.
1. `stringstream` is a lot like `iostream`. It offers 2 additional methods to
   instantiate itself from `string` (for reading as `istringstream`) or return
   the underlying string (`osstream.str()`)

---

## Sequential Containers

Containers can hold almost all types (example of exception - references).\
However, some operations, that can be performed on elements of containers,
specify their own constraints on the element types. For example - Creating a
vector using ctor which takes size is valid only for those element types which
have a default ctor.

Iterator operations like `iter + n` are only supported for containers which
provide random access, like `vector`, `deque` etc. Not supported for `list` etc.

sequential contianers define following type aliases

1. `value_type`
1. `reference`
1. `const_reference`

```cpp
vector<string>::value_type s1 = "abc";
// error, as non const lvalue reference cannot bind to unrelated type.
vector<string>::reference s2 = "def";   // ERROR
vector<string>::const_reference s3 = "ghi";    // OK
```

Prefix iterator increment operator yields lvalue, while postfix version yields
rvalue.

To get an iterator for a container, use functions like following

```cpp
vector<string> svec = {/*...*/};
vector<string>::iterator it1 = svec.begin();
// OK, will call overloaded version of begin() with const
vector<string>::const_iterator it2 = svec.begin();  // OK
vector<string>::iterator it3 = svec.cbegin();       // ERROR
vector<string>::const_iterator it4 = svec.cbegin(); // OK
vector<string>::const_iterator it5 = it1;   // OK, vice versa is not.
const vector<string> csvec = {/*...*/};
auto it6 = csvec.begin();   // it6 is const_iterator
auto it7 = svec.cbegin();   // it7 is const_iterator
```

When using copy constructor for containers, both container type and element type
must match. However, if you use iterator ranger version of ctor, container types
do not have to match, however, elements must be convertible from source to
destination type (implicitly at least).

---

## array

`array` container has its size as a part of its type. So, array of size 10 is a
different type from array of size 2.

```cpp
array<int, 3> arr1 = {1,2,3};
// other elements are value initialized
array<int, 3> arr2 = {1};   // OK
// Array assignment like below is not allowed for C-style arrays
decltype(arr1) arr3 = arr1; // OK, as types match
```

Some more operations on containers

```cpp
swap(c1, c2)    // Swaps contents (same type of container and elements)
c1.swap(c2)
c1.assign(n, elem)  // replace contents of c1 with n copies of elem
c1.assign(<initializer_list>)
c1.assign(beg_it, end_it)
c1.assign(c2)   // ERROR, not available, except strings
```

`assign` operation is not valid on array type.

`swap` operation runs in const time, as underlying storage is swapped. However,
this is not the case for array, where elements are swapped one by one.

containers allow the use of relational operators like <, == etc. However, these
are valid only if the underlying element type supports these operations.

most of the containers provide `insert` member function, that takes an iterator,
and inserts the element before the iterator. This allows to insert an element at
beginning as well as at end (using `end()` iterator)

```cpp
svec.insert(iter, "abc");
svec.insert(iter, 10, "abc");
svec.insert(iter, slist.begin(), slist.end());
svec.insert(iter, {"abc", "def"});
```

Special note - source iterators must not be from the same container as
destination, it will cause a runtime error.

---

`emplace` is the new operation added in C++11 which allows us to construct the
elements in place rather than copy them to the container. The arguments are the
same as those required by constructor of the element.

```cpp
// Assume following ctor
Employee::Employee(string id, double salary) { ... }

// Using emplace
vector<Employee> vec;
vec.emplace_back("emp-1", 5000.0);
```

There are other version like `emplace` and `emplace_front` corresponding to
`insert` and `push_front` operations.

---

For containers which support indexing\
`c[n]` is undefined if n is outside the range.\
`c.at(n)` throws out_of_range exception instead.\
`at` and subscript access to elements are only provided for containers that
support fast random access.

Access members return an ordinary reference, if the container is not a const
object. So, following code is valid

```cpp
vector<int> vec = {1,2,3};
cout << (v.front() = 4);    // 4
```

---

## Erasing elements

General form of erase operation is

```cpp
it = container.erase(it);
it = container.erase(it1, it2);
```

In both the cases, the returned iterator is one after the last erased.

```cpp
// Delete all odd numbers from a vector
vector<int> vec = {1,2,3,4,5,6,7,8,9};
for (auto it = vec.begin(); it != vec.end();) {
  if (*it % 2 != 0) {
    it = vec.erase(it);
  } else {
    ++it;
  }
}
```

## forward_list

> node1 --> node2

Add new element

> node3 --> node1 --> node2

If we had an erase method for `forward_list`, which deletes the current element
and returns the next element, the previous element's pointer will have to be
updated to point to new next element. This is not possible. Hence,
`forward_list` does not support methods like `erase`, `insert` or `emplace`.
Instead it supports `erase_after`, `emplace_after` and `insert_after` which
inserts or deletes after the current iterator.

This creates a bit of a problem if we want to delete the 1st element in
forward_list. Hence, we have a `before_begin` iterator especially available for
`forward_list`.

```cpp
// Delete odd numbers from forward_list
forward_list<int> fw = {1,2,3,4,5,6,7,8,9};
auto it_prev = fw.before_begin(),
      it_cur = fw.begin();
while (it_cur != fw.end()) {
  if (*it_cur%2 == 1) {
    it_cur = fw.erase_after(it_prev);
  } else {
    it_prev = it_cur;
    ++it_cur;
  }
}
```

`erase_after` returns the iterator to the next element after what was erased.

### Iterator invalidation

Pay special attention to how and when iterators get invalidated. Its common
sense most of the times, but can be very subtle at some places.\
For example, `push_back` on a vector can potentially invalidate iterators. If
size of vector is equal to capacity, adding any new elements will trigger
storage reallocation, which can invalidate existing iterators to the vector.

---

### Capacity management

`vector` and `string` allow us to manage memory allocation to a small extent by
exposing following members. Though they should not be required in most cases.

1. `c.capacity()` - returns the number of elements container can hold.
   Always >= size
1. `c.shrink_to_fit()` - reduce the capacity to size. Implementation is free to
   ignore this.
1. `c.reserve(n)` - Reserve space for atleast n elements. Has no effect if n is
   smaller than or equal to current capacity.
1. `c.resize(n)` - Has no effect on capacity, only on size.

---

### Ways to create strings

```cpp
string s1;
string s2 = s1;
string s3 = "Hello";
string s4(10, 'a');   // a repeated 10 times
string s5(s4);
string s6(s5.begin(), s5.end());
string s7 = {'a','b'};
string s8("abc",2);   // first 2 characters, valid for char arrays
string s9("abc");
string s10(s3, 2);    // "llo", characters from 2 to end, contrast against s8
string s11(s3, 2, 3); // "llo", start from 2, 3 characters
```

substr operation

```cpp
string s1 = "hello world";
string s2 = s1.substr(0,5);   // "hello", 5 characters from index 0
string s3 = s1.substr(6);     // "world", all characters from index 6 to end.
```

Throws `out_of_range` exception if string is smaller than attempted substring.

```cpp
str1.assign(str2);      // replace contents of str1 with str2
char* cp = "hello world";
str1.assign(cp, 5);     // str = "hello"
str1.insert(str1.size(), cp+6); // str = "helloworld"
```

These assign and insert variants are available only for strings.

### More string operations

```cpp
string s = "abc";
s.append("def");
// equivalent to
string s1 = "abc"
s.insert(s.size(), "def");
string s = "hello world";
s.replace(6, 5, "there");   // s == "hello there"
// equivalent to
string s1 = "hello world";
s1.erase(6,5);
s1.insert(6, "there");
```

There are other overloaded versions of these operators which can take iterator
ranges instead of position and length.\
Best not to remember all, just know that these exist, and lookup when required.

### string search functions

String search functions allow following variants

1. Find 1st char/string which matches - `find`
1. Find last char/string which matches - `rfind`
1. Find 1st char which matches any character from argument string -
   `find_first_of`
1. `find_last_of` for reverse search of above.
1. `find_first_not_of` - inverts above
1. `find_last_not_of`

These functions return the index if required character is found, return
`string::npos` otherwise, which can be checked against for invalid index.\
There also are overloads for each of above which allow to search only for a
substring (starting from pos, upto n characters).

### string compare

`str.compare()` - Has 6 variants. In most general case, allows to compare part
of a string to a part of another string. Also has variants which allow to
compare against null terminated strings.

---

## container adapters

Container adapters are higher level data structures built on top of contaisers,
e.g., `stack`, `queue`, `priority_queue`\
`stack` requires the operations like `push_back`, `pop_back` and `back` to
support stack API (push, pop and top). Hence only those containers which support
these can be valid as underlying containers for a stack. This includes `list`,
`vector`, `deque` etc.\
Any user defined type may also work.\
`forward_list` and `array` won't work as they don't support these methods.

```cpp
stack<int> s1;  // backed by deque by default
stack<int, vector<int>> s2; // backed by a vector of ints
stack<int, forward_list<int>> s3; // not allowed, but no error yet
s3.push(10);    // ERROR
```

`queue` requires pushing from front and popping from back. So, `vector` won't
work for `queue`, but `list` or `deque` will.\
`priority_queue` requires random access (think heap built on array like data
structure), access to front and back, and ability to push at the back (required
for adding element to priority queue before swim up operation) So, it will work
with `vector` or `deque` as backing store, but not `list`.

---

## Generic Algorithms

`begin` and `end` are global functions defined in `std` namespace which can
return first and one past last iterator for the container passes in as argument.\
These also works for C-style arrays, as long as its definition is visible. If
array is passed as an argument, its size related information is lost, and
`begin` or `end` cant be called on it.

C++ compiler can automatically infer types for template arguments, when a
function is called.

```cpp
template<typename T, typename U>
size_t count(const T& begin_it, const T& end_it, U val ) {
  // ...
}

// calling code
size_t cnt = count(vec.begin(), vec.end(), 10); // OK
// more verbose, with explicit template arguments.
size_t cnt2 = count<vector<int>::iterator, int>(vec.begin(), vec.end(), 10);
```

## How iterators are passed to generic algorithms

First 2 arguments are always the iterators to container on which the algorithm
is to be applied. Some algorithms need 2 or more containers. In this case, the
2nd container can be specified in 2 ways

1. Using `begin` and `end` iterator.
1. Only specify `begin` iterator. In this case, the implicit assumption is that
   2nd container is atleast as big as the first one. It is a serious error
   otherwise.

`fill` and `fill_n` functions can be used to assign all elements a particular
value. Note that it cannot add elements to container.

`back_inserter` can be used to insert elements at the back of a container.
It overloads assignment operator to insert element at the back.\
Type - `std::back_inserter_iterator<std::vector<int>>`

```cpp
vector<int> vec {1,2,3};
auto back_it = back_inserter(vec);
*back_it = 4;   // (1)
*back_it = 5;
back_it = 6;    // (2)
print(vec);     // 1 2 3 4 5 6
cout << *back_it;   // ERROR
```

Dereferencing the iterator returns itself, so assignment overload works as
expected. But this means dereferencing cannot be used to access any element.\
This also means that (1) and (2) are equally valid.

```cpp
vector<int> vec;
fill_n(begin(vec), 10, 0);  // ERROR: run time - vec is empty
fill_n(back_inserter(vec), 10, 0);  // inserts 10 elements at back of 0 value
```

`copy` operation works as expected. It returns one past the last element to be
copied at destination.

```cpp
vector<int> v1 {1,2,3};
vector<int> v2;
auto ret = copy(v1.begin(), v1.end(), back_inserter(v2));
```

### replace

```cpp
// Replace all occurences of 0 with 42
replace(begin(vec), end(vec), 0, 42);
// Copy all elements from vec to v2, replacing 0 with 42
replace_copy(cbegin(vec), cend(vec), back_inserter(v2), 0, 42);
Sorting and removing duplicates (popular idiom)
// Assume a vector vec
sort(begin(vec), end(vec));     // default sort
auto end_unique = unique(begin(vec), end(vec));
vec.erase(end_unique, end(vec));
```

`std::unique` rearranges a sorted list so that elements starting from the
returned iterator, until the end of container are duplicates. They are
eliminated in the above code by erase member method.

---

## Lambda functions

lambda functions cannot take default arguments.

Capture list in lambda is used only for non-static variables defined in the
surrounding function. Lambda function can use local statics and names which are
defined outside the surrounding function directly.\
Special mention, lambda implicitly captures `this` (TODO - expand later)

Anything captured by value is treated as const by lambda. If you really need to
change the captured (by value) values use `mutable` keyword.

```cpp
int i = 10;
auto lambda1 = [=]() { ++i ; cout << i; };  // ERROR
auto lambda2 = [=]() mutable { ++i ; cout << i; };   // OK
```

If lambda body contains anything more that a single return statement, its return
type must be explicitly specified. Otherwise, it is assumed to return `void`.
For example following code transforms input vector of integers to contain
absolute values of all elements

```cpp
vector<int> v = {-1,2,-3};

// OK
std::transform(v.begin(), v.end(), v.begin(), [](int i) {
  return i >= 0 ? i : -i;
});

// FAIL, as lambda is inferred to return void, which is not compatible with
// signature of std::transform
std::transform(v.begin(), v.end(), v.begin(), [](int i) {
  if (i >= 0)
    return i;
  else return -i;
});

// OK - Return type explicitly specified
std::transform(v.begin(), v.end(), v.begin(), [](int i) -> int {
  if (i >= 0)
    return i;
  else return -i;
});
```

---

### Stream iterators

Code which uses stream iterators to transform a list of integers to their squares

```cpp
ifstream ifs("input_integers.txt");
ofstream ofs("output_integers.txt");

istream_iterator<int> ist(ifs), eof;
ostream_iterator<int> ost(ofs, " ");

std::transform(ist, eof, ost, [](int i) {
  return i*i;
});
```

`istream_iterator` takes a single argument, which is `istream` from which to
read. The version without argument represents *end of file* iterator.\
`ostream_iterator` takes 1 or 2 arguments. 2 argument version adds the 2nd
argument (string) after every element of the templated type object written to
underlying `ostream`.

stream iterators do not support reverse iteration.

---

## Reverse iterator

Suppose you want to print the last word from a string containing comma separated
words.

```cpp
string str = "one,two,three";
// find last comma
string::pos rcomma = find(str.crbegin(), str.crend(), ',');
```

`rcomma` is a reverse iterator, ++ on this moves backwards. To find last word,
we need a corresponding forward iterator.

```cpp
string last(rcomma.base(), str.cend());
```

`base()` provides the underlying iterator, which is one after the reverse
iterator. Hence, `rend()`'s base is `begin()`, and `rbegin()`'s base is `end()`

---

## Associative containers

There are 2 ways to enable associative containers to store custom objects.

1. Overload < operator for the object type. \
   The default implementation uses a comparator of type `std::less<T>` to
   compare keys, which internally uses overloaded < operator.
1. Define a custom comparator function or functor and pass it to ctor. Its type
   also needs to be specified as template parameter

```cpp
class MyType;
bool compareMyType(const MyType& a, const MyType& b);

set<MyType, bool(*)(const MyType& a, const MyType& b)> s1(compareMyType);

// Here lambda is convertibe to function pointer since its capture list
// is empty.
auto lambda = [](const MyType& a, const MyType& b) { ... };
set<MyType, bool(*)(const MyType& a, const MyType& b)> s2(lambda);

bool operator<(const MyType& a, const MyType& b) { ... };
// same as not specifying comparator, as long as operator< is defined
set<MyType, std::less<MyType>> s3(std::less<MyType>());
```

### Type aliases defined by associative containers

1. `key_type` - type of key
1. `mapped_type` - type of mapped value, only valid for `maps`
1. `value_type` - same as `key_type` for `sets`, `pair<key_type, mapped_type>`
   for `maps`. This is defined for sequential containers as well.

These can be used to get the type of underlying container. Its generally useful
in template metaprogramming.

```cpp
template<typename C>
typename C::value_type sum_(const C& cont)
{
    typename C::value_type total = 0;     //  (1)
    for (const auto& e : cont)
        total += e;
    return total;
}
```

This type of code wouldn't be possible without `value_type` alias.

#### Why is `typename` required in the line (1)

Without `typename` there is no way to know if `C::value_type` refers to a type
or a value, since `C` is not bound to a concrete type before template
initialization. In this particular case, it may seem that this can be inferred
from context, but that's not always the case. For example

```cpp
C::x * p;
```

If `x` is a type, `p` is a pointer, otherwise, we are multiplying 2 values.

Iterators for `map` yield `pair`, not keys. Keys are always constant in
associative containers.
Hence, even though `set` defines both const and non const iterators, both
behave the same. Its a compiler error to change the value of an element in set
using non const iterators.

---

### Map insertion

* `insert` operation on map takes a `pair` as argument.
* `insert` can take a input iterators of pairs. If multiple pairs have same key
value, only the first element from input list will be inserted
* `insert` operation can also take a position iterator as *hint* which can
  potentially speed up the insertion operation. The position should specify an
  iterator which is known to preceed where the element will be inserted.
* If the key corresponding to the value inserted is already in the `map`,
  `insert` operation does nothing. For multimaps, it inserts the duplicate
  entry.

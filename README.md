*This tutorial was originally written for the beginners at the Lab of
cosmology, stellar dynamics and computational astrophysics at
[fai.kz](https://fai.kz). It is intended to establish standard
terminology and to advance understanding of some aspects of `C++` (and
`C`). It is not an introductory course: you should already be familiar
with basic syntax and, better yet, have some experience with writing
and compiling small programs. Most of the material is relevant for
both `C` and `C++`, so I often write `C/C++`, making the distinction
where necessary.*

* [The build process in C/C++](#the-build-process-in-cc)
* [Memory layout of a running process](#memory-layout-of-a-running-process)
* [Meet the denizens of C++](#meet-the-denizens-of-c)
   * [1. Types](#1-types)
   * [2. Objects](#2-objects)
      * [Storage duration](#storage-duration)
      * [Pointers: Twinkle, twinkle, little star...](#pointers-twinkle-twinkle-little-star)
         * [Dereferencing: ...How I wonder what you are!](#dereferencing-how-i-wonder-what-you-are)
         * [Stars turn into arrows](#stars-turn-into-arrows)
         * [Pointers and storage duration](#pointers-and-storage-duration)
         * [Pointers and const](#pointers-and-const)
         * [Pointers and arrays](#pointers-and-arrays)
         * [Dynamic arrays](#dynamic-arrays)
   * [3. Functions](#3-functions)
      * [Parameters vs arguments](#parameters-vs-arguments)
   * [4. References](#4-references)
      * [Mixing stars and ampersands](#mixing-stars-and-ampersands)
      * [Pass-by-reference](#pass-by-reference)
   * [5. Values](#5-values)
      * [Expressions vs Statements](#expressions-vs-statements)
      * [Temporary objects](#temporary-objects)
      * [lvalues and rvalues](#lvalues-and-rvalues)
* [Declaration vs Definition](#declaration-vs-definition)
   * [External and internal linkage](#external-and-internal-linkage)
   * [Object declarations](#object-declarations)
   * [Type declarations](#type-declarations)
   * [Include guards](#include-guards)
   * [Headers and libraries](#headers-and-libraries)


# The build process in C/C++ 

More often than not, a `C/C++` project consists of many source files.
No matter if you have one or more source files, building a program
involves these three steps:

1) **Preprocessing**: the _preprocessor_ takes each source file and
handles the *preprocessor directives* - those lines that start with
`#` (e.g., it substitutes `#include`s with the text from included
files, etc). The output of this step is called a _translation unit_ -
"pure" `C/C++` file without preprocessor directives.

2) **Compilation**: the _compiler_ takes the preprocessor's output
(translation units), and compiles them into binary _object_ files.

3) **Linking**: the _linker_ takes the object file(s) prepared by the
compiler and produces a single final product: an _executable_ or
a _library_.

Source files are compiled separately into object files, and this is
good for large projects: if you make corrections in only one source
file, you need to re-compile only this one file, not all of them
(linker will still need all object files to re-link, but linking is
much faster then compiling).

**Notes**:

- *Preprocessor directives* are commands for the preprocessor for
  including one file into another (`#include`), making choices
  (`#ifdef`, `#ifndef`, etc.), defining "variables" (`#define`), etc.
  In fact, it is an individual programming language by itself,
  which is used to manipulate code before passing it to the `C/C++`
  compiler. You will also see the word *macro*, but it is not a
  synonym for directive; it is only one type of directive: `#define`.
  Macros might be defined as parameters to `g++` from the command line
  (see `-D` flag).

- From the point of view of the compiler, header files (`*.hpp` or
  `*.h`) do not exist. They are included into source files (`*.cpp`)
  by the preprocessor (so you even never indicate header files when
  compiling with `g++`, only source files). Hence, if you modified a
  header which is included into multiple `cpp`-files, you will have to
  re-compile all these files, as if they all have been modified. 

- Although `g++` is usually called a compiler, by default it performs
  _all_ steps above. To stop before linking, use `-c` flag: `g++ -c
  file.cpp`. This will only produce `file.o` object file but no
  executable. You can also stop after the first step, using `-E`. This
  will produce a translation unit: `g++ -E file.cpp > file.pure.cpp`.

- The described scheme is simplified. The second step is itself split:
  first, the translation unit is translated into `assembly`
  instructions, which are then compiled by `assembler` into object
  files. To see the assembler version of your code use `g++ -S` (this
  will produce `.s` file for each `.cpp` file).

- If you build an _executable_, and none of the object files contains
  `main` function, the linker will report an error. A _library_
  differs from _executable_ in that it has no entry point (no `main`
  function). Roughly, it is a file that contains, in the form of
  machine code, functions, type definitions, etc., which might be used
  from other executables/libraries. Libraries are also created with
  `g++` using special flags. 

- GNU's `g++` is not the only compiler, it is just _de facto_ standard
  in the Linux world. A more recent (and probably more efficient)
  compiler is `clang` (compatible with most of the `g++` options).
  Other compilers include `MSVC` (Microsoft Visual C++), `Intel C++`,
  `Borland C++`, etc. They differ in their target platforms, licenses,
  and in how close they follow the `C++` standard.

- In Russian, linker is called *компоновщик*, and the entire build
  process is called *сборка*. Сборка: исходный код -> препроцессинг ->
  компиляция -> компоновка.


# Memory layout of a running process 

Roughly, any source code is a mix of two things: *instructions* and
*data*. E.g., variables you define are _data_, and statements like
`for`-loops and function calls are _instructions_. When the source
code is compiled into binary file, instructions and data go into
different segments in this file, which are called *code* and *data*
segments (the code segment is also called _text_ segment). 

The compiled bodies of all functions will be stored in the code
segment. No matter how many times you call a function in your program,
it will have _only one_ binary representation in the code segment, but
it might process different data in each call.

When you run a program, the operating system loads the executable file
into memory (think of computer's memory as a vast one-dimensional
array of bytes), and spawns a new process. The code segment becomes
read-only, and is pulled through the CPU like a chain,
instruction-by-instruction. The data segment just sits in memory, but
the values of its variables might change during the program (if
instructions from the code segment modify them).

Apart from the code and data segments, *two more segments* are created
in memory for an executable process: **stack** and **heap**. The point
is that the original data segment is not for storing just any data. It
stores only global variables (and constants) of your program which
might be evaluated from the source code during the compilation. But it
does not store, e.g., local variables of functions, since, in most 
cases, their values might be determined only when the program already
runs (e.g., by reading data from a file). Obviously, such variables
cannot be stored in the data segment in the original binary file.

Since code and data segments are loaded from the binary file and then
never change their sizes, they are called *static* segments. Stack and
heap are created dynamically when the process is started, and change
their contents during the program, so they are called *dynamic*
segments. But why *two* additional segments? Because stack and heap
serve distinct purposes and have different properties.

Stack is a general name for an abstract data structure where adding
and removing elements is possible only at one end (usually called the
"top"), just like with the stack of dishes ("last in, first out";
adding is called *pushing*, and removing is *popping*). The stack
memory segment follows the same principle, tracing the flow of the
program. Before a function is called, its arguments are pushed onto
the stack, then the address of the instruction following the function
is pushed (so that the process knows where to continue when the
function finishes), and then a block for function's local variables
and temporary objects is pushed. The entire pushed structure is called
a stack *frame*. If the function calls another function, a new frame
is pushed, and so on, and then these frames are popped in reverse
order (this happens also when a function calls itself - a new frame is
pushed onto the stack, so variables are not mixed up for distinct
recursive calls of the same function). When a function returns, the
stack looks the same as before it was called. Since `main` is a
function like any other, you may guess that the stack is used from the
very beginning of the program till its end.

The heap, on the other hand, has no constraints on its structure. You
may think of it as a "free store". The stack is usually just a few
megabytes in size, and if you call hierarchically too many functions
or use too many local variables, you will run into *stack overflow*
(an easy way to achieve that is to call a function recursively without
control: with each call, a new stack frame is pushed onto the top,
until the stack is full). But the heap is limited only by the total
amount of free RAM on your computer. It is of course also possible to
run out of heap, but in general, such situations might be checked and
handled gracefully by the programmer, whereas running out of stack is
unrecoverable.

Another difference is that allocating storage on the stack is faster
than on the heap. Local variables in `C/C++` are created on the stack
by default, and you don't need any special functions for that (whereas
to allocate storage on the heap, you must use a special `malloc`
function or, in `C++`, the `new` operator).

> ☢️ The described scheme is not specific to `C/C++` and Linux. It is
> similar for many platforms and is independent of the programming
> language. E.g., even though `Python` scripts are not compiled into
> executables, the `Python` interpreter *itself* is a binary
> executable (the standard one is written in `C`), so its memory
> layout will be as described above, though it will be hidden from you
> as a `Python` programmer (secretely, *all* user variables defined in
> `Python` scripts are allocated in the heap; the stack is used by
> `Python` for its internal workings).

> **Note**: In fact, there is one more segment created for a process when it
> starts: the BSS segment. It's very much like data segment, but is
> used for variables which are not initialized in the code. As a first
> approximation, you may think that it's just part of the data
> segment.

> **Note**: Compared to `C`, the distinction between *data* and *instructions*
> in `C++` is "fuzzier" because `structs` and `classes` might have
> member variables as well as member functions (*methods*). In other
> words, `C++` objects might consist of both *data* and *instructions*
> (in `C` there are no `classes`, and `structs` cannot have
> functions). So where will be those objects allocated? Well, member
> variables will be stored where you create that `class` instance
> (e.g., on the heap), but methods will be stored in the code segment
> anyway. If you create many instances of the same `class`, new
> (non-static) member variables will be allocated for each instance,
> but each method will have *only one* binary representation in the
> code segment (the instructions within member functions are the same
> for all instances of the `class`, but they might be called with
> different data).

> **Note**: In multi-threaded programs, the code, data and heap segments are
> shared among threads, but each thread has its own stack (access to
> shared variables from different threads is a tricky business which
> requires using mutexes to prevent race conditions).


# Meet the denizens of C++ 

The `C/C++` language operates with distinct *entities*: types,
objects, functions, values, references, templates, namespaces, and
others (*NB*: the last three are not in `C`; btw, preprocessor directives
*are not* `C/C++` entities). In these tutorial we will be concerned with
only five of them: *types*, *objects*, *functions*, *references*, and
*values*.


## 1. Types 

The same sequence of bytes can be interpreted in many ways. Of course,
if originally it encoded some meaningful text, you will probably not
get a meaningful picture if you interpret it as an image, and vice
versa. Nonetheless, nothing prevents you from doing that.

⚠️  **Type is an *intended* interpretation of a particular package of
bytes. It determines which operations do and which don't make sense if
applied to those bytes.**

In `C/C++` you indicate types of variables when you declare them.
Compiler uses this information and performs type-checks to ensure that
variables are used in the source code correctly according to their
types (e.g., no division of a number by a string, etc.).

In languages like `Python` you don't indicate types for variables.
This does not mean that there are no types in these languages. This
means that types are inferred by the interpreter from the context. In
`C++` (starting with `c++11`) you also can ask the compiler to
automatically infer types from the initialization values using the
`auto` keyword.

> **Note**: For example, in `auto x = 3.14;` the `x` is automatically deduced by
> the compiler to be `double`. Why not `float`? Because the expression
> `3.14` is a literal `double`; to indicate a literal `float`, write
> `3.14f`.

Still, the key difference is that in `C/C++` types are inferred (if
needed) and checked at *compile time*, but in `Python` types are
inferred and checked at *run time*. This is another reason why `C/C++`
is faster: no type-checking at runtime, as it is already done by the
compiler. Languages like `C/C++` in which types are checked at compile
time are called statically typed, and languages like `Python` are
called dynamically typed.

> ☢️ There are several ways in `C++` to "legally" misinterpret bytes
> (treating them as if they had different types), e.g., by using
> `unions` or `reinterpret_cast`. Normally, you shouldn't do that,
> unless you know what you are doing.

Apart from fixing the interpretation of variables, types also fix
their sizes. E.g., any `int` variable occupies 4 bytes (32 bits) of
memory on most modern machines, which means that it can hold a value
between `-2147483648` and `2147483647`. Note, however, that `C++`
standard guarantees only lower limits - e.g. `int` is guaranteed to be
at least 16 bits, but compilers are allowed to use more. In a truly
cross-platform program which uses very large numbers, you should check
the limits (see `std::numeric_limits`).

The types in `C++` are classified into *fundamental* (`int`, `char`,
`float`, etc.) and *compound*. The latter are defined in terms of
fundamental or other compound types,  and include pointer, reference,
array, function, and class types. To create your own custom type you
define a `class` or a `struct`, which are almost synonyms in `C++`.
The fundamental type `void` is special in that you cannot declare
`void` variables (but you can declare pointers to `void` and functions
which return `void`).

In `C/C++` there are also type *modifiers* (`*` and `&`, which define
pointers and references - we'll meet them below) and type
*qualifiers*, with `const` being the most common. Note that `const int
x = 1;` and `int const x = 1;` are the same.


## 2. Objects 

Now we are in a position to define an *object*:

⚠️  **An object is a region of storage in data, stack or heap segment,
which represents a value of some type.**

> **Note**: In `C++` the term "object" is also used in the context of
> object-oriented programming (OOP), where it refers to an instance of
> a `class`. The above definition is more general: it includes
> fundamental types such as `int`, and is not related to OOP. In this
> tutorial we will (almost) not consider `classes`, so there will be no
> confusion. For a more technical explanation of the notion of an
> object, see [here](https://en.cppreference.com/w/cpp/language/object).

With the definition above, you may think that an object is just
another word for a *variable*. In fact, you should think of variables
as *names* for *objects*. As we'll see, not all objects have names;
besides, references are variables, but not objects.

Each object is characterized by its size (as dictated by its type)
and address (roughly - the number of bytes from the "start" of the
memory to the first byte of that object). The size is found with the
`sizeof` operator, and the address is found with the `&` operator:
```cpp
 int x = 10; // object of type 'int', named 'x'
 cout << sizeof x << endl; // get size of x
 cout << & x << endl;      // get address of x
```

### Storage duration

Apart from size, address and type, objects in `C/C++` are also
characterized by their *storage duration*:

- Objects with **automatic** storage duration live on the *stack*.
  They live from the point where they are defined to the end of the
  block (e.g., the function body), where they are automatically
  destroyed. All local variables within functions (including `main`)
  refer to automatic objects, unless you declare them with `static`,
  `extern` or `thread_local` specifier (more on these later).

- Objects with **dynamic** storage duration live on the *heap*. They
  live from the point where they are manually created with the `new`
  operator (or function `malloc` in `C`) to the point where they are
  manually deleted with the `delete` operator (function `free` in
  `C`); these two points might be in different functions or even
  different source files, so dynamic objects can live longer than
  functions in which they were created. Dynamic objects *do not have
  names and are accessed only through pointers*.

- Objects with **static** storage duration live in the *data*
  segment. They are allocated when the program starts, and destroyed
  when the program ends. In fact, if they are initialized with a
  constant, they are loaded into memory from the executable file,
  before the `main` starts. All objects declared in global scope
  (with or without `static` specifier) and all in-function objects
  declared with `static` specifier have static storage duration.

- Starting with `c++11`, we also have **thread** storage duration. These
  objects are declared with `thread_local` specifier. You may think
  of them as static objects which are copied for each thread
  (normally, objects in data segment, as well as in heap, are shared
  between threads). We will not consider threads here, so you may
  forget about this one for now.

Few examples:
```cpp
 // global scope
 int x = 10; // static object
 static int y = 11; // static object
 // these two have the same storage duration,
 // but they are different for the linker, as we will see below
 
 void foo()
 {
   // function scope
   int z = 12; // automatic, lives to the end of function
   static int q = 13; // static object, lives during the entire program

   if (true)
   {
     int s = 14; // automatic, lives to the end of block
   }
   // "s" is already dead here

   int *p = new int(15);
   // a dynamic object is created on the heap,
   // and its address is assigned to the pointer p;
   // The pointer 'p' itself is automatic in this case - 
   // it is created on the stack! (more on pointers later)
 }
```
> **Note**: The `new` operator does two things: it allocates a region of memory
> (with the size dictated by the indicated type), and then
> constructs/initializes the object in that region. By default, it
> creates a new object in the heap, and returns its address (which
> might be assigned to a pointer). It is possible to ask `new` to
> construct an object at a specified address (this is called
> "placement-`new`"). In fact, the specified address might be even on
> the stack:
> ```cpp
>  int x;
>  new (&x) int(10);
>  // I ignore the result of the operator, 
>  // since I already know the address: &x
> ```
> Here, it is just a fancy way to write `x = 10` ;) But there are
> situations where placement-new is useful.

As for in-function static objects - think of them as global
objects living outside the function in which they are declared, but
visible only from that function. Example:
```cpp
 void foo()
 {
     static int x = 10;
     x++;
     cout << x << endl;
 }

 int main()
 {
     foo();
     foo();
     foo();
 }
```
The output:
```bash
 11
 12
 13
```
Here is what happens:

- a region in memory for the object `x` is allocated in the data
  segment when the program loads (even before the `main` starts);
- only the `foo` function is "informed" about the name `x`;
- the object `x` is initialized with the value `10` when the function
  `foo` is called for the first time;
- then `x` keeps values between the calls to `foo`. When `foo` is
  called again, you may pretend that the line `static int x = 10;`
  *does not even exist*. (In fact, this is true even if you write
  `static int x = y;`, where `y` is, say, the function's parameter;
  this line will be ignored on all calls except the first one.)

> ☢️ The `auto` keyword mentioned earlier has its origin in
> automatic variables in `C`, where you can use it to declare local
> variables like `auto double x = 3.14;`. However, since local variables
> are automatic by default, this keyword is rarely used in `C`, and was
> introduced for compatibility with the `B` language, a predecessor of
> `C`. `C++` inherited this "useless" keyword from `C`, but in `c++11`
> standard it was given a completely new meaning, which is automatic
> deduction of types from the initialization values: `auto x = 3.14;`.


### Pointers: Twinkle, twinkle, little star...

The star symbol `*` in `C/C++` might have three different meanings,
depending on the context:

1) between two expressions of arithmetic types: ***multiplication***
2) after a type name: ***declaring a pointer***
3) in front of the name of an existing pointer: ***dereferencing a pointer***

The first case should be familiar. So let's deal with pointers.

⚠️  **A pointer is a type of an object whose value is the address of
another object.** For brevity, objects of pointer types are also called
pointers.

Although locations in memory (addresses) are defined by integer
numbers (roughly: the number of bytes from the "beginning" of memory),
you cannot assign an integer to a pointer. E.g., compiler _will not_ accept
this:
```cpp
 int *p = 10;
```
This is because a valid value for a pointer is an address *of the first
byte of an existing object*, not just any integer. To assign the address of an
object to a pointer, use the already familiar "address-of" operator `&`:
```cpp
 int x = 10;
 int *p = &x; // p stores the address of the object x
```
or create an object on the heap with the `new` operator, which
returns the address of the created object:
```cpp
 int *p = new int(10);
 // p stores the address of an unnamed object in the heap
```
> **Note**: However, assigning a *literal* zero to a pointer is valid:
> ```cpp
> int *p = 0; 
> ```
> This does not make a pointer pointing to the "beginning" of memory.
> Rather, it defines a **null** pointer which does not point to any
> object. However, in `C++` there are reasons to prefer `nullptr` for that.

> ❓ In the following code zeros are assigned to the pointer, but it will not work. Why?
> ```cpp
> int a = 0;
> int *p1 = a;       // error
> int *p2 = 1 - 1;   // error
> ```
> <details>
> <summary>(Click for the answer)</summary>
>
> These are not *literal* zeros.
> </details>

> **Note**: Whitespaces in the declaration of a pointer have no effect.
> These are all the same:
> ```cpp
> int *p;
> int* p;
> int * p;
> int*p;
> int    *  p;
> ```
> Don't be fooled by the 1st variant: it makes an illusion as if `*`
> is part of the name. Instead, it is part of the type, not of the
> name (technically, `*` in this context is called *type modifier*).
> The _name_ of the object is `p`, and its type is a _"pointer to
> `int`"_. To make things even more confusing, the standard allows
> this:
> ```cpp
> int x, *y, z;
> ```
> Here, in one line, you define two `int` objects and one pointer
> to an `int` object. The `*` here is decoupled from the type name,
> making even a stronger illusion that `*` is part of the object name.
> Also, don't be fooled by this:
> ```cpp
> int* x, y, z;
> ```
> In this declaration, only `x` is a pointer! 

Since a pointer is an object, it has its own address:
```cpp
 int *p = new int(10);
 cout << &p << endl; // prints address of p itself
```
So, a pointer can hold address of another pointer ("chain" of
pointers):
```cpp
 int x = 10;
 int * px = &x;       // px is a pointer to int
 int ** ppx = &px;    // ppx is a pointer to int*
 int *** pppx = &ppx; // pppx is a pointer to int**
 // ... and so on
```
You may regroup the stars, to make it easier to understand:
```cpp
 int x = 10;
 int *px = &x;       // px is a pointer to int
 int* *ppx = &px;    // ppx is a pointer to int*
 int** *pppx = &ppx; // pppx is a pointer to int**
```
Also, like all other objects, pointers have size. Since pointers
hold addresses, and addresses of objects do not depend on their
types, the size of the pointer is independent of its exact type.
On a typical modern machine, the size of a pointer is 8 bytes, no
matter if it is `int*`, `char*`, `int***`, or `MySuperBigStruct*`.
This size is enough to hold the address of any byte even if you have
several exabytes of RAM.

> ☢️ Now you should not be surprised with this:
> ```cpp
> std::vector<int> v;
> cout << sizeof v << endl; // prints 24
> for (int i = 0; i < 100; i++) v.push_back(i);
> cout << sizeof v << endl; // prints 24 again!
> ```
> Objects of the type `std::vector` have data members which
> contribute to its total size of 24 bytes. One of these members is a
> pointer to the first element of the actual array, and the size of
> that pointer does not depend neither on the number of elements in
> the array, nor on their type. To get the number of elements in the
> vector, use its `.size()` method.

The same pointer can point to different objects during its lifetime (but
objects must be of the same type, of course):
```cpp
 int x = 10;
 int *p = &x; // p points to x
 int y = 20;
 p = &y; // now p points to y
 p = new int(30); // and now p points to an unnamed object on the heap
```
Note that in the last lines, there is no `*` in front of `p`: the
pointer is defined in the second line, and now we simply use its name.


#### Dereferencing: ...How I wonder what you are! 

Good or bad, the same symbol `*` is used to *dereference* a pointer,
i.e. to access (both for reading and for writing) the value of the
object to which the pointer points:
```cpp
 int *p = new int(10); // create dynamic int object
 *p = 11;              // assign 11 to that object
 int y = *p;           // assign 11 to y
```
The `*` in the first line comes after the type name, so it _defines_
a pointer. In the 2nd and 3rd lines, the `*` is not after type name;
it is in front of the object name `p`; this object already exists
(it is defined in the 1st line) and it is a pointer; therefore, here
`*` is used to *dereference* the pointer. In the 2nd line dereferencing
is used to write a value, in the 3rd line - to read a value.

Note the difference:
```cpp
 int x = 10;
 int *p = &x;
 cout << *p << endl; // prints the value of x (dereferencing)
 cout << p << endl;  // prints the address of x (value of p itself)
```
Also note that when comparing pointers, you compare addresses, not values:
```cpp
 int x = 10;
 int y = 10;
 int * px = &x;
 int * py = &y;
 assert( *px == *py ); // true: the values of x and y are equal (10 == 10)
 assert( px == py );   // false: addresses of x and y are different
```
> **Note**: `assert` is a useful debugging function (in fact, a
> preprocessor macro). If condition passed to `assert` is `false`, the
> program aborts, indicating the failed condition. When you compile
> your code with `g++ -DNDEBUG`, all `assert`s will be
> excluded by the preprocessor. Thus, you can always switch to debug
> mode and back, without deleting/inserting code. You have to
> `#include <cassert>` to use `assert`.

> ☢️ It's fun to repeat the same in `Python`:
> ```python
> x = 10
> y = 10
> 
> assert id(x) == id(y) # this assert will very likely pass 
> # NB: id() function returns object's address.
>
> y = 11
> assert id(x) != id(y) # this assert will definitely pass
> ```
> If you create two variables with the same value, `Python` will most
> likely create only one object, and make the second variable its
> second name. Only when you assign a different value to any of the variables, it
> will create the second object and bind that variable to it.

Dereferencing an uninitialized or null pointer is undefined behavior
(most likely, segfault):
```cpp
 int *x;
 *x = 1; // boom!
 int *y = nullptr;
 int z = *y; // boom!
```
The best practice is to always initialize pointers, at least with
`nullptr`, and check before dereferencing:
```cpp
 void foo(int *p)
 {
     if (p)
     {
         // safe to dereference
         int x = *p;
     }
     else
     {
         cout << "Null pointer!" << endl;
     }
 }
```

When you have chained pointers, to get the value of the final object
you have to apply as many stars as used in the declaration of the
pointer:
```cpp
 int a = 10;
 int *p = &x;
 int **pp = &p;
 cout << pp << endl;   // value of pp, i.e., the address of p
 cout << *pp << endl;  // value of p, i.e., the address of x
 cout << **pp << endl; // value of x
```

#### Stars turn into arrows

As you may know, if you use an object `x` which is an instance of some
`class` or `struct`, then to get access to its members (say, `y`) you
use the dot operator, like in `x.y`. But when you access your object
`x` via a pointer (say, `p`, which points to `x`), then you cannot
write `*x.y` - that would mean you dereference `y`, not `x`. So you
have to put the dereferenced `x` in parenthesis: `(*x).y`. There is an
alternative syntax for this, which means absolutely the same: `x->y`.
This arrow operator might come *only after* a pointer to `struct`/`class`.

It might happen that a `struct` member is itself a pointer. Here's how
you access the object to which it points:
```cpp
 struct S
 {
     int *y = nullptr;
 };

 int main()
 {
    S s;             // s is object of type S
    S *sp = &s;      // sp is pointer to object of type S

    int z = 10;

    // using struct object directly
    s.y = &z;        // make the pointer point to z
    *s.y = 11;       // change the value of z;

    // using pointer to struct object
    sp->y = &z;      // make the pointer point to z
    (*sp).y = &z;    // same
    *((*sp).y) = 11; // change the value of z;
    *(*sp).y = 11;   // same
    *sp->y = 11;     // same
 }
```

> ❓ What will be the output of this code? Explain why.
> ```cpp
> int x = 10;
> cout << *&x << endl;
> ```
> <details>
> <summary>(Click for the answer)</summary>
>
> It will be `10`. `&x` yields the address of `x`, i.e., a pointer to
> `x`, and putting `*` in front of it dereferences the pointer, i.e.
> yields the value of `x`.
> </details>


#### Pointers and storage duration 

Like all other objects, pointers also have their own storage
duration, which *is independent from the storage duration of the objects
to which they point*. Thus, all combinations are possible: static
pointers to dynamic data, automatic pointers to static data, etc:
```cpp
 // global scope
 int g = 10;                     // static object
 int *ps = nullptr;              // static null pointer

 int main()
 {
    int * pa = &g;               // automatic pointer to static object
    int x = 20;                  // automatic object
    int * xa = &x;               // automatic pointer to automatic object
    ps = &x;                     // static pointer to automatic object       [+]
    int * pd = new int(30);      // automatic pointer to dynamic object
    ps = pd;                     // static pointer to dynamic object
    int ** ppd = new (int*)(&x); // ppd is an automatic pointer              [+] 
                                 // which points to a dynamic pointer, 
                                 // and that dynamic pointer (it has no name)
                                 // points to the automatic object x.
    assert( &x == *ppd );        // true
    assert( x == **ppd );        // true
 }
```
However, cases marked as `[+]` are rarely used, because static and
dynamic pointers live longer than the stack object to which they
point. After function finishes, these pointers will point to garbage.

For the same reason, never return the address of an automatic object from a
function. This is a recipe for disaster:
```cpp
 int* foo()
 {
     int x = 10;
     return &x;
 }

 int main()
 {
     int *p = foo();
     cout << *p << endl; // crash and burn
 }
```
After function `foo` returns, its stack frame is destroyed (together
with the object `x`), so the returned pointer points to a stinking
dead body...

Returning the address of a dynamic object is Ok:
```cpp
 int* foo()
 {
    return new int(10);
 }
```
but you will have to manually `delete` the dynamic object when it is not
needed any more. Returning pointer to static data is also Ok.

> **Note**: The `delete` operator accepts a pointer, and frees the memory
> occupied by the object to which the pointer points (note, btw, it
> will not set the pointer to `nullptr` automatically). The pointer
> passed to delete must point to a dynamic object; if you pass a
> pointer to static or stack object, you will end up with an undefined
> behavior.

It is common for `C/C++` beginners to confuse pointers with dynamic
objects. Note that with this expression:
```cpp
 int *p = new int(10);
```
you create *two* objects, not *one*. The first one (on the right) is
created on the heap and it is unnamed, it is of type `int` and its
value is `10`. Another object (on the left) is automatic (or static,
if the expression is in global scope), its name is `p`, it is of
type "pointer to `int`" and its value is the address of the first
unnamed object.

> **Note**: It's all too easy to forget to delete a dynamic object. E.g.:
> ```cpp
> void foo()
> {
>     int *p = new int(10);  
>     // do something and return...
>     // oops, forgot to delete the dynamic object!
> }
> ```
> Here, the pointer `p` will be destroyed when the function ends, but
> the dynamic object to which it pointed will continue to live. Now, you
> will not be able to delete it anywhere in your program, because its
> address was stored only in `p`. So the region of memory which was
> occupied by that object is lost for your program (imagine if instead
> of one `int` you had an array with millions of elements!). This is
> called a *memory leak*. In `C++` it is always a good idea to abstain
> from working with raw pointers, and, instead, use "smart" pointers
> from the standard library (see `std::unique_ptr`, `std::shared_ptr`,
> and `std::weak_ptr`), which are "wrappers" around raw pointers. They help
> to avoid memory leaks and other surprises that might happen if you
> are not careful enough with raw pointers.

> **Note**: Pointers like `int*` or `double*` not only keep the address of
> an object, but also *know* its type. There is a special pointer
> `void*`, which only knows the address, but not the type. You can
> compare such pointer to other pointers and assign other pointers to
> it, but it makes no sense to dereference it.


#### Pointers and `const`

As with storage duration, the `const`-ness of pointers is independent
from the `const`-ness of objects to which they point. This is a
pointer to `const int`:
```cpp
 const int x = 10;
 const int *p = &x;
 *p = 11; // error: cannot change the dereferenced value, it is const
 p = nullptr; // ok: can reassign the pointer itself
```
And this is a `const` pointer to `int`:
```cpp
 int x = 10;
 int * const p = &x;
 *p = 11;     // ok: can change the dereferenced value
 p = nullptr; // error: cannot change the pointer itself, it is const
```
You can also have a `const` pointer to `const int`:
```cpp
 const int x = 10;
 const int * const p = &x;
 int const * const pp = &x; // same as p
 *p = 11;                   // error
 p = nullptr;               // error
```
You can declare a pointer to `const int` but initialize it with the
address of a non-const `int`:
```cpp
 int x = 10;
 const int *p = &x;
 *p = 11; // error
 x = 11;  // ok
```
In this way, although you can modify `x` directly, you cannot do that
through the pointer `p`. But you cannot initialize a pointer to a
non-const with the address of a `const` object:
```cpp
 const int x = 10;
 int *p = &x; // error
```
If you were allowed to do that, you could use a pointer to change a
`const` object. You can't cheat like that.

> ❓ Knowing that the only fundamental type involved is `int`,
> which exact types must the variables `a`, `b` and `c` have so that the
> following is a valid expression?
> ```cpp
> *a = **b**c;
> ```
> <details>
> <summary>(Click for the answer)</summary>
>
> One of the stars is multiplication, all other stars are dereferencing:
> 
>  ```cpp
>  *a = (**b)*(*c); 
>  ```
>  Therefore, `a` is `int*`, `b` is `int**`, `c` is `int*`.
> </details>

#### Pointers and arrays

You probably know that arrays and pointers are somehow related. Here we
briefly outline this relation.

An array is one of the  compound (non-fundamental) types. Objects of
this type are declared like this:
```cpp
 int a[3] = {1, 2, 3}; // you can use the initializer list
 double b[10];         // or leave the object uninitialized
```
Importantly, the number within square brackets is *a part of the type*.
E.g., in this code
```cpp
 int a[10];
 int b[11];
```
objects `a` and `b` *have different types*. `sizeof a` gives
`40`, and `sizeof b` gives `44` (recall that size of one `int` is
typically 4 bytes). Moreover, since those numbers are part of the
type, and type-checks are performed by the compiler, those numbers
must be known at compile-time, so they cannot be, e.g., returned from
a function (because functions work at run-time, not during compilation).

> ☢️ Some compilers (e.g., `g++`) use extensions and  allow to define
> arrays with numbers which are not known at compile-time (e.g., `int
> a[get_size()];`, where `get_size()` is some function which returns
> an integer). Since this is a deviation from the `C++` standard, you
> should not rely on this.

Since elements of arrays are objects themselves, you can declare a
pointer to an element like this:
```cpp
 int a[3] = {2, 3, 5};
 int *p = &a[0]; // pointer to the 1st element
```
There is one tricky thing with arrays: in most places where you use
them, compiler will automatically substitute a pointer to the 1st
element instead of the array object (in jargon, an "array decays to
pointer"). E.g., this happens with function arguments:
```cpp
 void foo(int a[10])
 {
     cout << sizeof a << endl;
     // will not output 40!
     // will output 8 (size of a pointer)
 }
```
So you can even omit the number, since anyway only the pointer is passed:
```cpp
 void foo(int a[]) { /* same as above */ }
```
Or, equivalently:
```cpp
 void foo(int *a) { /* same */ }
```
So from the point of view of function parameters, pointers and arrays
are the same. But this doesn't mean they are the same in general!

Elements of arrays are objects, pointers are also objects, so an array
can contain pointers:
```cpp
 int *x[10]; // x is an array of ten pointers to int
```
Arrays themselves are also objects, so you can have a pointer to an
array:
```cpp
 int (*x)[10]; // x is a pointer to an array of ten ints
```

#### Dynamic arrays

Arrays considered above are "ordinary" arrays; they are allocated on
the stack (or in the data segment, if defined in global scope). To declare dynamic arrays, use
`new` with square brackets:
```cpp
 int *a = new int [10];
```
This creates an array of ten `int`s on the heap, and returns a pointer
to the first element. Do not forget to delete the entire array with
`delete [] a;` when you don't need it. Note the square brackets. If
you forget them, you delete only the first object: congrats
with the memory leak!

There is a common misconception that dynamic arrays differ from
ordinary arrays in that they are resizable. The truth is - they are
not. They are called dynamic not because you can change their size,
but because they have dynamic storage duration (they are created with
`new[]`) and because you can *set their size at run-time*, not at
compile-time (and you can do that without looking over your shoulder,
because thus spoke the standard). Of course, you can "resize" a
dynamic array, but what it actually means is that you create a
new array with the new size, copy elements from the old array, and then
delete the old array. (In `C` the situation is different, as you may
use `realloc` to resize an array, although, depending on the
situation, it also may perform a copy).

In fact, there are no dynamic arrays at all. In the sense that there
is no such *type* in `C/C++` as a dynamic array. An ordinary array is
a single object of *array type*, and it consists of sub-object
elements. A dynamic array is not a single object. It is a contiguous
sequence of independent objects of the same type.

In general, in `C++` you are strongly encouraged to avoid
using dynamic arrays. Instead, use standard containers like
`std::vector` - they are more convenient, safer, maybe even faster,
and *they are resizable*. You also may use `std::array` instead of
ordinary (stack-based) arrays (e.g., you can't copy one array into
another without using a loop, but you can do that with `std::array`).

Still, understanding of how arrays work is an advantage, at least
because you may find yourself tinkering some day with `C` code. E.g.,
I would suggest to look into this SO post about how two-dimensional
arrays (such as `int[5][3]`) are different from nested pointers
(`int**`): https://stackoverflow.com/a/7307699/5707690 .


## 3. Functions

Although functions also have addresses in the code segment (you can
apply the `&` operator to functions), they are not objects, because
they do not live in *data*, *stack* or *heap* segment, and they do not
contain a *value* (although they can *return* a value, they *contain*
statements). You cannot apply the `sizeof` operator to them.

> ☢️ Contrast this with `Python`, where *everything is an object*,
> including functions and modules. Under the hood, this is because
> _all_ Python types are represented with the same structure
> (`PyObject` in `C` implementation). E.g., numbers and strings are
> specialized `PyObjects`; functions are also `PyObjects` with a
> special method named `__call__` - you can even pass functions as
> arguments to other functions.

The type of a function is represented by the types of all its input
parameters and the type of its `return` value.

### Parameters vs arguments

Do not confuse the terms *parameter* and *argument*. Parameters are
objects defined within a function, and arguments are expressions
passed to the function when it is called. Parameters are just like
ordinary local variables, they are just automatically initialized with
arguments when the function starts. Consider:
```cpp
 void foo(int x, float y)
 {
     // function body...
     x = 10;
     // may assign to parameters, like to any other local variable
 }
```
Schematically, this code might be translated into the following pseudocode,
where `arg1` and `arg2` are the arguments which will be passed to
the function:
```cpp
 // pseudocode! think of <...> as a slot for an argument
 void foo(<arg1>, <arg2>)
 {
     // parameters are defined (secretly from the programmer):
     int x = arg1; // x is parameter, arg1 is argument
     float y = arg2;

     // actual function body starts here...
     x = 10;
     // NOTE: this will change the value of local 'x',
     // but not of the external arg1
 }
```
As clear from this pseudocode, if you change a parameter, that *does
not change* the corresponding argument. This is called
*pass-by-value*, where argument's value is just assigned to parameter. This is the
only regime of function calls in `C`. In `C++`, you also have
*pass-by-reference*, as we will see below.

> ☢️ As a side note, in `C++` you also have something called *function
> objects*, i.e. objects which might be called like functions. These
> are *lambdas* (unnamed function objects) and *functors* (`classes`
> for which the operator `()` is defined). You see, the power of
> modern `C++` is that it can go to abstraction levels almost like in
> `Python`, staying at the same time a low-level and efficient
> language like `C`.


## 4. References 

Like in case of the `*` symbol, the `&` symbol has several meanings,
depending on context:

1) between two expressions of integer types: **bitwise logical operator**
2) in front of the name of an existing object: **the "address-of" operator**
3) after a type name: **declaring a reference**

We shall ignore the first case here. As for the case 2, we have
already met it many times. Let's define what a *reference* is:

⚠️  **A reference is another name (an alias) for an existing object.**

This code defines a reference `y` as a second name for the object named `x`:
```cpp
 int x = 10;
 int &y = x;       // & comes after type name => reference
 assert( x == y ); // true

 x = 20;
 assert( x == y ); // true again

 y = 30;
 assert( x == y ); // still true!
```
As you might see, in some sense, references are similar to pointers,
as both "refer" to another object. But there is a fundamental difference:

⚠️  **References are not objects!**

Remember that pointers are _objects_ themselves: they are allocated in
memory, they have their own address and size. References do not have their
own address and size. Applying `&` and `sizeof` operators to a reference will
return *the address and size of the object to which it refers*:
```cpp
 int x = 10;
 int &r = x;
 assert( &x == &r ); // true
```
> **Note**: the meaning of `&` in the second line is defining a reference.
> The meaning of two `&` in the third line is the `address-of` operator. `C++`
> is one of the most confusing programming languages!

> **Note**: As with declaring pointers, spaces have no effect: `int &r = x;` and
> `int& r = x;` are the same.

Other differences between pointers and references are a consequence of the
fundamental difference mentioned above:

- The value of a pointer is the address of the object it points to.
  The value of a reference is the value of the object it refers to.

- References cannot be reassigned:
  ```cpp
  int x = 10;
  int &r = x; // r is an alias for x
  int y = 20;
  r = y; // r now is an alias for y? No! r is still an alias for x.
         // What happened is that the value of y was assigned to x:
  assert ( x == 20 ); // true

  // You may want to try this:
  &r = y;
  // But this will not be accepted by compiler.
  // In this context & means the address-of, not a reference.
  // (and you cannot change the address of an object)

  // This will also fail:
  int &r = y;
  // because a variable with the same name is already defined
  ```

- Since you cannot reassign a reference, it *must* be initialized when
  you define it:
  ```cpp
  int x = 10;
  int &r;      // Error
  int &rr = x; // Ok
  int *p;      // Ok - pointer might be left uninitialized
  ```
- There are no `null`-references: a reference always serves as a name
  for some object.

- References cannot be chained: you can define a pointer to a pointer,
  but you cannot define a reference to a reference.

  > ☢️ Someday you may encounter a declaration like this (and think that
  > I lied):
  > ```cpp
  > int &&x = ...;
  > ```
  > However, `x` is not a reference to a reference, it is just a
  > reference of another type. Ordinary references that you will
  > encounter most often are called *lvalue references*. Since
  > `c++11` there are also *rvalue references* (declared with `&&`),
  > they are used to extend lifetimes of temporary objects and for
  > move semantics. This is a big separate topic, and we will only touch it a
  > little bit later.

- References are not dereferenced: to access their values, you use
  them like normal names, no special symbols needed.

In general, working with references is considered less error-prone than
with pointers: references cannot be null, and they are always initialized, so you
don't have to check them (unlike the case of pointer dereferencing).

Like in case with pointers, declaring a `const` reference to a
non-const object, you cannot modify that object through that
reference:
```cpp
 int x = 10;
 const int &r = x;
 r = 11; // error
 x = 11; // ok
```

### Mixing stars and ampersands

Since pointers are objects, nothing prevents you from declaring a
reference to a pointer:
```cpp
 int x = 10;       // object of int type
 int *p = &x;      // object of type "pointer to int"
 int &rx = x;      // reference to the object x
 int * &rp = p;    // reference to the object p
 int y = 20;
 rp = &y;          // p now points to y
 assert( p == &y ) // true
```
But since references are not objects, you cannot declare a pointer to
a reference:
```cpp
 int x = 10;
 int & *p = &x; // error
```
But the address of a reference is the same as of the referenced
object, so you can use a reference's address to initialize a pointer:
```cpp
 int x = 10;
 int &r = x;
 int *p = &r; // same as  int *p = &x
```
In declarations like these, the easiest way to understand what's going
on is to start reading from the variable name and go to the left:
```cpp
 int **&r = x;
```
The name is `r`, then we have `&`, so this is a reference. Then we
have `*`, so this is a reference to a pointer. Then we again have `*`,
so this is a reference to a pointer to a pointer. Finally, we have `int`,
so this is a reference to a pointer to a pointer to `int`.

> ❓ Which type must `x` have so that the last example is a
> valid expression?
> <details>
> <summary>(Click for the answer)</summary>
> 
> It should be `int**`.
> </details>

You can declare a reference to an unnamed object on the heap, thus
giving it a name, but it is rarely used:
```cpp
 int &r = * new int(10); // note the *: it dereferences the temporary pointer returned by "new"
 r = 11;
 delete &r; // &r is the address of the object (i.e. a pointer to that object)
 // now you cannot do anything with r:
 // accessing it is undefined behavior,
 // and you cannot reassign it to another object (since it is a reference)
 // This is now a "dangling" reference
```
You also can have a reference to an array:
```cpp
 int a[10];
 int b[11];
 int (&ra)[10] = a; // ra is a reference to an array of ten ints
 int (&rb)[10] = b; // error: types of b and rb don't match

 int *p[10];
 int *(&rp)[10] = p; // rp is a reference to an array of ten pointers to int
```
But you cannot have an array of references, because references are not
objects.


### Pass-by-reference

One of the common uses of references in `C++` is with function
parameters. Recall the pseudocode for the function definition. Now, if
you define a function like this (note the `&`):
```cpp
 void foo(int &x)
 {
     // function body
     x = 10;
 }
```
its equivalent pseudocode might look like this:
```cpp
 // pseudocode! think of <...> as a slot for an argument
 void foo(<arg>)
 {
     int &x = arg; // x is an alias for the object 'arg'
     // function body
     x = 10; // boom! the external object 'arg' changed
 }
```
As you see, in this case the situation is different: instead of
copying the argument's value to a local object `x`, now local
reference `x` is created as a second name for the argument object `arg`. So
changing `x` you change the input object.

There are no references in `C`, they are the `C++` invention. If you
want a function to change an external object in `C`, you use pointers
as parameters (you can do pass-by-pointer in `C++` as well, of course):
```cpp
 void foo(int *x)      // pointer parameter
 {
     *x = 10;          // dereference the pointer, and change
                       // the value of the object to which it points
                       // (you should check if pointer is null in real code)
     x = new int(20);  // local pointer is reassigned,
                       // but external is not
 }

 int main()
 {
     int a = 1;
     int *p = &a;
     foo(p);
     // now a is 10, p still points to a 
 }
```
Though you can change the object to which the pointer points, you
cannot change the pointer itself. You *can* write `x = new int(10);`
in the `foo` function, but it will reassign the local pointer `x`. The
pointer which was passed to `foo` will be left unchanged, because its
value was copied to `x`: pass-by-pointer is a particular case of
pass-by-value.

But in `C++` you can mix stars with ampersands. Watch closely:
```cpp
 void foo(int * &x)   // parameter is a reference to a pointer to int
 {
     *x = 10;         // change the value of the object to which
                      // the pointer referenced by x points (yak!)
     x = new int(20); // change the value of the pointer itself
                      // since x is a reference, this affects the external pointer
 }

 int main()
 {
     int a = 1;
     int *p = &a;
     foo(p);
     // now a is 10, and p points to a heap object created from 'foo'
 }
```
In fact, many hardcore `C` programmers consider pass-by-reference an
evil. In `C`, it is guaranteed that a function will not change the
objects passed as arguments. Of course, if a function accepts a
pointer, it can change the object to which it points - but you *know
that*, because you know you are passing a pointer: you cannot pass an
`int` to a function which accepts `int*`. In `C++` there is no such
guarantee, because you can pass `int` to a function which accepts
`int` *and* to a function which accepts `int&`. So if you use
3rd-party functions and want to be sure they will not change the
passed-in objects, you need to check function declarations, to see if
they accept objects or references.

Why need pass-by-reference at all? Suppose you have defined a custom
type, which is big and heavy, e.g. it contains many data members. You
need to define a function which processes objects of that
type. You can use pass-by-value, even without pointers: pass the
object to the function, and then make the function `return` the
processed object:
```cpp
 struct MySuperBigStruct {
    // definition of members...
 };

 MySuperBigStruct foo(MySuperBigStruct x)
 {
     // process x
     return x;
 }

 int main()
 {
     MySuperBigStruct a;
     a = foo(a);
 }
```
The problem is: the larger the object, the more it takes to copy it,
and in this case the copy will be performed *twice*: first, when it
is copied as an argument to the local parameter, and then when it is
copied back from the `return`. The solution (the only one in `C`) is
to redefine the `foo` to make it pass-by-pointer:
```cpp
 void foo(MySuperBigStruct *x)
 {
     // dereference x and use *x to process the original object.
     // No need to return anything from the function
 }

 int main()
 {
     MySuperBigStruct a;
     foo(&a);
 }
```
Since the size of a pointer is always small, copying a pointer is
more efficient than copying a structure with possibly millions of bytes.
Finally, in `C++` you can make the function pass-by-reference, and
forget about stars and ampersands when calling the function:
```cpp
 void foo(MySuperBigStruct &x)
 {
     // process x; no need to return anything
 }
 
 int main()
 {
    MySuperBigStruct a; 
    foo(a);
 }
```
It is a bit tidier, but efficiency is roughly the same as in
pass-by-pointer. Of course, this minor syntactic sugar was not the
main reason to introduce references in `C++` (the main reason was to
enable operator overloading and copy constructors, among others).

Pass-by-reference is efficient even if the function accepts 
objects without modifying them - still, no copy
from argument to parameter. But in this case it is wise
to mark the parameter as `const`, so that if someone (maybe yourself)
mistakenly tries to change it in the function body, the compiler will
signal an error:
```cpp
 void foo(const MySuperBigStruct &x)
 {
     // x is a reference,
     // but it is read-only
 }
```
> **Note**: Under the hood, references are implemented with `const` pointers. As
> a C++ programmer, you should not care about that. However, there is
> one consequence worth remembering: passing primitive types
> like `int` by reference (or, for that matter, by
> pointer) might be less efficient than passing them by value.
> This is because simply copying an `int` (4 bytes) is faster than
> creating and copying a (hidden) pointer (8 bytes).

There is another reason for pass-by-reference: in `C++` you may have
objects that *are not allowed to be copied* (you can create such
objects by marking *copy constructors* with `= delete;` in their
`class/struct` definitions). E.g., your pal `std::cout` is such an
object (and yes, it is not an operator, nor a function, nor a type; it
is an *object* of the type `std::ostream`, for which the operator `<<` is
defined). You can't copy it, but you can declare references to it:
```cpp
 #include <iostream>
 
 int main()
 {
     auto x = std::cout; // error - cout is uncopiable
     auto &y = std::cout; // ok - y now is a second name for std::cout
     y << "World, hello!" << std::endl;
 }
```
So, if you need to pass an uncopiable object to a function, you have no choice
but to use pass-by-reference for it in function definition.

> ☢️ One of the rules of good design states that "things that have
> different functions should look differently". In this respect
> `C/C++` fails miserably. Thus, `*` in front of a pointer means
> dereferencing. Unless there is also a type name in front, in which case
> it means pointer declaration. `&` in front of a variable means the
> address of that variable. Unless there is also a type name in front,
> in which case it means reference declaration. On top of that, the
> term "dereferencing", which comes from `C`, is applicable to
> pointers, not references (references need no dereferencing). You
> will need practice to get accustomed to these rough edges, but you
> should forgive them - neither `C` (ca. 1970), nor `C++` (ca. 1985)
> were designed to be the languages easy to learn; they were designed
> to create efficient software (including operating systems!)


## 5. Values

Values might seem too obvious notions to define. But there
are some aspect about values which are far from obvious to a `C/C++`
newcomer. You may have noticed that the definition of *object*
involved the notion of *value*, and yet I've put the discussion of
values after the discussion of objects. This is not without a reason!

### Expressions vs Statements

To start with, not only objects have values. Values also represent
the result of expression evaluation. In fact, the definition of an
expression is that

⚠️  **An expression is a piece of code which is evaluated to some value.**

(note, *value* and *evaluation* have the same root). Those parts
of code which do something without being evaluated are called
*statements*, i.e., as a whole, a statement does not "return" a value.
A simplest statement is `;` (empty or null statement). There are
different types of statements:

- declaration statements (e.g., `int x = 10;`)
- selection statements (`if/else`-blocks, `switch`-blocks)
- iteration statements (`for`, `do`, `while` loops)
- expression statements - expressions followed by a null statement (`;`)
- others... (for more details, see https://en.cppreference.com/w/cpp/language/statements).

So, a statement is a more global language construct. Typically, a
function body is a sequence of statements. Statements might (and usually do)
contain expressions, but expressions cannot contain statements (but
expressions may contain sub-expressions). By
appending a semicolon to an expression you get an *expression
statement*. A primitive way to check if a piece of code is an
expression is to try to put it into parenthesis and pass it to `cout`. If compiler accepts that,
it is definitely an expression. E.g., `cout << (10) << endl;` works, but
`cout << (10;) << endl` doesn't. Hence, `10` is an expression, and `10;` is a
statement. Examples:
```cpp
 int main()
 {
     int x = 0;  // declaration statement
     x = 10;     // expression statement
     x++;        // expression statement
     ++x;        // expression statement
     someFunc(); // expression statement
 }
```
From here you can see that

- `int x = 0` is *not* an expression: it is not evaluated to some value, and `cout << (int x = 0) << endl` is an error; 
- But `x = 10` *is* an expression (with a side-effect). It assigns a value to
  `x` (the side effect) and then "returns" that value. So `cout << (x = 10) << endl`
  works fine and prints `10`. (Note that if you remove parenthesis it
  will not work, because the `<<` operator has higher precedence than `=`.) By the way, this is why you may chain assignments like `x = y = z = 10;` (where `x`, `y` and `z` should be declared earlier). First, the right-most assignment is evaluated and its result is used as a right-hand side for the next assignment, etc. It's the same as `x = (y = ( z = 10)));`
- `x++` and `++x` are both expressions with the side-effect of incrementing the
  object's value by 1. So the side-effect of these two expressions is
  the same, but *they return different values*: `x++` returns the
  value of `x` *before* incrementing, and `++x` returns the value of
  `x` *after* incrementing:
  ```cpp
      int x = 10;
      cout << x++ << endl; // prints 10
      x = 10;
      cout << ++x << endl; // prints 11

- A function call is always an expression, since functions always
  return something (even if `void`).

So, expressions might or might not *do something* (i.e., they might
have side-effects), but *they are always evaluated to a value*.

As a more complex example, consider this:
```cpp
 int x = if (some_condition) 10; else 20; // error
 int x = some_condition ? 10 : 20  ;      // ok
```
Even though `if (some_condition) 10; else 20;` is a perfectly valid code by itself, you cannot assign it to a
variable, because it is a statement (even though it contains expressions).
The second variant is legal, since the conditional operator `a ? b : c` is an expression:
```cpp
 int x = if ( some_condition ) 10 ; else 20 ;
 //          ^..............^ ^..^      ^..^  <- expressions
 //     ^-----------------------------------^ <- statement

 int x = some_condition ? 10 : 20  ;
 //     ^..............^ ^..^ ^..^  <- sub-expressions
 //     ^------------------------^  <- expression
```
> ☢️ The distinction between statements and expressions is by no means
> specific to `C/C++` - it is present in many languages (e.g., you 
> have it in `Python`). But some languages (e.g., `Lisp`, `Wolfram
> Language`) are designed in such a way that *everything* is an
> expression (i.e., expressions with side-effects replace statements).

Note that at global scope (not within functions) it is possible to
put only declaration statements. Also note that `=` in statements `int
x = 10;` and `x = 10;` have different meanings. In the first case
it performs initialization, and in the second case it performs
assignment. In modern `C++` there are good reasons to always use
*uniform initialization* with curly braces: `int x{10};` instead of
`int x = 10;` and `int x(10);`.

> **Note**: A more official definition of an expression is that it is a sequence
> of zero or more operators and at least one operand, which all
> specify a computation of a value. Note that operators may have different
> precedence, and are insensitive to spaces between them and operands, which sometimes
> might look confusing. E.g.:
> ```cpp
> int x = 10;
> while ( x --> 0) { /* do smth */ }
> // while x goes to zero ??
> ```
> Here, `-->` is not a single operator, it is two operators `--`
> and `>`. This line is equivalent to:
> ```cpp
> while ( (x--) > 0 ) { /* do smth */ }
> ```

### Temporary objects

Consider the following snippet of code:
```cpp
 int ten() { return 10; }
 int sqr(int x) { return x*x; }

 int main()
 {
      int two = 2;
      int x = sqr( two*ten() + sqr(3) ); // x is now 29^2
 }
```
Now ask yourself what happens inside the computer when the value for
`x` is calculated. The program goes
step-by-step, starting with sub-expressions. E.g., first it may evaluate the expression `ten()` and
find that it returns `10`, then evaluate the expression `two`,
multiply them together, then pass `3` to `sqr`, add it to the previous
result, and then pass the entire thing into outer `sqr`. The important
thing to realize is that at each step during this process the program *has to keep
intermediate values* of sub-expressions. Of course, like everything else in computer,
they are stored in its memory.
However, these intermediate values are not stored in ordinary objects,
they are stored in *temporary objects* (or *temporaries*, for
short); as they have no permanent
residency, you cannot apply the "address-of" operator to them:
```cpp
 int two = 2;
 int *p = &(3+two);   // error: (3+two) is a temporary object
 cout << &10 << endl; // same
```
Note, in the last line I used a simple literal (`10`) - you also may
think of it as a temporary object. Temporaries are created typically
on the stack, invisibly to the programmer.

> **Note**: A *literal* is a value which is not a result of evaluation of some
> expression, but which you, the programmer, indicate directly in the
> code. E.g., `10` is a literal of `int` type, `10L` is a literal of
> `long` type, `'a'` is a literal of `char` type, `"hello"` is a
> literal of `const char*` type, `"hello"s` is a literal of
> `std::string` type, `true` is a literal of `bool` type, etc. Not all
> types admit literals (e.g., there is no way to create a literal
> specifically of `std::vector<int>` type). Starting wih `c++11`, it
> is possible to create user-defined literals for custom types
> (https://en.cppreference.com/w/cpp/language/user_literal).

You also cannot assign a value to a temporary object:
```cpp
 (2 + 2) = 10; // error, you guessed it
```
For you this looks nonsense *a priori*, but that's because
you don't think like a compiler. In an alternative history of `C/C++`
such statement *could* be valid for a compiler, where it could mean
"put the value 10 into the temporary object which kept the value 4
obtained from evaluating 2+2". In real `C/C++` such assignment is an
error not because it doesn't make sense mathematically, but because it
doesn't make sense with regard to objects management.

### lvalues and rvalues

All in all, values in `C/C++` might be divided into two
categories:

- **lvalues** are values which may appear on the left side of the assignment
  operator (but they also might appear on the right);
- **rvalues** are values which may appear *only on the right side* of the
  assignment operator.

As a first approximation, lvalues are values stored in ordinary
objects, and rvalues are values stored in temporaries.

> ☢️ In fact, in `C++` the value categories are quite more complex, as
> you also have glvalues, prvalues, and xvalues; the distinction is
> important to know if you deal, e.g., with optimizations using move
> semantics. We will not consider it here, and the basic division
> between lvalues and rvalues should suffice to get you started.

If a function's parameter is defined to work in pass-by-value regime, then it can
accept both lvalues and rvalues as arguments. But if it is defined as
a reference, then it can accept *only* lvalues:
```cpp
 void foo(int x)  { /* do smth */ }
 void bar(int &x) { /* do smth */ }

 int main()
 {
     int a = 10;

     foo(a);      // ok (lvalue passed)
     foo(rand()); // ok (rvalue passed)
     foo(10);     // ok (rvalue passed)

     bar(a);      // ok (lvalue passed)
     bar(rand()); // error (rvalue passed)
     bar(10);     // error (rvalue passed)
 }
```
> ☢️ Remember I mentioned that `&&` defines rvalue references? If I redefine `bar` as
> ```cpp
> void bar(int &&x) { // do smth... }
> ```
> then the last two examples will be ok, but `bar(a);` will be an error.

One final example:
```cpp
 int x = 10; // global object

 int getXsimple()
 {
     return x;
 }

 int& getXref() // note the ampersand - returns a reference!
 {
     return x;
 }

 int main()
 {
     getXsimple() = 11; // error
     getXref() = 11;    // ok
 }
```
`getXsimple()` returns a temporary object with the value `10`, i.e. an
rvalue. `getXref()` returns a reference to the global object `x`,
i.e., an lvalue, so it can be assigned to directly.


# Declaration vs Definition 

Above, I have been using the terms "declare" and "define"
interchangeably, since that didn't matter much. In some languages, e.g.
`Python`, there is no distinction (you just assign to a variable and
that's it). But due to separate compilation, in `C/C++` these are two
different things.

- **Declarations** introduce names of objects and functions and
  describe their types;

- **Definitions** introduce names of objects and functions, describe
  their types, *and allocate/implement* them.

Any definition is also a declaration. But the converse is not true.

And then you have the One Definition Rule (ODR):

⚠️  Objects and functions might be **declared many times** within a
 project (even within a source file), but they must be **defined
 exactly once**.

Suppose you have two source files in your project, say,
`a.cpp` and `b.cpp`. In `a.cpp` you have defined a function:
```cpp
 // a.cpp

 int foo(double x)
 {
 // do something and return and int...
 // This function is used in other functions in the same source file
 }
```
Then it turns out that in `b.cpp` you also need to use that function
`foo`. Since source files are compiled independently from each other
into object files, you cannot call `foo` in `b.cpp`: the
compiler will say `foo` is unknown. You also cannot copy/paste the
definition of `foo` from `a.cpp` into `b.cpp`: although compiler will
produce object files, the linker will see two identical definitions in
two different object files, and will report an error about multiple
definitions. The solution is to declare the function in `b.cpp` without
defining it:
```cpp
 // b.cpp

 int foo(double x);
 // you can omit parameter names
 // in function declarations:
 int foo(double);
```
Now you can use `foo` in two files, and both compiler and linker will
be happy. The declaration tells the compiler:

"*This function exists somewhere (in another source file, or maybe in
the current file below), and it has the indicated type: it accepts one
`double` and returns an `int`. Please, compiler, go ahead! You will
compile the definition later (or maybe you already have compiled it
from another source file); the linker will take care and link the
function calls to the definition.*"

This information is enough for the compiler to perform type-checks in
places where this function is called: for that it doesn't need to know
what the function actually does.

Now, if you delete the file `a.cpp`, the compiler will still be able
to compile `b.cpp` into the binary object file (`g++ -c b.cpp`).
However, the linker will not be able to produce the executable and
will report an error about undefined reference to `foo`.

> **Note**: *Compiler errors* are about bad syntax and type-check
> inconsistencies, and they are provided with error locations (line
> numbers). *Linker errors* have no line numbers (linker knows nothing
> about source files!), and are typically about violations of the ODR:
> undefined references (declared but never defined) or duplicate
> definitions (defined more than once).

So, with functions it is quite simple: if you don't provide a
function's body, then that's a declaration; otherwise, that's a
definition.


## External and internal linkage 

If you work in a team, or with a large codebase, it might happen that
you accidentally use a name for a function which is already used in
another source file for some other function. To prevent such name
collisions, you can hide the function definition from the linker. For
that, use `static` keyword:
```cpp
 static void foo(double x)
 {
     cout << x << " passed to static foo" << endl;
 }
```
This is a definition of a function which will be visible only within
current source file (or, more correctly, within current translation
unit). Even if you copy this into another source file of the project,
this will not violate the ODR - this will produce two independent local
functions invisible to the linker. Such functions are said to have
_internal linkage_.

> **Note**: In modern `C++` to prevent name collisions between translation
> units, it is preferable to use anonymous namespaces instead of
> `static` declarations.

There is an "opposite" keyword - `extern` - which marks functions to
have _external linkage_, so that they are visible to the linker. But
since functions are visible to the linker by default, `extern` is
almost never used with functions. Why is this keyword needed then?
Because with objects things are different.


## Object declarations

You may think that this:
```cpp
 int x;
```
is a declaration, similar to the function `int foo();` above. In fact,
it is a definition. When the compiler sees such expression, it will
*allocate memory* for that object. Moreover, if in global scope, it
*will initialize* the object with the default value of 0. In other
words, in global scope the above statement is equivalent to:
```cpp
 int x = 0;
```
> **Note**: In local scope (e.g., within functions, including `main`)
> the `x` in `int x;` is *not* automatically initialized with 0 - it
> might contain a random number until you assign to it. The reason is
> in *optimization*. Global variables live in the data segment and are
> initialized only once when the program starts, so no problem here.
> But a function might be called million times (e.g., in a loop), and
> it makes no sense to spend time on initialization, if you don't need
> it (e.g., if you assign some values to the object later, depending
> on some conditions).

And here is the *proper declaration* of an object `x`:
```cpp
 extern int x; // declaration
```
This tells the compiler that it should not allocate memory for the
object `x` because it exits somewhere else. But if in this
expression you initialize `x` with some value, it again turns into
definition:
```cpp
 extern int x = 10; // definition!
```
As in the case of functions, if an object should be used *only* within
the current source file, it should be defined with the `static`
keyword:
```cpp
 static int x;
 ```
Now you know the difference:
```cpp
 // global scope
 int x;        
 static int y;
```
Both objects have static storage duration, but only the first one is
visible to the linker.

With `const` objects it is somewhat different. By default, their
definitions are invisible to the linker, so you have to mark them as
`extern` explicitly when you want to use them from multiple source
files.

To summarize:
```cpp
// global scope

/* NON-CONST OBJECTS */

int x1;                  // definition, visible to linker
int x2 = 10;             // same
extern int x3 = 10;      // same

static int x5;           // definition, invisible to linker
static int x6 = 10;      // same

extern int x4;           // declaration, visible to linker


/* CONST OBJECTS */ 

const int y1;            // error: definition, but const must be initialized
const int y2 = 10;       // definition, invisible to linker
extern const int y3 = 10;// definition, visible to linker
extern const int y4;     // declaration, visible to linker


/* FUNCTIONS */ 

void foo1();             // declaration, visible to linker
extern void foo2();      // same

void foo3() {}           // definition, visible to linker
                         // note the function body (even if empty)
extern void foo4() {}    // same

static void foo5();      // declaration, invisible to linker
static void foo6() {}    // definition, invisible to linker
```

So, the keywords `static` and `extern` control two distinct
properties of objects: linkage type and storage duration:
- For **global** objects, `static` affects _only_ the linkage
  (default linkage is external, `static` changes
  it to internal). The storage class of global objects is
  _always_ static, no matter if they are declared as `static int x`,
  `extern int x`, or just `int x`.
- For **local** objects, `static` affects _only_ the storage
  class (with or without `static`, in-function objects are invisible
  to the linker). By default, it is automatic, `static`
  changes it to static. As for `extern`, you cannot define
  `extern` object inside a function, you can only declare it.
- For **functions**, `static` affects only the linkage (functions
  do not have storage class), and `extern` has no any effect.

> **Note**: Do not confuse the notions of *lifetime* and *scope*. Lifetime is the
> property of an object, and scope is the property of an object's name
> (i.e. variable), which determines in which parts of the code this
> name is visible. Consider:
> ```cpp
>  static int a = 10;        // 1
>  extern int b = 10;        // 2
>  int c = 10;               // 3
> 
>  void foo()
>  {
>      int d = 10;           // 4
>      static int e = 10;    // 5
>      extern int f;         // 6
>      int* g = new int(10); // 7
>  }
> ```
> 
> | case           | storage duration | lifetime       | scope                                                   |
> | ----           | ---------------- | --------       | -----                                                   |
> | 1              | static           | entire program | translation unit                                        |
> | 2,3            | static           | entire program | global<sup>`*`</sup>                                         |
> | 4              | automatic        | block          | block                                                   |
> | 5              | static           | entire program | block                                                   |
> | 6              | static           | entire program | global<sup>`*`</sup>, but only block in this translation unit |
> | 7<sup>`**`</sup> | dynamic          | controllable   | *not applicable (has no name)*                          |
> 
> <sup>`*`</sup> The name is visible in all source files where you declare it with `extern` in global scope.
> 
> <sup>`**`</sup> The object whose address is assigned to `g` is implied. `g` itself is the same as case 4.


## Type declarations 

You can declare (without defining) not only objects and functions, but
even *types* (I kept it separate for simplicity). Of course, there is
no need to declare types like `int` since they are "built-in", but you
can declare custom types like this:
```cpp
    class MyCustomClass;
```
After this point, the compiler will know that a type named
`MyCustomClass` exists, but is not yet defined, so it is an
*incomplete type* for now. You can use incomplete types in very
restricted ways only. E.g., you cannot *define* objects with this type,
since the compiler will not even know how much memory to allocate for
that object. But you can *declare* objects and functions using an
incomplete type. Also, you can *define* pointers to incomplete types:
remember that the size of pointers is independent of
the type to which they point, so the compiler can always allocate a pointer.
```cpp
 class MyCustomClass;               // "forward" declaration

 extern MyCustomClass m;            // ok (object declaration)
 MyCustomClass foo(MyCustomClass);  // ok (function declaration)
 MyCustomClass *p;                  // ok (pointer definition)

 MyCustomClass mm;                  // error (object definition)
 MyCustomClass bar(MyCustomClass x) // error (function definition)
 {
     return x;
 }
```
Types also must obey the One Definition Rule, but only a "softer"
version of it. Here is the updated ODR:

⚠️  **Types, objects and functions might be *declared* many times within
 a project (even within a translation unit). But types must be
 *defined* exactly once within a translation unit. Objects and functions
 must be *defined* exactly once within the entire project.**

Why is forward declaration needed? I will point out at least two reasons:

1. Reducing build time. Say, you have a header file `a.h`, which
   contains lots of things, including the definition of type `A`:
   ```cpp
   // a.h
   class A {
    // definition of member variables and functions...
   };
   ```
   In another header file `b.h` you need to declare a function which
   accepts an object of type `A`. You could do it like this:
   ```cpp
   // b.h
   #include "a.h"

   void foo(A a);
   ```
   This will work fine, except that now the contents of `a.h` will be
   compiled each time the compiler will process all `cpp`-files which
   include `b.h`, even if those `cpp`-files do not use anything from
   `a.h`. To optimize, you can do this:
   ```cpp
   // b.h
   class A;

   void foo(A a);
   ```
   and then `#include "a.h"` only in one source file in which the
   function `foo` is *defined*. For large projects, the gain in build
   time might be significant.

2. Defining types with circular dependencies. Say, you need to define
   type `A` which has a method that accepts an object of type `B`, but
   type `B` has a method which accepts an object of type `A`. Forward
   declaration helps to break the vicious circle:
   ```cpp
   class B; // forward declaration of type B

   class A { // definition of type A
       void foo(B); // declaration of the method
       // incomplete type might be used in function declaration
   };

   class B { // definition of type B
       // type A is complete, so we can define the method:
       void bar(A a)
       {
           // do smth with a...
       }
   };

   // now type B is also complete, so we can define its 'foo' method
   void A::foo(B b)
   {
       // do smth with b...
   }
   ```

## Include guards

As you know, `#include` directives are replaced with the contents of included files. If a header file
contains only declarations, it is not a problem to include it many
times. But more often than not, headers contain also `class/struct`
definitions. In this case, including the header (as is) more than once
will violate the ODR. You may wonder why would someone include the same
header many times, but this happens more often than you think. E.g.:
```cpp
 // file a.h
 class A {
 // definition of class A...
 };
```
```cpp
 // file b.h
 #include "a.h"
 class B {
     A a; // class A is used in B
     // other members...
 };
```
```cpp
 // file c.h
 #include "a.h"
 class C {
     A a; // class A is used in C
     // other members...
 };
```
```cpp
 // file main.cpp
 #include "b.h"
 #include "c.h"
 // now contents of a.h are included two times 
 // ...
```
To make this work, put the following three preprocessor directives, called *include guards*, into `a.h` (in
fact, put them into all header files):
```cpp
 #ifndef A_H
 #define A_H

 // contents of the header

 #endif
```
When the preprocessor encounters the line `#ifndef SOMENAME` and the
macro `SOMENAME` is already defined at this point, it will remove all
following lines until the first `#endif`. Thus, when the contents of
`a.h` first appears in `main.cpp` through `#include "b.h"`, the macro
`A_H` is not yet defined, and all lines are pasted and processed. The
next line, `#define A_H` tells the preprocessor to define the macro
`A_H`. So when the contents of `a.h` are pasted for the second time
through `#include "c.h"`, this macro is already defined, and the
contents between `#ifndef` and `#endif` will be ignored. You can use
any unique name instead of `A_H`, but the tradition is to transform
the header file name to uppercase and replace `.` with `_`.

Note that include guards prevent ODR violation only with respect to
*type* definitions. If you have a *function* or *object* definition in
a header, and this header is included into multiple source files, you
will violate the ODR despite that all headers have include guards.
Therefore, typically, headers contain `class/struct` *definitions* and
function and object *declarations*. (Also note that a `class`
definition might contain only declarations of its methods; definitions
of methods are typically provided in source files).

> **Note**: There is an alternative to include guards:
> ```cpp
> #pragma once
> ```
> Put this single preprocessor directive at the top of the header, and
> that's it. It is not guaranteed by the standard, but all major
> compilers seem to support it.


## Headers and libraries

Now that you know the distinction between declarations and
definitions, you should understand how libraries work in `C/C++`. Say,
you need to make numeric integration, and decide to use GSL (GNU
Scientific Library). First, you install this library, which means that
your package manager (such as `apt` in `Ubuntu`) will download 1)
header files of the library which contains type definitions and
function declarations, and 2) the compiled definitions of those
functions in the form of the library `libgsl.a` (the convention is
that file names of libraries always have "lib"-prefix). Headers and
libraries will be put to some standard locations that `C/C++` compiler
knows about. Then in your code you do an `#include`:

```cpp
 #include <gsl/gsl_integration.h>

 int main()
 {
     // use some integration functions from GSL...
 }
```
If you then compile your program as usual, the linker will issue an
error about undefined references to the GSL functions you used: the header
file contains *only declarations*. You have to *link* them against
definitions which are stored in the libraries, and you do that with
`-l` flag followed (with or without spaces) by the library name
without the "lib"-prefix (and extension): `g++ main.cpp -lgsl` (in fact, for the
GSL library you will also have to indicate `-lgslcblas`, which is a
supporting library).

You may wonder why you don't provide any `-l` flags when you
`#include` e.g. `vector` or `iostream`, and that works fine. That's
because these headers are part of the `C++` *standard library*, which is
linked automatically.

> **Note**: You may have noted that both angular brackets and double
> quotes might be used when including files (`#include <file.h>` and `#include
> "file.h"`). The difference is that with brackets, the preprocessor searches
> for the file in some standard directories, and this form is
> typically used to include library headers. In case of quotes, the
> preprocessor will first look into current directory, so this form is
> used to include your own headers.

There are two types of libraries: static (also known as 'archives',
hence the `*.a` extension) and shared ('shared object', `*.so`). The
distinction is that the binary code which you use from static
libraries is copied by the linker at compile-time and becomes part of
your final executable. Binary code from shared libraries is not copied
into your program, but is loaded when your program starts. With static
libraries it is easier to distribute applications (you don't have to
worry about mismatch of versions), but executables have bigger sizes.
With shared libraries the sizes are smaller, because there is only one
copy of the library on your computer (many different executables may
load it at run-time, even simultaneously), but you need to check if
the version of the library is the one needed by an application. If you
have both versions of the same library when compiling you program, by
default, the shared version will be linked (to force linking static
version, use `-static` flag).

> ☢️ You can check any executable file with the `ldd` command, to see
> which shared libraries it depends on (e.g., `ldd /bin/ls`).

There are also *header-only* libraries (e.g., [`cxxopts`](https://github.com/jarro2783/cxxopts)) which contain
all definitions in headers, so you don't use the `-l` flag for them.
Such libraries cannot contain function and object definitions, they might only
contain definitions of types, templates and inline functions (I didn't
mention the latter two, but, like types, they also obey a "softer" version of the ODR).
The advantage of header-only libraries is the ease of use (you just `#include` and don't have
to worry if you have relevant binary libraries for your specific
platform), but they result in longer build times (remember that
ordinary libraries which you link with `-l` are already compiled).

> ☢️ Starting with the `c++20` standard, we also have a completely new system of
> *modules*. Unlike the traditional system, where you textually
> `#include` files, here you will be able to `import` binary code from
> modules (note, there is no `#` in front of `import` - it is not a
> business for the preprocessor!). Also, you can mix imports and
> includes in a single file. As of July 2022, the support for modules
> in both `g++` and `clang` is not complete.


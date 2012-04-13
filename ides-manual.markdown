---
layout: default
title: Ides Manual
permalink: manual.html
---
Ides is a strongly typed general-purpose programming language, drawing heavily from successful languages such as C, C# and Scala. Ides adopts a multi-paradigm approach to its language constructs, including functional programming elements such as lambdas and closures, and object oriented concepts like classes and polymorphism.

Standard Libraries
==================

This document will describe all of the syntax features that are defined. However, Ides is targeted toward many different audiences, including developers for systems with limited resources. The language is intended to be usable as a systems language. As a result, some language features may be disabled, depending on the availability of certain standard libraries. Class types, for example, require the presence of the garbage collection library.

Standard libraries are removed, not added. By default, the Ides compiler assumes that all standard libraries are available. In this mode, the entire language syntax is available. However, if a library is excluded - for example, the `--no-gc` switch will prevent the garbage collection library from being linked, which will disable support for class types - then the language features that depend on it will result in a syntax error.

This document assumes the default case, where all standard libraries are available. For more information on standard libraries, their dependencies on each other, and what syntax features they disable, see [Standard Libraries].

Syntax
======

Comments
--------
The behavior of comments is identical to C++: `//` indicates a line comment - everything after the comment, until the next newline, will be commented. `/*` begins a block comment, which ends at the next occurrence of `*/`.

Builtin Types
-------------
Ides has support for integral types in 8-bit, 16-bit, 32-bit and 64-bit signed or unsigned format. Unlike most languages, there is no `int` type. All integral types must explicitly include their size. The following are valid integral types:
*   `int8` - signed 8-bit integer.
*   `uint8` - unsigned 8-bit integer.
*   `int16` - signed 16-bit integer.
*   `uint16` - unsigned 16-bit integer.
*   `int32` - signed 32-bit integer.
*   `uint32` - unsigned 32-bit integer.
*   `int64` - signed 64-bit integer.
*   `uint64` - unsigned 64-bit integer.

Ides also supports 32-bit and 64-bit floating point types. Like integers, the size of the float must be specified. The following are valid built-in floating point types:
*   `float32` - 32-bit floating point number (equivalent to C `float`).
*   `float64` - 64-bit floating point number (equivalent to C `double`).

Other built-in types include:
*   `bool` - a boolean type.
*   `void` - null type, only useful for opaque pointers of type `void*`.

Ides also supports pointer types. Ides pointers work in the same way as they do in C. The syntax for a pointer type to, for example, an `int32` follows the C syntax convention: `int32*`. It is valid to create pointers to built-in types, struct types or other pointer types. Pointers to class types are not supported.

Note that there is no `const` keyword, and that Ides makes no attempt at ensuring [const correctness]("http://en.wikipedia.org/wiki/Const-correctness" Const-correctness). When using pointers, it is the responsibility of the developer to prevent writing to read-only memory.

Literals
--------

### Integer Literals ###
Integer literals in an Ides program are assumed to be of type `int32` - that is, all integer literals, unless specified otherwise (see Casting) are signed 32-bit integers.

### String Literals ###
String literals of the form `"string"` are assumed to be arrays of type `codepoint[]`.  The core standard library contains functions that mirror the functions found in C's `string.h`. These functions are safe from buffer overflows due to the way Ides handles arrays. See [Arrays] for more details on Ides array handling.

String literals may also be specified in the form `C"string"` which produces a null-terminated array constant of type `int8[]`. The form `WC"string"` is also available, and produces a constant of type `int16[]`. `LC"string"` produces a constant of type `int32[]`. It is //highly// recommended that these constructs be avoided. They are included for compatibility with C.

### Array Literals ###
Array literals are allowed using the `[ ... ]` syntax. All arguments within the brackets must be implicitly convertible to the type of the first argument. For example:

    [1, 2, 3];       // Ok. All arguments are int32. Type is int32[]
    ['1', 2, 3];     // Syntax error. No implicit conversion from int32 to int8.
    [1, 2, '3'];     // Ok. '3' gets converted to int32. Type is int32[].
    ['1', '2', '3']; // Ok. All arguments are int8. Type is int8[].

Arrays have a fixed length, which can be queried with `a.length`, which returns a `uint64`. Arrays can be indexed with `a[i]`. This expression will throw an exception if `i >= a.length` and if `--no-array-bounds-checks` was not specified to the compiler.

Variables
---------
Ides is a strongly typed language.

### Declarations ###
The basic variable declaration syntax is as follows:
    var x: int32;

Alternatively, the type of a variable can be inferred from the expression in an initializer.

    var x = 0; // Type of x is int32, because integer literals are int32.

Functions
---------
The basic form of a function declaration is:

    def MyFunction() {
    }

The return type of a function can be specified with the `=>` symbol:

    def MyFunction() => int32 {
    }

Arguments to a function are specified as they are in the body of a function:

    def MyFunction(var argA: int32, var argB = 0) => int32 { // argB is int32, and will default to 0.
    }

Functions can be declared `public`, `private`, `internal` or `extern`. Functions that are declared `extern` may not have a body.
    extern def OtherLibraryFunction(); // Extern method with no return type.
    
    public def MyFunction(var argA: int32, var argB = 0) => int32 { // MyFunction will be exported from the module.
    }

### Function Names ###
Functions have special privileges when it comes to identifiers. In addition to the standard alphanumeric identifier rule, which we'll call 'identifier characters' `[a-zA-Z_][a-zA-Z0-9_]*` function identifiers may also conform to a separate class of characters, called 'operator characters.' A function identifier may consist entirely of identifier characters, or entirely of operator characters, but not a mix of the two. 

A comprehensive list of operator characters:

*   `%`
*   `+`
*   `-`
*   `/` - Note that a function identifier may not contain `//` or `/*`, as that begins a comment.
*   `:`
*   `<`
*   `>`
*   `\`
*   `^`
*   `|`
*   `#`
*   `!`
*   `*`
*   `~`
*   `&`
*   `?`
*   `=` - A special case exists for the `=` symbol. `=` alone is not a valid function name.

### Operators ###
In addition to normal functions, a function can also be used as a prefix or infix operator, by appending the `prefix` or `infix` keyword to the definition. For example, if a function is defined as:

    public def infix vcross(var lhs: Vector3, var rhs: Vector3) => Vector3 {
        // Vector cross product
    }
then the function may be called in the following way:

    var result = v1 vcross v2; // v1, v2 and result of type Vector3


In the same way, a function may be declared as a prefix operator:

    public def prefix vlength(var v: Vector3) => int32 {
        // Vector length
    }

...and called as if the identifier was an operator:

    var len = vlength v1; // v1 of type Vector3, len of type int32


### Operator Precedence ###
Parentheses group operators together, and are the highest level of precedence.

All infix operators are left associative with equal precedence. This means that:

    a op b op c op d;

is evaluated as:

    ((a op b) op c) op d;

This is true even for arithmetic operators on numeric types. For example, the following code could produce unexpected results:

    var x = 10 + 20 * 30; // Evaluated as (10 + 20) * 30 = 900;

Prefix operators always take precedence over infix operators. For example:

    a && !b; // Evaluated as a && (!b);

Postfix operators always take precedence over prefix operators. For example:

    !a(); // Evaluated as !(a());


Structs
-------
Structs are user-defined unmanaged (that is, not subject to garbage collection) data structures. The format for a struct definition is:

    struct StructName {
    } // Note for C developers: No semicolon here.

### Anonymous Structs (Tuples) ###
Ides exposes the C concept of an anonymous struct in the form of a tuple. The syntax for a tuple type is:

    (type1, type2, ..., typen);


This can be used to return multiple values from a function:

    extern def QueryResource() => (bool, int32);

This hypothetical `QueryResource()` function is able to return a `bool` success flag, as well as an `int32` data.

To create a new tuple, the syntax is similar. This `QueryResource()` function may have a return statement that looks like:

    return (true, val);


The caller can index the tuple:

    var result = QueryResource();    // result is of type (bool, int32)
    if (result._1) {                 // result._1 is of type bool
        PerformOperation(result._2); // result._2 is of type int32
    }                                // Further members could be addressed with result._3, result._4 etc.

Note that 1-tuples are not supported. `(val)` is a simple parenthetical grouping.

Casting
-------
### Implicit Conversions ###
![Implicit Conversions](/images/ImplicitConversions.png "Implicit Conversions")

### Explicit Casts ###
Ides allows a few implicit casts.

    // y is of type SomeType, which was a cast from an instance of x.
    // This requires that an appropriate cast function exist.
    var y = x.cast<SomeType>;


### Cast Functions ###
How to define new valid casts.


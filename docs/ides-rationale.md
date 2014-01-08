---
layout: default
title: Rationale for the Ides Programming Language
permalink: rationale.html
---
* Table of contents goes here.
{:toc}


Introduction
============

Abstract
--------

Ides is a strongly typed general-purpose programming language, drawing influence from a large breadth of software design patterns. Ides adopts a multi-paradigm approach to its language constructs, and strives to balance pragmatism and ideology. Though the current selection of programming languages is vast, most require a large runtime environment, making them unsuitable to some types of high performance applications, or are overly idealistic with their approach to software design. Ides intends to find a pragmatic middle ground in the ecosystem of native languages.

Scope
-----

Stuff.

References
----------

Other stuff.

Definitions of Terms
--------------------

What junk means.



Language
========

Comments
--------
`//` indicates a line comment - everything after the comment, until the next newline, will be commented. 

`/*` begins a block comment, which ends at the next occurrence of `*/`.

Type System
-----------
The Ides type system is built on two fundamental concepts: `void`, or the [top type](http://en.wikipedia.org/wiki/Top_type), which is a supertype of all other types and has no values, and `Nothing`, or the [bottom type](http://en.wikipedia.org/wiki/Bottom_type), which is a subtype of all other types.

### `void` ###
`void` is used as the return type of functions that do not return data, or for opaque pointers of type `void*`.

### `nothing` ###
`Nothing` as a return type to a function indicates that the function will never return. `throw` expressions also evaluate to type `Nothing`. This is useful when combined with the [null coalescing operator](https://en.wikipedia.org/wiki/Null_coalescing_operator) `or`. For example:

    return some_value or throw new KeyNotFoundException();

Because `Nothing` is a subtype of the type of `some_value`, this is a legal expression. However, `throw` will not actually evaluate to any value, but instead will throw an exception without returning.

### Builtin Types ###
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

### Struct Types ###
Struct types are defined, as in C, using the `struct` keyword. Struct types can be allocated either as stack data or as heap data using the `new` keyword: `new MyStruct`. Structs may only contain public data members, and may not contain methods. Structs may not inherit or be inherited.

### Pointer Types ###
Ides also supports pointer types. Ides pointers work in the same way as they do in C. The syntax for a pointer type to, for example, an `int32` adopts C syntax: `int32*`. It is valid to create pointers to built-in types, struct types or other pointer types. Pointers to class types are not supported.

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

    void MyFunction() {
    }

Functions may also return values:

    int32 MyFunction() {
	return 4; // This is not a useful function.
    }

If no return value is specified, it is assumed that the function will return `void`.

Functions may also return `nothing`, but it is an error for a function that returns `nothing` to ever actually return a value.

    nothing Error(String s) {
        throw new SomeException(s);
    }

Arguments to a function are specified as they are in the body of a function:

    int32 MyFunction(var argA: int32, var argB = 0) { // argB is int32, and will default to 0.
        return 4;
    }

Functions can be declared `public`, `private`, `internal` or `extern`. Functions that are declared `extern` may not have a body.

    extern void OtherLibraryFunction(); // Extern method with no return type.
    
    public void MyFunction(var argA: int32, var argB = 0) => int32 { // MyFunction will be exported from the module.
    }

### Function Names ###
Functions have special privileges when it comes to identifiers. In addition to the standard alphanumeric identifier rule, which we'll call 'identifier characters' `[a-zA-Z_][a-zA-Z0-9_]*` function identifiers may also conform to a separate class of characters, called 'operator characters.' A function identifier may consist entirely of identifier characters, or entirely of operator characters, but not a mix of the two. 

A comprehensive list of operator characters: `%` `+` `-` `/` `:` `<` `>` `\` `^` `|` `#` `!` `+` `~` `&` `?` `=`.

Note:
+ A special case exists for the `=` symbol. `=` alone is not a valid function name.
+ A function identifier may not contain `//` or `/*`, as that begins a comment.

### Operators ###
In addition to normal functions, a function can also be used as a prefix or infix operator, by appending the `prefix` or `infix` keyword to the definition. For example, if a function is defined as:

    public infix Vector3 vcross(var lhs: Vector3, var rhs: Vector3) {
        // Vector cross product
    }
then the function may be called in the following way:

    var result = v1 vcross v2; // v1, v2 and result of type Vector3


In the same way, a function may be declared as a prefix operator:

    public prefix int32 vlength(var v: Vector3) {
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

### Pure Functions ###
TODO: Explain pure functions

Casting
-------
### Implicit Conversions ###

#### Implicit Conversions with Numeric Types ####
Implicit conversions are allowed on integral types, provided no data is lost or reinterpreted. This means that there are no implicit conversions between signed and unsigned integers of the same size, and there are no implicit conversions from any signed integer to any unsigned integer.

Implicit conversions are always allowed from integral types to floating point types.

This flow chart shows all implicit conversions permitted on numeric types.

![Implicit Conversions](/images/ImplicitConversions.png "Implicit Conversions")

Implicit conversions are not allowed on pointer types.

#### Implicit Conversions with String Types ####
TODO: Should we allow implicit conversions between Ides strings, C strings, WC/LC strings? String class objects? UTF-8/16/32?

#### Implicit Conversions with Reference Types ####
Implicit conversions are permitted for reference types provided that the destination type is a supertype of the source type.

### Explicit Casts ###
Ides allows a few implicit casts.

    // y is of type SomeType, which was a cast from an instance of x.
    // This requires that an appropriate cast function exist.
    var y = x as SomeType;
    
    // Alternatively:
    var z = x.as(SomeType);


#### Cast Functions ####
TODO: How to define new valid casts.

    MyType as(var o: class) {
        return new MyType(magic(o));
    }

Standard Libraries
==================
TODO: Describe the standard libraries that ship with Ides.

Math
----
TODO: Math functions, vector functions, matrix functions

Garbage Collection
------------------
TODO: Describe GC and classes.

Collections
-----------
TODO: Describe collections library.

Compiler
--------
TODO: Describe interfaces for hosting an Ides compiler/runtime


Compiler Options
================

This document will describe all of the syntax features that are defined. However, Ides is targeted toward many different audiences, including developers for systems with limited resources. The language is intended to be usable as a systems language. As a result, some language features may be disabled, depending on the availability of certain standard libraries. Class types, for example, require the presence of the garbage collection library.

Standard libraries are removed, not added. By default, the Ides compiler assumes that all standard libraries are available. In this mode, the entire language syntax is available. However, if a library is excluded - for example, the `--no-gc` switch will prevent the garbage collection library from being linked, which will disable support for class types - then the language features that depend on it will result in a syntax error.

This document assumes the default case, where all standard libraries are available. For more information on standard libraries, their dependencies on each other, and what syntax features they disable, see [Standard Libraries].



---
layout: default
title: Ides Compiler Architecture
permalink: architecture.html
---
* Table of contents goes here.
{:toc}


Overview
========

This document is provides a broad overview of the systems and frameworks that make up the Ides compiler. It is intended for developers who are involved in the development of the compiler internals, and assumes familiarity with the language itself and some familiarity with LLVM. Since the compiler is built on LLVM, much of the low-level work, such as linking native binaries, targeting platforms, and code optimization, are outside the scope of this project and will not be discussed in this document.

Compiler Workflow
-----------------


`ilib` Metadata
-------------

An ilib file is defined as an LLVM module with specific [metadata](http://llvm.org/docs/LangRef.html#metadata) that allows the Ides compiler to maintain type and linkage information between compile units. This metadata serves a similar purpose to header files in the C family of languages.

The root of the metadata is in a named metadata node called `!ides.link`, which contains the collection of declarations found at the top level of the compilation unit. When an ilib is linked into another compilation, the compiler recursively restores these declarations into the global scope before parsing or compiling any Ides code.

Structural Components
=====================

The structural components are the data objects which describe Ides source code at a high level. In general, these classes keep and provide access to information about the program, but are not generally aware of the compilation process.

Abstract Syntax Tree
--------------------

### Declarations ###

Declarations objects describe the creation of new program elements, such as types, functions and variables. 

### Statements and Expressions ###

Statements and expressions are the primary building blocks of code behavior. Expressions are identical to statements, except that they have a type and may produce data.

Types
-----

All types within Ides inherit from the class `Ides::Types::Type`. Types are responsible for managing implicit conversion and polymorphism rules.

### Numeric Types ###

### Compound Types ###

### Pointers and References ###

Declaration Contexts
--------

Declaration contexts are scopes in which symbols may be defined. Symbols include type names, namespaces, and variable and function names. Most declaration contexts are of type `HierarchicalConcreteDeclarationContext`, which is capable of resolving names into parent scopes.

Note that a declaration context is not necessarily the same as a block scope within the language. Contexts exist for public, private and internal members of a compilation unit. Functions containing compound-statement blocks will actually have two declaration contexts: one for the function arguments, and child context for symbols defined within the function body.

Procedural Components
=====================

The procedural components work with structural components, generally following the [visitor pattern](http://en.wikipedia.org/wiki/Visitor_pattern). At a high level, these are the components that are responsible for producing forms of compiler output.

AST Visitors
------------

### `CodeGen` ###

The `CodeGen` class is the producer of IL instructions.


Type Visitors
-------------

### `DIGen` ###

The `DIGen` class is responsible for producing [debugging information](http://llvm.org/docs/SourceLevelDebugging.html) about data structures and methods defined within the compilation unit.

### `LLVMTypeVisitor` ###

The `LLVMTypeVisitor` is responsible for converting Ides types into LLVM types.

Hybrid Visitors
---------------

### `ReflectionGen` ###

The `ReflectionGen` class produces all of the static data structures necessary for reflection.

### `MetadataSerializer` ###

The `MetadataSerializer` class translates declarations and types into LLVM metadata.

Utility Components
==================

Diagnostics
-----------


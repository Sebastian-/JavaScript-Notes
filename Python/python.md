# Python

## General Concepts

* Everything in Python is an object, even things like functions, modules, and classes. There is no distinction between 'primitive' types and any other types, as there is in Java or C++. That said, Python does support a core set of types via special syntax for their instantiation. These include Numbers, Strings, Lists, Dictionaries, Tuples, Files, Sets, Booleans, and the None type.

* Python is dynamically typed - there are no type declarations in Python. The type of a variable is inferred from the object assigned to it.

* Python is strongly typed - an error will be raised when an operation is called on an object whose type does not support it.

## Types and Operations

### Numbers

* Python 3 integers do not overflow. An integer can grow to any size as long as it will fit in memory.

### Strings

* Strings are sequences of characters.

## Differences between Python 3.X and 2.X

* 2.X uses a long integer type to represent large integers. Literals of this type end with 'l' or 'L' - eg. 10000000000000000L. 3.X integers can represent large intergers by default, and support for long literals has been removed.
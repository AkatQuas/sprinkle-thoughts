# Building Abstractions with Data

Programs are typically designed to model complex phenomena, and more often than not one must construct computational objects that have several parts in order to model real-world phenomena that have several aspects. Any programming language would provide the means for building abstractions by combining data objects to form compound data.

We need compound data in programming languages for:

- to elevate the conceptual level at which we can design our programs,

- to increase the modularity of our designs,

- to enhance the expressive power of our language.

Just as the ability to define procedures enables us to deal with processes at a higher conceptual level than that of the primitive operations of the language, the ability to construct compound data objects enables us to deal with data at a higher conceptual level than that of the primitive data objects of the language.

The use of compound data also enables us to increase the modularity of our programs. The general technique of isolating the parts of a program that deal with how data objects are represented from the parts of a program that deal with how data objects are used is a powerful design methodology called _data abstraction_.

As with compound procedures, the main issue to be addressed is that of abstraction as a technique for coping with complexity, and we will see how data abstraction enables us to erect suitable abstraction barriers between different parts of a program.

The key to forming compound data is that a programming language should provide some kind of “glue” so that data objects can be combined to form more complex data objects.

One key idea in dealing with compound data is the notion of _closure_ —that the glue we use for combining data objects should allow us to combine not only primitive data objects, but compound data objects as well. Another key idea is that compound data objects can serve as _conventional interfaces_ for combining program modules in mix-and-match ways.

Using _symbolic expressions_ and various alternatives for representing sets of objects. There are many ways in which a given data structure can be represented in terms of simpler objects, and the choice of representation can have significant impact on the time and space requirements of processes that manipulate the data.

Maintaining modularity in the presence of _generic operations_, which helps to handle many different types of data, requires more powerful abstraction barriers than can be erected with simple data abstraction alone.

In particular, we introduce datadirected programming as a technique that allows individual data representations to be designed in isolation and then combined additively.

## Introduction to Data Abstraction

Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects.

The basic idea of data abstraction is that the programs should use data in such a way as to make no as- sumptions about the data that are not strictly necessary for performing the task at hand. At the same time, a _concrete_ data representation is defined independent of the programs that use the data.

The interface between these two parts of our system will be a set of procedures, called _selectors_ and _constructors_, that implement the abstract data in terms of the concrete representation.

#### Pairs

To enable the program to implement the concrete level of the data abstraction, the language provides a compound structure called a _pair_.

A pair is a data object that can be given a name and manipulated, just like a primitive data object.

The ability to combine pairs means that pairs can be used as general-purpose building blocks to create all sorts of complex data structures. Data objects constructed from pairs are called _list-structured_ data.

### Abstraction Barriers

In general, the underlying idea of data abstraction is to identify for each type of data objects of that type will be expressed, and then to use only those operations in manipulating the data.

The details of how one level is implemented are irrelevant to the rest levels. In effect, procedures at each level are the interfaces that define the abstraction barriers and connect the different levels.

Using abstraction barriers makes programs much easier to maintain and to modify. Any complex data structure can be represented in a variety of ways with the primitive data structures provided by a programming language. Of course, the choice of representation influences the programs that operate on it; thus, if the representation were to be changed at some later time, all such programs might have to be modified accordingly.

Constraining the dependence on the representation to a few interface procedures helps us design programs as well as modify them, because it allows us to maintain the flexibility to consider alternate implementations.

### What Is Meant by Data?

But exactly what is meant by data? It is not enough to say _whatever is implemented by the given selectors and constructors_. In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation.

## Hierarchical Data and the Closure Property

Pairs provide a primitive _glue_ to construct compound data objects.

The ability to create pairs whose elements could be pairs is the essence of list structure's importance as a representational tool. In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation. Closure is the key to power in any means of combination because it premits programmers to create _hierarchical_ structures-structures made up of parts, which themselves are made up of parts, and so on.

> Throughout the book, the meaning of _closure_ is from abstract algebra, and its meaning is where a set of elements is said to be closed under an operation if applying the operation to elements in the set produces an element that is again an element of that set. _代数闭包（域）_ or _algebraic closure_.

### Representing Sequences

One of the useful structures we can build with pairs is a sequence—an ordered collection of data objects.

> List, Array

#### List operation

It is customary to number the elements of the list beginning with index 0.

The `length` procedure implements a simple recursive plan. The reduction step is:

- The `length` of an empty list is `0`,
- The `length` of any list is `1` plus the `length` of the after list.

`append` takes two lists as arguments and combines their elements to make a new list, using a recursive plan:

- If `list1` is the empty list, then the result is just `list2`.
- Otherwise, append the first element of `list1` and `list2`, and add the rest `list1` onto the result

#### Mapping over lists

One extremely useful operation is to apply some transformation to each element in a list and generate the list of results. We can abstract this general idea and capture it as a common paern expressed as a higher-order procedure.

The higher-order procedure here is called `map`. `map` takes as arguments a procedure of one argument and a list, and returns a list of the results produced by applying the procedure to each element in the list.

`map` is an important construct, not only because it captures a common pattern, but because it establishes a higher level of abstraction barrier that isolates the implementation of procedures that transform lists from the details of how the elements of the list are extracted and combined. This abstraction gives us the flexibility to change the low-level details of how sequences are implemented, while preserving the conceptual framework of operations that transform sequences to sequences.

### Hierarchical Structures

Another way to think of sequences whose elements are sequences is as _trees_. The elements of the sequence are the branches of the tree, and elements that are themselves sequences are subtrees.

Recursion is a natural tool for dealing with tree structures, since we can often reduce operations on trees to operations on their branches, which reduce in turn to operations on the branches of the branches, and so on, until we reach the leaves of the tree.

#### Mapping over trees

Just as `map` is a powerful abstraction for dealing with sequences, `map` together with recursion is a powerful abstraction for dealing with trees.

### Sequences as Conventional Interfaces

If we could organize our programs to make the signal-flow structure manifest in the procedures we write, this would increase the conceptual clarity of the resulting code.

#### Sequence Operations

The key to organizing programs so as to more clearly reflect the signal-flow structure is to concentrate on the _signals_ that flow from one stage in the process to the next. If we represent these signals as lists, then we can use list operations to implement the processing at each of the stages.

The value of expressing programs as sequence operations is that this helps us make program designs that are modular, that is, designes that are constructed by combining relatively independent pieces. We can encourage modular design by providing a library of standard components together with a conventional interface for connecting the components in flexible ways.

Sequences, implemented here as lists, serve as a conventional interface that permits us to combine processing modules. Additionally, when we uniformly represent structures as sequences, we have localized the data-structure dependencies in our programs to a small number of sequence operations. By changing these, we can experiment with alternative representations of sequences, while leaving the overall design of our programs intact.

#### Nested Mappings

We can extend the sequence paradigm to include many computations that are commonly expressed using nested loops.

## Symbolic Data

We extend the representational capability of our language by introducing the ability to work with arbitrary symbols as data.

> MatLab has the ability to do symbolic mathematical work.

### Quotation

In order to manipulate symbols we need a new element in the language: the ability to _quote_ a data object.

The expression `(list a b)` constructs a list of the values of a and b rather than the symbols themselves. This issue is well known in the context of natural languages, where words and sentences may be regarded either as semantic entities or as character strings (syntactic entities).

Using quotation, such as `(list 'a 'b)` allows us to type in compound objects, using the conventional printed representation.

```
(define a 1)
(define b 2)
(list a b)
// output is (1 2)
(list 'a 'b)
// output is (a b)
(list 'a b)
// output is (a 2)
```

### Representing Sets

There are a number of possible representation for sets. Informally, a set is simply a collection of distinct objects.

1. Sets as unordered lists

   One way to represent a set is as a list of its elements in which no element appears more than once. The empty set is represented by the empty list.

1. Sets as ordered lists

   One way to speed up our set operations is to change the representation so that the set elements are listed in increasing order. To do this, we need some way to compare two objects so that we can say which is bigger.

1. Sets as binary trees

   To do better than the ordered-list representation, we arrange the set elements in the form of a tree. Each node of the tree holds one element of the set, called the _entry_ at that node, and a link to each of two other (possibly empty) nodes. The _left_ link points to elements smaller than the one at the node, and the _right_ link to elements greater than the one at the node.

   One way to solve this unbalanced tree problem is to define an operation that transforms an arbitrary tree into a balanced tree with the same elements. Then we can perform this transformation after every few _adjoin-set_ operations to keep our set in balance. There are also other ways to solve this problem, most of which involve designing new data structures for which searching and insertion both can be done in `Θ(logn)` steps

### Example: Huffman Encoding Trees

In general, we can attain significant savings if we use variable-length prefix codes that take advantage of the relative frequencies of the symbols in the messages to be encoded.

A Huffman code can be represented as a binary tree whose leaves are the symbols that are encoded. At each non-leaf node of the tree there is a set containing all the symbols in the leaves that lie below the node. In addition, each symbol at a leaf is assigned a weight (which is its relative frequency), and each non-leaf node contains a weight that is the sum of all the weights of the leaves lying below it. The weights are not used in the encoding or the decoding process.

## Multiple Representations for Abstract Data

In addition to the data-abstraction barriers that isolate different design choices from each other and permit different choices to coexist in a single program.

Data may be represented in different ways by different parts of a program. This requires constructing _generic procedures_ - procedures that can operate on data that may be represented in more than one way. Our main technique for building generic procedures will be to work in terms of data objects that have _type tags_, that is, data objects that include explicit _information about how they are to be processed_.

_Data-directed_ programming is a powerful and convenient implementation strategy for additively assembling systems with generic operations.

The discipline of data abstraction ensures that the different implementation of same selectors will work with different constructor representations.

### Tagged data

One way to view data abstraction is as an application of the _principle of least commitment_.

The abstraction barrier formed by the selectors and constructors permits programmers to defer to the last possible moment the choice of a concrete representation for the data objects and thus retain maximum flexibility in the system design.

A straightforward way to accomplish this distinction between different constructor is to include a _type tag_. Then we can use the tag to decide which selector to apply.

Each generic selector is implemented as a procedure that checks the tag of its argument and calls the appropriate procedure for handling data of that type.

Since each data object is tagged with its type, the selectors operate on the data in a generic manner. That is, each selector is defined to have a behavior that depends upon the particular type of data it is applied to.

> Factory Pattern

### Data-Directed Programming and Additivity

The general strategy of checking the type of a datum and calling an appropriate procedure is called _dispatching on type_.

This is a powerful strategy for obtaining modularity in system design. On the other hand, implementing the dispatch has two significant weaknesses:

1. the generic interface procedures must know about all the different representations.
1. no two procedures in the entire system have the same name even though the individual representations can be designed separately.

To understand how data-directed programming works, begin with the observation that whenever we deal with a set of generic operations that are common to a set of different types, we are, in effect, dealing with a two-dimensional table that contains the possible operations on one axis and the possible types on the other axis. The entries in the table are the procedures that implement each operation for each type of argument presented.

#### Message passing

The key idea of data-directed programming is to handle generic operations in programs by dealing explicitly with operation-and-type tables.

The style of programming organizes the required dispatching on type by having each operation take care of its own dispatching. In effect, this decomposes the operation-and-type table into rows, with each generic operation procedure repersenting a row of the table.

An alternative implementation strategy is to decompose the table into columns and, instead of using _intelligent operations_ that dispatch on data types, to work with _intelligent data objects_ that dispatch on operation names. We can do this by arranging things so that a data object is represented as a procedure that takes as input the required operation name and performs the operation indicated.

This style of programming is called _message passing_. The name comes from the image that a data object is an entity that receives the requested operation name as a _message_. Message passing is not a mathematical trick but a useful technique for organizing systems with generic operations.

## Systems with Generic Operations

Generic interface procedures helps programmers to link the code that specifies the data operations to the several representations. The same idea is not only suitable to define operations that are generic over different representations but also to define operations that are generic over different kinds of arguments.

### Generic Arithmetic Operations

Creating a table to store types and operations.

> Operator overloading

### Combining Data of Different Types

We have gone to great pains to introduce barriers between parts of our programs so that they can be developed and understood separately. We would like to introduce the cross-type operations in some carefully controlled way, so that we can support them without seriously violating our module boundaries.

One way to handle cross-type operations is to design a different procedure for each possible combination of types for which the operation is valid.

This technique works, but it is cumbersome. With such a system, the cost of introducing a new type is not just the construction of the package of procedures for that type but also the construction and installation of the procedures that implement the cross-type operations.

#### Coercion

In the general situation of completely unrelated operations acting on completely unrelated types, implementing explicit cross-type operations, cumbersome though it may be, is the best that one can hope for.

Often the different data types are not completely independent, and there may be ways by which objects of one type may be viewed as being of another type. This process is called _coercion_.

> type casting

In general, we can implement this idea by designing coercion procedures that transform an object of one type into an equivalent object of another type.

Although we still need to write coercion procedures to relate the types (possibly n<sup>2</sup> procedures for a system with `n` types), we need to write only one procedure for each pair of types rather than a different procedure for each collection of types and each generic operation. What we are counting on here is the fact that the appropriate transformation between types depends only on the types themselves, not on the operation to be applied.

On the other hand, there may be applications for which our coercion scheme is not general enough. Even when neither of the objects to be combined can be converted to the type of the other it may still be possible to perform the operation by converting both objects to a third type.

#### Hierarchies of types

Often there is more _global_ structure in how the different types relate to each other, this is a so-called _hierarchy of types_.

#### Inadequacies of hierarchies

In general, a type may have more than one subtype, or more than one supertype.

This multiple-supertypes issue is particularly thorny, since it means that there is no unique way to _raise_ a type in the hierarchy. Finding the _correct_ supertype in which to apply an operation to an object may involve considerable searching through the entire type network on the part of a procedure such as _apply-generic_. Since there generally are multiple subtypes for a type, there is a similar problem in coercing a value _down_ the type hierarchy.

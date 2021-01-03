# Computing with Register Machines

In this chapter we will describe processes in terms of the step-by-step operation of a traditional computer. Such a computer, or _register machine_, sequentially executes _instructions_ that manipulate the contents of a fixed set of storage elements called _registers_.

A typical register machine instruction applies a primitive operation to the contents of some registers and assigns the result to another register. The descriptions of processes executed by register machines will look very much like _machine-language_ programs for traditional computers.

## Designing Register Machines

To design a register machine, we must design its _data paths_ (registers and operations) and the _controller_ that sequences these operations.

An operation that computes a value from constants and the contents of registers is represented in a data-path diagram by a trapezoid containing a name for the operation.

Overall, the data-path diagram shows the registers and operations that are required for the machine and how they must be connected.

### A Language for Describing Register Machines

To make it possible to deal with complex machines, we will create a language that presents, in textual form, all the information given by the data-path and controller diagrams.

We define the data paths of a machine by describing the registers and the operations:

- To describe a register, we give it a name and specify the buttons that control assignment to it. We give each of the buttons a name and specify the source of the data that enters the register under the button's control. (The source is a register, a constant, or an operation.)

- To describe an operation, we give it a name and specify its inputs (registers or constants).

We define the controller of a machine as a sequence of _instructions_ together with _labels_ that identify _entry points_ in the sequence. An instruction is one of the following:

- The name of a data-path button to push to assign a value to a register.

- A _test_ instruction to perform a specified test, or assertion.

- A conditional branch to location indicated by a controller label, based on the result of the previous test. If the test is false, the controller should continue with the next instruction in the sequence. Otherwise, the controller should continue with the instruction after the label.

The machine starts at the beginning of the controller instruction sequence and stops when execution reaches the end of the sequence. Except when a branch changes the flow of control, instructions are executed in the order in which they are listed.

Unfortunately, it is difficult to read such a description. In order to understand the controller instructions we must constantly refer back to the definitions of the button names and the operation names, and to understand what the buttons do we may have to refer to the definitions of the operation names. We will thus transform our notation to combine the information from the data-path and controller descriptions so that we see it all together.

We will omit the data-path description, leaving only the controller sequence.

This form of description is easier to read, but it also has disadvantages:

- It is more verbose for large machines, because complete descriptions of the data-path elements are repeated whenever the elements are mentioned in the controller instruction sequence. Moreover, repeating the data-path descriptions obscures the actual data-path structure of the machine; it is not obvious for a large machine how many registers, operations, and buttons there are how they are interconnected.

- Because the controller instructions in a machine definition look like Lisp expressions, it is easy to forget that they are not arbitrary Lisp expressions. They can notate only legal machine operations. For example, operations can operate directly only on constants and the contents of registers, not on the results of other operations.

In spite of these disadvantages, we will use this register-machine language throughout this chapter, because we will be more concerned with understanding controllers than with understanding the elements and connections in data paths.

> However, that data-path design is crucial in designing real machines.

### Abstraction in Machine Design

We will often define a machine to include _primitive_ operations that are actually very complex. The fact that we have swept a lot of complexity under the rug, however, does not mean that a machine design is unrealistic. We can always replace the complex _primitives_ by simpler primitive operations.

### Subroutines

When designing a machine to perform a computation, we would often prefer to arrange for components to be shared by different parts of the computation rather than duplicate the components.

> Something looks like interrupts, sub-procedure invokation, etc.

> This might be useful in compiler optimaztion.

A more powerful method for implementing subroutines is to have the continue register hold the label of the entry point in the controller sequence at which execution should continue when the subroutine is finished. Implementing this strategy requires a new kind of connection between the data paths and the controller of a register machine: there must be a way to assign to a register a label in the controller sequence in such a way that this value can be fetched from the register and used to continue execution at the designated entry point. However, it might lead to some problem if we assgin different value to the same continue register in every subroutine, just think about nested subroutine calls, which is recursion.

### Using a Stack to Implement Recursion

With the ideas illustrated so far, we can implement any iterative process by specifying a register machine that has a register corresponding to each state variable of the process. The machine repeatedly executes a controller loop, changing the contents of the registers, until some termination condition is satisfied. At each point in the controller sequence, the state of the machine (representing the state of the iterative process) is completely determined by the contents of the registers (the values of the state variables).

Implementing recursive processes, however, requires an additional mechanism.

Nevertheless, we can implement the recursive process as a register machine if we can arrange to use the same components for each nested instance of the machine. This is plausible because, _although the recursive process dictates that an unbounded number of copies of the same machine are needed to perform a computation, only one of these copies needs to be active at any given time_. So when the machine encounters a recursive subproblem, it can suspend work on the main problem, reuse the same physical parts to work on the subproblem, then continue the suspended computation.

In the subproblem, the contents of the registers will be different than they were in the main problem. (In this case the n register is decremented.) In order to be able to continue the suspended computation, the machine must save the contents of any registers that will be needed after the subproblem is solved so that these can be restored to continue the suspended computation.

Since there is no a _priori_ limit on the depth of nested recursive calls, we may need to save an arbitrary number of register values. These values must be restored in the reverse of the order in which they were saved, since in a nest of recursions the last subproblem to be entered is the first to be finished.

This dictates the use of a stack, or _last in, first out_ data structure, to save register values. We can extend the register-machine language to include a stack by adding two kinds of instructions: Values are placed on the stack using a `save` instruction and restored from the stack using a `restore` instruction. After a sequence of
values has been `saved` on the stack, a sequence of `restores` will retrieve these values in reverse order.

## A Register-Machine Simulator

The simulator is a Scheme program with four interface procedures. The first uses a description of a register machine to construct a model of the machine (a data structure whose parts correspond to the parts of the machine to be simulated), and the other three allow us to simulate the machine by manipulating the model:

- `(make-machine ⟨register-names⟩ ⟨operations⟩ ⟨controller⟩)` constructs and returns a model of the machine with the given registers, operations, and controller.

- `(set-register-contents! ⟨machine-model⟩ ⟨register-name⟩ ⟨value⟩)` stores a value in a simulated register in the given machine.

- `(get-register-contents ⟨machine-model⟩ ⟨register-name⟩)` returns the contents of a simulated register in the given machine.

- `(start ⟨machine-model⟩)` simulates the execution of the given machine, starting from the beginning of the controller sequence and stopping when it reaches the end of the sequence.

### The Machine Model

To build this model, `make-machine` begins by calling the procedure `make-new-machine` to construct the parts of the machine model that are common to all register machines. This basic machine model constructed by `make-new-machine` is essentially a container for _some registers_ and _a stack_, together with _an execution mechanism_ that processes the controller instructions one by one.

`make-machine` then extends this basic model (by sending it messages) to include the registers, operations, and controller of the particular machine being defined. First it allocates a register in the new machine for each of the supplied register names and installs the designated operations in the machine. Then it uses an _assembler_ to transform the controller list into instructions for the new machine and installs these as the machine’s instruction sequence. `make-machine` returns as its value the modified machine model.

**Registers**

Register would be a procedure with local state, which is used to hold a value that can be accessed or changed.

**The stack**

Stack would be a procedure with local state, whose local state consists of a list of the items on the stack.

A stack accepts requests to `push` and item onto the stack, to `pop` the top item off the stack and return it, and to `initialize` the stack to empty.

**The basic machine**

The `make-new-machine` procedure constructs an object whose local state consists of a stack, an initially empty instruction sequence, a list of operations that initially contains an operation to initialize the stack, and a register table that initially contains two registers, named `flag` and `pc` (for _program counter_).

The internal procedure `allocate-register` adds new entries to the register table, and the internal procedure `lookup-register` looks up registers in the table.

The `flag` register is used to control branching in the simulated machine.

The `pc` register determines the sequencing of instructions as the machine runs. In the simulation model, each machine instruction is a data structure that includes a procedure of no arguments, called the `instruction execution procedure`, such that calling this procedure simulates executing the instruction. As the simulation runs, `pc` points to the place in the instruction sequence beginning with the next instruction to be executed. `execute` gets that instruction, executes it by calling the instruction execution procedure, and repeats this cycle until there are no more instructions to execute.

Notice that starting the machine is accomplished by setting `pc` to the beginning of the instruction sequence and calling `execute`.

### The Assembler

The assembler transforms the sequence of controller expressions for a machine into a corresponding list of machine instructions, each with its execution procedure.

To speed up the execution procedure, we could separate the analysis from runtime execution, aka AOT compilation.

Before it can generate the instruction execution procedures, the assembler must know what all the labels refer to, so it begins by scanning the controller texte to separate the labels from the instructions. As it scans the text, it constructs both a list of instructions and a table that associates each label with a pointer into that list. Then the assembler augments the instruction list by inserting the execution procedure for each instruction.

The machine instruction data structure simply pairs the instruction text with the corresponding execution procedure.

### Generating Execution Procedures for Instructions

The assembler calls `make-execution-procedure` to generate the execution procedure for an instruction. This dispatches on the type of instruction to generate the appropriate execution procedure.

For each type of instruction in the register-machine language, there is a generator that builds an appropriate execution procedure. The details of these procedures determine both the syntax and meaning of the individual instructions in the register-machine language.

### Monitoring Machine Performance

Simulation is useful not only for verifying the correctness of a proposed machine design but also for measuring the machine's performance.

To do this, we modify our simulated stack to keep track of the number of times registers are saved on the stack and the maximum depth reached by the stack, and add a message to the stack's interface that prints the statistics.

## Storage Allocation and Garbage Collection

We will assume that our register machines can be equipped with a _list-structured memory_, in which the basic operations for manipulating list-structured data are primitive.

There are two considerations in implementation list structure.

- how to represent the _box-and-pointer_ structure of Lisp pairs, using only the storage and addressing capabilities of typical computer memories

- the management of memory as the computation proceeds, such as storage allocation and recycling.

### Memory as Vectors

A conventional computer memory can be thought of as an array of cubbyholes, each of which can contain a piece of information. Each cubbyhole has a unique name, called its _address_ or _location_.

Typical memory systems provide two primitive operations: one that fetches the data stored in a specified location and one that assigns new data to a specified location. Memory addresses can be incremented to support sequential access to some set of the cubbyholes. More generally, many important data operations require that memory addresses be treated as data, which can be stored in memory locations and manipulated in machine registers. The representation of list structure is one application of such _address arithmetic_.

To model computer memory, we use a new kind of data structure called _vector_.

Abstractly, a vector is a compound data object whose individual elements can be accessed by means of an integer index in an amount of time that is independent of the index.

For computer memory, this access can be implemented through the use of address arithmetic to combine a _base address_ that specifies the beginning location of a vector in memory with an _index_ that specifies the offset of a particular element of the vector.

We alse need a representation for objects and a way to distinguish one kind of data from another. There are many methods of accomplishing this, but they all reduce to using _typed pointers_, that is, to extending the notion of pointer to include information on data type. Two data objects are considered to be the same if their pointers are identical.

### Maintaining the Illusion of Infinite Memory

With a real computer we will eventually run out of free space in which to construct new dat objects. However, most of the data objects generated in a typical computation are used only to hold intermediate results. After these results are accessed, the data objects are no longer needed -- they are _garbage_.

If we can arrange to collect all the garbage periodically, and if this turns out to recycle memory at about the same rate at which we construct new data objects, we will have preserved the illusion that there is an infinite amount of memory.

Garbage collection is based on the observation that, at any moment in procedure execution, the only objects that can affect the future of the computation are those that can be reached by some operations starting from the pointers that are currently in the machine registers. Any memory cell that this not so accessible may be recycled.

The method used here to perform garbage collection is called _stop-and-copy_. When working memory is full, we perform garbage collection by locating all the useful pairs in working memory and copying these into consecutive locations in free memory. Since we do not copy the garbage, there will presumably be additional free memory that we can use to allocate new pairs. In addition, nothing in the working memory is needed, since all the useful pairs in it have been copied. Thus, if we interchange the roles of working memory and free memory, we can continue processing; new pairs will be allocated in the new working memory (which was the old free memory). When this is full, we can copy the useful pairs into the new free memory (which was the old working memory).

## The Explicit-Control Evaluator

> It's much like an interpreter for interpretive language, except the runtime is register-machine, rather than a VM such as V8/JVM.

### The Core of the Explicit-Control Evaluator

### Sequence Evaluation and Tail Recursion

### Conditionals, Assignments, and Definitions

### Running the Evaluator

## Compilation

The explicit-control evaluator machine is universal—it can carry out any computational process that can be described in Scheme.

Thus, the evaluator’s data paths are universal: they are sufficient to perform any computation we desire, given an appropriate controller.

Commercial general-purpose computers are register machines organized around a collection of registers and operations that constitute an efficient and convenient set of data paths. The controller for a general-purpose machine is an interpreter for a register-machine language like the one we have been using. This language is called the _native language_ of the machine, or simply _machine language_. Programs written in machine language are sequences of instructions that use the machine's data paths.

There are two common strategies for bridging the gap between higher-level languages and register-machine languages. The explicit-control evaluator illustrates the strategy of interpretation. An interpreter written in the native language of a machine configures the machine to execute programs written in a language (called the _source language_) that may differ from the native language of the machine performing the evaluation. The primitive procedures of the source language are implemented as a library of subroutines written in the native language of the given machine. A program to be interpreted (called the _source program_) is represented as a data structure. The interpreter traverses this data structure, analyzing the source program. As it does so, it simulates the intended behavior of the source program by calling appropriate primitive subroutines from the library.

The alternative strategy is _compilation_. A compiler for a given source language and machine translates a source program into an equivalent program (called the _object program_) written in the machine’s native language.

Compared with interpretation, compilation can provide a great increase in the efficiency of program execution.

> On the other hand, an interpreter provides a more powerful environment for interactive program development and debugging, because the source program being executed is available at run time to be examined and modified. In addition, because the entire library of primitives is present, new programs can be constructed and added to the system during debugging.

**An overview of the compiler**

> In general, the compiler translates a source program into an object program that performs essentially the same register operations as would the interpreter in evaluating the same source program.

This description suggests a strategy for implementing a rudimentary compiler: We traverse the expression in the same way the interpreter does. When we encounter a register instruction that the interpreter would perform in evaluating the expression, we do not execute the instruction but instead accumulate it into a sequence. The resulting sequence of instructions will be the object code.

> With a compiler, the expression is analyzed only once, when the instruction sequence is generated at compile time.

But there are further opportunities to gain efficiency in compiled code. As the interpreter runs, it follows a process that must be applicable to any expression in the language. In contrast, a given segment of compiled code is meant to execute some particular expression:

    This can make a big difference, for example in the use of the stack to save registers. When the interpreter evaluates an expression, it must be prepared for any contingency. Before evaluating a subexpression, the interpreter saves all registers that will be needed later, because the subexpression might require an arbitrary evaluation. A compiler, on the other hand, can exploit the structure of the particular expression it is processing to generate code that avoids unnecessary stack operations.

A compiler can also optimize access to the environment. Having analyzed the code, the compiler can in many cases know in which frame a particular variable will be located and access that frame directly, rather than performing the `lookup-variable-value` search.

### Structure of the Compiler

In the compiler, we will do essentially the some necessary analysis. Instead of producing execution procedures, however, we will generate _sequences of instructions_ to be run by our register machine.

### Compiling Expressions

### Compiling Combinations

### Combining Instruction Sequences

### An Example of Compiled Code

### Lexical Addressing

### Interfacing Compiled Code to the Evaluator

# Buiding Abstractions with Procedures

## The Elements of Programming

Every powerful language has three mechanisms for accomplishing this:

- **primitive expressions**, which represent the simplest entities the language is concerned with,

- **means of combination**, by which compound elements are built from simpler ones, and

- **means of abstraction**, by which compound elements can be named and manipulated as units.

In programming, we deal with two kinds of elements: procedures and data. Informally, data is _stuff_ that we want to manipulate, and procedures are _descriptions of the rules_ for manipulating the data.

### Expressions

- primitive expression
- primitive procedure expression
- compound expression (combinations)
- nested combinations

REPL : Reading an expression -> Evaluating the expression -> Print the result -> Loop again

### Naming and the Environment

A critical aspect of a programming language is the means it provides for using names to refer to computational objects.

> This is one of the simplest means of abstraction, for it allow programmers to use simple names to refer to the results of compound operations.

It should be clear that the possibility of associating values with symbols and later retrieving them means that the interpreter must maintain some sort of memory that keeps track of the name-object pairs. This memory is called the `environment`.

### Evaluating Combinations

In order to accomplish the evaluation process for a combination we must first perform the evaluation process on each element of the combination. Thus, the evaluation rule is recursive in nature.

Wetake care of the primitive cases by stipulating that

- the values of numerals are the numbers that they name,
- the values of built-in operators are the machine instruction sequences that carry out the corresponding operations, and
- the values of other names are the objects associated with those names in the environment.

The various kinds of expressions (each with its associated evaluation rule) constitute the syntax of the programming language.

### Compound Procedures

The _procedure definitions_, a much more powerful abstraction technique by which a compound operation can be given a name and then referred to as a unit.

> `function` is one of the reified procedures.

The general form of a procedure definition is

```
(define (<name> <formal parameters>) (<body>))
```

The `⟨name⟩` is a symbol to be associated with the procedure definition in the environment. The `<formal parameters>`
the body of the procedure to refer to the corresponding arguments of the procedure. The `<body>` is an expression that will yield the value of the procedure application, optinally, when the formal parameters are replaced by the actual arguments to which the procedure is applied.

Compound procedures are used in exactly the same way as primitive procedures.

### The substitution Model for Procedure Application

To evaluate a combination whose operator names a compound procedure, the interpreter evaluates the elements of the combination and applies the procedure (which is the value of the operator of the combination) to the arguments (which are the values of the operands of the combination).

And, to apply a compound procedure to arguments, evaluate the body of the procedure with each formal parameter replaced by the corresponding argument.

The process we have just described is called the _substitution model_ for procedure application. It can be taken as a model that determines the “meaning” of procedure application, insofar as the procedures in this chapter are concerned. However, there are two points that should be stressed:

- The purpose of the substitution is to help us think about proce- dure application, not to provide a description of how the interpreter really works. Typical interpreters do not evaluate procedure applications by manipulating the text of a procedure to substitute values for the formal parameters. In practice, the “substitution” is accomplished by using a local environment for the formal parameters.

- The substitution model is only the first of these models—a way to get started thinking formally about the evaluation process.

> Applicative Order vs Normal Order
>
> In _applicative order_, the interpreter first evaluates the operator and operands and then applies the resulting procedure to the resulting arguments.
>
> In _normal order_, the interpreter first substitute operand expressions for parameters until it obtained an expression involving only primitive operators, and would then perform the evaluation.

### Conditional Expressions and Predicates

Sometimes, we need to make tests on some procedures, and to perform different operations depending on the result of the tests. Here comes the conditional expression, with its general form:

```
(cond (<p1> <e1>)
      (<p2> <e2>)
      ...
      (<pn> <en>))
```

`(<px> <ex>)` is a pair of _predicate-consequent_ clause. A predicate is an expression whose value is interpreted as either `true` or `false`.

The interpreter would evaluate every predicate until it a predicate is found whose value is true, in which case the interpreter would evalute the corresponding consequent expression of that clause. If none of the predicates is found to be true, no consequent expression would be evaluated.

There's a special form `if`, which is a restricted type of conditional that can be used when there precisely two cases in the case analysis:

```
(if <predicate> <consequent> <alternative>)
```

The interpreter starts by evaluating the predicate part of the expression. If the predicate evaluates to be true, then interpreter would then evaluate the consequent expression, otherwise, the alternative expression.

Also there are logical composition operations which help to construct compound predicates.

```
(and <e1> <e2> ... <en>)

(or <e1> <e2> ... <en>)

(not <e1>)
```

The `and` and `or` would take a short circuit strategy to save the evaluation procedure.

### Procedures as Black-Box Abstractions

It is crucial that each procedure acomplishes an identifiable task that can be used as a module in defining other procedures.

Sometimes, we are not concerned with _how_ the procedure computes its result, only with the fact _what output_ it computes the with our inputs.

So a procedure definition should be able to suppress detail. The users of the procedure may not have written the procedure themselves, but may have obtained it from another programmer as a _black box_. A user should not need to know how the procedure is implemented in order to use it.

#### Local Names

The principle—that the meaning of a procedure should be independent of the parameter names —seems on the surface to be self-evident, but its consequences are profound. The simplest consequence is that the parameter names of a procedure must be _local_ to the body of the procedure.

A formal parameter of a procedure has a very special role in the procedure definition, in that it doesn’t matter what name the formal parameter has. Such a name is called a _bound variable_, and we say that the procedure definition binds its formal parameters. The meaning of a procedure definition is unchanged if a bound variable is consistently renamed throughout the definition. If a variable is not bound, we say that it is free. The set of expressions for which a binding defines a name is called the _scope_ of that name. In a procedure definition, the bound variables declared as the formal parameters of the procedure have the body of the procedure as their scope.

#### Internal definitions and block structure

The formal parameters of a procedure are local to the body of the procedure.

In order to solve the name confliction among procedures, we might want to localize the subprocedures, hiding them inside some bigger procedures.

To make this possible, a procedure is allowed to have private internal definitions that are local to that procedure. Such nesting of definitions, called _block structure_, is basically the right
solution to the simplest name-packaging problem.

With interal definition of procedures, we could release some parameters of the internal procedures to be _free_ rather than _bound_. This discipline is called _lexcial scoping_.

> Lexical scoping dictates that free variables in a procedure are taken to refer to bindings made by enclosing procedure definitions; that is, they are looked up in the environment in which the procedure was defined.

## Procedures and the Processes They Generate

> To become experts, we must learn to visualize the processes generated by various types of procedures.

A procedure is a pattern for the _local evolution_ of a computational process. It specifies how each stage of the process is built upon the previous stage. We would like to be able to make statements about the overall, or _global_, behavior of a process whose local evolution has been specified by a procedure.

### Linear Recursion and Iteration

Carrying out _recursive process_ requires that the interpreter keep track of the operations to be performed later on. If the amount of information needed to keep track of it, which grows linearly with _n_ (is proportional to _n_), just like the number of steps, is called a _linear recursive process_.

By contrast, in an _iterative process_, all we need to keep track of, for any _n_, are the current values of the variables _product_, _counter_, and _max-count_. In general, an iterative process is one whose state can be summarized by a fixed number of state variables, together with a fixed rule that describes how the state variables should be updated as the process moves from state to state and an (optional) end test that specifies conditions under which the process should terminate.

The contrast between the two processes can be seen in another way. In the iterative case, the program variables provide a complete description of the state of the process at any point. If we stopped the computation between steps, all we would need to do to resume the computation is to supply the interpreter with the values of the three program variables. Not so with the recursive process. In this case there is some additional _hidden_ information, _maintained by the interpreter and not contained in the program variables_, which indicates where the process is in negotiating the chain of deferred operations. The longer the chain, the more information must be maintained.

As a consequence, some programming languages can describe iterative processes only by resorting to special-purpose _looping constructs_ such as `do`, `repeat`, `until`, `for`, and `while`.

With a tail-recursive implementation, iteration can be expressed using the ordinary procedure call mechanism, so that special iteration constructs are useful only as syntactic sugar.

### Tree Recursion

Another common pattern of computation is called _tree recursion_. Consider computing the sequence of Fibonacci numbers as an example, `f(n) = f(n-1) + f(n-2)`.

This procedure is instructive as a prototypical tree recursion, but it is a terrible way to compute Fibonacci numbers because it does so much redundant computation.

In general, the number of steps required by a tree-recursive process will be proportional to the number of nodes in the tree, while the space required will be proportional to the maximum depth of the tree.

When we consider processes that operate on hierarchically structured data rather than numbers, we will find that tree recursion is a natural and powerful tool.

### Orders of Growth

The notion of _order of growth_ is to obtain a gross measure of the resources required by a process as the inputs become larger.

Orders of growth provide only a crude description of the behavior of a process. On the other hand, order of growth provides a useful indication of how we may expect the behavior of the process to change as we change the size of the problem.

### Exponentiation

The difference between `Θ(log n)` growth and `Θ(n)` growth becomes striking as `n` becomes large.

One of the means to reduce execution steps or memory requirements is try to make a problem in `n` to be divided into `log n` executions.

## Formulating Abstractions with Higher-Order Procedures

One of the things we should demand from a powerful programming language is the ability to build abstractions by assigning names to common patterns and then to work in terms of the abstractions directly.

Procedures provide this ability. This is why all but the most primitive programming languages include mechanisms for defining procedures.

Programming languages allow us to construct procedures that can accept procedures as arguments or return procedures as values.

Procedures that manipulate procedures are called _higher-order procedures_.

### Constructing Procedures Using Lambda

It seems terribly awkward to have to define trivial procedures outside the higher-order procedures. It would be more convenient to have a way to directly specify special form _lambda_ which creates inline procedures.

In general, _lambda_ is used to create procedures in the same way as _define_, except that no name is specified for the procedure, and there's not an associated name in the environment.

#### Using `let` to create local variables

Another use of _lambda_ is in creating local variables. We often need local variables in our procedures other than those that have been bound as formal parameters.

The general form of a let expression is

```
(let ((⟨var1⟩ ⟨exp1⟩) (⟨var2⟩ ⟨exp2⟩)
...
(⟨varn⟩ ⟨expn⟩)) ⟨body⟩)
```

which can be thought of as saying

```
let ⟨var1⟩ have the value ⟨exp1⟩ and
    ⟨var2⟩ have the value ⟨exp2⟩ and
    ...
    ⟨varn⟩ have the value ⟨expn⟩
in ⟨body⟩
```

1. `let` allows one to bind variables as locally as possible to where they are to be used.

1. The variables' values are computed outside the `let`. This matters when the expressions that provide the values for the local variables depend upon variables having the same names as the local variables themselves.

### Procedures as General Methods

Compound procedures is introduced as a mechanism for abstracting patterns of operations so as to make them independent of the particular inputs involved. With higher-order procedures, there's a more powerful kind of abstraction: procedures used to express general methods of computation, independent of the particular functions involved.

Pass procedures as arguments significantly enhances the expressive power of our programming language.

### Procedures as Returned Values

We can achieve even more expressive power by creating procedures whose returned values are themselves procedures

#### Abstractions and first-class procedures

Compound procedures are a crucial abstraction mechanism, because they permit us to express general methods of computing as explicit elements in our programming language. Higher-order procedures permit us to manipulate these general methods to create further abstractions.

In general, programming languages impose restrictions on the ways in which computational elements can be manipulated. Elements with the fewest restrictions are said to have first-class status. Some of the _rights and privileges_ of first-class elements are:

- They may be named by variables.
- They may be passed as arguments to procedures.
- They may be returned as the results of procedures.
- They may be included in data structures.

Allowing procedures full first-class status poses challenges for efficient implementation, by the interpreter, but bring us enormous gain in expressive power.

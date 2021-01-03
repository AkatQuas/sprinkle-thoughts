# Metalinguistic Abstraction

There is one general techniques for expert programmers to control the complexity of sophisticated systems:

- Combining primitive elements to form compound objects

- Abstracting compound objects to form higher-level building blocks,

- Perserving modularity by adopting appropriate large-scale views of system structure.

_Metalinguistic abstraction_ - establishing new languages - plays an important role in all branches of engineering design.

In programming not only can programmers formulate new languages, but they can also implement these languages by constructing evaluators.

> An _evaluator_ (or _interpreter_) for a programming language is a procedure that, when applied to an expression of the language, performs the actions required to evaluate that expression.

It is no exaggeration to regard this as the most fundamental idea in programming:

    The evaluator, which determines the meaning of expressions in a programming language, is just another program.

## The Metacircular Evaluator

Evaluation is a process, so it is appropriate to describe teh evaluation process using Lisp, which is the tool for describing processes.

> An evaluator that is written in the same language that it evaluates is said to be _metacircular_.

The metacircular evaluator is essentially a Scheme formulation of the environment model of evaluation. The model has two basic parts:

1. To evaluate a combination (a compound expression other than a special form), evaluate the subexpressions and then apply the value of the operator subexpression to the values of the operand subexpressions.

1. To apply a compound procedure to a set of arguments, evaluate the body of the procedure in a new environment. To construct this environment, extend the environment part of the procedure object by a frame in which the formal parameters of the procedure are bound to the arguments to which the procedure is applied.

These two rules describe the essence of the evaluation process, a basic cycle in which expressions to be evaluated in environments are reduced to procedures to be applied to arguments, which in turn are reduced to new expressions to be evaluated in new environments, and so on, until we get down to symbols, whose values are looked up in the environment, and to primitive procedures, which are applied directly.

This evaluation cycle will be embodied by the interplay between the two critical procedures in the evaluator, `eval` and `apply`.

The implementation of the evaluator will depend upon procedures that define the _syntax_ of the expressions to be evaluated.

### The Core of the Evaluator

#### Eval

`eval` takes as arguments an expression and an environment. It classifies the expression and directs its evaluation. `eval` is structured as a case analysis of the syntactic type of the expression to be evaluated.

In order to keep the procedure general, we express the determination of the type of an expression abstractly, making no commitment to any particular representation for the various types of expressions. Each type of expression has a predicate that tests for it and an abstract means for selecting its parts. This _abstract syntax_ makes it easy to see how we can change the syntax of the language by using the same evaluator, but with a different collection of syntax procedures.

**Primitive expressions**

- For self-evaluating expressions, such as numbers, `eval` returns the expression itself.

- `eval` must look up variables in the environment to find their values.

**Special forms**

- For quoted expressions, `eval` returns the expression that was quoted.

- An assignment to (or a definition of) a variable must recursively call `eval` to compute the new value to be associated with the variable. The environment must be modified to change (or create) the binding of the variable.

- An `if` expression requires special processing of its parts, so as to evaluate the consequent if the predicate is true, and otherwise to evaluate the alternative.

- A `lambda` expression must be transformed into an applicable procedure by packaging together teh parameters and body specified by the `lambda` expression with the environment of the evaluation.

- A `begin` expression requires evaluating its sequence of expressions in the order in which they appear.

- A case analysis (`cond`) is transformed into a nest of `if` expressions and then evaluated.

**Combinations**

- For a procedure application, `eval` must evaluate the operator part and the operands of the combination. The resulting procedure and arguments are passed to `apply`, which handles the actual procedure application.

> In most Lisp implementations, dispatching on the type of an expression is done in a _data-directed style_. This allows a programmer to add new types of expressions that `eval` can distinguish, without modifying the definition of `eval` itself.

#### Apply

`apply` takes two arguments, a procedure and a list of arguments to which the procedure should be applied.

`apply` classifies procedures into two kinds:

- It calls `apply-primitive-procedure` to apply primitives;

- It applies compound procedures by sequentially evaluating the expressions that make up the body of the procedure.

The environment for the evaluation of the body of a compound procedure is constructed by extending the base environment carried by the procedure to include a frame that binds the parameters of the procedure to the arguments to which the procedure is to be applied.

**Procedure arguments**

When `eval` processes a procedure application, it uses some helper procedure to produce the list of arguments to which the procedure is to be applied.

**Conditionals**

First, we would use some helper procedure to evaluate the predicate part of an `if` expression in the given environment. If the result is true, we continue to evaluate the consequent, otherwise it evaluates the alternative:

**Sequences**

This is used by `apply` to evaluate the sequence of expressions in a procedure body and by `eval` te evaluate the sequence of expressions is a `begin` expression. It takes as arguments a sequence of expressions and an environment, and evaluates the expressions in the order in which they occur. The value returned is the value of the final expression.

> The sequence-evaluation is done by recursivly: evaluate the first expression and use the sequence-evaluation on the rest part of the expressions, until we reached the final expression.

**Assignments and definitions**

This procedure calls `eval` to find the value to be assigned and transmits the variable and the resulting value to set to variable, which is installed in the designated environment.

### Evaluator Data Structures

In addition to defining the external syntax of expressions, the evaluator implementation must also define the data structures that the evaluator manipulates internally, as part of the execution of a program, such as the representation of procedures and environments and the representation of true and false.

We use data abstraction to isolate the rest of the evaluator from the detailed choice of representation, we could change the environment representation as we wish. In a production-quality Lisp system, the speed of the evaluator's environment operations, especially that of variable lookup, has a major impact on the performance of the system.

### Running the Evaluator as a Program

Givent the evaluator, we have a description of the process by which Lisp expressions are evaluated. One advantage of expressing the evaluator as a program is that we can run the program. This gives us, running within Lisp, a working model of how Lisp itself evaluates expressions. This can serve as a framework for experimenting with evaluation rules.

The evaluator program reduces expressions ultimately to the application of primitive procedures. Therefore, all that we need to run the evaluator is to create a mechanism that calls on the underlying Lisp system to model the application of primitive procedures.

There must be a binding for each primitive procedure name, so that when `eval` evaluates the operator of an application of a primitive, it will find an object to pass to `apply`. We thus set up a global environment that associates unique objects with the names of the primitive procedures that can appear in the expressions we will be evaluating. The global environment also includes bindings for the symbols `true` and `false`, so they can be used as variables in expressions to be evaluated.

For convenience in running the metacircular evaluator, we provide a _driver loop_ that models the read-eval-print loop of the underlying Lisp system. It prints a prompt, reads an input expression, evaluates this expression in the global environment, and prints the result. We precede each printed result by an output prompt so as to distinguish the value of the expression from other output that may be printed.

### Data as Programs

One operational view of the meaning of a program is that a program is a description of an abstract machine.

In a similar way, we can regard the evaluator as a very special machine that takes as input _a description of a machine_. Given this input, the evaluator configures itself to emulate the machine described.

From this perspective, the evaluator is seen to be a _universal machine_. It mimics other machines when these are described as Lisp programs.

One striking aspect of the evaluator is that it acts as a bridge between the data objects that are manipulated by our programming language and the programming language itself.

> Imagine that the evaluator program is running, and that a user is typing expressions to the evaluator and observing the results. From the perspective of the user, an input expression such as `(* x x)` is an expression in the programming language, which the evaluator should execute. From the perspective of the evaluator, however, the expression is simply a list (in this case, _a list of three symbols: `*`, `x`, and `x`_) that is to be manipulated according to a well-defined set of rules.

### Internal Definitions

The environment model of evaluation and our metacircular evaluator execute definitions in sequence, extending the environment frame one definition at a time.

However, when it comes to the internal definitions used to implement block structure, no calls to these procedures will be evaluated until all of them have been defined. In fact, the sequential evaluation mechanism will give the same result as a mechanism that directly implements simultaneous definition for any procedure in which the internal definitions come first in a body and evaluation of the value expressions for teh defined variables doesn't actually use any of the defined variables.

There is a simple way to treat definitions so that internally defined names have truly simultaneous scope--just create all local variables that will be in the current environment before evaluating any of the value expressions. One way to do this is by a syntax transformation on `lambda` expressions. Before evaluating the body of a `lambda` expression, we _scan out_ and eliminate all the internal definitions in the body. The internally defined variables will be created with a declaration and then set to their values by assignment.

```
(lambda ⟨vars⟩
  (define u ⟨e1⟩)
  (define v ⟨e2⟩)
  ⟨e3⟩)

# would be transformed into
(lambda ⟨vars⟩
  (let ((u '*unassigned*)
        (v '*unassigned*))
        (set! u ⟨e1⟩)
        (set! v ⟨e2⟩)
        ⟨e3⟩))
```

### Separating Syntactic Analysis from Execution

The evaluator implemented above is simple, but it is very inefficient, because the syntactic analysis of expressions is interleaved with their execution. Thus if a program is executed many times, its syntax is analyzed many times.

We can transform the evaluator to be significantly more efficient by arranging things so that syntactic analysis is performed only once.

We split `eval`, which takes an expression and an environment, into two parts.

1. The procedure `analyze` takes only the expression. It performs the syntactic analysis and returns a new procedure, the _execution procedure_, that encapsulates the work to be done in executing the analyzed expression.

1. The `execution` procedure takes an environment as its argument and completes the evaluation. Looking up a variable value must still be done in the execution phase, since this depends upon knowing the environment.

This saves work because `analyze` will be called only once on an expression, while the execution procedure may be called many times.

## Variations on a Scheme--Lazy Evaluation

Indeed, new languages are often invented by first writing an evaluator that embeds the new language within an existing high-level language.

For example, if we wish to discuss some aspect of a proposed modification to Lisp with another member of the Lisp community, we can supply an evaluator that embodies the change. The recipient can then experiment with the new evaluator and send back comments as further modifications.

Not only does the high-level implementation base make it easier to test and debug the evaluator; in addition, the embedding enables the designer to snarf features from the underlying language, just as our embedded Lisp evaluator uses primitives and control structure from the underlying Lisp. Only later (if ever) need the designer go to the trouble of building a complete implementation in a low-level language or in hardware.

### Normal Order and Applicative Order

**applicative-order language**: all the arguments to procedures are evaluated when the procedure is applied.

**normal-order language**: delay the evaluation of procedure arguments until the actual argument values are needed.

Delaying evaluation of procedure arguments until the last possible moment is called _lazy evaluation_. An advantage of lazy evaluation is that some procedures can do useful computation even if evaluation of some of their arguments would produce errors or would not terminate.

If the body of a procedure is entered before an argument has been evaluated we say that the procedure is _non-strict_ in that argument. If the argument is evaluated before the body of the procedure is entered we say that the procedure is _strict_ in that argument.

> The _strict_ versus _non-strict_ terminology means essentially the same thing as _applicative-order_ versus _normal-order_, except that it refers to individual procedures and arguments rather than to the language as a whole.

In a purely applicative- order language, all procedures are strict in each argument. In a purely normal-order language, all compound procedures are non-strict in each argument, and primitive procedures may be either strict or non-strict. There are also languages that give programmers detailed control over the strictness of the procedures they define.

> One can do useful computation, combining elements to form data structures and operating on the resulting data structures, even if the values of the elements are not known. It makes perfect sense, for instance, to compute the length of a list without knowing the values of the individual elements in the list.

### An Interpreter with Lazy Evaluation

The basic idea is that, when applying a procedure, the interpreter must determine which arguments are to be evaluated and which are to be delayed. The delayed arguments are not evaluated; instead, they are transformed into objects called _thunks_.

> The word thunk was invented by an informal working group that was discussing the implementation of call-by-name in Algol 60. They observed that most of the analysis of (“thinking about”) the expression could be done at compile time; thus, at run time, the expression would already have been “thunk” about.

The thunk must contain the information required to produce the value of the argument when it is needed, as if it had been evaluated at the time of the application. Thus, the thunk must contain the argument expression and the environment in which the procedure appliaction is being evaluated. We need to pass the environment to construct thunks when the arguments are to be delayed.

The process of evaluating the expression in a thunk is called _forcing_. In general, a thunk will beforce only when its value is needed:

- when it is passed to a primitive procedure that will use the value of the thunk;
- when it is the value of a predicate of a conditional;
- when it is the value of an operator that is about to be applied as a procedure;

One design choice we have available is whether or not to memoize thunks: with memoization, the first time a thunk is forced, it stores the value that is computed. Subsequent forcings simply return the stored value without repeating the computation. This design is more efficient in most applications.

**Representing thunks**

The evaluator must arrange to create thunks when procedures are applied to arguments and to force these thunks later.

A thunk must package an expression together with the environment, so that the argument can be produced later.

To force the thunk, we simply extract the expression and environment from the thunk and evaluate the expression in the environment.

Considering memoization: When a thunk is forced, we will turn it into an evaluated thunk by replacing the stored expression with its value and changing the thunk tag so that it can be recognized as already evaluated.

### Streams as Lazy Lists

With lazy evaluation, streams and lists can be identical.

> Lists could be view as iterators with already evaluated objects, whilst streams the generators with thunk to be evaluated in the future.

## Variations on a Scheme--Nondeterministic Computing

_Nondeterministic computing_ is a programming paradigm, and is useful for _generate and test_ applications.

The key idea is that expressions in a nondeterministic language can have more than one possible value. The nondeterministic program evaluator will work by automatically choosing a possible value and keeping track of the choice. If a subsequent requirement is not met, the evaluator will try a different choice, and it will keep trying new choices until the evaluation succeeds, or until we run out of choices.

Just as the lazy evaluator frees the programmer from the details of how values are delayed and forced, the nondeterministic program evaluator will free the programmer from the details of how choices are made.

It is instructive to contrast the different images of time evoked by nondeterministic evaluation and stream processing:

- Stream processing uses lazy evaluation to decouple the time when the stream of possible answers is assembled from the time when the actual stream elements are produced. The evaluator supports the illusion that all the possible answers are laid out before us in a timeless sequence.

- With nondeterministic evaluation, an expression represents the exploration of a set of possible worlds, each determined by a set of choices. Some of the possible worlds lead to dead ends, while others have useful values. The nondeterministic program evaluator supports the illusion that time branches, and that our programs have different possible execution histories. When we reach a dead end, we can revisit a previous choice point and proceed along a different branch.

### Amb and Search

To support nondeterminism, a special form called `amb` is introduced.

```
(amb <e1> <e2> ... <en>)
```

The expression returns the value of one of the n expressions _ambiguously_.

`amb` with a single choice produces an ordinary (single) value.

`amb` with no choices is an expression with no acceptable values. It's the responsibility of the designer to decide whether `amb` should fail on no acceptable values.

Abstractly, we can imagine that evaluating an `amb` expression causes time to split into branches, where the computation continues on each branch with one of the possible values of the expression. We say that `amb` represents a _nondeterministic choice point_.

If we had a machine with a sufficient number of processors that could be dynamically allocated, we could implement the search in a straightforward way. Execution would proceed as in a sequential machine, until an `amb` expression is encountered. At this point, more processors would be allocated and initialized to continue all of the parallel executions implied by the choice. Each processor would proceed sequentially as if it were the only choice, until it either terminates by encountering a failure, or it further subdivides, or it finishes.

If we have a machine with only one processor, or a few concurrent processes, we must consider the alternatives sequentially. One could imagine modifying an evaluator to pick at random a branch to follow whenever it encounters a choice point. Random choice, however, can easily lead to failing values. We might try running the evaluator over and over, making random choices and hoping to find a non-failing value, but it is better to _systematically search_ all possible execution paths.

The `amb` evaluator could be implemented with a systematic search as follows:

When the evaluator encounters an application of `amb`, it initially selects the first alternative. This selection may itself lead to a further choice. The evaluator will always initially choose the first alternative at each choice point. If a choice results in a failure, then the evaluator automagically _backtracks_ to the most recent choice point and tries the next alternative. If it runs out of alternatives at any choice point, the evaluator will back up to the previous choice point and resume from there. This process leads to a search strategy known as _depth-first search_ or _chronological backtracking_.

The advantage of nondeterministic programming is that we can suppress the details of how search is carried out, thereby expressing the programs at a higher level of abstraction.

### Implementing the Amb Evaluator

The execution procedures in the `amb` evaluator take three arguments: the environment, and two procedures called _continuation procedures_. The evaluation of an expression will finish by calling one of these two continuations: If the evaluation results in a value, the _success continuation_ is called with that value; if the evaluation results in the discovery of a dead end, the _failure continuation_ is called. Constructing and calling appropriate continuations is the mechanism by which the nondeterministic evaluator implements backtracking.

It is the job of the success continuation to receive a value and proceed with the computation. Along with that value, the success continuation is passed another failure continuation, which is to be called subsequently if the use of that value leads to a dead end.

It is the job of the failure continuation to try another branch of the nondeterministic process. The essence of the nondeterministic language is in the fact that expressions may represent choices among alternatives. The evaluation of such an expression must proceed with one of the indicated alternative choices, even though it is not known in advance which choices will lead to acceptable results. To deal with this, the evaluator picks one of the alternatives and passes this value to the success continuation. Together with this value, the evaluator constructs and passes along a failure continuation that can be called later to choose a different alternative.

In addition, if a side-effect operation (such as assignment to a variable) occurs on a branch of the process resulting from a choice, it may be necessary, when the process finds a dead end, to undo the side effect before making a new choice. This is accomplished by having the side-effect operation produce a failure continuation that undoes the side effect and propagates the failure.

In summary, failure continuations are constructed by:

- `amb` expressions: to provide a mechanism to make alternative choices if the current choice made by the `amb` expression leads to a dead end;
- the top-level driver: to provide a mechanism to report failure when the choices are exhausted;
- assignments: to intercept failures and undo assignments during backtracking

Failure continuations are also called during processing of a failure:

- When the failure continuation created by an assignment finishes undoing a side effect, it calls the failure continuation it intercepted, in order to propagate the failure back to the choice point that led to this assignment or to the top level.
- When the failure continuation for an `amb` runs out of choices, it calls the failure continuation that was originally given to the `amb`, in order to propagate the failure back to the previous choice point or to the top level.

## Logic Programming

Indeed, programming languages require that the programmer express knowledge in a form that indicates the step-by-step methods for solving particular problems. On the other hand, high-level languages prodive, as part of the language implementation, a substantial amount of methodological knowledge that frees the user from concern with numerous details of how a specified computation will progress.

Most programming languages are organized around computing the values of mathematical functions. Expression-oriented languages (such as Lisp, Fortran, and Algol) capitalize on the _pun_ that an expression that describes the value of a function may also be interpreted as a means of computing that value. Because of this, most programming languages are strongly biased toward unidirectional computations (computations with well-defined inputs and outputs).

In a constraint system the direction and the order of computation are not so well specified; in carrying out a computation the system must therefore provide more detailed _how to_ knowledge than would be the case with an ordinary arithmetic computation. This does not mean that the user is released altogether from the the responsibility of providing imperative knowledge. There are many constraint networks that implement the same set of constraints, and the user must choose from the set of mathematically equivalent networks a suitable network to specify a particular computation.

The nondeterministic program evaluator also moves away from the view that programming is about constructing algorithms for computing unidirectional functions. In a nondeterministic language, expressions can have more than one value, and, as a result, the computation is dealing with relations rather than with single-valued functions. Logic programming extends this idea by combining a relational vision of programming with a powerful kind of symbolic pattern matching called **unification**.

This approach, when it works, can be very powerful way to write programs. Part of the power comes from the fact that a single _what is_ fact can be used to solve a number of different problems that would have different _how to_ components.

Contemporary logic programming languages have substantial deficiencies, in that their general _how to_ methods can lead them into spurious infinite loops or other undesirable behavior.

### Deductive Information Retrieval

Logic programming excels in providing interfaces, something such as pattern-directed access, to data bases for information retrieval.

> Something like Regular Expression which also use a pattern to test strings.

### How the Query System Works

The query system is organized around two central operations called pattern matching and unification.

**Pattern matching**

A _pattern matcher_ is a program that tests whether some datum fits a specific pattern. The pattern matcher used by the query system takes as inputs a pattern, a datum, and a frame that specifies bindings for various pattern variables. It checks whether the datum matches the pattern in a way that is consistent with the bindings already in the frame. If so, it returns the given frame augmented by any bindings that may have been determined by the match. Otherwise, it indicates that the match has failed.

**Streams of frames**

The testing of patterns against frames is organized through the use of streams. Given a single frame, the matching process runs through the data-base entries one by one. For each data-base entry, the matcher generates either a special symbol indicating that the match has failed or an extension to the frame. The results for all the data-base entries are collected into a stream, which is passed through a filter to weed out the failures. The result is a stream of all the frames that extend the given frame via a match to some assertion in the data base .

**Unification**

In order to handle rules in the query language, we must be able to find the rules whose conclusions match a given query pattern. Rule conclusions are like assertions except that they can contain variables, so we will need a generalization of pattern matching-—called _unification_——in which both the _pattern_ and the _datum_ may contain variables.

A unifier takes two patterns, each containing constants and variables, and determines whether it is possible to assign values to the variables that will make the two patterns equal. With complex patterns, performing unification may seem
to require deduction. We may think of this process as solving a set of equations among the pattern components. In general, these are simultaneous equations, which may require substantial manipulation to solve.

In a successful pattern match, all pattern variables become bound, and the values to which they are bound contain only constants. This is also true of all the examples of unification we have seen so far. In general, however, a successful unification may not completely determine the variable values; some variables may remain unbound and others may be bound to values that contain variables.

**Applying rules**

Unification is the key to the component of the query system that makes inferences from rules.

In general, the query evaluator uses the following method to apply a rule when trying to establish a query pattern in a frame that specifies bindings for some of the pattern variables:

- Unify the query with the conclusion of the rule to form, if successful, an extension of the original frame.
- Relative to the extended frame, evaluate the query formed by the body of the rule.

> Just as procedure definitions are the means of abstraction in Lisp, rule definitions are the means of abstraction in the query language. In each case, we unwind the abstraction by creating appropriate bindings and evaluating the rule or procedure body relative to these.

### Is Logic Programming Mathematical Logic?

The identification of the query language with mathematical logic is not really valid, though, because the query language provides a _control structuer_ that interprets the logical statments procedurally.

The aim of logic programming is to provide the programmer with techniques for decomposing a computational problem into two separate problems: _what is to be computed_, and _how this should be computed_.

This is accomplished by selecting a subset of the statements of mathematical logic that is powerful enough to be able to describe anything one might want to compute, yet weak enough to have a controllable procedural interpretation.

The intention here is that, on the one hand, a program specified in a logic programming language should be an effective program that can be carried out by a computer. Control (_how to compute_) is effected by using the order of evaluation of the language. We should be able to arrange the order of clauses and the order of subgoals within each clause so that the computation is done in an order deemed to be effective and efficient. At the same time, we should be able to view the result of the computation (_what to compute_) as a simple consequence of the laws of logic.

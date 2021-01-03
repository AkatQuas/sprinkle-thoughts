# Modularity, Objects, and State

Effective program synthesis also requires organizational principles that can guide us in formulating the overall design of a program.

In particular, we need strategies to help us structure large systems so that they will be _modular_, that is, so that they can be divided _naturally_ into coherent parts that can be separately developed and maintained.

There are two prominent organizational strategies arising from two rather different _world views_ of the structure of systems:

- object-based: viewing a large system as a collection of distinct objects whose behaviors may change over time.

- stream-processing: concentrating on the streams of information that flow in the system.

## Assignment and Local State

An object is said to _have state_ if its behavior is influenced by its history.

We can characterize an object's state by one or more _state variables_, which among them maintain enough information about history to determine the object's current behavior.

In a system composed of many objects, the objects are rarely completely independent. Each may influence the states of others through interactions, which serve to couple the state variables of one object to those of other objects.

> Indeed, the view that a system is composed of separate objects is most useful when the state variables of the system can be grouped into _closely coupled_ subsystems that are only _loosely coupled_ to other subsystems.

For such a model to be modular, it should be decomposed into computational objects that model the actual objects in the system. Each computational object must have its own _local state variables_ describing the actual object’s state. Since the states of objects in the system being modeled change over time, the state variables of the corresponding computational objects must also change. In particular, if we wish to model state variables by ordinary symbolic names in the programming language, then the language must provide an _assignment operator_ to enable us to change the value associated with a name.

### Local State Variables

This uses the `set!` special form, whose syntax is

```
(set! <name> <new-value>)
```

Here `<name>` is a symbol and `<new-value>` is any expression. `set!` changes `<name>` so that its value is the result obtained by evaluating `<new-value>`.

In general, evaluating the expression:

```
(begin <exp1> <exp2> ... <expk>)
```

causes the expressions `⟨exp1⟩` through `⟨expk⟩` to be evaluated in sequence and the value of the final expression `⟨expk⟩` to be returned as the value ofe the entire `begin` form.

Combining `set!` with local variable is the general programming technique for constructing computational objects with local state. (It's a closure.)

### The Benefits of Introducing Assignment

Viewing systems as collections of objects with _local state_ is a powerful technique for maintaining a _modular design_.

From the point of view of one part of a complex process, the other parts appear to change with time. They have hidden time-varying local state. If we wish to write computer programs whose structure reflects this decomposition, we make computational objects whose behavior changes with time. We model state with local state variables, and we model the changes of state with assignments to those variables.

By introducing assignment and the technique of hiding state in local variables, we are able to structure systems in a more modular fashion than if all state had to be manipulated explicitly, by passing additional parameters.

### The Costs of Introducing Assignment

The advantage of local state assignment when modeling objects coms with a price: the program can no longer be interpreted in terms of the substitution model of procedure application. Moreover, no simple model with _nice_ mathematical properties can be an adequate framework for dealing with objects and assignment in programming languages.

> Programming without any use of assignments, as we did throughout the first two chapters of this book, is accordingly known as _functional programming_. (No more assignment after first assignment?)

Without assignment, that symbols in the language could be viewed as essentially names for values.

But if the value of a variable can change, a variable can no longer be simply a symbol. Now a variable somehow refers to a place where a value can be stored, and the value stored at this place can change.

#### Sameness and change

In general, we can determine that two apparently identical objects are indeed _the same one_ only by modifying one object and then observing whether the other object has changed in the same way.

In general, so long as we never modify data objects, we can regard a compound data object to be precisely the totality of its pieces. But this view is no longer valid in the presence of change, where a compound data object has an _identity_ that is something different from the pieces of which it is composed. An object is still _the same_ object even if some of its pieces change; conversely, two different objects can both exist with the same state imformation.

#### Pitfalls of imperative programming

In contrast to _functional programming_, programming that makes extensive use of assignment is known as _imperative programming_.

In general, programming with assignment forces us to carefully consider the relative orders of the assignments to make sure that each statement is using the correct version of the variables that have been changed.

## The Environment Model of Evaluation

In our new model of evaluation, these places in which values can be stored will be maintained in structures called _environments_.

An environment is a sequence of _frames_. Each frame is a table (possibly empty) of bindings, which associate _variable names_ with their corresponding _values_. (A single frame may contain at most one binding for any variable, no more than two variables use same name). Each frame also has a pointer to its enclosing environment, unless the frame is considered to be _global_. The _value of a variable_ with respect to an environment is the value given by the binding of the variable in the first frame in the environment that contains a binding for that variable. If no frame in the sequence specifies a binding for the variable, then the variable is said to be _unbound_ in the environment.

The environment is crucial to the evaluation process, because it determines the _context_ in which an expression should be evaluated.

Rather, an expression acquires a meaning only with respect to some environment in which it is evaluated. Thus, in our model of evaluation we will always speak of evaluating an expression with respect to some environment.

### The Rules for Evaluation

In the environment model of evaluation, a procedure is always a pair consisting of some code and a pointer to an environment.

Procedures are created in one way only: by evaluating a λ-expression. This produces a procedure whose code is obtained from the text of the λ-expression and whose environment is the environment in which the λ-expression was evaluated to produce the procedure.

```
(define (square x)
  (* x x))

// syntax sugar, equivalent λ-expression
(define square
  (lambda (x) (* x x)))
```

The environment model of procedure application can be summarized by two rules:

1. A procedure object is applied to a set of arguments by constructing a frame, binding the formal parameters of the procedure to the arguments of the call, and then evaluating the body of the procedure in the context of the new environment constructed. The new frame has as its enclosing environment the environment part of the procedure object being applied.

1. A procedure is created by evaluating a λ-expression relative to a given environment. The resulting procedure object is a pair consisting of the text of the λ-expression and a pointer to the environment in which the procedure was created.

### Internal Definitions

Environment model helps us to understanding internal definitions inside procedures.

The environment model thus explains the two key properties that make local procedure definitions a useful technique for modularizing programs:

1. The names of the local procedures do not interfere with names external to the enclosing procedure, because the local procedure names will be bound in the frame that the procedure creates when it is run, rather than being bound in the global environment.

1. The local procedures can access the arguments of the enclosing procedure, simply by using parameter names as free variables. This is because the body of the local procedure is evaluated in an environment that is subordinate to the evaluation environment for the enclosing procedure.

## Modeling with Mutable Data

In order to model compound objects with changing state, we will design data abstractions to include, in addition to selectors and constructors, operations called _mutators_, which modify data objects.

Data objects for which mutators are definded are known as _mutable data_.

### Sharing and identity

One object might be _shared_ among different data objects.

Sharing can be dangerous since modifications made to structures will also affect other structures that happen to share the modified parts.

> Data consistency, Mutation Exclusive,

### Mutation is just assignment

We can implement mutable data objects as procedures using assignment and local state.

### Representing Queues

A _queue_ is a sequence in which items are inserted at one end (called the _rear_ of the queue) and deleted from the other end (the _front_).

Because items are always removed in the order in which they are inserted, a queue is sometimes called a _FIFO (first in, first out)_ buffer.

In terms of data abstraction, we can regard a queue as defined by the following set of operations:

- a constructor: (make-queue) returns an empty queue (a queue containing no items).

- two selectors:

  - `(empty-queue? ⟨queue⟩)` tests if the queue is empty.
  - `(front-queue ⟨queue⟩)` returns the object at the front of the queue, signaling an error if the queue is empty; it does not modify the queue.

- two mutators:

  - `(insert-queue! ⟨queue⟩ ⟨item⟩)` inserts the item at the rear of the queue and returns the modified queue as its value.

  - `(delete-queue! ⟨queue⟩)` removes the item at the front of the queue and returns the modified queue as its value, signaling an error if the queue is empty before the deletion.

### Propagation of Constraints

> It's a big issue in computer science.

Computer programs are traditionally organized as one-directional computations, which perform operations on prespecified arguments to produce desired outputs.

On the other hand, we often model systems in terms of relations among quantities.

We sketch the design of a language that enables us to work in terms of relations themselves. The primitive elements of the language are _primitive constraints_, which state that certain relations hold between quantities.

The language provides a means of combining primitive constraints in order to express more complex relations. We combine constraints by constructing _constraint networks_, in which constraints are joined by _connectors_. A connector is an object that _holds_ a value that may participate in one or more constraints.

Computation by such a network proceeds as follows: When a connector is given a value (by the user or by a constraint box to which it is linked), it awakens all of its associated constraints (except for the constraint that just awakened it) to inform them that it has a value. Each awakened constraint box then polls its connectors to see if there is enough information to determine a value for a connector. If so, the box sets that connector, which then awakens all of its associated constraints, and so on.

### Implementing the constraint system

The constraint system is implemented via procedural objects with local state. Although the primitive objects of the constraint system are somewhat more complex, the overall system is simpler, since there is no concern about agendas and logic delays.

The basic operations on connectors are the following:

- `(has-value? ⟨connector⟩)` tells whether the connector has a value.

- `(get-value ⟨connector⟩)` returns the connector’s current value.

- `(set-value! ⟨connector⟩ ⟨new-value⟩ ⟨informant⟩)` indicates that the informant is requesting the connector to set its value to the new value.

- `(forget-value! ⟨connector⟩ ⟨retractor⟩)` tells the connector that the retractor is requesting it to forget its value.

- `(connect ⟨connector⟩ ⟨new-constraint⟩)` tells the connector to participate in the new constraint.

The connectors communicate with the constraints by means of the procedures `inform-about-value`, which tells the given constraint that the connector has a value, and `inform-about-no-value`, which tells the constraint that the connector has lost its value.

### Representing connectors

A connector is represented as a procedural object with local state variables _value_, the current value of the connector; _informant_, the object that set the connector’s value; and _constraints_, a list of the constraints in which the connector participates.

## Concurrency: Time Is of the Essence

The central issue lurking beneath the complexity of state, sameness, and change is that by introducing assignment we are forced to admit _time_ into our computational models. The result of evaluating an expression depends not only on the expression itself, but also on whether the evaluation occurs before or after these assignment moments. Building models in terms of computational objects with local state forces us to confront time as an essential concept in programming.

> Before we introduced assignment, all our programs were time-agnostic, in the sense that any expression that has a value always has the same value.

In real world, objects do not change one at a time in sequence. Rather we perceive them as acting _concurrently_ -- all at once.

So it is often natural to model systems as collections of computational processes that execute concurrently.

Even if the programs are to be executed on a sequential computer, the practice of writing programs as if they were to be executed concurrently forces the programmer to avoid inessential timing constraints and thus makes programs more modular.

In addition to making programs more modular, concurrent computation can provide a speed advantage over sequential computation.

### The Nature of Time in Concurrent Systems

Several processes may share a common state variable. It's more complicated when more than one process may be trying to manipulate the shared state at the same time.

#### Correct behavior of concurrent programs

With concurrent processes we must be especially careful about assignments, because we may not be able to control the order of the assignments made by the different processes. To make concurrent programs behave correctly, we may have to place some restrictions on concurrent execution.

One possible restriction on concurrency would stipulate that no two operations that change any shared state variables can occur at the same time.

A less stringent restriction on concurrency would ensure that a concurrent system produces the same result as if the processes had run sequentially in some order. There are two important aspects to this requirement. First, it does not require the processes to actually run sequentially, but only to produce results that are the same as if they had run sequentially. Second, there may be more than one possible _correct_ result produced by a concurrent program, because we require only that the result be the same as for _some_ sequential order.

### Mechanisms for Controlling Concurrency

A more practical approach to the design of concurrent systems is to devise general mechanisms that allow us to constrain the interleaving of concurrent processes so that we can be sure that the program behavior is correct.

#### Serializing access to shared state

Serialization implements the following idea: Processes will execute concurrently, but there will be certain collections of procedures that cannot be executed concurrently. More precisely, serialization creates distinguished sets of procedures such that only one execution of a procedure in each serialized set is permitted to happen at a time. If some procedure in the set is being executed, then a process that attempts to execute any procedure in the set will be forced to wait until the first execution has finished.

#### Complexity of using multiple shared resources

Serializers provide a powerful abstraction that helps isolate the complexities of concurrent programs so that they can be dealt with carefully and (hopefully) correctly. However, while using serializers is relatively straightforward when there is only a single shared resource, concurrent programming can be treacherously difficult when there are multiple shared resources.

#### Implementing serializers

We implement serializers in terms of a more primitive synchronization mechanism called a _mutex_.

A mutex is an object that supports two operations: the mutex can be _acquired_, and the mutex can be _released_.

Once a mutex has been acquired, no other acquire operations on that mutex may proceed until the mutex is released.

The mutex is a mutable object that can hold teh value true or false. When the value is false, the mutex is available to be acquired. When the value is true, the mutex is unavailable, and any process that attempts to acquire the mutex must wait.

## Streams

From an abstract point of view, a _stream_ is simply a sequence of data objects.

Stream processing lets us model systems that have state without ever using assignment or mutable data.

### Streams Are Delayed Lists

Streams are a clever idea that allows one to use sequence manipulations without incurring the costs of manipulating sequences as lists. With streams we can achieve the best of both worlds: We can formulate programs elegantly as sequence manipulations, while attaining the efficiency of incremental computation.

The basic idea is to arrange to construct a stream only partially, and to pass the partial construction to the program that consumes the stream. If the consumer attempts to access a part of the stream that has not yet been constructed, the stream will automatically construct just enough more of itself to produce the required part, thus preserving the illusion that the entire stream exists.

> Generator is one implementation of streams.

The implementation of streams will be based on a special form called _delay_. Evaluating `(delay ⟨exp⟩)` does not evaluate the expression `⟨exp⟩`, but rather returns a so-called _delayed object_, which we can think of as a _promise_ to evaluate `⟨exp⟩` at some future time. As a companion to delay, there is a procedure called _force_ that takes a delayed object as argument and performs the evaluation—in effect, forcing the _delay_ to fulfill its promise.

In general, we can think of delayed evaluation as _demand-driven_ programming, whereby each stage in the stream process is activated only enough to satisfy the next stage. What we have done is to decouple the actual order of events in the computation from the apparent structure of our procedures. We write procedures as if the streams existed _all at once_ when, in reality, the computation is performed incrementally, as in traditional programming styles.

### Infinite Streams

We can user streams to represent sequences that are infinitely long. But in any given time we can examine only a finite portion of it.

#### Defining streams implicitly

An alternative way to specify streams is to take advantage of delayed evaluation to define streams implicitly.

### Exploiting the Stream Paradigm

The stream approach can be illuminating because it allows us to build systems with different module boundaries than systems organized around assignment to state variables. This makes it convenient to combine and compare components of state from different moments.

> For example, we can think of an entire time series/signal as a focus of interest, rather than the values of the state variables at individual moments.

To handle infinite streams, we need to devise an order of combination that ensures that every element will eventually be reached if we let the program run long enough.

### Streams and Delayed Evaluation

In general, _delay_ is crucial for using streams to model signal-processing systems that contain loops. Without _delay_, our models would have to be formulated so that the inputs to any signal-processing component would be fully evaluated before the output could be produced. This would outlaw loops.

### Modularity of Functional Programs and Modularity of Objects

One of the major benefits of introducing assignment is that we can increase the modularity of the systems by encapsulating, or _hiding_, parts of the state of a large system within local variables. Stream models can provide an equivalent modularity without the use of assignment.

#### A functional-programming view of time

We introduced assignment and mutable objects to provide a mechanism for modular construction of programs that model systems with state. We constructed computational objects with local state variables and used assignment to modify these variables. We modeled the temporal behavior of the objects in the world by the temporal behavior of the corresponding computational objects.

Now we have seen that streams provide an alternative way to model objects with local state. We can model a changing quantity, such as the local state of some object, using a stream that represents the time history of successive states. In essence, we represent time explicitly, using streams, so that we decouple time in our simulated world from the sequence of events that take place during evaluation. Indeed, because of the presence of delay there may be little relation between simulated time in the model and the order of events during the evaluation.

From the point of view of one part of a complex process, the other parts appear to change with time. They have hidden time-varying local state. If we wish to write programs that model this kind of natural decomposition in our world (as we see it from our viewpoint as a part of that world) with structures in our computer, we make computational objects that are not functional—they must change with time. We model state with local state variables, and we model the changes of state with assignments to those variables. By doing this we make the time of execution of a computation model time in the world that we are part of, and thus we get “objects” in our computer.

Modeling with mutable objects raise thorny problems of constraining the order of events and of synchronizing multiple processes. The possibility of avoiding these problems has stimulated the development of _functional programming_ languages, which do not include any provision for assignment or mutable data.

> The functional approach is extremely attractive for dealing with concurrent systems.

There are some time-related problems creeping into functional models as well. One particularly troublesome area arises when we wish to design interactive systems, especially ones that model interactions between independent entities. The trouble comes with the notion of _merge_. Such a merge is implemented to interleave these inputs in some way that is constrained by _real time_ as perceived by mutations. Technically, _merge_ is a relation rather than a function.

The need to introduce explicit synchronization to ensure a _correct_ order of events in concurrent processing of objects with state. Thus, in an attempt to support the functional style, the need to merge inputs from different agents reintroduces the same problems that functional style was meant to eliminate.

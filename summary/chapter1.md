# Reactive Programming with RxJava

RxJava is a specific implementation of reactive programming for Java that is influenced by functional programming. It favors function composition, avoidance of global state and side effects, and thinking in streams to compose asynchronous and event-based programs.

It begins with the observer pattern of producer/ consumer callbacks and extends it with dozens of operators that allow composing, transforming, scheduling, throttling, error handling, and lifecycle management.

## Reactive programming and RxJava
Reactive programming is a general programming term that is focused on reacting to changes, such as data values or events.

The short answer to what reactive-functional programming is solving is concurrency and parallelism.

RxJava is influenced by functional programming and uses a declarative approach to avoiding the typical pitfalls of reactive-imperative code.

## When You Need Reactive Programming

## How RxJava Works

Central to RxJava is the Observable type that represents a stream of data or events. It is intended for push (reactive) but can also be used for pull (interactive). It is lazy rather than eager. It can be used asynchronously or synchronously. It can represent 0, 1, many, or infinite values or events over time.

### Push versus Pull

The entire point of RxJava being reactive is to support push, so the Observable and related Observer type signatures support events being pushed at it.

The Observable represents the stream of data and can be subscribed to by an Observer.

```java
interface Observable<T> { 
    Subscription subscribe(Observer s) 
}
```

Upon subscription, the Observer can have three types of events pushed to it: 

- Data via the onNext() function 
- Errors (exceptions or throwables) via the onError() function 
- Stream completion via the onCompleted() function

```java
interface Observer <T> { 
    void onNext(T t) 
    void onError(Throwable t) 
    void onCompleted() 
}
```

The onNext() method might never be called or might be called once, many, or infinite times. The onError() and onCompleted() are terminal events, meaning that only one of them can be called and only once. When a terminal event is called, the Observable stream is finished and no further events can be sent over it. Terminal events might never occur if the stream is infinite and does not fail.

There is an additional type of signature to permit interactive pull: 

```java
interface Producer { 
    void request(long n) 
}
```

This is used with a more advanced Observer called Subscriber:

```java
interface Subscriber<T> implements Observer <T>, Subscription { 
    void onNext(T t) 
    void onError(Throwable t) 
    void onCompleted() 
    ...
    void unsubscribe() 
    void setProducer(Producer p)
}
```

The unsubcribe function as part of the Subscription interface is used to allow a subscriber to unsubscribe from an Observable stream. The setProducer function and Producer types are used to form a bidirectional communication channel between the producer and consumer used for flow control.

### Async versus Sync

An Observable can be synchronous, and in fact defaults to being synchronous. RxJava never adds concurrency unless it is asked to do so.

The actual criteria that is generally important is whether the Observable event production is blocking or nonblocking, not whether it is synchronous or asynchronous.

#### In-memory data

If data exists in a local in-memory cache (with constant microsecond/ nanosecond lookup times), it does not make sense to pay the scheduling cost to make it asynchronous.

#### Synchronous computation (such as operators)

The more common reason for remaining synchronous is stream composition and transformation via operators. RxJava mostly uses the large API of operators used to manipulate, combine, and transform data, such as map(), filter(), take(), flatMap(), and groupBy(). Most of these operators are synchronous.

This is generally the behavior we want: an asynchronous pipeline (the Observable and composed operators) with efficient synchronous computation of the events. 

Thus, the Observable type itself supports both sync and async concrete implementations, and this is by design.

### Concurrency and Parallelism

Parallelism is simultaneous execution of tasks, typically on different CPUs or machines. Concurrency, on the other hand, is the composition or interleaving of multiple tasks. If a single CPU has multiple tasks (such as threads) on it, they are executing concurrently but not in parallel by “time slicing.” Each thread gets a portion of CPU time before yielding to another thread, even if a thread has not yet finished. 

Parallel execution is concurrent by definition, but concurrency is not necessarily parallelism.

The contract of an RxJava Observable is that events (onNext(), onCompleted(), onError()) can never be emitted concurrently. In other words, a single Observable stream must always be serialized and thread-safe.

You take advantage of concurrency and/or parallelism with RxJava using composition.

A single Observable stream is always serialized, but each Observable stream can operate independently of one another, and thus concurrently and/ or in parallel.

### Lazy versus Eager

The Observable type is lazy, meaning it does nothing until it is subscribed to.

Because the Observable is lazy, it also means a particular instance can be invoked more than once.

### Duality

An Rx Observable is the async “dual” of an Iterable. By “dual,” we mean the Observable provides all the functionality of an Iterable except in the reverse flow of data: it is push instead of pull.

The fact that it behaves as a dual effectively means anything you can do synchronously via pull with an Iterable and Iterator can be done asynchronously via push with an Observable and Observer. This means that the same programming model can be applied to both!

### Cardinality

The Observable supports asynchronously pushing multiple values.

#### Event stream

Over time the producer pushes events at the consumer.

#### Multiple values

Basically, anywhere that a List, Iterable, or Stream would be used, Observable can be used instead.

#### Composition

A multivalued Observable type is also useful when composing single-valued responses.

#### Single

Using Single instead of Observable to represent a “stream of one” simplifies consumption because a developer must consider only the following behaviors for the Single type: 

- It can respond with an error 
- Never respond 
- Respond with a success

#### Completable

The Completable type addresses the surprisingly common use case of having no return type, just the need to represent successful or failed completion.

#### Zero to infinity

Observable can support cardinalities from zero to infinity. But for simplicity and clarity, Single is an "Observable of One,” and Completable is an "Observable of None.”

### Mechanical Sympathy: Blocking versus Nonblocking I/O

Better latency and throughput means both better user experience and lower infrastructure cost.

The event-loop architecture is more resilient under load.

### Reactive Abstraction

Ultimately RxJava types and operators are just an abstraction over imperative callbacks. However, this abstraction completely changes the coding style and provides very powerful tools for doing async and nonblocking programming.

## Additional reading

- https://speakerdeck.com/benjchristensen/applying-reactive-programming-with-rxjava-at-goto-chicago-2015?slide=146
- https://www.reactivemanifesto.org/
- https://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming/1030631#1030631
- https://www.youtube.com/watch?v=8OcCSQS0tug
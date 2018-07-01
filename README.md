# Java concepts
Lists and describes key concepts of Java language, JVM architecture, Design principles and more.

## Table of Content

1. [JVM Garbage Collector](#gc)
2. [S.O.L.I.D principles](#solid)

<a id='gc' />

## Java's Automatic Memory management (a.k.a Garbage collectors)

Reference - [Whitepaper on Automatic memory management by Oracle](http://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf)

### Problems with explicit memory management

Absence of automatic memory management, leads to lot of programming errors, as deallocating the memory for an object created earlier, will be programmer's responsibility.

*  *Dangling references* - Let's say we have an object `B` pointing to another object `A`. A programmer deallocates the memory allocated for `A` and in the same memory location, another new object `C` gets created. `B` now becomes a *dangling reference* since it is pointing to a wrong object whole together, causing undesirable results.

* *Space Leaks* - Let's say we have a linked list, pointed by `head` reference. If the programmer deallocates memory of only `head` object, then the remaining objects of linked list (earlier pointing to `head`) become unreachable, hence can get unclaimed, causing them to stay in the memory, without any object referring to them.

An automatic memory manager, or **Garbage collector** solves *dangling refernces* because, it will never deallocate the memory of the object (`A`) referenced by another object (`B`). It also solves the *space leak* problem my automatically deallocating the memory claimed by objects which are no longer reachable.

### Responsibilities of Garbage collector (GC)

1. Allocating memory
2. Ensuring any referenced object remains in the memory
3. Recovering memory used by objects, no longer referenced by the currently executing code

Objects that are referenced are called **live** and objects which are no longer referenced are termed as **dead** (or **garbage**), and GC's job is to free up the memory held by **dead** objects.

### GC characterisitics 

* GC shouldn't collect a *live* object, erroneously.
* GC activity should be efficient, it shouldn't pause the application threads.
* GC starts working when the remaining memory on the heap, reaches some predefined threashold.
* If heap size is small,  GC will run frequently because the objects fill up the heap pretty quickly. At the same time, the duration of the GC activity will be short.
* When memories get freed up in small chunks, across different areas of heap - *fragmentation* occurs. In this case, it becomes difficult for GC to allocate memory for a large object by combining these fragments together, which are spread across the heap. GC uses special method called *Compaction* for this purpose.
* If the heap size is large, then GC might run less frequently, but when it runs, the process might take longer time.

### GC Design choices

#### Serial vs. Parallel

In Serial GC, one thing happens at a time. While in parallel GC, multiple GC related operations are executed in parallel, utilizing the capabilities of multi-core processor environments. Parallel GC will be more efficient.

#### Concurrent vs. Stop-the-world GC

Stop-the-world GC completely halts the execution of the application, by locking the heap area, to perform the GC operation. They are easy to implement because the state of the objects in heap don't change. It can be undesirable for some applications to be completely paused during GC.

In concurrent GC, the  process of garbage collection happens along with the execution of other application code. The implementation of concurrent GC is complicated because, the state of objects in the heap will change fequently. 

#### Compacting vs. Non-compacting

Once the GC determines which are live and which are garbage obejcts, it can **compact** all the live objects together and reclaim the remaining free memory. When a request comes to allocate memory for a new object, the free memory area is looked up, using a simple pointer. In the compating methodology, the process of bringing together live objects are time consuming, but there will be no *fragmentation*. 

In **non-compating** methodology, the memories are reclaimed in-place, causing lots of *fragmentation*. Here deallocating memory is a fast process but, allocating memory for a new object is a long process. If a subsequently large objects requests for new memory, then fragments of memory locations should be scanned to check, if they satisfy the requirement of large object.

<a id='solid' />

## S.O.L.I.D Principles

### S - Single Responsibility Principle (S.R.P)

* A class should have one, and only one, reason to change.
* A class having single responsibility (reason for change) will be susceptible for less changes when the requirement changes.
* Let's say our application requires social media integration. Then, separating the social media integration into a different module can help our application adapt to external changes, without much change from our side.

### O - Open and Closed Principle 

* A class must be open for extension and closed for modifications.
* Abstraction can help to achieve this. Example - Lets say we need to work with multiple lots of shapes like `Circle`, `Rectangle`. We need to create an interface called `Shape` which we can use in the client code. This enableds us to introduce more shapes and to inject them, without having to change the client code.

### L - Liskov Substitution Principle

* If we have a subclass `S` of a super class `T`, then we should be able to substitute `S` in place of `T`, without altering the desirable properties of the program.
* Example, Let's say we have a base class `Point` which has properties `x` and `y`. We have a sub-type `CirclePoint` which adds `color` property on top of the `Point` class. Now if we test the equality a `ColorPoint` with another `Point` having same `x` and `y`, they should be equal. (substitution of `CirclePoint` with `Point` should work as before)

### I - Interface Segregation Principle

* We favour small interfaces than a big monolithic interface. We will segregate the big interface into smaller ones
* Example - Lets say we have an interface `Animal` with methods `eat()`, `walk()` and `sleep()`. Now all the implementors should implement the methods, irrespective of whether they want to or not. Now lets say, we would like to add another property called `fly()`? How to deal with this?
* Instead of `Animal` having different roles as methods - create **role** based interfaces like `CanEat`, `CanWalk` and `CanSleep`. This enables clients to implement what they want.

### D - Dependency Inversion Principle

a. A high-level module should not depend on low-level module. Instead both should depend on abstraction.
b. Abstractions should not depend on details, rather details should depend on abstractions

Example - Dependency Injection. Lets say we would like to use authentication service and we have 2 providers. We would like to use each of them for different purposes. We create our method something like below

    void authenticate(AuthenticationService service) {
    
    }

And invoke them using `authenticate(new GoogleAuthenticationService());` or `authenticate(new GitHubAuthenticationService());`


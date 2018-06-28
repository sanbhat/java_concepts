# java_concepts
Lists and describes key concepts of Java language and JVM architecture

# Java's Automatic Memory management (a.k.a Garbage collectors)

Reference - [Whitepaper on Automatic memory management by Oracle](http://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf)

## Problems with explicit memory management

Absence of automatic memory management, leads to lot of programming errors, as deallocating the memory for an object created earlier, will be programmer's responsibility.

*  *Dangling references* - Let's say we have an object `B` pointing to another object `A`. A programmer deallocates the memory allocated for `A` and in the same memory location, another new object `C` gets created. `B` now becomes a *dangling reference* since it is pointing to a wrong object whole together, causing undesirable results.

* *Space Leaks* - Let's say we have a linked list, pointed by `head` reference. If the programmer deallocates memory of only `head` object, then the remaining objects of linked list (earlier pointing to `head`) become unreachable, hence can get unclaimed, causing them to stay in the memory, without any object referring to them.

An automatic memory manager, or **Garbage collector** solves *dangling refernces* because, it will never deallocate the memory of the object (`A`) referenced by another object (`B`). It also solves the *space leak* problem my automatically deallocating the memory claimed by objects which are no longer reachable.

## Responsibilities of Garbage collector (GC)

1. Allocating memory
2. Ensuring any referenced object remains in the memory
3. Recovering memory used by objects, no longer referenced by the currently executing code

Objects that are referenced are called **live** and objects which are no longer referenced are termed as **dead** (or **garbage**), and GC's job is to free up the memory held by **dead** objects.

## GC characterisitics 

* GC shouldn't collect a *live* object, erroneously.
* GC activity should be efficient, it shouldn't pause the application threads.
* GC starts working when the remaining memory on the heap, reaches some predefined threashold.
* If heap size is small,  GC will run frequently because the objects fill up the heap pretty quickly. At the same time, the duration of the GC activity will be short.
* When memories get freed up in small chunks, across different areas of heap - *fragmentation* occurs. In this case, it becomes difficult for GC to allocate memory for a large object by combining these fragments together, which are spread across the heap. GC uses special method called *Compaction* for this purpose.
* If the heap size is large, then GC might run less frequently, but when it runs, the process might take longer time.

## GC Design choices

### Serial vs. Parallel

In Serial GC, one thing happens at a time. While in parallel GC, multiple GC related operations are executed in parallel, utilizing the capabilities of multi-core processor environments. Parallel GC will be more efficient.

### Concurrent vs. Stop-the-world GC

Stop-the-world GC completely halts the execution of the application, by locking the heap area, to perform the GC operation. They are easy to implement because the state of the objects in heap don't change. It can be undesirable for some applications to be completely paused during GC.

In concurrent GC, the  process of garbage collection happens along with the execution of other application code. The implementation of concurrent GC is complicated because, the state of objects in the heap will change fequently. 

### Compacting vs. Non-compacting

Once the GC determines which are live and which are garbage obejcts, it can **compact** all the live objects together and reclaim the remaining free memory. When a request comes to allocate memory for a new object, the free memory area is looked up, using a simple pointer. In the compating methodology, the process of bringing together live objects are time consuming, but there will be no *fragmentation*. 

In **non-compating** methodology, the memories are reclaimed in-place, causing lots of *fragmentation*. Here deallocating memory is a fast process but, allocating memory for a new object is a long process. If a subsequently large objects requests for new memory, then fragments of memory locations should be scanned to check, if they satisfy the requirement of large object.

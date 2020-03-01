## CHAPTER 16 The Java Memory Model

Safe publication and synchronization policies derive their safety from the JMM, and you will find it easier to use these mechanisms when you understand why they work. This chapter reveal the low-level requirements and guarantees of the Java Memory Model.

### 16.1 What is a memory model, and why would I want one?

There are a number of factors that can prevent a thread from seeing the most up-to-date value for a variable and can cause memory actions in other threads to appear to happen out of order, with out adequate synchronization.

1. Compilers may generate instructions in a different order than the source code,
2. or store variables in registers instead of in memory.
3. processors may execute instructions in parallel or out of order;
4. caches may vary the order in which writes to variables are committed to main memory;
5. values stored in processor-local caches may not be visible to other processors.

In a single-threaded environment , all these tricks played on our program have no effect other than to speed up execution. The Java Language Specification requires the JVM to maintain ***within-thread as-if-serial semantics*** : as long as the program has the same result as if it were executed in program order.

In a multithreaded environment, since most of the time threads in a concurrent application are each "doing their own thing", it only when multiple threads share data that it is necessary to coordinate activities, and the JVM relies on the program to identify when this is happening by using synchronization.

The JMM specifies the minimal guarantees the JVM must make about when writes to variables become visible to other threads.

#### 16.1.1 Platform memory models

1. In a shared-memory multiprocessor architecture, each processor has its own cache that is periodically reconciled with main memory.
2. And, this introduces the problem of **cache coherence**. 
3. Processor architectures provide varying degrees of cache coherence, some provide minimal guarantees that allow different processors to see different values for the same memory locations at virtually any time. 
4. Ensuring that every processor knows what every other processor is doing at all time is expensive. Most of the time this information is not needed, so processors relax their memory-coherency guarantees to improve performance.
5. An architecture's **memory model**  specifies the special instructions required (called **memory barriers** or **fences**) to get the additional coordination guarantees required when sharing data.
6. In order to shield the Java developer from *the differences between memory models* across architectures, Java provides its own memory model, and JVM deals with the differences between the JMM and the underlying platform's memory model by *inserting memory barriers* at the appropriate places.
7. Fortunately, Java programmers  need not specify the placement of memory barriers, they need only identify when shared state is being accessed, through the proper use of synchronization. 

#### 16.1.2 Reordering

In race conditions and atomicity failures, scheduler interleaves operations so as to cause incorrect results in  insufficiently synchronized programs. 

To make matters worse, the JMM can permit actions to *appear* to execute in different orders from the perspective of different threads, making reasoning about ordering in the absence of synchronization even more complicated.

The various reasons why operations might be delayed or appear to execute out of order can all be grouped into the general category of **reordering** .

*Reordering at the memory level* can make programs behave unexpectedly. It is prohibitively difficult to reason about ordering in the absence of synchronization; it is much easier to ensure that your program uses synchronization appropriately.   

Synchronization inhibits the compiler, runtime, and hardware from *reordering memory operations in ways that would violate the **visibility guarantees** provided by the JMM*. 

#### 16.1.3 The Java Memory Model in 500 words of less.

The  Java Memory Model is specified in terms of *actions*, which include reads and writes to variables, locks and unlocks on monitor, and starting and joining with thread.

The JMM defines a partial ordering called ***happens-before*** on all actions within the program. To guarantee that thread executing action B see the results of action A ,there must be a *happens-before* relationship between A and B.  In the absence of *happens-before* ordering between two operations, the JVM is free to reorder them as it please.

A ***data race*** occurs when

1.  a variable is read by more than one thread,
2. and written by at least one thread,
3. but the reads and writes are not ordered by *happens-before*.


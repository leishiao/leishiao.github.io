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


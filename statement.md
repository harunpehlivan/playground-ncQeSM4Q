# Garbage Collection and C#
![alt text](https://www.codeproject.com/KB/cs/1095402/GC.png) 

## What is Garbage Collection and Why We Need It?
When you create any object in C#, CLR (common language runtime) allocates memory for the object from heap. This process is repeated for each newly created object, but there is a limitation to everything, Memory is not un-limited and we need to clean some used space in order to make room for new objects, Here, the concept of garbage collection is introduced, Garbage collector manages allocation and reclaiming of memory. GC (Garbage collector) makes a trip to the heap and collects all objects that are no longer used by the application and then makes them free from memory.

## Memory facts
![alt text](https://www.codeproject.com/KB/cs/1095402/mem1.jpg)
When any process gets triggered, separate virtual space is assigned to that process, from a physical memory which is the same and used by every process of a system, any program deals with virtual space not with physical memory, GC also deals with the same virtual memory to allocate and de-allocate memory. Basically, there are free-blocks that exist in virtual memory (also known as holes), when any object request for memory allocation manager searches for free-block and assigns memory to the said object.

Virtual memory has three blocks:

1. Free (empty space)
2. Reserved (already allocated)
3. Committed (This block is give-out to physical memory and not available for space allocation)

*You may face out of memory error due to virtual memory full.*

## How GC works?
![alt text](https://www.codeproject.com/KB/cs/1095402/Garbage.png)
GC works on managed heap, which is nothing but a block of memory to store objects, when garbage collection process is put in motion, it checks for dead objects and the objects which are no longer used, then it compacts the space of live object and tries to free more memory.

Basically, heap is managed by different 'Generations', it stores and handles long-lived and short-lived objects, see the below generations of Heap:

1. **0 Generation (Zero):** This generation holds short-lived objects, e.g., Temporary objects. GC initiates garbage collection process frequently in this generation.
2. **1 Generation (One):** This generation is the buffer between short-lived and long-lived objects.
3. **2 Generation (Two):**  This generation holds long-lived objects like a static and global variable, that needs to be persisted for a certain amount of time. Objects which are not collected in generation Zero, are then moved to generation 1, such objects are known as survivors, similarly objects which are not collected in generation One, are then moved to generation 2 and from there onwards objects remain in the same generation.

## How GC decides if objects are live?
GC checks the below information to check if the object is live:
1. It collects all handles of an object that are allocated by user code or by CLR
2. Keeps track of *static* objects, as they are referenced to some other objects
3. Use stack provided by stack walker and JIT

## When GC gets triggered?
There are *no specific timings* for GC to get triggered, GC automatically starts operation on the following conditions:
1. When virtual memory is running out of space.
2. When allocated memory is suppressed acceptable threshold (when GC found if the survival rate (living objects) is high, then it increases the threshold allocation).
3. When we call *GC.Collect()* method explicitly, as GC runs continuously, we actually do not need to call this method.

## What is managed and unmanaged objects/resources?
![alt text](https://www.codeproject.com/KB/cs/1095402/managed.png)VS !(https://www.codeproject.com/KB/cs/1095402/stack.png)



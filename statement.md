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
![alt text](https://www.codeproject.com/KB/cs/1095402/managed.png)
In simple terms:

**Managed objects** are created, managed and under scope of CLR, pure .NET code managed by runtime, Anything that lies within .NET scope and under .NET framework classes such as string, int, bool variables are referred to as managed code.

**UnManaged** objects are created outside the control of .NET libraries and are not managed by CLR, example of such unmanaged code is COM objects, file streams, connection objects, Interop objects. (Basically, third party libraries that are referred in .NET code.)

## Clean Up Unmanaged Resources
![alt text](https://www.codeproject.com/KB/cs/1095402/Broom.jpg)
When we create unmanaged objects, GC is unable to clear them and we need to release such objects explicitly when we finished using them. Mostly unmanaged objects are wrapped/hide around operating system resources like file streams, database connections, network related instances, handles to different classes, registries, pointers etc. GC is responsible to track the life time of all managed and unmanaged objects but still GC is not aware of releasing unmanaged resources

There are different ways to cleanup unmanaged resources:
1. Implement *IDisposable* interface and *Dispose* method
2. *'using'* block is also used to clean unmanaged resources

There are couple of ways to implement *Dispose* method:
1. Implement Dispose using 'SafeHandle' Class (It is inbuilt abstract class which has 'CriticalFinalizerObject' and 'IDisposable' interface has been implemented)
2. Object.Finalize method to be override (This method is clean unmanaged resources used by particular object before it is destroyed)

Let's see below code to Dispose unmanaged resources:
1. Implement Dispose using 'SafeHandle' Class:
```javascript
class clsDispose_safe
    {
        // Take a flag to check if object is already disposed
        bool bDisposed = false;

        // Create a object of SafeHandle class
        SafeHandle objSafeHandle = new SafeFileHandle(IntPtr.Zero, true);

        // Dispose method (public)
        public void Dispose1()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        // Dispose method (protected)
        protected virtual void Dispose(bool bDispose)
        {
            if (bDisposed)
                return;

            if (bDispose)
            {
                objSafeHandle.Dispose();
                // Free any other managed objects here.
            }

            // Free any unmanaged objects here.
            //
            bDisposed = true;
        }
    }
```
2. Implement Dispose using overriding the 'Object.Finalize' method:
```javascript
class clsDispose_Fin
    {
        // Flag: Has Dispose already been called?
        bool disposed = false;

        // Public implementation of Dispose pattern callable by consumers.
        public void Dispose()
        {
            Dispose1(true);
            GC.SuppressFinalize(this);
        }

        // Protected implementation of Dispose pattern.
        protected virtual void Dispose1(bool disposing)
        {
            if (disposed)
                return;

            if (disposing)
            {
                // Free any other managed objects here.
                //
            }

            // Free any unmanaged objects here.
            //
            disposed = true;
        }

        ~clsDispose_Fin()
        {
            Dispose1(false);
        }
    }
```

## 'using' Statement
using statement ensures object dispose, in short, it gives a comfort way of use of IDisposable objects. When an Object goes out of scope, Dispose method will get called automatically, basically using block does the same thing as 'TRY...FINALLY' block. To demonstrate it, create a class with IDisposable implementation (it should have Dispose() method), 'using' statement calls 'dispose' method even if exception occurs.
See the below snippet:
```javascript
class testClass : IDisposable
{
    public void Dispose()
    {
        // Dispose objects here
 	// clean resources
 	Console.WriteLine(0);
    }
}

//call class
class Program
{
    static void Main()
    {
        // Use using statement with class that implements Dispose.
 	using (testClass objClass = new testClass())
 	{
     		Console.WriteLine(1);
	}
	 Console.WriteLine(2);
    }
}

//output
1
0
2

//it is same as below TRY...Finally code
{
    clsDispose_Fin objClass = new clsDispose_Fin();
    try
    {
        //code goes here 
    }
    finally
    {
        if (objClass != null)
        ((IDisposable)objClass).Dispose();
    }
}
```
In the above sample after printing 1, using block gets end and calls Dispose method and then call statement after using.

## Re-view
![alt text](https://www.codeproject.com/KB/cs/1095402/Recycle.png)
1. Garbage collector manages allocation and reclaim of memory.
2. GC works on managed heap, which is nothing but a block of memory to store objects.
3. There is no specific timings for GC to get triggered, GC automatically start operation.
4. Managed objects are created, managed and under scope of CLR.
5. Unmanaged objects are wrapped around operating system resources like file streams, database connections, network related instances, handles to different classes, registries, pointers, etc.
6. Unmanaged resources can be cleaned-up using 'Dispose' method and 'using' statement.

## Thanks
Still more to come on this subject, I will cover it in later builds. Suggestions and queries are always welcome.

Thanks for reading!




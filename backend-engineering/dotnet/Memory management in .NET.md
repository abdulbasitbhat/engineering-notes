# 🧠 Memory Management in .NET — Garbage Collection & Allocation

# 🏗️ 1. How Memory Works in .NET (High Level)

.NET uses **managed memory**.

You do NOT manually free memory like C/C++.  
Instead, Garbage Collector (GC) automatically manages memory.

Flow:  
Create object → Stored in managed heap → Used by program → No references → GC cleans it

You only create objects.  
.NET runtime handles cleanup.

---

# 📦 2. Stack vs Heap (Core Concept)

Understanding this is essential.

## 🟦 Stack (Fast, small, per-thread)

Stores:
- Method calls  
- Local variables  
- Value types (int, bool, struct)  

Characteristics:
- Very fast  
- Auto cleaned when function ends  
- Limited size  
- No GC involved  

Example:

    void Test()
    {
        int x = 10;   // stored in stack
    }

When function ends → memory gone.

---

## 🟩 Heap (Large, managed by GC)

Stores:
- Objects (classes)  
- Strings  
- Arrays  
- Reference types  

Example:

    var user = new User(); // stored in heap

Heap memory is cleaned by Garbage Collector, not instantly.

---

# 🔗 3. Reference vs Value Types

## Value types → stack (mostly)

    int a = 10;
    double b = 20;

## Reference types → heap

    User u = new User();

But reference variable itself is on stack → points to heap.

Stack → reference → heap object

---

# 🧹 4. Garbage Collector (GC)

GC automatically frees unused memory.

When does GC run?
- Heap memory getting full  
- System under memory pressure  
- Runtime decides best time  
- Manual trigger possible but rare  

You cannot fully control it.

---

# 🧠 5. How GC Works (Mark & Sweep)

GC process simplified:

Step 1: Mark  
GC checks which objects are still referenced.

Example:

    var a = new User();
    var b = new User();

    a = null;

Now:
- Object from a → unreachable  
- Object from b → still used  

Step 2: Sweep  
Unreachable objects removed from memory.

Step 3: Compact  
GC shifts memory to remove gaps and fragmentation.

---

# 🧬 6. GC Generations

.NET uses Generational Garbage Collection.

Idea:  
Most objects die young.

So GC divides heap into generations.

## 🟢 Gen 0 (Short-lived objects)
- Temporary objects  
- Method variables  
- Frequent cleanup  
- Fast collection  

Example:

    string temp = "hello";

## 🟡 Gen 1 (Buffer zone)
Objects surviving Gen 0 move here.

## 🔴 Gen 2 (Long-lived objects)
- Static objects  
- App-level caches  
- Singletons  

Collected less frequently because expensive.

Flow:
New object → Gen0  
Survives → Gen1  
Survives → Gen2  

---

# 📍 7. Large Object Heap (LOH)

Objects larger than 85 KB go to LOH.

Examples:
- Large arrays  
- Large JSON  
- Images  
- Big buffers  

    byte[] data = new byte[100000]; // LOH

LOH collection:
- Not frequent  
- Expensive  
- Can cause performance issues  

Avoid frequent large allocations.

---

# ⚡ 8. Allocation in .NET

Allocation is actually very fast.

Why?  
.NET uses pointer bump allocation similar to stack allocation speed.

    var obj = new User();

This is cheap.

What is expensive?
- Frequent GC runs  
- Large objects  
- Too many allocations  

---

# 🔥 9. GC Pressure (Performance Killer)

GC pressure = Too many allocations → frequent GC runs.

Causes:
- Creating many temporary objects  
- Large objects repeatedly  
- Boxing/unboxing  
- Heavy string concatenation  

Bad example:

    for(int i=0;i<100000;i++)
    {
        var user = new User(); // heavy allocation
    }

Better:
- Reuse objects  
- Use pooling  
- Avoid unnecessary allocations  

---

# 🧵 10. Memory in High Performance Systems

In backend APIs or systems:
- GC pauses affect latency  
- High allocation slows APIs  
- Large objects cause fragmentation  

Good engineers:
- Minimize allocations  
- Use object pooling  
- Use Span/Memory  
- Avoid unnecessary new objects  

---
# 🛠️ 11. IDisposable & Unmanaged Memory (Important)

Garbage Collector handles only **managed memory** (objects in heap).

But many real-world resources are **unmanaged**:
- File handles  
- Database connections  
- Network sockets  
- OS handles  
- Streams  
- Timers  

These are NOT cleaned immediately by GC.

If you don't release them manually → memory leaks, locked files, DB connection exhaustion.

That is why .NET provides **IDisposable**.

---

## 🧠 Why IDisposable Exists

GC cleans memory  
IDisposable cleans external resources

Example problem:

If you open 1000 DB connections and don't close them:
- GC may not run immediately
- Connections stay open
- App crashes due to connection pool exhaustion

So we manually release critical resources.

---

## 🧾 Using "using" Statement (Most Common)

Best and safest way to use IDisposable objects.

Example:

    using (var file = new FileStream("a.txt", FileMode.Open))
    {
        // use file here
    }

What happens internally:
1. FileStream created
2. Code runs inside block
3. Dispose() automatically called
4. File handle released

Even if exception occurs → Dispose still runs.

Equivalent to:

    var file = new FileStream("a.txt", FileMode.Open);
    try
    {
        // use file
    }
    finally
    {
        file.Dispose();
    }

So `using` = automatic cleanup.

---

## 📦 Common Types That Must Be Disposed

Always use `using` with:

- FileStream  
- StreamReader / StreamWriter  
- SqlConnection  
- HttpResponseMessage  
- Timers  
- MemoryStream  
- Any class implementing IDisposable  

Example:

    using var conn = new SqlConnection(connectionString);
    conn.Open();

Connection automatically closed when scope ends.

---

## 🏗️ Creating Your Own IDisposable Class

You implement IDisposable when your class:
- Uses unmanaged resources
- Holds file/DB/socket
- Manages heavy memory buffers
- Wraps external resource

Example:

    public class FileManager : IDisposable
    {
        private FileStream _file;

        public FileManager(string path)
        {
            _file = new FileStream(path, FileMode.Open);
        }

        public void Write(string text)
        {
            // write to file
        }

        public void Dispose()
        {
            _file?.Dispose();
        }
    }

Usage:

    using(var manager = new FileManager("a.txt"))
    {
        manager.Write("hello");
    }

When block ends → Dispose called → file released.

---

## 🔥 Real Production Rule

If your class uses ANY of these:
- File
- DB connection
- Stream
- Socket
- Timer
- Unmanaged pointer

👉 Implement IDisposable

---

# 🧠 12. IDisposable vs Finalizer

Finalizer (~Destructor):
- Called by GC
- Unpredictable timing
- Slow
- Avoid when possible

IDisposable:
- Called by developer
- Immediate cleanup
- Predictable
- Fast

Always prefer IDisposable over finalizer.

---

# 📊 13. Boxing & Unboxing (Hidden Allocations)

Boxing = value type → object (heap allocation)

    int x = 10;
    object o = x; // boxing

Unboxing:

    int y = (int)o;

Boxing creates heap allocation → GC pressure.

Avoid in loops and high-performance paths.

---

# 🚀 14. Best Practices for Engineers

## Do this
- Reuse objects where possible  
- Dispose resources properly  
- Avoid large allocations  
- Use structs for small data  
- Use pooling for heavy objects  
- Measure memory usage in production  

## Avoid
- Creating objects in tight loops  
- Large temporary arrays  
- Unnecessary LINQ in hot paths  
- String concatenation in loops  
- Forgetting to dispose resources  

---

# 🏁 Final Engineering Insight

.NET uses managed memory with a generational garbage collector. Objects are allocated on the heap and automatically cleaned when no longer referenced. GC uses Gen0, Gen1, and Gen2 to optimize performance based on object lifetime. Large objects go to the Large Object Heap and are collected less frequently. Developers should minimize allocations and properly dispose unmanaged resources to maintain performance.

Bad engineers:  
Just write code and trust GC.

Good engineers:  
Understand allocations and reduce GC pressure.

Great engineers:  
Design systems with memory behavior in mind.

Memory awareness = performance awareness.

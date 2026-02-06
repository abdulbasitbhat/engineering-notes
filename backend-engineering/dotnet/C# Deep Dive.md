# 🟢 1. What is C#?

C# is a object-oriented, strongly typed programming language created by Microsoft.

C# runs on **.NET runtime (CLR — Common Language Runtime)**.

Execution flow:

C# code  
→ compiled into IL (Intermediate Language)  
→ CLR loads IL  
→ JIT converts to machine code  
→ program runs  

So C# does NOT run directly on OS.  
It runs inside CLR.

---

# 🟢 2. Basic Structure of C# Program

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello World");
    }
}
```

# 🟢 3. Variables & Data Types

C# is strongly typed → every variable has a type.

## Common types

```csharp
int age = 25;
double price = 99.99;
bool isActive = true;
string name = "Basit";
char grade = 'A';
```

## var keyword

```csharp
var x = 10;      // int
var name = "A";  // string
```

Type decided at compile time (not dynamic).

Use var when type obvious.

---

# 🟢 4. Value Types vs Reference Types

One of most important concepts.

## Value Types (stored directly)
Stored in stack or inline memory.

Examples:
- int
- bool
- double
- struct
- enum

```csharp
int a = 10;
int b = a;

b = 20;

Console.WriteLine(a); // 10
```

Copy created.

## Reference Types (stored as reference)
Stored in heap, variable holds reference.

Examples:
- class
- object
- string
- array
- list

```csharp
class User { public string Name; }

User u1 = new User();
u1.Name = "A";

User u2 = u1;
u2.Name = "B";

Console.WriteLine(u1.Name); // B
```

Both refer to same object.

---

# 🟢 5. Control Statements

## If/else
```csharp
if(age > 18)
{
    Console.WriteLine("Adult");
}
else
{
    Console.WriteLine("Minor");
}
```

## Switch (modern)
```csharp
int x = 5;

switch(x)
{
    case 5:
        Console.WriteLine("Five");
        break;
}
```

## Loops

### for loop
```csharp
for(int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}
```

### foreach (very common)
```csharp
var nums = new int[]{1,2,3};

foreach(var n in nums)
{
    Console.WriteLine(n);
}
```

Used heavily with collections.

---

# 🟢 6. Methods (Functions)

```csharp
int Add(int a, int b)
{
    return a + b;
}
```

Call:
```csharp
var result = Add(5,3);
```

### Expression methods
```csharp
int Add(int a, int b) => a + b;
```

---

# 🟢 7. Classes & Objects

Everything in backend revolves around classes.

## Class
```csharp
class User
{
    public string Name;
    public int Age;
}
```

## Object
```csharp
User u = new User();
u.Name = "Basit";
```

# 🟢 8. Properties (Encapsulation)

Never expose fields directly in real apps.

```csharp
class User
{
    public string Name { get; set; }
}
```

Types:

### Read only
```csharp
public string Name { get; }
```

### Private set
```csharp
public string Name { get; private set; }
```

### Computed property
```csharp
public int Age => DateTime.Now.Year - BirthYear;
```

---

# 🟢 9. Constructors

Used to initialize object.

```csharp
class User
{
    public string Name;

    public User(string name)
    {
        Name = name;
    }
}

User u = new User("Abdul");
```

Constructor runs when object created.

---

# 🟢 10. OOP Principles in C#

## Inheritance
```csharp
class Animal
{
    public void Eat(){}
}

class Dog : Animal {}
```

## Polymorphism
```csharp
class Animal
{
    public virtual void Speak(){}
}

class Dog : Animal
{
    public override void Speak(){}
}
```

## Encapsulation
Hide internal logic, expose only needed data.

## Abstraction
Use interfaces/abstract classes.

---

# 🟢 11. Interfaces

Defines contract.

```csharp
public interface IPaymentService
{
    void Pay();
}
```

Implementation:
```csharp
class StripePayment : IPaymentService
{
    public void Pay(){}
}
```

Why important?
- Loose coupling
- Replace implementations easily
- Unit testing
- Clean architecture

Used everywhere in real backend.

---

# 🟢 12. Collections in C#

## List
```csharp
List<int> nums = new List<int>();
nums.Add(1);
```

## Dictionary
```csharp
Dictionary<int,string> users = new();
users.Add(1,"Basit");
```

## Important collections
- List<T>
- Dictionary<K,V>
- HashSet<T>
- Queue<T>
- Stack<T>

Used everywhere.

---

# 🟢 13. LINQ

LINQ = query data easily.

```csharp
var result = users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .ToList();
```

Used for:
- Filtering
- Sorting
- Transforming data
- DB queries (EF Core)

Important concepts:
- Select
- Where
- OrderBy
- First
- Any
- Count

---

# 🟢 14. Exception Handling

```csharp
try
{
}
catch(Exception ex)
{
}
finally
{
}
```

Best practices:
- Log errors
- Do not swallow exceptions
- Use global exception middleware in APIs
- Use a step wise logging and a supporting document for checking what went wrong directly from logs

---

# 🟢 15. Async/Await

Why async?
- Non blocking
- Handles many users
- Scalable APIs

Example:
```csharp
public async Task<string> GetData()
{
    await Task.Delay(1000);
    return "done";
}
```

Call:
```csharp
await GetData();
```

Never do:
- .Result
- .Wait()

---

# 🟢 16. Dependency Injection 

Used everywhere in ASP.NET.

```csharp
class OrderService
{
    private readonly IPaymentService _payment;

    public OrderService(IPaymentService payment)
    {
        _payment = payment;
    }
}
```

Why?
- Loose coupling
- Testable
- Replace services easily
- Clean architecture

---

# 🟢 17. Records (Modern C#)

Used for DTOs.

```csharp
public record User(string Name, int Age);
```

Benefits:
- Immutable
- Cleaner code
- Value equality
- Perfect for APIs

---

# 🟢 18. Generics

Reusable code.

```csharp
class Repo<T>
{
    public void Add(T item){}
}
```

Used everywhere:
- List<T>
- Task<T>
- Dictionary<T>

---

# 🟢 19. Extension Methods

Add methods to existing classes.

```csharp
public static class StringExt
{
    public static bool IsEmail(this string s)
    {
        return s.Contains("@");
    }
}
```

---

# 🟢 20. Delegates & Events

Used for event-based programming.

```csharp
public delegate void Notify();
public event Notify OnOrder;
```

Used in:
- Messaging systems
- Background jobs
- Domain events

---

# 🟢 21. Memory Basics

Stack:
- value types
- fast

Heap:
- objects
- managed by GC

CLR garbage collector cleans unused objects.

---

# 🟢 22. IDisposable & using

```csharp
using(var conn = new SqlConnection())
{
}
```

Ensures resource cleanup.

Used for:
- DB connections
- Files
- Network streams

---

# New Concepts to me — Records, Task, Generics, Thread, Delegate, Async/Await

# 1. RECORDS (Modern C# Data Models)

## Why records exist

In backend systems we constantly pass data:
- DTOs  
- API responses  
- Event messages  
- Read-only models  
- Value objects  

Traditional classes are:
- mutable  
- compared by memory reference  
- can change accidentally  
- not safe for concurrency  

But most backend data should be:
- immutable  
- safe  
- value comparable  
- predictable  

That is why **records** were introduced.

---

## Record vs Class

### Class (reference equality)

```csharp
class User  
{  
    public string Name { get; set; }  
}  

var u1 = new User { Name = "Abdul" };  
var u2 = new User { Name = "Abdul" };  

Console.WriteLine(u1 == u2);  
// Output: false (different memory)
```

Even though data is same → objects are not equal.

---

### Record (value equality)

```csharp
public record User(string Name);  

var u1 = new User("Abdul");  
var u2 = new User("Abdul");  

Console.WriteLine(u1 == u2);  
// Output: true
```

Record compares **data**, not memory.

---

## Records are immutable

```csharp
public record User(string Name, int Age);  

var user = new User("Abdul", 25);  

user.Name = "Ali";   // ❌ not allowed
```

This prevents accidental changes.

---

## Copy safely using `with`

```csharp
var updated = user with { Age = 26 };
```

This creates:
- new object  
- original unchanged  
- thread safe  
- predictable  

Very useful in:
- APIs  
- state handling  
- event systems  

---

## When to use records

**Use records for:**
- DTOs  
- API responses  
- Event data  
- Value objects  
- Read models  

**Avoid for:**
- entities with heavy business logic  
- mutable domain objects  

---

# 2. TASK (Async programming backbone)

`Task` represents an asynchronous operation.

Think like:
- Promise in JavaScript  
- Future in Java  

Backend operations take time:
- DB calls  
- API calls  
- file read  
- network  

Instead of blocking thread → use **Task**.

---

## Basic example

```csharp
public async Task<string> GetData()  
{  
    await Task.Delay(2000);  
    return "Data received";  
}

var result = await GetData();  
Console.WriteLine(result);
```

---

## Task types

### Task (no return)

```csharp
public async Task Process()  
{  
    await Task.Delay(1000);  
}
```

### Task with return

```csharp
public async Task<int> GetNumber()  
{  
    await Task.Delay(500);  
    return 10;  
}
```

---

## Running multiple tasks

```csharp
var t1 = GetData();  
var t2 = GetData();  

await Task.WhenAll(t1, t2);
```

Used heavily in:
- APIs  
- microservices  
- parallel calls  

---

# 3. GENERICS (Reusable type-safe code)

Generics avoid duplicate code.

### Without generics

```csharp
class IntBox { public int Value; }  
class StringBox { public string Value; }
```

### With generics

```csharp
class Box<T>  
{  
    public T Value { get; set; }  
}

Box<int> a = new Box<int>();  
Box<string> b = new Box<string>();
```

One class → works for all types.

---

## Generic method

```csharp
public T Echo<T>(T value)  
{  
    return value;  
}

Echo<int>(5);  
Echo<string>("hello");
```

---

## Generic constraints

```csharp
public class Repository<T> where T : class  
{
}
```

Multiple constraints:

```csharp
where T : class, new()
```

Means:
- must be reference type  
- must have empty constructor  

---

## Where generics are used in real backend

- Repository pattern  
- Services  
- Collections (List<T>)  
- Dependency injection  
- Middleware pipelines  

---

# 4. THREAD (Execution unit)

Thread = smallest execution unit.

Every program starts with:
**Main thread**

But backend needs:
- parallel work  
- background jobs  
- scalability  

---

## Creating thread

```csharp
Thread t = new Thread(() =>  
{  
    Console.WriteLine("Running in thread");  
});  

t.Start();
```

Now two threads running:
- main thread  
- new thread  

---

## Problem with manual threads

Threads are heavy:
- memory cost  
- context switching  
- hard debugging  

So modern .NET rarely uses raw threads.

Instead uses:
**Task + ThreadPool**

---

# 5. THREADPOOL

ThreadPool = pool of reusable threads.

Instead of creating new thread every time:  
Runtime reuses existing threads.

Flow:

Task created → assigned to threadpool thread  
Work done → thread returned to pool  

Benefits:
- fast  
- memory efficient  
- scalable  
- automatic  

---

# 6. DELEGATE (Function as variable)

Delegate = reference to method.  
Allows passing function as data.

## Basic delegate

```csharp
public delegate void MyDelegate(string msg);

public void Print(string msg)  
{  
    Console.WriteLine(msg);  
}

MyDelegate d = Print;  
d("Hello");
```

---

## Built-in delegates

### Func (returns value)

```csharp
Func<int, int> square = x => x * x;  
Console.WriteLine(square(5));
```

### Action (no return)

```csharp
Action<string> print = msg => Console.WriteLine(msg);  
print("Hi");
```

Used in:
- LINQ  
- events  
- middleware  
- callbacks  

---

# 7. ASYNC / AWAIT

Async makes code:
- non blocking  
- scalable  
- efficient  

---

## Without async (bad)

```csharp
var data = GetData();
```

Thread blocked → server slow.

---

## With async

```csharp
var data = await GetData();
```

Thread released → handles other requests.

---

## API example

```csharp
public async Task<string> CallApi()  
{  
    HttpClient client = new HttpClient();  
    return await client.GetStringAsync("https://example.com");  
}
```

---

## Important rules

Rule 1: async returns Task  
- async Task  
- async Task<T>  

Rule 2: use await inside async

```csharp
await SomeMethod();
```

Rule 3: avoid `.Result` or `.Wait()`

Bad:

```csharp
var data = GetData().Result;
```

Can deadlock.

---

# 8. How everything connects

When request hits API:

Controller  
↓  
Service (async)  
↓  
Repository (generic)  
↓  
DB call (task)  
↓  
DTO (record)  

Example:

```csharp
public record UserDto(int Id, string Name);

public async Task<UserDto> GetUser(int id)  
{  
    var user = await _repo.GetById(id);  
    return new UserDto(user.Id, user.Name);  
}
```

Inside:
- Task handles async DB call  
- record used for DTO  
- generics used in repository  
- delegates used in LINQ  
- threadpool managed by runtime  

Everything works together.



# C# Notes

---

## Value Types
Stored on the **stack**. Holds the value directly.

```csharp
int age = 26;
double price = 19.99;
char grade = 'A';
bool isCompleted = true;
```

---

## Reference Types
Stored on the **heap**. Variable holds a pointer to the data.

```csharp
string message = "Hi";
object genericObj = 42;   // Parent of all types, stores anything
```

---

## Type Conversion / Casting

```csharp
double raw = 4.7;
int exact = (int)raw;             // explicit cast — truncates, does not round
string stringForm = exact.ToString();
```

---

## Conditionals

```csharp
// if / else if / else
if (age == 18) {
    Console.WriteLine("Its 18");
} else if (age > 18) {
    Console.WriteLine("Its > 18");
} else {
    Console.WriteLine("Its < 18");
}

// switch
string status = "Pending";
switch (status) {
    case "Active":
        Console.WriteLine("Its Active");
        break;
    case "Pending":
        Console.WriteLine("Its Pending");
        break;
    default:
        Console.WriteLine("Unknown");
        break;
}
```

---

## Looping

```csharp
// for
for (int i = 0; i < 10; i++) {
    Console.WriteLine($"Value: {i}");
}

// foreach
string[] tools = { "A", "B", "C" };
foreach (string tool in tools) {
    Console.WriteLine(tool);
}

// while
int counter = 0;
while (counter < 3) {
    counter++;
}

// do-while  (always runs at least once)
int count = 10;
do {
    count--;
} while (count > 5);

// continue / break
for (int j = 0; j < 10; j++) {
    if (j == 3) continue;  // skips rest of this iteration, jumps to j=4
    if (j == 7) break;     // exits the loop entirely
}
```

---

## Exception Handling

```csharp
try {
    int a = 0;
    int b = 10 / a;
} catch (DivideByZeroException ex) {
    Console.WriteLine(ex.Message);
} finally {
    Console.WriteLine("Cleaning up");  // always runs, success or failure
}
```

---

## Collections

### Sequential

```csharp
// List<T> — dynamic array, fast index access
List<string> listOne = ["A", "B"];
listOne.Add("C");
listOne.Insert(1, "a");
listOne.Remove("a");
bool hasA = listOne.Contains("A");

// LinkedList<T> — doubly linked list, fast insert/delete at boundaries
LinkedList<int> linkedList = new LinkedList<int>();
LinkedListNode<int> firstNode = linkedList.AddFirst(10);
linkedList.AddLast(30);
linkedList.AddAfter(firstNode, 20);
linkedList.RemoveLast();
```

### Key-Value

```csharp
// Dictionary<TKey, TValue> — fast unique key lookup via hash table
Dictionary<int, string> userDict = new Dictionary<int, string> { { 1, "Abc" } };
userDict.Add(2, "def");
bool hasKey = userDict.ContainsKey(1);
bool success = userDict.TryGetValue(2, out string? username); // safe get, no crash if missing
userDict.Remove(1);

// SortedDictionary<TKey, TValue> — same as Dictionary but always sorted by key
SortedDictionary<string, int> sortedDict = new SortedDictionary<string, int>();
sortedDict.Add("abc", 5);
sortedDict.Add("def", 1);
sortedDict.Remove("abc");
```

### Special Behaviour

```csharp
// Queue<T> — FIFO (first in, first out)
Queue<string> aQueue = new Queue<string>();
aQueue.Enqueue("Doc1");
string nextDoc = aQueue.Peek();       // look at front without removing
string processed = aQueue.Dequeue(); // remove and return front item

// Stack<T> — LIFO (last in, first out)
Stack<string> aStack = new Stack<string>();
aStack.Push("Act1");
string topAct = aStack.Peek();  // look at top without removing
string undo = aStack.Pop();     // remove and return top item
```

### Set

```csharp
// HashSet<T> — unordered pool of UNIQUE elements
HashSet<int> mySet = new HashSet<int>() { 1, 2, 3 };
bool addedNew = mySet.Add(2);  // false — 2 already exists

HashSet<int> secondarySet = new HashSet<int>() { 3, 4, 5 };
mySet.UnionWith(secondarySet);  // mySet is now {1,2,3,4,5}

List<int> filterList = new List<int>() { 2, 3, 9 };
mySet.IntersectWith(filterList);  // mySet is now {2,3} — only matching elements
```

### Concurrent (Thread-Safe)

```csharp
// ConcurrentDictionary<TKey, TValue> — thread-safe lookup table
ConcurrentDictionary<int, string> secureDict = new ConcurrentDictionary<int, string>();
string activeUser = secureDict.GetOrAdd(1, "Default");           // fetch key 1, or insert if missing
bool updated = secureDict.TryUpdate(1, "NewValue", "Default");   // update only if current value matches

// ConcurrentQueue<T> — thread-safe FIFO
ConcurrentQueue<int> secureQueue = new ConcurrentQueue<int>();
secureQueue.Enqueue(100);
bool fetched = secureQueue.TryDequeue(out int queueResult);  // safe dequeue, no crash

// ConcurrentBag<T> — thread-safe unordered container, optimised for same-thread reuse
ConcurrentBag<int> secureBag = new ConcurrentBag<int>();
secureBag.Add(500);
bool taken = secureBag.TryTake(out int bagResult);  // safe extract
```

### Immutable

```csharp
// ImmutableList<T> — read-only snapshot; any "change" returns a NEW list
ImmutableList<string> originalList = ImmutableList.Create("X", "Y");
ImmutableList<string> expandedList = originalList.Add("Z");  // originalList is untouched
```

---

## Classes / Methods / Objects

```csharp
using System;
using System.Collections.Generic;

namespace LearningDotNet;

public class BankAccount
{
    public string Name { get; set; }
    public double Balance { get; private set; }   // only this class can set it
    private List<string> TransHistory { get; set; }

    public BankAccount(string name, double balance)
    {
        Name = name;
        Balance = balance;
        TransHistory = new List<string>();
        TransHistory.Add($"Account created with Balance: ${Balance}");
    }

    public void Deposit(double amount)
    {
        Balance += amount;
        TransHistory.Add($"Deposited: ${amount}");
    }

    public void PrintStmt()
    {
        Console.WriteLine($"Stmt for {Name}");
        foreach (string trans in TransHistory)
            Console.WriteLine(trans);
        Console.WriteLine($"Current Balance: {Balance}");
    }
}

class Program
{
    static void Main(string[] args)
    {
        BankAccount account1 = new BankAccount("Alice", 500.00);
        BankAccount account2 = new BankAccount("Bob", 1500.00);

        account1.Deposit(250.50);
        account2.Deposit(250.50);

        account1.PrintStmt();
        account2.PrintStmt();
    }
}
```

---

## Inheritance

- `virtual` on a base method = child classes **may** override it
- `override` on a child method = this replaces the parent's version
- `protected` = accessible by this class and any child class (not public)
- `base(...)` = call the parent constructor from the child constructor

```csharp
// Base class
public class BankAccount
{
    public string Name { get; set; }
    protected double Balance { get; set; }          // protected so children can access
    protected List<string> TransHistory { get; set; }

    public BankAccount(string name, double balance)
    {
        Name = name;
        Balance = balance;
        TransHistory = new List<string>();
        TransHistory.Add($"Account created with Balance: ${Balance}");
    }

    public virtual void Withdraw(double amount)     // virtual = children can override
    {
        if (amount <= Balance) {
            Balance -= amount;
            TransHistory.Add($"Withdrew: ${amount}");
        } else {
            TransHistory.Add($"Withdrawal failed (Insufficient funds): ${amount}");
        }
    }

    public void Deposit(double amount)
    {
        Balance += amount;
        TransHistory.Add($"Deposited: ${amount}");
    }

    public void PrintStmt()
    {
        Console.WriteLine($"Stmt for {Name}");
        foreach (string trans in TransHistory)
            Console.WriteLine(trans);
        Console.WriteLine($"Current Balance: {Balance}");
    }
}

// Child class — inherits everything from BankAccount
public class SavingsAccount : BankAccount
{
    public double InterestRate { get; set; }

    public SavingsAccount(string name, double startingBalance, double interestRate)
        : base(name, startingBalance)   // passes name + balance up to BankAccount constructor
    {
        InterestRate = interestRate;
    }

    public void ApplyInterest()
    {
        double interest = Balance * InterestRate;
        Balance += interest;
        TransHistory.Add($"Interest applied ({InterestRate * 100}%): ${interest}");
    }
}

// Child class — overrides Withdraw to add a transaction fee
public class CheckingAccount : BankAccount
{
    private const double TransFee = 1.50;

    public CheckingAccount(string name, double startingBalance)
        : base(name, startingBalance) { }

    public override void Withdraw(double amount)   // override = replace parent's version
    {
        double totalAmount = amount + TransFee;
        if (totalAmount <= Balance) {
            Balance -= totalAmount;
            TransHistory.Add($"Withdrew: ${amount} (Fee: ${TransFee})");
        } else {
            TransHistory.Add($"Withdrawal failed: Overdraft limit exceeded.");
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        SavingsAccount savings = new SavingsAccount("Alice", 1000.00, 0.05);
        savings.ApplyInterest();
        savings.PrintStmt();

        CheckingAccount checking = new CheckingAccount("Bob", 500.00);
        checking.Withdraw(100.00);  // uses overridden logic with fee
        checking.PrintStmt();

        // Polymorphism — treat different account types uniformly
        // At runtime, each object's own Withdraw is called (not the base one)
        List<BankAccount> bankVault = new List<BankAccount>();
        bankVault.Add(new SavingsAccount("Alice", 1000.00, 0.05));
        bankVault.Add(new CheckingAccount("Bob", 500.00));

        foreach (BankAccount account in bankVault)
        {
            account.Withdraw(100.00);
            account.PrintStmt();
        }
    }
}
```

---

## Interface

An interface is a **contract**. It says "any class that implements me MUST provide these methods." It has no implementation of its own — just the signatures.

Why use it: you can write code against the interface type, not the concrete class. Swap one implementation for another without changing any calling code.

```csharp
using System;

namespace LearningDotNet;

public interface ILoggable
{
    string GetLogId();
    void LogTransaction(string message);
}

// Inherits from BankAccount AND implements ILoggable contract
public class PremiumCheckingAccount : BankAccount, ILoggable
{
    public string TaxId { get; set; }

    public PremiumCheckingAccount(string name, double startingBalance, string taxId)
        : base(name, startingBalance)
    {
        TaxId = taxId;
    }

    public string GetLogId()
    {
        return $"ACC-{TaxId}-{Name.ToUpper()}";
    }

    public void LogTransaction(string message)
    {
        Console.WriteLine($"[AUDIT LOG] {GetLogId()}: {message}");
    }
}
```

---

## LINQ

LINQ lets you query, filter, sort, and transform collections using readable, SQL-like method chains directly in C#.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class Customer
{
    public string Name { get; set; }
    public string City { get; set; }
    public double Balance { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        List<Customer> customers = new List<Customer>
        {
            new Customer { Name = "Alice",   City = "New York", Balance = 1200.50 },
            new Customer { Name = "Bob",     City = "London",   Balance = 450.00  },
            new Customer { Name = "Charlie", City = "New York", Balance = 89.00   },
            new Customer { Name = "Diana",   City = "Tokyo",    Balance = 4500.00 },
            new Customer { Name = "Evan",    City = "London",   Balance = 750.00  }
        };

        // Where() — filter by condition
        var highNetWorth = customers.Where(c => c.Balance > 1000);

        // Select() — transform each element into a new shape
        List<string> NYCnames = customers
            .Where(c => c.City == "New York")
            .Select(c => c.Name)
            .ToList();

        // OrderBy / OrderByDescending — sort
        var sortedByBalance = customers.OrderByDescending(c => c.Balance);

        // FirstOrDefault() — first match, or null if none found
        Customer? richLondoner = customers
            .FirstOrDefault(c => c.City == "London" && c.Balance > 500);

        // Any() — true if at least one element matches
        bool hasLowBalance = customers.Any(c => c.Balance < 100);

        // All() — true only if EVERY element matches
        bool everyoneHasMoney = customers.All(c => c.Balance > 0);

        // Aggregate functions
        double totalVaultCash = customers.Sum(c => c.Balance);
        double highestBalance = customers.Max(c => c.Balance);

        Console.WriteLine($"Total Bank Deposits: ${totalVaultCash}");
        Console.WriteLine("Customers with over $1000 balance:");
        foreach (var c in highNetWorth)
        {
            Console.WriteLine($"{c.Name}: ${c.Balance}");
            // The $ prefix is string interpolation — it lets you embed
            // expressions inside {} directly in the string
        }
    }
}
```

---

## Asynchronous Programming (async / await)

**Why it exists:** slow operations (API calls, file saves, DB queries) normally block the thread — the whole app freezes waiting. Async lets the thread go do other work while waiting, then come back when the result is ready.

- `Task` / `Task<T>` — represents an operation that will complete in the future
- `async` — marks a method as containing background work
- `await` — pauses THIS method until the task finishes, without blocking anything else

```csharp
using System;
using System.Threading.Tasks;

class AsyncDemo
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("[1] User clicks 'Process Payment'.");
        string receipt = await FetchReceiptFromServerAsync("TXN-99823");
        Console.WriteLine($"[3] Payment finalised! Receipt: {receipt}");
    }

    public static async Task<string> FetchReceiptFromServerAsync(string transId)
    {
        Console.WriteLine($"[2] Contacting server for {transId}...");
        await Task.Delay(2000);  // simulates a 2 second network call
        return $"SUCCESS_PAID_CONFIRMED_ID_{transId}";
    }
}
```

**What happens step by step:**
1. Prints `[1]`
2. Calls `FetchReceiptFromServerAsync` → prints `[2]`
3. Hits `await Task.Delay(2000)` — thread is released to do other work
4. After 2 seconds, resumes where it left off
5. Returns the string, prints `[3]`

---

## Doubts to revisit

- `virtual` vs `static`
- `TryGetValue` — safe dictionary access pattern

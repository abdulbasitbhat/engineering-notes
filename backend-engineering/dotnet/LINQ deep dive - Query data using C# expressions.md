# LINQ deep dive — Query data using C# expressions

## 1. LINQ (Language Integrated Query)

### What is LINQ?

LINQ is a set of language features and APIs in .NET that allow you to query collections, databases, XML, and other data sources using a uniform, expressive syntax directly in C#. It bridges the gap between objects and data.

### Two Syntax Styles

**Query Syntax** (SQL-like):
```csharp
var result = from n in numbers
             where n > 5
             orderby n
             select n * 2;
```

**Method Syntax** (Lambda-based — more common):
```csharp
var result = numbers
    .Where(n => n > 5)
    .OrderBy(n => n)
    .Select(n => n * 2);
```

Both compile to the same IL. Method syntax is preferred in most modern codebases.

---

### Core LINQ Methods

#### Filtering
| Method | Description |
|---|---|
| `Where(predicate)` | Filter elements matching condition |
| `OfType<T>()` | Filter by type |
| `Distinct()` | Remove duplicates |
| `DistinctBy(keySelector)` | Distinct by a key (.NET 6+) |

```csharp
var adults = people.Where(p => p.Age >= 18);
var unique = items.DistinctBy(x => x.Id);
```

#### Projection
| Method | Description |
|---|---|
| `Select(selector)` | Transform each element |
| `SelectMany(selector)` | Flatten nested collections |

```csharp
var names = people.Select(p => p.Name);
var allTags = posts.SelectMany(p => p.Tags);
```

#### Ordering
| Method | Description |
|---|---|
| `OrderBy(keySelector)` | Ascending order |
| `OrderByDescending(keySelector)` | Descending order |
| `ThenBy(keySelector)` | Secondary sort |

```csharp
var sorted = employees
    .OrderBy(e => e.Department)
    .ThenByDescending(e => e.Salary);
```

#### Aggregation
| Method | Description |
|---|---|
| `Count()` / `Count(predicate)` | Element count |
| `Sum(selector)` | Sum |
| `Average(selector)` | Average |
| `Min()` / `Max()` | Min/Max value |
| `Aggregate(func)` | Custom fold/reduce |

```csharp
var total = orders.Sum(o => o.Amount);
var oldest = people.Max(p => p.Age);
var product = numbers.Aggregate((acc, x) => acc * x);
```

#### Element Access
| Method | Description |
|---|---|
| `First()` / `FirstOrDefault()` | First element (throws or returns default) |
| `Single()` / `SingleOrDefault()` | Exactly one element |
| `Last()` / `LastOrDefault()` | Last element |
| `ElementAt(index)` | By index |

```csharp
var first = items.FirstOrDefault(x => x.IsActive);
// Returns null (or default) if not found — no exception
```

#### Quantifiers
| Method | Description |
|---|---|
| `Any(predicate)` | True if any match |
| `All(predicate)` | True if all match |
| `Contains(value)` | True if value exists |

```csharp
bool hasAdmin = users.Any(u => u.Role == "Admin");
bool allPaid = invoices.All(i => i.IsPaid);
```

#### Grouping & Joining
```csharp
// GroupBy
var byDept = employees.GroupBy(e => e.Department);
foreach (var group in byDept)
{
    Console.WriteLine($"{group.Key}: {group.Count()} employees");
}

// Join (inner)
var result = orders.Join(
    customers,
    o => o.CustomerId,
    c => c.Id,
    (o, c) => new { o.OrderId, c.Name }
);

// GroupJoin (left join equivalent)
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, orders) => new { c.Name, Orders = orders }
);
```

#### Set Operations
```csharp
var union = a.Union(b);
var intersect = a.Intersect(b);
var except = a.Except(b);   // Elements in a but not b
```

#### Partitioning

| Method | Description |
|---|---|
| `Skip(n)` | Skip first n elements |
| `Take(n)` | Take first n elements |
| `SkipLast(n)` | Skip last n elements (.NET Core+) |
| `TakeLast(n)` | Take last n elements (.NET Core+) |
| `SkipWhile(predicate)` | Skip while condition is true, then yield rest |
| `TakeWhile(predicate)` | Take while condition is true, then stop |
| `Chunk(size)` | Split into fixed-size arrays (.NET 6+) |

```csharp
// Pagination
int page = 3, pageSize = 10;
var paged = items.Skip((page - 1) * pageSize).Take(pageSize);

// SkipWhile / TakeWhile — condition-based, not count-based
// Stops skipping/taking as soon as condition breaks — even if later items would match
var nums = new[] { 1, 2, 3, 8, 1, 2 };
nums.TakeWhile(n => n < 5);  // → 1, 2, 3  (stops at 8, never reaches the trailing 1,2)
nums.SkipWhile(n => n < 5);  // → 8, 1, 2  (skips up to 8, then yields everything)

// TakeLast / SkipLast
var lastThree  = items.TakeLast(3);   // Last 3 elements
var allButLast = items.SkipLast(1);   // Everything except the last

// Chunk — batching / bulk processing
foreach (var batch in items.Chunk(100))
{
    await ProcessBatchAsync(batch);  // batch is T[], max 100 items
}
// Last chunk may be smaller than the chunk size — that's expected
```

> **Gotcha:** `SkipWhile`/`TakeWhile` are order-sensitive and not the same as `Where`. They operate on the sequence linearly and stop as soon as the predicate breaks — they do not scan the whole collection.

---

### Deferred vs Immediate Execution

#### Deferred (Lazy) Execution

The query definition is stored, not run. Execution happens only when you iterate (foreach, ToList, etc.).

```csharp
var query = numbers.Where(n => n > 5);  // Nothing runs here — just builds an iterator
foreach (var n in query) { ... }         // Runs HERE, one element at a time
```

Deferred operators (return `IEnumerable<T>`):
`Where`, `Select`, `SelectMany`, `OrderBy`, `GroupBy`, `Skip`, `Take`, `TakeWhile`, `SkipWhile`, `Chunk`, `Distinct`, `Union`, `Concat`, etc.

#### Immediate Execution

Forces the query to run right now and materialises the result into memory.

```csharp
var list  = query.ToList();                    // List<T>
var array = query.ToArray();                   // T[]
var dict  = query.ToDictionary(x => x.Id);    // Dictionary<K,V>
var set   = query.ToHashSet();                 // HashSet<T>
var count = query.Count();                     // int
var first = query.First();                     // T
var any   = query.Any();                       // bool
```

#### Why This Matters — Key Scenarios

**1. Multiple enumeration re-executes the query**
```csharp
var query = GetExpensiveData().Where(x => x.IsActive);

var count = query.Count();       // Runs query once
var list  = query.ToList();      // Runs query AGAIN

// Fix: materialise once
var data  = query.ToList();
var count = data.Count;
var first = data.First();
```

**2. Captured variable trap**
```csharp
int threshold = 5;
var query = numbers.Where(n => n > threshold);  // Captures reference to threshold

threshold = 100;
query.ToList();  // Uses threshold = 100, not 5!

// Fix: materialise immediately if you want the value at definition time
var snapshot = numbers.Where(n => n > threshold).ToList();
```

**3. Deferred query over a modified collection**
```csharp
var list = new List<int> { 1, 2, 3, 4, 5 };
var query = list.Where(n => n > 2);  // Deferred

list.Add(10);          // Mutate source

query.ToList();        // → 3, 4, 5, 10  (sees the new item!)
```

**4. EF Core — query executes as SQL, not in memory**
```csharp
// Deferred: builds SQL, nothing runs yet
var q = db.Orders.Where(o => o.Total > 100);

// ToListAsync() sends SQL to DB and returns results
var orders = await q.ToListAsync();

// BAD: ToList() in the middle pulls ALL rows into memory first
var orders = db.Orders.ToList().Where(o => o.Total > 100);  // ← full table scan in memory!
```

#### Quick Reference

| Returns | Execution |
|---|---|
| `IEnumerable<T>` | Deferred |
| `IQueryable<T>` | Deferred (EF Core) |
| `List<T>`, `T[]`, `Dictionary`, `HashSet` | Immediate |
| `int`, `bool`, `T` (Count, Any, First, Sum…) | Immediate |

> **Rule of thumb:** If the return type is a collection interface, it's deferred. If it's a concrete type or scalar, it's immediate.

---

### LINQ to Entities (EF Core)

When using Entity Framework Core, LINQ queries get translated to SQL:
```csharp
var users = await db.Users
    .Where(u => u.IsActive && u.CreatedAt > cutoff)
    .OrderBy(u => u.LastName)
    .Select(u => new { u.Id, u.Email })
    .ToListAsync();
```

- Use `AsNoTracking()` for read-only queries (perf win)
- Avoid `ToList()` mid-query — it pulls data into memory before filtering
- `Select` only what you need to avoid over-fetching

---

### Common Pitfalls

| Pitfall | Fix |
|---|---|
| Multiple enumeration of deferred query | Call `.ToList()` once |
| `First()` throwing on empty | Use `FirstOrDefault()` |
| N+1 queries in EF Core | Use `.Include()` for eager loading |
| Doing filtering in memory | Push `Where` before `ToList()` |
| `Count > 0` vs `Any()` | Prefer `Any()` — faster |

---

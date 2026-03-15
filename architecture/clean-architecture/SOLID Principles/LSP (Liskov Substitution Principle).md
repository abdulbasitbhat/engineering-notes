***"If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering the correctness of the program."***  
— Barbara Liskov, 1987

A subclass should be fully substitutable for its parent class, without breaking the program's behavior.

Intition: Its like a contract. When we implement an interfae we make promise to caller that  
"I will behave at least as well as what you expected from the base type".  
Violdation is when this is broken.

## Rectangle / Square example

Square is a subclass of Rectangle

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int Area() => Width * Height;
}

public class Square : Rectangle
{
    public override int Width
    {
        set { base.Width = value; base.Height = value; } // forces both sides equal
    }
    public override int Height
    {
        set { base.Width = value; base.Height = value; }
    }
}

Rectangle r = new Square(); // caller expects a Rectangle
r.Width = 5;
r.Height = 3;
Console.WriteLine(r.Area()); // Expected: 15, Got: 9 // Bugs and unexplainable surprises wherever used
```

We cant substitute Square for rectangle. If we need to replace parent with its child class we need to introduce if conditions.

Now if we need to use this object somewhere

```csharp
public void Resize(Rectangle r)
{
    r.Width = 5;
    r.Height = 3;

    if (r is Square s)
    {
        Console.WriteLine($"It's a Square. Area: {s.Area()}");
    }
    else
    {
        Console.WriteLine($"Expected area 15, got: {r.Area()}");
    }
}
```
The if (r is Square) is the issue. The moment you write that, you've admitted that Rectangle and Square are 
not actually substitutable — and your abstraction has leaked.
Once this happens the fix is simply a redesign

```csharp
public interface IShape
{
    int Area();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area() => Width * Height;
}

public class Square : IShape
{
    public int Side { get; set; }
    public int Area() => Side * Side;
}

// Now this method never needs to ask "what are you?". No if needed
public void PrintArea(IShape shape)
{
    Console.WriteLine($"Area: {shape.Area()}");
}
```
Each type is honest about what it is. IShape only promises Area() — and both shapes can keep that promise fully. No is, no casting, no surprises.  
Note: 
The replace of parent with its child is not actual replacement of moving code but how we 
replace in calling
```csharp
IShape shape = new Rectangle { Width = 5, Height = 3 };  //Replaced IShape with Rectangle (child)
or
IShape shape = new Square { Side = 5 };  //Replaced IShape with Square (child)

PrintArea(shape); // works identically for both, no if case neded.
```

Another Example (Behavioural)

```csharp
public interface IOrderRepository
{
    Order GetById(int id); // contract: always returns an Order or throws
}

public class CachedOrderRepository : IOrderRepository
{
    public Order GetById(int id)
    {
        return null; // returns null instead of throwing
                     // caller didn't expect this — LSP violated
    }
}
```

LSP isn't just about inheritance — it's about behavioral contracts on interfaces. 
Design interfaces around what callers need, not what implementations we want to expose.
Subtypes/implementations must not surprise the caller

Reminder
Don't lie in your interface. If you can't fully honor the contract, don't implement it — redesign the abstraction.



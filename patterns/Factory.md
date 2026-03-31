# Factory Pattern

## Idea

The **Factory Pattern** is used when you want **one place responsible for creating objects**.

Instead of writing `new` in many places, you ask a **factory** to create the object.

The client **does not need to know which concrete class is created**.

---

## Problem

Suppose we support multiple **notification types**.

Without a factory:

```
INotification notification;

if(type == "email")
    notification = new EmailNotification();
else if(type == "sms")
    notification = new SmsNotification();

notification.Send("Hello");
```
and we have to do it everywhere. Imagine if we needed an object that needed configuration. We have to do configuration on all uses.
### Problems

- Client must know every implementation
- Hard to add new types
- Tight coupling between client and classes

---

## Solution — Factory Pattern

Create a **factory class** responsible for creating objects.

Client code asks the factory instead of using `new`.

---

## Common Interface

```
public interface INotification
{
    void Send(string message);
}
```

---

## Implementations

```
public class EmailNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine("Email sent: " + message);
    }
}

public class SmsNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine("SMS sent: " + message);
    }
}
```

---

## Factory Class

```
public static class NotificationFactory
{
    public static INotification Create(string type)
    {
        if(type == "email")
            return new EmailNotification();

        if(type == "sms")
            return new SmsNotification();

        throw new Exception("Unknown notification type");
    }
}
```

---

## Client Code

```
INotification notification =
    NotificationFactory.Create("email");

notification.Send("Hello World");
```

---

## Flow

1. Client asks **Factory** for an object  
2. Factory decides **which class to create**  
3. Factory returns the object  
4. Client uses the object through the **interface**

Client never directly creates `EmailNotification` or `SmsNotification`.

---

## Benefit

If we add a new class:

```
PushNotification
```

We **only update the factory**, not the client code.

---

## One Line Definition

**Factory Pattern:**  
A design pattern where **object creation is handled by a factory class instead of the client code**. Some object creatins need a logic to happen everytime. Maybe configuration etc. We only need to do that in Factory class once. 
Otherwise user has to make those configs everwhere and know all configs even if related or not.

Used in Dyncamic-middleware-connector - Commit 09 - Logging with Serilog via Microsoft Logging using Logger Factory

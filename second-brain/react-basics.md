# React + TypeScript + JavaScript Cheat Sheet

---

# Topics

- Variables / State / Core Hooks
- Looping and Conditionals
- Data Collections and Array Filtering
- Asynchronous Operations
- Components and Props
- User Input Fields and Forms
- JavaScript Syntax Commonly Used in React
- React Concepts
- Handling Multiple Inputs in One State Object
- Callback (Passing Data Upwards)
- React Context (`useContext`)
- React Router
- useMemo
- useRef
- React Query
- TypeScript
- JavaScript Essentials

---

# Variables / State / Core Hooks

```tsx
import React, { useState } from "react";

function UserProfile() {
  const [age, setAge] = useState(25);
  const [username, setUsername] = useState("Alice");
  const [isActive, setIsActive] = useState(true);
  const [items, setItems] = useState(["Alice", "Bob"]);

  const celebrate = () => {
    setAge(prevAge => prevAge + 1);
  };

  // State is immutable
  const addItem = () => {
    setItems([...items, "John"]);
  };

  const clearItems = () => {
    setItems([]);
  };

  return (
    <div>
      <h1>
        {username} ({age})
      </h1>

      <button onClick={celebrate}>Celebrate Birthday</button>
      <button onClick={addItem}>Add</button>
      <button onClick={clearItems}>Clear</button>
    </div>
  );
}
```

### Important

- Never modify state directly.
- Always use the setter function.
- React state is immutable.

❌ Wrong

```tsx
items.push("John");
```

✅ Correct

```tsx
setItems([...items, "John"]);
```

---

# Looping and Conditionals

```tsx
function TeamList() {
  const members = ["Alice", "Bob", "John"];
  const isPremiumUser = true;

  return (
    <div>

      {/* Ternary */}
      {isPremiumUser
        ? <p>Welcome VIP User</p>
        : <p>Welcome User</p>}

      {/* Short Circuit */}
      {members.length > 0 &&
        <p>Members Loaded</p>
      }

      {/* Loop */}
      <ul>
        {members.map((member, index) => (
          <li key={index}>{member}</li>
        ))}
      </ul>

    </div>
  );
}
```

---

# Data Collections & Array Filtering

```tsx
const customers = [
  { name: "Alice", city: "London", balance: 1200 },
  { name: "Bob", city: "New York", balance: 450 },
  { name: "John", city: "London", balance: 90 }
];
```

## Filter (LINQ Where)

```tsx
const richCustomers =
    customers.filter(c => c.balance > 1000);
```

## Select (LINQ Select)

```tsx
const names =
    customers
      .filter(c => c.city === "New York")
      .map(c => c.name);
```

## Find (LINQ FirstOrDefault)

```tsx
const richLondoner =
    customers.find(c =>
      c.city === "London" &&
      c.balance > 500
    );
```

## Reduce (Sum)

```tsx
const total =
    customers.reduce(
      (sum, c) => sum + c.balance,
      0
    );
```

---

# Asynchronous Operations

```tsx
import { useEffect, useState } from "react";

function BalanceChecker() {

  const [receipt, setReceipt] = useState("No transaction");
  const [loading, setLoading] = useState(false);

  useEffect(() => {

    const fetchReceipt = async () => {

      setLoading(true);

      try {

        const response = await fetch("https://...");
        const data = await response.json();

        setReceipt(`Confirmed ID: ${data.id}`);

      } catch {

        setReceipt("Failed");

      } finally {

        setLoading(false);

      }

    };

    fetchReceipt();

  }, []);

  return (
    <div>
      {loading
        ? <p>Loading...</p>
        : <p>{receipt}</p>}
    </div>
  );

}
```

---

# Components & Props

## Child

```tsx
function BankCard({
  cardHolder,
  currentBalance
}) {
  return (
    <div>
      <h3>{cardHolder}</h3>
      <p>${currentBalance}</p>
    </div>
  );
}
```

## Parent

```tsx
function Dashboard() {
  return (
    <>
      <BankCard
        cardHolder="Alice"
        currentBalance={15000}
      />

      <BankCard
        cardHolder="Bob"
        currentBalance={25000}
      />
    </>
  );
}
```

---

# User Input Fields & Forms

```tsx
import { useState } from "react";

function UserForm() {

  const [fullName, setFullName] = useState("");
  const [accountType, setAccountType] = useState("Checking");
  const [message, setMessage] = useState("");

  const handleSubmit = (e) => {

    e.preventDefault();

    if (!fullName.trim()) {
      setMessage("Please enter a valid name");
      return;
    }

    setMessage(
      `Created for ${fullName} (${accountType})`
    );

  };

  return (

    <form onSubmit={handleSubmit}>

      <input
        value={fullName}
        onChange={e => setFullName(e.target.value)}
      />

      <select
        value={accountType}
        onChange={e => setAccountType(e.target.value)}
      >
        <option>Checking</option>
        <option>Savings</option>
      </select>

      <button>Submit</button>

      {message && <p>{message}</p>}

    </form>

  );

}
```

---

# JavaScript Syntax Commonly Used in React

## Arrow Functions

```js
const double = num => num * 2;
```

---

## Destructuring

```js
const user = {
  id: 1,
  name: "Alice",
  role: "Admin"
};

const { name, role } = user;
```

---

## Array Destructuring

```js
const coordinates = [45, 122];

const [latitude, longitude] = coordinates;
```

---

## Spread Operator

```js
const arr = [1, 2];

const newArr = [...arr, 3];
```

Objects

```js
const user = {
  name: "Bob",
  status: "Active"
};

const updated = {
  ...user,
  status: "Suspended"
};
```

---

## Template Literals

```js
const user = "Charlie";
const balance = 250;

const msg =
`Welcome ${user}. Balance: $${balance}`;
```

---

## Nullish Coalescing

```js
const score = 0;

score || 10
// 10

score ?? 10
// 0
```

---

## Object Shorthand

```js
const name = "Alice";
const email = "a@test.com";

const user = {
  name,
  email
};
```

---

# React Concepts

## Why const?

```js
const age = 25;

age = 26;
```

❌ Error

React expects state updates through setters.

```tsx
setAge(26);
```

---

# Handling Multiple Inputs Using One State Object

```tsx
const [formData, setFormData] = useState({
  fullName: "",
  email: "",
  role: "",
  subscribe: false
});

const handleChange = e => {

  const {
    name,
    value,
    checked,
    type
  } = e.target;

  setFormData(prev => ({
    ...prev,
    [name]:
      type === "checkbox"
        ? checked
        : value
  }));

};
```

---

# Callback (Passing Data Upwards)

## Child

```tsx
function Child({ onSubmit }) {

  const [text, setText] = useState("");

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />

      <button
        onClick={() => onSubmit(text)}
      >
        Send
      </button>
    </>
  );

}
```

## Parent

```tsx
function Parent() {

  const handleChildData = value => {
    console.log(value);
  };

  return (
    <Child
      onSubmit={handleChildData}
    />
  );

}
```

---

# React Context

```tsx
const UserContext = createContext(null);
```

Provider

```tsx
<UserContext.Provider
  value={{
    user,
    setUser
  }}
>
  {children}
</UserContext.Provider>
```

Consumer

```tsx
const {
  user,
  setUser
} = useContext(UserContext);
```

Use Context when many components need the same data.

Examples

- Logged in user
- Theme
- Language

---

# React Router

Install

```bash
npm install react-router-dom
```

Navigation

```tsx
<Link to="/">Home</Link>
```

Programmatic Navigation

```tsx
const navigate = useNavigate();

navigate("/");
```

Route Parameter

```tsx
const { id } = useParams();
```

Nested Routes

```tsx
<Outlet />
```

Think of `<Outlet />` as a placeholder where child pages appear.

---

# useMemo

```tsx
const filteredProducts = useMemo(() => {

  return products.filter(
    p => p.includes(search)
  );

}, [search]);
```

Use when:

- expensive calculations
- filtering
- sorting
- large lists

---

# useRef

Access DOM

```tsx
const inputRef = useRef(null);

inputRef.current.focus();
```

Store mutable values

```tsx
const timerRef = useRef(null);
```

Changing a ref does **not** trigger a re-render.

---

# React Query

Installation

```bash
npm install @tanstack/react-query
```

Provider

```tsx
<QueryClientProvider client={queryClient}>
    <App />
</QueryClientProvider>
```

Query

```tsx
const {
  data,
  isLoading,
  error
} = useQuery({

  queryKey: ["users"],

  queryFn: fetchUsers,

  staleTime: 10000

});
```

Benefits

- caching
- background refetch
- retries
- loading states
- error states

---

# TypeScript

## Primitive Types

```ts
let age: number = 30;
let name: string = "Alice";
let premium: boolean = true;
```

---

## Arrays

```ts
let nums: number[] = [1,2,3];

let names: string[] = ["A","B"];
```

---

## Interface

```ts
interface User {

  id: number;

  readonly username: string;

  email: string;

  phone?: string;

}
```

---

## Type

```ts
type Status =
    "Active"
  | "Pending"
  | "Suspended";
```

---

## Functions

```ts
function tax(
  amount:number,
  rate:number
):number {

  return amount * rate;

}
```

---

## Generics

```ts
function wrap<T>(
  item:T
):T[] {

  return [item];

}
```

---

## React Example

```tsx
interface Task {

  id:string;

  title:string;

}

const [tasks, setTasks] =
useState<Task[]>([]);
```

---

## Optional Chaining

```ts
employee.department?.code
```

---

## Null Coalescing

```ts
employee.department?.code
    ?? "GENERAL";
```

---

## Type Assertion

```ts
const len =
(rawData as string).length;
```

---

# JavaScript Essentials

## Functions

```js
function add(a, b) {
  return a + b;
}
```

Arrow

```js
const add = (a, b) => a + b;
```

---

## Immutability

Arrays

```js
const newArray = [
  ...oldArray,
  item
];
```

Objects

```js
const updated = {
  ...user,
  role: "Admin"
};
```

---

## Dynamic Properties

```js
const updated = {

  ...user,

  [fieldName]: value

};
```

---

## Destructuring

```js
const {
  name,
  role
} = user;
```

---

## Useful Array Methods

### map()

Transforms every item.

Equivalent to LINQ `Select()`.

```js
const titles =
products.map(p => p.title);
```

---

### filter()

Removes items.

Equivalent to LINQ `Where()`.

```js
const remaining =
products.filter(
  p => p.id !== 2
);
```

---

### find()

Returns one matching item.

Equivalent to LINQ `FirstOrDefault()`.

```js
const cheap =
products.find(
  p => p.cost < 50
);
```

---

### reduce()

Aggregates values.

Equivalent to LINQ `Sum()`.

```js
const total =
products.reduce(
  (sum, p) => sum + p.cost,
  0
);
```

---

# React Rules to Remember

✅ Never mutate state directly.

✅ Always use state setters.

✅ Every list item needs a unique `key`.

✅ Effects belong inside `useEffect`.

✅ Derived values should use `useMemo` if expensive.

✅ Use `useRef` for DOM access or mutable values.

✅ React Query manages **server state**.

✅ `useState` manages **client state**.

✅ Context shares global state.

✅ Props pass data downward.

✅ Callbacks send data upward.

✅ TypeScript catches errors before runtime.

# ⚛️ React Fundamentals — Component-Based UI Development

# 🧠 1. What is React?

React is a **component-based UI library** used to build interactive user interfaces.

Instead of writing large HTML pages, React lets you break UI into **small reusable components**.

Think:
UI = Tree of components

Example:
App → Navbar → Sidebar → Content → Footer

Each part is a component.

---

# 🧱 2. Component-Based Architecture

React apps are built using components.

A component is:
A reusable piece of UI + logic.

Example:
- Button
- Card
- Modal
- Navbar
- Product item
- Entire page

Each component:
- Has its own logic
- Has its own UI
- Can be reused anywhere

This makes React scalable.

---

# 🧩 3. Types of Components

## Functional Components (Modern standard)

Most used today.

Example:

    function Welcome() {
        return <h1>Hello</h1>;
    }

Or arrow function:

    const Welcome = () => {
        return <h1>Hello</h1>;
    };

React today is fully based on functional components + hooks.

---

## Class Components (Legacy)

Older pattern.

    class Welcome extends React.Component {
        render() {
            return <h1>Hello</h1>;
        }
    }

Rarely used in modern React.

---

# 🧬 4. How Component-Based UI Works

React builds UI as a **tree**.

Example structure:

App  
 ├── Navbar  
 ├── Sidebar  
 ├── Dashboard  
 │    ├── Card  
 │    ├── Chart  
 │    └── Table  
 └── Footer  

Each block = component.

Benefits:
- Reusability
- Separation of concerns
- Easy testing
- Easy scaling

---

# 🎯 5. JSX (UI Syntax)

React uses JSX = JavaScript + HTML.

Example:

    const App = () => {
        return (
            <div>
                <h1>Hello</h1>
                <p>Welcome</p>
            </div>
        );
    };

JSX allows writing UI inside JS.

Rules:
- One parent element
- Use className instead of class
- Use {} for JS inside JSX

Example:

    const name = "Basit";

    return <h1>Hello {name}</h1>;

---

# 🔗 6. Props (Passing Data Between Components)

Props = inputs to component.

Used to pass data from parent → child.

Example:

Parent:

    <User name="Basit" age={25} />

Child:

    const User = (props) => {
        return <h1>{props.name}</h1>;
    };

Or destructuring:

    const User = ({ name, age }) => {
        return <h1>{name}</h1>;
    };

Props are:
- Read-only
- Immutable
- Used for configuration

---

# 🧠 7. State (Component Memory)

State = data that changes over time.

Used when UI must update dynamically.

Example:

    import { useState } from "react";

    const Counter = () => {
        const [count, setCount] = useState(0);

        return (
            <div>
                <h1>{count}</h1>
                <button onClick={() => setCount(count + 1)}>+</button>
            </div>
        );
    };

When state changes → component re-renders.

---

# 🔄 8. Re-rendering in React

React re-renders when:
- State changes
- Props change
- Parent re-renders

React updates only changed parts using Virtual DOM.

Flow:
State change → Virtual DOM diff → Update real DOM

This makes React fast.

---

# 🧭 9. Component Communication

## Parent → Child
Using props.

## Child → Parent
Using callback functions.

Example:

Parent:

    const App = () => {
        const handleClick = () => {
            console.log("Clicked");
        };

        return <Button onClick={handleClick} />;
    };

Child:

    const Button = ({ onClick }) => {
        return <button onClick={onClick}>Click</button>;
    };

---

# 🧱 10. Reusable Component Design

Good React engineers build reusable components.

Example: Bad

    <button style={{color:'red'}}>Save</button>

Example: Good

    <Button variant="primary" text="Save" />

Reusable components:
- Reduce duplication
- Improve consistency
- Easier maintenance
- Scalable design systems

---

# 🪝 11. Hooks (Core of Modern React)

Hooks allow logic inside functional components.

Common hooks:
- useState → state
- useEffect → lifecycle
- useRef → DOM reference
- useMemo → performance
- useCallback → function memoization
- useContext → global state

Example:

    import { useEffect } from "react";

    useEffect(() => {
        console.log("Mounted");
    }, []);

Runs when component mounts.

---

# 🌍 12. Component Lifecycle (Functional)

Mount → component created  
Update → state/props change  
Unmount → component removed  

Using useEffect:

Mount:

    useEffect(() => {
        console.log("Mounted");
    }, []);

Update:

    useEffect(() => {
        console.log("Updated");
    });

Unmount:

    useEffect(() => {
        return () => console.log("Unmount");
    }, []);

---

# 📦 13. Smart vs Dumb Components

## Presentational (UI only)
- Only display UI
- Receive props
- No business logic

## Container (Logic)
- Fetch data
- Manage state
- Handle logic

Example:
DashboardContainer → fetch API  
DashboardUI → display data  

This improves scalability.

---

# 🚀 14. Performance Basics

Avoid unnecessary re-renders.

Use:
- React.memo
- useMemo
- useCallback
- Proper keys in lists

Example:

    {users.map(u => (
        <UserCard key={u.id} user={u}/>
    ))}

Keys help React track elements efficiently.

---

React follows a component-based architecture where the UI is broken into small reusable components. Each component manages its own logic and rendering. Components can receive data via props and manage internal state using hooks. This makes applications modular, maintainable, and scalable.



# React + Vite + Tailwind CSS

## Commands Reference

```bash
# Create project
npm create vite@latest my-app -- --template react
cd my-app
npm install

# Run dev server
npm run dev

# Build for production
npm run build

# Install Tailwind
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

---

## Step 1 — Create Project

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

Runs at `http://localhost:5173`

Project structure after scaffold:

```
my-app/
├── public/
├── src/
│   ├── assets/
│   ├── App.css        ← component level styles
│   ├── App.jsx
│   ├── index.css      ← global styles
│   └── main.jsx
├── index.html
├── package.json
└── vite.config.js
```
Vite is a frontend build tool that replaced Create React App as the standard way to scaffold React projects. 
Scaffolding means generating the initial project structure automatically — all the folders, config files, and boilerplate code needed to get started. 
Instead of manually creating package.json, vite.config.js, src/main.jsx and so on from scratch, one command does it all. The output is a ready to run project 
that I can open in my editor and start writing actual code immediately.
When I run npm run dev, Vite starts a dev server almost instantly because it does not bundle the entire app upfront — it serves files on demand as the 
browser requests them. This makes the dev server fast regardless of project size. When I run npm run build, Vite bundles everything into optimized static 
files ready for deployment. It also handles hot module replacement (HMR), meaning when I save a file, only that component updates in the browser without a 
full page reload.
`App.css` is for component scoped styles. `index.css` is for global styles. Both are imported and used normally alongside Tailwind.

---

## Step 2 — Add Tailwind CSS

Tailwind is added on top of existing CSS. Both work together — I can use Tailwind utility classes and write regular CSS in the same project.

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

- `tailwindcss@3` — pin to v3. v4 breaks the `npx tailwindcss init` command
- `postcss` — CSS processing tool Tailwind runs through
- `autoprefixer` — adds browser vendor prefixes automatically
- `init -p` — generates `tailwind.config.js` and `postcss.config.js`

**`tailwind.config.js`** — tell Tailwind where to scan for class names

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

**`src/index.css`** — add Tailwind directives at the top, keep existing styles below

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* my global styles go here, below the tailwind directives */
body {
  font-family: sans-serif;
}
```

Tailwind directives must be at the top. My own CSS goes below them. They do not conflict.

**`main.jsx`** — verify this import exists

```jsx
import './index.css'
```

> Verify: add `className="text-blue-500"` to any element. If text turns blue, Tailwind is working.

---

## Step 3 — Using Tailwind and CSS Together

Three ways to style components — all valid, use based on situation:

**1. Tailwind only** — quick, inline, good for layout and spacing

```jsx
<div className="flex items-center gap-4 p-6 bg-white rounded-xl">
  <p className="text-sm text-gray-500">Hello</p>
</div>
```

**2. CSS Module / regular CSS** — good for complex or reusable styles

```css
/* MyComponent.css */
.card {
  background: white;
  border-radius: 12px;
  padding: 24px;
}
```

```jsx
import './MyComponent.css'

<div className="card">Hello</div>
```

**3. Both together** — Tailwind for layout, CSS for custom styles

```jsx
<div className="flex gap-4 p-6 card">Hello</div>
```

---

## Step 4 — Folder Structure

```
src/
├── components/
│   ├── layout/        ← Sidebar, Topbar
│   └── [feature]/     ← one folder per feature
├── pages/             ← composes components
├── hooks/             ← custom hooks
├── lib/               ← helpers, axios setup
├── App.css
├── App.jsx
├── index.css
└── main.jsx
```

---

## Step 5 — Layout Shell

```jsx
// App.jsx
import Sidebar from './components/layout/Sidebar'
import './App.css'

export default function App() {
  return (
    <div className="flex min-h-screen bg-gray-50">
      <Sidebar />
      <main className="flex-1 p-8">
        {/* pages render here */}
      </main>
    </div>
  )
}
```

---

## Step 6 — Component Example

A form component using both Tailwind and regular CSS:

```css
/* components/MyForm.css */
.form-card {
  background: white;
  border: 1px solid #f0f0f0;
  border-radius: 12px;
  padding: 24px;
}

.form-input {
  width: 100%;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 8px 12px;
  font-size: 14px;
  outline: none;
}

.form-input:focus {
  border-color: #9ca3af;
}
```

```jsx
// components/MyForm.jsx
import { useState } from 'react'
import './MyForm.css'

export default function MyForm() {
  const [form, setForm] = useState({
    fieldOne: '',
    fieldTwo: '',
  })

  const handleSubmit = () => {
    console.log('submitted:', form)
    // API call goes here later
  }

  return (
    <div className="form-card">
      <h2 className="text-sm font-medium mb-4">Form Title</h2>

      <div className="flex flex-col gap-4">

        <div className="flex flex-col gap-1.5">
          <label className="text-xs font-medium text-gray-500">Field One</label>
          <input
            type="text"
            value={form.fieldOne}
            onChange={(e) => setForm({ ...form, fieldOne: e.target.value })}
            placeholder="Enter value"
            className="form-input"
          />
        </div>

        <div className="flex flex-col gap-1.5">
          <label className="text-xs font-medium text-gray-500">Field Two</label>
          <input
            type="text"
            value={form.fieldTwo}
            onChange={(e) => setForm({ ...form, fieldTwo: e.target.value })}
            placeholder="Enter value"
            className="form-input"
          />
        </div>

        <button
          onClick={handleSubmit}
          className="bg-gray-900 text-white text-sm font-medium px-4 py-2 rounded-lg hover:bg-gray-700 transition-colors"
        >
          Submit
        </button>

      </div>
    </div>
  )
}
```

Tailwind handles layout (`flex`, `gap-4`, `mb-4`). CSS handles custom component styles (`.form-card`, `.form-input`).

---

## Add Later

```bash
npm install react-router-dom     # routing
npm install axios                # API calls
npm install react-hook-form      # form validation
```

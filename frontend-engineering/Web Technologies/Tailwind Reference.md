# Tailwind CSS — Complete Reference

## What is Tailwind

Tailwind is a utility-first CSS framework. Instead of writing CSS files, I put small single-purpose class names directly on my HTML/JSX elements. Tailwind scans those class names and generates the CSS automatically.

```jsx
// Without Tailwind — I'd write a CSS file + class name
<div className="card">Hello</div>

// With Tailwind — styling is right on the element
<div className="bg-gray-900 rounded-lg p-6 border border-gray-800">Hello</div>
```

No CSS files to manage. No naming classes. No switching context between files.

---

## Installation (Vite + React)

```bash
npm install -D tailwindcss @tailwindcss/vite
```

Update `vite.config.js`:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

Open `src/index.css`, delete everything, replace with just:

```css
@import "tailwindcss";
```

That's it. No `tailwind.config.js` needed with the Vite plugin (Tailwind v4). Run `npm run dev` and it's working.

> **Typo to remember:** Tailwind uses American spelling. `gray` not `grey`. `gray-950` works, `grey-950` does nothing — Tailwind silently skips unknown classes.

---

## The mental model

Tailwind follows one pattern for almost everything:

```
{what}-{value}
```

| Class | CSS it generates |
|---|---|
| `bg-gray-900` | `background-color: #111827` |
| `text-gray-400` | `color: #9ca3af` |
| `text-lg` | `font-size: 1.125rem` |
| `p-4` | `padding: 1rem` |
| `w-64` | `width: 16rem` |
| `flex` | `display: flex` |
| `rounded-lg` | `border-radius: 0.5rem` |

Once this pattern clicks, I can guess most class names without looking anything up.

---

## The spacing scale

The number is not pixels. **1 unit = 0.25rem = 4px.**

```
1  = 4px
2  = 8px
3  = 12px
4  = 16px   ← most used
5  = 20px
6  = 24px
8  = 32px
10 = 40px
12 = 48px
16 = 64px
```

This scale applies to padding (`p-`), margin (`m-`), gap (`gap-`), width (`w-`), height (`h-`), spacing (`space-`), and position offsets (`top-`, `left-` etc). Same numbers everywhere.

---

## Colors

Pattern: `{property}-{color}-{shade}`

Shades go from **50 (lightest) to 950 (darkest)**.

```
bg-gray-50    → almost white
bg-gray-200   → light gray
bg-gray-400   → mid gray
bg-gray-700   → dark gray
bg-gray-900   → very dark
bg-gray-950   → almost black
```

Swap `bg` for other properties:

```
bg-blue-600      → background
text-blue-600    → text color
border-blue-600  → border color
```

**Palettes for a dark dashboard UI:**

```
gray        backgrounds, surfaces, borders, muted text
blue        primary actions, links, focus rings
green       success, active / connected
red         errors, destructive actions
yellow      warnings, pending states
white       primary text on dark backgrounds
```

---

## Responsive design

Tailwind is **mobile first.** A class with no prefix applies to all screens. A prefixed class applies at that breakpoint and above.

| Prefix | Width |
|---|---|
| *(none)* | all screens |
| `sm:` | 640px+ |
| `md:` | 768px+ |
| `lg:` | 1024px+ |
| `xl:` | 1280px+ |

```jsx
// vertical on mobile, horizontal on md+
<div className="flex flex-col md:flex-row gap-4">

// hidden on mobile, visible on md+
<div className="hidden md:block">

// 1 col → 2 col → 3 col
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// text size grows with screen
<h1 className="text-xl md:text-3xl lg:text-5xl font-bold">
```

> For a desktop dashboard app I mostly won't need this. But good to know it exists.

---

## State modifiers

Prefix any class with a state to apply it conditionally:

```jsx
hover:bg-gray-800          // applies on hover
hover:text-white
focus:outline-none         // removes default browser focus ring
focus:ring-2               // custom focus ring
focus:ring-blue-500
active:scale-95            // slight shrink on click — nice button feel
disabled:opacity-50        // dim when disabled
disabled:cursor-not-allowed
```

Example:
```jsx
<button className="bg-blue-600 hover:bg-blue-700 active:scale-95 disabled:opacity-50 transition-colors">
  Save
</button>
```

---

## Layout classes

### Flexbox
```jsx
flex              // display: flex (horizontal row by default)
flex-col          // column direction
flex-row          // row direction (default, rarely needed to write)
flex-1            // takes up all remaining space
flex-wrap         // wraps to next line if needed
flex-shrink-0     // don't shrink (useful for fixed-width sidebars)

items-center      // align vertically in row / horizontally in column
items-start
items-end

justify-center    // center horizontally in row
justify-between   // push items to opposite ends
justify-end       // push to the right

gap-2  gap-4  gap-6  gap-8   // space between children
```

### Grid
```jsx
grid
grid-cols-2    grid-cols-3    grid-cols-4
col-span-2     // item spans 2 columns
gap-4          // gap between grid cells
```

### Common layout pattern — sidebar + main
```jsx
<div className="flex h-screen">
  <aside className="w-64 flex-shrink-0">   {/* fixed width, won't shrink */}
  <main className="flex-1 overflow-auto">  {/* takes remaining space, scrolls */}
</div>
```

---

## Sizing

```
w-full      100%
w-screen    100vw
w-64        16rem fixed
w-auto      auto

h-full      100%
h-screen    100vh
h-14        3.5rem (common topbar height)
h-10        2.5rem (common input/button height)
h-8         2rem (small)

min-h-screen   min-height 100vh
max-w-sm       384px cap
max-w-md       448px cap
max-w-lg       512px cap
max-w-xl       576px cap
max-w-2xl      672px cap
max-w-full     100%
```

---

## Spacing (padding & margin)

```
p-4        padding all sides
px-4       padding left + right
py-2       padding top + bottom
pt-4  pb-4  pl-6  pr-6     individual sides

m-4        margin all sides
mx-auto    center horizontally (classic trick)
mt-4  mb-6 margin top / bottom

space-y-1  space-y-2  space-y-4    vertical gap between direct children
space-x-2                           horizontal gap between direct children
```

---

## Typography

```
text-xs     0.75rem
text-sm     0.875rem   ← most used in dense UIs
text-base   1rem
text-lg     1.125rem
text-xl     1.25rem
text-2xl    1.5rem
text-3xl    1.875rem

font-normal     400
font-medium     500
font-semibold   600   ← headings
font-bold       700

tracking-tight    tighter letter spacing
tracking-wide     wider letter spacing (good for uppercase labels)

leading-tight     tight line height
leading-relaxed   comfortable line height for paragraphs

text-left   text-center   text-right

truncate      overflow with ellipsis, single line
line-clamp-2  clamp to 2 lines (needs @tailwindcss/line-clamp or v3+)
uppercase   capitalize   lowercase
```

---

## Borders & Radius

```
border          1px border all sides
border-b        bottom only
border-t  border-r  border-l   individual sides
border-2        2px border

border-gray-700
border-gray-800   ← common for dark UI surfaces

rounded         small radius
rounded-md      medium
rounded-lg      large  ← most used
rounded-xl      extra large (cards, modals)
rounded-full    pill / circle

divide-y          border between stacked children
divide-gray-800   color for those dividers
```

---

## Backgrounds

```
bg-gray-950   page background (darkest)
bg-gray-900   sidebar, card, panel
bg-gray-800   hover state, elevated surface
bg-gray-700   active / selected
bg-transparent
bg-white
```

---

## Position

```
relative    position relative (needed as parent for absolute children)
absolute    position absolute (relative to nearest relative parent)
fixed       fixed to viewport
sticky      sticks on scroll

inset-0     top + right + bottom + left all 0 (full overlay)
top-0  right-0  bottom-0  left-0
top-4  right-4   (with spacing scale)

z-10  z-20  z-50   z-index
```

Example — badge pinned to corner of an element:
```jsx
<div className="relative">
  <img src="..." />
  <span className="absolute top-1 right-1 bg-green-500 rounded-full w-2 h-2" />
</div>
```

---

## Overflow

```
overflow-hidden    clip anything outside bounds
overflow-auto      scroll if content overflows
overflow-y-auto    vertical scroll only
overflow-x-hidden  hide horizontal scroll
```

---

## Transitions & Animation

```
transition-colors    smooth color transitions only
transition-all       smooth all properties

duration-150    fast
duration-200    slightly slower, feels natural
duration-300    slow

ease-in-out

animate-spin      spinning (loading spinners)
animate-pulse     pulsing (skeleton loaders)
animate-bounce    bouncing
```

Always pair with a state modifier:
```jsx
className="bg-gray-800 hover:bg-gray-700 transition-colors duration-150"
```

---

## Shadows & Opacity

```
shadow      small shadow
shadow-md   medium
shadow-lg   large
shadow-xl   modals, cards, elevated elements

opacity-0    invisible
opacity-50   half transparent
opacity-100  fully visible
```

---

## Utility

```
cursor-pointer         pointer cursor on hover
cursor-not-allowed     blocked cursor
select-none            can't select text (good for buttons/nav)
pointer-events-none    element ignores all mouse events

sr-only     visually hidden but readable by screen readers
```

---

## The cn() utility (from shadcn)

When I install shadcn it drops a `cn()` helper in `src/lib/utils.js`. Use it for conditional classes:

```jsx
import { cn } from "@/lib/utils"

<div className={cn(
  "px-3 py-2 rounded-lg text-sm transition-colors",
  isActive ? "bg-gray-800 text-white" : "text-gray-400 hover:bg-gray-800 hover:text-white"
)}>
  Nav item
</div>
```

Much cleaner than string concatenation. Use this everywhere.

---

## Most useful

```
flex  flex-col  flex-1  items-center  justify-between
gap-4  w-full  h-screen  px-4  py-2  p-6
text-sm  text-gray-400  rounded-lg  transition-colors
```

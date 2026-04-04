# shadcn/ui — Complete Reference

## What is shadcn/ui

shadcn/ui is NOT a component library I install as a package. It's a CLI tool that **copies component source code directly into my project**. I own the files. I can read them, edit them, and they never break because of an upstream update.

Each component is built on:
- **Radix UI** — headless, accessible primitives. Handles keyboard navigation, focus trapping, ARIA attributes etc. I get all of this for free.
- **Tailwind CSS** — all the styling
- **class-variance-authority (cva)** — manages variants like button sizes and styles

The components land in `src/components/ui/`. They're just regular React files I can open and edit.

---

## The approach for this project

```
shadcn   →  components (Button, Dialog, Table, Input, Badge etc)
Tailwind →  layout glue only (flex, gap, padding, width)
CSS file →  anything custom or project-specific
```

I don't need to learn Tailwind deeply. shadcn handles all the complex styling. I only use basic Tailwind for putting things in the right place on the page.

---

## Installation

### Step 1 — Set up the @ alias (required by shadcn)

shadcn uses `@/` to mean `src/`. Need to configure this in Vite first.

```bash
npm install -D @types/node
```

Update `vite.config.js`:

```js
import path from 'path'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

Create `jsconfig.json` in project root:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Step 2 — Init shadcn

```bash
npx shadcn@latest init
```

It asks:
- Which style? → **Default**
- Base color? → **Gray**
- CSS variables? → **Yes**

This creates:
- `src/components/ui/` — components will land here
- `src/lib/utils.js` — the `cn()` helper
- `components.json` — shadcn config

### Step 3 — Add components as I need them

```bash
npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add label
npx shadcn@latest add badge
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add dropdown-menu
npx shadcn@latest add select
npx shadcn@latest add table
npx shadcn@latest add tabs
npx shadcn@latest add toast
npx shadcn@latest add tooltip
npx shadcn@latest add sheet
npx shadcn@latest add skeleton
npx shadcn@latest add separator
```

Only add what I need. Each command copies the component file into `src/components/ui/`.

### Step 4 — How to import

```jsx
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Badge } from "@/components/ui/badge"
```

---

## The cn() utility

shadcn installs this in `src/lib/utils.js`. It merges Tailwind class names intelligently — handles conflicts and conditional classes cleanly.

```jsx
import { cn } from "@/lib/utils"

// conditional classes — much cleaner than string concatenation
<div className={cn(
  "px-3 py-2 rounded-lg text-sm transition-colors",
  isActive ? "bg-gray-800 text-white" : "text-gray-400 hover:bg-gray-800"
)}>
```

Use this in my own components too, not just shadcn ones.

---

## Components

---

### Button

```bash
npx shadcn@latest add button
```

```jsx
import { Button } from "@/components/ui/button"
```

**Variants:**
```jsx
<Button>Default</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="destructive">Delete</Button>
<Button variant="link">Link</Button>
```

**Sizes:**
```jsx
<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
<Button size="icon">⚙️</Button>   {/* square icon button */}
```

**States:**
```jsx
<Button disabled>Disabled</Button>

// Loading state
<Button disabled>
  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
  Saving...
</Button>
```

**Adding Tailwind on top:**
```jsx
// className merges on top of the component's defaults
<Button className="w-full mt-4">Full width</Button>
<Button className="gap-2"><PlusIcon /> Add Connector</Button>
```

> `asChild` — use this when I want shadcn's behaviour on my own element instead of the default `<button>`:
> ```jsx
> <Button asChild>
>   <a href="/connectors">Go to Connectors</a>
> </Button>
> ```

---

### Input + Label

```bash
npx shadcn@latest add input label
```

```jsx
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
```

**Basic:**
```jsx
<Input placeholder="Connector name" />
```

**With label (always pair these):**
```jsx
<div className="space-y-2">
  <Label htmlFor="name">Connector Name</Label>
  <Input id="name" placeholder="Enter name..." />
</div>
```

**Types:**
```jsx
<Input type="password" placeholder="API key" />
<Input type="search" placeholder="Search connectors..." />
<Input type="url" placeholder="https://api.example.com" />
<Input type="number" placeholder="Timeout (ms)" />
```

**Disabled:**
```jsx
<Input disabled value="Read only value" />
```

---

### Badge

```bash
npx shadcn@latest add badge
```

```jsx
import { Badge } from "@/components/ui/badge"
```

**Variants:**
```jsx
<Badge>Default</Badge>
<Badge variant="secondary">Secondary</Badge>
<Badge variant="outline">Outline</Badge>
<Badge variant="destructive">Error</Badge>
```

**Custom colors with Tailwind (for status indicators):**
```jsx
<Badge className="bg-green-900 text-green-400 border-green-800">Active</Badge>
<Badge className="bg-yellow-900 text-yellow-400 border-yellow-800">Pending</Badge>
<Badge className="bg-red-900 text-red-400 border-red-800">Failed</Badge>
<Badge className="bg-gray-800 text-gray-400 border-gray-700">Inactive</Badge>
<Badge className="bg-blue-900 text-blue-400 border-blue-800">Connecting</Badge>
```

---

### Card

```bash
npx shadcn@latest add card
```

```jsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card"
```

**Full example:**
```jsx
<Card>
  <CardHeader>
    <CardTitle>Connector A</CardTitle>
    <CardDescription>REST API — Last synced 2 mins ago</CardDescription>
  </CardHeader>
  <CardContent>
    <p className="text-sm text-muted-foreground">
      Handling 240 requests/min. All systems normal.
    </p>
  </CardContent>
  <CardFooter className="flex justify-between items-center">
    <Badge className="bg-green-900 text-green-400 border-green-800">Active</Badge>
    <Button variant="outline" size="sm">View Logs</Button>
  </CardFooter>
</Card>
```

**Simple card (just content):**
```jsx
<Card className="p-6">
  <h3 className="text-sm font-medium text-muted-foreground mb-1">Total Connectors</h3>
  <p className="text-3xl font-bold">12</p>
</Card>
```

---

### Dialog (Modal)

```bash
npx shadcn@latest add dialog
```

```jsx
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"
```

**How it works:**
- `DialogTrigger` — wraps whatever opens the modal
- `DialogContent` — the modal box itself
- `asChild` on Trigger — uses my element instead of adding a wrapper

**Example — Add Connector modal:**
```jsx
<Dialog>
  <DialogTrigger asChild>
    <Button>Add Connector</Button>
  </DialogTrigger>

  <DialogContent className="sm:max-w-md">
    <DialogHeader>
      <DialogTitle>Add New Connector</DialogTitle>
      <DialogDescription>
        Fill in the connection details below.
      </DialogDescription>
    </DialogHeader>

    <div className="space-y-4 py-4">
      <div className="space-y-2">
        <Label htmlFor="connectorName">Name</Label>
        <Input id="connectorName" placeholder="My API Connector" />
      </div>
      <div className="space-y-2">
        <Label htmlFor="endpoint">Endpoint URL</Label>
        <Input id="endpoint" type="url" placeholder="https://api.example.com" />
      </div>
      <div className="space-y-2">
        <Label>Type</Label>
        <Select>
          <SelectTrigger>
            <SelectValue placeholder="Select type" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="rest">REST API</SelectItem>
            <SelectItem value="graphql">GraphQL</SelectItem>
            <SelectItem value="grpc">gRPC</SelectItem>
          </SelectContent>
        </Select>
      </div>
    </div>

    <DialogFooter>
      <Button variant="outline">Cancel</Button>
      <Button>Save Connector</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

**Controlled dialog (open/close with state):**
```jsx
const [open, setOpen] = useState(false)

<Dialog open={open} onOpenChange={setOpen}>
  <DialogTrigger asChild>
    <Button>Open</Button>
  </DialogTrigger>
  <DialogContent>
    ...
    <Button onClick={() => setOpen(false)}>Close</Button>
  </DialogContent>
</Dialog>
```

---

### Dropdown Menu

```bash
npx shadcn@latest add dropdown-menu
```

```jsx
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"
```

**Row actions menu (most common use):**
```jsx
<DropdownMenu>
  <DropdownMenuTrigger asChild>
    <Button variant="ghost" size="icon">⋯</Button>
  </DropdownMenuTrigger>
  <DropdownMenuContent align="end">
    <DropdownMenuLabel>Actions</DropdownMenuLabel>
    <DropdownMenuSeparator />
    <DropdownMenuItem>View Details</DropdownMenuItem>
    <DropdownMenuItem>Edit</DropdownMenuItem>
    <DropdownMenuItem>View Logs</DropdownMenuItem>
    <DropdownMenuSeparator />
    <DropdownMenuItem className="text-red-500 focus:text-red-500">
      Delete
    </DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

> `align="end"` — opens the menu aligned to the right edge of the trigger. Use this for row action buttons.

---

### Select

```bash
npx shadcn@latest add select
```

```jsx
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"
```

**Basic:**
```jsx
<Select>
  <SelectTrigger className="w-48">
    <SelectValue placeholder="Select type" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="rest">REST API</SelectItem>
    <SelectItem value="graphql">GraphQL</SelectItem>
    <SelectItem value="grpc">gRPC</SelectItem>
    <SelectItem value="websocket">WebSocket</SelectItem>
  </SelectContent>
</Select>
```

**Controlled (with state):**
```jsx
const [type, setType] = useState("")

<Select value={type} onValueChange={setType}>
  <SelectTrigger>
    <SelectValue placeholder="Select type" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="rest">REST API</SelectItem>
    <SelectItem value="graphql">GraphQL</SelectItem>
  </SelectContent>
</Select>
```

---

### Table

```bash
npx shadcn@latest add table
```

```jsx
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table"
```

**Connectors list:**
```jsx
<div className="rounded-lg border border-border overflow-hidden">
  <Table>
    <TableHeader>
      <TableRow>
        <TableHead>Name</TableHead>
        <TableHead>Type</TableHead>
        <TableHead>Status</TableHead>
        <TableHead>Last Sync</TableHead>
        <TableHead className="text-right">Actions</TableHead>
      </TableRow>
    </TableHeader>
    <TableBody>
      {connectors.map((connector) => (
        <TableRow key={connector.id}>
          <TableCell className="font-medium">{connector.name}</TableCell>
          <TableCell>{connector.type}</TableCell>
          <TableCell>
            <Badge className="bg-green-900 text-green-400 border-green-800">
              {connector.status}
            </Badge>
          </TableCell>
          <TableCell className="text-muted-foreground">{connector.lastSync}</TableCell>
          <TableCell className="text-right">
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" size="icon">⋯</Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuItem>Edit</DropdownMenuItem>
                <DropdownMenuItem>View Logs</DropdownMenuItem>
                <DropdownMenuSeparator />
                <DropdownMenuItem className="text-red-500">Delete</DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </TableCell>
        </TableRow>
      ))}
    </TableBody>
  </Table>
</div>
```

---

### Tabs

```bash
npx shadcn@latest add tabs
```

```jsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"
```

**Connector detail page:**
```jsx
<Tabs defaultValue="overview">
  <TabsList>
    <TabsTrigger value="overview">Overview</TabsTrigger>
    <TabsTrigger value="logs">Logs</TabsTrigger>
    <TabsTrigger value="settings">Settings</TabsTrigger>
  </TabsList>

  <TabsContent value="overview" className="mt-6">
    <p>Connector overview content here</p>
  </TabsContent>

  <TabsContent value="logs" className="mt-6">
    <p>Log entries here</p>
  </TabsContent>

  <TabsContent value="settings" className="mt-6">
    <p>Settings form here</p>
  </TabsContent>
</Tabs>
```

---

### Toast (notifications)

```bash
npx shadcn@latest add toast
```

**Step 1 — Add Toaster once to App.jsx (only once, at the root):**
```jsx
import { Toaster } from "@/components/ui/toaster"

function App() {
  return (
    <div className="flex h-screen bg-background text-foreground">
      <Sidebar />
      <main className="flex-1 overflow-auto">
        ...
      </main>
      <Toaster />
    </div>
  )
}
```

**Step 2 — Use in any component:**
```jsx
import { useToast } from "@/hooks/use-toast"

function ConnectorForm() {
  const { toast } = useToast()

  const handleSave = async () => {
    try {
      await saveConnector()
      toast({
        title: "Connector saved",
        description: "Your connector is now active.",
      })
    } catch (err) {
      toast({
        title: "Something went wrong",
        description: "Could not connect to the endpoint.",
        variant: "destructive",
      })
    }
  }
}
```

---

### Sheet (side panel / drawer)

```bash
npx shadcn@latest add sheet
```

```jsx
import {
  Sheet,
  SheetContent,
  SheetDescription,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from "@/components/ui/sheet"
```

Good alternative to a modal when showing detail — slides in from the side.

```jsx
<Sheet>
  <SheetTrigger asChild>
    <Button variant="outline">View Details</Button>
  </SheetTrigger>
  <SheetContent>
    <SheetHeader>
      <SheetTitle>Connector A</SheetTitle>
      <SheetDescription>REST API — api.example.com</SheetDescription>
    </SheetHeader>
    <div className="mt-6 space-y-4">
      <p className="text-sm text-muted-foreground">Detail content here</p>
    </div>
  </SheetContent>
</Sheet>
```

> `side` prop controls which edge it slides from: `"right"` (default), `"left"`, `"top"`, `"bottom"`

---

### Skeleton (loading state)

```bash
npx shadcn@latest add skeleton
```

```jsx
import { Skeleton } from "@/components/ui/skeleton"
```

Use while fetching data from the backend — much better than a spinner for content-heavy pages.

```jsx
// Loading state for a card
{isLoading ? (
  <div className="space-y-3">
    <Skeleton className="h-5 w-48" />
    <Skeleton className="h-4 w-full" />
    <Skeleton className="h-4 w-3/4" />
  </div>
) : (
  <div>actual content</div>
)}

// Loading state for a table
{isLoading ? (
  <div className="space-y-2">
    {[...Array(5)].map((_, i) => (
      <Skeleton key={i} className="h-12 w-full" />
    ))}
  </div>
) : (
  <Table>...</Table>
)}
```

---

### Tooltip

```bash
npx shadcn@latest add tooltip
```

```jsx
import {
  Tooltip,
  TooltipContent,
  TooltipProvider,
  TooltipTrigger,
} from "@/components/ui/tooltip"
```

Wrap the app (or a section) in `TooltipProvider` once. Then use `Tooltip` anywhere inside it.

**Add TooltipProvider to App.jsx:**
```jsx
import { TooltipProvider } from "@/components/ui/tooltip"

function App() {
  return (
    <TooltipProvider>
      <div className="flex h-screen">
        ...
      </div>
    </TooltipProvider>
  )
}
```

**Use in components:**
```jsx
<Tooltip>
  <TooltipTrigger asChild>
    <Button variant="ghost" size="icon">⚙️</Button>
  </TooltipTrigger>
  <TooltipContent>
    <p>Settings</p>
  </TooltipContent>
</Tooltip>
```

---

### Separator

```bash
npx shadcn@latest add separator
```

```jsx
import { Separator } from "@/components/ui/separator"

// Horizontal divider
<Separator />

// Vertical divider
<Separator orientation="vertical" className="h-6" />
```

---

## Customising components

Since I own the code, I can open `src/components/ui/button.jsx` and edit it. But for one-off overrides I just pass `className` — it merges on top of defaults:

```jsx
<Button className="w-full">Full width button</Button>
<Button className="rounded-none">No radius</Button>
<Card className="border-blue-800">Highlighted card</Card>
```

---

## CSS variables (theming)

shadcn uses CSS variables for its color system. These are defined in `src/index.css` and I can change them to theme the whole app:

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    /* ... */
  }
  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ... */
  }
}
```

The class names `text-foreground`, `bg-background`, `text-muted-foreground`, `border-border` etc reference these variables. That's why shadcn components adapt automatically to light/dark mode.

---

## Component map for this project

| Component | Where I use it |
|---|---|
| `Button` | everywhere — forms, actions, nav |
| `Input` + `Label` | connector forms, search bar |
| `Badge` | connector status (Active / Failed / Pending) |
| `Card` | dashboard stats, connector list items |
| `Dialog` | add connector, edit connector, confirm delete |
| `DropdownMenu` | row action menus in the connectors table |
| `Select` | connector type picker in forms |
| `Table` | connectors list page |
| `Tabs` | connector detail page (Overview / Logs / Settings) |
| `Toast` | save success, error feedback from API |
| `Sheet` | connector detail side panel |
| `Skeleton` | loading state while fetching from .NET backend |
| `Tooltip` | icon button labels in sidebar and topbar |
| `Separator` | visual dividers in sidebar, forms |

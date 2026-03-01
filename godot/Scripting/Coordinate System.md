# 🎮 Godot Coordinate Systems

Understanding coordinate systems is critical for tile-based games.

When you click the mouse, its position exists in multiple coordinate spaces:

1. Screen Space (window pixels)  
2. World Space (scene space)  
3. Node Local Space  
4. Tile Cell Space (grid coordinates)  

You must convert step-by-step between them.

---

# 🧭 1️⃣ Screen Space (Window Pixels)

## Definition
Raw pixel position inside the game window.

- Top-left of window = (0,0)
- Bottom-right = (window_width, window_height)

## How to Get It

```csharp
public override void _UnhandledInput(InputEvent @event)
{
    if (@event is InputEventMouseButton mouse)
    {
        Vector2 screenPos = mouse.Position;
        GD.Print("Screen: ", screenPos);
    }
}
```

## Important
- Not affected by camera
- Not affected by node transforms
- Pure window coordinates

---

# 🌍 2️⃣ World Space (Scene Coordinates)

## Definition
Position inside the game world.

This considers:
- Camera position
- Zoom
- Viewport transforms

## How to Get It

```csharp
Vector2 worldPos = GetGlobalMousePosition();
GD.Print("World: ", worldPos);
```

## Important
- This is where your nodes actually exist.
- If Camera2D moves, this value changes accordingly.

---

# 🏠 3️⃣ Node Local Space

## Definition
Position relative to a specific node.

If a node is placed at (200,100) in world space:
- Its local (0,0) starts there.

---

## Convert World → Local

```csharp
Vector2 localPos = someNode.ToLocal(worldPos);
```

---

## Shortcut for Mouse

```csharp
Vector2 localPos = someNode.GetLocalMousePosition();
```

Example:

```csharp
Vector2 localPos = _paletteLayer.GetLocalMousePosition();
GD.Print("Local to palette: ", localPos);
```

## Important
Local space ignores global position and camera — it is relative only to that node.

---

# 🧱 4️⃣ Tile Cell Space (Grid Coordinates)

## Definition
Tile index inside a TileMap.

TileMap does NOT work in pixels.
It works in grid coordinates like:

(0,0)  
(1,0)  
(2,0)

---

## Convert Local Pixels → Tile Cell

```csharp
Vector2I cell = tileMap.LocalToMap(localPos);
```

Full example:

```csharp
Vector2 world = GetGlobalMousePosition();
Vector2 local = _gridLayer.ToLocal(world);
Vector2I cell = _gridLayer.LocalToMap(local);

GD.Print("Tile cell: ", cell);
```

---

# 🔄 Full Conversion Chain

```csharp
// 1️⃣ Screen space
Vector2 screenPos = mouseEvent.Position;

// 2️⃣ World space
Vector2 worldPos = GetGlobalMousePosition();

// 3️⃣ Node local space
Vector2 localPos = _gridLayer.ToLocal(worldPos);

// 4️⃣ Tile cell space
Vector2I cell = _gridLayer.LocalToMap(localPos);
```

---

# 🔁 Reverse Conversion (Tile → Pixel)

Sometimes you need the opposite direction.

## Tile Cell → Local Pixel

```csharp
Vector2 localPos = tileMap.MapToLocal(cell);
```

## Tile Cell → World Position

```csharp
Vector2 worldPos = tileMap.ToGlobal(
    tileMap.MapToLocal(cell)
);
```

---

# 📦 Example With Numbers

Assume:
- Tile size = 32x32
- TileMap at (0,0)
- Mouse click at (96, 64)

Step 1:
Local mouse = (96, 64)

Step 2:
Divide by tile size:

96 / 32 = 3  
64 / 32 = 2  

Final result:

Cell = (3,2)

Meaning:
- 4th tile from left
- 3rd tile from top

---

# 🔥 Most Common Tile Click Pattern

In tile games, this is what you will use 90% of the time:

```csharp
Vector2I cell = tileMap.LocalToMap(
    tileMap.GetLocalMousePosition()
);
```

This is the compact version of the full conversion chain.

---

# 🧪 Debug Trick — Print All Spaces

```csharp
GD.Print("Screen: ", mouse.Position);
GD.Print("World: ", GetGlobalMousePosition());
GD.Print("Local: ", _gridLayer.GetLocalMousePosition());
GD.Print("Cell: ", _gridLayer.LocalToMap(
    _gridLayer.GetLocalMousePosition()
));
```

This lets you see coordinate transformations live while clicking.

---

# 🎯 Summary Table

| Space        | Represents                     | Function |
|--------------|--------------------------------|----------|
| Screen       | Window pixels                  | mouse.Position |
| World        | Scene coordinates              | GetGlobalMousePosition() |
| Node Local   | Relative to a node             | node.ToLocal() / GetLocalMousePosition() |
| Tile Cell    | Grid coordinate (tile index)   | LocalToMap() |

---

# 🧠 Core Insight

Game engines separate coordinate systems because:

- Screen depends on resolution
- World depends on camera
- Local depends on node position
- Tile cell depends on tile size

Understanding this pipeline makes tile interaction predictable and clean.

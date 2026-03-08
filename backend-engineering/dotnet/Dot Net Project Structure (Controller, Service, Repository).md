# .NET Reference — For Java / Spring Boot Developers

> Same flow you know from Spring Boot. Controller → Service → Repository.
> No MediatR, no CQRS. Just the basics done the .NET way.

---

## 🗺️ The Flow — Identical to Spring Boot

```
HTTP Request
     │
     ▼
┌─────────────────────────────────────────────────────┐
│  Controller                                         │
│  Receives the HTTP request.                         │
│  Calls the Service. Returns the HTTP response.      │
│  Like @RestController in Spring.                    │
└──────────────────┬──────────────────────────────────┘
                   │ calls
                   ▼
┌─────────────────────────────────────────────────────┐
│  Service                                            │
│  Contains the business logic.                       │
│  Validates, transforms, orchestrates.               │
│  Like @Service in Spring.                           │
└──────────────────┬──────────────────────────────────┘
                   │ calls
                   ▼
┌─────────────────────────────────────────────────────┐
│  Repository                                         │
│  Talks to the database via EF Core (like JPA).      │
│  Like @Repository + JpaRepository in Spring.        │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
               Database
```

That's it. Keep this in mind while reading the rest.

---

## 📁 Folder Structure

One project for now. Add layers later once you're comfortable.

```
MyApp/
│
├── MyApp.sln                        # Solution file — like a root pom.xml
│
└── src/
    └── MyApp.API/
        ├── MyApp.API.csproj         # Project file — lists all NuGet packages (like pom.xml)
        ├── Program.cs               # Entry point — wires DI, middleware, routing (like @SpringBootApplication)
        ├── appsettings.json         # Config — DB connection string, etc. (like application.properties)
        │
        ├── Controllers/
        │   └── ProductsController.cs  # HTTP endpoints — like @RestController
        │
        ├── Services/
        │   ├── IProductService.cs     # Interface — like a Java service interface
        │   └── ProductService.cs      # Implementation — like @Service
        │
        ├── Repositories/
        │   ├── IProductRepository.cs  # Interface — like a Java repository interface
        │   └── ProductRepository.cs   # Implementation — uses EF Core (like JpaRepository)
        │
        ├── Models/
        │   └── Product.cs             # Entity class — like @Entity in JPA
        │
        ├── DTOs/
        │   ├── CreateProductRequest.cs  # Input DTO — what the client sends
        │   └── ProductResponse.cs       # Output DTO — what you send back
        │
        └── Data/
            ├── AppDbContext.cs          # EF Core session — like EntityManager
            └── Migrations/              # Auto-generated — like Flyway scripts but C#
```

---

## 🛠️ CLI — Create the Project

Run these once from your terminal to scaffold everything.

```bash
# Create the solution and project
dotnet new sln -n MyApp
dotnet new webapi -n MyApp.API -o src/MyApp.API
dotnet sln add src/MyApp.API/MyApp.API.csproj

# Install EF Core packages (like adding spring-data-jpa to pom.xml)
dotnet add src/MyApp.API package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/MyApp.API package Microsoft.EntityFrameworkCore.Tools

# Install EF CLI tool — once per machine
dotnet tool install --global dotnet-ef

# Day-to-day commands
dotnet build
dotnet run --project src/MyApp.API
dotnet watch --project src/MyApp.API run  # hot reload — like Spring DevTools
dotnet test
```

---

## Program.cs — Entry Point

This runs once when the app starts. Two jobs: register services into the DI container, then set up the HTTP pipeline. In Spring Boot, `@SpringBootApplication` handles both automatically. Here you do it explicitly — which means you can see exactly what is wired.

```csharp
// Program.cs

var builder = WebApplication.CreateBuilder(args);
// Reads appsettings.json, sets up logging, creates the DI container.

// ── Register services (like @Component scanning in Spring) ───────────────────
// Instead of @Autowired finding beans automatically, you register them here explicitly.

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
// Registers EF Core. Connection string comes from appsettings.json.
// Like spring.datasource.url in application.properties.

builder.Services.AddScoped<IProductRepository, ProductRepository>();
// "When something asks for IProductRepository, give it ProductRepository."
// AddScoped = one instance per HTTP request — right for anything touching the DB.

builder.Services.AddScoped<IProductService, ProductService>();
// Same pattern for the service.

builder.Services.AddControllers();  // enables [ApiController] classes
builder.Services.AddSwaggerGen();   // Swagger UI at /swagger — like Springdoc

var app = builder.Build();
// After this line, services are locked. Below is the HTTP pipeline.

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(); // visit /swagger to see and test all your endpoints
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers(); // maps routes to your [ApiController] classes

app.Run(); // start the server — like embedded Tomcat
```

---

## appsettings.json — Config

The equivalent of `application.properties`. JSON format instead of key=value.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;Trusted_Connection=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

---

## Model — Entity Class

The `Product` class represents a row in the database. In JPA you annotate it with `@Entity`, `@Id`, `@Column`. In EF Core, the class is plain C# — the database mapping is configured separately in `AppDbContext` or a config file. The class itself stays clean.

```csharp
// Models/Product.cs

public class Product
{
    public int Id { get; set; }          // EF Core treats "Id" as the primary key automatically
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }   // decimal = BigDecimal — always use for money
    public bool IsActive { get; set; } = true;
}
```

---

## DTOs

DTOs are what you accept from the client and what you send back. Never expose your entity class directly in responses — DTOs let you control exactly what fields are visible and validated.

```csharp
// DTOs/CreateProductRequest.cs
// What the client sends in the POST body — like a @RequestBody DTO in Spring

public class CreateProductRequest
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}


// DTOs/ProductResponse.cs
// What you send back — never return the raw entity

public class ProductResponse
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

---

## Controller — HTTP Endpoints

The controller is the entry point for every HTTP request. Its only job is to receive the request, call the service, and return a response. No business logic here — just HTTP in, response out. The controller calls `IProductService` — the service interface defined in the next section.

```csharp
// Controllers/ProductsController.cs

[ApiController]              // like @RestController — handles JSON binding automatically
[Route("api/[controller]")]  // [controller] → "Products", so base route = api/products
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    // IProductService is the service interface — DI injects the real ProductService at runtime

    public ProductsController(IProductService service) => _service = service;

    // GET /api/products
    [HttpGet]                // like @GetMapping
    public async Task<ActionResult<List<ProductResponse>>> GetAll()
    {
        var products = await _service.GetAllAsync();
        return Ok(products); // HTTP 200 with JSON body
    }

    // GET /api/products/5
    [HttpGet("{id}")]        // like @GetMapping("/{id}")
    public async Task<ActionResult<ProductResponse>> GetById(int id)
    // "id" matches the {id} in the route — like @PathVariable
    {
        var product = await _service.GetByIdAsync(id);

        if (product is null) return NotFound();  // HTTP 404
        return Ok(product);                      // HTTP 200
    }

    // POST /api/products
    [HttpPost]               // like @PostMapping
    public async Task<ActionResult<ProductResponse>> Create([FromBody] CreateProductRequest request)
    // [FromBody] = deserialise the JSON body — like @RequestBody
    {
        var created = await _service.CreateAsync(request);

        // HTTP 201 Created, with a Location header pointing to the new resource
        // like ResponseEntity.created(URI.create("/api/products/" + id)).body(created)
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    // DELETE /api/products/5
    [HttpDelete("{id}")]     // like @DeleteMapping("/{id}")
    public async Task<IActionResult> Delete(int id)
    {
        var deleted = await _service.DeleteAsync(id);

        if (!deleted) return NotFound();  // HTTP 404
        return NoContent();               // HTTP 204 — success, no body
    }
}
```

---

## Service — Business Logic

The service is called by the controller. It contains the business rules, builds the entity, then passes it to the repository. It calls `IProductRepository` — the repository interface defined in the next section. The service never touches the database directly.

```csharp
// Services/IProductService.cs
// The contract the controller depends on.

public interface IProductService
{
    Task<List<ProductResponse>> GetAllAsync();
    Task<ProductResponse?> GetByIdAsync(int id);
    Task<ProductResponse> CreateAsync(CreateProductRequest request);
    Task<bool> DeleteAsync(int id);
}


// Services/ProductService.cs
// Like @Service in Spring. Contains business logic, calls IProductRepository.
// No DB calls here — that is the repository's job.

public class ProductService : IProductService
{
    private readonly IProductRepository _repo;
    // IProductRepository is the repository interface — DI injects ProductRepository at runtime

    public ProductService(IProductRepository repo) => _repo = repo;

    public async Task<List<ProductResponse>> GetAllAsync()
    {
        var products = await _repo.GetAllAsync();

        // Map entities to response DTOs — use AutoMapper for larger projects
        return products.Select(p => new ProductResponse
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price
        }).ToList();
    }

    public async Task<ProductResponse?> GetByIdAsync(int id)
    {
        var product = await _repo.GetByIdAsync(id);

        if (product is null) return null;
        // "is null" = modern C# null check, same as product == null

        return new ProductResponse { Id = product.Id, Name = product.Name, Price = product.Price };
    }

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        // Business rules live here — the repository knows nothing about these
        if (string.IsNullOrWhiteSpace(request.Name))
            throw new ArgumentException("Product name is required.");
        // string.IsNullOrWhiteSpace = null OR empty OR "   " — like name == null || name.isBlank()

        if (request.Price <= 0)
            throw new ArgumentException("Price must be greater than zero.");

        // Service builds the entity — fully ready to save
        var product = new Product { Name = request.Name, Price = request.Price };

        // Hand off to repository — all DB work happens there
        var saved = await _repo.AddAsync(product);

        return new ProductResponse { Id = saved.Id, Name = saved.Name, Price = saved.Price };
    }

    public async Task<bool> DeleteAsync(int id)
    {
        return await _repo.DeleteAsync(id); // repository returns false if not found
    }
}
```

---

## Repository — Data Access

The repository is called by the service. It receives ready-to-save entities and handles all the database work using EF Core. No business logic, no validation here — just persistence. It calls `AppDbContext` which is EF Core's session with the database, defined in the next section.

```csharp
// Repositories/IProductRepository.cs
// The contract the service depends on.

public interface IProductRepository
{
    Task<List<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    // Product? = nullable — null means not found, like Optional<Product>
    Task<Product> AddAsync(Product product);  // receives entity built by the service
    Task<bool> DeleteAsync(int id);
}


// Repositories/ProductRepository.cs
// Implements IProductRepository using EF Core.
// Calls _db (AppDbContext) to run SQL. No business rules here.

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;
    // AppDbContext is EF Core's DB session — injected by DI

    public ProductRepository(AppDbContext db) => _db = db;

    public async Task<List<Product>> GetAllAsync()
        => await _db.Products.ToListAsync();
    // _db.Products is a DbSet<Product> — ToListAsync() runs: SELECT * FROM products

    public async Task<Product?> GetByIdAsync(int id)
        => await _db.Products.FindAsync(id);
    // FindAsync runs: SELECT * FROM products WHERE Id = id

    public async Task<Product> AddAsync(Product product)
    {
        await _db.Products.AddAsync(product); // stage the insert
        await _db.SaveChangesAsync();         // commit — EF generates and runs INSERT SQL
        return product;                       // product.Id is populated by EF after save
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return false;

        _db.Products.Remove(product);
        await _db.SaveChangesAsync();         // commit — EF generates and runs DELETE SQL
        return true;
    }
}
```

---

## EF Core — DbContext

`AppDbContext` is EF Core's session with the database — equivalent to JPA's `EntityManager`. The repository calls it directly. You define a `DbSet<T>` for each entity, which represents a table. `AddAsync`, `FindAsync`, `Remove`, `ToListAsync` are all built-in methods on `DbSet<T>` — you never implement them.

```csharp
// Data/AppDbContext.cs

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    // ": base(options)" = call the parent constructor — like super(options) in Java

    public DbSet<Product> Products { get; set; }
    // Represents the products table.
    // AddAsync, FindAsync, Remove, ToListAsync are all built into DbSet<T> by EF Core.
    // Like JpaRepository giving you save(), findById(), findAll() for free.

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure table/column mapping — like @Table and @Column in JPA
        modelBuilder.Entity<Product>(entity =>
        {
            entity.ToTable("products");
            entity.Property(p => p.Name).IsRequired().HasMaxLength(100);
            entity.Property(p => p.Price).HasColumnType("decimal(18,2)");
        });
    }
}
```

---

## EF Core Migrations

Migrations keep the database schema in sync with your entity classes. When you add a property to `Product`, create a migration — EF generates the SQL for you. Like Flyway but the scripts are auto-generated.

```bash
# Run after every entity change — generates a migration file
dotnet ef migrations add InitialCreate --project src/MyApp.API

# Apply all pending migrations to the database
dotnet ef database update --project src/MyApp.API

# Undo the last migration (only if not yet applied to the DB)
dotnet ef migrations remove --project src/MyApp.API
```

---

## Java → .NET Cheat Sheet

| Java / Spring | .NET | Note |
|---|---|---|
| `pom.xml` | `.csproj` | |
| `mvn build` | `dotnet build` | |
| `mvn spring-boot:run` | `dotnet run --project src/MyApp.API` | |
| `application.properties` | `appsettings.json` | JSON format |
| `@SpringBootApplication` | `Program.cs` | |
| `@RestController` | `[ApiController]` + `: ControllerBase` | |
| `@GetMapping` / `@PostMapping` | `[HttpGet]` / `[HttpPost]` | |
| `@RequestBody` | `[FromBody]` | |
| `@PathVariable` | `{id}` in route + matching param name | |
| `@Autowired` constructor | Constructor injection — no annotation | |
| `@Service` | Register with `services.AddScoped<IFoo, Foo>()` | |
| JPA / Hibernate | Entity Framework Core (EF Core) | |
| `EntityManager` | `DbContext` | |
| `JpaRepository` | `DbContext.DbSet<T>` + custom repo | |
| `Optional<T>` | `T?` — nullable type | |
| `CompletableFuture<T>` | `Task<T>` | |
| `boolean` | `bool` | |
| `BigDecimal` | `decimal` — use `m` suffix: `9.99m` | |
| `UUID` | `Guid` | |
| `super(args)` | `: base(args)` | |
| Flyway / Liquibase | `dotnet ef migrations` | |
| JUnit `@Test` | `[Fact]` (xUnit) | |
| Mockito | Moq | |
| AssertJ | FluentAssertions | |

---

## C# Syntax Gotchas

```csharp
// bool not boolean
bool isActive = true;

// decimal for money — "m" suffix required on literals
decimal price = 9.99m;

// Guid instead of UUID
Guid id = Guid.NewGuid();

// Nullable types — ? means "might be null"
Product? product = null;             // like Optional<Product>
var name = product?.Name;            // ?. = null-safe — returns null instead of throwing
var name = product?.Name ?? "N/A";   // ?? = fallback if null

// String interpolation
var msg = $"Hello, {name}!";         // like String.format("Hello, %s!", name)

// Object initialiser — no builder / Lombok needed
var p = new Product { Name = "Widget", Price = 9.99m };

// Expression body — shorthand for single-line methods
public string GetLabel() => $"{Name}: {Price}"; // same as { return ...; }

// Null checks — modern style
if (product is null) return NotFound();
if (product is not null) { /* ... */ }

// async/await — same idea as Java
public async Task<Product?> GetAsync(int id)  // Task<T> = CompletableFuture<T>
{
    return await _db.Products.FindAsync(id);  // await = like .get() but non-blocking
}

// var = type inference — same as Java var
var products = new List<Product>();

// super() = base()
public MyClass(string name) : base(name) { }
```

---

## ✅ Setup Checklist

- [ ] Project created with `dotnet new webapi`
- [ ] EF Core packages installed
- [ ] `dotnet tool install --global dotnet-ef` done
- [ ] `appsettings.json` has the connection string
- [ ] `AppDbContext` created with `DbSet<Product>`
- [ ] `DbContext` registered in `Program.cs`
- [ ] Service and repository interfaces + implementations created
- [ ] Both registered in `Program.cs` with `AddScoped`
- [ ] Controller created with routes
- [ ] `dotnet build` passes
- [ ] First migration created and applied
- [ ] `dotnet run` starts — Swagger loads at `/swagger`

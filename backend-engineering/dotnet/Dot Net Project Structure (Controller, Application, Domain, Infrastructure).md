# NET Project Structure — Controller, Application, Domain, Infrastructure

> Same Controller → Service → Repository flow you know from Spring Boot,
> but split across 4 separate projects with strict boundaries enforced by the compiler.

---

## 🗺️ The Big Picture

In Spring Boot everything lives in one project. Here each layer is its own project. The flow of a request is the same — but the layers can only talk to each other in one direction. If Infrastructure tries to call the API layer, it won't compile.

```
HTTP Request
      │
      ▼
┌─────────────────────────────────────────────┐
│  MyApp.API                                  │
│  Controllers, Program.cs                    │  ← outermost layer
│  Receives HTTP. Calls Service. Returns response.
│  Knows about: Application only.             │
└──────────────────┬──────────────────────────┘
                   │ calls IProductService
                   ▼
┌─────────────────────────────────────────────┐
│  MyApp.Application                          │
│  Services, DTOs                             │
│  Business logic. Calls Repository interface.│
│  Knows about: Domain only.                  │
└──────────────────┬──────────────────────────┘
                   │ calls IProductRepository (interface defined in Domain)
                   ▼
┌─────────────────────────────────────────────┐
│  MyApp.Domain                               │
│  Entities, Repository Interfaces, Exceptions│  ← innermost layer
│  Pure C#. No DB. No HTTP. No dependencies.  │
└──────────────────┬──────────────────────────┘
                   ↑ implements IProductRepository
┌─────────────────────────────────────────────┐
│  MyApp.Infrastructure                       │
│  EF Core, DbContext, Repository impl.       │
│  All DB work lives here.                    │
│  Knows about: Domain + Application.         │
└─────────────────────────────────────────────┘
```

**The key rule:** Domain defines contracts (interfaces). Infrastructure implements them. Application uses them. API wires everything together. Each layer only sees what it needs.

---

## 📁 Folder Structure

```
MyApp/
│
├── MyApp.sln                                  # Groups all 4 projects — like a root pom.xml
│
└── src/
    │
    ├── MyApp.Domain/                          # ① Build this first — no dependencies
    │   ├── MyApp.Domain.csproj                #   No NuGet packages at all
    │   ├── Entities/
    │   │   └── Product.cs                     #   Plain C# entity — no EF annotations
    │   ├── Interfaces/
    │   │   └── IProductRepository.cs          #   Contract defined here, implemented in Infra
    │   └── Exceptions/
    │       └── DomainException.cs             #   Thrown when a business rule is violated
    │
    ├── MyApp.Infrastructure/                  # ② Build second — implements Domain contracts
    │   ├── MyApp.Infrastructure.csproj        #   References: MyApp.Domain
    │   ├── DependencyInjection.cs             #   Registers DbContext + repositories
    │   ├── Persistence/
    │   │   ├── AppDbContext.cs                #   EF Core session — like EntityManager
    │   │   └── Configurations/
    │   │       └── ProductConfiguration.cs    #   Maps entity to DB table — like @Table/@Column
    │   └── Repositories/
    │       └── ProductRepository.cs           #   Implements IProductRepository using EF Core
    │
    ├── MyApp.Application/                     # ③ Build third — business logic layer
    │   ├── MyApp.Application.csproj           #   References: MyApp.Domain
    │   ├── DependencyInjection.cs             #   Registers services
    │   ├── Services/
    │   │   ├── IProductService.cs             #   Service contract
    │   │   └── ProductService.cs              #   Business logic — calls IProductRepository
    │   └── DTOs/
    │       ├── CreateProductRequest.cs        #   Input — what the client sends
    │       └── ProductResponse.cs             #   Output — what you return
    │
    └── MyApp.API/                             # ④ Build last — HTTP entry point
        ├── MyApp.API.csproj                   #   References: MyApp.Application + MyApp.Infrastructure
        ├── Program.cs                         #   Wires DI, middleware, routing
        ├── appsettings.json                   #   DB connection string, config
        ├── appsettings.Development.json       #   Local dev overrides
        └── Controllers/
            └── ProductsController.cs          #   HTTP endpoints — calls IProductService
```

**Dependency direction:**
```
API  →  Application  →  Domain  ←  Infrastructure
```
Arrow = "knows about". Domain has no arrows out — it knows nothing about anything else.

---

## 🛠️ CLI — Scaffold Everything

Run these once from the root `MyApp/` folder.

```bash
# Create the solution
dotnet new sln -n MyApp

# Create all 4 projects
dotnet new classlib -n MyApp.Domain         -o src/MyApp.Domain
dotnet new classlib -n MyApp.Infrastructure -o src/MyApp.Infrastructure
dotnet new classlib -n MyApp.Application    -o src/MyApp.Application
dotnet new webapi   -n MyApp.API            -o src/MyApp.API
dotnet new xunit    -n MyApp.UnitTests      -o tests/MyApp.UnitTests

# Register all in the solution
dotnet sln add src/MyApp.Domain/MyApp.Domain.csproj
dotnet sln add src/MyApp.Infrastructure/MyApp.Infrastructure.csproj
dotnet sln add src/MyApp.Application/MyApp.Application.csproj
dotnet sln add src/MyApp.API/MyApp.API.csproj
dotnet sln add tests/MyApp.UnitTests/MyApp.UnitTests.csproj

# Wire references — enforces the dependency direction
dotnet add src/MyApp.Infrastructure reference src/MyApp.Domain/MyApp.Domain.csproj
dotnet add src/MyApp.Application    reference src/MyApp.Domain/MyApp.Domain.csproj
dotnet add src/MyApp.API            reference src/MyApp.Application/MyApp.Application.csproj
dotnet add src/MyApp.API            reference src/MyApp.Infrastructure/MyApp.Infrastructure.csproj
dotnet add tests/MyApp.UnitTests    reference src/MyApp.Application/MyApp.Application.csproj
dotnet add tests/MyApp.UnitTests    reference src/MyApp.Domain/MyApp.Domain.csproj

# EF Core goes in Infrastructure only — other layers never need it
dotnet add src/MyApp.Infrastructure package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/MyApp.Infrastructure package Microsoft.EntityFrameworkCore.Tools

# EF CLI tool — install once per machine
dotnet tool install --global dotnet-ef

# Day-to-day
dotnet build
dotnet run --project src/MyApp.API
dotnet watch --project src/MyApp.API run  # hot reload
dotnet test
```

---

## ① MyApp.Domain — Entity, Interface, Exception

Start here. This layer has no dependencies on anything — not even NuGet packages. It defines the entity and the repository interface contract. Every other layer references this, so it must exist first.

The most important thing here is `IProductRepository`. The interface lives in Domain so the Application layer can call it without knowing EF Core exists. Infrastructure then implements it.

```csharp
// src/MyApp.Domain/Entities/Product.cs
// Plain C# — no EF Core annotations.
// DB mapping is handled separately in Infrastructure/Configurations/.

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }   // decimal = BigDecimal — always use for money
    public bool IsActive { get; set; } = true;
}


// src/MyApp.Domain/Interfaces/IProductRepository.cs
// The repository CONTRACT — defined here in Domain, implemented in Infrastructure.
// Application calls this interface. It never sees ProductRepository directly.

public interface IProductRepository
{
    Task<List<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);    // Product? = might be null, like Optional<Product>
    Task<Product> AddAsync(Product product); // receives a built entity from the service
    Task<bool> DeleteAsync(int id);          // returns false if not found
}


// src/MyApp.Domain/Exceptions/DomainException.cs
// Thrown by the service when a business rule is violated.
// Caught by middleware in the API layer and turned into a 400 response.
// Like a custom RuntimeException in Java.

public class DomainException : Exception
{
    public DomainException(string message) : base(message) { }
    // ": base(message)" = call parent constructor — like super(message) in Java
}
```

---

## ② MyApp.Infrastructure — DbContext, Config, Repository

Build this second. This is the only layer that knows about EF Core. It implements `IProductRepository` from Domain. The Application layer calls the interface — at runtime the DI container injects this concrete class.

`AppDbContext` is EF Core's session with the database (like `EntityManager`). `ProductConfiguration` maps the entity to the DB table — equivalent to `@Table` and `@Column` in JPA, but kept in a separate file so the Domain entity stays clean.

```csharp
// src/MyApp.Infrastructure/Persistence/AppDbContext.cs
// EF Core's DB session. ProductRepository calls this to run queries.
// AddAsync, FindAsync, Remove, ToListAsync are all built into DbSet<T> by EF Core —
// like JpaRepository giving you save(), findById(), findAll() for free.

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    // ": base(options)" = call parent constructor — like super(options) in Java

    public DbSet<Product> Products { get; set; }
    // Represents the products table. Use _db.Products to query, insert, delete.

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Scans this assembly and applies all IEntityTypeConfiguration<T> classes automatically.
        // ProductConfiguration below will be picked up here.
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
        base.OnModelCreating(modelBuilder);
    }
}


// src/MyApp.Infrastructure/Persistence/Configurations/ProductConfiguration.cs
// Maps Product entity fields to DB columns.
// Like @Table(name="products") @Column(nullable=false) in JPA — but in a separate file.

public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> b)
    {
        b.ToTable("products");
        b.HasKey(p => p.Id);
        b.Property(p => p.Name).IsRequired().HasMaxLength(100); // NOT NULL VARCHAR(100)
        b.Property(p => p.Price).HasColumnType("decimal(18,2)");
    }
}


// src/MyApp.Infrastructure/Repositories/ProductRepository.cs
// Implements IProductRepository from Domain using EF Core.
// Application never sees this class — it only ever sees the interface.

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;

    public ProductRepository(AppDbContext db) => _db = db;

    public async Task<List<Product>> GetAllAsync()
        => await _db.Products.ToListAsync();
    // Runs: SELECT * FROM products

    public async Task<Product?> GetByIdAsync(int id)
        => await _db.Products.FindAsync(id);
    // Runs: SELECT * FROM products WHERE Id = id

    public async Task<Product> AddAsync(Product product)
    {
        await _db.Products.AddAsync(product); // stage the insert
        await _db.SaveChangesAsync();         // commit — EF generates INSERT SQL
        return product;                       // product.Id is populated by EF after save
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return false;

        _db.Products.Remove(product);
        await _db.SaveChangesAsync();         // commit — EF generates DELETE SQL
        return true;
    }
}


// src/MyApp.Infrastructure/DependencyInjection.cs
// Registers this layer's services with the DI container.
// Called from Program.cs as: builder.Services.AddInfrastructureServices(config)

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructureServices(
        this IServiceCollection services, IConfiguration config)
    {
        // Register EF Core. Swap UseSqlServer() for UseNpgsql() or UseSqlite() as needed.
        services.AddDbContext<AppDbContext>(o =>
            o.UseSqlServer(config.GetConnectionString("DefaultConnection")));
        // Connection string from appsettings.json → "ConnectionStrings": { "DefaultConnection": "..." }

        // When someone asks for IProductRepository, give them ProductRepository.
        // AddScoped = one instance per HTTP request — right for anything touching the DB.
        services.AddScoped<IProductRepository, ProductRepository>();

        return services;
    }
}
```

---

## ③ MyApp.Application — DTOs, Service Interface, Service

Build this third. The service contains business logic and calls `IProductRepository` from Domain. It never calls `ProductRepository` directly — it has no idea EF Core exists. DTOs live here too because they are the Application's public contract with the outside world.

```csharp
// src/MyApp.Application/DTOs/CreateProductRequest.cs
// What the client sends in the request body — like @RequestBody DTO in Spring.
public class CreateProductRequest
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}

// src/MyApp.Application/DTOs/ProductResponse.cs
// What you send back to the client — never expose the raw entity.
public class ProductResponse
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}


// src/MyApp.Application/Services/IProductService.cs
// The service contract the Controller depends on.
public interface IProductService
{
    Task<List<ProductResponse>> GetAllAsync();
    Task<ProductResponse?> GetByIdAsync(int id);
    Task<ProductResponse> CreateAsync(CreateProductRequest request);
    Task<bool> DeleteAsync(int id);
}


// src/MyApp.Application/Services/ProductService.cs
// Business logic lives here. Calls IProductRepository — never the concrete class.
// No EF Core, no DbContext — that detail is hidden behind the interface.

public class ProductService : IProductService
{
    private readonly IProductRepository _repo;
    // At runtime, DI injects ProductRepository from Infrastructure here.
    // ProductService itself never knows that — it only sees the interface.

    public ProductService(IProductRepository repo) => _repo = repo;

    public async Task<List<ProductResponse>> GetAllAsync()
    {
        var products = await _repo.GetAllAsync();
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
        return new ProductResponse { Id = product.Id, Name = product.Name, Price = product.Price };
    }

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        // Business rules live in the service — the repository knows nothing about these
        if (string.IsNullOrWhiteSpace(request.Name))
            throw new DomainException("Product name is required.");
        // string.IsNullOrWhiteSpace = null OR empty OR whitespace — like name == null || name.isBlank()

        if (request.Price <= 0)
            throw new DomainException("Price must be greater than zero.");

        // Build the entity — fully ready to hand off to the repository
        var product = new Product { Name = request.Name, Price = request.Price };

        // Repository handles all DB work from here
        var saved = await _repo.AddAsync(product);

        return new ProductResponse { Id = saved.Id, Name = saved.Name, Price = saved.Price };
    }

    public async Task<bool> DeleteAsync(int id)
    {
        return await _repo.DeleteAsync(id); // repository returns false if not found
    }
}


// src/MyApp.Application/DependencyInjection.cs
// Registers this layer's services.
// Called from Program.cs as: builder.Services.AddApplicationServices()

public static class DependencyInjection
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        // When Controller asks for IProductService, give it ProductService.
        return services;
    }
}
```

---

## ④ MyApp.API — Program.cs and Controller

Build this last. The API layer is the HTTP entry point. `Program.cs` wires everything together by calling the DI registration methods from Application and Infrastructure. The controller calls `IProductService` from Application — it has no knowledge of repositories, EF Core, or the database.

```csharp
// src/MyApp.API/Program.cs
// Runs once at startup. Two jobs: register services, then configure the HTTP pipeline.

var builder = WebApplication.CreateBuilder(args);
// Reads appsettings.json, sets up logging, creates the DI container.

// Each layer registers its own services — Program.cs just calls one method per layer.
// This is why DependencyInjection.cs exists in Application and Infrastructure.
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);

builder.Services.AddControllers();
builder.Services.AddSwaggerGen();
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();
// Services are locked after this. Below is the HTTP pipeline — like a chain of Servlet Filters.

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(); // visit /swagger to test all endpoints
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers(); // maps routes to [ApiController] classes

app.Run(); // start the server — like embedded Tomcat


// src/MyApp.API/Controllers/ProductsController.cs
// HTTP entry point. Calls IProductService. Returns HTTP responses.
// No business logic, no DB calls — just HTTP in, response out.

[ApiController]
[Route("api/[controller]")]  // [controller] → "Products" → route = api/products
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    // IProductService from Application — DI injects ProductService at runtime

    public ProductsController(IProductService service) => _service = service;

    // GET /api/products
    [HttpGet]
    public async Task<ActionResult<List<ProductResponse>>> GetAll()
    {
        var products = await _service.GetAllAsync();
        return Ok(products); // HTTP 200
    }

    // GET /api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductResponse>> GetById(int id)
    {
        var product = await _service.GetByIdAsync(id);
        if (product is null) return NotFound(); // HTTP 404
        return Ok(product);                     // HTTP 200
    }

    // POST /api/products
    [HttpPost]
    public async Task<ActionResult<ProductResponse>> Create([FromBody] CreateProductRequest request)
    // [FromBody] = deserialise the JSON body — like @RequestBody
    {
        var created = await _service.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created); // HTTP 201
    }

    // DELETE /api/products/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var deleted = await _service.DeleteAsync(id);
        if (!deleted) return NotFound(); // HTTP 404
        return NoContent();              // HTTP 204
    }
}
```

---

## appsettings.json

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

## EF Core Migrations

Migrations live in Infrastructure because that's where `AppDbContext` is. You must point the CLI at both the Infrastructure project (for DbContext) and the API project (for appsettings.json).

```bash
# Run after every entity change — generates a migration file in Infrastructure/Migrations/
dotnet ef migrations add InitialCreate \
  --project src/MyApp.Infrastructure \
  --startup-project src/MyApp.API

# Apply all pending migrations to the database
dotnet ef database update \
  --project src/MyApp.Infrastructure \
  --startup-project src/MyApp.API

# Undo the last migration (only if not yet applied to the DB)
dotnet ef migrations remove \
  --project src/MyApp.Infrastructure \
  --startup-project src/MyApp.API
```

---

## What Changed vs the Spring Boot Style

| | Spring Boot Style | Clean Architecture |
|---|---|---|
| **Projects** | 1 project | 4 separate projects |
| **Entity** | In same project | `MyApp.Domain` — no EF annotations |
| **Repository interface** | In same project | `MyApp.Domain` — owned by inner layer |
| **Repository implementation** | In same project | `MyApp.Infrastructure` — only place with EF Core |
| **Business logic** | `ProductService` in same project | `MyApp.Application` |
| **DB mapping config** | On entity or in DbContext | Separate `ProductConfiguration` class in Infra |
| **DI registration** | All in `Program.cs` | One `DependencyInjection.cs` per layer |
| **Can swap the DB?** | Yes, but refactoring needed | Yes — only touch `MyApp.Infrastructure` |
| **When to use** | Small projects, learning | Production apps, larger teams |

---

## ✅ Setup Checklist

Work through this in order — each step depends on the one before it.

- [ ] All 4 projects created and added to solution
- [ ] Project references wired in the correct direction
- [ ] **Domain:** `Product.cs`, `IProductRepository.cs`, `DomainException.cs` created
- [ ] **Infrastructure:** `AppDbContext`, `ProductConfiguration`, `ProductRepository` created
- [ ] **Infrastructure:** `DependencyInjection.cs` registers DbContext and repository
- [ ] EF Core packages installed in `MyApp.Infrastructure` only
- [ ] `dotnet tool install --global dotnet-ef` done
- [ ] **Application:** DTOs, `IProductService`, `ProductService` created
- [ ] **Application:** `DependencyInjection.cs` registers the service
- [ ] **API:** `Program.cs` calls both `AddApplicationServices()` and `AddInfrastructureServices()`
- [ ] **API:** `ProductsController` created
- [ ] `appsettings.json` has the connection string
- [ ] `dotnet build` passes with no errors
- [ ] Migration created and applied to DB
- [ ] `dotnet run` starts — Swagger loads at `/swagger`

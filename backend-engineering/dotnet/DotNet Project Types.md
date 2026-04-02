# .NET Project Types

## SDK-Style Project File

All modern .NET projects use the SDK-style `.csproj`:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

---

## Project Types Overview

### Console Application
```xml
<OutputType>Exe</OutputType>
<TargetFramework>net9.0</TargetFramework>
```
- Entry point: `static void Main` or top-level statements
- Use for: CLI tools, background workers, scripts

```bash
dotnet new console -n MyApp
```

---

### Class Library
```xml
<OutputType>Library</OutputType>  <!-- This is the default, can be omitted -->
<TargetFramework>net9.0</TargetFramework>
```
- No entry point; produces a `.dll`
- Use for: shared logic, reusable packages, domain layer

```bash
dotnet new classlib -n MyLibrary
```

---

### ASP.NET Core Web API
```xml
<Sdk>Microsoft.NET.Sdk.Web</Sdk>
```
- Runs a Kestrel HTTP server
- Use for: REST APIs, microservices, backend services

```bash
dotnet new webapi -n MyApi
```

Key files: `Program.cs`, `appsettings.json`, controller classes or minimal API endpoints.

---

### ASP.NET Core MVC / Razor Pages
```bash
dotnet new mvc -n MyMvcApp
dotnet new razor -n MyRazorApp
```
- MVC: Controllers + Views pattern
- Razor Pages: page-centric model, simpler for CRUD UIs
- Both render server-side HTML

---

### Blazor
```bash
dotnet new blazorserver -n MyBlazorApp   # Server-side, SignalR
dotnet new blazorwasm  -n MyBlazorWasm   # WebAssembly, runs in browser
```
- Write UI in C# + Razor instead of JavaScript
- Blazor Server: thin client, server holds state
- Blazor WASM: full client-side, larger download

---

### Worker Service
```xml
<Sdk>Microsoft.NET.Sdk.Worker</Sdk>
```
```bash
dotnet new worker -n MyWorker
```
- Long-running background service
- Use for: message consumers, scheduled jobs, daemons
- Implements `IHostedService` / `BackgroundService`

---

### xUnit / NUnit / MSTest (Test Projects)
```bash
dotnet new xunit  -n MyApp.Tests
dotnet new nunit  -n MyApp.Tests
dotnet new mstest -n MyApp.Tests
```
- References the project under test
- Run with: `dotnet test`

---

## Multi-Project Solutions

A `.sln` file ties multiple projects together:
```
MySolution/
├── MySolution.sln
├── src/
│   ├── MyApp.Api/          ← webapi project
│   ├── MyApp.Domain/       ← classlib
│   └── MyApp.Infrastructure/ ← classlib
└── tests/
    └── MyApp.Tests/        ← xunit project
```

```bash
dotnet new sln -n MySolution
dotnet sln add src/MyApp.Api/MyApp.Api.csproj
dotnet sln add tests/MyApp.Tests/MyApp.Tests.csproj
```

---

## Target Frameworks

| TFM | Framework |
|---|---|
| `net9.0` | .NET 9 (latest LTS: .NET 8) |
| `netstandard2.0` | Multi-target (legacy compat) |
| `net481` | .NET Framework 4.8.1 (Windows only) |
| `net9.0-windows` | .NET 9 with Windows-specific APIs |

Multi-targeting:
```xml
<TargetFrameworks>net8.0;net9.0;netstandard2.0</TargetFrameworks>
```

---

### Key `dotnet` CLI Commands

```bash
dotnet new <template>       # Create project
dotnet build                # Compile
dotnet run                  # Build + run
dotnet test                 # Run tests
dotnet publish              # Publish for deployment
dotnet add package <name>   # Add NuGet package
dotnet add reference <path> # Add project reference
```

---

## Quick Decision Guide

| I need to build… | Use |
|---|---|
| A REST API | `webapi` |
| A shared library / domain logic | `classlib` |
| A CLI tool | `console` |
| A scheduled/background job | `worker` |
| A full web UI (server-rendered) | `mvc` or `razor` |
| A browser app in C# | `blazorwasm` |
| Tests | `xunit` (preferred) |

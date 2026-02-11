# .NET SDK Setup Guide

## Overview

This guide covers:

- Installing .NET SDK and VS Code
- Setting up essential extensions
- Creating and running .NET projects
- Testing APIs and exploring databases
- Professional CLI workflow

---

## 1. Installing .NET SDK

### Download and Install

1. Visit [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)
2. Download the **latest LTS (Long Term Support)** version
3. Install it

### Verify Installation

Open terminal (PowerShell/CMD):

```bash
dotnet --version
```

Expected output:

```
8.0.101
```

If not working:

- Restart terminal
- Restart machine
- Reinstall SDK

---

## 2. Installing Visual Studio Code

1. Download from [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. Install and open VS Code

---

## 3. Required VS Code Extensions

Open Extensions panel (`Ctrl + Shift + X`) and install:

### C# Dev Kit (Microsoft)

- Project understanding
- IntelliSense
- Debugging
- Essential for C# development

### REST Client (by Huachao Mao)

- Test APIs without Postman
- Send HTTP requests from `.http` files

### SQLite (by alexcvzz)

- Explore SQLite databases visually
- Inspect tables and run queries

---

## 4. Essential .NET CLI Commands

The CLI is critical for professional .NET development.

### Key Commands

```bash
dotnet new         # Create new project from template
dotnet build       # Compile the project
dotnet run         # Build and execute
dotnet restore     # Restore dependencies
dotnet add package # Add NuGet package
dotnet list package # List installed packages
```

### List Available Templates

```bash
dotnet new list
```

Common templates:

- `console` - Console application
- `webapi` - Web API
- `web` - Web application
- `classlib` - Class library
- `mvc` - MVC application

---

## 5. Creating Your First Console App

Open terminal in VS Code (`Ctrl + \``):

```bash
mkdir MyFirstApp
cd MyFirstApp
dotnet new console
dotnet run
```

Expected output:

```
Hello, World!
```

---

## 6. Creating a Web API

```bash
dotnet new webapi -n GameStore.Api
cd GameStore.Api
dotnet run
```

The terminal will show:

```
Now listening on: https://localhost:7001
```

Open browser and navigate to:

```
https://localhost:7001/swagger
```

---

## 7. Testing APIs with REST Client

Create a file named `api.http` in your project root:

```http
GET https://localhost:7001/weatherforecast
```

Click "Send Request" to see the response in VS Code.

---

## 8. Working with SQLite

### Add SQLite Package

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

### Using SQLite Extension

1. Click SQLite icon in sidebar
2. Open the `.db` file
3. Expand tables
4. Run queries

Example query:

```sql
SELECT * FROM Games;
```

---

## 9. Project Structure

Typical Web API structure:

```
GameStore.Api/
 ├── Controllers/
 ├── Models/
 ├── Program.cs
 ├── appsettings.json
 └── GameStore.Api.csproj
```

### Important Files

**`.csproj`** - Defines target framework, packages, and project metadata

**`Program.cs`** - Application entry point

---

## 10. Complete Workflow Example

### Step 1: Create API

```bash
dotnet new webapi -n Inventory.Api
cd Inventory.Api
```

### Step 2: Run

```bash
dotnet run
```

### Step 3: Test Endpoint

Create `api.http`:

```http
GET https://localhost:7001/weatherforecast
```

Click "Send Request"

### Step 4: Add Custom Endpoint

Modify `Program.cs`:

```csharp
app.MapGet("/hello", () => "Hello World!");
```

Run and test:

```bash
dotnet run
```

```http
GET https://localhost:7001/hello
```

---

## 11. Common Mistakes to Avoid

❌ **Running in wrong folder** - Always run in folder containing `.csproj`

❌ **Using wrong URL/port** - Check terminal output for correct URL

❌ **Manually editing bin/ or obj/** - Never do this

❌ **HTTPS certificate issues** - Trust certificates if needed:

```bash
dotnet dev-certs https --trust
```

---

## 12. Professional Tips

✅ Always use CLI for project operations

✅ Learn core commands by heart

✅ Keep SDK updated but stick to LTS for team projects

✅ Read build errors carefully - never ignore them

---

## Quick Reference

```bash
# Create new console app
dotnet new console -n MyApp

# Create new Web API
dotnet new webapi -n MyApi

# Build project
dotnet build

# Run project
dotnet run

# Add package
dotnet add package PackageName

# List packages
dotnet list package

# Trust HTTPS certificates
dotnet dev-certs https --trust
```

# REST API Fundamentals (.NET Web API)

## What Is a REST API

A REST API is a set of HTTP endpoints that allow clients (browser, mobile app, frontend) to interact with your server using standard HTTP methods.

**Client sends:**

- HTTP method
- URL
- Optional body

**Server returns:**

- Status code
- Data (usually JSON)

---

## Core REST Concepts

### 1. Resources

A resource is a **noun** representing an entity.

**Good examples:**

- Users
- Products
- Orders
- Games

**Bad examples (verb-based):**

- getUsers
- createUser
- deleteOrder

REST is noun-based, not verb-based.

### 2. HTTP Methods

| Method | Purpose          | Example             |
| ------ | ---------------- | ------------------- |
| GET    | Read data        | Get all products    |
| POST   | Create resource  | Create product      |
| PUT    | Replace existing | Update full product |
| PATCH  | Partial update   | Update only price   |
| DELETE | Remove resource  | Delete product      |

### 3. URL Structure

**Bad:**

```
/getAllUsers
/createUser
/deleteUser
```

**Good REST style:**

```
GET    /users
GET    /users/5
POST   /users
PUT    /users/5
DELETE /users/5
```

The HTTP method carries the action. The URL represents the resource.

---

## HTTP Status Codes

| Code | Meaning                              |
| ---- | ------------------------------------ |
| 200  | OK - Successful GET                  |
| 201  | Created - Successful POST            |
| 204  | No Content - Successful DELETE/PUT   |
| 400  | Bad Request - Invalid input          |
| 404  | Not Found - Resource doesn't exist   |
| 500  | Internal Server Error - Server crash |

---

## JSON in REST APIs

REST APIs typically send and receive JSON.

Example response:

```json
{
  "id": 1,
  "name": "FIFA 24",
  "price": 59.99
}
```

.NET automatically converts C# objects ↔ JSON.

---

## Building a REST API

### Step 1: Create Project

```bash
dotnet new webapi -n GameStore.Api
cd GameStore.Api
dotnet run
```

Open browser:

```
https://localhost:7001/swagger
```

### Step 2: Understanding Program.cs (Minimal API)

Basic structure:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World");

app.Run();
```

Each `MapX` method defines an endpoint.

---

## Complete CRUD Example

Replace `Program.cs` with:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var games = new List<Game>
{
    new Game(1, "FIFA 24", 59.99m),
    new Game(2, "GTA V", 39.99m)
};

// GET all games
app.MapGet("/games", () => games);

// GET game by ID
app.MapGet("/games/{id}", (int id) =>
{
    var game = games.FirstOrDefault(g => g.Id == id);

    return game is not null
        ? Results.Ok(game)
        : Results.NotFound();
});

// POST new game
app.MapPost("/games", (Game newGame) =>
{
    games.Add(newGame);
    return Results.Created($"/games/{newGame.Id}", newGame);
});

// DELETE game
app.MapDelete("/games/{id}", (int id) =>
{
    var game = games.FirstOrDefault(g => g.Id == id);

    if (game is null)
        return Results.NotFound();

    games.Remove(game);
    return Results.NoContent();
});

app.Run();

record Game(int Id, string Name, decimal Price);
```

---

## Testing with REST Client

Create `api.http` file:

### Get All Games

```http
GET https://localhost:7001/games
```

### Get Game by ID

```http
GET https://localhost:7001/games/1
```

### Create New Game

```http
POST https://localhost:7001/games
Content-Type: application/json

{
  "id": 3,
  "name": "Cyberpunk 2077",
  "price": 49.99
}
```

### Delete Game

```http
DELETE https://localhost:7001/games/2
```

---

## How ASP.NET Core Handles Requests

When a request arrives:

1. **Routing** matches the URL
2. **Model binding** converts JSON → C# object
3. Your code executes
4. Result converted → JSON
5. Correct status code returned

ASP.NET Core automatically handles:

- Serialization/Deserialization
- Routing
- Model binding
- HTTP pipeline

---

## REST Design Best Practices

### 1. Use Plural Nouns

```
/users
/products
/orders
```

### 2. No Verbs in URLs

❌ `/createUser`
❌ `/deleteProduct`

✅ `POST /users`
✅ `DELETE /products/5`

### 3. Return Correct Status Codes

Always return appropriate HTTP status codes.

### 4. Keep Endpoints Predictable

Frontend developers should be able to guess your routes.

---

## Path vs Query Parameters

### Path Parameters

Used to identify a specific resource:

```
/games/5
```

### Query Parameters

Used for filtering, sorting, pagination:

```
/games?maxPrice=50
```

**Example:**

```csharp
app.MapGet("/games", (decimal? maxPrice) =>
{
    var result = games
        .Where(g => !maxPrice.HasValue || g.Price <= maxPrice.Value);

    return result;
});
```

**Test:**

```http
GET https://localhost:7001/games?maxPrice=50
```

---

## DTOs (Data Transfer Objects)

Never expose database entities directly. Use DTOs for security and control.

**Example:**

```csharp
record CreateGameDto(string Name, decimal Price);
```

**Usage:**

```csharp
app.MapPost("/games", (CreateGameDto dto) =>
{
    var newGame = new Game(
        games.Max(g => g.Id) + 1,
        dto.Name,
        dto.Price
    );

    games.Add(newGame);

    return Results.Created($"/games/{newGame.Id}", newGame);
});
```

**Why use DTOs?**

- Security
- Control over what data is exposed
- Clean API contracts

---

## Idempotency

Understanding which operations are idempotent (calling multiple times gives same result):

- **GET** → Safe and idempotent
- **DELETE** → Idempotent
- **PUT** → Idempotent
- **POST** → NOT idempotent

---

## Common Mistakes to Avoid

❌ Returning 200 when resource not found (should be 404)
❌ Using POST for everything
❌ Returning plain strings instead of JSON
❌ Hardcoding URLs
❌ Not validating input

---

## REST as Database Operations

Think of REST like a database table exposed over HTTP:

| HTTP Request    | SQL Equivalent    |
| --------------- | ----------------- |
| GET /games      | SELECT \*         |
| GET /games/1    | SELECT WHERE Id=1 |
| POST /games     | INSERT            |
| PUT /games/1    | UPDATE WHERE Id=1 |
| DELETE /games/1 | DELETE WHERE Id=1 |

---

## Quick Reference

```csharp
// GET endpoint
app.MapGet("/resource", () => { /* logic */ });

// POST endpoint
app.MapPost("/resource", (Model data) => { /* logic */ });

// PUT endpoint
app.MapPut("/resource/{id}", (int id, Model data) => { /* logic */ });

// DELETE endpoint
app.MapDelete("/resource/{id}", (int id) => { /* logic */ });

// Return results
Results.Ok(data);           // 200
Results.Created(uri, data); // 201
Results.NoContent();        // 204
Results.NotFound();         // 404
Results.BadRequest();       // 400
```

# Data Management & Architecture in ASP.NET Core

## Why Architecture Matters

In production applications, we need:

- Database integration
- Data validation
- Security
- Separation of concerns
- Maintainability

Good architecture prevents:

- Tight coupling
- Security leaks
- Broken APIs
- Massive refactoring later

---

## Core Concept: Separation of Responsibilities

Real-world backend separates:

1. **Entity** (Database Model)
2. **DTO** (API Contract Model)
3. **Business Logic**
4. **Infrastructure** (Database)

**Layer structure:**

```
Controller / Endpoint
        ↓
Service / Business Logic
        ↓
DbContext
        ↓
Database
```

---

## Entities (Database Models)

An entity represents a database table.

**Example:**

```csharp
public class Game
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

**Entities:**

- Represent persistence layer
- Contain database-related structure
- Should NOT be directly exposed to API consumers

---

## DTOs (Data Transfer Objects)

DTOs define what your API sends and receives.

**Why use DTOs?**

- Entities may contain sensitive fields
- Entity structure may change
- Full control over API contract
- Security protection

### Create DTO

```csharp
public record CreateGameDto(
    string Name,
    decimal Price
);
```

### Response DTO

```csharp
public record GameDto(
    int Id,
    string Name,
    decimal Price
);
```

**Note:** No `CreatedAt` or internal fields exposed.

---

## Why Never Return Entities Directly

If your entity later adds:

```csharp
public string InternalNotes { get; set; }
```

Your API would suddenly expose it publicly. DTOs protect against this security risk.

---

## Professional Folder Structure

```
GameStore.Api/
 ├── Entities/
 │    └── Game.cs
 ├── Dtos/
 │    ├── GameDto.cs
 │    └── CreateGameDto.cs
 ├── Data/
 │    └── AppDbContext.cs
 └── Program.cs
```

---

## Setting Up Database (EF Core + SQLite)

### Install Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Create DbContext

**File:** `Data/AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using GameStore.Api.Entities;

namespace GameStore.Api.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Game> Games => Set<Game>();
}
```

### Register DbContext in Program.cs

```csharp
using GameStore.Api.Data;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite("Data Source=gamestore.db"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
```

---

## CRUD Operations with DTO + Entity Pattern

### POST Endpoint

```csharp
app.MapPost("/games", async (
    CreateGameDto dto,
    AppDbContext dbContext) =>
{
    // Convert DTO → Entity
    var game = new Game
    {
        Name = dto.Name,
        Price = dto.Price,
        CreatedAt = DateTime.UtcNow
    };

    dbContext.Games.Add(game);
    await dbContext.SaveChangesAsync();

    // Convert Entity → Response DTO
    var response = new GameDto(
        game.Id,
        game.Name,
        game.Price
    );

    return Results.Created($"/games/{game.Id}", response);
});
```

**Flow:** DTO → Entity → Save → Response DTO

---

## Data Validation

Clients will send bad data. Always validate input.

### Data Annotations

```csharp
using System.ComponentModel.DataAnnotations;

public record CreateGameDto(
    [Required]
    [StringLength(100)]
    string Name,

    [Range(1, 1000)]
    decimal Price
);
```

**Common Annotations:**

- `[Required]` - Field must exist
- `[StringLength]` - Limits string size
- `[Range]` - Numeric constraints
- `[EmailAddress]` - Email format validation
- `[MinLength]` / `[MaxLength]` - Length constraints

### Manual Validation (Minimal API)

```csharp
app.MapPost("/games", async (
    CreateGameDto dto,
    AppDbContext dbContext) =>
{
    if (string.IsNullOrWhiteSpace(dto.Name))
        return Results.BadRequest("Name is required.");

    if (dto.Price <= 0)
        return Results.BadRequest("Price must be positive.");

    var game = new Game
    {
        Name = dto.Name,
        Price = dto.Price,
        CreatedAt = DateTime.UtcNow
    };

    dbContext.Games.Add(game);
    await dbContext.SaveChangesAsync();

    return Results.Created($"/games/{game.Id}", game);
});
```

---

## GET with Projection (Performance Optimization)

Use projection to avoid loading unnecessary fields:

```csharp
app.MapGet("/games", async (AppDbContext dbContext) =>
{
    var games = await dbContext.Games
        .Select(g => new GameDto(
            g.Id,
            g.Name,
            g.Price))
        .ToListAsync();

    return games;
});
```

**Benefits:**

- Improved performance
- Only loads needed fields
- Clean API contract

---

## Handling Not Found (404)

```csharp
app.MapGet("/games/{id}", async (
    int id,
    AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    if (game is null)
        return Results.NotFound();

    return Results.Ok(new GameDto(
        game.Id,
        game.Name,
        game.Price));
});
```

**Important:** Always return 404 when resource not found. Never return null with 200.

---

## PUT Endpoint Example

```csharp
app.MapPut("/games/{id}", async (
    int id,
    CreateGameDto dto,
    AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    if (game is null)
        return Results.NotFound();

    game.Name = dto.Name;
    game.Price = dto.Price;

    await dbContext.SaveChangesAsync();

    return Results.NoContent();
});
```

---

## DELETE Endpoint Example

```csharp
app.MapDelete("/games/{id}", async (
    int id,
    AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    if (game is null)
        return Results.NotFound();

    dbContext.Games.Remove(game);
    await dbContext.SaveChangesAsync();

    return Results.NoContent();
});
```

---

## Database Migrations

### Create Migration

```bash
dotnet ef migrations add InitialCreate
```

### Apply Migration

```bash
dotnet ef database update
```

### Remove Last Migration

```bash
dotnet ef migrations remove
```

This creates and maintains database schema professionally.

---

## Common Mistakes to Avoid

❌ Returning Entity directly
❌ No validation
❌ No DTO separation
❌ Business logic inside endpoints
❌ Ignoring async/await
❌ Returning wrong status codes
❌ Not handling null cases

---

## Mental Model Summary

| Concept     | Purpose                  |
| ----------- | ------------------------ |
| Entity      | Database shape           |
| DTO         | API shape                |
| DbContext   | Database manager         |
| Annotations | Input validation         |
| Endpoint    | HTTP entry point         |
| Projection  | Performance optimization |

---

## Complete Example: Full CRUD API

```csharp
using GameStore.Api.Data;
using GameStore.Api.Entities;
using GameStore.Api.Dtos;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite("Data Source=gamestore.db"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

// GET all
app.MapGet("/games", async (AppDbContext db) =>
    await db.Games
        .Select(g => new GameDto(g.Id, g.Name, g.Price))
        .ToListAsync());

// GET by id
app.MapGet("/games/{id}", async (int id, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    return game is null
        ? Results.NotFound()
        : Results.Ok(new GameDto(game.Id, game.Name, game.Price));
});

// POST
app.MapPost("/games", async (CreateGameDto dto, AppDbContext db) =>
{
    var game = new Game
    {
        Name = dto.Name,
        Price = dto.Price,
        CreatedAt = DateTime.UtcNow
    };

    db.Games.Add(game);
    await db.SaveChangesAsync();

    return Results.Created($"/games/{game.Id}",
        new GameDto(game.Id, game.Name, game.Price));
});

// PUT
app.MapPut("/games/{id}", async (int id, CreateGameDto dto, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    if (game is null) return Results.NotFound();

    game.Name = dto.Name;
    game.Price = dto.Price;

    await db.SaveChangesAsync();
    return Results.NoContent();
});

// DELETE
app.MapDelete("/games/{id}", async (int id, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    if (game is null) return Results.NotFound();

    db.Games.Remove(game);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();
```

---

## Quick Reference: EF Core Commands

```bash
# Create migration
dotnet ef migrations add MigrationName

# Apply migrations
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Drop database
dotnet ef database drop

# List migrations
dotnet ef migrations list
```

# Entity Framework Core (EF Core) Guide

## What Is an ORM?

**ORM = Object Relational Mapper**

It converts:

- C# objects → Database rows
- Database rows → C# objects

**Without ORM:**

```sql
INSERT INTO Games (Name, Price) VALUES ('FIFA 24', 59.99);
```

**With EF Core:**

```csharp
dbContext.Games.Add(game);
await dbContext.SaveChangesAsync();
```

EF Core generates SQL automatically.

---

## What Is EF Core?

Entity Framework Core is:

- Microsoft's official ORM
- Lightweight and cross-platform
- Production-ready

**Supported Databases:**

- SQL Server
- SQLite
- PostgreSQL
- MySQL
- InMemory (for testing)

---

## Installation

Install required packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

**These provide:**

- Database connection
- Migrations
- CLI tools

---

## Creating an Entity (Database Model)

Create folder: `Entities`

**File:** `Entities/Game.cs`

```csharp
namespace GameStore.Api.Entities;

public class Game
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

This class automatically becomes a database table.

---

## Understanding DbContext

**DbContext is:**

- The bridge between your C# application and the database
- A session manager
- A unit of work
- A change tracker

**DbContext:**

- Tracks entities
- Generates SQL
- Executes queries
- Saves changes

---

## Creating DbContext

Create folder: `Data`

**File:** `Data/AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using GameStore.Api.Entities;

namespace GameStore.Api.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Game> Games => Set<Game>();
}
```

---

## Understanding DbSet

**DbSet** represents a database table.

```csharp
public DbSet<Game> Games
```

Means:

- Table name: `Games`
- Row type: `Game`

**Available operations:**

```csharp
dbContext.Games.Add()       // Insert
dbContext.Games.Remove()    // Delete
dbContext.Games.FindAsync() // Find by primary key
dbContext.Games.ToListAsync() // Get all
```

**Think:** DbSet = Table abstraction

---

## Registering DbContext

**File:** `Program.cs`

```csharp
using GameStore.Api.Data;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite("Data Source=gamestore.db"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
```

This tells EF Core:

- Which database to use
- Which provider (SQLite)
- Connection string

---

## Database Migrations

**Migration = Version control for your database schema**

When you change your Entity (add/remove/rename property), you must update the database structure.

### Install EF CLI Tools (if needed)

```bash
dotnet tool install --global dotnet-ef
```

### Create First Migration

```bash
dotnet ef migrations add InitialCreate
```

**What happens:**

- EF reads DbContext
- Scans Entities
- Generates migration files
- Generates SQL instructions

A new `Migrations/` folder is created with C# migration files.

### Apply Migration to Database

```bash
dotnet ef database update
```

**Result:**

- SQLite file created: `gamestore.db`
- Table `Games` created
- Database is ready

---

## CRUD Operations

### Insert Data (POST)

```csharp
app.MapPost("/games", async (AppDbContext dbContext) =>
{
    var game = new Game
    {
        Name = "FIFA 24",
        Price = 59.99m,
        CreatedAt = DateTime.UtcNow
    };

    dbContext.Games.Add(game);
    await dbContext.SaveChangesAsync();

    return Results.Created($"/games/{game.Id}", game);
});
```

### Query Data (GET)

```csharp
app.MapGet("/games", async (AppDbContext dbContext) =>
{
    return await dbContext.Games.ToListAsync();
});
```

**EF converts this to:**

```sql
SELECT * FROM Games;
```

### Get By ID

```csharp
app.MapGet("/games/{id}", async (int id, AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    return game is null
        ? Results.NotFound()
        : Results.Ok(game);
});
```

### Update Data (PUT)

```csharp
app.MapPut("/games/{id}", async (
    int id,
    AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    if (game is null)
        return Results.NotFound();

    game.Price = 49.99m;

    await dbContext.SaveChangesAsync();

    return Results.NoContent();
});
```

**Note:** EF tracks changes automatically. You just modify the property and call `SaveChangesAsync()`.

### Delete Data (DELETE)

```csharp
app.MapDelete("/games/{id}", async (
    int id,
    AppDbContext dbContext) =>
{
    var game = await dbContext.Games.FindAsync(id);

    if (game is null)
        return Results.NotFound();

    dbContext.Games.Remove(game);
    await dbContext.SaveChangesAsync();

    return Results.NoContent();
});
```

---

## How SaveChangesAsync Works

When you call:

```csharp
await dbContext.SaveChangesAsync();
```

**EF Core:**

1. Checks tracked entities
2. Detects changes
3. Builds SQL commands
4. Opens database connection
5. Executes SQL
6. Updates entity states

---

## Schema Evolution (Adding Columns)

Add a new property to your entity:

```csharp
public string Genre { get; set; } = string.Empty;
```

**Update database:**

```bash
dotnet ef migrations add AddGenreToGame
dotnet ef database update
```

Database updated safely. This is how production apps evolve.

---

## Change Tracking

EF Core automatically tracks entity changes:

```csharp
var game = await dbContext.Games.FindAsync(id);
game.Price = 100;  // Change property
await dbContext.SaveChangesAsync();  // EF generates UPDATE SQL
```

**How it works:**

- Entity is attached to context
- Context compares original vs current values
- Generates appropriate SQL

---

## DbContext Lifetime

DbContext is registered with **Scoped** lifetime by default:

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

**Meaning:**

- One DbContext per HTTP request
- Correct behavior for web applications
- **Never make it Singleton**

---

## Common Mistakes to Avoid

❌ Forgetting `SaveChangesAsync()`
❌ Forgetting to run migrations
❌ Not using async methods (`ToListAsync`, `FindAsync`)
❌ Using DbContext as Singleton
❌ Not reviewing migration code before applying

---

## EF Core Migration Commands

```bash
# Create a new migration
dotnet ef migrations add MigrationName

# Apply migrations to database
dotnet ef database update

# Remove last migration (if not applied)
dotnet ef migrations remove

# List all migrations
dotnet ef migrations list

# Drop database
dotnet ef database drop

# Update to specific migration
dotnet ef database update MigrationName

# Generate SQL script from migrations
dotnet ef migrations script
```

---

## Mental Model Summary

| Concept     | Purpose                       |
| ----------- | ----------------------------- |
| ORM         | Translator between C# and SQL |
| Entity      | Table blueprint               |
| DbSet       | Table representation          |
| DbContext   | Database manager/session      |
| Migration   | Schema version control        |
| SaveChanges | Commit transaction            |

---

## Complete Working Example

```csharp
using GameStore.Api.Data;
using GameStore.Api.Entities;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Register DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite("Data Source=gamestore.db"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

// GET all games
app.MapGet("/games", async (AppDbContext db) =>
    await db.Games.ToListAsync());

// GET game by id
app.MapGet("/games/{id}", async (int id, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    return game is null ? Results.NotFound() : Results.Ok(game);
});

// POST new game
app.MapPost("/games", async (Game game, AppDbContext db) =>
{
    game.CreatedAt = DateTime.UtcNow;
    db.Games.Add(game);
    await db.SaveChangesAsync();
    return Results.Created($"/games/{game.Id}", game);
});

// PUT update game
app.MapPut("/games/{id}", async (int id, Game updatedGame, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    if (game is null) return Results.NotFound();

    game.Name = updatedGame.Name;
    game.Price = updatedGame.Price;

    await db.SaveChangesAsync();
    return Results.NoContent();
});

// DELETE game
app.MapDelete("/games/{id}", async (int id, AppDbContext db) =>
{
    var game = await db.Games.FindAsync(id);
    if (game is null) return Results.NotFound();

    db.Games.Remove(game);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();
```

---

## Workflow Summary

1. **Create Entity** (C# class)
2. **Create DbContext** (define DbSet properties)
3. **Register DbContext** (in Program.cs)
4. **Create Migration** (`dotnet ef migrations add`)
5. **Apply Migration** (`dotnet ef database update`)
6. **Use DbContext** in endpoints to perform CRUD operations
7. **Call SaveChangesAsync()** to persist changes

---

# Advanced Architecture Concepts in ASP.NET Core

## Design Patterns for Production APIs

### Pattern #1: Separation of Concerns

Each layer should have a single responsibility:

```
Controller → Service → DbContext
```

**Never put:**

- Business logic in controllers
- Database logic in DTOs
- Configuration inside methods

### Pattern #2: Service Layer Pattern

**Controllers should only:**

- Receive HTTP requests
- Validate input
- Return responses

**Business logic belongs in Services.**

#### ❌ Bad Practice

```csharp
app.MapPost("/games", async (CreateGameDto dto, AppDbContext db) =>
{
    if (dto.Price < 0)
        return Results.BadRequest();

    var game = new Game { /* ... */ };

    db.Games.Add(game);
    await db.SaveChangesAsync();

    return Results.Ok();
});
```

Business logic mixed with HTTP logic.

#### ✅ Good Practice

**File:** `Services/GameService.cs`

```csharp
using GameStore.Api.Data;
using GameStore.Api.Entities;

namespace GameStore.Api.Services;

public class GameService
{
    private readonly AppDbContext _db;

    public GameService(AppDbContext db)
    {
        _db = db;
    }

    public async Task<Game> CreateGameAsync(string name, decimal price)
    {
        if (price <= 0)
            throw new ArgumentException("Price must be positive");

        var game = new Game
        {
            Name = name,
            Price = price,
            CreatedAt = DateTime.UtcNow
        };

        _db.Games.Add(game);
        await _db.SaveChangesAsync();

        return game;
    }

    public async Task<List<Game>> GetAllAsync()
    {
        return await _db.Games.ToListAsync();
    }

    public async Task<Game?> GetByIdAsync(int id)
    {
        return await _db.Games.FindAsync(id);
    }
}
```

**Endpoint:**

```csharp
app.MapPost("/games", async (
    CreateGameDto dto,
    GameService service) =>
{
    var game = await service.CreateGameAsync(dto.Name, dto.Price);
    return Results.Created($"/games/{game.Id}", game);
});
```

**Benefits:** Cleaner, testable, maintainable.

---

## Dependency Injection (DI)

ASP.NET Core has a built-in DI container.

### What Is DI?

Instead of manually creating instances:

```csharp
var service = new GameService();
```

The framework creates and injects dependencies automatically.

### Why Use DI?

- Decoupling
- Testability
- Lifecycle control
- Cleaner architecture

### Registering Services

**In `Program.cs`:**

```csharp
builder.Services.AddScoped<GameService>();
```

### Injecting Services

```csharp
app.MapGet("/games", async (GameService service) =>
{
    return await service.GetAllAsync();
});
```

The framework automatically resolves and injects the service.

---

## Service Lifetimes

**There are 3 lifetimes:**

### 1. Transient

Created every time it's requested.

```csharp
builder.Services.AddTransient<MyService>();
```

**Use when:**

- Lightweight services
- Stateless operations
- No shared state

### 2. Scoped (Most Common)

Created once per HTTP request.

```csharp
builder.Services.AddScoped<GameService>();
```

**Use for:**

- DbContext
- Business services
- Request-specific operations

**Why:** Each HTTP request gets its own instance. Safe for web applications.

### 3. Singleton

Created once for the entire application lifetime.

```csharp
builder.Services.AddSingleton<MyCacheService>();
```

**Use for:**

- Caching services
- Configuration providers
- Expensive initialization objects

**Never use for:**

- DbContext
- Request-specific services

### Why DbContext Is Scoped

DbContext should be **Scoped** because:

- It tracks entity changes
- It is NOT thread-safe
- It should live only for the duration of a request

**Default registration:**

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

This automatically registers it as **Scoped**.

---

## Configuration System (appsettings.json)

Production applications never hardcode configuration values.

### Basic Configuration File

**File:** `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=gamestore.db"
  },
  "GameSettings": {
    "MaxPrice": 500,
    "MinPrice": 1
  }
}
```

### Reading Connection Strings

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Strongly Typed Configuration

**Create configuration class:**

**File:** `Configurations/GameSettings.cs`

```csharp
namespace GameStore.Api.Configurations;

public class GameSettings
{
    public int MaxPrice { get; set; }
    public int MinPrice { get; set; }
}
```

**Register in `Program.cs`:**

```csharp
builder.Services.Configure<GameSettings>(
    builder.Configuration.GetSection("GameSettings"));
```

**Inject into service:**

```csharp
using Microsoft.Extensions.Options;

namespace GameStore.Api.Services;

public class GameService
{
    private readonly AppDbContext _db;
    private readonly GameSettings _settings;

    public GameService(AppDbContext db, IOptions<GameSettings> options)
    {
        _db = db;
        _settings = options.Value;
    }

    public async Task<Game> CreateGameAsync(string name, decimal price)
    {
        if (price > _settings.MaxPrice)
            throw new ArgumentException("Price exceeds maximum allowed.");

        if (price < _settings.MinPrice)
            throw new ArgumentException("Price below minimum allowed.");

        var game = new Game
        {
            Name = name,
            Price = price,
            CreatedAt = DateTime.UtcNow
        };

        _db.Games.Add(game);
        await _db.SaveChangesAsync();

        return game;
    }
}
```

---

## Asynchronous Programming

### Why Async Matters

Web servers handle thousands of concurrent requests. Blocking threads kills performance and scalability.

**Always use async for:**

- Database calls
- External API calls
- File I/O operations

### ❌ Blocking Code

```csharp
var games = dbContext.Games.ToList();
```

### ✅ Async Version

```csharp
var games = await dbContext.Games.ToListAsync();
```

### Method Signature Rules

If a method calls async operations, it must be async:

```csharp
public async Task<List<Game>> GetAllAsync()
{
    return await _db.Games.ToListAsync();
}
```

**Return types:**

- `Task<T>` - Async method returning a value
- `Task` - Async method with no return value

### Never Use .Result or .Wait()

**Never do this:**

```csharp
var result = SomeAsyncMethod().Result;  // ❌ Can cause deadlocks
var result = SomeAsyncMethod().Wait();  // ❌ Can cause deadlocks
```

**This can cause:**

- Deadlocks
- Thread starvation
- Performance issues

**Always use `await`:**

```csharp
var result = await SomeAsyncMethod();  // ✅ Correct
```

---

## Thread Safety

Singleton services must be thread-safe.

### ❌ Bad Singleton Example

```csharp
public class CounterService
{
    public int Count { get; set; }  // Not thread-safe!
}
```

Multiple requests modifying `Count` simultaneously → race condition.

### ✅ Thread-Safe Singleton

Use immutable state or proper locking:

```csharp
public class CounterService
{
    private int _count;
    private readonly object _lock = new();

    public void Increment()
    {
        lock (_lock)
        {
            _count++;
        }
    }

    public int GetCount()
    {
        lock (_lock)
        {
            return _count;
        }
    }
}
```

---

## Production Project Structure

```
GameStore.Api/
│
├── Entities/
│   └── Game.cs
├── Dtos/
│   ├── GameDto.cs
│   └── CreateGameDto.cs
├── Services/
│   └── GameService.cs
├── Data/
│   └── AppDbContext.cs
├── Configurations/
│   └── GameSettings.cs
├── Program.cs
└── appsettings.json
```

---

## Complete Production Example

### Program.cs

```csharp
using GameStore.Api.Data;
using GameStore.Api.Services;
using GameStore.Api.Configurations;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Configure DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Register services
builder.Services.AddScoped<GameService>();

// Configure settings
builder.Services.Configure<GameSettings>(
    builder.Configuration.GetSection("GameSettings"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

// Endpoints
app.MapGet("/games", async (GameService service) =>
    await service.GetAllAsync());

app.MapGet("/games/{id}", async (int id, GameService service) =>
{
    var game = await service.GetByIdAsync(id);
    return game is null ? Results.NotFound() : Results.Ok(game);
});

app.MapPost("/games", async (CreateGameDto dto, GameService service) =>
{
    var game = await service.CreateGameAsync(dto.Name, dto.Price);
    return Results.Created($"/games/{game.Id}", game);
});

app.Run();
```

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=gamestore.db"
  },
  "GameSettings": {
    "MaxPrice": 500,
    "MinPrice": 1
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

---

## Common Mistakes to Avoid

❌ Using Singleton lifetime for DbContext
❌ Blocking async calls with `.Result` or `.Wait()`
❌ Hardcoding configuration values
❌ Putting business logic inside controllers/endpoints
❌ Ignoring service lifetimes
❌ Mixing concerns across layers

---

## Mental Model Summary

| Concept   | Purpose                           |
| --------- | --------------------------------- |
| DI        | Framework manages object creation |
| Scoped    | One instance per HTTP request     |
| Transient | New instance every time           |
| Singleton | One instance for app lifetime     |
| DbContext | Always use Scoped lifetime        |
| Async     | Never block threads               |
| Config    | Never hardcode values             |

---

## Service Lifetime Decision Guide

**Use Transient when:**

- Service is lightweight
- Service is stateless
- No shared state needed

**Use Scoped when:**

- Working with DbContext
- Service needs request-specific data
- Most business services (default choice)

**Use Singleton when:**

- Service is expensive to create
- Service maintains app-wide state
- Caching services
- Must ensure thread safety

---

# Code Organization & Integration in ASP.NET Core

## Why Code Organization Matters

When your API has 3 endpoints, everything in `Program.cs` works fine. But when it grows to 40+ endpoints with multiple services, entities, and DTOs, poor organization becomes unmaintainable.

Professional APIs must be organized for scalability.

---

## Professional Project Structure

```
GameStore.Api/
│
├── Data/
│   └── AppDbContext.cs
│
├── Entities/
│   └── Game.cs
│
├── Dtos/
│   ├── GameDto.cs
│   ├── CreateGameDto.cs
│   └── UpdateGameDto.cs
│
├── Services/
│   └── GameService.cs
│
├── Extensions/
│   ├── GameEndpoints.cs
│   └── MappingExtensions.cs
│
├── Program.cs
└── appsettings.json
```

This structure scales effectively.

---

## Route Groups (Organizing Endpoints)

### ❌ Messy Approach

All endpoints in `Program.cs`:

```csharp
app.MapGet("/games", ...);
app.MapPost("/games", ...);
app.MapPut("/games/{id}", ...);
app.MapDelete("/games/{id}", ...);
```

Program.cs becomes unreadable with growth.

### ✅ Clean Approach Using Route Groups

**Step 1: Create Extension File**

**File:** `Extensions/GameEndpoints.cs`

```csharp
using GameStore.Api.Services;
using GameStore.Api.Dtos;

namespace GameStore.Api.Extensions;

public static class GameEndpoints
{
    public static RouteGroupBuilder MapGameEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/games");

        // GET all games
        group.MapGet("/", async (GameService service) =>
        {
            var games = await service.GetAllAsync();
            return Results.Ok(games);
        });

        // GET game by id
        group.MapGet("/{id}", async (int id, GameService service) =>
        {
            var game = await service.GetByIdAsync(id);
            return game is null ? Results.NotFound() : Results.Ok(game);
        });

        // POST new game
        group.MapPost("/", async (
            CreateGameDto dto,
            GameService service) =>
        {
            var game = await service.CreateAsync(dto);
            return Results.Created($"/games/{game.Id}", game);
        });

        // PUT update game
        group.MapPut("/{id}", async (
            int id,
            UpdateGameDto dto,
            GameService service) =>
        {
            var updated = await service.UpdateAsync(id, dto);
            return updated ? Results.NoContent() : Results.NotFound();
        });

        // DELETE game
        group.MapDelete("/{id}", async (
            int id,
            GameService service) =>
        {
            var deleted = await service.DeleteAsync(id);
            return deleted ? Results.NoContent() : Results.NotFound();
        });

        return group;
    }
}
```

**Step 2: Use in Program.cs**

```csharp
app.MapGameEndpoints();
```

Now `Program.cs` stays clean and readable.

---

## Mapping Extensions (Entity ↔ DTO)

Never repeat mapping logic across endpoints and services.

### ❌ Bad Practice

```csharp
return new GameDto(game.Id, game.Name, game.Price);
```

Repeated everywhere → messy and error-prone.

### ✅ Use Mapping Extension Methods

**File:** `Extensions/MappingExtensions.cs`

```csharp
using GameStore.Api.Entities;
using GameStore.Api.Dtos;

namespace GameStore.Api.Extensions;

public static class MappingExtensions
{
    public static GameDto ToDto(this Game game)
    {
        return new GameDto(
            game.Id,
            game.Name,
            game.Price
        );
    }

    public static Game ToEntity(this CreateGameDto dto)
    {
        return new Game
        {
            Name = dto.Name,
            Price = dto.Price,
            CreatedAt = DateTime.UtcNow
        };
    }
}
```

### Using Mapping Extensions

**In Service:**

```csharp
public async Task<GameDto> CreateAsync(CreateGameDto dto)
{
    var game = dto.ToEntity();

    _db.Games.Add(game);
    await _db.SaveChangesAsync();

    return game.ToDto();
}
```

**Benefits:** Clean, reusable, maintainable.

---

## Complete Service Layer Example

**File:** `Services/GameService.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using GameStore.Api.Data;
using GameStore.Api.Dtos;
using GameStore.Api.Extensions;

namespace GameStore.Api.Services;

public class GameService
{
    private readonly AppDbContext _db;

    public GameService(AppDbContext db)
    {
        _db = db;
    }

    public async Task<List<GameDto>> GetAllAsync()
    {
        return await _db.Games
            .Select(g => g.ToDto())
            .ToListAsync();
    }

    public async Task<GameDto?> GetByIdAsync(int id)
    {
        var game = await _db.Games.FindAsync(id);
        return game?.ToDto();
    }

    public async Task<GameDto> CreateAsync(CreateGameDto dto)
    {
        var game = dto.ToEntity();

        _db.Games.Add(game);
        await _db.SaveChangesAsync();

        return game.ToDto();
    }

    public async Task<bool> UpdateAsync(int id, UpdateGameDto dto)
    {
        var game = await _db.Games.FindAsync(id);

        if (game is null)
            return false;

        game.Name = dto.Name;
        game.Price = dto.Price;

        await _db.SaveChangesAsync();
        return true;
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var game = await _db.Games.FindAsync(id);

        if (game is null)
            return false;

        _db.Games.Remove(game);
        await _db.SaveChangesAsync();
        return true;
    }
}
```

**Clean separation:**

- Endpoints = HTTP layer
- Service = Business logic
- Mapping = Extension methods
- Entity = Database model

---

## DTOs for Complete CRUD

**File:** `Dtos/GameDto.cs`

```csharp
namespace GameStore.Api.Dtos;

public record GameDto(
    int Id,
    string Name,
    decimal Price
);
```

**File:** `Dtos/CreateGameDto.cs`

```csharp
namespace GameStore.Api.Dtos;

public record CreateGameDto(
    string Name,
    decimal Price
);
```

**File:** `Dtos/UpdateGameDto.cs`

```csharp
namespace GameStore.Api.Dtos;

public record UpdateGameDto(
    string Name,
    decimal Price
);
```

---

## Frontend Integration

### Enabling CORS

Frontend applications run on different ports. Without CORS, browsers block requests.

**In `Program.cs`:**

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend",
        policy =>
        {
            policy.WithOrigins("http://localhost:5173") // React/Vite default
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});
```

**Apply CORS middleware:**

```csharp
app.UseCors("AllowFrontend");
```

**For multiple origins:**

```csharp
policy.WithOrigins(
    "http://localhost:3000",  // React
    "http://localhost:5173",  // Vite
    "http://localhost:4200"   // Angular
)
```

### How Frontend Calls Your API

**JavaScript Example:**

```javascript
// GET all games
fetch("https://localhost:7001/games")
  .then((res) => res.json())
  .then((data) => console.log(data));

// POST new game
fetch("https://localhost:7001/games", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "FIFA 24",
    price: 59.99,
  }),
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

### API Requirements for Frontend

Your API must:

- Return proper JSON
- Return correct HTTP status codes
- Use predictable route structures
- Maintain consistent DTO contracts

---

## Consistent API Contracts

Frontend depends on stable JSON structure:

```json
{
  "id": 1,
  "name": "FIFA 24",
  "price": 59.99
}
```

**Important:** Changing field names breaks frontend. This is why DTO stability matters.

---

## Error Handling for Frontend

Return consistent error format:

```csharp
return Results.BadRequest(new
{
    error = "Price must be positive",
    field = "price"
});
```

**Multiple errors:**

```csharp
return Results.BadRequest(new
{
    errors = new[]
    {
        new { field = "name", message = "Name is required" },
        new { field = "price", message = "Price must be positive" }
    }
});
```

Frontend can then display errors properly. Never return plain strings randomly.

---

## Complete Integration Flow

1. **Frontend** sends `POST /games`
2. **API** validates input
3. **Service** saves to database
4. **Mapping** converts Entity → DTO
5. **API** returns `201 Created`
6. **Frontend** updates UI

---

## Complete Program.cs Example

```csharp
using GameStore.Api.Data;
using GameStore.Api.Services;
using GameStore.Api.Extensions;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Configure database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

// Register services
builder.Services.AddScoped<GameService>();

// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend",
        policy =>
        {
            policy.WithOrigins("http://localhost:5173")
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

// Enable CORS
app.UseCors("AllowFrontend");

// Map endpoints
app.MapGameEndpoints();

app.Run();
```

---

## When to Refactor

Refactor when you notice:

- Any file exceeds 200 lines
- Duplicate mapping logic
- Business logic inside endpoints
- Hardcoded strings everywhere
- Difficulty finding code

**Refactoring improves:**

- Readability
- Testability
- Maintainability
- Team collaboration

---

## Organization Patterns Summary

| Pattern           | Purpose                    |
| ----------------- | -------------------------- |
| Route Groups      | Organize related endpoints |
| Extension Methods | Reusable mapping logic     |
| Mapping Layer     | Protect architecture       |
| Service Layer     | Business logic isolation   |
| CORS              | Enable frontend access     |
| DTOs              | Stable API contracts       |

---

## Best Practices Checklist

✅ Move endpoints into extension classes
✅ Use route groups for related endpoints
✅ Create mapping extensions for Entity ↔ DTO
✅ Keep service layer clean and focused
✅ Enable CORS for frontend integration
✅ Return structured, consistent errors
✅ Maintain stable DTO contracts
✅ Keep `Program.cs` minimal and readable
✅ Use proper HTTP status codes
✅ Never expose entities directly to API consumers

---

## Complete File Structure Example

```
GameStore.Api/
│
├── Data/
│   └── AppDbContext.cs
│
├── Entities/
│   └── Game.cs
│
├── Dtos/
│   ├── GameDto.cs
│   ├── CreateGameDto.cs
│   └── UpdateGameDto.cs
│
├── Services/
│   └── GameService.cs
│
├── Extensions/
│   ├── GameEndpoints.cs
│   └── MappingExtensions.cs
│
├── Configurations/
│   └── GameSettings.cs
│
├── Migrations/
│   └── [EF Core migrations]
│
├── Program.cs
├── appsettings.json
└── appsettings.Development.json
```

---

## Quick Reference: Adding New Entity

**1. Create Entity** (`Entities/Product.cs`)
**2. Create DTOs** (`Dtos/ProductDto.cs`, `CreateProductDto.cs`)
**3. Add DbSet** to `AppDbContext`
**4. Create Service** (`Services/ProductService.cs`)
**5. Create Mapping Extensions** (`Extensions/MappingExtensions.cs`)
**6. Create Endpoints** (`Extensions/ProductEndpoints.cs`)
**7. Register Service** in `Program.cs`
**8. Map Endpoints** in `Program.cs`
**9. Run Migration** (`dotnet ef migrations add AddProduct`)
**10. Update Database** (`dotnet ef database update`)

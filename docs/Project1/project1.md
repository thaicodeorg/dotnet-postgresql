# สร้าง Project ที่รวม Webapp และ MVC เข้าด้วยกัน
- คำสั่ง dotnet CLI ทั้งหมด สำหรับสร้าง โปรเจกต์ .NET 9 ที่รองรับทั้ง MVC + WebApi + EF Core + PostgreSQL ในรูปแบบ Clean Architecture
- รวม MVC + WebApi ให้สร้าง dotnet new web แล้ว add builder.Services.AddControllersWithViews() + builder.Services.AddControllers() เอง จะยืดหยุ่นที่สุด
- และโครงสร้าง เป็น CleanArchitechture

```bash
$ dotnet-ef

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 9.0.8

Usage: dotnet ef [options] [command]

Options:
  --version        Show version information
  -h|--help        Show help information
  -v|--verbose     Show verbose output.
  --no-color       Don't colorize output.
  --prefix-output  Prefix output with level.

Commands:
  database    Commands to manage the database.
  dbcontext   Commands to manage DbContext types.
  migrations  Commands to manage migrations.

Use "dotnet ef [command] --help" for more information about a command.

```

# เริ่มต้นสร้าง project
```bash
mkdir CleanArchDemo
cd CleanArchDemo

# สร้าง solution
dotnet new sln -n CleanArchDemo
```
---

# สร้าง Project ตามแนวทาง Clean Architecture

```bash
# Domain layer
dotnet new classlib -n CleanArchDemo.Domain -o src/Domain
dotnet sln add src/Domain/CleanArchDemo.Domain.csproj

# Application layer
dotnet new classlib -n CleanArchDemo.Application -o src/Application
dotnet sln add src/Application/CleanArchDemo.Application.csproj

# Infrastructure layer
dotnet new classlib -n CleanArchDemo.Infrastructure -o src/Infrastructure
dotnet sln add src/Infrastructure/CleanArchDemo.Infrastructure.csproj

# Web Project (รวม MVC + WebApi)
dotnet new web -n CleanArchDemo.Web -o src/Web
dotnet sln add src/Web/CleanArchDemo.Web.csproj
```

---

# ตั้งค่า Project Reference
```bash
# Application -> Domain
dotnet add src/Application/CleanArchDemo.Application.csproj reference src/Domain/CleanArchDemo.Domain.csproj

# Infrastructure -> Application + Domain
dotnet add src/Infrastructure/CleanArchDemo.Infrastructure.csproj reference src/Application/CleanArchDemo.Application.csproj

dotnet add src/Infrastructure/CleanArchDemo.Infrastructure.csproj reference src/Domain/CleanArchDemo.Domain.csproj

# Web -> Application + Infrastructure
dotnet add src/Web/CleanArchDemo.Web.csproj reference src/Application/CleanArchDemo.Application.csproj

dotnet add src/Web/CleanArchDemo.Web.csproj reference src/Infrastructure/CleanArchDemo.Infrastructure.csproj

```
---

# ติดตั้ง EF Core + PostgreSQL
```bash
# ติดตั้ง EF Core Design (ใช้สร้าง migrations)
dotnet add src/Infrastructure/CleanArchDemo.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Design

# EF Core PostgreSQL provider
dotnet add src/Infrastructure/CleanArchDemo.Infrastructure.csproj package Npgsql.EntityFrameworkCore.PostgreSQL

# EF Core runtime
dotnet add src/Infrastructure/CleanArchDemo.Infrastructure.csproj package Microsoft.EntityFrameworkCore

dotnet add src/Web/CleanArchDemo.Web.csproj package Microsoft.EntityFrameworkCore.Design


# ติดตั้ง dotnet-ef CLI tool (global)
dotnet tool install --global dotnet-ef
```

# สร้าง DbContext (Infrastructure)

```csharp
# src/Infrastructure/Persistence/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using CleanArchDemo.Domain.Entities;

namespace CleanArchDemo.Infrastructure.Persistence;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<User> Users => Set<User>();
}

```

# สร้าง Entity (Domain)
```csharp
// src/Domain/Entities/User.cs
namespace CleanArchDemo.Domain.Entities;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}
```

# ตั้งค่า Programe.cs (web+mvc+wepapi)
```csharp

using CleanArchDemo.Infrastructure.Persistence;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// PostgreSQL connection
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add services
builder.Services.AddControllers();              // WebApi
builder.Services.AddControllersWithViews();     // MVC

var app = builder.Build();

// Routing
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();        // WebApi
    endpoints.MapControllerRoute(      // MVC
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});

app.Run();

```
---
# Connection String (appstrings.json)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=cleanarchdb;Username=myuser;Password=mypassword"
  }
}
```

---
# สร้าง Migration + Update Database
```bash
# สร้าง Migration
dotnet ef migrations add InitialCreate --project src/Infrastructure --startup-project src/Web

# Update Database
dotnet ef database update --project src/Infrastructure --startup-project src/Web
```

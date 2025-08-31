# Install SDK

## Install SDK
``` bash 
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-9.0
```

## Install Runtime
```
sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-9.0
```

## Check
```
dotnet --list-sdks
dotnet --list-runtimes
```

## Dependecies on Ubuntu
```
ca-certificates
libc6
libgcc-s1
libicu74
liblttng-ust1t64
libssl3t64
libstdc++6
zlib1g
```

```
sudo apt install \
ca-certificates \
libc6 \
libgcc-s1 \
libicu74 \
liblttng-ust1t64 \
libssl3t64 \
libstdc++6 \
zlib1g
```

## ติดตั้ง Entity Framework Core Tools

```
dotnet tool install --global dotnet-ef
```
output:
```
Tools directory '/home/sysadmin/.dotnet/tools' is not currently on the PATH environment variable.
If you are using bash, you can add it to your profile by running the following command:

cat << \EOF >> ~/.bash_profile
# Add .NET Core SDK tools
export PATH="$PATH:/home/sysadmin/.dotnet/tools"
EOF

You can add it to the current session by running the following command:

export PATH="$PATH:/home/sysadmin/.dotnet/tools"

```

## ตรวจสอบ
```
dotnet ef --version

Entity Framework Core .NET Command-line Tools
9.0.8
```

## สร้าง โครงสร้าง project เพื่อให้สามารถรองรับ CleanArchitecture

เป้าหมายโครงสร้าง File
```
src/
 ├── Application       (Business logic, Use Cases)
 ├── Domain            (Entities, Core models)
 ├── Infrastructure    (EF Core, PostgreSQL, External services)
 └── WebApi            (Presentation layer, Controllers, Startup)
```

```
mkdir dotnet-postgresql
cd dotnet-postgresql

dotnet new sln -n MySolution

tree . 

.
└── MySolution.sln
```

ใน .NET solution (.sln) เป็นเพียงไฟล์ metadata สำหรับรวม projects (.csproj) ไว้ด้วยกันเท่านั้น

- ชื่อ ไฟล์ .sln ไม่จำเป็นต้องตรงกับชื่อ folder
- เวลาเปิดงานด้วย dotnet sln add หรือใน Visual Studio / Rider, มันใช้ path ที่อ้างถึง ไม่ได้ขึ้นอยู่กับชื่อไฟล์ .sln

## คำสั่งเพื่อสร้าง Project Structure
```
# Domain layer
dotnet new classlib -n MySolution.Domain -o src/Domain
dotnet sln add src/Domain/MySolution.Domain.csproj
```

**ตัวอย่างผลทางหน้าจอ**
```
(base) ➜  dotnet-postgresql dotnet new classlib -n MySolution.Domain -o src/Domain
The template "Class Library" was created successfully.

Processing post-creation actions...
Restoring /home/sysadmin/DotnetProj/dotnet-postgresql/src/Domain/MySolution.Domain.csproj:
Restore succeeded.


(base) ➜  dotnet-postgresql tree .
.
├── MySolution.sln
└── src
    └── Domain
        ├── Class1.cs
        ├── MySolution.Domain.csproj
        └── obj
            ├── MySolution.Domain.csproj.nuget.dgspec.json
            ├── MySolution.Domain.csproj.nuget.g.props
            ├── MySolution.Domain.csproj.nuget.g.targets
            ├── project.assets.json
            └── project.nuget.cache

4 directories, 8 files
(base) ➜  dotnet-postgresql dotnet sln add src/Domain/MySolution.Domain.csproj
Project `src/Domain/MySolution.Domain.csproj` added to the solution.
```


สร้าง Layer ที่เหลือ
```
dotnet new classlib -n MySolution.Application -o src/Application
dotnet sln add src/Application/MySolution.Application.csproj
```

**ตัวอย่างผลทางหน้าจอ**
```
(base) ➜  dotnet-postgresql dotnet new classlib -n MySolution.Application -o src/Application
The template "Class Library" was created successfully.

Processing post-creation actions...
Restoring /home/sysadmin/DotnetProj/dotnet-postgresql/src/Application/MySolution.Application.csproj:
Restore succeeded.


(base) ➜  dotnet-postgresql dotnet sln add src/Application/MySolution.Application.csproj    
Project `src/Application/MySolution.Application.csproj` added to the solution.

```

```
.
├── MySolution.sln
└── src
    ├── Application
    │   ├── Class1.cs
    │   ├── MySolution.Application.csproj
    │   └── obj
    │       ├── MySolution.Application.csproj.nuget.dgspec.json
    │       ├── MySolution.Application.csproj.nuget.g.props
    │       ├── MySolution.Application.csproj.nuget.g.targets
    │       ├── project.assets.json
    │       └── project.nuget.cache
    └── Domain
        ├── Class1.cs
        ├── MySolution.Domain.csproj
        └── obj
            ├── MySolution.Domain.csproj.nuget.dgspec.json
            ├── MySolution.Domain.csproj.nuget.g.props
            ├── MySolution.Domain.csproj.nuget.g.targets
            ├── project.assets.json
            └── project.nuget.cache

6 directories, 15 files
```

## Layer Infrastructure และ WebApi
```
# Infrastructure layer
dotnet new classlib -n MySolution.Infrastructure -o src/Infrastructure
dotnet sln add src/Infrastructure/MySolution.Infrastructure.csproj

# WebApi layer
dotnet new webapi -n MySolution.WebApi -o src/WebApi
dotnet sln add src/WebApi/MySolution.WebApi.csproj
```

```
.
├── MySolution.sln
└── src
    ├── Application
    │   ├── Class1.cs
    │   ├── MySolution.Application.csproj
    │   └── obj
    │       ├── MySolution.Application.csproj.nuget.dgspec.json
    │       ├── MySolution.Application.csproj.nuget.g.props
    │       ├── MySolution.Application.csproj.nuget.g.targets
    │       ├── project.assets.json
    │       └── project.nuget.cache
    ├── Domain
    │   ├── Class1.cs
    │   ├── MySolution.Domain.csproj
    │   └── obj
    │       ├── MySolution.Domain.csproj.nuget.dgspec.json
    │       ├── MySolution.Domain.csproj.nuget.g.props
    │       ├── MySolution.Domain.csproj.nuget.g.targets
    │       ├── project.assets.json
    │       └── project.nuget.cache
    ├── Infrastructure
    │   ├── Class1.cs
    │   ├── MySolution.Infrastructure.csproj
    │   └── obj
    │       ├── MySolution.Infrastructure.csproj.nuget.dgspec.json
    │       ├── MySolution.Infrastructure.csproj.nuget.g.props
    │       ├── MySolution.Infrastructure.csproj.nuget.g.targets
    │       ├── project.assets.json
    │       └── project.nuget.cache
    └── WebApi
        ├── appsettings.Development.json
        ├── appsettings.json
        ├── MySolution.WebApi.csproj
        ├── MySolution.WebApi.http
        ├── obj
        │   ├── MySolution.WebApi.csproj.nuget.dgspec.json
        │   ├── MySolution.WebApi.csproj.nuget.g.props
        │   ├── MySolution.WebApi.csproj.nuget.g.targets
        │   ├── project.assets.json
        │   └── project.nuget.cache
        ├── Program.cs
        └── Properties
            └── launchSettings.json

11 directories, 33 files
```

**สรุป** 
- การสร้าง Layer และ Reference
```bash
# Domain Layer
dotnet new classlib -n MySolution.Domain -o src/Domain
dotnet sln add src/Domain/MySolution.Domain.csproj

# Application Layer
dotnet new classlib -n MySolution.Application -o src/Application
dotnet sln add src/Application/MySolution.Application.csproj


# Infrastructure layer
dotnet new classlib -n MySolution.Infrastructure -o src/Infrastructure
dotnet sln add src/Infrastructure/MySolution.Infrastructure.csproj

# WebApi layer
dotnet new webapi -n MySolution.WebApi -o src/WebApi
dotnet sln add src/WebApi/MySolution.WebApi.csproj
```

## ติดตั้ง EF Core + PostgreSQL Provider
```bash
# ที่ Infrastructure layer (เพราะ Infrastructure จัดการ DB)
dotnet add src/Infrastructure/MySolution.Infrastructure.csproj package Microsoft.EntityFrameworkCore
dotnet add src/Infrastructure/MySolution.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Design
dotnet add src/Infrastructure/MySolution.Infrastructure.csproj package Npgsql.EntityFrameworkCore.PostgreSQL

# ติดตั้ง ef tools (ถ้ายังไม่ได้ทำ)
dotnet tool install --global dotnet-ef
```

## เพิ่ม DbContext (Infrastructure)
```c++
// src/Infrastructure/Persistence/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using MySolution.Domain.Entities;

namespace MySolution.Infrastructure.Persistence;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<User> Users => Set<User>();
}
```

##  สร้าง Entity (Domain)
```c++
// src/Domain/Entities/User.cs
namespace MySolution.Domain.Entities;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}
```

## ตั้งค่า EF ใน WebApi (Program.cs)
```c++
using MySolution.Infrastructure.Persistence;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// PostgreSQL Connection
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

## ใ่ส่ Connection String
```c++
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=cleanarchdb;Username=myuser;Password=mypassword"
  }
}
```

## สร้าง Migration + update Database
```bash
dotnet ef migrations add InitialCreate --project src/Infrastructure --startup-project src/WebApi
dotnet ef database update --project src/Infrastructure --startup-project src/WebApi
```
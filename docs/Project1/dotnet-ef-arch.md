# โครงสร้างการทำงานของ dotnet-ef

คำถามคือ dotnet-ef CLI tools และ Microsoft.EntityFrameworkCore.Design package สองตัวนี้ทำงานสัมพันธ์กันอย่างไรเวลาใช้งาน Migrations
---
# 1 บทบาทของแต่ละตัว 
## dotnet-ef (CLI Tools)
- เป็น คำสั่งบรรทัดคำสั่ง (command line interface)
- ช่วยรันคำสั่งพิเศษ เช่น
    - dotnet ef migrations add
    - dotnet ef database update
    - dotnet ef migrations script
- เป็น ตัวเรียกใช้งาน ไม่ได้มี EF runtime code อยู่ในตัว

## Microsoft.EntityFrameworkCore.Design (NuGet Package)
- เป็น library ที่ใช้ใน runtime ของ EF
- มี service และ API ที่ dotnet-ef ต้องการใช้เบื้องหลัง เช่น
    - การ generate code สำหรับ Migrations
    - การอ่านค่า DbContext จากโปรเจกต์
    - การสร้าง SQL script
---
## 2. ทำไมต้องใช้คู่กัน

- ถ้ามีแค่ dotnet-ef → คุณจะรันคำสั่งได้ แต่จะ ล้มเหลว เพราะไม่มีโค้ดฝั่ง Design ที่ช่วยแปลงข้อมูล (จะ error ว่า no design-time services found)

- ถ้ามีแค่ Microsoft.EntityFrameworkCore.Design → โปรเจกต์มีโค้ดพร้อม แต่คุณ ไม่มีคำสั่ง สำหรับสร้าง migration หรือ update database

ดังนั้นต้อง ใช้คู่กัน:

- dotnet-ef = เครื่องมือสั่งงาน (command)
- EFCore.Design = เบื้องหลังที่ทำงานจริง

---
## 3. การทำงานร่วมกัน ระหว่าง  dotnet-ef  และ  Microsoft.EntityFrameworkCore.Design

```
dotnet ef migrations add InitialCreate
```

ลำดับเหตุการณ์:

- dotnet-ef (CLI tool) → เรียกคำสั่ง

- Tool จะโหลดโปรเจกต์ Startup → หา DbContext

- Tool จะเรียกใช้ service จาก Microsoft.EntityFrameworkCore.Design → เพื่อสร้างไฟล์ Migration (.cs)

- Migration ถูก generate และ compile เข้าโปรเจกต์

```
[ คุณพิมพ์คำสั่ง CLI ]
       |
       v
+-------------------+
|   dotnet-ef CLI   |
| (global tool)     |
+-------------------+
       |
       | โหลดโปรเจกต์ Startup (.csproj)
       v
+-----------------------------+
| Microsoft.EntityFramework   |
| .Core.Design (NuGet pkg)    |
|   - Design-time services    |
|   - Code generation         |
|   - SQL script generator    |
+-----------------------------+
       |
       | ใช้ Reflection หา DbContext
       v
+-----------------------------+
|   Application/Infrastructure|
|   - AppDbContext.cs         |
|   - Entities (Domain)       |
+-----------------------------+
       |
       | เขียนผลลัพธ์
       v
+-----------------------------+
|  Migrations folder          |
|  (C# Migration files)       |
+-----------------------------+
```

| ตัว                    | หน้าที่                                                                 |
|------------------------|------------------------------------------------------------------------|
| dotnet-ef CLI           | ตัวเรียกใช้คำสั่ง เช่น `migrations add`, `database update`           |
| EFCore.Design           | ให้บริการเบื้องหลัง (design-time) เช่น generate code, สร้าง SQL      |
| DbContext (ในโปรเจกต์) | ต้นแบบโครงสร้าง DB ที่ EF ใช้เป็น reference                            |
| Migrations              | โค้ดที่ EF สร้างขึ้น เพื่อนำไป apply กับ Database                      |


```
dotnet ef migrations add InitialCreate \
  --project src/Infrastructure \
  --startup-project src/WebApi
```

- dotnet-ef → อ่าน WebApi (Startup Project)
- โหลด EF Design services จาก Infrastructure
- หาคลาส AppDbContext
- Generate ไฟล์ Migrations/20250831123456_InitialCreate.cs
- พร้อมใช้ dotnet ef database update เพื่อสร้างตารางใน PostgreSQL
- ``--project`` → บอกว่า migrations อยู่ที่ project ไหน (ส่วนใหญ่คือ Infrastructure)
- ``--startup-project`` → project ที่มี Program.cs (ส่วนใหญ่คือ WebApi)

---
## 4. เบื้องหลังการทำงาน

```bash
dotnet ef migrations add InitialCreate
```

- 1. dotnet-ef (CLI Tool) ทำงานก่อน
    - ตัวนี้เปรียบเหมือน รีโมทคอนโทรล
    - เวลาคุณพิมพ์คำสั่ง มันจะพยายามเรียกใช้โค้ดในโปรเจกต์เพื่อหาว่า DbContext อยู่ตรงไหน
    - แต่ตัวมันเองไม่มีสมองเรื่อง EF Core (มันแค่รับคำสั่ง)

2. Microsoft.EntityFrameworkCore.Design (Package ในโปรเจกต์)
    - ตัวนี้คือ สมอง ที่อยู่ในโปรเจกต์
    - ให้บริการ design-time เช่น
        - การหา DbContext
        - การแปลง Entity → Migration Code
        - การสร้าง SQL script
    - ถ้าไม่มี package นี้ dotnet-ef จะเจอ error ว่า "No design-time services were found"

3. โหลด DbContext
    - จาก Program.cs หรือจาก constructor
    - ตัวอย่างเช่น AppDbContext : DbContext
    - dotnet-ef + EFCore.Design จะทำงานร่วมกันเพื่อ อ่าน model ของคุณ
4. Generate Migration
- EFCore.Design จะ generate ไฟล์ Migration ออกมา เช่น

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Users",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                          .Annotation("Npgsql:ValueGenerationStrategy", 
                                       NpgsqlValueGenerationStrategy.IdentityByDefaultColumn),
                Name = table.Column<string>(nullable: true)
            },
            constraints: table => table.PrimaryKey("PK_Users", x => x.Id));
    }
}

```

## 5. update Database
```
dotnet ef database update
```
- dotnet-ef → ส่งคำสั่ง
- EFCore.Design → แปลง migration เป็น SQL
- Database (เช่น PostgreSQL) → ได้ตาราง Users ถูกสร้างขึ้นจริง


**สรุปเปรียบเทียบแบบง่าย**

- dotnet-ef = รีโมทคอนโทรล (กดปุ่ม)  
- EFCore.Design = สมองที่ตีความว่า “ปุ่มนี้ต้องสร้างตาราง”  
- DbContext = แบบพิมพ์เขียว (blueprint) ของฐานข้อมูล  
- Database = สิ่งก่อสร้างจริง

# Workflow

```bash
คุณสั่ง CLI:
dotnet ef migrations add InitialCreate
        |
        v
+-------------------------+
| dotnet-ef CLI Tool      |
| - รับคำสั่งจากคุณ      |
| - หา Startup project   |
+-------------------------+
        |
        v
+-------------------------+
| EFCore.Design Package   |
| - Design-time service   |
| - อ่าน DbContext        |
| - Generate Migration    |
+-------------------------+
        |
        v
+-------------------------+
| DbContext & Entities    |
| - AppDbContext.cs       |
| - Entity class User     |
| - Blueprint ของ DB      |
+-------------------------+
        |
        v
+-------------------------+
| Migration Files         |
| - InitialCreate.cs      |
| - Record Up()/Down()    |
+-------------------------+
        |
        v
คุณสั่ง CLI ต่อ:
dotnet ef database update
        |
        v
+-------------------------+
| dotnet-ef CLI Tool      |
| - ส่งคำสั่ง apply       |
+-------------------------+
        |
        v
+-------------------------+
| EFCore.Design Package   |
| - แปลง Migration → SQL  |
+-------------------------+
        |
        v
+-------------------------+
| PostgreSQL Database     |
| - สร้างตาราง Users      |
| - สร้าง __EFMigrationsHistory |
+-------------------------+

```

**สรุปง่าย ๆ**

- คุณสั่งคำสั่ง CLI → dotnet-ef ทำงาน  
- Design Package → แปลง blueprint ของ DbContext → generate Migration  
- Migration Files → เก็บเป็น C# ไฟล์   
- database update → dotnet-ef ส่งคำสั่ง  
- Design Package แปลง SQL → PostgreSQL สร้างตารางจริง
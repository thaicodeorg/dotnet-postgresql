# Script Generate project

```bash
cat <<EOF > create_cleanarch_dotnet.sh
#!/bin/bash
set -e

# 1️⃣ ตั้งชื่อ solution / project
SOLUTION_NAME="CleanArchDemo"
DOMAIN_PROJECT="CleanArchDemo.Domain"
APPLICATION_PROJECT="CleanArchDemo.Application"
INFRA_PROJECT="CleanArchDemo.Infrastructure"
WEB_PROJECT="CleanArchDemo.Web"

# 2️⃣ สร้างโฟลเดอร์และ solution
mkdir -p \$SOLUTION_NAME/src
cd \$SOLUTION_NAME
dotnet new sln -n \$SOLUTION_NAME

# 3️⃣ สร้าง Projects
echo "Creating projects..."
dotnet new classlib -n \$DOMAIN_PROJECT -o src/Domain
dotnet new classlib -n \$APPLICATION_PROJECT -o src/Application
dotnet new classlib -n \$INFRA_PROJECT -o src/Infrastructure
dotnet new web -n \$WEB_PROJECT -o src/Web

# 4️⃣ Add projects to solution
echo "Adding projects to solution..."
dotnet sln add src/Domain/\$DOMAIN_PROJECT.csproj
dotnet sln add src/Application/\$APPLICATION_PROJECT.csproj
dotnet sln add src/Infrastructure/\$INFRA_PROJECT.csproj
dotnet sln add src/Web/\$WEB_PROJECT.csproj

# 5️⃣ Set Project References
echo "Setting project references..."
dotnet add src/Application/\$APPLICATION_PROJECT.csproj reference src/Domain/\$DOMAIN_PROJECT.csproj
dotnet add src/Infrastructure/\$INFRA_PROJECT.csproj reference src/Application/\$APPLICATION_PROJECT.csproj
dotnet add src/Infrastructure/\$INFRA_PROJECT.csproj reference src/Domain/\$DOMAIN_PROJECT.csproj
dotnet add src/Web/\$WEB_PROJECT.csproj reference src/Application/\$APPLICATION_PROJECT.csproj
dotnet add src/Web/\$WEB_PROJECT.csproj reference src/Infrastructure/\$INFRA_PROJECT.csproj

# 6️⃣ Install EF Core + PostgreSQL packages
echo "Installing EF Core packages..."
dotnet add src/Infrastructure/\$INFRA_PROJECT.csproj package Microsoft.EntityFrameworkCore
dotnet add src/Infrastructure/\$INFRA_PROJECT.csproj package Microsoft.EntityFrameworkCore.Design
dotnet add src/Infrastructure/\$INFRA_PROJECT.csproj package Npgsql.EntityFrameworkCore.PostgreSQL

# 7️⃣ Install dotnet-ef global tool if not exist
if ! command -v dotnet-ef &> /dev/null
then
    echo "Installing dotnet-ef CLI tool..."
    dotnet tool install --global dotnet-ef
else
    echo "dotnet-ef CLI already installed"
fi

echo "🎉 Clean Architecture project structure created successfully!"
echo "Next: Add DbContext, Entities, ConnectionString, then run migrations.
EOF
```
# ⚙️ Setup Inicial - Criar Solução e Projetos

## 🚀 Criar Solution do Zero

### 1. Criar Solution

```powershell
# Navegar para backend
cd src/backend

# Criar solution
dotnet new sln --name <SolutionName>

# Resultado: <SolutionName>.sln criado
```

### 2. Criar Projetos Class Library

```powershell
# 1. Domain (Núcleo - sem dependências externas)
dotnet new classlib --name <SolutionName>.Domain --framework net10.0
cd <SolutionName>.Domain
# Remover Class1.cs gerado automaticamente
rm Class1.cs
cd ..

# 2. Application (Orquestração)
dotnet new classlib --name <SolutionName>.Application --framework net10.0
cd <SolutionName>.Application
rm Class1.cs
cd ..

# 3. Infrastructure (Detalhes técnicos)
dotnet new classlib --name <SolutionName>.Infrastructure --framework net10.0
cd <SolutionName>.Infrastructure
rm Class1.cs
cd ..
```

### 3. Criar Projeto API

```powershell
# 4. API (Apresentação)
dotnet new webapi --name <SolutionName>.API --framework net10.0 --skip-openapi
cd <SolutionName>.API
cd ..
```

### 4. Adicionar Projetos à Solution

```powershell
# De dentro de src/backend/
dotnet sln add <SolutionName>.Domain/<SolutionName>.Domain.csproj
dotnet sln add <SolutionName>.Application/<SolutionName>.Application.csproj
dotnet sln add <SolutionName>.Infrastructure/<SolutionName>.Infrastructure.csproj
dotnet sln add <SolutionName>.API/<SolutionName>.API.csproj
```

### 5. Adicionar Referências entre Projetos

```powershell
# Application referencia Domain
dotnet add <SolutionName>.Application/<SolutionName>.Application.csproj \
    reference <SolutionName>.Domain/<SolutionName>.Domain.csproj

# Infrastructure referencia Domain e Application
dotnet add <SolutionName>.Infrastructure/<SolutionName>.Infrastructure.csproj \
    reference <SolutionName>.Domain/<SolutionName>.Domain.csproj
dotnet add <SolutionName>.Infrastructure/<SolutionName>.Infrastructure.csproj \
    reference <SolutionName>.Application/<SolutionName>.Application.csproj

# API referencia Application e Infrastructure
dotnet add <SolutionName>.API/<SolutionName>.API.csproj \
    reference <SolutionName>.Application/<SolutionName>.Application.csproj
dotnet add <SolutionName>.API/<SolutionName>.API.csproj \
    reference <SolutionName>.Infrastructure/<SolutionName>.Infrastructure.csproj
```

---

## 📦 Instalar NuGet Packages

### Domain (Sem dependências externas - apenas se necessário)

```powershell
# Nenhum pacote obrigatório - manter puro
```

### Application

```powershell
cd <SolutionName>.Application

# FluentValidation para validadores
dotnet add package FluentValidation --version 11.11.0

# Logging
dotnet add package Microsoft.Extensions.Logging.Abstractions --version 8.0.0

cd ..
```

### Infrastructure

```powershell
cd <SolutionName>.Infrastructure

# Entity Framework Core
dotnet add package Microsoft.EntityFrameworkCore --version 8.2.0
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 8.2.0
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.2.0

# Npgsql (PostgreSQL)
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0

# Logging
dotnet add package Microsoft.Extensions.Logging --version 8.0.0

cd ..
```

### API

```powershell
cd <SolutionName>.API

# ASP.NET Core Web API (já incluído em webapi template)

# Autenticação JWT
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 8.0.0
dotnet add package System.IdentityModel.Tokens.Jwt --version 8.0.1

# Logging
dotnet add package Microsoft.Extensions.Logging --version 8.0.0
dotnet add package Serilog.AspNetCore --version 8.1.1

cd ..
```

### Tests

```powershell
# Criar pasta para testes
mkdir tests

cd tests

# Criar projeto de testes
dotnet new xunit --name <SolutionName>.UnitTests --framework net10.0

cd <SolutionName>.UnitTests

# Test Tools
dotnet add package FluentAssertions --version 6.12.2
dotnet add package Moq --version 4.20.70
dotnet add package NSubstitute --version 5.1.0

# Referências
dotnet add package <SolutionName>.Domain --project ../../../src/backend/<SolutionName>.Domain/<SolutionName>.Domain.csproj
dotnet add package <SolutionName>.Application --project ../../../src/backend/<SolutionName>.Application/<SolutionName>.Application.csproj

cd ../..
```

---

## 📁 Criar Estrutura de Pastas

### Domain

```powershell
cd <SolutionName>.Domain

# Criar pastas
mkdir Common Entities ValueObjects Enums Interfaces

# Criar arquivos base
echo 'global using System;
global using System.Collections.Generic;
global using System.Linq;' > GlobalUsings.cs

cd ..
```

### Application

```powershell
cd <SolutionName>.Application

# Criar pastas
mkdir Common
mkdir Common\Results
mkdir Common\Pagination
mkdir Common\Security
mkdir Common\Helpers
mkdir UseCases\Usuario
mkdir UseCases\Usuario\CriarUsuario
mkdir UseCases\Usuario\AtualizarUsuario
mkdir UseCases\Usuario\ObterUsuarioPorId
mkdir UseCases\Usuario\ObterUsuarioPaginado
mkdir UseCases\Usuario\ExcluirUsuario
mkdir UseCases\Autenticacao
mkdir UseCases\Autenticacao\AutenticarUsuario
mkdir Extensions

echo 'global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using FluentValidation;
global using Microsoft.Extensions.Logging;' > GlobalUsings.cs

cd ..
```

### Infrastructure

```powershell
cd <SolutionName>.Infrastructure

# Criar pastas
mkdir Data
mkdir Data\Configurations
mkdir Data\Migrations
mkdir Repositories
mkdir Security
mkdir Extensions

echo 'global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using Microsoft.EntityFrameworkCore;
global using Microsoft.EntityFrameworkCore.Metadata.Builders;
global using Microsoft.Extensions.Logging;' > GlobalUsings.cs

cd ..
```

### API

```powershell
cd <SolutionName>.API

# Remover Controllers padrão gerado
rm -r Controllers

# Criar pastas
mkdir Controllers
mkdir Services
mkdir Middleware
mkdir Extensions

echo 'global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using Microsoft.AspNetCore.Mvc;
global using Microsoft.Extensions.Logging;' > GlobalUsings.cs

cd ..
```

---

## 🔧 Arquivo .csproj Padrão

### Domain.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

</Project>
```

### Application.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="FluentValidation" Version="11.11.0" />
        <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="8.0.0" />
    </ItemGroup>

    <ItemGroup>
        <ProjectReference Include="..\<SolutionName>.Domain\<SolutionName>.Domain.csproj" />
    </ItemGroup>

</Project>
```

### Infrastructure.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.2.0" />
        <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.2.0">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.2.0">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.0.0" />
        <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
    </ItemGroup>

    <ItemGroup>
        <ProjectReference Include="..\<SolutionName>.Domain\<SolutionName>.Domain.csproj" />
        <ProjectReference Include="..\<SolutionName>.Application\<SolutionName>.Application.csproj" />
    </ItemGroup>

</Project>
```

### API.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.0" />
        <PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="8.0.1" />
        <PackageReference Include="Serilog.AspNetCore" Version="8.1.1" />
    </ItemGroup>

    <ItemGroup>
        <ProjectReference Include="..\<SolutionName>.Application\<SolutionName>.Application.csproj" />
        <ProjectReference Include="..\<SolutionName>.Infrastructure\<SolutionName>.Infrastructure.csproj" />
    </ItemGroup>

</Project>
```

---

## ✅ Build & Restore

```powershell
# Na raiz do backend (src/backend/)

# Restaurar packages
dotnet restore

# Build
dotnet build -c Debug

# Se houver erros, verificar:
# 1. Arquivo .csproj
# 2. Versões de packages
# 3. Framework target
```

---

## ✅ Checklist de Setup

- [ ] Solution criada (<SolutionName>.sln)
- [ ] 4 projetos criados (Domain, Application, Infrastructure, API)
- [ ] Referências entre projetos corretas (API → Infra → App → Domain)
- [ ] NuGet packages instalados
- [ ] Pastas criadas para cada camada
- [ ] GlobalUsings.cs em cada projeto
- [ ] Build executado com sucesso
- [ ] Nenhum erro no Solution
- [ ] README.md atualizado com instruções

---

**Próxima Seção**: Ver [`08-legacy-migration.md`](./08-legacy-migration.md) para padrões legados.

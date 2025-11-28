# ⚙️ Setup Inicial - Criar Solução e Projetos

## 🚀 Criar Solution do Zero

### 1. Criar Solution

```powershell
# Navegar para backend
cd src/backend

# Criar solution
dotnet new sln --name AgenteViagem

# Resultado: AgenteViagem.sln criado
```

### 2. Criar Projetos Class Library

```powershell
# 1. Domain (Núcleo - sem dependências externas)
dotnet new classlib --name AgenteViagem.Domain --framework net10.0
cd AgenteViagem.Domain
# Remover Class1.cs gerado automaticamente
rm Class1.cs
cd ..

# 2. Application (Orquestração)
dotnet new classlib --name AgenteViagem.Application --framework net10.0
cd AgenteViagem.Application
rm Class1.cs
cd ..

# 3. Infrastructure (Detalhes técnicos)
dotnet new classlib --name AgenteViagem.Infrastructure --framework net10.0
cd AgenteViagem.Infrastructure
rm Class1.cs
cd ..
```

### 3. Criar Projeto API

```powershell
# 4. API (Apresentação)
dotnet new webapi --name AgenteViagem.API --framework net10.0 --skip-openapi
cd AgenteViagem.API
cd ..
```

### 4. Adicionar Projetos à Solution

```powershell
# De dentro de src/backend/
dotnet sln add AgenteViagem.Domain/AgenteViagem.Domain.csproj
dotnet sln add AgenteViagem.Application/AgenteViagem.Application.csproj
dotnet sln add AgenteViagem.Infrastructure/AgenteViagem.Infrastructure.csproj
dotnet sln add AgenteViagem.API/AgenteViagem.API.csproj
```

### 5. Adicionar Referências entre Projetos

```powershell
# Application referencia Domain
dotnet add AgenteViagem.Application/AgenteViagem.Application.csproj \
    reference AgenteViagem.Domain/AgenteViagem.Domain.csproj

# Infrastructure referencia Domain e Application
dotnet add AgenteViagem.Infrastructure/AgenteViagem.Infrastructure.csproj \
    reference AgenteViagem.Domain/AgenteViagem.Domain.csproj
dotnet add AgenteViagem.Infrastructure/AgenteViagem.Infrastructure.csproj \
    reference AgenteViagem.Application/AgenteViagem.Application.csproj

# API referencia Application e Infrastructure
dotnet add AgenteViagem.API/AgenteViagem.API.csproj \
    reference AgenteViagem.Application/AgenteViagem.Application.csproj
dotnet add AgenteViagem.API/AgenteViagem.API.csproj \
    reference AgenteViagem.Infrastructure/AgenteViagem.Infrastructure.csproj
```

---

## 📦 Instalar NuGet Packages

### Domain (Sem dependências externas - apenas se necessário)

```powershell
# Nenhum pacote obrigatório - manter puro
```

### Application

```powershell
cd AgenteViagem.Application

# FluentValidation para validadores
dotnet add package FluentValidation --version 11.11.0

# Logging
dotnet add package Microsoft.Extensions.Logging.Abstractions --version 8.0.0

cd ..
```

### Infrastructure

```powershell
cd AgenteViagem.Infrastructure

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
cd AgenteViagem.API

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
dotnet new xunit --name AgenteViagem.UnitTests --framework net10.0

cd AgenteViagem.UnitTests

# Test Tools
dotnet add package FluentAssertions --version 6.12.2
dotnet add package Moq --version 4.20.70
dotnet add package NSubstitute --version 5.1.0

# Referências
dotnet add package AgenteViagem.Domain --project ../../../src/backend/AgenteViagem.Domain/AgenteViagem.Domain.csproj
dotnet add package AgenteViagem.Application --project ../../../src/backend/AgenteViagem.Application/AgenteViagem.Application.csproj

cd ../..
```

---

## 📁 Criar Estrutura de Pastas

### Domain

```powershell
cd AgenteViagem.Domain

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
cd AgenteViagem.Application

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
cd AgenteViagem.Infrastructure

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
cd AgenteViagem.API

# Remover Controllers padrão gerado
rm -r Controllers

# Criar pastas
mkdir Controllers
mkdir Controllers\v1
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
        <ProjectReference Include="..\AgenteViagem.Domain\AgenteViagem.Domain.csproj" />
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
        <ProjectReference Include="..\AgenteViagem.Domain\AgenteViagem.Domain.csproj" />
        <ProjectReference Include="..\AgenteViagem.Application\AgenteViagem.Application.csproj" />
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
        <ProjectReference Include="..\AgenteViagem.Application\AgenteViagem.Application.csproj" />
        <ProjectReference Include="..\AgenteViagem.Infrastructure\AgenteViagem.Infrastructure.csproj" />
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

- [ ] Solution criada (AgenteViagem.sln)
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

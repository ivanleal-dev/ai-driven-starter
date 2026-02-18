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

> 📋 **TargetFramework & Versões**: [09-stack.md](./09-stack.md)

```powershell
# 1. Domain (Núcleo - sem dependências externas)
dotnet new classlib --name <SolutionName>.Domain --framework {{TargetFramework}}
cd <SolutionName>.Domain
# Remover Class1.cs gerado automaticamente
rm Class1.cs
cd ..

# 2. Application (Orquestração)
dotnet new classlib --name <SolutionName>.Application --framework {{TargetFramework}}
cd <SolutionName>.Application
rm Class1.cs
cd ..

# 3. Infrastructure (Detalhes técnicos)
dotnet new classlib --name <SolutionName>.Infrastructure --framework {{TargetFramework}}
cd <SolutionName>.Infrastructure
rm Class1.cs
cd ..
```

### 3. Criar Projeto API

```powershell
# 4. API (Apresentação)
dotnet new webapi --name <SolutionName>.API --framework {{TargetFramework}} --skip-openapi
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

> 📋 **Fonte Única de Verdade**: [../../09-stack.md](../../09-stack.md)
> 
> ⚠️ **Importante**: Use SEMPRE as versões específicas definidas no arquivo de requisitos técnicos

### Quick Install (seguir instruções detalhadas no arquivo de requisitos)

```powershell
# Domain: SEM DEPENDÊNCIAS (manter puro)

# Consulte 09-stack.md para comandos de instalação específicos
# Todas as versões e comandos estão definidos no arquivo de requisitos técnicos
```

> 📋 Para **lista completa, versões atualizadas e configurações**, consulte: [../../09-stack.md](../../09-stack.md)

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
 
> 📋 **TargetFramework & Versões**: [../../09-stack.md](../../09-stack.md)

### Domain.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>{{TargetFramework}}</TargetFramework>
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
        <TargetFramework>{{TargetFramework}}</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <!-- Ver versões específicas em 09-stack.md -->
    <ItemGroup>
        <PackageReference Include="FluentValidation" Version="{{Version}}" />
        <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="{{Version}}" />
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
        <TargetFramework>{{TargetFramework}}</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <!-- Ver versões específicas em 09-stack.md -->
    <ItemGroup>
        <PackageReference Include="Microsoft.EntityFrameworkCore" Version="{{Version}}" />
        <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="{{Version}}">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="{{Version}}">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="{{Version}}" />
        <PackageReference Include="Microsoft.Extensions.Logging" Version="{{Version}}" />
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
        <TargetFramework>{{TargetFramework}}</TargetFramework>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <!-- Ver versões específicas em 09-stack.md -->
    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="{{Version}}" />
        <PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="{{Version}}" />
        <PackageReference Include="Serilog.AspNetCore" Version="{{Version}}" />
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

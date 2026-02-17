# 📋 Backend Requirements - Technical Specifications

> ⚠️ **Fonte Única de Verdade** para todas as definições técnicas do backend

---

## 🎯 Stack Tecnológico

### Runtime & Framework
| Tecnologia | Versão | Status |
|------------|--------|--------|
| **.NET** | `10.0` | ✅ Ativo |
| **C#** | `12` | ✅ Ativo |
| **ASP.NET Core** | `8.0` | ✅ Web API |

### Banco de Dados
| Tecnologia | Versão | Provider |
|------------|--------|----------|
| **PostgreSQL** | `16+` | Npgsql `8.0.0` |
| **Entity Framework Core** | `8.2.0` | ORM Principal |

---

## 📦 Dependências NuGet - Versões Oficiais

### 🏛️ **Domain Layer**
```powershell
# ✅ SEM DEPENDÊNCIAS EXTERNAS - Manter camada pura
# Apenas referências do .NET base
```

### ⚙️ **Application Layer**
| Package | Versão | Uso |
|---------|--------|-----|
| `FluentValidation` | `11.11.0` | Validadores de requests |
| `Microsoft.Extensions.Logging.Abstractions` | `8.0.0` | Interface de logging |

```powershell
dotnet add package FluentValidation --version 11.11.0
dotnet add package Microsoft.Extensions.Logging.Abstractions --version 8.0.0
```

### 🗄️ **Infrastructure Layer**
| Package | Versão | Uso |
|---------|--------|-----|
| `Microsoft.EntityFrameworkCore` | `8.2.0` | ORM core |
| `Microsoft.EntityFrameworkCore.Tools` | `8.2.0` | CLI tools (migrations) |
| `Microsoft.EntityFrameworkCore.Design` | `8.2.0` | Design-time services |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | `8.0.0` | Provider PostgreSQL |
| `Microsoft.Extensions.Logging` | `8.0.0` | Logging implementation |

```powershell
dotnet add package Microsoft.EntityFrameworkCore --version 8.2.0
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 8.2.0
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.2.0
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0
dotnet add package Microsoft.Extensions.Logging --version 8.0.0
```

### 🌐 **API Layer**
| Package | Versão | Uso |
|---------|--------|-----|
| `Microsoft.AspNetCore.Authentication.JwtBearer` | `8.0.0` | Auth JWT |
| `System.IdentityModel.Tokens.Jwt` | `8.0.1` | JWT tokens |
| `Microsoft.Extensions.Logging` | `8.0.0` | Logging |
| `Serilog.AspNetCore` | `8.1.1` | Structured logging |

```powershell
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 8.0.0
dotnet add package System.IdentityModel.Tokens.Jwt --version 8.0.1
dotnet add package Microsoft.Extensions.Logging --version 8.0.0
dotnet add package Serilog.AspNetCore --version 8.1.1
```

### 🧪 **Test Projects**
| Package | Versão | Uso |
|---------|--------|-----|
| `xunit` | `2.9.0` | Test framework |
| `FluentAssertions` | `6.12.2` | Assertions expressivas |
| `NSubstitute` | `5.1.0` | Mocking (preferido sobre Moq) |
| `Microsoft.EntityFrameworkCore.InMemory` | `8.2.0` | InMemory DB para testes |

```powershell
dotnet new xunit --framework net10.0
dotnet add package FluentAssertions --version 6.12.2
dotnet add package NSubstitute --version 5.1.0
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 8.2.0
```

---

## 🏗️ Configurações de Projeto

### Target Framework
```xml
<TargetFramework>net10.0</TargetFramework>
<LangVersion>12</LangVersion>
<Nullable>enable</Nullable>
<ImplicitUsings>enable</ImplicitUsings>
```

### Global Usings
```csharp
// Adicionar em cada projeto conforme necessidade
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
```

---

## 🗄️ Configuração de Banco

### Connection String (Development)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "User ID=postgres;Password=postgres;Server=localhost;Port=5432;Database=ai_driven_starter;Integrated Security=false;"
  }
}
```

### DbContext Registration
```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"))
);
```

### Convenções de Nomenclatura
- **Tabelas**: `usuarios`, `pedidos` (português, snake_case)
- **Colunas**: `data_criacao`, `nome_completo` (português, snake_case)
- **Índices**: `ix_{tabela}_{coluna}`
- **PKs**: `pk_{tabela}`
- **FKs**: `fk_{tabela_origem}_{tabela_destino}`

---

## 🔒 Autenticação & Autorização

### JWT Configuration
```json
{
  "JwtSettings": {
    "SecretKey": "sua-chave-secreta-muito-forte-aqui",
    "Issuer": "ai-driven-starter",
    "Audience": "ai-driven-starter-clients",
    "ExpiryInHours": 24
  }
}
```

### Security Headers
```csharp
// Program.cs - Configurações obrigatórias
app.UseAuthentication();
app.UseAuthorization();
app.UseHsts(); // Production
app.UseHttpsRedirection();
```

---

## 🧪 Configuração de Testes

### Structure
```
tests/
├── <SolutionName>.UnitTests/           # Domain + Application tests
├── <SolutionName>.IntegrationTests/   # API + Infrastructure tests
└── <SolutionName>.ArchitectureTests/  # Architecture compliance
```

### Testing Standards
- **Unit Tests**: Domain + Application layers
- **Integration Tests**: Repository + Controller
- **Coverage mínimo**: 80%+ logic branches
- **Naming**: `MetodoTestado_Cenario_ResultadoEsperado`

---

## 🚀 Deployment & Environment

### Environments Suportados
| Environment | Database | Logging Level |
|-------------|----------|---------------|
| **Development** | PostgreSQL local | Debug |
| **Staging** | PostgreSQL Azure | Information |
| **Production** | PostgreSQL Azure | Warning |

### Docker Requirements
```dockerfile
# Minimal runtime image
FROM mcr.microsoft.com/dotnet/aspnet:10.0-alpine
# Development image  
FROM mcr.microsoft.com/dotnet/sdk:10.0-alpine
```

---

## 📋 Compliance & Standards

### Code Quality
- **Analyzer**: Microsoft.CodeAnalysis.NetAnalyzers
- **EditorConfig**: Enabled
- **Nullable**: Required
- **Warnings as Errors**: Production builds

### Performance Standards
- **Response Time**: < 200ms (95th percentile)
- **Memory**: < 100MB working set
- **CPU**: < 50% under normal load

---

## 📚 Referências Externas

| Documentação | Link |
|--------------|------|
| **.NET 10 Release** | [docs.microsoft.com/dotnet/core/whats-new](https://docs.microsoft.com/dotnet/core/whats-new) |
| **EF Core 8.x** | [docs.microsoft.com/ef/core](https://docs.microsoft.com/ef/core) |
| **ASP.NET Core 8** | [docs.microsoft.com/aspnet/core](https://docs.microsoft.com/aspnet/core) |
| **PostgreSQL 16** | [postgresql.org/docs/16](https://postgresql.org/docs/16) |
| **FluentValidation** | [docs.fluentvalidation.net](https://docs.fluentvalidation.net) |

---

## ⚠️ Importante

- **Este arquivo é a fonte única de verdade** para todas as versões e configurações
- **Qualquer alteração** deve ser feita AQUI primeiro, depois propagada
- **Consulte este arquivo** antes de instalar qualquer dependência
- **Valide conformidade** após qualquer setup de projeto

---

*Última atualização: 11 de Fevereiro, 2026*
*Versão: 1.0.0*
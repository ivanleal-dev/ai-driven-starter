# 📋 Backend Requirements - Technical Specifications

> ⚠️ **Fonte Única de Verdade** para todas as definições técnicas do backend

---

## 🎯 Stack Tecnológico

### Runtime & Framework
| Tecnologia | Versão | Status |
|------------|--------|--------|
| **.NET** | `11.0` | ✅ Ativo |
| **C#** | `13` | ✅ Ativo |
| **ASP.NET Core** | `11.0` | ✅ Web API |

### Banco de Dados
| Tecnologia | Versão | Provider |
|------------|--------|----------|
| **PostgreSQL** | `17+` | Npgsql `8.1.0` |
| **Entity Framework Core** | `11.0.0` | ORM Principal |

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
| `FluentValidation` | `11.12.0` | Validadores de requests |
| `Microsoft.Extensions.Logging.Abstractions` | `11.0.0` | Interface de logging |

```powershell
dotnet add package FluentValidation --version 11.12.0
dotnet add package Microsoft.Extensions.Logging.Abstractions --version 11.0.0
```

### 🗄️ **Infrastructure Layer**
| Package | Versão | Uso |
|---------|--------|-----|
| `Microsoft.EntityFrameworkCore` | `11.0.0` | ORM core |
| `Microsoft.EntityFrameworkCore.Tools` | `11.0.0` | CLI tools (migrations) |
| `Microsoft.EntityFrameworkCore.Design` | `11.0.0` | Design-time services |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | `8.1.0` | Provider PostgreSQL |
| `Microsoft.Extensions.Logging` | `11.0.0` | Logging implementation |

```powershell
dotnet add package Microsoft.EntityFrameworkCore --version 11.0.0
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 11.0.0
dotnet add package Microsoft.EntityFrameworkCore.Design --version 11.0.0
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.1.0
dotnet add package Microsoft.Extensions.Logging --version 11.0.0
```

### 🌐 **API Layer**
| Package | Versão | Uso |
|---------|--------|-----|
| `Microsoft.AspNetCore.Authentication.JwtBearer` | `11.0.0` | Auth JWT |
| `System.IdentityModel.Tokens.Jwt` | `8.1.0` | JWT tokens |
| `Microsoft.Extensions.Logging` | `11.0.0` | Logging |
| `Serilog.AspNetCore` | `9.0.0` | Structured logging |

```powershell
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 11.0.0
dotnet add package System.IdentityModel.Tokens.Jwt --version 8.1.0
dotnet add package Microsoft.Extensions.Logging --version 11.0.0
dotnet add package Serilog.AspNetCore --version 9.0.0
```

### 🧪 **Test Projects**
| Package | Versão | Uso |
|---------|--------|-----|
| `xunit` | `2.10.0` | Test framework |
| `FluentAssertions` | `7.0.0` | Assertions expressivas |
| `NSubstitute` | `5.2.0` | Mocking (preferido sobre Moq) |
| `Microsoft.EntityFrameworkCore.InMemory` | `11.0.0` | InMemory DB para testes |

```powershell
dotnet new xunit --framework net11.0
dotnet add package FluentAssertions --version 7.0.0
dotnet add package NSubstitute --version 5.2.0
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 11.0.0
```

---

## 🏗️ Configurações de Projeto

### Target Framework
```xml
<TargetFramework>net11.0</TargetFramework>
<LangVersion>13</LangVersion>
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
FROM mcr.microsoft.com/dotnet/aspnet:11.0-alpine
# Development image  
FROM mcr.microsoft.com/dotnet/sdk:11.0-alpine
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
| **.NET 11 Release** | [docs.microsoft.com/dotnet/core/whats-new](https://docs.microsoft.com/dotnet/core/whats-new) |
| **EF Core 11.x** | [docs.microsoft.com/ef/core](https://docs.microsoft.com/ef/core) |
| **ASP.NET Core 11** | [docs.microsoft.com/aspnet/core](https://docs.microsoft.com/aspnet/core) |
| **PostgreSQL 17** | [postgresql.org/docs/17](https://postgresql.org/docs/17) |
| **FluentValidation** | [docs.fluentvalidation.net](https://docs.fluentvalidation.net) |

---

## ⚠️ Importante

- **Este arquivo é a fonte única de verdade** para todas as versões e configurações
- **Qualquer alteração** deve ser feita AQUI primeiro, depois propagada
- **Consulte este arquivo** antes de instalar qualquer dependência
- **Valide conformidade** após qualquer setup de projeto

---

*Última atualização: 20 de Fevereiro, 2026*
*Versão: 1.1.0*
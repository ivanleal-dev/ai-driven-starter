---
applyTo: 'src/backend/**'
---

# Backend - Instruções Gerais & Índice

## 📚 Documentação Backend Organizada

**Leia nesta ordem:**

### 1. **Fundamentos da Arquitetura** 
📖 Ver: `instructions/01-architecture.md`
- Clean Architecture principles
- 4 camadas (Domain → Application → Infrastructure → API)
- SOLID principles
- Dependency Inversion

### 2. **Templates de Código Reutilizáveis**
📖 Ver: `instructions/02-patterns.md`
- Result Pattern (sucesso/falha)
- Entity Pattern (factory methods, invariantes)
- Repository Pattern
- Handler Pattern (orquestração)
- Validator Pattern
- Controller Pattern
- Value Object Pattern

### 3. **Nomenclatura & Convenções**
📖 Ver: `instructions/03-conventions.md`
- Tudo em **PORTUGUÊS**: Handlers, Entities, Requests, Responses
- C# 12 & .NET 10 features
- PascalCase (tipos), camelCase (variáveis)
- Global Usings

### 4. **Estrutura de Pastas**
📖 Ver: `instructions/04-folder-structure.md`
- Organização por camada
- Vertical slices em Application
- Navegação entre camadas

### 5. **Entity Framework Core & Banco de Dados**
📖 Ver: `instructions/05-database.md`
- Setup DbContext
- EF Configurations (português)
- Migrations
- Repositories e UnitOfWork
- Paginação

### 6. **Testes Unitários (Domain + Application)**
📖 Ver: `instructions/06-testing.md`
- xunit setup
- Domain entity tests
- Handler tests
- Validators
- Mocking patterns

### 7. **Setup Inicial (Criar Solução)**
📖 Ver: `instructions/07-setup.md`
- Criar solution + 4 projetos
- Instalar packages
- Configurar pastas

### 8. **Migração de Padrões Legados**
📖 Ver: `instructions/08-legacy-migration.md`
- Specification → Direct queries
- Services → Handlers
- AutoMapper → Records
- Exceptions → Result Pattern
- MediatR → Direct DI

---

## 🎯 Quick Reference

### Projeto: {{ProjectBase}}

**Stack:**
- .NET 10, C# 12
- ASP.NET Core (Minimal APIs)
- Entity Framework Core + Npgsql (PostgreSQL)
- FluentValidation, xunit, NSubstitute

**Camadas:**
```
Domain             (lógica pura, sem dependências)
    ↓
Application        (handlers, validators, DTOs)
    ↓
Infrastructure     (EF Core, Repositories)
    ↓
API                (Controllers, HTTP routing)
```

**Padrões Obrigatórios:**
- ✅ Result<T> para retorno de operações
- ✅ Handlers com DI direto (SEM MediatR)
- ✅ Testes: Domain + Application (SEM Infra/API/E2E)
- ✅ Nomes em português
- ✅ Entity Configurations com nomes BD em português

---

## 📋 Implementação de Nova Feature (Checklist)

### 1. **Planejar com PRD**
```
Ler: /docs/prd/PRD-XXXX/USYYY/PRD-*.md
Identificar: entidades, campos, regras, endpoints
```

### 2. **Implementar (Ordem Rigorosa)**
- [ ] **Domain**: Entities (factory methods), Interfaces de Repositório
- [ ] **Application**: Handlers, Validators, DTOs (Request/Response)
- [ ] **Infrastructure**: EF Config, Repositories, Migrations
- [ ] **API**: Controller injetando handlers

### 3. **Validar**
- [ ] Compilação: `dotnet build`
- [ ] Testes: `dotnet test`
- [ ] Migration: `dotnet ef migrations add ...`

### 4. **Commit**
```bash
git commit -m "feat(usXXX): implementar [funcionalidade]

- Adicionar entidade {Entity}
- Criar handlers (Criar/Atualizar/ObterPorId/ObterPaginado/Excluir)
- Endpoints CRUD em {Entity}sController
- Gerar migration

Refs: #USXXX"
```

---

## ✅ Checklist Rápido para Handler Novo

**Domain:**
- [ ] Entity com factory method `Criar(...)`
- [ ] Invariantes validadas no factory
- [ ] Métodos de comportamento (Ativar, Desativar, Atualizar)
- [ ] Interface de repositório criada

**Application:**
- [ ] Request record com `required` properties
- [ ] Validator com FluentValidation
- [ ] Handler com `HandleAsync()` retornando `Result<T>`
- [ ] Response record

**Infrastructure:**
- [ ] EF Configuration com nomes em português
- [ ] Repository implementado
- [ ] Índices criados
- [ ] Migration gerada

**API:**
- [ ] Controller injetando handlers (não services)
- [ ] Endpoints mapeados com `.Match()`
- [ ] ProblemDetails para erros

---

## 🚀 Comandos Úteis

```bash
# Build
dotnet build src/backend/{{ProjectBase}}.sln -c Debug

# Test
dotnet test tests/{{ProjectBase}}.UnitTests/

# Database Update
dotnet ef database update \
    --project src/backend/{{ProjectBase}}.Infrastructure \
    --startup-project src/backend/{{ProjectBase}}.API

# Create Migration
dotnet ef migrations add NomeMigration \
    --project src/backend/{{ProjectBase}}.Infrastructure \
    --startup-project src/backend/{{ProjectBase}}.API

# Run API
dotnet run --project src/backend/{{ProjectBase}}.API
```

---

## 🚫 Padrões Proibidos

- ❌ MediatR / Mediator (use handlers injetados)
- ❌ AutoMapper (map manualmente com records)
- ❌ Specification Pattern (queries diretas)
- ❌ Generic Service classes (use handlers verticais)
- ❌ Exceptions em fluxo normal (use Result Pattern)
- ❌ Infrastructure no Application
- ❌ Controllers com lógica de negócio
- ❌ Tests de Controllers, Infrastructure, E2E

---

## 📞 Referência de Prompts

| Tarefa | Agent | Arquivo |
|--------|-------|---------|
| User Story | @story-writer | `.github/prompts/story.writter.prompt.md` |
| PRD | @prd-writer | `.github/prompts/prd.writter.prompt.md` |
| Backend | @backend-api | `.github/prompts/backend.writter.prompt.md` |
| Tests | @unittest-writer | `.github/prompts/unittest.writter.prompt.md` |

---

## 📁 Estrutura de Documentação

```
src/backend/.github/
├── copilot-instructions.md         ← Índice (você está aqui)
└── instructions/                    ← Documentação especializada
    ├── 01-architecture.md           (camadas, SOLID, DDD)
    ├── 02-patterns.md               (templates de código)
    ├── 03-conventions.md            (nomenclatura, C# 12)
    ├── 04-folder-structure.md       (organização)
    ├── 05-database.md               (EF Core, BD)
    ├── 06-testing.md                (testes unitários)
    ├── 07-setup.md                  (criar solution)
    └── 08-legacy-migration.md       (migrar padrões)
```

---

## 💡 Quando Consultar Cada Arquivo

| Pergunta | Arquivo |
|----------|---------|
| "Como está organizada a arquitetura?" | `01-architecture.md` |
| "Qual é o template para Entity/Handler?" | `02-patterns.md` |
| "Como nomear minhas classes?" | `03-conventions.md` |
| "Onde criar novas pastas?" | `04-folder-structure.md` |
| "Como configurar EF Core?" | `05-database.md` |
| "Como escrever testes?" | `06-testing.md` |
| "Como criar um novo projeto?" | `07-setup.md` |
| "Tenho Specification Pattern legado" | `08-legacy-migration.md` |

---

**Backend Version:** ASP.NET Core .NET 10  
**Última atualização:** 2025-01-27

---
applyTo: 'src/backend/**'
---

# Backend - Instruções Gerais & Índice

## 📚 Documentação Backend Organizada

**Leia nesta ordem:**

### 0. **Regras de Segurança** ⚠️
📖 Ver: `./00-security.md`
- Migrations (confirmação obrigatória)
- Criação de solution
- Operações DDL

### 1. **Fundamentos da Arquitetura** 
📖 Ver: `./01-architecture.md`
- Clean Architecture principles
- 4 camadas (Domain → Application → Infrastructure → API)
- SOLID principles
- Dependency Inversion

### 2. **Templates de Código Reutilizáveis**
📖 Ver: `./02-patterns.md`
- Result Pattern (sucesso/falha)
- Entity Pattern (factory methods, invariantes)
- Repository Pattern
- Handler Pattern (orquestração)
- Validator Pattern
- Controller Pattern
- Value Object Pattern

### 3. **Nomenclatura & Convenções**
📖 Ver: `./03-conventions.md`
- Tudo em **PORTUGUÊS**: Handlers, Entities, Requests, Responses
- 📋 **Stack & Versões**: [../../backend-requirements.md](../../backend-requirements.md)
- PascalCase (tipos), camelCase (variáveis)
- Global Usings

### 4. **Estrutura de Pastas**
📖 Ver: `./04-folder-structure.md`
- Organização por camada
- Vertical slices em Application
- Navegação entre camadas

### 5. **Entity Framework Core & Banco de Dados**
📖 Ver: `./05-database.md`
- Setup DbContext
- EF Configurations (português)
- Migrations
- Repositories (sem UnitOfWork)
- Paginação

### 6. **Testes Unitários (Domain + Application)**
📖 Ver: `./06-testing.md`
- xunit setup
- Domain entity tests
- Handler tests
- Validators
- Mocking patterns

### 7. **Setup Inicial (Criar Solução)**
📖 Ver: `./07-setup.md`
- Criar solution + 4 projetos
- 📋 **Dependências**: [../../backend-requirements.md](../../backend-requirements.md) (versões atualizadas)
- Configurar pastas

### 8. **Migração de Padrões Legados**
📖 Ver: `./08-legacy-migration.md`
- Specification → Direct queries
- Services → Handlers
- AutoMapper → Records
- Exceptions → Result Pattern
- MediatR → Direct DI

---

## 🎯 Quick Reference

### Projeto: {{ProjectBase}}

**Stack:**
> 📋 [../../backend-requirements.md](../../backend-requirements.md)

- ASP.NET Core (Controllers)
- Entity Framework Core + PostgreSQL
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

### 1. **Planejar com Feature Spec**
```
Ler: docs/specs/SPEC-XXX-*.md
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
git commit -m "feat(spec-xxx): implementar [funcionalidade]

- Adicionar entidade {Entity}
- Criar handlers (Criar/Atualizar/ObterPorId/ObterPaginado/Excluir)
- Endpoints CRUD em {Entity}sController
- Gerar migration

Refs: #SPEC-XXX"
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
- [ ] Validação no handler com `validation.ToValidationError()`
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
dotnet build src/backend/{{ProjectBase}}.slnx -c Debug

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
- ❌ IUnitOfWork na Application (viola Clean Architecture)
- ❌ Infrastructure no Application
- ❌ Controllers com lógica de negócio
- ❌ Tests de Controllers, Infrastructure, E2E

---

## 📞 Referência de Skills

| Tarefa | Agent | Skill |
|--------|-------|-------|
| Feature Spec | @spec-writer | `.github/skills/spec-writer.skill.md` |
| Backend | @backend-api | `.github/skills/backend-api.skill.md` |
| Tests | @unittest-writer | `.github/skills/unittest-writer.skill.md` |

---

## 📁 Estrutura de Documentação

```
.github/rules/backend/
├── copilot-instructions.md         ← Índice (você está aqui)
├── 01-architecture.md              (camadas, SOLID, DDD)
├── 02-patterns.md                  (templates de código)
├── 03-conventions.md               (nomenclatura & features)
├── 04-folder-structure.md          (organização)
├── 05-database.md                  (EF Core, BD)
├── 06-testing.md                   (testes unitários)
├── 07-setup.md                     (criar solution)
└── 08-legacy-migration.md          (migrar padrões)
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

> 📋 **Stack Completo**: [../../backend-requirements.md](../../backend-requirements.md)

**Última atualização:** 2026-02-10

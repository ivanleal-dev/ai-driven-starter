---
agent: agent
---

# Prompt: Implementação Backend Guiada por product-requirements-document (PRD)

## 🎯 Objetivo

Implementar código backend completo baseado em PRD de User Stories, seguindo Clean Architecture e padrões do projeto.

> **Documentação técnica:** `/src/backend/.github/instructions/`
> - `01-architecture.md` - Clean Architecture principles
> - `02-patterns.md` - Code templates (Entity, Handler, Repository)
> - `03-conventions.md` - Naming & C# conventions
> - `04-folder-structure.md` - Folder organization
> - `05-database.md` - EF Core & database
> - `06-testing.md` - Unit tests

---

## 🔐 Pre-Requisites

### 1. User Story & PRD
- **Require**: User Story ID (ex: US001)
- **Require**: PRD path `/docs/product-requirements/PRD-XXXX/USYYY/PRD-*.md`
- **Enforce**: ONE User Story per execution

### 2. Infrastructure Instructions (READ IN ORDER)
1. `.github/copilot-instructions.md`
2. `/src/backend/.github/copilot-instructions.md`
3. Extract solution name (ex: AgenteViagem)

---

## 📋 Implementation Pipeline

### Phase 1: Analysis & Planning

1. Load product-requirements from given PRD path
2. Extract:
   - Feature objective
   - Entities and fields
   - Business rules
   - Expected endpoints
3. Generate plan (show all artifacts)
4. Confirm: "Proceed? (yes/no/adjust)"

### Phase 2: Code Generation

**Generation order (STRICT):**
1. **Domain**: Entities (factory methods), Repository interfaces
2. **Application**: Handlers (Criar/Atualizar/ObterPorId/ObterPaginado/Excluir), Validators, DTOs
3. **Infrastructure**: EF Configurations, Repositories, Migrations
4. **API**: Controllers

**Naming (PORTUGUESE MANDATORY):**
- Handlers: `Criar{Entidade}Handler`, `Atualizar{Entidade}Handler`, `ObterPorIdHandler`
- Requests: `Criar{Entidade}Request`
- Responses: `{Entidade}Response`
- Repositories: `{Entidade}Repository`
- DB tables/columns: Portuguese (ex: `usuarios`, `data_criacao`)

---

## 📝 Code Templates

**DO NOT duplicate templates here** → Reference instead:

**Entity & Repository**: `/src/backend/.github/instructions/02-patterns.md` → Entity Pattern, Repository Pattern

**Handler**: `/src/backend/.github/instructions/02-patterns.md` → Handler Pattern

**Validator**: `/src/backend/.github/instructions/02-patterns.md` → Validator Pattern

**EF Configuration**: `/src/backend/.github/instructions/05-database.md` → Entity Configurations

**Controller**: `/src/backend/.github/instructions/02-patterns.md` → Controller Pattern

---

## 🗄️ Database (PORTUGUESE NAMES)

- Tables: plural Portuguese (ex: `usuarios`, `pedidos`)
- Columns: Portuguese (ex: `data_criacao`, `tentativas_falhadas`)
- Indexes: Portuguese names (ex: `ix_usuarios_email_unique`)
- Migrations: date prefix (ex: `20250127_US002_AddAutenticacao`)

---

## 🎯 Architecture Constraints

**ALWAYS follow Clean Architecture:**
- Domain: Business rules only (NO framework dependencies)
- Application: Orchestration, Handlers, Validators (NO Infrastructure)
- Infrastructure: EF Core, Repositories (implements Domain interfaces)
- API: Controllers, HTTP routing (delegates to handlers)

**ALWAYS use Result Pattern:**
- Handlers return: `Result<T>` or `Result`
- Controllers map with: `.Match(onSuccess, onFailure)`
- NO exceptions for expected errors (validation, not found, conflict)

**NEVER use MediatR/Mediator:**
- Inject handlers directly via DI
- Controllers receive `Criar{Entity}Handler`, `Atualizar{Entity}Handler`, etc

---

## ✅ Validation Checklist

Domain:
- ✓ Entity with factory method `Criar(...)`
- ✓ Business invariants validated inside
- ✓ Behavior methods (Ativar, Desativar, Atualizar)
- ✓ Repository interface created
- ✓ NO audit/soft-delete concepts

Application:
- ✓ Vertical slices: one folder per use case
- ✓ DTOs isolated: Request, Response, Validator per use case
- ✓ Handlers return Result Pattern
- ✓ NO generic Service class
- ✓ Validation before business logic

Infrastructure:
- ✓ EF Configuration with Portuguese names (ToTable, HasColumnName)
- ✓ Repository implemented
- ✓ Migration generated

API:
- ✓ Controller injects Handlers (not Mediator)
- ✓ Result mapped to IActionResult (Match pattern)
- ✓ ProblemDetails for errors
- ✓ XML comments on endpoints

---

## 🚫 NEVER Do

1. Generate code without PRD
2. Mix multiple User Stories
3. Create endpoints not in PRD
4. Duplicate logic between Handler and Controller
5. Expose Domain entities directly
6. Use `new Entity()` outside Entity class
7. Put validation in Controller
8. Use Specification Pattern (query directly in Repository)
9. Use MediatR/Mediator
10. Generate Integration/E2E tests (only Unit tests: Domain+Application)

---

## 📊 Output Format

**Structure:**

```
✅ IMPLEMENTATION COMPLETED - US{NNN}

🏗️ DOMAIN LAYER ({N} files)
├─ Entities/{EntityName}.cs
├─ Interfaces/I{EntityName}Repository.cs
[FULL CODE]

📋 APPLICATION LAYER ({N} files)
├─ UseCases/{EntityName}/{EntityName}Response.cs
├─ UseCases/{EntityName}/Criar{EntityName}Handler.cs
[FULL CODE]

🔧 INFRASTRUCTURE LAYER ({N} files)
├─ Data/Configurations/{EntityName}Configuration.cs
├─ Repositories/{EntityName}Repository.cs
[FULL CODE]

🌐 API LAYER
├─ Controllers/v1/{EntityNames}Controller.cs
[FULL CODE]

📝 README Update
[PRD README content]

✅ VALIDATION CHECKLIST
- ✓ [All items checked]

📋 NEXT STEPS
1. Apply migration: dotnet ef database update...
2. Commit: git commit -m "feat(us{nnn}): implement..."
```

---

## 🧪 Unit Tests

**Scope:**
- Domain entities (factory, invariants)
- Application handlers (success, validation failures)

**Tools**: xunit, FluentAssertions, Moq

**Coverage target**: 80%+ Application, 100% Domain

**See**: `/src/backend/.github/instructions/06-testing.md`

**NOTE**: Tests generated by `@unittest-writer` agent (separate execution)

---

**Reference all doubts to `/src/backend/.github/instructions/` files**


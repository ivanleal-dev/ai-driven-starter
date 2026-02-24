---
name: backend-develop
description: "Implementa código backend completo baseado em Feature Specs"
---

# Skill: Backend API Implementation


Você é o **@backend-api**. Implementa código backend completo baseado em Feature Specs, seguindo Clean Architecture, Domain-Driven Design, Vertical Slices e Result Pattern.

> 📋 **Requisitos Técnicos**: Ler `.github/instructions/backend/09-stack.md`

---

## 🔧 Modos de Operação

### Modo 1: Desenvolvimento com SPEC (Feature Completa)
Quando o usuário fornecer uma **Feature Spec** (`SPEC-XXX`):
- Seguir todo o fluxo de implementação documentado
- Criar todos os artefatos (Domain → Application → Infrastructure → API)
- Gerar DTOs, Commands, Handlers conforme especificado
- Solicitar SPEC se tentar implementar feature sem ela

### Modo 2: Correção/Ajuste Direto (Sem SPEC)
Quando o usuário descrever uma **correção, ajuste ou melhoria pontual** no chat:
- ✅ Pode atuar diretamente sem exigir SPEC
- ✅ Exemplos válidos:
  - "Corrigir bug no método X"
  - "Adicionar validação no campo Y"
  - "Refatorar este trecho para melhor legibilidade"
  - "Ajustar mensagem de erro"
  - "Renomear variável/método"
  - "Adicionar log em tal ponto"
  - "Corrigir typo"
  - "Melhorar performance deste query"

### Como Identificar o Modo

| Indicador | Modo |
|-----------|------|
| Usuário menciona `SPEC-XXX` ou anexa documento de spec | **Modo 1** |
| Usuário pede nova feature/endpoint/entidade completa | **Modo 1** (solicitar SPEC) |
| Usuário descreve correção/bug/ajuste pontual | **Modo 2** |
| Usuário pede refatoração localizada | **Modo 2** |
| Usuário mostra código e pede melhoria específica | **Modo 2** |

### Regras para Modo 2 (Correções Diretas)

1. **Escopo limitado**: Apenas alterações pontuais, não features completas
2. **Mesmo padrão de código**: Seguir todas as convenções do projeto
3. **Testes**: Sugerir ajustes em testes existentes se necessário
4. **Documentação**: Atualizar XML docs se o comportamento mudar
5. **Se crescer demais**: Se a correção evoluir para algo maior, sugerir criação de SPEC

> ⚠️ **Atenção**: Mesmo no Modo 2, as regras de **Constraints** continuam válidas.  
> Se a instrução não estiver clara, **pergunte antes de agir**

---

## Inputs Esperados

O usuário fornecerá:
- **spec_id** (obrigatório): ID da Feature Spec (ex: SPEC-001)
- **task_id** (opcional): ID da task específica (ex: T-001). Se omitido, implementa todas
- **spec_path** (opcional): Caminho da spec. Default: `docs/specs/SPEC-XXX-*.md`

---

## Pre-Requisitos

### 1. Feature Spec
- Localizar spec em `docs/specs/SPEC-XXX-*.md`
- Processar **UMA** Feature Spec por execução

### 2. Leitura Obrigatória (em ordem)
1. `.github/instructions/copilot-instructions.md`
2. `.github/instructions/backend/copilot-instructions.md`
3. **`.github/instructions/backend/04-folder-structure.md`** (OBRIGATÓRIO)
4. Extrair nome da solution (ex: {{ProjectBase}})

### 3. Conformidade com Estrutura de Pastas
- ✓ UseCases organizados como: `{Entity}/{Operation}/`
- ✓ Files agrupados por operação (Request, Response, Validator, Handler)
- ✓ Infrastructure/Data/Configurations/ para configs EF
- ✓ API/Controllers/ (nomes plurais: UsuariosController)

---

## Steps

### Phase 1: Análise & Planejamento

1. Carregar Feature Spec do caminho dado
2. Extrair:
   - Objetivo (da seção User Story)
   - Entidades e campos (do diagrama ER)
   - Regras de negócio (da tabela RN)
   - Endpoints esperados (dos API Contracts)
3. Gerar plano (mostrar todos os artefatos)
4. Confirmar: "Prosseguir? (sim/não/ajustar)"

### Phase 2: Geração de Código

**ANTES de gerar qualquer código:**
1. Carregar `.github/instructions/backend/04-folder-structure.md`
2. Verificar que caminhos estão corretos
3. Criar pastas se necessário

**Ordem de geração (ESTRITA):**
1. **Domain**: `{Solution}.Domain/Entities/{Entity}.cs`, `Interfaces/I{Entity}Repository.cs`
2. **Application**: `{Solution}.Application/UseCases/{Entity}/{Operation}/`
   - Files: `{Operation}Request.cs`, `{Operation}Response.cs`, `{Operation}Validator.cs`, `{Operation}Handler.cs`
3. **Infrastructure**: 
   - `{Solution}.Infrastructure/Data/Configurations/{Entity}Configuration.cs`
   - `{Solution}.Infrastructure/Repositories/{Entity}Repository.cs`
4. **API**: `{Solution}.API/Controllers/{Entities}Controller.cs` (plural)

**Nomenclatura (PORTUGUÊS OBRIGATÓRIO):**
- Handlers: `Criar{Entidade}Handler`, `Atualizar{Entidade}Handler`, `ObterPorIdHandler`
- Requests: `Criar{Entidade}Request`
- Responses: `{Entidade}Response`
- Repositories: `{Entidade}Repository`
- Tabelas/colunas DB: Português (ex: `usuarios`, `data_criacao`)

### Phase 3: Validação
- Verificar checklist de arquitetura
- Atualizar Implementation Status na spec

---

## Templates de Código

**NÃO duplicar templates** → Referenciar:
- **Entity & Repository**: `.github/instructions/backend/02-patterns.md`
- **Handler**: `.github/instructions/backend/02-patterns.md`
- **Validator**: `.github/instructions/backend/02-patterns.md`
- **EF Configuration**: `.github/instructions/backend/05-database.md`
- **Controller**: `.github/instructions/backend/02-patterns.md`

---

## Database (NOMES EM PORTUGUÊS)

- Tabelas: plural português (ex: `usuarios`, `pedidos`)
- Colunas: português (ex: `data_criacao`, `tentativas_falhadas`)
- Indexes: nomes em português (ex: `ix_usuarios_email_unique`)
- Migrations: prefixo data (ex: `20250127_US002_AddAutenticacao`)

---

## Architecture Constraints

### SEMPRE seguir Clean Architecture:
- **Domain**: Regras de negócio apenas (SEM dependências de framework)
- **Application**: Orquestração, Handlers, Validators (SEM Infrastructure)
- **Infrastructure**: EF Core, Repositories (implementa interfaces do Domain)
- **API**: Controllers, HTTP routing (delega para handlers)

### SEMPRE usar Result Pattern:
- Handlers retornam: `Result<T>` ou `Result`
- Controllers mapeiam com: `.Match(onSuccess, onFailure)`
- SEM exceptions para erros esperados (validação, not found, conflict)

### NUNCA usar MediatR/Mediator:
- Injetar handlers diretamente via DI
- Controllers recebem `Criar{Entity}Handler`, `Atualizar{Entity}Handler`, etc.

---

## Validation Checklist

**Domain:**
- ✓ Entity com factory method `Criar(...)`
- ✓ Invariantes de negócio validadas internamente
- ✓ Métodos de comportamento (Ativar, Desativar, Atualizar)
- ✓ Interface de repository criada
- ✓ SEM conceitos de audit/soft-delete

**Application:**
- ✓ Vertical slices: uma pasta por use case
- ✓ DTOs isolados: Request, Response, Validator por use case
- ✓ Handlers retornam Result Pattern
- ✓ SEM classe Service genérica
- ✓ Validação antes de business logic

**Infrastructure:**
- ✓ EF Configuration com nomes portugueses (ToTable, HasColumnName)
- ✓ Repository implementado
- ✓ Migration gerada

**API:**
- ✓ Controller injeta Handlers (não Mediator)
- ✓ Result mapeado para IActionResult (Match pattern)
- ✓ ProblemDetails para erros
- ✓ XML comments nos endpoints

**Folder Structure:**
- ✓ Application/UseCases/{Entity}/{Operation}/
- ✓ Request, Response, Validator, Handler na mesma pasta
- ✓ Infrastructure/Data/Configurations/{Entity}Configuration.cs
- ✓ API/Controllers/{Entities}Controller.cs (plural)

---

## Output

```
✅ IMPLEMENTATION COMPLETED - SPEC-XXX

🏗️ DOMAIN LAYER (N files)
├─ Domain/Entities/{EntityName}.cs
├─ Domain/Interfaces/I{EntityName}Repository.cs

📋 APPLICATION LAYER (N files)
├─ Application/UseCases/{Entity}/{Operation}/
│   ├─ {Operation}Request.cs
│   ├─ {Operation}Response.cs
│   ├─ {Operation}Validator.cs
│   └─ {Operation}Handler.cs

🔧 INFRASTRUCTURE LAYER (N files)
├─ Infrastructure/Data/Configurations/{Entity}Configuration.cs
├─ Infrastructure/Repositories/{Entity}Repository.cs

🌐 API LAYER
├─ API/Controllers/{Entities}Controller.cs

📋 NEXT STEPS
1. Apply migration
2. Run tests: /unittest-writer
3. Commit: git commit -m "feat(spec-xxx): implement..."
```

---

## 🔒 Security Rules

Consultar `.github/instructions/backend/00-security.md` para:
- Migrations (confirmação obrigatória)
- Criação de solution (confirmação obrigatória)
- Operações DDL

---

## Constraints

- **SE NÃO ENTENDEU, NÃO MEXA** — preferir inação a mudanças incorretas
- **NUNCA** implementar ou alterar código quando a instrução não estiver 100% clara
- **NUNCA** assumir intenção — se houver dúvida, pergunte antes de agir
- **NUNCA** gerar código sem Feature Spec
- **NUNCA** misturar múltiplas specs
- **NUNCA** criar endpoints fora da spec
- **NUNCA** duplicar lógica entre Handler e Controller
- **NUNCA** expor entidades Domain diretamente
- **NUNCA** usar `new Entity()` fora da classe Entity
- **NUNCA** colocar validação no Controller
- **NUNCA** usar Specification Pattern
- **NUNCA** usar MediatR/Mediator
- **NUNCA** gerar testes de Integração/E2E

---

## Example

**Input:**
```
/backend-api implemente SPEC-001
```

**Output:**
```
📂 Solution detectada: TodoApp

Analisando SPEC-001-cadastro-vendedor...

📋 Plano de implementação:
- T-001: Domain (Entity Vendedor + IVendedorRepository)
- T-002: Application (CriarVendedorHandler + Validator)
- T-003: Application (ObterVendedorPorIdHandler)
- T-007: Infrastructure (EF Config + Migration)
- T-008: API (VendedoresController)

Prosseguir? (sim/não)
```

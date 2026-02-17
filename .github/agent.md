# GitHub Copilot Custom Agents

> 📋 **Requisições Técnicas**: [09-stack.md](09-stack.md)

## Fluxo de Desenvolvimento Padrão

### Fluxo Unificado (Recomendado)

```mermaid
flowchart LR
    A[@spec-writer] --> B[@backend-api]
    B --> C[@unittest-writer]
    C --> D[Frontend - Futuro]
```

**Sequência recomendada:**
1. `@spec-writer`: Criar Feature Spec unificada (salva em `docs/specs/`)
2. `@backend-api`: Implementar backend (Domain → Application → Infrastructure → API)
3. `@unittest-writer`: Criar testes unitários (Domain + Application)
4. Frontend (aguardando definição)

---

## Agents Principais

### Agent: @spec-writer

### Description
Especialista em criar Feature Specs unificadas — documentos que combinam perspectiva de negócio (User Story) e detalhamento técnico (PRD) em um único artefato completo e rastreável.

### Expertise
- Escrita de User Stories (Como/Quero/Para)
- Critérios de aceitação testáveis (Dado/Quando/Então)
- Mapeamento de regras de negócio
- API Contracts (Endpoints, DTOs, HTTP codes)
- Diagramas Mermaid (ER, Sequence, Flow)
- Implementation Status tracking
- **Geração de Implementation Tasks com dependências ordenadas**
- Matriz de rastreabilidade
- Validações e exceções

### Instructions
Ao criar Feature Specs:
1. Processar UMA funcionalidade por vez
2. Gerar documento unificado com seções de negócio + técnica
3. Usar formato Como/Quero/Para para a User Story
4. Definir critérios de aceitação testáveis (Dado/Quando/Então)
5. Mapear regras de negócio específicas
6. Gerar diagramas Mermaid (BD, fluxos, integrações)
7. Definir API Contracts completos (Endpoints + DTOs)
8. Criar tabela Implementation Status
9. **Gerar Implementation Tasks com dependências (Domain → App → Infra → API → FE → Tests)**
10. Nunca inventar requisitos não mencionados
11. Salvar em `docs/specs/SPEC-XXX-slug.md`
12. Sugerir `@backend-api implemente T-001 da SPEC-XXX` após criação

### Context
```markdown
# Feature Spec: [Título]
SpecID: SPEC-XXX
BackendStatus: planned|partial|complete
FrontendStatus: planned|partial|complete

## 📖 Visão de Negócio
### User Story
**Como** [usuário], **Quero** [objetivo], **Para** [benefício].

## ✅ Critérios de Aceitação
- [ ] **CA-001**: Dado [contexto], Quando [ação], Então [resultado]

## 🔧 Especificação Técnica
### API Contracts
| EndpointID | Método | Path | RequestDTO | ResponseDTO |
```

### Reference
Ver skill completa em: `.github/skills/spec-writer/SKILL.md`

### Examples
```
User: @spec-writer preciso permitir cadastro de vendedores
Agent: Vou criar uma Feature Spec unificada com visão de negócio e técnica...
[Gera spec completa e salva em docs/specs/]

Próxima ação sugerida: @backend-api implemente SPEC-001
```

---

## Agent: @backend-api

### Description
Especialista em implementação backend completa seguindo Clean Architecture, Domain-Driven Design, Vertical Slices e Result Pattern. Implementa funcionalidades baseadas em Feature Specs.

### Expertise
- Clean Architecture (Domain → Application → Infrastructure → API)
- Vertical Slices / Handlers pattern (UseCases isolados)
- Domain-Driven Design (Entities, Value Objects, Aggregates)
- Result Pattern (sem exceptions para fluxo de negócio)
- Entity Framework Core (nomes de BD em português)
- FluentValidation por caso de uso
- Controllers REST com handlers injetados diretamente
- Migrations e configurações EF
- **SEM Mediator/MediatR** (injeção direta de handlers)

### Instructions
Ao implementar backend baseado em Feature Spec:
1. 📋 **Consultar SEMPRE**: [09-stack.md](09-stack.md) para versões e dependências
2. Seguir Clean Architecture: Domain → Application (UseCases/Handlers) → Infrastructure → API
2. **Vertical Slices**: cada caso de uso em pasta própria (Create, Update, GetById, etc)
3. **Handlers** isolados por operação (sem Services centrais)
4. **Result Pattern**: retornar Result<T> (sem exceptions para fluxo)
5. **Factory Methods** em Entities (Criar/Create) validando invariantes
6. **FluentValidation** por Request (não validadores genéricos)
7. Controllers injetam handlers diretamente (**SEM Mediator/MediatR**)
8. Nomes de tabelas/colunas em português no BD
9. Retornar DTOs (Response), nunca entidades
10. Migrations nomeadas: `YYYYMMDD_SPEC_XXX_NomeEntidade`
11. Soft delete somente quando a SPEC exigir (não é padrão global)
12. Seguir estrutura definida na Feature Spec

### Reference
Ver skill completa em: `.github/skills/backend-develop/SKILL.md`

### Examples
```
User: @backend-api implemente SPEC-001
Agent: Vou analisar a Feature Spec e implementar seguindo Clean Architecture...
[Gera Domain (Entity, Repository), Application (Handlers, Validators), 
 Infrastructure (Config EF, Migration), API (Controller)]

Próxima ação sugerida: @unittest-writer SPEC-001
```

---

## Agent: @unittest-writer

### Description
Especialista em testes unitários para backend (Domain e Application). Cobre Entities, Value Objects, Validators e Handlers. **Não gera testes de integração, API ou E2E.**

### Expertise
- xUnit para .NET
- FluentAssertions para assertions expressivas
- NSubstitute para mocking (preferido sobre Moq)
- Cobertura de invariantes de domínio
- Testes de Result Pattern
- Testes de validadores FluentValidation

### Instructions
Ao criar testes unitários:
1. Cobrir APENAS Domain (Entities, ValueObjects) e Application (Handlers, Validators)
2. Seguir padrão AAA (Arrange, Act, Assert)
3. Nomes descritivos: `Metodo_Estado_Resultado`
4. Mockar dependências com NSubstitute
5. Testar invariantes de domínio (factory methods)
6. Testar fluxos de sucesso E falha em Handlers
7. Validar Result Pattern (IsSuccess, IsFailure, Error codes)
8. NÃO criar testes de Infra, API ou integração

### Reference
Ver skill completa em: `.github/skills/backend-develop/SKILL.md`

### Examples
```
User: @unittest-writer SPEC-001
Agent: Vou gerar testes unitários para Domain e Application da SPEC-001...
[Gera testes de Entities, Validators e Handlers]
```

---

## Agents Futuros (Roadmap)

| Agent | Área | Prioridade |
|-------|------|------------|
| `@react-component` | Frontend | 🔜 Alta |
| `@api-client` | Frontend | 🔜 Alta |
| `@security` | Backend | 📋 Média |
| `@performance` | Full-stack | 📋 Média |
| `@docker` | DevOps | 📋 Baixa |
| `@refactor` | Full-stack | 📋 Baixa |

> Para contribuir com novos agents, siga o template em "Criando Seus Próprios Agents".

---

## Como Usar os Agents

### No Chat do Copilot
```bash
# Invocar um agent específico
@backend-api implemente SPEC-XXX
@react-component crie um data grid com ordenação
@test-writer gere testes para o ProductService

# Combinar agents
@entity-designer e @db-migration crie o modelo de faturamento

# Agent com contexto de arquivo
@refactor #file:UserController.cs melhore este código
```

### Comandos Globais
```bash
# Listar agents disponíveis
@help

# Ver detalhes de um agent
@backend-api help

# Usar agent com workspace
@backend-api @workspace analise a arquitetura atual
```

### Fluxo Completo de Exemplo
```bash
# 1. Criar Feature Spec (combina User Story + PRD)
@spec-writer preciso cadastrar vendedores com CPF
# Salva: docs/specs/SPEC-001-cadastro-vendedor.md

# 2. Implementar Backend
@backend-api implemente SPEC-001
# Gera: Domain, Application (Handlers), Infrastructure, API, Migration

# 3. Criar Testes Unitários
@unittest-writer SPEC-001
# Gera: tests/<Solution>.UnitTests/Domain/ e /Application/
```

## Criando Seus Próprios Agents

Para criar um novo agent, adicione uma seção seguindo este template:

```markdown
## Agent: @nome-do-agent

### Description
Breve descrição do que o agent faz

### Expertise
- Lista de especialidades
- Tecnologias que domina
- Padrões que conhece

### Instructions
Regras específicas que o agent deve seguir

### Context
Código de exemplo ou padrões a seguir

### Examples
Exemplos de uso do agent
```

## Melhores Práticas

1. **Agents focados**: Cada agent deve ter uma responsabilidade clara
2. **Instruções específicas**: Quanto mais específico, melhor o output
3. **Exemplos práticos**: Incluir exemplos reais do seu projeto
4. **Contexto relevante**: Adicionar snippets de código como referência
5. **Atualização contínua**: Manter agents atualizados com novas práticas

## Integração com Instructions.md

Os agents trabalham em conjunto com o `instructions.md`:
- `instructions.md`: Define regras gerais do projeto
- `agent.md`: Define especialistas para tarefas específicas

Quando você invoca um agent, ele usa tanto as instruções gerais quanto suas instruções específicas para gerar código.
# 🤖 AI-Driven Starter - Desenvolvimento Orientado por IA

Um monorepo de produção com backend **ASP.NET Core** e frontend **React + TypeScript**, desenvolvido usando **GitHub Copilot com arquivos de instruções especializados**.

## 📖 Índice

- [🎯 Visão Geral](#-visão-geral)
- [🔄 Fluxo de Desenvolvimento com IA](#-fluxo-de-desenvolvimento-com-ia)
- [📚 Arquivos de Instruções](#-arquivos-de-instruções)
- [🚀 Agentes Especializados](#-agentes-especializados)
- [📋 Exemplo Prático](#-exemplo-prático)
- [🏗️ Estrutura do Projeto](#-estrutura-do-projeto)
- [📖 Stack Tecnológico](#-stack-tecnológico)

---

## 🎯 Visão Geral

Este projeto implementa um **sistema de desenvolvimento orientado por IA** onde GitHub Copilot atua como um desenvolvedor full-stack, recebendo instruções estruturadas via:

- **Arquivos de instruções** no repositório (`.github/` e subpastas)
- **User Stories** documentadas em Markdown
- **PRDs técnicos** detalhados
- **Prompts especializados** para cada tipo de tarefa

O objetivo é **automatizar 80%+ do desenvolvimento** mantendo qualidade, padrões arquiteturais e documentação.

### 🔒 Regras de Segurança

Para garantir segurança e controle, o sistema implementa regras rígidas:

- ⚠️ **Migrations**: NUNCA executar comandos `dotnet ef` sem confirmação explícita do usuário
- ⚠️ **Solution**: SEMPRE perguntar o nome antes de criar projetos backend
- ⚠️ **Banco de Dados**: Qualquer operação DDL (ALTER, DROP, CREATE TABLE) requer aprovação prévia
- ⚠️ **API Versionamento**: Não incluir versionamento de API neste escopo
- ✅ **Exceção**: Gerar apenas código (.cs) sem executar comandos de infra

---

## 🔄 Fluxo de Desenvolvimento com IA

O desenvolvimento segue um pipeline estruturado em **4 fases principais**:

```
┌─────────────────────────────────────────────────────────────┐
│  FASE 1: DEFINIÇÃO (Story Writer)                          │
│  ├─ Criar User Story em linguagem de negócio               │
│  ├─ Definir critérios de aceitação                         │
│  └─ Salvar em: docs/stories/US{XXX}-nome.md                │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│  FASE 2: ESPECIFICAÇÃO TÉCNICA (PRD Writer)                │
│  ├─ Gerar PRD baseado na User Story                        │
│  ├─ Definir API Contracts (endpoints)                      │
│  ├─ Diagramas e fluxos de dados                            │
│  └─ Salvar em: docs/product-requirements/PRD-XXXX/USYYY/   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│  FASE 3: IMPLEMENTAÇÃO BACKEND (Backend Writer)            │
│  ├─ Domain Layer (entidades, invariantes)                  │
│  ├─ Application Layer (handlers, validadores)              │
│  ├─ Infrastructure Layer (repositórios, EF)                │
│  ├─ API Layer (controllers)                                │
│  └─ EF Core Migrations                                     │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│  FASE 4: TESTES & DOCUMENTAÇÃO (Unit Test Writer)          │
│  ├─ Domain layer tests                                     │
│  ├─ Application layer tests                                │
│  └─ Coverage validation                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Arquivos de Instruções

O projeto utiliza **hierarquia de arquivos de instruções** que guiam o Copilot:

### 🌍 Instruções Globais
```
.github/copilot-instructions.md
```
**Escopo**: Todas as operações no monorepo

**Contém**:
- Glossário de termos (PRD Bloco, US, PRD, RF, EP, DTO, FRONT)
- Fluxo de desenvolvimento padrão
- Convenções gerais (commits, branches)
- Regras de segurança (migrations, solution naming)
- Referência de agentes disponíveis

**Exemplo**:
```markdown
- **PRD (Bloco de Conhecimento Integrado)**: Agrupamento lógico de User Stories relacionadas (ex: PRD-0001-gestao-vendedores)
- **US (User Story)**: Item de backlog que descreve funcionalidade do ponto de vista do usuário (ex: US001)
- **PRD (Product Requirements Document)**: Documento técnico detalhado de uma User Story
- **RF (Requisito Funcional)**: Funcionalidade específica a ser implementada (mapeado no PRD)
- **EP (Endpoint)**: Rota da API RESTful (ex: EP-001: POST /api/vendedores)
- **DTO (Data Transfer Object)**: Objeto de transferência de dados entre camadas (Request/Response)
- **FRONT**: Item de implementação frontend (mapeado no PRD)
```

### 🔙 Instruções Backend
```
src/backend/.github/copilot-instructions.md
```
**Escopo**: Desenvolvimento backend ASP.NET Core

**Referencia**:
```
src/backend/.github/instructions/
├── 01-architecture.md       (Clean Architecture principles)
├── 02-patterns.md           (Templates de código: Entity, Handler, Validator, BaseApiController)
├── 03-conventions.md        (Nomenclatura em português, C# 12)
├── 04-folder-structure.md   (Organização de pastas)
├── 05-database.md           (EF Core, migrations com aprovação obrigatória)
├── 06-testing.md            (Unit tests com xunit)
├── 07-setup.md              (Criar solution com <SolutionName> dinâmico)
└── 08-legacy-migration.md   (Migrar padrões legados)
```

**Nota**: Todos os templates usam `<SolutionName>` como placeholder para nomes de projeto dinâmicos.

### 🎨 Instruções Frontend
```
src/frontend/.github/copilot-instructions.md
```
**Escopo**: Desenvolvimento React + TypeScript

**Contém**: Padrões React, estrutura de componentes, testes

### 📝 Prompts Especializados
```
.github/prompts/
├── story.writer.prompt.md      (@story-writer)
├── prd.writer.prompt.md        (@prd-writer)
├── backend.writer.prompt.md    (@backend-writer)
└── unittest.writer.prompt.md   (@unittest-writer)
```

---

## 🚀 Agentes Especializados

Cada agente é um "especialista" com seu próprio prompt estruturado:

### 1️⃣ **@story-writer** - Criador de User Stories

**Arquivo**: `.github/prompts/story.writer.prompt.md`

**Responsabilidade**: Escrever User Stories bem estruturadas

**Entrada**: Descrição de funcionalidade em linguagem natural

**Saída**: User Story markdown com:
- Narrativa (Como X, quero Y, para Z)
- Critérios de aceitação
- Estimativa de esforço
- Riscos e dependências

**Exemplo de uso**:
```
@story-writer: Preciso de uma funcionalidade de cadastro de usuário com validação de email
```

**Resultado**:
```markdown
# US001 - Cadastro de Usuário

## Narrativa
Como um novo usuário,
Quero me registrar na aplicação,
Para acessar features autenticadas.

## Critérios de Aceitação
- [ ] Email válido (RFC 5322)
- [ ] Senha entre 8-25 caracteres
- [ ] Confirmação de senha obrigatória
- [ ] Email único no sistema
- [ ] Retornar 201 Created

## Estimativa: 1-2 dias
```

---

### 2️⃣ **@prd-writer** - Especificador Técnico

**Arquivo**: `.github/prompts/prd.writer.prompt.md`

**Responsabilidade**: Gerar PRD técnico detalhado

**Entrada**: User Story aprovada

**Saída**: PRD com:
- API Contracts (endpoints em tabela)
- Sequência de implementação (banco → validação → backend → frontend)
- DTOs (Request/Response)
- Diagrama ER (Mermaid)
- Fluxos (Mermaid sequence diagram)
- Checklist de implementação

**Exemplo de uso**:
```
@prd-writer: Gere PRD técnico para a US001 (Cadastro de Usuário)
Base: docs/stories/US001-cadastro-usuario.md
```

**Resultado parcial**:
```markdown
## API Contracts

| EndpointID | Método | Path | Auth | RequestDTO | ResponseDTO | HTTP Codes |
|------------|--------|------|------|------------|-------------|------------|
| EP-001 | POST | /api/v1/usuarios | none | `CriarUsuarioRequest` | `UsuarioResponse` | 201,400,409,500 |

## Sequência de Implementação
### Task 1: Base de Dados
- Criar tabela `usuarios` com colunas (Id, Email, SenhaHash, DataCriacao, Ativo)
- Índice único em Email

### Task 2: Validação
- Email: NotEmpty, EmailAddress, MaxLength(320)
- Senha: Length(8,25)
- ConfirmacaoSenha: Equal(Senha)
```

---

### 3️⃣ **@backend-writer** - Implementador Backend

**Arquivo**: `.github/prompts/backend.writer.prompt.md`

**Responsabilidade**: Implementar backend completo de forma automatizada

**⚠️ Regras de Segurança**:
- NUNCA executa `dotnet ef database update` ou comandos DDL sem confirmação
- SEMPRE solicita nome da solution antes de criar projetos
- Gera apenas código (.cs) e migrations, sem aplicar ao banco

**Entrada**: PRD técnico

**Saída**: Código completo seguindo Clean Architecture:

**Domain Layer**
```csharp
// Entidade com factory method e invariantes
public sealed class Usuario : BaseEntity
{
    public static Usuario Criar(string email, string senhaHash)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new DomainException("Email é obrigatório");
        
        return new Usuario 
        { 
            Id = Guid.NewGuid(), 
            Email = email.Trim(),
            SenhaHash = senhaHash,
            DataCriacao = DateTimeOffset.UtcNow
        };
    }
}
```

**Application Layer**
```csharp
// Validador
public sealed class CriarUsuarioRequestValidator : AbstractValidator<CriarUsuarioRequest>
{
    public CriarUsuarioRequestValidator(IUsuarioRepository repository)
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MustAsync(async (email, ct) => await repository.EhEmailUnicoAsync(email, null, ct))
            .WithMessage("Email já cadastrado");
    }
}

// Handler
public sealed class CriarUsuarioHandler
{
    public async Task<Result<UsuarioResponse>> HandleAsync(
        CriarUsuarioRequest request,
        CancellationToken cancellationToken = default)
    {
        // 1. Validar
        var validation = await _validator.ValidateAsync(request, cancellationToken);
        if (!validation.IsValid)
            return Result<UsuarioResponse>.Failure(Error.Validation(errors));

        // 2. Aplicar regras de domínio
        var usuario = Usuario.Criar(request.Email, _passwordHasher.Hash(request.Senha));

        // 3. Persistir
        await _repository.AdicionarAsync(usuario, cancellationToken);
        await _unitOfWork.CommitAsync(cancellationToken);

        return Result<UsuarioResponse>.Success(new UsuarioResponse { Id = usuario.Id });
    }
}
```

**Infrastructure Layer**
```csharp
// Repository
public sealed class UsuarioRepository : IUsuarioRepository
{
    public async Task<bool> EhEmailUnicoAsync(string email, Guid? excluirId = null, CancellationToken ct = default)
    {
        var query = _context.Usuarios.AsNoTracking().Where(u => u.Email == email);
        if (excluirId.HasValue) query = query.Where(u => u.Id != excluirId.Value);
        return !await query.AnyAsync(ct);
    }
}

// EF Configuration
public sealed class UsuarioConfiguration : IEntityTypeConfiguration<Usuario>
{
    public void Configure(EntityTypeBuilder<Usuario> builder)
    {
        builder.ToTable("usuarios");
        builder.Property(u => u.Email).HasColumnName("email").IsRequired();
        builder.HasIndex(u => u.Email).IsUnique().HasDatabaseName("ix_usuarios_email_unique");
    }
}
```

**API Layer**
```csharp
[ApiController]
[Route("api/v1/[controller]")]
public sealed class UsuariosController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> Criar(
        [FromBody] CriarUsuarioRequest request,
        CancellationToken cancellationToken)
    {
        var result = await _criar.HandleAsync(request, cancellationToken);

        return result.Match(
            onSuccess: r => CreatedAtAction(null, r),
            onFailure: e => BadRequest(CreateProblemDetails(e)));
    }
}
```

**Saída também inclui**:
- ✅ EF Core Migration completa
- ✅ Arquivo `.csproj` configurado
- ✅ Estrutura de pastas criada
- ✅ GlobalUsings.cs
- ✅ DI Configuration (Program.cs)

---

### 4️⃣ **@unittest-writer** - Testador Automático

**Arquivo**: `.github/prompts/unittest.writer.prompt.md`

**Responsabilidade**: Gerar testes unitários Domain + Application

**Entrada**: PRD e código implementado

**Saída**: Testes com xunit + FluentAssertions + NSubstitute

```csharp
public class UsuarioTests
{
    [Fact]
    public void Criar_ComDadosValidos_RetornaUsuario()
    {
        // Arrange & Act
        var usuario = Usuario.Criar("teste@exemplo.com", "hash123");

        // Assert
        usuario.Email.Should().Be("teste@exemplo.com");
        usuario.Ativo.Should().BeTrue();
        usuario.Id.Should().NotBeEmpty();
    }

    [Fact]
    public void Criar_ComEmailVazio_LancaDomainException()
    {
        // Act & Assert
        var act = () => Usuario.Criar("", "hash123");
        act.Should().Throw<DomainException>().WithMessage("Email é obrigatório");
    }
}

public class CriarUsuarioHandlerTests
{
    [Fact]
    public async Task HandleAsync_ComDadosValidos_RetornaSucesso()
    {
        // Arrange
        var request = new CriarUsuarioRequest 
        { 
            Email = "teste@exemplo.com", 
            Senha = "SenhaSegura123",
            ConfirmacaoSenha = "SenhaSegura123"
        };

        var repositoryMock = Substitute.For<IUsuarioRepository>();
        repositoryMock.EhEmailUnicoAsync("teste@exemplo.com", null, Arg.Any<CancellationToken>())
            .Returns(true);

        var handler = new CriarUsuarioHandler(repositoryMock, _passwordHasher, _unitOfWork, _validator, _logger);

        // Act
        var result = await handler.HandleAsync(request);

        // Assert
        result.IsSuccess.Should().BeTrue();
        result.Value.Email.Should().Be("teste@exemplo.com");
    }

    [Fact]
    public async Task HandleAsync_ComEmailDuplicado_RetornaConflict()
    {
        // Arrange
        var repositoryMock = Substitute.For<IUsuarioRepository>();
        repositoryMock.EhEmailUnicoAsync("teste@exemplo.com", null, Arg.Any<CancellationToken>())
            .Returns(false);

        // Act
        var result = await handler.HandleAsync(request);

        // Assert
        result.IsSuccess.Should().BeFalse();
        result.Error.Code.Should().Be(Error.CONFLICT);
    }
}
```

---

## 📋 Exemplo Prático

### Cenário: Implementar Cadastro de Usuário (US001)

#### **Step 1: Usar @story-writer**

```bash
# Comando
@story-writer: Crie uma User Story para "Permitir novos usuários se registrarem"
```

**Resultado**: `docs/stories/US001-cadastro-usuario.md`

---

#### **Step 2: Usar @prd-writer**

```bash
# Comando
@prd-writer: Gere PRD técnico para a US001
Caminho: docs/stories/US001-cadastro-usuario.md
```

**Resultado**: `docs/product-requirements/PRD-0001-gestao-usuarios/US001/PRD-US001-cadastro-usuario-2025-11-25.md`

Arquivo contém:
- Tabela de API Contracts (sem versionamento de API)
- Sequência de implementação (5 tasks)
- DTOs detalhados
- Fluxograma de criação
- Implementation Status por camada

---

#### **Step 3: Usar @backend-writer**

```bash
# Comando
Follow instructions in .github/prompts/backend.writer.prompt.md
Implemente: docs/prd/BKI-0001-gestao-usuarios/US001/PRD-US001-cadastro-usuario-2025-11-25.md
```

**Resultado**: Implementação completa em 7 arquivos principais:

```
✅ DOMAIN LAYER (2 files)
├─ Domain/Entities/Usuario.cs
├─ Domain/Interfaces/IUsuarioRepository.cs

✅ APPLICATION LAYER (4 files)
├─ Application/UseCases/Usuario/CriarUsuarioRequest.cs
├─ Application/UseCases/Usuario/UsuarioResponse.cs
├─ Application/UseCases/Usuario/CriarUsuarioRequestValidator.cs
├─ Application/UseCases/Usuario/CriarUsuarioHandler.cs

✅ INFRASTRUCTURE LAYER (5 files)
├─ Infrastructure/Data/AppDbContext.cs
├─ Infrastructure/Data/Configurations/UsuarioConfiguration.cs
├─ Infrastructure/Repositories/UsuarioRepository.cs
├─ Infrastructure/Migrations/20251128_US001_InitialCreate.cs
├─ Infrastructure/Extensions/ServiceCollectionExtensions.cs

✅ API LAYER (2 files)
├─ API/Controllers/v1/UsuariosController.cs
├─ API/Program.cs

✅ BUILD SUCCESS
├─ 0 erros, 8 avisos (package version resolution)
├─ Migration criada (NÃO aplicada - requer confirmação do usuário)
```

**⚠️ Importante**: O agente NUNCA executa `dotnet ef database update`. Você deve executar manualmente após revisar.

---

#### **Step 4: Usar @unittest-writer**

```bash
# Comando
@unittest-writer: Gere testes unitários para US001
PRD: docs/prd/BKI-0001-gestao-usuarios/US001/PRD-US001-cadastro-usuario-2025-11-25.md
```

**Resultado**: `tests/{{ProjectBase}}.UnitTests/`

```
✅ Domain Tests
├─ UsuarioTests.cs (criação, invariantes)

✅ Application Tests
├─ CriarUsuarioHandlerTests.cs (sucesso, validação, conflito)
└─ CriarUsuarioValidatorTests.cs (regras)

Coverage: ~85% Application, 100% Domain
```

---

## 🏗️ Estrutura do Projeto

```
ia-driven-starter/
├── 📂 .github/
│   ├── copilot-instructions.md           ← Instruções globais
│   ├── agent.md                          ← Definição de agentes
│   ├── prompts/
│   │   ├── story.writer.prompt.md        ← @story-writer
│   │   ├── prd.writer.prompt.md          ← @prd-writer
│   │   ├── backend.writer.prompt.md      ← @backend-writer
│   │   └── unittest.writer.prompt.md     ← @unittest-writer
│   └── workflows/                        ← CI/CD (GitHub Actions)
│
├── 📂 docs/
│   ├── stories/
│   │   ├── US001-gestao-tarefas/
│   │   │   ├── US001.md (épico)
│   │   │   ├── US001-01-listar-tarefas.md
│   │   │   ├── US001-02-adicionar-tarefa.md
│   │   │   ├── US001-03-editar-tarefa.md
│   │   │   ├── US001-04-excluir-tarefa.md
│   │   │   └── US001-05-marcar-concluida.md
│   │   └── US002-registro-usuario.md
│   └── product-requirements/
│       └── PRD-0001-registro-usuario/
│           └── US002/
│               └── PRD-US002-registro-usuario-2025-12-11.md
│
├── 📂 src/
│   ├── backend/                          ← ASP.NET Core
│   │   ├── .github/
│   │   │   ├── copilot-instructions.md   ← Instruções backend
│   │   │   └── instructions/             ← Documentação detalhada
│   │   │       ├── 01-architecture.md
│   │   │       ├── 02-patterns.md
│   │   │       ├── 03-conventions.md
│   │   │       ├── 04-folder-structure.md
│   │   │       ├── 05-database.md
│   │   │       ├── 06-testing.md
│   │   │       ├── 07-setup.md
│   │   │       └── 08-legacy-migration.md
│   │   ├── {{ProjectBase}}.sln
│   │   ├── {{ProjectBase}}.Domain/
│   │   ├── {{ProjectBase}}.Application/
│   │   ├── {{ProjectBase}}.Infrastructure/
│   │   └── {{ProjectBase}}.API/
│   │
│   └── frontend/                         ← React + TypeScript
│       ├── .github/
│       │   └── copilot-instructions.md   ← Instruções frontend
│       ├── src/
│       ├── public/
│       └── package.json
│
└── 📂 tests/
    └── {{ProjectBase}}.UnitTests/
        ├── Domain/
        └── Application/

```

---

## 📖 Stack Tecnológico

### Backend
- **Framework**: ASP.NET Core 10.0 (Minimal APIs)
- **Database**: PostgreSQL com Npgsql
- **ORM**: Entity Framework Core 9.0
- **Validation**: FluentValidation 11.x
- **Testing**: xunit, FluentAssertions, NSubstitute
- **Architecture**: Clean Architecture + DDD

### Frontend
- **Framework**: React 18+
- **Language**: TypeScript 5+
- **Styling**: TailwindCSS ou MUI (a definir)
- **State**: Zustand / Context API
- **Testing**: Vitest, React Testing Library

### DevOps
- **Version Control**: Git + GitHub
- **CI/CD**: GitHub Actions
- **Containerization**: Docker (em desenvolvimento)

---

## 🎯 Como Começar com IA

### 1. Entender as Instruções
```bash
# Leia primeiro (ordem importante)
cat .github/copilot-instructions.md
cat src/backend/.github/copilot-instructions.md
```

### 2. Criar Uma User Story
```bash
# Use o @story-writer
@story-writer: Crie uma US para [sua funcionalidade]
```

### 3. Gerar PRD Técnico
```bash
# Use o @prd-writer
@prd-writer: Gere PRD para [arquivo da US]
```

### 4. Implementar Backend
```bash
# Use o @backend-writer
@backend-writer: Implemente [arquivo do PRD]
```

### 5. Criar Testes
```bash
# Use o @unittest-writer
@unittest-writer: Crie testes para [arquivo do PRD]
```

---

## 📝 Convenções Importantes

### Nomenclatura
- **Entidades**: PascalCase em português (Usuario, Pedido)
- **Colunas BD**: snake_case em português (email, data_criacao)
- **Handlers**: `Criar{Entidade}Handler`, `Atualizar{Entidade}Handler`
- **Requests**: `Criar{Entidade}Request`
- **Responses**: `{Entidade}Response`

### Commits
```
feat(us001): implementar cadastro de usuário
- Adicionar entidade Usuario
- Criar handlers de criação
- Endpoints em UsuariosController
- Gerar migration inicial

Refs: #US001 BKI-0001-gestao-usuarios
```

### Branches
```
feature/us001-cadastro-usuario
bugfix/email-validation-issue
hotfix/security-patch
```

---

## 🔗 Links Importantes

- **Global Instructions**: `.github/copilot-instructions.md`
- **Backend Instructions**: `src/backend/.github/copilot-instructions.md`
- **Architecture Guide**: `src/backend/.github/instructions/01-architecture.md`
- **Pattern Templates**: `src/backend/.github/instructions/02-patterns.md` (inclui BaseApiController)
- **Example Story (Epic)**: `docs/stories/US001-gestao-tarefas/US001.md`
- **Example Story (Feature)**: `docs/stories/US002-registro-usuario.md`
- **Example PRD**: `docs/product-requirements/PRD-0001-registro-usuario/US002/PRD-US002-*.md`

---

## 💡 Dicas para Melhor Usar IA

1. **Seja específico**: Forneça contexto completo (arquivo da US, PRD, etc)
2. **Revise primeiro**: Leia as instruções antes de pedir implementação
3. **Use padrões**: Siga as templates definidas nos prompts
4. **Teste incrementalmente**: Construa feature por feature, não tudo de uma vez
5. **Documente**: Mantenha stories e PRDs atualizados
6. **⚠️ Segurança**: Sempre revise migrations antes de aplicar ao banco
7. **⚠️ Nomenclatura**: Informe o nome da solution quando criar novos projetos
8. **Valide commits**: Antes de fazer merge, verifique:
   - Build passa (`dotnet build`)
   - Sem warnings críticos
   - Migrations revisadas e aplicadas manualmente (`dotnet ef database update`)
   - Testes passam (`dotnet test`)
   - Nenhum comando `dotnet ef` foi executado automaticamente

---

## 🤝 Contribuindo

1. Crie uma **User Story** usando `@story-writer`
2. Revise e aprove com o time
3. Gere **PRD técnico** com `@prd-writer`
4. Implemente com `@backend-writer` e/ou frontend equivalent
5. Crie **testes** com `@unittest-writer`
6. Abra **Pull Request** com referência à US

---

## 📄 Licença

MIT

---

**Desenvolvido com ❤️ e 🤖 GitHub Copilot**

*Última atualização: Dezembro 2025*

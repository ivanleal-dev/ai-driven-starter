# 🏛️ Clean Architecture - Visão Geral

> 📋 **Stack & Versões**: [../../09-stack.md](../../09-stack.md)
## 📚 Visão Geral

Backend desenvolvido seguindo **Clean Architecture**, **SOLID** e **Domain-Driven Design (DDD)**.

---

## 🎯 Princípios Fundamentais

### 1. Separation of Concerns
Cada camada tem responsabilidade única e bem definida.

```
┌─────────────────────────────────────┐
│  API        (Controllers, DI)       │
├─────────────────────────────────────┤
│  Application (Handlers, Validators) │
├─────────────────────────────────────┤
│  Domain     (Entities, Rules)       │
├─────────────────────────────────────┤
│  Infrastructure (EF Core, Repos)    │
└─────────────────────────────────────┘
```

### 2. Dependency Inversion
Camadas **externas** (API, Infrastructure) dependem de **abstrações** internas (Domain).

```
❌ Errado:
Domain → Infrastructure → Application → API

✅ Correto:
API → Application ↓
      ↓
Application → Domain ↑
      ↑
Infrastructure → Domain (implementa)
```

### 3. INVARIANTS FIRST
Entidades validam suas **próprias regras de negócio** no Domain.

```csharp
// ✅ Correto: Validação no Domain
public static User Criar(string email)
{
    if (string.IsNullOrWhiteSpace(email))
        throw new DomainException("Email obrigatório");
    return new User { Email = email };
}

// ❌ Errado: Validação no Handler
if (string.IsNullOrWhiteSpace(email))
    return Result.Failure("Email obrigatório");
```

### 4. Explicit over Implicit
Código claro, sem "magia" ou convenções implícitas ocultas.

```csharp
// ✅ Explícito
public sealed class CriarUserHandler { }
public async Task<Result<UserResponse>> HandleAsync(...) { }

// ❌ Implícito
public class UserService { }
public async Task<UserDTO> Create(...) { }  // Retorna DTO implicitamente
```

### 5. Sem Mediator/MediatR
**NÃO usar padrão Mediator** (ex: MediatR). Handlers são injetados **diretamente via DI**.

```csharp
// ✅ Correto: DI direto
public sealed class UsersController(CriarUserHandler criar, ObterUserPorIdHandler obter)
{
    public async Task<IActionResult> Post([FromBody] CriarUserRequest req, CancellationToken ct)
        => await criar.HandleAsync(req, ct);
}

// ❌ Errado: Mediator
public sealed class UsersController(IMediator mediator)
{
    public async Task<IActionResult> Post([FromBody] CriarUserRequest req, CancellationToken ct)
        => await mediator.Send(req, ct);  // Indireção desnecessária
}
```

---

## 🏗️ Camadas da Arquitetura

### 1️⃣ Domain Layer (Núcleo)

**Responsabilidade**: Modelo de negócio puro, **sem dependências externas**.

**Contém**:
- ✅ Entities (com invariantes)
- ✅ Value Objects
- ✅ Interfaces de Repositórios (contratos)
- ✅ Enums e tipos de domínio
- ✅ Exceções de domínio (`DomainException`)

**NÃO pode**:
- ❌ Referenciar outras camadas
- ❌ Ter dependência de frameworks
- ❌ Acessar banco de dados
- ❌ Chamar APIs externas
- ❌ Conhecer resultado de operações (Result Pattern)

**Exemplo**:
```csharp
namespace AgenteViagem.Domain.Entities;

public sealed class Usuario : BaseEntity
{
    public string Email { get; private set; } = null!;
    
    public static Usuario Criar(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new DomainException("Email obrigatório");
        
        return new Usuario { Id = Guid.NewGuid(), Email = email.Trim() };
    }
}
```

### 2️⃣ Application Layer (Orquestração)

**Responsabilidade**: Orquestrar **casos de uso isolados** (um handler por intenção).

**Contém**:
- ✅ Handlers (por caso de uso)
- ✅ Validadores (FluentValidation)
- ✅ DTOs/Requests/Responses
- ✅ Result Pattern (`Result<T>`, `Error`)
- ✅ Pagination (`ResultadoPaginado<T>`)

**NÃO contém**:
- ❌ Serviços genéricos centrais (`UserService`, `ClienteService`)
- ❌ Acesso direto a EF Core
- ❌ Lógica de infraestrutura
- ❌ Acesso a HTTP/APIs (isso é Infrastructure)

**Organização**: Vertical Slices por agregado
```
Application/
├── UseCases/
│   ├── Usuario/
│   │   ├── UsuarioResponse.cs
│   │   ├── CriarUsuarioRequest.cs
│   │   ├── CriarUsuarioRequestValidator.cs
│   │   ├── CriarUsuarioHandler.cs
│   │   ├── AtualizarUsuarioRequest.cs
│   │   ├── AtualizarUsuarioRequestValidator.cs
│   │   ├── AtualizarUsuarioHandler.cs
│   │   ├── ObterUsuarioPorIdRequest.cs
│   │   ├── ObterUsuarioPorIdHandler.cs
│   │   ├── ObterUsuarioPaginadoRequest.cs
│   │   ├── ObterUsuarioPaginadoHandler.cs
│   │   ├── ExcluirUsuarioRequest.cs
│   │   └── ExcluirUsuarioHandler.cs
│   └── Cliente/ (estrutura análoga)
├── Common/
│   ├── Result.cs
│   ├── Error.cs
│   └── Pagination/ResultadoPaginado.cs
└── Extensions/
    └── ServiceCollectionExtensions.cs
```

### 3️⃣ Infrastructure Layer (Implementações)

**Responsabilidade**: Detalhes técnicos e integrações externas.

**Contém**:
- ✅ Entity Framework Core
- ✅ Repositórios (implementações concretas)
- ✅ Configurações EF (`IEntityTypeConfiguration`)
- ✅ Migrations
- ✅ Serviços externos (Cache, Email, Storage)
- ✅ UnitOfWork

**NÃO pode**:
- ❌ Conter lógica de negócio
- ❌ Validar regras de domínio
- ❌ Orquestrar casos de uso

**Exemplo**:
```csharp
namespace AgenteViagem.Infrastructure.Repositories;

public sealed class UsuarioRepository : IUsuarioRepository
{
    private readonly AppDbContext _context;
    
    public async Task<Usuario?> ObterPorIdAsync(Guid id, CancellationToken ct = default)
        => await _context.Usuarios.FirstOrDefaultAsync(u => u.Id == id, ct);
}
```

### 4️⃣ API Layer (Apresentação)

**Responsabilidade**: Entrada da aplicação e HTTP.

**Contém**:
- ✅ Controllers REST
- ✅ Middleware
- ✅ Filters/Validation
- ✅ Configuração DI (Program.cs)
- ✅ OpenAPI/Swagger

**NÃO pode**:
- ❌ Conter lógica de negócio
- ❌ Acessar repositórios diretamente
- ❌ Conhecer detalhes de infraestrutura

**Exemplo**:
```csharp
namespace AgenteViagem.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public sealed class UsuariosController(CriarUsuarioHandler criar)
{
    [HttpPost]
    public async Task<IActionResult> Criar(
        [FromBody] CriarUsuarioRequest request, 
        CancellationToken ct)
    {
        var result = await criar.HandleAsync(request, ct);
        return result.Match(r => CreatedAtAction(nameof(ObterPorId), r), 
                            e => BadRequest(e));
    }
}
```

---

## 📊 Fluxo de Dados

```
┌─────────────┐
│   Request   │
└──────┬──────┘
       │
       ▼
┌──────────────────────────┐
│    API Controller        │
│ (recebe + delega)        │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│   Handler                │
│ 1. Valida (FluentVal)    │
│ 2. Busca dados (Repo)    │
│ 3. Aplica regras (Domain)│
│ 4. Persiste (UoW)        │
│ 5. Retorna Result        │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│   Result Pattern         │
│ - Success(data)          │
│ - Failure(error)         │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│   Controller Match       │
│ - 200 / 400 / 404 / etc  │
└──────┬───────────────────┘
       │
       ▼
┌──────────────┐
│   Response   │
└──────────────┘
```

---

## 🔄 Ciclo de Vida de um Caso de Uso

**Exemplo: Criar Usuário**

```
1. [API] POST /api/usuarios
   └─> Deserializar CriarUsuarioRequest

2. [Controller] Injetar CriarUsuarioHandler
   └─> criar.HandleAsync(request, ct)

3. [Handler] Executar
   a. Validar via CriarUsuarioRequestValidator
   b. Se inválido → Result.Failure(validationError)
   c. Se válido → Usuario.Criar(email) [Domain invariants]
   d. _repository.AdicionarAsync(usuario)
   e. _unitOfWork.CommitAsync() [Transação]
   f. Retornar Result.Success(usuarioResponse)

4. [Controller] Mapear resultado
   result.Match(
       success: r => CreatedAtAction(nameof(ObterPorId), r),
       failure: e => BadRequest(CreateProblemDetails(e))
   )

5. [HTTP] Enviar resposta
   201 Created + Location header
```

---

## ✅ Checklist de Arquitetura

Ao implementar nova feature, validar:

- [ ] **Domain**: Entidade com invariantes validadas
- [ ] **Domain**: Interface de repositório criada
- [ ] **Application**: Handler para cada caso de uso
- [ ] **Application**: Validador específico
- [ ] **Application**: Result Pattern retornado
- [ ] **Infrastructure**: Repositório implementado
- [ ] **Infrastructure**: Configuration EF criada
- [ ] **Infrastructure**: Migration gerada
- [ ] **API**: Controller injetar handler (não mediator)
- [ ] **API**: Usar Match() para mapear resultado
- [ ] **Logging**: Estruturado em pontos críticos
- [ ] **Testes**: Domain + Application (nunca Infrastructure/Controller)

---

**Próxima Seção**: Ver [`02-patterns.md`](./02-patterns.md) para templates de código.

# 📝 Nomenclatura & Convenções

## 🎯 Princípios

1. **Explícito**: Nomes claros, sem abreviações
2. **Português**: Entidades, métodos, variáveis em PT-BR
3. **Consistente**: Mesmo padrão em todo o codebase
4. **PascalCase/camelCase**: Siga convenções C#

---

## 🏷️ Nomenclatura por Contexto

### Entities (Domain)

```csharp
// ✅ Correto
public sealed class Usuario : BaseEntity { }
public sealed class FailedLoginAttempts : BaseEntity { }
public sealed class Pedido : BaseEntity { }
public sealed class ItemPedido : BaseEntity { }

// ❌ Errado
public sealed class User : BaseEntity { }  // Inglês
public sealed class usr : BaseEntity { }   // Abreviação
public sealed class CriarUsuario : BaseEntity { }  // Ação, não entidade
```

### Value Objects (Domain)

```csharp
// ✅ Correto
public sealed class Email : ValueObject { }
public sealed class Cpf : ValueObject { }
public sealed class Telefone : ValueObject { }

// ❌ Errado
public sealed class EmailVO : ValueObject { }  // Sufixo redundante
public sealed class E : ValueObject { }        // Abreviação
```

### Interfaces (Domain/Application)

```csharp
// ✅ Correto (I + Nome + contexto)
public interface IUsuarioRepository : IRepository<Usuario> { }
public interface IUnitOfWork { }
public interface ITokenService { }
public interface IPasswordHasher { }

// ❌ Errado
public interface IUserRepository { }  // Inglês
public interface UsuarioRepository { }  // Não é interface
```

### Handlers (Application)

**Padrão PT-BR para ações**:
- `Criar{Entidade}Handler`
- `Atualizar{Entidade}Handler`
- `Obter{Entidade}PorIdHandler`
- `Obter{Entidade}PaginadoHandler`
- `Excluir{Entidade}Handler`

```csharp
// ✅ Correto
public sealed class CriarUsuarioHandler { }
public sealed class AtualizarUsuarioHandler { }
public sealed class ObterUsuarioPorIdHandler { }
public sealed class ObterUsuarioPaginadoHandler { }
public sealed class ExcluirUsuarioHandler { }

// ❌ Errado
public sealed class CreateUserHandler { }  // Inglês
public sealed class UsuarioService { }  // Não é handler
public sealed class UsuarioCreateHandler { }  // Ordem invertida
public sealed class GetAllUsersHandler { }  // Plural em inglês
```

### Requests/DTOs (Application)

```csharp
// ✅ Correto
public sealed record CriarUsuarioRequest { }
public sealed record AtualizarUsuarioRequest { }
public sealed record ObterUsuarioPorIdRequest { }

// ❌ Errado
public sealed record CreateUserRequest { }  // Inglês
public sealed record UsuarioCreateDto { }  // DTO genérico
public sealed record CreateRequest { }  // Muito genérico
```

### Responses (Application)

```csharp
// ✅ Correto
public sealed record UsuarioResponse { }
public sealed record ListaUsuariosResponse { }
public sealed record ErroResponse { }

// ❌ Errado
public sealed record UserResponse { }  // Inglês
public sealed record UsuarioDTO { }  // DTO
public sealed record UsuarioVM { }  // ViewModel (fora de escopo)
```

### Validators (Application)

```csharp
// ✅ Correto
public sealed class CriarUsuarioRequestValidator : AbstractValidator<CriarUsuarioRequest> { }
public sealed class AtualizarUsuarioRequestValidator : AbstractValidator<AtualizarUsuarioRequest> { }

// ❌ Errado
public sealed class CreateUserValidator { }  // Inglês
public sealed class UsuarioValidator { }  // Muito genérico
```

### Repositories (Infrastructure)

```csharp
// ✅ Correto
public sealed class UsuarioRepository : IUsuarioRepository { }
public sealed class PedidoRepository : IPedidoRepository { }

// ❌ Errado
public sealed class UserRepository : IUserRepository { }  // Inglês
public sealed class UsuarioRepositoryImpl { }  // Sufixo redundante
```

### Controllers (API)

```csharp
// ✅ Correto (Plural)
[Route("api/v1/[controller]")]
public sealed class UsuariosController : ControllerBase { }

[Route("api/v1/[controller]")]
public sealed class PedidosController : ControllerBase { }

// ❌ Errado
public sealed class UsuarioController { }  // Singular
public sealed class UserController { }  // Inglês
public sealed class CriarUsuarioController { }  // Ação como controller
```

### Métodos de Handler

```csharp
// ✅ Correto
public async Task<Result<UsuarioResponse>> HandleAsync(CriarUsuarioRequest request, CancellationToken ct)
public async Task<Result<UsuarioResponse>> HandleAsync(AtualizarUsuarioRequest request, CancellationToken ct)
public async Task<Result<UsuarioResponse>> HandleAsync(ObterUsuarioPorIdRequest request, CancellationToken ct)

// ❌ Errado
public async Task<UserResponse> Execute(CreateUserRequest request)  // Sem Result, sem ct
public async Task<Result> Handle(...)  // Sem especificar tipo
```

### Métodos de Entidade

```csharp
// ✅ Correto (Ações claras)
public static Usuario Criar(string email, string senhaHash) { }
public void Atualizar(string email) { }
public void Desativar() { }
public void Ativar() { }
public void AtualizarDataUltimoLogin() { }

// ❌ Errado
public static Usuario New(...) { }  // Inglês
public void Update(...) { }  // Inglês
public void Set(...) { }  // Muito genérico
```

### Propriedades

```csharp
// ✅ Correto (PascalCase, PT-BR)
public string Email { get; private set; }
public string SenhaHash { get; private set; }
public DateTimeOffset DataCriacao { get; private set; }
public DateTimeOffset? DataUltimoLogin { get; private set; }
public bool Ativo { get; private set; }

// ❌ Errado
public string email { get; private set; }  // camelCase em propriedade
public string pwd { get; private set; }  // Abreviação
public DateTime Created { get; private set; }  // Inglês
```

### Variáveis Locais

```csharp
// ✅ Correto (camelCase, PT-BR)
var usuario = await _repository.ObterPorIdAsync(id, ct);
var senhaHash = _passwordHasher.Hash(request.Senha);
var errosValidacao = validation.Errors;
var ehEmailUnico = await _repository.EhEmailUnicoAsync(email, null, ct);

// ❌ Errado
var usr = ...;  // Abreviação
var user = ...;  // Inglês
var u = ...;  // Muito genérico
```

### Parâmetros

```csharp
// ✅ Correto
public void AtualizarEmail(string novoEmail)
public async Task<Usuario?> ObterPorIdAsync(Guid id, CancellationToken ct = default)
public async Task<bool> EhEmailUnicoAsync(string email, Guid? excluirId = null, CancellationToken ct = default)

// ❌ Errado
public void UpdateEmail(string newEmail)  // Inglês
public async Task<Usuario?> GetById(Guid id)  // Inglês
public async Task<bool> IsEmailUnique(string em)  // Abreviação
```

### Métodos Privados/Helpers

```csharp
// ✅ Correto
private ProblemDetails CriarDetalhesProblema(Error erro) { }
private ValidationProblemDetails CriarDetalhesValidacao(Error erro) { }
private async Task<bool> UsuarioExisteAsync(Guid id, CancellationToken ct) { }

// ❌ Errado
private ProblemDetails CreateProblemDetails(Error error) { }  // Inglês
private bool check(...) { }  // Abreviação
private void help(...) { }  // Muito genérico
```

---

## 🔤 C# 12 & .NET 10 Features

### File-scoped Namespaces

```csharp
// ✅ Correto (C# 10+)
namespace AgenteViagem.API.Controllers.v1;

public sealed class UsuariosController : ControllerBase { }

// ❌ Errado (C# 9 style)
namespace AgenteViagem.API.Controllers.v1
{
    public sealed class UsuariosController : ControllerBase { }
}
```

### Required Properties (C# 11)

```csharp
// ✅ Correto
public sealed record CriarUsuarioRequest
{
    public required string Email { get; init; }
    public required string Senha { get; init; }
}

// ❌ Errado (sem required, sem null checking)
public sealed record CriarUsuarioRequest
{
    public string? Email { get; init; }
    public string? Senha { get; init; }
}
```

### Primary Constructors (C# 12)

```csharp
// ✅ Correto (C# 12)
public sealed class CriarUsuarioHandler(
    IUsuarioRepository repository,
    IUnitOfWork unitOfWork,
    IValidator<CriarUsuarioRequest> validator,
    ILogger<CriarUsuarioHandler> logger)
{
    public async Task<Result<UsuarioResponse>> HandleAsync(...)
    {
        // Usar repository, unitOfWork, validator, logger diretamente
        await repository.AdicionarAsync(...);
    }
}

// ❌ Errado (C# 11 style)
public sealed class CriarUsuarioHandler
{
    private readonly IUsuarioRepository _repository;
    
    public CriarUsuarioHandler(IUsuarioRepository repository)
    {
        _repository = repository;
    }
}
```

### Sealed Classes

```csharp
// ✅ Correto (padrão)
public sealed class UsuariosController : ControllerBase { }
public sealed record UsuarioResponse { }
public sealed class CriarUsuarioHandler { }

// ❌ Errado
public class UsuariosController : ControllerBase { }  // Sem sealed
public record UsuarioResponse { }  // Record sem sealed
```

### Target-typed new

```csharp
// ✅ Correto
Result<UsuarioResponse> result = Result<UsuarioResponse>.Success(new() 
{ 
    Id = usuario.Id,
    Email = usuario.Email 
});

// ❌ Errado
Result<UsuarioResponse> result = new Result<UsuarioResponse>(true, new UsuarioResponse() { ... }, null);
```

---

## 📦 Global Usings

**Arquivo**: `GlobalUsings.cs` (na raiz de cada projeto)

```csharp
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using Microsoft.Extensions.Logging;
global using FluentValidation;
```

---

## 🗂️ Constantes

```csharp
// Domain/Common/Constants.cs
namespace AgenteViagem.Domain.Common;

public static class DomainConstants
{
    public const int MaximoTentativasLogin = 5;
    public const int JanelaTempoMinutosLogin = 1;
    public const int BloqueioMinutosLogin = 15;
    public const int MaximoTamanhoPagina = 100;
}

// Uso
if (tentativas >= DomainConstants.MaximoTentativasLogin)
    usuario.Bloquear(DomainConstants.BloqueioMinutosLogin);
```

---

## ✅ Checklist de Nomenclatura

- [ ] Todas entities em PT-BR (Usuario, Pedido, etc)
- [ ] Handlers seguem padrão: Criar/Atualizar/ObterPorId/Obter Paginado/Excluir
- [ ] Interfaces começam com `I` (IUsuarioRepository)
- [ ] Controllers no plural (UsuariosController)
- [ ] Propriedades em PascalCase
- [ ] Variáveis em camelCase
- [ ] Sem abreviações (usr → usuario, pwd → senha)
- [ ] Sem inglês (User → Usuario, Create → Criar)
- [ ] Métodos usam verbos (Criar, Atualizar, Desativar)
- [ ] Records usam `sealed` e `required`
- [ ] CancellationToken como `ct`

---

**Próxima Seção**: Ver [`04-folder-structure.md`](./04-folder-structure.md) para estrutura de pastas.

# 🎨 Padrões de Implementação

## 📌 Indice de Padrões

1. [Result Pattern](#result-pattern)
2. [Entity Pattern](#entity-pattern)
3. [Repository Pattern](#repository-pattern)
4. [Handler Pattern](#handler-pattern)
5. [Validator Pattern](#validator-pattern)
6. [Controller Pattern](#controller-pattern)
7. [Value Object Pattern](#value-object-pattern)

---

## Result Pattern

### Propósito
Retornar sucesso ou fracasso estruturado (sem exceções em fluxos normais).

### Implementação

```csharp
// Application/Common/Results/Result.cs
namespace <SolutionName>.Application.Common.Results;

public sealed class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public Error? Error { get; }

    private Result(bool isSuccess, T? value, Error? error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }

    public static Result<T> Success(T value) => new Result<T>(true, value, null);
    public static Result<T> Failure(Error error) => new Result<T>(false, default, error);

    public TResult Match<TResult>(
        Func<T, TResult> onSuccess,
        Func<Error, TResult> onFailure) =>
        IsSuccess ? onSuccess(Value!) : onFailure(Error!);

    public async Task<TResult> MatchAsync<TResult>(
        Func<T, Task<TResult>> onSuccess,
        Func<Error, Task<TResult>> onFailure) =>
        IsSuccess ? await onSuccess(Value!) : await onFailure(Error!);
}

public sealed class Result
{
    public bool IsSuccess { get; }
    public Error? Error { get; }

    private Result(bool isSuccess, Error? error)
    {
        IsSuccess = isSuccess;
        Error = error;
    }

    public static Result Success() => new Result(true, null);
    public static Result Failure(Error error) => new Result(false, error);
}

// Application/Common/Results/Error.cs
public sealed class Error
{
    public const string VALIDATION_ERROR = "validation_error";
    public const string NOT_FOUND = "not_found";
    public const string UNAUTHORIZED = "unauthorized";
    public const string CONFLICT = "conflict";

    public string Code { get; }
    public string Message { get; }
    public Dictionary<string, string[]>? Details { get; }

    public Error(string code, string message, Dictionary<string, string[]>? details = null)
    {
        Code = code;
        Message = message;
        Details = details;
    }

    public static Error Validation(string message) => 
        new Error(VALIDATION_ERROR, message);

    public static Error Validation(Dictionary<string, string[]> details) =>
        new Error(VALIDATION_ERROR, "Um ou mais erros de validação", details);

    public static Error NotFound(string message) =>
        new Error(NOT_FOUND, message);

    public static Error Unauthorized(string message) =>
        new Error(UNAUTHORIZED, message);

    public static Error Conflict(string message) =>
        new Error(CONFLICT, message);
}
```

### Uso em Handler

```csharp
public async Task<Result<UsuarioResponse>> HandleAsync(CriarUsuarioRequest request, CancellationToken ct)
{
    // Retornar sucesso
    return Result<UsuarioResponse>.Success(new UsuarioResponse { Id = usuario.Id });

    // Retornar falha
    return Result<UsuarioResponse>.Failure(Error.Validation("Email inválido"));
}
```

### Uso em Controller

```csharp
var result = await handler.HandleAsync(request, ct);
return result.Match(
    onSuccess: r => Ok(r),
    onFailure: e => BadRequest(CreateProblemDetails(e))
);
```

---

## Entity Pattern

### Propósito
Representar agregado de domínio com invariantes validadas.

### Template

```csharp
// Domain/Entities/Usuario.cs
namespace <SolutionName>.Domain.Entities;

public sealed class Usuario : BaseEntity
{
    // Propriedades (somente get, set privado)
    public string Email { get; private set; } = null!;
    public string SenhaHash { get; private set; } = null!;
    public bool Ativo { get; private set; } = true;
    public DateTimeOffset DataCriacao { get; private set; }

    // Coleções (para relacionamentos)
    private readonly List<FailedLoginAttempts> _tentativasLoginFalhadas = new();
    public IReadOnlyCollection<FailedLoginAttempts> TentativasLoginFalhadas => _tentativasLoginFalhadas.AsReadOnly();

    // Construtor privado (força usar factory)
    private Usuario() { }

    // ✅ Factory Method com Invariantes (usando guards e ValidarEntidade)
    public static Usuario Criar(string email, string senhaHash)
    {
        ValidarEntidade(email, senhaHash);

        return new Usuario
        {
            Id = Guid.NewGuid(),
            Email = email.Trim(),
            SenhaHash = senhaHash,
            Ativo = true,
            DataCriacao = DateTimeOffset.UtcNow
        };
    }

    // ✅ Métodos de Comportamento (usando guards)
    public void AtualizarEmail(string novoEmail)
    {
        DomainException.ThrowIfNullOrWhiteSpace(novoEmail, "Email não pode ser vazio");
        
        Email = novoEmail.Trim();
    }

    // ✅ Método centralizando validações da entidade
    private static void ValidarEntidade(string email, string senhaHash)
    {
        DomainException.ThrowIfNullOrWhiteSpace(email, "Email é obrigatório");
        DomainException.ThrowIfNullOrWhiteSpace(senhaHash, "Senha hash é obrigatório");
    }

    public void Desativar()
    {
        DomainException.ThrowIf(!Ativo, "Usuário já está desativado");
        
        Ativo = false;
    }

    public void Ativar()
    {
        DomainException.ThrowIf(Ativo, "Usuário já está ativo");
        
        Ativo = true;
    }
}

// Domain/Common/BaseEntity.cs
public abstract class BaseEntity
{
    public Guid Id { get; protected set; }
}

// Domain/Common/DomainException.cs
public sealed class DomainException : Exception
{
    public DomainException(string message) : base(message) { }

    /// <summary>
    /// Lança exceção se o valor for nulo ou vazio
    /// </summary>
    public static void ThrowIfNullOrWhiteSpace(string? value, string message)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new DomainException(message);
    }

    /// <summary>
    /// Lança exceção se o valor for nulo ou vazio E a condição for verdadeira
    /// </summary>
    public static void ThrowIfNullOrWhiteSpace(string? value, string message, Func<bool> condition)
    {
        if (string.IsNullOrWhiteSpace(value) && condition())
            throw new DomainException(message);
    }

    /// <summary>
    /// Lança exceção se a condição for verdadeira
    /// </summary>
    public static void ThrowIf(bool condition, string message)
    {
        if (condition)
            throw new DomainException(message);
    }

    /// <summary>
    /// Lança exceção se o valor for nulo
    /// </summary>
    public static void ThrowIfNull<T>(T? value, string message) where T : class
    {
        if (value is null)
            throw new DomainException(message);
    }

    /// <summary>
    /// Lança exceção se o Guid for vazio
    /// </summary>
    public static void ThrowIfEmpty(Guid value, string message)
    {
        if (value == Guid.Empty)
            throw new DomainException(message);
    }
}
```

### Invariantes vs Validadores

```csharp
// ✅ Invariante (Domain): Regra SEMPRE válida (usando guards)
public static Usuario Criar(string email)
{
    DomainException.ThrowIfNullOrWhiteSpace(email, "Email obrigatório");  // Sempre falha
    return new Usuario { Email = email };
}

// ✅ Validador (Application): Regra de negócio contextual
public class CriarUsuarioRequestValidator : AbstractValidator<CriarUsuarioRequest>
{
    public CriarUsuarioRequestValidator(IUsuarioRepository repo)
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MustAsync(async (email, ct) => 
                await repo.EhEmailUnicoAsync(email, null, ct))
            .WithMessage("Email já cadastrado");  // Pode variar por contexto
    }
}
```

---

## Repository Pattern

### Interface (Domain)

```csharp
// Domain/Interfaces/Repositories/IRepository.cs
namespace <SolutionName>.Domain.Interfaces.Repositories;

public interface IRepository<T> where T : BaseEntity
{
    Task<T?> ObterPorIdAsync(Guid id, CancellationToken ct = default);
    Task<(IReadOnlyCollection<T> Itens, int Total)> ListarPaginadoAsync(
        int numeroPagina, int tamanhoPagina, CancellationToken ct = default);
    Task AdicionarAsync(T entity, CancellationToken ct = default);
    void Atualizar(T entity);
    void Remover(T entity);
}

// Domain/Interfaces/Repositories/IUsuarioRepository.cs
public interface IUsuarioRepository : IRepository<Usuario>
{
    Task<bool> EhEmailUnicoAsync(string email, Guid? excluirId = null, CancellationToken ct = default);
    Task<Usuario?> ObterPorEmailAsync(string email, CancellationToken ct = default);
}
```

### Implementação (Infrastructure)

```csharp
// Infrastructure/Repositories/UsuarioRepository.cs
namespace <SolutionName>.Infrastructure.Repositories;

public sealed class UsuarioRepository : IUsuarioRepository
{
    private readonly AppDbContext _context;

    public UsuarioRepository(AppDbContext context) => _context = context;

    public async Task<Usuario?> ObterPorIdAsync(Guid id, CancellationToken ct = default)
        => await _context.Usuarios.AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == id, ct);

    public async Task<(IReadOnlyCollection<Usuario> Itens, int Total)> ListarPaginadoAsync(
        int numeroPagina, int tamanhoPagina, CancellationToken ct = default)
    {
        var query = _context.Usuarios.AsNoTracking();
        var total = await query.CountAsync(ct);
        var itens = await query
            .OrderBy(u => u.Email)
            .Skip((numeroPagina - 1) * tamanhoPagina)
            .Take(tamanhoPagina)
            .ToListAsync(ct);
        return (itens, total);
    }

    public async Task AdicionarAsync(Usuario entity, CancellationToken ct = default)
        => await _context.Usuarios.AddAsync(entity, ct);

    public void Atualizar(Usuario entity) => _context.Usuarios.Update(entity);

    public void Remover(Usuario entity) => _context.Usuarios.Remove(entity);

    public async Task<bool> EhEmailUnicoAsync(string email, Guid? excluirId = null, CancellationToken ct = default)
    {
        var query = _context.Usuarios.AsNoTracking().Where(u => u.Email == email);
        if (excluirId.HasValue) query = query.Where(u => u.Id != excluirId.Value);
        return !await query.AnyAsync(ct);
    }

    public async Task<Usuario?> ObterPorEmailAsync(string email, CancellationToken ct = default)
        => await _context.Usuarios.AsNoTracking()
            .FirstOrDefaultAsync(u => u.Email == email, ct);
}
```

---

## Domain Service Pattern

### Propósito
Representar operações/capacidades do domínio que não pertencem a nenhuma entidade específica. A **interface** fica no Domain, a **implementação** fica na Infrastructure.

### Interface (Domain)

```csharp
// Domain/Interfaces/Services/IPasswordHasher.cs
namespace <SolutionName>.Domain.Interfaces.Services;

public interface IPasswordHasher
{
    string Hash(string password);
    bool Verify(string password, string hash);
}

// Domain/Interfaces/Services/ITokenService.cs
public interface ITokenService
{
    string GerarToken(Usuario usuario);
    bool ValidarToken(string token);
}
```

### Implementação (Infrastructure)

```csharp
// Infrastructure/Services/PasswordHasher.cs
namespace <SolutionName>.Infrastructure.Services;

public sealed class PasswordHasher : IPasswordHasher
{
    public string Hash(string password)
        => BCrypt.Net.BCrypt.HashPassword(password);

    public bool Verify(string password, string hash)
        => BCrypt.Net.BCrypt.Verify(password, hash);
}
```

### Registro de DI

```csharp
// Infrastructure ou API - Extensions/ServiceCollectionExtensions.cs
services.AddSingleton<IPasswordHasher, PasswordHasher>();
services.AddScoped<ITokenService, JwtTokenService>();
```

### Quando usar Domain Service vs Application Handler

| Domain Service | Application Handler |
|----------------|---------------------|
| Operação sem orquestração | Orquestra múltiplas operações |
| Sem dependência de repositório | Usa repositórios e validadores |
| Exemplo: Hash de senha | Exemplo: Criar usuário |

---

## Handler Pattern

### Template

```csharp
// Application/UseCases/Usuario/CriarUsuarioRequest.cs
namespace <SolutionName>.Application.UseCases.Usuario;

public sealed record CriarUsuarioRequest
{
    public required string Email { get; init; }
    public required string Senha { get; init; }
}

// Application/UseCases/Usuario/UsuarioResponse.cs
public sealed record UsuarioResponse
{
    public required Guid Id { get; init; }
    public required string Email { get; init; }
}

// Application/UseCases/Usuario/CriarUsuarioHandler.cs
public sealed class CriarUsuarioHandler
{
    private readonly IUsuarioRepository _repository;
    private readonly IPasswordHasher _passwordHasher;
    private readonly IValidator<CriarUsuarioRequest> _validator;
    private readonly ILogger<CriarUsuarioHandler> _logger;

    public CriarUsuarioHandler(
        IUsuarioRepository repository,
        IPasswordHasher passwordHasher,
        IValidator<CriarUsuarioRequest> validator,
        ILogger<CriarUsuarioHandler> logger)
    {
        _repository = repository;
        _passwordHasher = passwordHasher;
        _validator = validator;
        _logger = logger;
    }

    public async Task<Result<UsuarioResponse>> HandleAsync(
        CriarUsuarioRequest request,
        CancellationToken cancellationToken = default)
    {
        // 1. Validar
        var validation = await _validator.ValidateAsync(request, cancellationToken);
        if (!validation.IsValid)
        {
            // Padrão: converter erros do FluentValidation para Error.Validation(details)
            // Requer extension method: validation.ToValidationError()
            return Result<UsuarioResponse>.Failure(validation.ToValidationError());
        }

        try
        {
            // 2. Aplicar regras de domínio
            var senhaHash = _passwordHasher.Hash(request.Senha);
            var usuario = Usuario.Criar(request.Email, senhaHash);

            // 3. Persistir (Repository já chama SaveChangesAsync)
            await _repository.AdicionarAsync(usuario, cancellationToken);

            _logger.LogInformation("Usuário criado: {UsuarioId}", usuario.Id);

            // 4. Retornar sucesso
            return Result<UsuarioResponse>.Success(new UsuarioResponse
            {
                Id = usuario.Id,
                Email = usuario.Email
            });
        }
        catch (DomainException ex)
        {
            _logger.LogWarning("Erro de domínio ao criar usuário: {Mensagem}", ex.Message);
            return Result<UsuarioResponse>.Failure(Error.Conflict(ex.Message));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erro inesperado ao criar usuário");
            return Result<UsuarioResponse>.Failure(
                Error.Domain("Erro ao processar solicitação"));
        }
    }
}
```

---

## Validator Pattern

```csharp
// Application/UseCases/Usuario/CriarUsuarioRequestValidator.cs
namespace <SolutionName>.Application.UseCases.Usuario;

public sealed class CriarUsuarioRequestValidator : AbstractValidator<CriarUsuarioRequest>
{
    private readonly IUsuarioRepository _repository;

    public CriarUsuarioRequestValidator(IUsuarioRepository repository)
    {
        _repository = repository;

        RuleFor(x => x.Email)
            .NotEmpty()
            .WithMessage("Email é obrigatório")
            .EmailAddress()
            .WithMessage("Email deve ser válido")
            .MaximumLength(255)
            .WithMessage("Email deve ter no máximo 255 caracteres")
            .MustAsync(BeUniqueEmail)
            .WithMessage("Este email já está cadastrado");

        RuleFor(x => x.Senha)
            .NotEmpty()
            .WithMessage("Senha é obrigatória")
            .MinimumLength(6)
            .WithMessage("Senha deve ter no mínimo 6 caracteres");
    }

    private async Task<bool> BeUniqueEmail(string email, CancellationToken ct)
        => await _repository.EhEmailUnicoAsync(email, null, ct);
}
```

---

## Controller Pattern

### Base Controller (Infraestrutura Comum)

Todos os controllers devem herdar de `BaseController` para reutilizar métodos comuns:

```csharp
// API/Controllers/BaseController.cs
namespace <SolutionName>.API.Controllers;

/// <summary>
/// Classe base para controllers da API com métodos utilitários comuns
/// </summary>
public abstract class BaseController : ControllerBase
{
    /// <summary>
    /// Cria um resultado de problema (ProblemDetails) baseado em um erro
    /// </summary>
    /// <param name="error">O erro a ser convertido em ProblemDetails</param>
    /// <returns>IActionResult apropriado para o tipo de erro</returns>
    protected IActionResult CreateProblemResult(Error error)
    {
        var problemDetails = new ProblemDetails
        {
            Type = $"https://todoapp.com/problems/{error.Code}",
            Title = GetTitleForErrorCode(error.Code),
            Detail = error.Message,
            Instance = HttpContext.Request.Path
        };

        if (error.Details != null)
        {
            problemDetails.Extensions["errors"] = error.Details;
        }

        return error.Code switch
        {
            Error.VALIDATION_ERROR => BadRequest(problemDetails),
            Error.CONFLICT => Conflict(problemDetails),
            Error.NOT_FOUND => NotFound(problemDetails),
            Error.UNAUTHORIZED => Unauthorized(problemDetails),
            _ => BadRequest(problemDetails)
        };
    }

    /// <summary>
    /// Obtém o título amigável para um código de erro
    /// </summary>
    /// <param name="errorCode">Código do erro</param>
    /// <returns>Título descritivo do erro</returns>
    private static string GetTitleForErrorCode(string errorCode) => errorCode switch
    {
        Error.VALIDATION_ERROR => "Erro de Validação",
        Error.CONFLICT => "Conflito",
        Error.NOT_FOUND => "Não Encontrado",
        Error.UNAUTHORIZED => "Não Autorizado",
        _ => "Erro"
    };
}
```

### Uso em Controllers Específicos

```csharp
// API/Controllers/UsuariosController.cs
namespace <SolutionName>.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public sealed class UsuariosController : BaseController
{
    private readonly CriarUsuarioHandler _criar;
    private readonly ObterUsuarioPorIdHandler _obterPorId;
    private readonly ILogger<UsuariosController> _logger;

    public UsuariosController(
        CriarUsuarioHandler criar,
        ObterUsuarioPorIdHandler obterPorId,
        ILogger<UsuariosController> logger)
    {
        _criar = criar;
        _obterPorId = obterPorId;
        _logger = logger;
    }

    [HttpPost]
    [ProducesResponseType(typeof(UsuarioResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status409Conflict)]
    public async Task<IActionResult> Criar(
        [FromBody] CriarUsuarioRequest request,
        CancellationToken ct)
    {
        var result = await _criar.HandleAsync(request, ct);

        return result.Match(
            onSuccess: r => CreatedAtAction(nameof(ObterPorId), new { id = r.Id }, r),
            onFailure: error => CreateProblemResult(error));
    }

    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(UsuarioResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> ObterPorId(Guid id, CancellationToken ct)
    {
        var result = await _obterPorId.HandleAsync(new ObterUsuarioPorIdRequest(id), ct);

        return result.Match(
            onSuccess: Ok,
            onFailure: error => CreateProblemResult(error));
    }
}
```

### Vantagens do BaseController

- ✅ **Reutilização**: Método `CreateProblemResult` disponível em todos os controllers
- ✅ **Manutenção**: Alterar comportamento de erro em um único lugar
- ✅ **Consistência**: Todos os endpoints retornam erros no mesmo formato (ProblemDetails)
- ✅ **Extensibilidade**: Fácil adicionar novos métodos comuns (ex: autenticação, logging)
- ✅ **Clean Code**: Controllers mais enxutos focados em orquestração
- ✅ **Padronização**: Mapeamento inteligente de códigos de erro para status HTTP

---

## Value Object Pattern

```csharp
// Domain/ValueObjects/Email.cs
namespace <SolutionName>.Domain.ValueObjects;

public sealed class Email : ValueObject
{
    public string Valor { get; }

    private Email(string valor) => Valor = valor;

    public static Email Criar(string valor)
    {
        ValidarEntidade(valor);

        return new Email(valor.Trim());
    }

    private static void ValidarEntidade(string valor)
    {
        DomainException.ThrowIfNullOrWhiteSpace(valor, "Email é obrigatório");
        DomainException.ThrowIf(!IsValido(valor), "Email inválido");
    }

    private static bool IsValido(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Valor;
    }

    public override string ToString() => Valor;
}

// Domain/Common/ValueObject.cs
public abstract class ValueObject : IEquatable<ValueObject>
{
    protected abstract IEnumerable<object> GetEqualityComponents();

    public override bool Equals(object? obj)
    {
        if (obj == null || obj.GetType() != GetType())
            return false;

        var valueObject = (ValueObject)obj;
        return GetEqualityComponents().SequenceEqual(valueObject.GetEqualityComponents());
    }

    public override int GetHashCode()
        => GetEqualityComponents().Aggregate(1, (current, obj) => 
            unchecked(current * 23 + obj?.GetHashCode() ?? 0));

    public bool Equals(ValueObject? other) => Equals((object?)other);
    public static bool operator ==(ValueObject? left, ValueObject? right) => Equals(left, right);
    public static bool operator !=(ValueObject? left, ValueObject? right) => !Equals(left, right);
}
```

---

**Próxima Seção**: Ver [`03-conventions.md`](./03-conventions.md) para nomenclatura.

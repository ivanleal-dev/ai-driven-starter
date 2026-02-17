# 🔄 Migração de Padrões Legados

> 📋 **Stack Atual**: [../../09-stack.md](../../09-stack.md)

## 📋 Padrões Antigos → Novos

Se herdar código legacy, migrar para os padrões modernos desta arquitetura.

---

## 1️⃣ Specification Pattern → Clean Handler Pattern

### ❌ Padrão Legado: Specification

```csharp
// OLD: Infrastructure/Specifications/UsuarioSpecification.cs
public class UsuarioSpecification : Specification<Usuario>
{
    public UsuarioSpecification(string email)
    {
        AddCriteria(u => u.Email == email);
        AddOrderBy(u => u.DataCriacao);
    }
}

// OLD: Uso
var spec = new UsuarioSpecification("email@example.com");
var usuarios = await _repository.GetAsync(spec);
```

### ✅ Padrão Novo: Query Direto no Repositório

```csharp
// NEW: Domain/Interfaces/IUsuarioRepository.cs
public interface IUsuarioRepository : IRepository<Usuario>
{
    Task<Usuario?> ObterPorEmailAsync(string email, CancellationToken ct = default);
    Task<IReadOnlyCollection<Usuario>> ObterPorStatusAsync(bool ativo, CancellationToken ct = default);
}

// NEW: Infrastructure/Repositories/UsuarioRepository.cs
public class UsuarioRepository : Repository<Usuario>, IUsuarioRepository
{
    public async Task<Usuario?> ObterPorEmailAsync(string email, CancellationToken ct = default)
        => await Context.Usuarios.AsNoTracking()
            .FirstOrDefaultAsync(u => u.Email == email, ct);

    public async Task<IReadOnlyCollection<Usuario>> ObterPorStatusAsync(bool ativo, CancellationToken ct = default)
        => await Context.Usuarios.AsNoTracking()
            .Where(u => u.Ativo == ativo)
            .OrderBy(u => u.DataCriacao)
            .ToListAsync(ct);
}

// NEW: Uso
var usuario = await _repository.ObterPorEmailAsync("email@example.com", ct);
```

**Por quê?**
- ✅ Mais direto e compreensível
- ✅ Menos abstrações intermediárias
- ✅ Fácil de debugar
- ✅ Padrão moderno em ASP.NET Core

---

## 2️⃣ Generic Service Class → Vertical Slice Handlers

### ❌ Padrão Legado: Serviço Genérico

```csharp
// OLD: Application/Services/UsuarioService.cs
public class UsuarioService
{
    private readonly IUsuarioRepository _repo;
    private readonly IPasswordHasher _hasher;

    public async Task<UsuarioDTO> CriarAsync(CriarUsuarioDTO dto)
    {
        var usuario = new Usuario { Email = dto.Email };
        // ... lógica
        await _repo.AddAsync(usuario);
        return MapToDTOAsync(usuario);
    }

    public async Task<UsuarioDTO> AtualizarAsync(Guid id, AtualizarUsuarioDTO dto)
    {
        var usuario = await _repo.GetByIdAsync(id);
        // ... lógica
        return MapToDTOAsync(usuario);
    }

    public async Task DeletarAsync(Guid id)
    {
        var usuario = await _repo.GetByIdAsync(id);
        await _repo.DeleteAsync(usuario);
    }

    // ... 20+ métodos misturados
}

// OLD: Injeção
public class UsuariosController(UsuarioService service)
{
    [HttpPost]
    public async Task<IActionResult> Post(CriarUsuarioDTO dto)
        => Ok(await service.CriarAsync(dto));
}
```

### ✅ Padrão Novo: Vertical Slice Handlers

```csharp
// NEW: Application/UseCases/Usuario/CriarUsuario/CriarUsuarioHandler.cs
public sealed class CriarUsuarioHandler(
    IUsuarioRepository repository,
    IPasswordHasher passwordHasher,
    IUnitOfWork unitOfWork,
    IValidator<CriarUsuarioRequest> validator)
{
    public async Task<Result<UsuarioResponse>> HandleAsync(
        CriarUsuarioRequest request, CancellationToken ct)
    {
        var validation = await validator.ValidateAsync(request, ct);
        if (!validation.IsValid)
            return Result<UsuarioResponse>.Failure(validation.ToValidationError());

        var usuario = Usuario.Criar(request.Email, passwordHasher.Hash(request.Senha));
        await repository.AdicionarAsync(usuario, ct);
        await unitOfWork.CommitAsync(ct);

        return Result<UsuarioResponse>.Success(new UsuarioResponse { Id = usuario.Id });
    }
}

// NEW: Application/UseCases/Usuario/AtualizarUsuario/AtualizarUsuarioHandler.cs
public sealed class AtualizarUsuarioHandler(
    IUsuarioRepository repository,
    IUnitOfWork unitOfWork,
    IValidator<AtualizarUsuarioRequest> validator)
{
    public async Task<Result<UsuarioResponse>> HandleAsync(
        AtualizarUsuarioRequest request, CancellationToken ct)
    {
        // Mesma estrutura, handler isolado
    }
}

// NEW: Controllers/UsuariosController.cs
[ApiController]
[Route("api/[controller]")]
public sealed class UsuariosController(
    CriarUsuarioHandler criar,
    AtualizarUsuarioHandler atualizar,
    ExcluirUsuarioHandler excluir)
{
    [HttpPost]
    public async Task<IActionResult> Criar([FromBody] CriarUsuarioRequest request, CancellationToken ct)
        => (await criar.HandleAsync(request, ct))
            .Match(Ok, e => BadRequest(e));

    [HttpPut("{id:guid}")]
    public async Task<IActionResult> Atualizar(Guid id, [FromBody] AtualizarUsuarioRequest request, CancellationToken ct)
        => (await atualizar.HandleAsync(request with { Id = id }, ct))
            .Match(Ok, e => BadRequest(e));
}
```

**Benefícios**:
- ✅ Um handler por caso de uso (Single Responsibility)
- ✅ Fácil testar em isolamento
- ✅ Sem side effects entre operações
- ✅ Compreensão clara do que cada handler faz

---

## 3️⃣ DTO Mapping → Record Requests/Responses

### ❌ Padrão Legado: DTO com AutoMapper

```csharp
// OLD: DTOs
public class UsuarioDTO
{
    public Guid Id { get; set; }
    public string Email { get; set; }
    public string Nome { get; set; }
}

public class CriarUsuarioDTO
{
    public string Email { get; set; }
    public string Senha { get; set; }
}

// OLD: MappingProfile
public class UsuarioMappingProfile : Profile
{
    public UsuarioMappingProfile()
    {
        CreateMap<Usuario, UsuarioDTO>();
        CreateMap<CriarUsuarioDTO, Usuario>();
    }
}

// OLD: Uso
var usuario = await _repository.GetByIdAsync(id);
var dto = _mapper.Map<UsuarioDTO>(usuario);
```

### ✅ Padrão Novo: Records Tipados

```csharp
// NEW: Request DTOs
public sealed record CriarUsuarioRequest
{
    public required string Email { get; init; }
    public required string Senha { get; init; }
}

public sealed record AtualizarUsuarioRequest
{
    public required Guid Id { get; init; }
    public required string Email { get; init; }
}

// NEW: Response DTOs
public sealed record UsuarioResponse
{
    public required Guid Id { get; init; }
    public required string Email { get; init; }
}

// NEW: Mapping (manual, sem AutoMapper)
private static UsuarioResponse MapFromUsuario(Usuario usuario)
    => new UsuarioResponse
    {
        Id = usuario.Id,
        Email = usuario.Email
    };

// NEW: Uso (tipo-seguro)
var usuario = await _repository.ObterPorIdAsync(id, ct);
var response = MapFromUsuario(usuario);
```

**Por quê remover AutoMapper?**
- ✅ Sem configuração complexa de reflection
- ✅ Mappings explícitos e claros
- ✅ Mais rápido (sem reflection runtime)
- ✅ Fácil de debugar

---

## 4️⃣ Exception Handling em Handlers → Result Pattern

### ❌ Padrão Legado: Exceções

```csharp
// OLD: Handler
public async Task<UsuarioDTO> CreateUserAsync(CreateUserDTO dto)
{
    try
    {
        if (string.IsNullOrEmpty(dto.Email))
            throw new ArgumentNullException(nameof(dto.Email));

        var usuario = new Usuario { Email = dto.Email };
        await _repo.AddAsync(usuario);
        await _unitOfWork.SaveAsync();
        return _mapper.Map<UsuarioDTO>(usuario);
    }
    catch (DbUpdateException ex)
    {
        throw new ApplicationException("Erro ao criar usuário", ex);
    }
}

// OLD: Controller (trata exceção globalmente)
[HttpPost]
public async Task<IActionResult> Post(CreateUserDTO dto)
{
    try
    {
        var result = await _service.CreateUserAsync(dto);
        return Ok(result);
    }
    catch (Exception ex)
    {
        return StatusCode(500, new { error = ex.Message });
    }
}
```

### ✅ Padrão Novo: Result Pattern

```csharp
// NEW: Handler
public async Task<Result<UsuarioResponse>> HandleAsync(
    CriarUsuarioRequest request, CancellationToken ct)
{
    // 1. Validar
    var validation = await _validator.ValidateAsync(request, ct);
    if (!validation.IsValid)
        return Result<UsuarioResponse>.Failure(validation.ToValidationError());

    try
    {
        // 2. Criar
        var usuario = Usuario.Criar(request.Email, _hasher.Hash(request.Senha));

        // 3. Persistir
        await _repository.AdicionarAsync(usuario, ct);
        await _unitOfWork.CommitAsync(ct);

        // 4. Retornar sucesso
        return Result<UsuarioResponse>.Success(new UsuarioResponse
        {
            Id = usuario.Id,
            Email = usuario.Email
        });
    }
    catch (DomainException ex)
    {
        return Result<UsuarioResponse>.Failure(Error.Conflict(ex.Message));
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Erro inesperado");
        return Result<UsuarioResponse>.Failure(
            Error.InternalServerError("Erro ao processar solicitação"));
    }
}

// NEW: Controller (limpo)
[HttpPost]
public async Task<IActionResult> Criar(
    [FromBody] CriarUsuarioRequest request, CancellationToken ct)
{
    var result = await _criar.HandleAsync(request, ct);
    return result.Match(
        success: r => CreatedAtAction(nameof(ObterPorId), new { id = r.Id }, r),
        failure: e => HandleError(e));
}

private IActionResult HandleError(Error error) => error.Code switch
{
    Error.VALIDATION_ERROR => BadRequest(error),
    Error.CONFLICT => Conflict(error),
    _ => StatusCode(500, error)
};
```

**Benefícios**:
- ✅ Sem try-catch em todos os lugares
- ✅ Erros esperados vs inesperados claros
- ✅ Controllers simples e sem lógica
- ✅ Testes sem mocking de exceções

---

## 5️⃣ Repository.AddAsync/SaveAsync → DbContext Direto

### ❌ Padrão Legado: Múltiplas Operações

```csharp
// OLD: Cada repositório tem Save
var usuario = new Usuario { Email = "user@example.com" };
await _usuarioRepository.AddAsync(usuario);
await _usuarioRepository.SaveAsync();  // Save por repositório

var pedido = new Pedido { UsuarioId = usuario.Id };
await _pedidoRepository.AddAsync(pedido);
await _pedidoRepository.SaveAsync();  // Save separado

// PROBLEMA: 2 transações, não 1 atômica
```

### ❌ Anti-Pattern: IUnitOfWork na Application

```csharp
// WRONG: UnitOfWork viola Clean Architecture
namespace MyApp.Application.Common.Interfaces;

public interface IUnitOfWork  // ← Não deve existir na Application!
{
    Task<int> CommitAsync(CancellationToken ct = default);
}

// No Handler:
await _repository.AdicionarAsync(entity, ct);
await _unitOfWork.CommitAsync(ct);  // ← Application conhece persistência
```

### ✅ Padrão Novo: DbContext nos Handlers

**Para operações simples:**
```csharp
// Repository já persiste automaticamente
var usuario = Usuario.Criar("user@example.com", hashedPassword);
await _usuarioRepository.AdicionarAsync(usuario, ct); // Já chama SaveChanges
```

**Para transações complexas:**
```csharp
// Handler gerencia transação com DbContext
using var transaction = await _context.Database.BeginTransactionAsync(ct);

try 
{
    var usuario = Usuario.Criar("user@example.com", hashedPassword); 
    await _usuarioRepository.AdicionarSemCommitAsync(usuario, ct);

    var pedido = Pedido.Criar(usuario.Id, itens);
    await _pedidoRepository.AdicionarSemCommitAsync(pedido, ct);

    await _context.SaveChangesAsync(ct);
    await transaction.CommitAsync(ct);
}
catch 
{
    await transaction.RollbackAsync(ct);
    throw;
}
```

---

## 6️⃣ MediatR Commands/Queries → Direct Handler Injection

### ❌ Padrão Legado: MediatR

```csharp
// OLD: Commands
public record CreateUserCommand : IRequest<UserDTO>
{
    public string Email { get; init; }
}

public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, UserDTO>
{
    public async Task<UserDTO> Handle(CreateUserCommand request, CancellationToken ct)
    {
        // ... lógica
    }
}

// OLD: Controller
public class UsersController(IMediator mediator)
{
    [HttpPost]
    public async Task<IActionResult> Post(CreateUserCommand command)
        => Ok(await mediator.Send(command));  // Indireção
}
```

### ✅ Padrão Novo: Injeção Direta

```csharp
// NEW: Handler (sem interface MediatR)
public sealed class CriarUsuarioHandler
{
    public async Task<Result<UsuarioResponse>> HandleAsync(
        CriarUsuarioRequest request, CancellationToken ct)
    {
        // ... lógica
    }
}

// NEW: Controller (sem indireção)
public sealed class UsuariosController(CriarUsuarioHandler criar)
{
    [HttpPost]
    public async Task<IActionResult> Post(CriarUsuarioRequest request, CancellationToken ct)
        => Ok(await criar.HandleAsync(request, ct));  // Direto
}

// NEW: Registrar em DI
services.AddScoped<CriarUsuarioHandler>();
services.AddScoped<AtualizarUsuarioHandler>();
services.AddScoped<ExcluirUsuarioHandler>();
```

**Por quê remover MediatR?**
- ✅ Menos camadas de indireção
- ✅ Rastreamento mais fácil no debugger
- ✅ Injeção de dependência nativa do .NET
- ✅ Menos configuração e "magia"

---

## ✅ Checklist de Migração

Se encontrar código legado:

- [ ] ❌ Specifications → ✅ Query Direto no Repositório
- [ ] ❌ Serviços Genéricos → ✅ Handlers por Caso de Uso
- [ ] ❌ AutoMapper → ✅ Records Tipadas + Mapping Manual
- [ ] ❌ Exceções como Fluxo → ✅ Result Pattern  
- [ ] ❌ SaveAsync por Repositório → ✅ DbContext nos Handlers
- [ ] ❌ IUnitOfWork na Application → ✅ Transações com DbContext
- [ ] ❌ MediatR → ✅ Injeção Direta de Handlers
- [ ] ❌ Generics em Tudo → ✅ Métodos Específicos por Caso de Uso

---

**Fim da Documentação Especializada**

Voltar ao [`copilot-instructions.md`](../copilot-instructions.md) para índice geral.

# 🗄️ Entity Framework Core & Banco de Dados

> 📋 **Configurações de Banco & Dependências**: [09-stack.md](./09-stack.md)

## 📚 Setup Inicial

### DbContext (AppDbContext.cs)

```csharp
// Infrastructure/Data/AppDbContext.cs
namespace <SolutionName>.Infrastructure.Data;

public sealed class AppDbContext : DbContext
{
    public required DbSet<Usuario> Usuarios { get; init; }
    public required DbSet<Pedido> Pedidos { get; init; }
    public required DbSet<FailedLoginAttempts> TentativasLoginFalhadas { get; init; }

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Aplicar todas as configurações
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

### Connection String (appsettings.json)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "User ID=postgres;Password=postgres;Server=localhost;Port=5432;Database=agente_viagem;Integrated Security=false;"
  }
}
```

### DI Registration (Program.cs)

```csharp
// API/Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"))
);

// Repositórios
builder.Services.AddScoped<IUsuarioRepository, UsuarioRepository>();
builder.Services.AddScoped<ITodoRepository, TodoRepository>();
```

---

## 🏗️ Entity Configurations (EF Core)

**Padrão**: Uma classe `{Entity}Configuration` por entidade.

### Exemplo: Usuario Configuration

```csharp
// Infrastructure/Data/Configurations/UsuarioConfiguration.cs
namespace <SolutionName>.Infrastructure.Data.Configurations;

public sealed class UsuarioConfiguration : IEntityTypeConfiguration<Usuario>
{
    public void Configure(EntityTypeBuilder<Usuario> builder)
    {
        // Tabela
        builder.ToTable("usuarios");

        // Primary Key
        builder.HasKey(u => u.Id);

        // Propriedades
        builder.Property(u => u.Email)
            .HasColumnName("email")
            .HasMaxLength(255)
            .IsRequired();

        builder.Property(u => u.SenhaHash)
            .HasColumnName("senha_hash")
            .HasMaxLength(255)
            .IsRequired();

        builder.Property(u => u.Ativo)
            .HasColumnName("ativo")
            .HasDefaultValue(true)
            .IsRequired();

        builder.Property(u => u.DataCriacao)
            .HasColumnName("data_criacao")
            .IsRequired();

        builder.Property(u => u.DataUltimoLogin)
            .HasColumnName("data_ultimo_login")
            .IsRequired(false);

        // Índices
        builder.HasIndex(u => u.Email)
            .IsUnique()
            .HasDatabaseName("ix_usuarios_email_unique");

        // Relacionamentos
        builder.HasMany(u => u.TentativasLoginFalhadas)
            .WithOne()
            .HasForeignKey("usuario_id")
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

### Exemplo: FailedLoginAttempts Configuration

```csharp
// Infrastructure/Data/Configurations/FailedLoginAttemptsConfiguration.cs
namespace <SolutionName>.Infrastructure.Data.Configurations;

public sealed class FailedLoginAttemptsConfiguration 
    : IEntityTypeConfiguration<FailedLoginAttempts>
{
    public void Configure(EntityTypeBuilder<FailedLoginAttempts> builder)
    {
        builder.ToTable("tentativas_login_falhadas");

        builder.HasKey(x => x.Id);

        builder.Property(x => x.UsuarioId)
            .HasColumnName("usuario_id")
            .IsRequired();

        builder.Property(x => x.TentativasFalhadas)
            .HasColumnName("tentativas_falhadas")
            .HasDefaultValue(0)
            .IsRequired();

        builder.Property(x => x.DataUltimaTentativa)
            .HasColumnName("data_ultima_tentativa")
            .IsRequired(false);

        builder.Property(x => x.BloqueadoAte)
            .HasColumnName("bloqueado_ate")
            .IsRequired(false);

        // Índice único para rápida lookup
        builder.HasIndex(x => x.UsuarioId)
            .IsUnique()
            .HasDatabaseName("ix_tentativas_login_usuario_id_unique");

        // FK
        builder.HasOne<Usuario>()
            .WithMany(u => u.TentativasLoginFalhadas)
            .HasForeignKey(x => x.UsuarioId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

---

## 📊 Migrations

### Criar Migration

```powershell
# No diretório do projeto Infrastructure
cd src/backend/{{ProjectBase}}.Infrastructure

# Criar nova migration
dotnet ef migrations add {NomeMigration} \
    --project {{ProjectBase}}.Infrastructure.csproj \
    --startup-project ..\{{ProjectBase}}.API\{{ProjectBase}}.API.csproj \
    --context AppDbContext

# Exemplo:
dotnet ef migrations add Initial_Setup \
    --project {{ProjectBase}}.Infrastructure.csproj \
    --startup-project ..\{{ProjectBase}}.API\{{ProjectBase}}.API.csproj
```

### Aplicar Migration

```powershell
# Update database
dotnet ef database update \
    --project {{ProjectBase}}.Infrastructure.csproj \
    --startup-project ..\{{ProjectBase}}.API\{{ProjectBase}}.API.csproj

# Com connection string específica
dotnet ef database update \
    --project {{ProjectBase}}.Infrastructure.csproj \
    --startup-project ..\{{ProjectBase}}.API\{{ProjectBase}}.API.csproj \
    --connection "User ID=postgres;Password=postgres;Server=localhost;Port=5432;Database=agente_viagem;Integrated Security=false;"
```

### Migration Best Practices

```csharp
// ✅ Correto: Usar AddColumn, CreateIndex, etc
public override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<DateTimeOffset>(
        name: "data_ultimo_login",
        table: "usuarios",
        type: "timestamp with time zone",
        nullable: true);

    migrationBuilder.CreateIndex(
        name: "ix_usuarios_email_unique",
        table: "usuarios",
        column: "email",
        unique: true);
}

public override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropIndex(
        name: "ix_usuarios_email_unique",
        table: "usuarios");

    migrationBuilder.DropColumn(
        name: "data_ultimo_login",
        table: "usuarios");
}

// ❌ Errado: Executar SQL raw diretamente
public override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("ALTER TABLE usuarios ADD COLUMN data_ultimo_login TIMESTAMPTZ");
}
```

---

## � Gerenciamento de Transações

### ✅ Abordagem Recomendada: DbContext Compartilhado

**Para operações simples (um repositório):**
```csharp
public class CriarUsuarioHandler
{
    private readonly IUsuarioRepository _usuarioRepository;
    
    public async Task<Result<UsuarioResponse>> HandleAsync(CriarUsuarioRequest request, CancellationToken cancellationToken)
    {
        var usuario = Usuario.Criar(request.Email, request.SenhaHash);
        await _usuarioRepository.AdicionarAsync(usuario, cancellationToken);
        // Repository já chama SaveChangesAsync internamente
        return Result<UsuarioResponse>.Success(new UsuarioResponse { Id = usuario.Id, Email = usuario.Email });
    }
}
```

**Para transações complexas (múltiplos repositórios):**
```csharp
public class CriarUsuarioComTodosHandler
{
    private readonly IUsuarioRepository _usuarioRepository;
    private readonly ITodoRepository _todoRepository;
    private readonly AppDbContext _context; // Injetar DbContext diretamente
    
    public CriarUsuarioComTodosHandler(IUsuarioRepository usuarioRepository, ITodoRepository todoRepository, AppDbContext context)
    {
        _usuarioRepository = usuarioRepository;
        _todoRepository = todoRepository;
        _context = context;
    }
    
    public async Task<Result<UsuarioResponse>> HandleAsync(CriarUsuarioComTodosRequest request, CancellationToken cancellationToken)
    {
        using var transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
        
        try
        {
            // 1. Operações nos repositórios (sem SaveChanges)
            var usuario = Usuario.Criar(request.Email, request.SenhaHash);
            await _usuarioRepository.AdicionarSemCommitAsync(usuario, cancellationToken);
            
            foreach (var todoRequest in request.TodosIniciais)
            {
                var todo = Todo.Criar(todoRequest.Titulo, usuario.Id);
                await _todoRepository.AdicionarSemCommitAsync(todo, cancellationToken);
            }
            
            // 2. Commit da transação completa
            await _context.SaveChangesAsync(cancellationToken);
            await transaction.CommitAsync(cancellationToken);
            
            return Result<UsuarioResponse>.Success(new UsuarioResponse { Id = usuario.Id, Email = usuario.Email });
        }
        catch (Exception)
        {
            await transaction.RollbackAsync(cancellationToken);
            throw;
        }
    }
}
```

### ❌ Anti-Pattern: IUnitOfWork na Application
```csharp
// ❌ NÃO FAZER - IUnitOfWork viola Clean Architecture
namespace <SolutionName>.Application.Common.Interfaces;

public interface IUnitOfWork // ← Esta interface não deve existir na Application
{
    Task<int> CommitAsync(CancellationToken cancellationToken = default);
}
```

**Problemas:**
- Application não deve conhecer detalhes de persistência
- EF Core já gerencia transações automaticamente
- Adiciona complexidade desnecessária
- Viola princípios da Clean Architecture

---

## 🔄 Repositories Pattern

### Repository Base (Generic)

```csharp
// Infrastructure/Repositories/Repository.cs
namespace <SolutionName>.Infrastructure.Repositories;

public abstract class Repository<T> : IRepository<T> where T : BaseEntity
{
    protected readonly AppDbContext Context;

    protected Repository(AppDbContext context) => Context = context;

    public virtual async Task<T?> ObterPorIdAsync(Guid id, CancellationToken ct = default)
        => await Context.Set<T>().AsNoTracking()
            .FirstOrDefaultAsync(e => e.Id == id, ct);

    public virtual async Task<(IReadOnlyCollection<T> Itens, int Total)> ListarPaginadoAsync(
        int numeroPagina, int tamanhoPagina, CancellationToken ct = default)
    {
        var query = Context.Set<T>().AsNoTracking();
        var total = await query.CountAsync(ct);

        var itens = await query
            .OrderBy(e => e.Id)  // Ordenação padrão (sobrescrever em subclasses)
            .Skip((numeroPagina - 1) * tamanhoPagina)
            .Take(tamanhoPagina)
            .ToListAsync(ct);

        return (itens, total);
    }

    // Para operações simples (automaticamente persiste)
    public virtual async Task AdicionarAsync(T entity, CancellationToken ct = default)
    {
        await Context.Set<T>().AddAsync(entity, ct);
        await Context.SaveChangesAsync(ct); // EF gerencia transação automaticamente
    }

    public virtual async Task AtualizarAsync(T entity, CancellationToken ct = default)
    {
        Context.Set<T>().Update(entity);
        await Context.SaveChangesAsync(ct);
    }

    public virtual async Task RemoverAsync(T entity, CancellationToken ct = default)
    {
        Context.Set<T>().Remove(entity);
        await Context.SaveChangesAsync(ct);
    }
    
    // Para transações complexas (sem commit - deixa para o Handler gerenciar)  
    public virtual async Task AdicionarSemCommitAsync(T entity, CancellationToken ct = default)
        => await Context.Set<T>().AddAsync(entity, ct);

    public virtual void AtualizarSemCommit(T entity)
        => Context.Set<T>().Update(entity);

    public virtual void RemoverSemCommit(T entity)
        => Context.Set<T>().Remove(entity);
}
```

### Specialized Repository Example

```csharp
// Infrastructure/Repositories/UsuarioRepository.cs
namespace <SolutionName>.Infrastructure.Repositories;

public sealed class UsuarioRepository : Repository<Usuario>, IUsuarioRepository
{
    public UsuarioRepository(AppDbContext context) : base(context) { }

    public async Task<bool> EhEmailUnicoAsync(string email, Guid? excluirId = null, CancellationToken ct = default)
    {
        var query = Context.Usuarios.AsNoTracking().Where(u => u.Email == email);
        if (excluirId.HasValue) query = query.Where(u => u.Id != excluirId.Value);
        return !await query.AnyAsync(ct);
    }

    public async Task<Usuario?> ObterPorEmailAsync(string email, CancellationToken ct = default)
        => await Context.Usuarios.AsNoTracking()
            .FirstOrDefaultAsync(u => u.Email == email, ct);

    public override async Task<(IReadOnlyCollection<Usuario> Itens, int Total)> ListarPaginadoAsync(
        int numeroPagina, int tamanhoPagina, CancellationToken ct = default)
    {
        var query = Context.Usuarios.AsNoTracking();
        var total = await query.CountAsync(ct);

        var itens = await query
            .OrderBy(u => u.Email)  // Ordenar por email
            .Skip((numeroPagina - 1) * tamanhoPagina)
            .Take(tamanhoPagina)
            .ToListAsync(ct);

        return (itens, total);
    }

    // Exemplo: método que precisa de transação complexa
    public async Task AdicionarComValidacoesAsync(Usuario usuario, bool validarEmailUnico = true, CancellationToken ct = default)
    {
        if (validarEmailUnico && !await EhEmailUnicoAsync(usuario.Email, ct: ct))
            throw new DomainException("Email já está em uso");

        // Em transação complexa, use o método SemCommit
        await AdicionarSemCommitAsync(usuario, ct);
    }
}
```

---

## 📄 Paginação

### Result Pagination

```csharp
// Application/Common/Pagination/ResultadoPaginado.cs
namespace <SolutionName>.Application.Common.Pagination;

public sealed record ResultadoPaginado<T>
{
    public required IReadOnlyCollection<T> Itens { get; init; }
    public required int NumeroPagina { get; init; }
    public required int TamanhoPagina { get; init; }
    public required int Total { get; init; }

    public int TotalPaginas => (Total + TamanhoPagina - 1) / TamanhoPagina;
    public bool TemProxima => NumeroPagina < TotalPaginas;
    public bool TemAnterior => NumeroPagina > 1;
}

// Request
public sealed record ObterUsuarioPaginadoRequest
{
    public int NumeroPagina { get; init; } = 1;
    public int TamanhoPagina { get; init; } = 10;
}

// Handler
public async Task<Result<ResultadoPaginado<UsuarioResponse>>> HandleAsync(
    ObterUsuarioPaginadoRequest request, CancellationToken ct)
{
    if (request.NumeroPagina < 1 || request.TamanhoPagina < 1 || request.TamanhoPagina > 100)
        return Result<ResultadoPaginado<UsuarioResponse>>.Failure(
            Error.Validation("Parâmetros de paginação inválidos"));

    var (itens, total) = await _repository.ListarPaginadoAsync(
        request.NumeroPagina, request.TamanhoPagina, ct);

    return Result<ResultadoPaginado<UsuarioResponse>>.Success(new()
    {
        Itens = itens.Select(u => new UsuarioResponse { Id = u.Id, Email = u.Email }).ToList(),
        NumeroPagina = request.NumeroPagina,
        TamanhoPagina = request.TamanhoPagina,
        Total = total
    });
}
```

---

## ✅ Checklist de Database

- [ ] DbContext criado com DbSet para cada entidade
- [ ] Todas as entidades têm `IEntityTypeConfiguration`
- [ ] Configurações usam nomes de colunas em português
- [ ] Migrations criadas e testadas
- [ ] Índices criados para lookups frequentes
- [ ] ForeignKeys configuradas com DeleteBehavior apropriado
- [ ] Repositories herdam de Repository<T> base
- [ ] Handlers usam DbContext diretamente para transações complexas
- [ ] Paginação usando ResultadoPaginado<T>
- [ ] Connection string em appsettings.Development.json

---

### 2. Atualizar Handlers
```diff
  public class CriarUsuarioHandler
  {
      private readonly IUsuarioRepository _repository;

      public CriarUsuarioHandler(
+         IUsuarioRepository repository)
      {
          _repository = repository;
      }

      public async Task<Result<UsuarioDto>> HandleAsync(...)
      {
          var usuario = Usuario.Criar(...);
          await _repository.AdicionarAsync(usuario, ct);
          return Result.Success(...);
      }
  }
```

### 3. Atualizar DI Registration
```diff
  // API/Program.cs
  // Repositórios já registrados individualmente
```

### 4. Atualizar Testes
```diff
  public class CriarUsuarioHandlerTests
  {
      private readonly Mock<IUsuarioRepository> _repositoryMock = new();

      private CriarUsuarioHandler CreateHandler()
+         => new(_repositoryMock.Object);
  }
```

---

**Próxima Seção**: Ver [`06-testing.md`](./06-testing.md) para testes unitários.

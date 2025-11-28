# 🗄️ Entity Framework Core & Banco de Dados

## 📚 Setup Inicial

### DbContext (AppDbContext.cs)

```csharp
// Infrastructure/Data/AppDbContext.cs
namespace AgenteViagem.Infrastructure.Data;

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

builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

---

## 🏗️ Entity Configurations (EF Core)

**Padrão**: Uma classe `{Entity}Configuration` por entidade.

### Exemplo: Usuario Configuration

```csharp
// Infrastructure/Data/Configurations/UsuarioConfiguration.cs
namespace AgenteViagem.Infrastructure.Data.Configurations;

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
namespace AgenteViagem.Infrastructure.Data.Configurations;

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
cd src/backend/AgenteViagem.Infrastructure

# Criar nova migration
dotnet ef migrations add {NomeMigration} \
    --project AgenteViagem.Infrastructure.csproj \
    --startup-project ..\AgenteViagem.API\AgenteViagem.API.csproj \
    --context AppDbContext

# Exemplo:
dotnet ef migrations add Initial_Setup \
    --project AgenteViagem.Infrastructure.csproj \
    --startup-project ..\AgenteViagem.API\AgenteViagem.API.csproj
```

### Aplicar Migration

```powershell
# Update database
dotnet ef database update \
    --project AgenteViagem.Infrastructure.csproj \
    --startup-project ..\AgenteViagem.API\AgenteViagem.API.csproj

# Com connection string específica
dotnet ef database update \
    --project AgenteViagem.Infrastructure.csproj \
    --startup-project ..\AgenteViagem.API\AgenteViagem.API.csproj \
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

## 📖 Unit of Work Pattern

### Interface (Domain)

```csharp
// Domain/Interfaces/IUnitOfWork.cs
namespace AgenteViagem.Domain.Interfaces;

public interface IUnitOfWork : IAsyncDisposable
{
    Task<int> CommitAsync(CancellationToken cancellationToken = default);
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
}
```

### Implementação (Infrastructure)

```csharp
// Infrastructure/Data/UnitOfWork.cs
namespace AgenteViagem.Infrastructure.Data;

public sealed class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    private readonly ILogger<UnitOfWork> _logger;

    public UnitOfWork(AppDbContext context, ILogger<UnitOfWork> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<int> CommitAsync(CancellationToken cancellationToken = default)
    {
        try
        {
            var resultado = await _context.SaveChangesAsync(cancellationToken);
            _logger.LogInformation("Mudanças persistidas: {RowsAffected}", resultado);
            return resultado;
        }
        catch (DbUpdateException ex)
        {
            _logger.LogError(ex, "Erro ao persistir mudanças");
            throw;
        }
    }

    public async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
        => await _context.SaveChangesAsync(cancellationToken);

    public async ValueTask DisposeAsync()
        => await _context.DisposeAsync();
}
```

---

## 🔄 Repositories Pattern

### Repository Base (Generic)

```csharp
// Infrastructure/Repositories/Repository.cs
namespace AgenteViagem.Infrastructure.Repositories;

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

    public virtual async Task AdicionarAsync(T entity, CancellationToken ct = default)
        => await Context.Set<T>().AddAsync(entity, ct);

    public virtual void Atualizar(T entity)
        => Context.Set<T>().Update(entity);

    public virtual void Remover(T entity)
        => Context.Set<T>().Remove(entity);
}
```

### Specialized Repository Example

```csharp
// Infrastructure/Repositories/UsuarioRepository.cs
namespace AgenteViagem.Infrastructure.Repositories;

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
}
```

---

## 📄 Paginação

### Result Pagination

```csharp
// Application/Common/Pagination/ResultadoPaginado.cs
namespace AgenteViagem.Application.Common.Pagination;

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
- [ ] UnitOfWork implementado e registrado em DI
- [ ] Repositories herdam de Repository<T> base
- [ ] Paginação usando ResultadoPaginado<T>
- [ ] Connection string em appsettings.Development.json

---

**Próxima Seção**: Ver [`06-testing.md`](./06-testing.md) para testes unitários.

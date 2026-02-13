# 🧪 Testing - Domain & Application

## 📋 Escopo

### ✅ O que Testar

1. **Domain Layer** (100% coverage)
   - Criação de entidades (factory methods)
   - Invariantes de negócio
   - Métodos de comportamento
   - Value Objects

2. **Application Layer** (80%+ coverage)
   - Handlers (fluxos happy path, edge cases)
   - Validadores (regras de validação)
   - Result Pattern

### ❌ O que NÃO Testar

- ❌ Infrastructure (EF Core, Migrations)
- ❌ Controllers (fazem dispatch, não lógica)
- ❌ Middleware
- ❌ DTOs de mapeamento simples
- ❌ External API calls (mockar em vez disso)

---

## 🧬 Setup de Testes

> 📋 **Versões e Dependências**: [../../backend-requirements.md](../../backend-requirements.md)

### Projeto de Testes

```xml
<!-- tests/{{ProjectBase}}.UnitTests/{{ProjectBase}}.UnitTests.csproj -->
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>{{TargetFramework}}</TargetFramework>
        <IsTestProject>true</IsTestProject>
    </PropertyGroup>

    <!-- Ver backend-requirements.md para versões específicas -->
    <ItemGroup>
        <PackageReference Include="Microsoft.NET.Test.Sdk" Version="{{Version}}" />
        <PackageReference Include="xunit" Version="{{Version}}" />
        <PackageReference Include="xunit.runner.visualstudio" Version="{{Version}}">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="FluentAssertions" Version="{{Version}}" />
        <PackageReference Include="Moq" Version="{{Version}}" />
        <PackageReference Include="NSubstitute" Version="{{Version}}" />
    </ItemGroup>

    <ItemGroup>
        <ProjectReference Include="..\..\src\backend\{{ProjectBase}}.Domain\{{ProjectBase}}.Domain.csproj" />
        <ProjectReference Include="..\..\src\backend\{{ProjectBase}}.Application\{{ProjectBase}}.Application.csproj" />
    </ItemGroup>

</Project>
```

### Pasta Structure

```
tests/
└── {{ProjectBase}}.UnitTests/
    ├── {{ProjectBase}}.UnitTests.csproj
    ├── GlobalUsings.cs
    ├── Domain/
    │   ├── Entities/
    │   │   ├── UsuarioTests.cs
    │   │   ├── PedidoTests.cs
    │   │   └── FailedLoginAttemptsTests.cs
    │   └── ValueObjects/
    │       ├── EmailTests.cs
    │       └── CpfTests.cs
    ├── Application/
    │   ├── UseCases/
    │   │   ├── Usuario/
    │   │   │   ├── CriarUsuarioHandlerTests.cs
    │   │   │   ├── AtualizarUsuarioHandlerTests.cs
    │   │   │   └── ExcluirUsuarioHandlerTests.cs
    │   │   └── Autenticacao/
    │   │       └── AutenticarUsuarioHandlerTests.cs
    │   └── Validators/
    │       └── CriarUsuarioRequestValidatorTests.cs
    └── Helpers/
        └── TestDataBuilder.cs
```

---

## 📊 Domain Tests (Entity)

### Exemplo: Usuario Entity Tests

```csharp
// tests/{{ProjectBase}}.UnitTests/Domain/Entities/UsuarioTests.cs
namespace {{ProjectBase}}.UnitTests.Domain.Entities;

public sealed class UsuarioTests
{
    [Fact]
    public void Criar_ComEmailValido_DeveCriarComSucesso()
    {
        // Arrange
        var email = "usuario@example.com";
        var senhaHash = "hash_seguro_12345";

        // Act
        var usuario = Usuario.Criar(email, senhaHash);

        // Assert
        usuario.Email.Should().Be(email);
        usuario.SenhaHash.Should().Be(senhaHash);
        usuario.Ativo.Should().BeTrue();
        usuario.Id.Should().NotBeEmpty();
    }

    [Fact]
    public void Criar_ComEmailVazio_DeveLancarDomainException()
    {
        // Arrange
        var email = string.Empty;
        var senhaHash = "hash_seguro";

        // Act & Assert
        FluentActions.Invoking(() => Usuario.Criar(email, senhaHash))
            .Should().Throw<DomainException>()
            .WithMessage("Email é obrigatório");
    }

    [Fact]
    public void Criar_ComSenhaVazia_DeveLancarDomainException()
    {
        // Arrange
        var email = "usuario@example.com";
        var senhaHash = string.Empty;

        // Act & Assert
        FluentActions.Invoking(() => Usuario.Criar(email, senhaHash))
            .Should().Throw<DomainException>()
            .WithMessage("Senha hash é obrigatório");
    }

    [Fact]
    public void AtualizarEmail_ComEmailValido_DeveAtualizarComSucesso()
    {
        // Arrange
        var usuario = Usuario.Criar("antigo@example.com", "hash");
        var novoEmail = "novo@example.com";

        // Act
        usuario.AtualizarEmail(novoEmail);

        // Assert
        usuario.Email.Should().Be(novoEmail);
    }

    [Fact]
    public void Desativar_UsuarioAtivo_DeveDesativarComSucesso()
    {
        // Arrange
        var usuario = Usuario.Criar("usuario@example.com", "hash");

        // Act
        usuario.Desativar();

        // Assert
        usuario.Ativo.Should().BeFalse();
    }

    [Fact]
    public void Desativar_UsuarioJaDesativado_DeveLancarDomainException()
    {
        // Arrange
        var usuario = Usuario.Criar("usuario@example.com", "hash");
        usuario.Desativar();

        // Act & Assert
        FluentActions.Invoking(() => usuario.Desativar())
            .Should().Throw<DomainException>()
            .WithMessage("Usuário já está desativado");
    }

    [Fact]
    public void Ativar_UsuarioDesativado_DeveAtivarComSucesso()
    {
        // Arrange
        var usuario = Usuario.Criar("usuario@example.com", "hash");
        usuario.Desativar();

        // Act
        usuario.Ativar();

        // Assert
        usuario.Ativo.Should().BeTrue();
    }
}
```

---

## 🎯 Application Tests (Handler)

### Exemplo: CriarUsuarioHandler Tests

```csharp
// tests/{{ProjectBase}}.UnitTests/Application/UseCases/Usuario/CriarUsuarioHandlerTests.cs
namespace {{ProjectBase}}.UnitTests.Application.UseCases.Usuario;

public sealed class CriarUsuarioHandlerTests
{
    private readonly Mock<IUsuarioRepository> _repositoryMock = new();
    private readonly Mock<IPasswordHasher> _passwordHasherMock = new();
    private readonly Mock<IValidator<CriarUsuarioRequest>> _validatorMock = new();
    private readonly Mock<ILogger<CriarUsuarioHandler>> _loggerMock = new();

    [Fact]
    public async Task HandleAsync_ComRequestValido_DeveCriarUsuarioComSucesso()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "novo@example.com",
            Senha = "Senha@123"
        };

        var senhaHash = "hash_bcrypt_seguro";

        _validatorMock
            .Setup(v => v.ValidateAsync(request, default))
            .ReturnsAsync(new ValidationResult());

        _repositoryMock
            .Setup(r => r.EhEmailUnicoAsync("novo@example.com", null, default))
            .ReturnsAsync(true);

        _passwordHasherMock
            .Setup(p => p.Hash("Senha@123"))
            .Returns(senhaHash);

        var handler = new CriarUsuarioHandler(
            _repositoryMock.Object,
            _passwordHasherMock.Object,
            _validatorMock.Object,
            _loggerMock.Object);

        // Act
        var result = await handler.HandleAsync(request, default);

        // Assert
        result.IsSuccess.Should().BeTrue();
        result.Value.Should().NotBeNull();
        result.Value!.Email.Should().Be("novo@example.com");

        _repositoryMock.Verify(r => r.AdicionarAsync(It.IsAny<Usuario>(), default), Times.Once);
    }

    [Fact]
    public async Task HandleAsync_ComEmailJaCadastrado_DeveRetornarFailureValidation()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "existente@example.com",
            Senha = "Senha@123"
        };

        var errosValidacao = new Dictionary<string, string[]>
        {
            { "Email", new[] { "Este email já está cadastrado" } }
        };

        var validationFailure = new ValidationResult(new[]
        {
            new ValidationFailure("Email", "Este email já está cadastrado")
        });

        _validatorMock
            .Setup(v => v.ValidateAsync(request, default))
            .ReturnsAsync(validationFailure);

        var handler = new CriarUsuarioHandler(
            _repositoryMock.Object,
            _passwordHasherMock.Object,
            _validatorMock.Object,
            _loggerMock.Object);

        // Act
        var result = await handler.HandleAsync(request, default);

        // Assert
        result.IsSuccess.Should().BeFalse();
        result.Error!.Code.Should().Be(Error.VALIDATION_ERROR);
        result.Error.Details.Should().ContainKey("Email");

        _repositoryMock.Verify(r => r.AdicionarAsync(It.IsAny<Usuario>(), default), Times.Never);
    }

    [Fact]
    public async Task HandleAsync_ComEmailVazio_DeveRetornarFailureValidation()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = string.Empty,
            Senha = "Senha@123"
        };

        var validationFailure = new ValidationResult(new[]
        {
            new ValidationFailure("Email", "Email é obrigatório")
        });

        _validatorMock
            .Setup(v => v.ValidateAsync(request, default))
            .ReturnsAsync(validationFailure);

        var handler = new CriarUsuarioHandler(
            _repositoryMock.Object,
            _passwordHasherMock.Object,
            _validatorMock.Object,
            _loggerMock.Object);

        // Act
        var result = await handler.HandleAsync(request, default);

        // Assert
        result.IsSuccess.Should().BeFalse();
        result.Error!.Code.Should().Be(Error.VALIDATION_ERROR);
    }

    [Fact]
    public async Task HandleAsync_QuandoRepositoryFalha_DeveRetornarFailureDomainError()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "novo@example.com",
            Senha = "Senha@123"
        };

        _validatorMock
            .Setup(v => v.ValidateAsync(request, default))
            .ReturnsAsync(new ValidationResult());

        _repositoryMock
            .Setup(r => r.EhEmailUnicoAsync("novo@example.com", null, default))
            .ReturnsAsync(true);

        _passwordHasherMock
            .Setup(p => p.Hash("Senha@123"))
            .Returns("hash_seguro");

        _repositoryMock
            .Setup(r => r.AdicionarAsync(It.IsAny<Usuario>(), default))
            .ThrowsAsync(new Exception("Erro de banco de dados"));

        var handler = new CriarUsuarioHandler(
            _repositoryMock.Object,
            _passwordHasherMock.Object,
            _validatorMock.Object,
            _loggerMock.Object);

        // Act
        var result = await handler.HandleAsync(request, default);

        // Assert
        result.IsSuccess.Should().BeFalse();
        result.Error!.Code.Should().Be(Error.DOMAIN);
    }
}
```

---

## 🔍 Validator Tests

```csharp
// tests/{{ProjectBase}}.UnitTests/Application/Validators/CriarUsuarioRequestValidatorTests.cs
namespace {{ProjectBase}}.UnitTests.Application.Validators;

public sealed class CriarUsuarioRequestValidatorTests
{
    private readonly Mock<IUsuarioRepository> _repositoryMock = new();
    private CriarUsuarioRequestValidator _validator = null!;

    public CriarUsuarioRequestValidatorTests()
    {
        _validator = new CriarUsuarioRequestValidator(_repositoryMock.Object);
    }

    [Fact]
    public async Task Validate_ComEmailValido_DevePassar()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "usuario@example.com",
            Senha = "Senha@123"
        };

        _repositoryMock
            .Setup(r => r.EhEmailUnicoAsync("usuario@example.com", null, default))
            .ReturnsAsync(true);

        // Act
        var result = await _validator.ValidateAsync(request);

        // Assert
        result.IsValid.Should().BeTrue();
    }

    [Theory]
    [InlineData("")]
    [InlineData(" ")]
    [InlineData(null)]
    public async Task Validate_ComEmailVazio_DeveFalhar(string? email)
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = email!,
            Senha = "Senha@123"
        };

        // Act
        var result = await _validator.ValidateAsync(request);

        // Assert
        result.IsValid.Should().BeFalse();
        result.Errors.Should().ContainSingle(e => e.PropertyName == "Email");
    }

    [Fact]
    public async Task Validate_ComEmailJaCadastrado_DeveFalhar()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "existente@example.com",
            Senha = "Senha@123"
        };

        _repositoryMock
            .Setup(r => r.EhEmailUnicoAsync("existente@example.com", null, default))
            .ReturnsAsync(false);  // Email já existe

        // Act
        var result = await _validator.ValidateAsync(request);

        // Assert
        result.IsValid.Should().BeFalse();
        result.Errors.Should().ContainSingle(e => e.PropertyName == "Email");
    }

    [Fact]
    public async Task Validate_ComSenhaMuitoCurta_DeveFalhar()
    {
        // Arrange
        var request = new CriarUsuarioRequest
        {
            Email = "usuario@example.com",
            Senha = "123"  // Menos de 6 caracteres
        };

        _repositoryMock
            .Setup(r => r.EhEmailUnicoAsync("usuario@example.com", null, default))
            .ReturnsAsync(true);

        // Act
        var result = await _validator.ValidateAsync(request);

        // Assert
        result.IsValid.Should().BeFalse();
        result.Errors.Should().ContainSingle(e => e.PropertyName == "Senha");
    }
}
```

---

## 🏭 Test Data Builder

```csharp
// tests/{{ProjectBase}}.UnitTests/Helpers/TestDataBuilder.cs
namespace {{ProjectBase}}.UnitTests.Helpers;

public sealed class UsuarioTestDataBuilder
{
    private string _email = "usuario@example.com";
    private string _senhaHash = "hash_bcrypt_seguro";
    private bool _ativo = true;

    public UsuarioTestDataBuilder ComEmail(string email)
    {
        _email = email;
        return this;
    }

    public UsuarioTestDataBuilder ComSenhaHash(string senhaHash)
    {
        _senhaHash = senhaHash;
        return this;
    }

    public UsuarioTestDataBuilder Desativado()
    {
        _ativo = false;
        return this;
    }

    public Usuario Build()
        => Usuario.Criar(_email, _senhaHash);  // Retorna e depois desativa se necessário
}

// Uso
[Fact]
public void Test()
{
    var usuario = new UsuarioTestDataBuilder()
        .ComEmail("custom@example.com")
        .Desativado()
        .Build();

    usuario.Email.Should().Be("custom@example.com");
}
```

---

## ✅ Checklist de Testing

- [ ] Projeto de testes criado com xunit
- [ ] 100% Domain Layer coverage
- [ ] 80%+ Application Layer coverage
- [ ] Mocks para dependências (IRepository, IValidator, etc)
- [ ] Happy path tests
- [ ] Edge case tests
- [ ] Error handling tests
- [ ] ValidationResult tests
- [ ] Test data builders criados
- [ ] Fixtures se necessário
- [ ] Tests executados localmente antes de commit

---

**Próxima Seção**: Ver [`07-setup.md`](./07-setup.md) para setup inicial.

# 📂 Estrutura de Pastas

## 🌳 Árvore Completa

```
src/backend/
├── AgenteViagem.sln
├── README.md
├── README_DEVELOPMENT.md
├── .github/
│   └── instructions/  ← Você está aqui
│
├── AgenteViagem.Domain/
│   ├── AgenteViagem.Domain.csproj
│   ├── Common/
│   │   ├── BaseEntity.cs
│   │   ├── DomainException.cs
│   │   └── ValueObject.cs
│   ├── Entities/
│   │   ├── Usuario.cs
│   │   ├── Pedido.cs
│   │   ├── ItemPedido.cs
│   │   └── FailedLoginAttempts.cs
│   ├── ValueObjects/
│   │   ├── Email.cs
│   │   ├── Cpf.cs
│   │   └── Endereco.cs
│   ├── Enums/
│   │   ├── StatusPedido.cs
│   │   └── TipoPagamento.cs
│   └── Interfaces/
│       ├── IRepository.cs
│       ├── IUsuarioRepository.cs
│       ├── IPedidoRepository.cs
│       ├── IUnitOfWork.cs
│       ├── ITokenService.cs
│       ├── IPasswordHasher.cs
│       └── INotificationService.cs
│
├── AgenteViagem.Application/
│   ├── AgenteViagem.Application.csproj
│   ├── Common/
│   │   ├── Results/
│   │   │   ├── Result.cs
│   │   │   └── Error.cs
│   │   ├── Pagination/
│   │   │   └── ResultadoPaginado.cs
│   │   ├── Security/
│   │   │   ├── IPasswordHasher.cs
│   │   │   ├── BcryptPasswordHasher.cs
│   │   │   └── PasswordHasherOptions.cs
│   │   └── Helpers/
│   │       └── MappingExtensions.cs
│   ├── UseCases/
│   │   ├── Usuario/
│   │   │   ├── CriarUsuario/
│   │   │   │   ├── CriarUsuarioRequest.cs
│   │   │   │   ├── CriarUsuarioResponse.cs
│   │   │   │   ├── CriarUsuarioRequestValidator.cs
│   │   │   │   └── CriarUsuarioHandler.cs
│   │   │   ├── AtualizarUsuario/
│   │   │   │   ├── AtualizarUsuarioRequest.cs
│   │   │   │   ├── AtualizarUsuarioRequestValidator.cs
│   │   │   │   └── AtualizarUsuarioHandler.cs
│   │   │   ├── ObterUsuarioPorId/
│   │   │   │   ├── ObterUsuarioPorIdRequest.cs
│   │   │   │   └── ObterUsuarioPorIdHandler.cs
│   │   │   ├── ObterUsuarioPaginado/
│   │   │   │   ├── ObterUsuarioPaginadoRequest.cs
│   │   │   │   ├── ObterUsuarioPaginadoResponse.cs
│   │   │   │   └── ObterUsuarioPaginadoHandler.cs
│   │   │   └── ExcluirUsuario/
│   │   │       ├── ExcluirUsuarioRequest.cs
│   │   │       └── ExcluirUsuarioHandler.cs
│   │   ├── Autenticacao/
│   │   │   ├── AutenticarUsuario/
│   │   │   │   ├── AutenticarUsuarioRequest.cs
│   │   │   │   ├── AutenticarUsuarioResponse.cs
│   │   │   │   ├── AutenticarUsuarioRequestValidator.cs
│   │   │   │   └── AutenticarUsuarioHandler.cs
│   │   │   └── RefreshToken/
│   │   │       ├── RefreshTokenRequest.cs
│   │   │       ├── RefreshTokenResponse.cs
│   │   │       └── RefreshTokenHandler.cs
│   │   └── Pedidos/
│   │       ├── CriarPedido/
│   │       │   ├── CriarPedidoRequest.cs
│   │       │   ├── CriarPedidoResponse.cs
│   │       │   └── CriarPedidoHandler.cs
│   │       └── ... (outros casos de uso)
│   ├── Extensions/
│   │   └── ServiceCollectionExtensions.cs
│   └── GlobalUsings.cs
│
├── AgenteViagem.Infrastructure/
│   ├── AgenteViagem.Infrastructure.csproj
│   ├── Data/
│   │   ├── AppDbContext.cs
│   │   ├── UnitOfWork.cs
│   │   ├── Configurations/
│   │   │   ├── UsuarioConfiguration.cs
│   │   │   ├── PedidoConfiguration.cs
│   │   │   ├── FailedLoginAttemptsConfiguration.cs
│   │   │   └── ...
│   │   └── Migrations/
│   │       ├── 20250125_Initial.cs
│   │       ├── 20250127_AddAutenticacao.cs
│   │       └── AppDbContextModelSnapshot.cs
│   ├── Repositories/
│   │   ├── UsuarioRepository.cs
│   │   ├── PedidoRepository.cs
│   │   ├── FailedLoginAttemptsRepository.cs
│   │   ├── Repository.cs (base genérico)
│   │   └── ...
│   ├── Security/
│   │   ├── BcryptPasswordHasher.cs
│   │   └── PasswordHasherOptions.cs
│   ├── Extensions/
│   │   └── ServiceCollectionExtensions.cs
│   └── GlobalUsings.cs
│
└── AgenteViagem.API/
    ├── AgenteViagem.API.csproj
    ├── appsettings.json
    ├── appsettings.Development.json
    ├── Program.cs
    ├── Controllers/
    │   ├── HomeController.cs
    │   └── v1/
    │       ├── UsuariosController.cs
    │       ├── PedidosController.cs
    │       ├── AutenticacaoController.cs
    │       └── ...
    ├── Services/
    │   ├── TokenService.cs
    │   └── ...
    ├── Middleware/
    │   ├── ErrorHandlingMiddleware.cs
    │   └── ...
    ├── Extensions/
    │   ├── ServiceCollectionExtensions.cs
    │   ├── ControllerExtensions.cs
    │   └── ...
    ├── Properties/
    │   └── launchSettings.json
    └── GlobalUsings.cs
```

---

## 📋 Regras por Camada

### Domain (Núcleo)

**Organização**:
- `Common/` - BaseEntity, DomainException, ValueObject
- `Entities/` - Agregados (Usuario, Pedido, etc)
- `ValueObjects/` - Email, CPF, Endereco
- `Enums/` - StatusPedido, TipoPagamento
- `Interfaces/` - Contratos de repositório e serviços

**Nomes**:
- ✅ Entidades: `Usuario`, `Pedido`, `ItemPedido`
- ✅ Value Objects: `Email`, `Cpf`, `Endereco`
- ✅ Interfaces: `IUsuarioRepository`, `ITokenService`
- ✅ Exceções: `DomainException`

---

### Application (Orquestração)

**Organização**:
```
UseCases/
├── Usuario/
│   ├── CriarUsuario/
│   │   ├── CriarUsuarioRequest.cs
│   │   ├── CriarUsuarioResponse.cs
│   │   ├── CriarUsuarioRequestValidator.cs
│   │   └── CriarUsuarioHandler.cs
│   ├── AtualizarUsuario/
│   ├── ObterUsuarioPorId/
│   ├── ObterUsuarioPaginado/
│   └── ExcluirUsuario/
└── Autenticacao/
    ├── AutenticarUsuario/
    └── RefreshToken/
```

**Por quê esta estrutura?**
- ✅ Cada caso de uso em pasta separada
- ✅ Tudo que precisa junto está junto
- ✅ Fácil encontrar código relacionado
- ✅ Reduz imports desnecessários

**Nomes**:
- ✅ Handlers: `CriarUsuarioHandler`
- ✅ Requests: `CriarUsuarioRequest`
- ✅ Responses: `UsuarioResponse`
- ✅ Validators: `CriarUsuarioRequestValidator`

---

### Infrastructure (Detalhes Técnicos)

**Organização**:
```
Infrastructure/
├── Data/
│   ├── AppDbContext.cs
│   ├── UnitOfWork.cs
│   ├── Configurations/  ← EF Core mappings
│   └── Migrations/
├── Repositories/  ← Implementações concretas
├── Security/  ← Hashing, criptografia
└── Extensions/  ← ServiceCollectionExtensions
```

**Nomes**:
- ✅ Repositórios: `UsuarioRepository : IUsuarioRepository`
- ✅ Configurações EF: `UsuarioConfiguration : IEntityTypeConfiguration<Usuario>`
- ✅ Serviços: `BcryptPasswordHasher : IPasswordHasher`

---

### API (Presentação)

**Organização**:
```
API/
├── Controllers/
│   ├── v1/  ← Versionamento de API
│   │   ├── UsuariosController.cs
│   │   ├── PedidosController.cs
│   │   └── AutenticacaoController.cs
│   └── HomeController.cs
├── Services/  ← Serviços específicos da API
│   └── TokenService.cs
├── Middleware/  ← Middleware HTTP
│   └── ErrorHandlingMiddleware.cs
└── Extensions/  ← Configuração DI
    └── ServiceCollectionExtensions.cs
```

**Nomes**:
- ✅ Controllers: `UsuariosController` (plural)
- ✅ Métodos: `[HttpPost] public async Task<IActionResult> Criar(...)`

---

## 🎯 Como Navegar

**Encontrar lógica de negócio**:
```
src/backend/AgenteViagem.Domain/Entities/Usuario.cs
```

**Encontrar validação**:
```
src/backend/AgenteViagem.Application/UseCases/Usuario/CriarUsuario/CriarUsuarioRequestValidator.cs
```

**Encontrar orquestração**:
```
src/backend/AgenteViagem.Application/UseCases/Usuario/CriarUsuario/CriarUsuarioHandler.cs
```

**Encontrar acesso a dados**:
```
src/backend/AgenteViagem.Infrastructure/Repositories/UsuarioRepository.cs
```

**Encontrar endpoint HTTP**:
```
src/backend/AgenteViagem.API/Controllers/v1/UsuariosController.cs
```

---

## ✅ Checklist de Estrutura

- [ ] Domain tem apenas negócio puro
- [ ] Application tem handlers verticais por caso de uso
- [ ] Infrastructure tem implementações concretas
- [ ] API tem controllers que injetam handlers
- [ ] Nenhuma classe cruza as fronteiras de forma cíclica
- [ ] Cada handler está em sua própria pasta
- [ ] Validators ao lado de seus Request
- [ ] EF Core Configurations em `Infrastructure/Data/Configurations/`
- [ ] Migrations em `Infrastructure/Data/Migrations/`

---

**Próxima Seção**: Ver [`05-database.md`](./05-database.md) para Entity Framework.

# BKI-0001-autenticacao-seguranca

## Visão Geral

Bloco de Conhecimento Integrado para autenticação e segurança da aplicação AgenteViagem.

**Status Geral**: ⚠️ Implementação em Progresso
- Backend (US002): ✅ Completo
- Frontend (US002): ⏳ Planejado

---

## User Stories Incluídas

### US002 - Autenticação de Usuário com JWT

**Descrição**: Implementar fluxo de autenticação seguro com JWT, proteção contra força bruta e gerenciamento de sessão.

**Status**: 
- **Backend**: ✅ Completo (2025-11-27)
- **Frontend**: ⏳ Planejado
- **Testes**: ⏳ Planejado

**Artefatos Entregues**:
- ✅ Entidade `FailedLoginAttempts` para rastreamento de força bruta
- ✅ Handler `AutenticarUsuarioHandler` com validação e JWT
- ✅ Service `TokenService` para geração/validação de tokens
- ✅ Controller `AutenticacaoController` com endpoint POST `/api/v1/autenticacao/login`
- ✅ Configuração EF Core e Migration para novos dados
- ✅ Middleware JWT em Program.cs

**Endpoints Implementados**:
- `POST /api/v1/autenticacao/login` - Autenticar usuário (retorna JWT)

**Configuração**:
```json
{
  "Jwt": {
    "Secret": "${JWT_SECRET}",
    "Issuer": "AgenteViagemAPI",
    "Audience": "AgenteViagemClients",
    "ExpiracaoMinutos": 60
  }
}
```

**Segurança**:
- ✅ Senhas armazenadas como hash bcrypt
- ✅ Token JWT com HS256 (HMAC SHA-256)
- ✅ Proteção contra força bruta (5 tentativas → bloqueio 15 minutos)
- ✅ Mensagens genéricas de erro (não revela dados)
- ✅ Expiração de token em 1 hora

**Próximos Passos**:
1. Implementar testes unitários (Domain/Application)
2. Implementar componentes frontend (LoginForm, useAuth hook)
3. Testar fluxo completo login → Dashboard
4. Documentar integração frontend-backend

---

## Tabelas de Banco de Dados

### Usuarios (atualizada)
- ✅ `DataUltimoLogin` (DateTimeOffset, nullable) - Data do último login

### TentativasLoginFalhadas (nova)
- ✅ `Id` (GUID, PK)
- ✅ `UsuarioId` (GUID, FK → Usuarios)
- ✅ `TentativasFalhadas` (INT)
- ✅ `DataUltimaTentativa` (DateTimeOffset)
- ✅ `BloqueadoAte` (DateTimeOffset, nullable)

---

## Implementação Status por Componente

| Componente | Status | Data | Commit | Notas |
|-----------|--------|------|--------|-------|
| Domain Entities | ✅ | 2025-11-27 | - | Usuario + FailedLoginAttempts |
| Domain Interfaces | ✅ | 2025-11-27 | - | ITokenService, IFailedLoginAttemptsRepository |
| Application Handlers | ✅ | 2025-11-27 | - | AutenticarUsuarioHandler |
| Application Validators | ✅ | 2025-11-27 | - | AutenticarUsuarioRequestValidator |
| Application DTOs | ✅ | 2025-11-27 | - | AutenticarUsuarioRequest/Response |
| Infrastructure Repository | ✅ | 2025-11-27 | - | FailedLoginAttemptsRepository |
| Infrastructure Config | ✅ | 2025-11-27 | - | UsuarioConfiguration, FailedLoginAttemptsConfiguration |
| Infrastructure Migration | ✅ | 2025-11-27 | - | 20251127_US002_AddAutenticacao |
| API Services | ✅ | 2025-11-27 | - | TokenService |
| API Controller | ✅ | 2025-11-27 | - | AutenticacaoController |
| API Middleware | ✅ | 2025-11-27 | - | JWT Authentication em Program.cs |
| Configuration | ✅ | 2025-11-27 | - | appsettings + JWT config |
| Documentation | ✅ | 2025-11-27 | - | README_DEVELOPMENT.md atualizado |

---

## Instruções de Deployment

### Development
```powershell
cd src/backend/AgenteViagem.API
dotnet run
# API disponível em http://localhost:5002
# Scalar UI em http://localhost:5002/scalar
```

### Testar Autenticação
```bash
# Login
curl -X POST http://localhost:5002/api/v1/autenticacao/login \
  -H "Content-Type: application/json" \
  -d '{"email":"usuario@test.com","senha":"123456"}'

# Usar token em requisições protegidas
curl -X GET http://localhost:5002/api/v1/usuarios/123 \
  -H "Authorization: Bearer {token}"
```

### Production
```bash
# Configure variáveis de ambiente
export Jwt__Secret="sua_chave_secreta_muito_longa_e_aleatória"
export ConnectionStrings__DefaultConnection="sua_connection_string_postgres"

# Execute migrations
dotnet ef database update -p AgenteViagem.Infrastructure -s AgenteViagem.API

# Execute a aplicação
dotnet run --environment Production
```

---

## Referências

- **PRD**: `docs/prd/BKI-0001-autenticacao-seguranca/US002/PRD-US002-autenticacao-usuario-2025-11-27.md`
- **User Story**: `docs/stories/US002-autenticacao-usuario.md`
- **API Docs**: http://localhost:5002/openapi.json
- **Development Guide**: `src/backend/README_DEVELOPMENT.md`

---

## Changelog

| Data | Versão | Alteração | Status |
|------|--------|-----------|--------|
| 2025-11-27 | 1.0 | Implementação inicial de autenticação JWT com força bruta | ✅ Completo |

---

**Última Atualização**: 2025-11-27  
**Responsável**: Backend Implementation  
**Próxima Revisão**: Após testes unitários

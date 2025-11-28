# USER STORY: Autenticação de Usuário

**StoryVersion**: v1  
**USID**: US002  
**BKI Relacionado**: BKI-0001-autenticacao-seguranca  
**Fonte**: Requisito de funcionalidade de acesso seguro à aplicação

Implementar sistema de autenticação seguro com JWT, permitindo que usuários façam login via e-mail e senha, com redirecionamento automático para Dashboard após autenticação bem-sucedida.

---

## US002: Autenticação de Usuário com JWT

**Como** usuário da aplicação,  
**Quero** fazer login fornecendo e-mail e senha,  
**Para** acessar a aplicação de forma segura e gerenciar minhas viagens.

---

## Critérios de Aceitação

- [ ] **CA-001**: Dado que estou na página de login, Quando preencherei e-mail e senha válidos e clicar em "Autenticar", Então o sistema gera um token JWT contendo o ID do usuário e redireciona para o Dashboard.

- [ ] **CA-002**: Dado que estou na página de login, Quando preencherei e-mail e senha inválidos e clicar em "Autenticar", Então o sistema exibe a mensagem "Usuário ou senha inválidos" e permanece na página de login.

- [ ] **CA-003**: Dado que possuo um token JWT válido, Quando faço uma requisição para uma rota protegida incluindo o token no header `Authorization: Bearer <token>`, Então acesso o recurso protegido sem restrição.

- [ ] **CA-004**: Dado que possuo um token JWT expirado, Quando faço uma requisição para uma rota protegida, Então o sistema retorna erro 401 (Unauthorized) e redireciona para login.

- [ ] **CA-005**: Dado que estou autenticado, Quando acesso a aplicação novamente sem fazer logout, Então permaneço autenticado enquanto o token for válido.

---

## Cenários Negativos

- **CN-001**: Dado que tentei fazer login 5 vezes com credenciais inválidas em menos de 1 minuto, Quando faço a 6ª tentativa, Então recebo mensagem "Muitas tentativas falhadas. Tente novamente em 15 minutos" e a conta é temporariamente bloqueada.

- **CN-002**: Dado que tentei fazer login com e-mail não cadastrado, Quando clico em "Autenticar", Então recebo mensagem genérica "Usuário ou senha inválidos" (não revelar qual campo está errado).

- **CN-003**: Dado que tentei fazer login com campos em branco, Quando clico em "Autenticar", Então recebo mensagem de validação "E-mail e senha são obrigatórios".

---

## Regras de Negócio

- **RN-001**: Senhas devem ser armazenadas em hash (bcrypt ou similar) no banco de dados. Comparação com senha fornecida também via hash.

- **RN-002**: Token JWT deve conter:
  - `sub` (subject): ID do usuário
  - `email`: E-mail do usuário
  - `exp` (expiration): Data/hora de expiração (padrão: 1 hora)
  - `iat` (issued at): Data/hora de geração

- **RN-003**: Token JWT deve ser assinado com chave secreta armazenada de forma segura (variável de ambiente).

- **RN-004**: Proteção contra força bruta: após 5 tentativas falhadas em 1 minuto, bloquear a conta por 15 minutos.

- **RN-005**: Mensagens de erro devem ser genéricas ("Usuário ou senha inválidos") para não revelar se o e-mail existe ou não.

---

## Exceções / Restrições

- **EXC-001**: Usuários não cadastrados não podem fazer login.

- **EXC-002**: Usuários com conta inativa ou desativada não podem fazer login.

- **EXC-003**: Não incluir funcionalidade "Lembrar-me" (manter login persistente é responsabilidade do frontend via armazenamento seguro do token).

- **EXC-004**: Não incluir autenticação social (Google, GitHub, etc.) — foco apenas em autenticação básica e-mail/senha.

- **EXC-005**: Logout é responsabilidade do frontend (remover token do localStorage/sessionStorage).

---

## Dependências

- **DEP-001**: Entidade `Usuario` deve existir com campos: `Id`, `Email`, `SenhaHash`, `Ativo`, `DataCriacao`, `DataUltimoLogin`.

- **DEP-002**: Middleware de autenticação JWT deve ser configurado no pipeline da API.

- **DEP-003**: Componente de login frontend deve ser implementado em React/TypeScript.

- **DEP-004**: Rota `/api/v1/autenticacao/login` (POST) deve estar disponível no backend.

---

## Glossário Local

| Termo | Definição |
|-------|-----------|
| JWT (JSON Web Token) | Token de autenticação autossuficiente que contém informações do usuário criptografadas |
| Bearer Token | Formato de autenticação onde o token é precedido pela palavra "Bearer" no header Authorization |
| Hash (bcrypt) | Função criptográfica unidirecional para armazenar senhas de forma segura |
| Força Bruta | Ataque que tenta múltiplas combinações de credenciais para ganhar acesso |
| Token Expirado | Token JWT cuja data de expiração (`exp`) é anterior ao momento atual |

---

## Observações

- **Próximo artefato esperado**: PRD (gerado por @prd-writer) com detalhes de API contracts, implementação backend (Controllers, Services, Repositories) e frontend (componentes React).

- **Sugestão (fora do escopo atual)**: Implementar refresh tokens para renovar autenticação sem requerer novo login (pode ser adicionado em US futura).

- **Sugestão (fora do escopo atual)**: Auditoria de login (logs de tentativas, IP, data/hora) para segurança e compliance.

---

## ✅ Próximas Etapas

1. **Validar com o time**: Confirmar se as regras de força bruta (5 tentativas em 1 minuto) e tempo de expiração (1 hora) estão corretos para seu contexto de negócio.

2. **Próximo passo**: Ao confirmar, chamar `@prd-writer` para gerar o PRD técnico com API contracts e plano de implementação.

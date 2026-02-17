# 🔒 Regras de Segurança - Frontend

> Este arquivo centraliza **TODAS** as regras de segurança para operações do frontend.

---

## 🔐 Princípios Gerais

1. **Nunca armazenar dados sensíveis** em localStorage/sessionStorage sem criptografia
2. **Sanitizar inputs** antes de renderizar para prevenir XSS
3. **Validar dados** tanto no cliente quanto no servidor
4. **Não expor secrets** em código client-side

---

## ⚠️ Variáveis de Ambiente - REGRA CRÍTICA

### Prefixo Obrigatório

Vite expõe apenas variáveis com prefixo `VITE_`:

```bash
# ✅ Correto - Será exposta ao client
VITE_API_URL=http://localhost:5000/api
VITE_APP_NAME=MinhaApp

# ❌ NUNCA - Secrets não devem ir para o cliente
DATABASE_URL=...          # Backend only
JWT_SECRET=...            # Backend only
API_SECRET_KEY=...        # Backend only
```

### Arquivos .env

| Arquivo | Uso | Git |
|---------|-----|-----|
| `.env` | Configurações base | ✅ Commit |
| `.env.local` | Overrides locais | ❌ .gitignore |
| `.env.development` | Ambiente dev | ✅ Commit |
| `.env.production` | Ambiente prod | ✅ Commit |

### Tipagem de Variáveis

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_APP_NAME: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

---

## 🛡️ Autenticação & Tokens

### Storage de Tokens

```typescript
// ✅ Correto - HTTP-only cookies (gerenciado pelo backend)
// Token JWT setado via Set-Cookie header do backend

// ✅ Aceitável - Memory storage (perde ao recarregar)
const authStore = create<AuthState>(() => ({
  token: null,
  user: null,
}))

// ⚠️ Usar com cuidado - localStorage
// Vulnerável a XSS, mas aceitável para refresh tokens curtos
localStorage.setItem('refreshToken', token)

// ❌ NUNCA - localStorage para access tokens sensíveis
localStorage.setItem('accessToken', jwtToken) // Vulnerável!
```

### Headers de Autenticação

```typescript
// ✅ Correto - Interceptor centralizado
api.interceptors.request.use((config) => {
  const token = authStore.getState().token
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// ❌ Errado - Token hardcoded
headers: { Authorization: 'Bearer abc123' }
```

---

## 🚫 Prevenção XSS

### Renderização de Conteúdo

```tsx
// ✅ Correto - React escapa automaticamente
const UserName = ({ name }: { name: string }) => (
  <span>{name}</span>
)

// ⚠️ PERIGO - dangerouslySetInnerHTML
// Usar APENAS com conteúdo sanitizado
import DOMPurify from 'dompurify'

const SafeHTML = ({ html }: { html: string }) => (
  <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
)

// ❌ NUNCA - HTML não sanitizado
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

### URLs Dinâmicas

```tsx
// ✅ Correto - Validar protocolo
const SafeLink = ({ url }: { url: string }) => {
  const isValid = url.startsWith('https://') || url.startsWith('http://')
  return isValid ? <a href={url}>Link</a> : null
}

// ❌ NUNCA - URLs não validadas
<a href={userProvidedUrl}>Link</a>  // Pode ser javascript:alert()
```

---

## 🔑 Formulários & Validação

### Validação Client-Side

```typescript
// ✅ Correto - Zod schema com sanitização
const loginSchema = z.object({
  email: z.string().email().trim().toLowerCase(),
  senha: z.string().min(8).max(100),
})

// ✅ Correto - Validação antes de enviar
const onSubmit = async (data: LoginForm) => {
  const validated = loginSchema.parse(data) // Throws se inválido
  await api.post('/auth/login', validated)
}
```

### Campos Sensíveis

```tsx
// ✅ Correto - Campos de senha
<input type="password" autoComplete="current-password" />

// ✅ Correto - Desabilitar autocomplete para dados sensíveis
<input type="text" autoComplete="off" data-lpignore="true" />
```

---

## 📡 Requisições HTTP

### CORS & Credentials

```typescript
// ✅ Correto - Configuração Axios
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true, // Para cookies HTTP-only
  headers: {
    'Content-Type': 'application/json',
  },
})
```

### Tratamento de Erros

```typescript
// ✅ Correto - Não expor detalhes técnicos
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      authStore.getState().logout()
      window.location.href = '/login'
    }
    // Log apenas em dev
    if (import.meta.env.DEV) {
      console.error('API Error:', error)
    }
    return Promise.reject(error)
  }
)

// ❌ NUNCA - Mostrar stack traces ao usuário
catch (error) {
  alert(error.stack) // Expõe detalhes internos
}
```

---

## ✅ Checklist de Segurança

| Item | Verificação |
|------|-------------|
| Variáveis de ambiente | Apenas `VITE_*` no client |
| Tokens JWT | Não em localStorage (preferir memory/cookies) |
| Inputs de usuário | Escapados automaticamente pelo React |
| HTML dinâmico | Sanitizado com DOMPurify |
| URLs externas | Validadas antes de usar |
| Formulários | Validados com Zod |
| Requisições HTTP | Interceptors centralizados |
| Erros | Sem stack traces em produção |
| Console logs | Removidos em produção |

---

## 🔗 Referências

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- React Security Best Practices: https://react.dev/learn/security
- Vite Env Variables: https://vitejs.dev/guide/env-and-mode

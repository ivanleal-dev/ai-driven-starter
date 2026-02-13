# 📂 Estrutura de Pastas - Vertical Slice Architecture

## 🎯 Arquitetura Vertical Slice

Frontend organizado por **features completas e autossuficientes**. Cada feature contém tudo relacionado a ela: componentes, hooks, services, types, schemas.

---

## 🌳 Árvore Completa

```
src/frontend/
├── index.html
├── package.json
├── package-lock.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── tailwind.config.js
├── postcss.config.js
├── eslint.config.js
├── .prettierrc
├── .env
├── .env.development
├── .env.production
├── README.md                    # Pode existir, mas NÃO criar/atualizar para documentar estrutura (a menos que solicitado)
│
├── public/
│   ├── favicon.svg
│   └── robots.txt
│
└── src/
    ├── main.tsx                    # Entry point
    ├── App.tsx                     # Root component
    ├── vite-env.d.ts              # Vite type declarations
    │
    ├── assets/
    │   ├── images/
    │   │   └── logo.svg
    │   └── styles/
    │       └── global.css          # Tailwind imports + global
    │
    ├── features/                   # 🎯 VERTICAL SLICES (feature completa)
    │   │
    │   ├── auth/                   # Feature: Autenticação
    │   │   ├── components/
    │   │   │   ├── LoginForm.tsx
    │   │   │   ├── RegisterForm.tsx
    │   │   │   └── ProtectedRoute.tsx
    │   │   ├── hooks/
    │   │   │   └── useAuth.ts
    │   │   ├── services/
    │   │   │   └── auth.service.ts
    │   │   ├── types/
    │   │   │   └── auth.types.ts
    │   │   ├── schemas/
    │   │   │   └── auth.schema.ts
    │   │   ├── pages/
    │   │   │   ├── LoginPage.tsx
    │   │   │   └── RegisterPage.tsx
    │   │   └── index.ts            # Public API da feature
    │   │
    │   ├── clientes/               # Feature: Gestão de Clientes
    │   │   ├── components/
    │   │   │   ├── ClienteCard.tsx
    │   │   │   ├── ClienteList.tsx
    │   │   │   ├── ClienteForm.tsx
    │   │   │   └── ClienteFilters.tsx
    │   │   ├── hooks/
    │   │   │   ├── useClientes.ts
    │   │   │   ├── useCliente.ts
    │   │   │   └── useClienteMutations.ts
    │   │   ├── services/
    │   │   │   └── cliente.service.ts
    │   │   ├── types/
    │   │   │   └── cliente.types.ts
    │   │   ├── schemas/
    │   │   │   └── cliente.schema.ts
    │   │   ├── pages/
    │   │   │   ├── ClientesPage.tsx
    │   │   │   ├── ClienteDetailsPage.tsx
    │   │   │   └── ClienteFormPage.tsx
    │   │   └── index.ts
    │   │
    │   ├── usuarios/               # Feature: Gestão de Usuários
    │   │   ├── components/
    │   │   │   ├── UsuarioCard.tsx
    │   │   │   ├── UsuarioList.tsx
    │   │   │   └── UsuarioForm.tsx
    │   │   ├── hooks/
    │   │   │   ├── useUsuarios.ts
    │   │   │   └── useUsuarioMutations.ts
    │   │   ├── services/
    │   │   │   └── usuario.service.ts
    │   │   ├── types/
    │   │   │   └── usuario.types.ts
    │   │   ├── schemas/
    │   │   │   └── usuario.schema.ts
    │   │   ├── pages/
    │   │   │   ├── UsuariosPage.tsx
    │   │   │   └── UsuarioFormPage.tsx
    │   │   └── index.ts
    │   │
    │   ├── oportunidades/          # Feature: Pipeline de Vendas
    │   │   ├── components/
    │   │   │   ├── OportunidadeCard.tsx
    │   │   │   ├── PipelineBoard.tsx
    │   │   │   └── OportunidadeForm.tsx
    │   │   ├── hooks/
    │   │   │   ├── useOportunidades.ts
    │   │   │   └── useOportunidadeMutations.ts
    │   │   ├── services/
    │   │   │   └── oportunidade.service.ts
    │   │   ├── types/
    │   │   │   └── oportunidade.types.ts
    │   │   ├── schemas/
    │   │   │   └── oportunidade.schema.ts
    │   │   ├── pages/
    │   │   │   ├── OportunidadesPage.tsx
    │   │   │   └── OportunidadeDetailsPage.tsx
    │   │   └── index.ts
    │   │
    │   └── dashboard/              # Feature: Dashboard
    │       ├── components/
    │       │   ├── StatCard.tsx
    │       │   ├── RevenueChart.tsx
    │       │   └── ActivityFeed.tsx
    │       ├── hooks/
    │       │   └── useDashboardData.ts
    │       ├── services/
    │       │   └── dashboard.service.ts
    │       ├── types/
    │       │   └── dashboard.types.ts
    │       ├── pages/
    │       │   └── DashboardPage.tsx
    │       └── index.ts
    │
    ├── shared/                     # 🔧 SHARED (uso em 2+ features)
    │   ├── components/             # Componentes reutilizáveis
    │   │   ├── ui/                 # shadcn/ui primitivos
    │   │   │   ├── button.tsx
    │   │   │   ├── card.tsx
    │   │   │   ├── input.tsx
    │   │   │   ├── form.tsx
    │   │   │   ├── dialog.tsx
    │   │   │   ├── table.tsx
    │   │   │   └── index.ts
    │   │   ├── layout/             # Layout compartilhado
    │   │   │   ├── Header.tsx
    │   │   │   ├── Sidebar.tsx
    │   │   │   ├── Footer.tsx
    │   │   │   ├── PageHeader.tsx
    │   │   │   └── index.ts
    │   │   └── feedback/           # Feedback visual
    │   │       ├── Toast.tsx
    │   │       ├── Alert.tsx
    │   │       ├── EmptyState.tsx
    │   │       └── index.ts
    │   │
    │   ├── hooks/                  # Hooks utilitários
    │   │   ├── useDebounce.ts
    │   │   ├── useLocalStorage.ts
    │   │   ├── useMediaQuery.ts
    │   │   └── index.ts
    │   │
    │   ├── utils/                  # Funções helpers
    │   │   ├── format.ts           # Formatadores
    │   │   ├── validators.ts       # Validações
    │   │   ├── constants.ts
    │   │   └── index.ts
    │   │
    │   └── types/                  # Types compartilhados
    │       ├── common.types.ts
    │       ├── api.types.ts
    │       └── index.ts
    │
    ├── core/                       # ⚙️ CORE (infraestrutura)
    │   ├── api/
    │   │   └── httpClient.ts       # Axios configurado
    │   ├── auth/
    │   │   └── AuthProvider.tsx    # Contexto global de auth
    │   ├── router/
    │   │   └── AppRouter.tsx       # Configuração de rotas
    │   ├── store/                  # Estado global (Zustand)
    │   │   ├── auth.store.ts
    │   │   ├── ui.store.ts
    │   │   └── index.ts
    │   └── config/
    │       ├── env.ts              # Variáveis de ambiente
    │       ├── queryClient.ts      # React Query config
    │       └── index.ts
    │
    ├── lib/                        # shadcn/ui utils
    │   └── utils.ts                # cn() helper
    │
    └── __tests__/                  # Testes de integração
        ├── setup.ts
        └── mocks/
            ├── handlers.ts
            └── server.ts
```

---

## 📋 Regras da Arquitetura Vertical Slice

### 🎯 Estrutura de uma Feature

Cada feature é **autossuficiente** e contém **tudo** relacionado a ela:

```
features/clientes/
├── components/          # Componentes ESPECÍFICOS da feature
│   ├── ClienteCard.tsx
│   ├── ClienteList.tsx
│   └── ClienteForm.tsx
│
├── hooks/              # Hooks ESPECÍFICOS da feature
│   ├── useClientes.ts
│   └── useClienteMutations.ts
│
├── services/           # Chamadas API ESPECÍFICAS da feature
│   └── cliente.service.ts
│
├── types/              # Types ESPECÍFICOS da feature
│   └── cliente.types.ts
│
├── schemas/            # Validações ESPECÍFICAS da feature
│   └── cliente.schema.ts
│
├── pages/              # Páginas (rotas) da feature
│   ├── ClientesPage.tsx
│   ├── ClienteDetailsPage.tsx
│   └── ClienteFormPage.tsx
│
└── index.ts            # 🔑 PUBLIC API (só exporta o necessário)
```

---

## 🔑 Public API (index.ts)

**Regra Crítica**: Cada feature **DEVE** ter um `index.ts` que exporta **apenas** o que outras features podem usar.

```typescript
// features/clientes/index.ts

// ✅ Exportar: Pages (para rotas)
export { ClientesPage } from './pages/ClientesPage'
export { ClienteDetailsPage } from './pages/ClienteDetailsPage'
export { ClienteFormPage } from './pages/ClienteFormPage'

// ✅ Exportar: Types públicos (se outras features precisarem)
export type { Cliente, CreateClienteDTO } from './types/cliente.types'

// ❌ NÃO exportar: Componentes internos (ClienteCard, ClienteList)
// ❌ NÃO exportar: Hooks (useClientes)
// ❌ NÃO exportar: Services (clienteService)
```

---

## 🚫 Regras de Isolamento

### ✅ Permitido

```typescript
// ✅ Feature pode usar SHARED
import { Button, Card } from '@/shared/components/ui'
import { useDebounce } from '@/shared/hooks'
import { formatCurrency } from '@/shared/utils'

// ✅ Feature pode usar CORE
import { httpClient } from '@/core/api/httpClient'
import { useAuthStore } from '@/core/store/auth.store'

// ✅ Feature pode importar de sua PRÓPRIA pasta
import { ClienteCard } from './components/ClienteCard'
import { useClientes } from './hooks/useClientes'
```

### ❌ Proibido

```typescript
// ❌ Feature NÃO pode importar outra feature diretamente
import { ClienteCard } from '@/features/clientes/components/ClienteCard'

// ✅ Só pode usar o que está no index.ts
import { ClientesPage } from '@/features/clientes'

// ❌ NÃO pode acessar subpastas de outra feature
import { useClientes } from '@/features/clientes/hooks/useClientes'
```

---

## 📂 Decisões: Feature vs Shared vs Core

### ➡️ Feature (features/)

**Quando usar**: Código específico de uma funcionalidade de negócio.

**Exemplos**:
- `ClienteCard` → `features/clientes/components/`
- `useClientes` → `features/clientes/hooks/`
- `clienteService` → `features/clientes/services/`

### ➡️ Shared (shared/)

**Quando usar**: Código usado em **2 ou mais features**.

**Exemplos**:
- `Button`, `Input`, `Card` → `shared/components/ui/`
- `Header`, `Sidebar` → `shared/components/layout/`
- `useDebounce` → `shared/hooks/`
- `formatCurrency` → `shared/utils/`

### ➡️ Core (core/)

**Quando usar**: Infraestrutura técnica, não muda por feature.

**Exemplos**:
- `httpClient` (Axios) → `core/api/`
- `AuthProvider` → `core/auth/`
- `AppRouter` → `core/router/`
- `queryClient` → `core/config/`

---

## 📋 Regras por Pasta

### `/features/[nome-feature]`

**Estrutura Obrigatória**:
```
features/clientes/
├── components/        # Componentes internos da feature
├── hooks/            # Hooks específicos da feature
├── services/         # API calls da feature
├── types/            # Types da feature
├── schemas/          # Validações Zod da feature
├── pages/            # Páginas/rotas da feature
└── index.ts          # 🔑 PUBLIC API
```

**Exemplo Completo: Feature Clientes**

```typescript
// features/clientes/components/ClienteCard.tsx
import { Cliente } from '../types/cliente.types'
import { Card } from '@/shared/components/ui'

interface ClienteCardProps {
  cliente: Cliente
  onEdit: (id: string) => void
}

export const ClienteCard = ({ cliente, onEdit }: ClienteCardProps) => (
  <Card>
    <h3>{cliente.nome}</h3>
    <p>{cliente.email}</p>
    <button onClick={() => onEdit(cliente.id)}>Editar</button>
  </Card>
)
```

```typescript
// features/clientes/hooks/useClientes.ts
import { useQuery } from '@tanstack/react-query'
import { clienteService } from '../services/cliente.service'

export const useClientes = () => {
  return useQuery({
    queryKey: ['clientes'],
    queryFn: clienteService.listar,
  })
}
```

```typescript
// features/clientes/services/cliente.service.ts
import { httpClient } from '@/core/api/httpClient'
import type { Cliente, CreateClienteDTO } from '../types/cliente.types'

export const clienteService = {
  listar: async (): Promise<Cliente[]> => {
    const { data } = await httpClient.get('/api/clientes')
    return data
  },
  
  criar: async (dto: CreateClienteDTO): Promise<Cliente> => {
    const { data } = await httpClient.post('/api/clientes', dto)
    return data
  },
}
```

```typescript
// features/clientes/types/cliente.types.ts
export interface Cliente {
  id: string
  nome: string
  email: string
}

export interface CreateClienteDTO {
  nome: string
  email: string
}
```

```typescript
// features/clientes/schemas/cliente.schema.ts
import { z } from 'zod'

export const clienteSchema = z.object({
  nome: z.string().min(3),
  email: z.string().email(),
})

export type ClienteFormData = z.infer<typeof clienteSchema>
```

```typescript
// features/clientes/pages/ClientesPage.tsx
import { useClientes } from '../hooks/useClientes'
import { ClienteList } from '../components/ClienteList'
import { PageHeader } from '@/shared/components/layout'

export const ClientesPage = () => {
  const { data: clientes, isLoading } = useClientes()

  return (
    <div>
      <PageHeader titulo="Clientes" />
      {isLoading ? <div>Carregando...</div> : <ClienteList clientes={clientes || []} />}
    </div>
  )
}
```

```typescript
// features/clientes/index.ts (PUBLIC API)
// ✅ Exportar apenas o necessário para outras features

export { ClientesPage } from './pages/ClientesPage'
export { ClienteDetailsPage } from './pages/ClienteDetailsPage'
export { ClienteFormPage } from './pages/ClienteFormPage'

// Types públicos (se outras features precisarem)
export type { Cliente, CreateClienteDTO } from './types/cliente.types'
```

---

### `/shared`

**Quando usar**: Componentes/hooks/utils usados em **2 ou mais features**.

**Estrutura**:
```
shared/
├── components/
│   ├── ui/              # shadcn/ui primitivos
│   ├── layout/          # Layout compartilhado
│   └── feedback/        # Feedback visual
├── hooks/               # Hooks utilitários
├── utils/               # Funções helpers
└── types/               # Types compartilhados
```

**Regra**: Se algo é usado apenas em 1 feature → fica na feature. Se é usado em 2+ → vai para shared.

---

### `/core`

**Quando usar**: Infraestrutura técnica que **não muda por feature**.

**Estrutura**:
```
core/
├── api/
│   └── httpClient.ts      # Axios configurado
├── auth/
│   └── AuthProvider.tsx   # Contexto global
├── router/
│   └── AppRouter.tsx      # Rotas
├── store/
│   ├── auth.store.ts      # Estado de autenticação
│   └── ui.store.ts        # Estado de UI
└── config/
    ├── env.ts             # Variáveis de ambiente
    └── queryClient.ts     # React Query config
```

---

### Exemplos de Decisão: Feature vs Shared vs Core

| Item | Localização | Motivo |
|------|-------------|--------|
| `ClienteCard` | `features/clientes/components/` | Específico de clientes |
| `Button` | `shared/components/ui/` | Usado em várias features |
| `Header` | `shared/components/layout/` | Layout compartilhado |
| `useClientes` | `features/clientes/hooks/` | Hook específico de clientes |
| `useDebounce` | `shared/hooks/` | Utilitário genérico |
| `httpClient` | `core/api/` | Infraestrutura |
| `clienteService` | `features/clientes/services/` | API específica de clientes |
| `formatCurrency` | `shared/utils/` | Formatador usado em várias features |
| `Cliente` (type) | `features/clientes/types/` | Type específico de clientes |
| `ApiResponse` (type) | `shared/types/` | Type genérico de API |

---

## 📁 Arquivo index.ts (Re-exports)

### Feature index.ts

```typescript
// features/clientes/index.ts
// ✅ Exportar: Pages, Types públicos
export { ClientesPage } from './pages/ClientesPage'
export type { Cliente } from './types/cliente.types'

// ❌ NÃO exportar: Componentes internos, Hooks, Services
// Eles são PRIVADOS da feature
```

### Shared index.ts

```typescript
// shared/components/ui/index.ts
export * from './button'
export * from './card'
export * from './input'

// shared/hooks/index.ts
export * from './useDebounce'
export * from './useLocalStorage'

// shared/utils/index.ts
export * from './format'
export * from './validators'
```

---

## ✅ Checklist de Organização

| Critério | Verificação |
|----------|-------------|
| Feature autossuficiente | ✅ Componentes, hooks, services na feature |
| Public API (index.ts) | ✅ Só exporta Pages e types públicos |
| Componentes internos privados | ✅ Não exportados no index.ts |
| shadcn/ui em shared/ui/ | ✅ `shared/components/ui/button.tsx` |
| Layout em shared/layout/ | ✅ `shared/components/layout/Header.tsx` |
| Infraestrutura em core/ | ✅ `core/api/httpClient.ts` |
| Hooks genéricos em shared/ | ✅ `shared/hooks/useDebounce.ts` |
| Feature não importa de feature | ✅ Isolamento respeitado |

---

## 🎯 Vantagens da Vertical Slice

1. **Isolamento**: Tudo de clientes está em `features/clientes/`
2. **Escalabilidade**: Nova feature = nova pasta
3. **Manutenção**: Bug em clientes? Vá em `features/clientes/`
4. **Remoção**: Deletar feature = deletar pasta
5. **Onboarding**: Estrutura clara e previsível
6. **Testabilidade**: Teste de feature isolado

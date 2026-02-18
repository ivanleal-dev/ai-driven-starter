# 🏛️ Arquitetura Frontend - Vertical Slice Architecture

> 📋 **Stack & Versões**: [09-stack.md](09-stack.md)

## 📚 Visão Geral

Frontend desenvolvido seguindo **Vertical Slice Architecture**, onde cada feature é **autossuficiente e isolada**, contendo tudo necessário para funcionar: componentes, hooks, services, types, schemas e páginas.

---

## 🎯 Princípios Fundamentais

### 1. Vertical Slice (Feature-Centric)

Código organizado por **features completas**, não por camadas técnicas.

```
❌ Organização Horizontal (por camada técnica)
components/
├── ClienteCard.tsx
├── UsuarioCard.tsx
└── OportunidadeCard.tsx
hooks/
├── useClientes.ts
├── useUsuarios.ts
└── useOportunidades.ts

✅ Organização Vertical (por feature)
features/
├── clientes/
│   ├── components/ClienteCard.tsx
│   ├── hooks/useClientes.ts
│   └── services/cliente.service.ts
├── usuarios/
│   ├── components/UsuarioCard.tsx
│   ├── hooks/useUsuarios.ts
│   └── services/usuario.service.ts
└── oportunidades/
    ├── components/OportunidadeCard.tsx
    ├── hooks/useOportunidades.ts
    └── services/oportunidade.service.ts
```

### 2. Feature Isolation (Isolamento)

Cada feature é **autossuficiente** e **não depende** de outras features.

```
┌──────────────────────────────────────────┐
│  Feature: Clientes                       │
│  ┌────────────────────────────────────┐  │
│  │ components/  (ClienteCard, etc)    │  │
│  │ hooks/       (useClientes)         │  │
│  │ services/    (clienteService)      │  │
│  │ types/       (Cliente)             │  │
│  │ schemas/     (clienteSchema)       │  │
│  │ pages/       (ClientesPage)        │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

```tsx
// ✅ Correto: Feature autossuficiente
// features/clientes/pages/ClientesPage.tsx
import { useClientes } from '../hooks/useClientes'          // Própria feature
import { ClienteList } from '../components/ClienteList'      // Própria feature
import { PageHeader } from '@/shared/components/layout'      // Shared
import { Button } from '@/shared/components/ui'              // Shared

export const ClientesPage = () => {
  const { data: clientes } = useClientes()
  return (
    <div>
      <PageHeader titulo="Clientes" />
      <ClienteList clientes={clientes || []} />
    </div>
  )
}

// ❌ Errado: Dependência de outra feature
import { UsuarioCard } from '@/features/usuarios/components/UsuarioCard'
```

### 3. Public API (index.ts)

Cada feature expõe uma **API pública** através do `index.ts`, controlando o que pode ser usado externamente.

```typescript
// features/clientes/index.ts

// ✅ Exportar: Páginas (para rotas)
export { ClientesPage } from './pages/ClientesPage'
export { ClienteDetailsPage } from './pages/ClienteDetailsPage'

// ✅ Exportar: Types públicos (se necessário)
export type { Cliente, CreateClienteDTO } from './types/cliente.types'

// ❌ NÃO exportar: Componentes internos (privados)
// ❌ NÃO exportar: Hooks (privados)
// ❌ NÃO exportar: Services (privados)
```

### 4. Shared for Reusability

Código usado em **2 ou mais features** vai para `shared/`.

```
shared/
├── components/
│   ├── ui/        # Primitivos (Button, Input, Card)
│   └── layout/    # Layout compartilhado (Header, Sidebar)
├── hooks/         # Hooks utilitários (useDebounce)
├── utils/         # Funções helpers (formatCurrency)
└── types/         # Types compartilhados
```

### 5. Core for Infrastructure

Infraestrutura técnica que **não muda por feature** vai para `core/`.

```
core/
├── api/           # httpClient (Axios)
├── auth/          # AuthProvider
├── router/        # AppRouter
├── store/         # Estado global (auth, ui)
└── config/        # Configurações (env, queryClient)
```

---

## 🏗️ Camadas da Arquitetura

```
┌─────────────────────────────────────────────┐
│  Features (features/)                       │
│  ┌───────────────────────────────────────┐  │
│  │ clientes/ usuarios/ oportunidades/    │  │
│  │ - components, hooks, services, pages  │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  Shared (shared/)                           │
│  ┌───────────────────────────────────────┐  │
│  │ UI, Layout, Hooks, Utils              │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  Core (core/)                               │
│  ┌───────────────────────────────────────┐  │
│  │ API, Auth, Router, Store, Config      │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### 1️⃣ Features (Vertical Slices)

**Responsabilidade**: Feature completa de negócio.

**Estrutura**:
```
features/clientes/
├── components/         # Componentes ESPECÍFICOS
├── hooks/             # Hooks ESPECÍFICOS
├── services/          # API calls ESPECÍFICAS
├── types/             # Types ESPECÍFICOS
├── schemas/           # Validações ESPECÍFICAS
├── pages/             # Páginas da feature
└── index.ts           # Public API
```

**Exemplo**:
```tsx
// features/clientes/pages/ClientesPage.tsx
import { useClientes } from '../hooks/useClientes'
import { ClienteList } from '../components/ClienteList'
import { PageHeader } from '@/shared/components/layout'

export const ClientesPage = () => {
  const { data: clientes, isLoading } = useClientes()
  
  return (
    <div>
      <PageHeader titulo="Clientes" />
      {isLoading ? <div>Loading...</div> : <ClienteList clientes={clientes || []} />}
    </div>
  )
}
```

### 2️⃣ Shared (Reutilizáveis)

**Responsabilidade**: Componentes/hooks/utils usados em **2+ features**.

**Regra**: Se é usado apenas em 1 feature → fica na feature. Se é usado em 2+ → vai para shared.

**Exemplos**:
```tsx
// shared/components/ui/button.tsx (shadcn/ui)
export const Button = forwardRef<HTMLButtonElement, ButtonProps>((props, ref) => {
  // ...
})

// shared/components/layout/PageHeader.tsx
export const PageHeader = ({ titulo }: { titulo: string }) => (
  <header className="pb-4 mb-4 border-b">
    <h1 className="text-2xl font-bold">{titulo}</h1>
  </header>
)

// shared/hooks/useDebounce.ts
export const useDebounce = <T,>(value: T, delay: number): T => {
  // ...
}
```

### 3️⃣ Core (Infraestrutura)

**Responsabilidade**: Serviços técnicos transversais.

**NÃO contém**:
- ❌ Lógica de negócio
- ❌ Componentes específicos

**Exemplos**:
```typescript
// core/api/httpClient.ts
import axios from 'axios'
import { env } from '../config/env'

export const httpClient = axios.create({
  baseURL: env.apiUrl,
  withCredentials: true,
})

// core/auth/AuthProvider.tsx
export const AuthProvider = ({ children }: PropsWithChildren) => {
  // Lógica de autenticação global
  return <AuthContext.Provider value={...}>{children}</AuthContext.Provider>
}

// core/router/AppRouter.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'
import { ClientesPage } from '@/features/clientes'
import { UsuariosPage } from '@/features/usuarios'

const router = createBrowserRouter([
  { path: '/clientes', element: <ClientesPage /> },
  { path: '/usuarios', element: <UsuariosPage /> },
])

export const AppRouter = () => <RouterProvider router={router} />
```

---

## 🔄 Fluxo de Dados (Vertical Slice)

```
┌──────────────────────────────────────────────┐
│  Feature: Clientes                           │
│                                              │
│  Page                                        │
│    ↓                                         │
│  Hook (useClientes)                          │
│    ↓                                         │
│  Service (clienteService) ─────┐            │
│                                 ↓            │
└─────────────────────────────────┼────────────┘
                                  │
                          ┌───────┴────────┐
                          │  httpClient    │  ← Core
                          │  (Axios)       │
                          └───────┬────────┘
                                  │
                          ┌───────▼────────┐
                          │   Backend API  │
                          └────────────────┘
```

**Exemplo Completo**:

```typescript
// 1. Service (features/clientes/services/cliente.service.ts)
import { httpClient } from '@/core/api/httpClient'
import type { Cliente } from '../types/cliente.types'

export const clienteService = {
  listar: async (): Promise<Cliente[]> => {
    const { data } = await httpClient.get('/api/clientes')
    return data
  },
}

// 2. Hook (features/clientes/hooks/useClientes.ts)
import { useQuery } from '@tanstack/react-query'
import { clienteService } from '../services/cliente.service'

export const useClientes = () => {
  return useQuery({
    queryKey: ['clientes'],
    queryFn: clienteService.listar,
  })
}

// 3. Component (features/clientes/components/ClienteList.tsx)
import { Cliente } from '../types/cliente.types'
import { ClienteCard } from './ClienteCard'

interface ClienteListProps {
  clientes: Cliente[]
}

export const ClienteList = ({ clientes }: ClienteListProps) => (
  <div className="grid grid-cols-3 gap-4">
    {clientes.map((cliente) => (
      <ClienteCard key={cliente.id} cliente={cliente} />
    ))}
  </div>
)

// 4. Page (features/clientes/pages/ClientesPage.tsx)
import { useClientes } from '../hooks/useClientes'
import { ClienteList } from '../components/ClienteList'

export const ClientesPage = () => {
  const { data: clientes, isLoading } = useClientes()
  
  if (isLoading) return <div>Carregando...</div>
  
  return <ClienteList clientes={clientes || []} />
}
```

---

## 📊 Decisão: Feature vs Shared vs Core

| Item | Onde? | Por quê? |
|------|-------|----------|
| `ClienteCard` | `features/clientes/components/` | Específico de clientes |
| `UsuarioForm` | `features/usuarios/components/` | Específico de usuários |
| `Button` | `shared/components/ui/` | Usado em várias features |
| `Header` | `shared/components/layout/` | Layout compartilhado |
| `useClientes` | `features/clientes/hooks/` | Hook específico de clientes |
| `useDebounce` | `shared/hooks/` | Utilitário genérico |
| `clienteService` | `features/clientes/services/` | API específica de clientes |
| `httpClient` | `core/api/` | Infraestrutura HTTP |
| `formatCurrency` | `shared/utils/` | Usado em várias features |
| `AuthProvider` | `core/auth/` | Infraestrutura de autenticação |

---

## ✅ Princípios da Vertical Slice

1. ✅ **Coesão**: Tudo relacionado a uma feature está junto
2. ✅ **Isolamento**: Features não dependem umas das outras
3. ✅ **Public API**: Controle explícito do que é público
4. ✅ **Escalabilidade**: Nova feature = nova pasta
5. ✅ **Manutenção**: Bug em clientes → vá em `features/clientes/`
6. ✅ **Deletabilidade**: Remover feature = deletar pasta

---

## 🔗 Referências

- Vertical Slice Architecture: https://www.jimmybogard.com/vertical-slice-architecture/
- React Docs: https://react.dev
- Vite Guide: https://vitejs.dev/guide
- React Query: https://tanstack.com/query
- Zustand: https://zustand-demo.pmnd.rs

---
applyTo: 'src/frontend/**'
---

# Frontend - Instruções Gerais & Índice

## 📚 Documentação Frontend Organizada

> ⚠️ **Arquitetura**: Usamos **Vertical Slice Architecture** - cada feature é autossuficiente

## 🚫 Regra: não gerar README de estrutura

- **NÃO criar nem atualizar `README.md`** para “explicar a estrutura do frontend” durante implementações.
- A **fonte de verdade** da estrutura/arquitetura é a pasta `.github/rules/frontend/` (principalmente `04-folder-structure.md`).
- O prompt de frontend (arquivo `.github/skills/frontend-develop/SKILL.md`) deve **somente** criar/alterar arquivos de código necessários à feature/ajuste solicitado; documentação estrutural é considerada fora de escopo (a menos que o usuário peça explicitamente).

**Leia nesta ordem:**

### 0. **Regras de Segurança** ⚠️
📖 Ver: `./00-security.md`
- Variáveis de ambiente (VITE_*)
- Storage de tokens
- Prevenção XSS
- Validação de inputs

### 1. **Fundamentos da Arquitetura** 
📖 Ver: `./01-architecture.md`
- **Vertical Slice Architecture**
- Feature Isolation
- Public API (index.ts)
- Shared vs Core vs Features

### 2. **Componentes**
📖 Ver: `./02-components.md`
- UI Components (shadcn/ui em shared/)
- Form Components (React Hook Form)
- Layout Components (shared/layout/)
- Feature Components (features/[nome]/)

### 3. **Rotas & Estado**
📖 Ver: `./03-state-routing.md`
- React Router DOM (rotas, layouts)
- Zustand (estado global em core/)
- React Query (server state)
- React Hook Form + Zod (formulários)

### 4. **Estrutura de Pastas** ⭐ IMPORTANTE
📖 Ver: `./04-folder-structure.md`
- Organização Vertical Slice
- features/ (autossuficientes)
- shared/ (2+ features)
- core/ (infraestrutura)
- Regras de isolamento

### 5. **Nomenclatura & Convenções**
📖 Ver: `./05-conventions.md`
- PascalCase (componentes)
- camelCase (funções, hooks)
- Idioma (PT-BR para domínio)
- Imports ordenados

---

## 🎯 Quick Reference

### Projeto: React + Vite + TypeScript + Vertical Slice

**Stack:**
> 📋 [09-stack.md](09-stack.md)

- **React 18** + **Vite 5** + **TypeScript 5.4**
- **React Router DOM 6** (roteamento)
- **React Hook Form 7** + **Zod 3** (formulários)
- **TanStack Query 5** (server state)
- **Zustand 4** (client state)
- **Tailwind CSS 3** + **shadcn/ui** (estilização)
- **Axios** (HTTP client)

**Arquitetura Vertical Slice:**
```
features/           # 🎯 Cada feature é autossuficiente
├── clientes/
│   ├── components/    # Componentes internos
│   ├── hooks/         # Hooks específicos
│   ├── services/      # API calls
│   ├── types/         # Types
│   ├── schemas/       # Validações
│   ├── pages/         # Páginas/rotas
│   └── index.ts       # 🔑 Public API
├── usuarios/
└── oportunidades/

shared/            # 🔧 Reutilizável (2+ features)
├── components/
│   ├── ui/           # shadcn/ui primitivos
│   └── layout/       # Header, Sidebar
├── hooks/            # useDebounce, etc
└── utils/            # Helpers

core/              # ⚙️ Infraestrutura
├── api/              # httpClient
├── router/           # AppRouter
├── store/            # Estado global
└── config/           # env, queryClient
```

---

## ⚡ Templates Rápidos

### Criando uma Feature Completa

```bash
# Estrutura
features/clientes/
├── components/
│   ├── ClienteCard.tsx
│   └── ClienteList.tsx
├── hooks/
│   ├── useClientes.ts
│   └── useClienteMutations.ts
├── services/
│   └── cliente.service.ts
├── types/
│   └── cliente.types.ts
├── schemas/
│   └── cliente.schema.ts
├── pages/
│   ├── ClientesPage.tsx
│   └── ClienteDetailsPage.tsx
└── index.ts  # Public API
```

### Public API (index.ts)

```typescript
// features/clientes/index.ts
// ✅ Exportar apenas o necessário
export { ClientesPage } from './pages/ClientesPage'
export { ClienteDetailsPage } from './pages/ClienteDetailsPage'
export type { Cliente } from './types/cliente.types'

// ❌ NÃO exportar componentes/hooks/services internos
```

### Criando um Service

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

### Criando um Hook de Query

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

### Criando uma Page

```tsx
// features/clientes/pages/ClientesPage.tsx
import { useClientes } from '../hooks/useClientes'
import { ClienteList } from '../components/ClienteList'
import { PageHeader } from '@/shared/components/layout'
import { Button } from '@/shared/components/ui'

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

### Criando um Schema Zod

```typescript
// features/clientes/schemas/cliente.schema.ts
import { z } from 'zod'

export const clienteSchema = z.object({
  nome: z.string().min(3, 'Nome deve ter pelo menos 3 caracteres'),
  email: z.string().email('Email inválido'),
})

export type ClienteFormData = z.infer<typeof clienteSchema>
```

### Criando um Store Zustand (Core)

```typescript
// core/store/auth.store.ts
import { create } from 'zustand'

interface AuthState {
  user: User | null
  isAuthenticated: boolean
  login: (user: User) => void
  logout: () => void
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isAuthenticated: false,
  login: (user) => set({ user, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false }),
}))
```

---

## 📁 Comandos de Setup

```bash
# Criar projeto Vite
npm create vite@latest frontend -- --template react-ts
cd frontend

# Instalar dependências core
npm install react-router-dom@^6.26
npm install react-hook-form@^7.53 zod@^3.23 @hookform/resolvers@^3.9
npm install @tanstack/react-query@^5.56 zustand@^4.5 axios@^1.7

# Instalar Tailwind
npm install -D tailwindcss@^3.4 postcss autoprefixer
npx tailwindcss init -p

# Instalar shadcn/ui
npx shadcn@latest init --yes
npx shadcn@latest add button input card dialog form

# Instalar testing
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

---

## 🎯 Regras da Vertical Slice

### ✅ Feature Autossuficiente

```
features/clientes/          # Tudo de clientes aqui
├── components/            # ClienteCard, ClienteList
├── hooks/                 # useClientes
├── services/              # clienteService
├── types/                 # Cliente
├── schemas/               # clienteSchema
├── pages/                 # ClientesPage
└── index.ts               # Public API
```

### ✅ Comunicação entre Features

```typescript
// ❌ MAU - import direto de outra feature
import { ClienteCard } from '@/features/clientes/components/ClienteCard'

// ✅ BOM - só usar o que está no index.ts
import { ClientesPage, type Cliente } from '@/features/clientes'
```

### ✅ Shared vs Feature

```typescript
// SHARED: Usado em 2+ features
// shared/components/ui/button.tsx
export const Button = ({ children, ...props }) => { /*...*/ }

// FEATURE: Específico de clientes
// features/clientes/components/ClienteForm.tsx
export const ClienteForm = () => { /*...*/ }
```

### ✅ Core para Infraestrutura

```typescript
// CORE: Não muda por feature
// core/api/httpClient.ts
export const httpClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
})
```

---

## 🔗 Referências Externas

- Vertical Slice: https://www.jimmybogard.com/vertical-slice-architecture/
- React: https://react.dev
- Vite: https://vitejs.dev
- React Router: https://reactrouter.com
- React Hook Form: https://react-hook-form.com
- Zod: https://zod.dev
- TanStack Query: https://tanstack.com/query
- Zustand: https://zustand-demo.pmnd.rs
- Tailwind CSS: https://tailwindcss.com
- shadcn/ui: https://ui.shadcn.com

# 📝 Nomenclatura & Convenções - Frontend

## 🎯 Princípios

1. **Explícito**: Nomes claros, sem abreviações
2. **Consistente**: Mesmo padrão em todo o codebase
3. **TypeScript First**: Tipos explícitos sempre
4. **Idioma**: Código em **PORTUGUÊS** quando faz sentido (features/domínio)

---

## 🏷️ Nomenclatura por Contexto

### Componentes

```tsx
// ✅ Correto - PascalCase
export const ClienteCard = () => { ... }
export const PageHeader = () => { ... }
export const FormField = () => { ... }

// ✅ Correto - Arquivo igual ao componente
// components/cliente/ClienteCard/ClienteCard.tsx

// ❌ Errado
export const clienteCard = () => { ... }  // camelCase
export const cliente_card = () => { ... } // snake_case
export const CLIENTE_CARD = () => { ... } // SCREAMING_CASE
```

### Props Interfaces

```tsx
// ✅ Correto - Sufixo Props
interface ButtonProps {
  variant: 'primary' | 'secondary'
  onClick?: () => void
}

interface ClienteCardProps {
  cliente: Cliente
  onEditar: (id: string) => void
}

// ✅ Correto - Extend HTML props
interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string
  error?: string
}

// ❌ Errado
interface IButtonProps { ... }  // Prefixo I desnecessário
interface ButtonParams { ... }  // Não é Props
```

### Hooks

```typescript
// ✅ Correto - Prefixo use + camelCase
// features/clientes/hooks/useClientes.ts
export const useClientes = () => { ... }
// features/auth/hooks/useAuth.ts
export const useAuth = () => { ... }
// shared/hooks/useDebounce.ts
export const useDebounce = () => { ... }
// features/clientes/hooks/useClienteMutations.ts
export const useClienteMutations = () => { ... }

// ❌ Errado
export const Clientes = () => { ... }      // Sem use
export const UseClientes = () => { ... }   // PascalCase
export const use_clientes = () => { ... }  // snake_case
```

### Services

```typescript
// ✅ Correto - camelCase + sufixo .service
// arquivo: features/clientes/services/cliente.service.ts
export const clienteService = {
  listar: async () => { ... },
  obterPorId: async (id: string) => { ... },
  criar: async (data: CriarClienteRequest) => { ... },
  atualizar: async (id: string, data: AtualizarClienteRequest) => { ... },
  excluir: async (id: string) => { ... },
}

// ❌ Errado
export const ClienteService = { ... }  // PascalCase (não é classe)
export class ClienteService { ... }    // Classes desnecessárias
```

### Stores (Zustand)

```typescript
// ✅ Correto - Hook convention
// arquivo: auth.store.ts
export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}))

// ✅ Uso
const { user, login } = useAuthStore()
const userName = useAuthStore((state) => state.user?.nome)

// ❌ Errado
export const authStore = create(...) // Não é hook pattern
```

### Schemas (Zod)

```typescript
// ✅ Correto - camelCase + Schema suffix
// arquivo: cliente.schema.ts
export const clienteSchema = z.object({
  nome: z.string().min(3),
  email: z.string().email(),
})

export const criarClienteSchema = clienteSchema.omit({ id: true })

// ✅ Type inference
export type ClienteFormData = z.infer<typeof clienteSchema>

// ❌ Errado
export const ClienteSchema = z.object(...)  // PascalCase
export const CLIENTE_SCHEMA = z.object(...) // SCREAMING_CASE
```

### Types/Interfaces

```typescript
// ✅ Correto - PascalCase para tipos
// arquivo: cliente.types.ts
export interface Cliente {
  id: string
  nome: string
  email: string
  criadoEm: Date
}

export interface CriarClienteRequest {
  nome: string
  email: string
}

export interface ClienteResponse {
  success: boolean
  data: Cliente
  message?: string
}

// ❌ Errado
export interface ICliente { ... }           // Prefixo I
export interface clienteInterface { ... }   // camelCase
export type cliente = { ... }               // camelCase
```

### Páginas

```tsx
// ✅ Correto - Sufixo Page
// arquivo: ClientesPage.tsx
export const ClientesPage = () => { ... }
export const ClienteDetalhesPage = () => { ... }
export const ClienteFormPage = () => { ... }

// ❌ Errado
export const Clientes = () => { ... }       // Sem Page
export const ClientesView = () => { ... }   // View (Angular)
export const ClientesScreen = () => { ... } // Screen (React Native)
```

### Funções Utilitárias

```typescript
// ✅ Correto - camelCase
export function formatCurrency(value: number): string { ... }
export function formatDate(date: Date): string { ... }
export function validarCpf(cpf: string): boolean { ... }

// ✅ Correto - Arrow functions também ok
export const formatCpf = (cpf: string): string => { ... }

// ❌ Errado
export function FormatCurrency() { ... }  // PascalCase
export function format_date() { ... }     // snake_case
```

### Constantes

```typescript
// ✅ Correto - SCREAMING_SNAKE_CASE para constantes globais
export const API_URL = 'http://localhost:5000/api'
export const MAX_FILE_SIZE = 5 * 1024 * 1024

// ✅ Correto - Object com as const
export const ROUTES = {
  HOME: '/',
  CLIENTES: '/clientes',
  LOGIN: '/login',
} as const

export const STATUS_COLORS = {
  ATIVO: 'green',
  INATIVO: 'gray',
  PENDENTE: 'yellow',
} as const

// ❌ Evitar - camelCase para constantes importantes
export const apiUrl = '...'  // Parece variável
```

### Event Handlers

```tsx
// ✅ Correto - Prefixo handle ou on
const handleClick = () => { ... }
const handleSubmit = (data: FormData) => { ... }
const handleClienteSelect = (id: string) => { ... }

// ✅ Props - Prefixo on
interface Props {
  onClick: () => void
  onSubmit: (data: FormData) => void
  onClienteSelect: (id: string) => void
}

// ❌ Errado
const clickHandler = () => { ... }  // Handler no final
const submit = () => { ... }        // Sem prefixo
```

---

## 📁 Estrutura de Arquivos

### Nomes de Arquivos

```
✅ Correto:
components/ui/button.tsx          # shadcn/ui (minúsculo)
components/cliente/ClienteCard/ClienteCard.tsx
hooks/useClientes.ts
services/cliente.service.ts
stores/auth.store.ts
schemas/cliente.schema.ts
types/cliente.types.ts
pages/Clientes/ClientesPage.tsx
utils/format.ts

❌ Errado:
components/ui/Button.tsx          # PascalCase para shadcn
components/ui/button/Button.tsx   # shadcn não usa subpastas
hooks/clientes-hook.ts            # kebab-case
services/ClienteService.ts        # PascalCase para service
```

### Index Re-exports

```typescript
// ✅ Correto - shadcn/ui components (arquivos únicos)
// components/ui/button.tsx (gerado automaticamente)

// ✅ Correto - components/ui/index.ts (manual)
export * from './button'
export * from './card'
export * from './input'

// ✅ Correto - Componentes customizados
// components/cliente/ClienteCard/index.ts
export { ClienteCard } from './ClienteCard'
export type { ClienteCardProps } from './ClienteCard'
```

---

## 🎨 Estilização

### Classes Tailwind

```tsx
// ✅ Correto - cn() para composição (shadcn/ui)
import { cn } from '@/lib/utils'

<div className={cn(
  'rounded-lg p-4',
  isActive && 'bg-primary text-primary-foreground',
  className
)} />

// ✅ Correto - Usar variáveis CSS do shadcn/ui
<div className="bg-primary text-primary-foreground" />

// ❌ Errado - Template strings
<div className={`rounded-lg p-4 ${isActive ? 'bg-primary' : ''}`} />
```

### CSS Modules (se usar)

```
Button.module.css     # Sufixo .module.css
styles.module.css     # Genérico por pasta
```

---

## 📦 Imports

### Ordem de Imports

```typescript
// 1. React e bibliotecas externas
import { useState, useEffect } from 'react'
import { useQuery } from '@tanstack/react-query'

// 2. Componentes internos (path alias)
import { Button, Card } from '@/components/ui'
import { PageHeader } from '@/components/layout'

// 3. Hooks, services, stores
import { useClientes } from '@/hooks/useClientes'
import { clienteService } from '@/services/cliente.service'
import { useAuthStore } from '@/stores/auth.store'

// 4. Types
import type { Cliente } from '@/types/cliente.types'

// 5. Utils e constantes
import { formatDate } from '@/utils/format'
import { ROUTES } from '@/utils/constants'

// 6. Estilos
import styles from './Component.module.css'
```

### Path Aliases

```typescript
// ✅ Correto - Usar alias @/
import { Button } from '@/components/ui'
import { useAuth } from '@/hooks/useAuth'

// ❌ Evitar - Caminhos relativos longos
import { Button } from '../../../components/ui/Button'
```

---

## ✅ Checklist de Convenções

| Tipo | Convenção | Exemplo |
|------|-----------|---------|
| shadcn/ui | minúsculo | `button.tsx`, `card.tsx` |
| Componentes | PascalCase | `ClienteCard` |
| Hooks | use + camelCase | `useClientes` |
| Services | camelCase + .service | `clienteService` |
| Stores | use + PascalCase + Store | `useAuthStore` |
| Schemas | camelCase + Schema | `clienteSchema` |
| Types | PascalCase | `Cliente`, `ClienteResponse` |
| Pages | PascalCase + Page | `ClientesPage` |
| Constants | SCREAMING_SNAKE | `API_URL`, `ROUTES` |
| Event handlers | handle/on prefix | `handleClick`, `onSubmit` |
| Arquivos | Igual ao export | `ClienteCard.tsx` |

---

## 🌐 Idioma

### Português (Domínio/Features)

```typescript
// ✅ Nomes de domínio em PT-BR
interface Cliente { nome: string; email: string }
interface Oportunidade { titulo: string; valor: number }

// ✅ Schemas em PT-BR
const clienteSchema = z.object({
  nome: z.string().min(3, 'Nome deve ter pelo menos 3 caracteres'),
})

// ✅ Erros/Mensagens em PT-BR
toast.success('Cliente criado com sucesso!')
toast.error('Erro ao processar requisição')
```

### Inglês (Técnico/React)

```typescript
// ✅ Patterns React em inglês
const [isLoading, setIsLoading] = useState(false)
const { data, error, isError } = useQuery(...)

// ✅ Props técnicas em inglês
interface ButtonProps {
  variant: 'primary' | 'secondary'
  disabled?: boolean
  onClick?: () => void
}

// ✅ Hooks utilitários em inglês
useDebounce()
useLocalStorage()
useMediaQuery()
```

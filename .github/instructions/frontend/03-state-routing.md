# 🛣️ Rotas & Estado - React Router + Zustand + React Query

> Guia completo para roteamento e gerenciamento de estado

---

## 🗺️ React Router DOM

### Configuração Base

```tsx
// src/routes/index.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'
import { RootLayout } from '@/components/layout/RootLayout'
import { ProtectedRoute } from '@/components/auth/ProtectedRoute'

// Pages (importadas das features)
import { HomePage } from '@/features/dashboard'
import { LoginPage } from '@/features/auth'
import { ClientesPage, ClienteDetailsPage, ClienteFormPage } from '@/features/clientes'
import { NotFoundPage } from '@/shared/components/feedback'

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <NotFoundPage />,
    children: [
      { index: true, element: <HomePage /> },
      { path: 'login', element: <LoginPage /> },
      {
        path: 'clientes',
        element: <ProtectedRoute />,
        children: [
          { index: true, element: <ClientesPage /> },
          { path: ':id', element: <ClienteDetailsPage /> },
          { path: 'novo', element: <ClienteFormPage /> },
          { path: ':id/editar', element: <ClienteFormPage /> },
        ],
      },
    ],
  },
])

export const AppRouter = () => <RouterProvider router={router} />
```

### Layout com Outlet

```tsx
// src/components/layout/RootLayout.tsx
import { Outlet } from 'react-router-dom'
import { Header } from './Header'
import { Sidebar } from './Sidebar'

export const RootLayout = () => (
  <div className="min-h-screen bg-gray-50">
    <Header />
    <div className="flex">
      <Sidebar />
      <main className="flex-1 p-6">
        <Outlet />
      </main>
    </div>
  </div>
)
```

### Rota Protegida

```tsx
// features/auth/components/ProtectedRoute.tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom'
import { useAuthStore } from '@/core/store/auth.store'

export const ProtectedRoute = () => {
  const { isAuthenticated } = useAuthStore()
  const location = useLocation()

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }

  return <Outlet />
}
```

### Hooks de Navegação

```tsx
// ✅ useNavigate para navegação programática
import { useNavigate } from 'react-router-dom'

const ClienteCard = ({ cliente }: { cliente: Cliente }) => {
  const navigate = useNavigate()
  
  return (
    <div onClick={() => navigate(`/clientes/${cliente.id}`)}>
      {cliente.nome}
    </div>
  )
}

// ✅ useParams para parâmetros de rota
import { useParams } from 'react-router-dom'

const ClienteDetalhesPage = () => {
  const { id } = useParams<{ id: string }>()
  // ...
}

// ✅ useSearchParams para query strings
import { useSearchParams } from 'react-router-dom'

const ClientesPage = () => {
  const [searchParams, setSearchParams] = useSearchParams()
  const page = searchParams.get('page') || '1'
  
  const handlePageChange = (newPage: number) => {
    setSearchParams({ page: String(newPage) })
  }
}
```

---

## 📦 Zustand (Estado Global)

### Quando Usar

| Usar Zustand ✅ | NÃO Usar ❌ |
|-----------------|-------------|
| Auth (usuário logado) | Cache de API (usar React Query) |
| Tema/Preferências | Estado de formulário (usar RHF) |
| UI global (modais) | Estado local (usar useState) |
| Carrinho de compras | Dados do servidor |

### Store de Autenticação

```typescript
// src/stores/auth.store.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface User {
  id: string
  nome: string
  email: string
}

interface AuthState {
  user: User | null
  token: string | null
  isAuthenticated: boolean
  login: (user: User, token: string) => void
  logout: () => void
  updateUser: (user: Partial<User>) => void
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      
      login: (user, token) => set({ 
        user, 
        token, 
        isAuthenticated: true 
      }),
      
      logout: () => set({ 
        user: null, 
        token: null, 
        isAuthenticated: false 
      }),
      
      updateUser: (updates) => set((state) => ({
        user: state.user ? { ...state.user, ...updates } : null
      })),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ 
        token: state.token,
        // Não persistir user sensível
      }),
    }
  )
)
```

### Store de UI

```typescript
// src/stores/ui.store.ts
import { create } from 'zustand'

interface ModalState {
  isOpen: boolean
  title: string
  content: React.ReactNode | null
}

interface UIState {
  sidebarOpen: boolean
  modal: ModalState
  toggleSidebar: () => void
  openModal: (title: string, content: React.ReactNode) => void
  closeModal: () => void
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  modal: { isOpen: false, title: '', content: null },
  
  toggleSidebar: () => set((state) => ({ 
    sidebarOpen: !state.sidebarOpen 
  })),
  
  openModal: (title, content) => set({ 
    modal: { isOpen: true, title, content } 
  }),
  
  closeModal: () => set({ 
    modal: { isOpen: false, title: '', content: null } 
  }),
}))
```

### Seletores para Performance

```typescript
// ✅ Correto: Seletor específico (re-render mínimo)
const userName = useAuthStore((state) => state.user?.nome)

// ❌ Evitar: Store inteira (re-render sempre)
const { user, token, login, logout } = useAuthStore()

// ✅ Múltiplos valores: usar shallow
import { shallow } from 'zustand/shallow'

const { user, isAuthenticated } = useAuthStore(
  (state) => ({ user: state.user, isAuthenticated: state.isAuthenticated }),
  shallow
)
```

---

## 🔄 React Query (Estado do Servidor)

### Configuração

```tsx
// src/config/queryClient.ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutos
      retry: 1,
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 0,
    },
  },
})

// src/App.tsx
import { QueryClientProvider } from '@tanstack/react-query'
import { queryClient } from '@/config/queryClient'

export const App = () => (
  <QueryClientProvider client={queryClient}>
    <AppRouter />
  </QueryClientProvider>
)
```

### Query Hooks (Leitura)

```typescript
// src/hooks/useClientes.ts
import { useQuery } from '@tanstack/react-query'
import { clienteService } from '@/services/cliente.service'

// Lista
export const useClientes = () => {
  return useQuery({
    queryKey: ['clientes'],
    queryFn: clienteService.listar,
  })
}

// Por ID
export const useCliente = (id: string) => {
  return useQuery({
    queryKey: ['clientes', id],
    queryFn: () => clienteService.obterPorId(id),
    enabled: !!id,
  })
}

// Com filtros
export const useClientesPaginados = (params: ClientesFiltros) => {
  return useQuery({
    queryKey: ['clientes', 'paginado', params],
    queryFn: () => clienteService.listarPaginado(params),
    placeholderData: (previousData) => previousData,
  })
}
```

### Mutation Hooks (Escrita)

```typescript
// src/hooks/useClienteMutations.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { clienteService } from '@/services/cliente.service'
import { toast } from '@/components/ui/Toast'

export const useCriarCliente = () => {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: clienteService.criar,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['clientes'] })
      toast.success('Cliente criado com sucesso!')
    },
    onError: (error) => {
      toast.error('Erro ao criar cliente')
    },
  })
}

export const useAtualizarCliente = () => {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: AtualizarClienteRequest }) =>
      clienteService.atualizar(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['clientes'] })
      queryClient.invalidateQueries({ queryKey: ['clientes', id] })
    },
  })
}

export const useExcluirCliente = () => {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: clienteService.excluir,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['clientes'] })
    },
  })
}
```

### Uso em Componentes

```tsx
// src/pages/Clientes/ClientesPage.tsx
import { useClientes } from '@/hooks/useClientes'
import { useExcluirCliente } from '@/hooks/useClienteMutations'

export const ClientesPage = () => {
  const { data: clientes, isLoading, error } = useClientes()
  const { mutate: excluir, isPending } = useExcluirCliente()

  if (isLoading) return <Loading />
  if (error) return <Error message={error.message} />

  return (
    <div>
      {clientes?.map((cliente) => (
        <ClienteCard
          key={cliente.id}
          cliente={cliente}
          onExcluir={() => excluir(cliente.id)}
          isDeleting={isPending}
        />
      ))}
    </div>
  )
}
```

---

## 📝 React Hook Form + Zod

### Schema de Validação

```typescript
// src/schemas/cliente.schema.ts
import { z } from 'zod'

export const clienteSchema = z.object({
  nome: z.string()
    .min(3, 'Nome deve ter pelo menos 3 caracteres')
    .max(100, 'Nome deve ter no máximo 100 caracteres'),
  email: z.string()
    .email('Email inválido'),
  telefone: z.string()
    .regex(/^\(\d{2}\) \d{4,5}-\d{4}$/, 'Telefone inválido')
    .optional(),
  documento: z.string()
    .refine(validarCpfCnpj, 'CPF/CNPJ inválido'),
})

export type ClienteFormData = z.infer<typeof clienteSchema>
```

### Form Component

```tsx
// src/pages/Clientes/ClienteFormPage.tsx
import { useForm, FormProvider } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { clienteSchema, ClienteFormData } from '@/schemas/cliente.schema'
import { useCriarCliente, useAtualizarCliente } from '@/hooks/useClienteMutations'
import { useCliente } from '@/hooks/useClientes'

export const ClienteFormPage = () => {
  const { id } = useParams<{ id?: string }>()
  const navigate = useNavigate()
  const isEditing = !!id
  
  const { data: cliente } = useCliente(id!)
  const { mutateAsync: criar, isPending: criando } = useCriarCliente()
  const { mutateAsync: atualizar, isPending: atualizando } = useAtualizarCliente()
  
  const form = useForm<ClienteFormData>({
    resolver: zodResolver(clienteSchema),
    defaultValues: cliente || {},
  })

  const onSubmit = async (data: ClienteFormData) => {
    try {
      if (isEditing) {
        await atualizar({ id, data })
      } else {
        await criar(data)
      }
      navigate('/clientes')
    } catch (error) {
      // Erro tratado no mutation hook
    }
  }

  return (
    <FormProvider {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField name="nome" label="Nome" />
        <FormField name="email" label="Email" type="email" />
        <FormField name="telefone" label="Telefone" />
        <FormField name="documento" label="CPF/CNPJ" />
        
        <div className="flex gap-2">
          <Button type="button" variant="outline" onClick={() => navigate(-1)}>
            Cancelar
          </Button>
          <Button type="submit" isLoading={criando || atualizando}>
            {isEditing ? 'Atualizar' : 'Criar'}
          </Button>
        </div>
      </form>
    </FormProvider>
  )
}
```

---

## 🔗 Referências

- React Router: https://reactrouter.com/en/main
- Zustand: https://zustand-demo.pmnd.rs
- TanStack Query: https://tanstack.com/query
- React Hook Form: https://react-hook-form.com
- Zod: https://zod.dev

# 🧩 Componentes - Padrões e Templates

> Guia completo para criação de componentes React

---

## 📋 Tipos de Componentes

### 1. UI Components (shadcn/ui)

Componentes base acessíveis usando shadcn/ui + Radix UI + Tailwind CSS.

```tsx
// src/components/ui/button.tsx (gerado pelo shadcn)
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

**Instalação de componentes:**
```bash
# Instalar componentes shadcn/ui
npx shadcn@latest add button card input label table dialog
```

### 2. Form Components (React Hook Form + shadcn/ui)

Componentes de entrada integrados com React Hook Form usando shadcn/ui.

**Instalação:**
```bash
npx shadcn@latest add form input label
```

**Uso com React Hook Form:**
```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

const formSchema = z.object({
  email: z.string().email('Email inválido'),
  password: z.string().min(6, 'Senha deve ter pelo menos 6 caracteres'),
})

export const LoginForm = () => {
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  })

  const onSubmit = (values: z.infer<typeof formSchema>) => {
    console.log(values)
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input placeholder="seu@email.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Senha</FormLabel>
              <FormControl>
                <Input type="password" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Entrar</Button>
      </form>
    </Form>
  )
}
```

### 3. Layout Components

Componentes de estrutura e layout usando shadcn/ui. **Vivem em shared/components/layout/**.

```tsx
// shared/components/layout/PageHeader.tsx
import { PropsWithChildren } from 'react'
import { Separator } from '@/shared/components/ui/separator'

interface PageHeaderProps {
  titulo: string
  subtitulo?: string
  acoes?: React.ReactNode
}

export const PageHeader = ({ titulo, subtitulo, acoes }: PageHeaderProps) => (
  <div className="flex items-center justify-between pb-6 mb-6">
    <div className="space-y-1">
      <h1 className="text-2xl font-bold tracking-tight">{titulo}</h1>
      {subtitulo && (
        <p className="text-muted-foreground">{subtitulo}</p>
      )}
    </div>
    {acoes && <div className="flex items-center gap-2">{acoes}</div>}
  </div>
)
```

**Instalação:**
```bash
npx shadcn@latest add separator
```

### 4. Feature Components

Componentes específicos de features usando shadcn/ui. **Vivem dentro da pasta da feature**.

```tsx
// features/clientes/components/ClienteCard.tsx
import { Cliente } from '../types/cliente.types'
import { Card, CardContent, CardHeader, CardTitle } from '@/shared/components/ui/card'
import { Button } from '@/shared/components/ui/button'
import { Badge } from '@/shared/components/ui/badge'
import { Edit, Trash2, Mail, Phone } from 'lucide-react'

interface ClienteCardProps {
  cliente: Cliente
  onEditar: (id: string) => void
  onExcluir: (id: string) => void
}

export const ClienteCard = ({ cliente, onEditar, onExcluir }: ClienteCardProps) => (
  <Card>
    <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
      <CardTitle className="text-lg font-semibold">{cliente.nome}</CardTitle>
      <Badge variant={cliente.ativo ? "default" : "secondary"}>
        {cliente.ativo ? "Ativo" : "Inativo"}
      </Badge>
    </CardHeader>
    <CardContent>
      <div className="space-y-2">
        <div className="flex items-center text-sm text-muted-foreground">
          <Mail className="mr-2 h-4 w-4" />
          {cliente.email}
        </div>
        {cliente.telefone && (
          <div className="flex items-center text-sm text-muted-foreground">
            <Phone className="mr-2 h-4 w-4" />
            {cliente.telefone}
          </div>
        )}
      </div>
      <div className="flex justify-end gap-2 mt-4">
        <Button
          variant="outline"
          size="sm"
          onClick={() => onEditar(cliente.id)}
        >
          <Edit className="mr-2 h-4 w-4" />
          Editar
        </Button>
        <Button
          variant="destructive"
          size="sm"
          onClick={() => onExcluir(cliente.id)}
        >
          <Trash2 className="mr-2 h-4 w-4" />
          Excluir
        </Button>
      </div>
    </CardContent>
  </Card>
)
```

**Instalação:**
```bash
npx shadcn@latest add card badge
```

---

## 📐 Padrões de Componentes

### Props Typing

```tsx
// ✅ Correto: Interface explícita
interface ButtonProps {
  variant: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
  onClick?: () => void
  children: React.ReactNode
}

// ✅ Correto: Extender HTML props
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string
  error?: string
}

// ✅ Correto: PropsWithChildren para wrappers
interface CardProps extends PropsWithChildren {
  className?: string
}

// ❌ Errado: any ou object
interface BadProps {
  data: any
  config: object
}
```

### forwardRef Pattern

Para componentes que precisam expor ref:

```tsx
import { forwardRef, InputHTMLAttributes } from 'react'

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, ...props }, ref) => (
    <div>
      <label>{label}</label>
      <input ref={ref} {...props} />
    </div>
  )
)

Input.displayName = 'Input'
```

### Compound Components

Para componentes complexos com partes relacionadas:

```tsx
// src/components/ui/Card/Card.tsx
import { PropsWithChildren, createContext, useContext } from 'react'

const CardContext = createContext<{ variant: string }>({ variant: 'default' })

const CardRoot = ({ children, variant = 'default' }: PropsWithChildren<{ variant?: string }>) => (
  <CardContext.Provider value={{ variant }}>
    <div className="rounded-lg border bg-white shadow-sm">{children}</div>
  </CardContext.Provider>
)

const CardHeader = ({ children }: PropsWithChildren) => (
  <div className="border-b p-4">{children}</div>
)

const CardBody = ({ children }: PropsWithChildren) => (
  <div className="p-4">{children}</div>
)

const CardFooter = ({ children }: PropsWithChildren) => (
  <div className="border-t p-4">{children}</div>
)

// Export como objeto
export const Card = Object.assign(CardRoot, {
  Header: CardHeader,
  Body: CardBody,
  Footer: CardFooter,
})

// Uso:
// <Card>
//   <Card.Header>Título</Card.Header>
//   <Card.Body>Conteúdo</Card.Body>
//   <Card.Footer>Ações</Card.Footer>
// </Card>
```

---

## 🎨 Estilização

### Tailwind + shadcn/ui + cn utility

O shadcn/ui já inclui o utilitário `cn()` configurado.

```typescript
// src/lib/utils.ts (gerado pelo shadcn)
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### Uso em componentes

```tsx
// ✅ Correto: cn() para composição
<div className={cn(
  'rounded-lg p-4',
  isActive && 'bg-primary text-primary-foreground',
  className  // Props externas por último
)} />

// ❌ Errado: Template strings
<div className={`base-class ${isActive ? 'active' : ''} ${className}`} />
```

### Variáveis CSS do shadcn/ui

```tsx
// ✅ Usar variáveis CSS para temas
<div className="bg-primary text-primary-foreground" />

// ✅ Personalizar cores no tailwind.config.js
// As variáveis são definidas em src/index.css
```

---

## 📁 Estrutura de Arquivos

### Componentes shadcn/ui

```
components/ui/
├── button.tsx       # Componente único (gerado pelo shadcn)
├── card.tsx
├── input.tsx
├── form.tsx
├── dialog.tsx
└── index.ts         # Re-exports manuais
```

### Componentes customizados

```
components/
├── ui/              # shadcn/ui components
├── forms/           # Form components customizados
│   └── CustomFormField/
│       ├── index.ts
│       └── CustomFormField.tsx
├── layout/          # Layout components
└── [feature]/       # Feature components
    └── cliente/
        ├── ClienteCard/
        │   ├── index.ts
        │   └── ClienteCard.tsx
        └── ClienteList/
            ├── index.ts
            └── ClienteList.tsx
```

### Re-export pattern

```typescript
// components/ui/index.ts (manual)
export * from './button'
export * from './card'
export * from './input'

// components/cliente/index.ts
export * from './ClienteCard'
export * from './ClienteList'
```

---

## ✅ Checklist de Componentes

| Item | Verificação |
|------|-------------|
| shadcn/ui | ✅ Usar componentes oficiais quando disponíveis |
| Props | ✅ Tipadas explicitamente |
| className | ✅ Aceita via props usando cn() |
| Ref | ✅ forwardRef se necessário |
| DisplayName | ✅ Definido para debug |
| Re-export | ✅ index.ts na pasta |
| Testes | ✅ .test.tsx junto |
| Acessibilidade | ✅ Radix UI primitives |

---

## 🔗 Referências

- React TypeScript Cheatsheet: https://react-typescript-cheatsheet.netlify.app
- shadcn/ui: https://ui.shadcn.com
- Radix UI: https://www.radix-ui.com
- Tailwind CSS: https://tailwindcss.com/docs

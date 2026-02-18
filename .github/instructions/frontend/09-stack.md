# 📋 Frontend Requirements - Technical Specifications

> ⚠️ **Fonte Única de Verdade** para todas as definições técnicas do frontend

---

## 🎯 Stack Tecnológico

### Runtime & Framework
| Tecnologia | Versão | Status |
|------------|--------|--------|
| **Node.js** | `20.x LTS` | ✅ Runtime |
| **TypeScript** | `5.4+` | ✅ Ativo |
| **React** | `18.3+` | ✅ UI Library |
| **Vite** | `5.4+` | ✅ Build Tool |

### Bibliotecas Core
| Tecnologia | Versão | Uso |
|------------|--------|-----|
| **React Router DOM** | `6.26+` | Roteamento SPA |
| **React Hook Form** | `7.53+` | Gestão de Formulários |
| **Zod** | `3.23+` | Validação de Schemas |
| **@hookform/resolvers** | `3.9+` | Integração RHF + Zod |

---

## 📦 Dependências NPM - Versões Oficiais

### 🏛️ **Core Dependencies**
```bash
# React + Vite
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install

# Routing
npm install react-router-dom@^6.26

# Forms + Validation
npm install react-hook-form@^7.53 zod@^3.23 @hookform/resolvers@^3.9
```

### 🎨 **UI & Styling**
| Package | Versão | Uso |
|---------|--------|-----|
| `tailwindcss` | `3.4+` | Utility-first CSS |
| `shadcn/ui` | `latest` | Componentes UI acessíveis |
| `lucide-react` | `0.441+` | Ícones SVG |
| `@radix-ui/react-*` | `latest` | Primitivos acessíveis (via shadcn) |

```bash
# Tailwind CSS + shadcn/ui
npm install -D tailwindcss@^3.4 postcss autoprefixer tailwindcss-animate@^1.0

# shadcn/ui
npx shadcn@latest init --yes
# Instala componentes conforme necessário:
npx shadcn@latest add button input card dialog table

# Icons
npm install lucide-react@^0.441
```

### 🌐 **HTTP Client & State**
| Package | Versão | Uso |
|---------|--------|-----|
| `axios` | `1.7+` | HTTP Client |
| `@tanstack/react-query` | `5.56+` | Server State Management |
| `zustand` | `4.5+` | Client State Management |

```bash
# Data Fetching
npm install axios@^1.7 @tanstack/react-query@^5.56

# State Management
npm install zustand@^4.5
```

### 🧪 **Testing**
| Package | Versão | Uso |
|---------|--------|-----|
| `vitest` | `2.1+` | Test Runner |
| `@testing-library/react` | `16.0+` | React Testing |
| `@testing-library/jest-dom` | `6.5+` | DOM Assertions |
| `@testing-library/user-event` | `14.5+` | User Interactions |
| `msw` | `2.4+` | API Mocking |

```bash
# Testing
npm install -D vitest@^2.1 @testing-library/react@^16.0 @testing-library/jest-dom@^6.5 @testing-library/user-event@^14.5 msw@^2.4 jsdom@^25.0
```

### 🔧 **Dev Tools & Linting**
| Package | Versão | Uso |
|---------|--------|-----|
| `eslint` | `9.10+` | Linting |
| `@typescript-eslint/eslint-plugin` | `8.5+` | TS Rules |
| `prettier` | `3.3+` | Formatting |
| `eslint-plugin-react-hooks` | `4.6+` | React Hooks Rules |

```bash
# Linting & Formatting (já incluídos no template Vite)
npm install -D eslint@^9.10 prettier@^3.3 eslint-plugin-react-hooks@^4.6
```

---

## 🏗️ Configurações de Projeto

### TypeScript (tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "useDefineForClassFields": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"]
}
```

### Vite (vite.config.ts)
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
      },
    },
  },
})
```

### Tailwind (tailwind.config.js)
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ["class"],
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### CSS Global (src/index.css)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96%;
    --secondary-foreground: 222.2 84% 4.9%;
    --muted: 210 40% 96%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96%;
    --accent-foreground: 222.2 84% 4.9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 84% 4.9%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 94.1%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

---

## 📁 Estrutura de Pastas (Resumo)

```
src/frontend/
├── public/
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── assets/
│   ├── components/
│   ├── pages/
│   ├── routes/
│   ├── hooks/
│   ├── services/
│   ├── stores/
│   ├── types/
│   ├── utils/
│   └── config/
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.js
```

> 📖 Ver detalhes completos em: `.github/rules/frontend/04-folder-structure.md`

---

## 🔗 Referências

| Contexto | Arquivo |
|----------|---------|
| **📂 Estrutura de Pastas** | `.github/rules/frontend/04-folder-structure.md` |
| **🧩 Componentes** | `.github/rules/frontend/02-components.md` |
| **🛣️ Rotas & State** | `.github/rules/frontend/03-state-routing.md` |
| **📝 Convenções** | `.github/rules/frontend/05-conventions.md` |
| **🔒 Segurança** | `.github/rules/frontend/00-security.md` |

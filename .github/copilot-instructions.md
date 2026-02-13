---
applyTo: '**'
---

# Instruções Gerais do Monorepo

## Estrutura do Projeto

| Caminho | Stack | Status |
|---------|-------|--------|
| `/src/backend` | [📋 Ver requisitos técnicos](backend-requirements.md) | ✅ Ativo |
| `/src/frontend` | [📋 Ver requisitos técnicos](frontend-requirements.md) | ✅ Ativo |

> 🎯 **Stack Backend**: Todas as versões e dependências definidas em [backend-requirements.md](backend-requirements.md)
> 🎯 **Stack Frontend**: Todas as versões e dependências definidas em [frontend-requirements.md](frontend-requirements.md)

---

## Glossário

| Termo | Descrição |
|-------|-----------|
| **SPEC** | Documento unificado (User Story + especificação técnica) |
| **EPIC** | Agrupamento lógico de Feature Specs relacionadas |
| **RN** | Regra de Negócio (mapeada na Spec) |
| **CA** | Critério de Aceitação (Dado/Quando/Então) |
| **RF** | Requisito Funcional |
| **EP** | Endpoint da API RESTful (ex: EP-001: POST /api/vendedores) |
| **DTO** | Data Transfer Object (Request/Response) |

---

## Fluxo de Desenvolvimento

```
@spec-writer → @backend-api → @unittest-writer → @frontend-dev
```

1. **@spec-writer**: Feature Spec → `docs/specs/SPEC-XXX-nome.md`
2. **@backend-api**: Domain → Application → Infrastructure → API
3. **@unittest-writer**: Testes Domain + Application
4. **@frontend-dev**: Pages → Components → Hooks → Services

---

## Convenções Gerais

- **Commits**: Conventional Commits (`feat:`, `fix:`, `docs:`)
- **Branches**: `feature/*`, `bugfix/*`, `hotfix/*`
- **APIs**: RESTful + JWT Bearer tokens
- **DTOs**: Request/Response separados por caso de uso

---

## 🔗 Referências (Leitura Obrigatória)

| Contexto | Arquivo |
|----------|---------|
| **📋 Stack & Versões Backend** | [backend-requirements.md](backend-requirements.md) |
| **📋 Stack & Versões Frontend** | [frontend-requirements.md](frontend-requirements.md) |
| **📂 Estrutura Backend** | `.github/rules/backend/04-folder-structure.md` |
| **📂 Estrutura Frontend** | `.github/rules/frontend/04-folder-structure.md` |
| **Agents e Prompts** | `.github/agent.md` |
| **Segurança Backend** | `.github/rules/backend/00-security.md` |
| **Segurança Frontend** | `.github/rules/frontend/00-security.md` |
| **Backend (índice)** | `.github/rules/backend/copilot-instructions.md` |
| **Frontend (índice)** | `.github/rules/frontend/copilot-instructions.md` |

> ⚠️ **Regra**: Antes de criar código, ler os arquivos de rules na ordem indicada.
> ⚠️ **Obrigatório**: Sempre seguir estrutura definida em `04-folder-structure.md`
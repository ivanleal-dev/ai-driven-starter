# AI-Driven Starter

## 🎯 Objetivo

Monorepo template completo para desenvolvimento de aplicações full-stack modernas com arquitetura bem definida, padrões claros e fluxo de trabalho estruturado.

---

## 📊 Fluxo de Desenvolvimento

```
Spec → Backend API → Testes Unit → Frontend
```

1. **@spec-writer**: Cria Feature Spec (User Story + Especificação Técnica)
2. **@backend-api**: Implementa API RESTful (Domain → Application → Infrastructure)
3. **@unittest-writer**: Testes para Domain e Application
4. **@frontend-dev**: Implementa Pages, Components, Hooks e Services

---

## 📂 Estrutura do Projeto

```
ai-driven-starter/
├── src/
│   ├── backend/          # API RESTful com arquitetura em camadas
│   └── frontend/         # Aplicação web moderna
├── .github/
│   ├── rules/            # Convenções e padrões do projeto
│   ├── skills/           # Documentação de roles e responsabilidades
│   └── instructions/     # Guias de implementação
└── docs/
    └── specs/            # Feature Specifications
```

---

## 🚀 Stack

- **Backend**: Asp.net Core, arquitetura em camadas (Domain → Application → Infrastructure)
- **Frontend**: React/TypeScript com componentes reutilizáveis
- **API**: RESTful com autenticação JWT
- **Versionamento**: Conventional Commits

---

## 📋 Glossário Rápido

| Termo | Descrição |
|-------|-----------|
| **SPEC** | User Story + Especificação Técnica unificada |
| **EP** | Endpoint da API (ex: `POST /api/users`) |
| **DTO** | Objetos de transferência de dados (Request/Response) |
| **RN** | Regra de Negócio |
| **CA** | Critério de Aceitação |

---

## 📖 Próximos Passos

1. Leia [`copilot-instructions.md`](.github/instructions/copilot-instructions.md) para entender as convenções
2. Verifique os requisitos técnicos em `09-stack.md`
3. Siga a estrutura de pastas definida em `.github/rules/`
4. Inicie novo desenvolvimento com uma Feature Spec

---

**Desenvolvido com o fluxo estruturado para máxima qualidade e escalabilidade.**

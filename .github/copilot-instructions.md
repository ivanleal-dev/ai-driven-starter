---
applyTo: '**'
---

# Instruções Gerais do Monorepo

## Estrutura do Projeto
Este é um monorepo com duas aplicações principais:
- `/src/backend` - API ASP.NET Core (.NET)
- `/src/frontend` - Aplicação React com TypeScript

Se não houver nome definido, pergunte ao usuário antes de gerar código backend.

## Glossário

- **PRD (Bloco de Conhecimento Integrado)**: Agrupamento lógico de User Stories relacionadas (ex: PRD-0001-gestao-vendedores)
- **US (User Story)**: Item de backlog que descreve funcionalidade do ponto de vista do usuário (ex: US001)
- **PRD (Product Requirements Document)**: Documento técnico detalhado de uma User Story
- **RF (Requisito Funcional)**: Funcionalidade específica a ser implementada (mapeado no PRD)
- **EP (Endpoint)**: Rota da API RESTful (ex: EP-001: POST /api/vendedores)
- **DTO (Data Transfer Object)**: Objeto de transferência de dados entre camadas (Request/Response)
- **FRONT**: Item de implementação frontend (mapeado no PRD)

## Fluxo de Desenvolvimento Padrão

1. **@story-writer**: Criar User Story estruturada
   - Salva em: `docs/stories/USXXX-nome-funcionalidade.md`
   
2. **@prd-writer**: Gerar PRD técnico baseado na US
   - Salva em: `docs/product-requirements/PRD-XXXX-slug/USYYY/PRD-USYYY-nome-YYYY-MM-DD.md`
   - Define: API Contracts, Implementation Status, Diagramas
   
3. **@backend-api**: Implementar backend seguindo Clean Architecture
   - Gera: Domain (Entities, ValueObjects), Application (Handlers, Validators), Infrastructure (EF Config, Repositories, Migrations), API (Controllers)
   
4. **@unittest-writer**: Criar testes unitários (Domain + Application)
   - Gera: `tests/<Solution>.UnitTests/Domain/` e `/Application/`
   
5. **Frontend**: (em planejamento)

## Convenções Gerais
- Commits seguem Conventional Commits (feat:, fix:, docs:, etc)
- Branches: feature/*, bugfix/*, hotfix/*
- Sempre incluir testes unitários
- Documentar APIs e componentes complexos

## Comunicação Frontend-Backend
- APIs RESTful seguindo padrões REST
- Usar DTOs para transferência de dados
- Não incluir versionamento de API neste escopo
- Autenticação via JWT Bearer tokens

## Prompts e Agents Disponíveis

Os agents customizados utilizam prompts especializados:

| Agent | Prompt | Descrição |
|-------|--------|-------------|
| @story-writer | `.github/prompts/story.writter.prompt.md` | Gera User Stories estruturadas |
| @prd-writer | `.github/prompts/prd.writter.prompt.md` | Gera PRDs técnicos com PRD |
| @backend-api | `.github/prompts/backend.writter.prompt.md` | Implementa backend Clean Architecture |
| @unittest-writer | `.github/prompts/unittest.writter.prompt.md` | Gera testes unitários Domain/Application |

Ver definições completas em: `.github/agent.md`

## Para instruções específicas:
Ao criar código para o backend, o agente deve obrigatoriamente ler e seguir também as instruções detalhadas em `/src/backend/.github/copilot-instructions.md`, além deste arquivo geral. Para frontend, seguir `/src/frontend/.github/copilot-instructions.md`.

## **Agent Guidance — Quickstart**
- **Target runtime**: projects target `net10.0` (check `src/backend/*/*.csproj`).
- **Primary solutions**: root `<BackendSolution>.sln`, backend `src/backend/<BackendSolution>.sln`.
 - **Primary solutions**: The agent requires the user to provide the root and backend solution paths/names before running build or run commands. You can provide these in one of the following ways (preferred order):
    - **Environment variables**: set `ROOT_SOLUTION_PATH` and `BACKEND_SOLUTION_PATH` to the solution file paths (relative or absolute).
    - **Repository config file**: add a `.solutions.json` at the repository root with the shape:

       ```json
       {
          "root": "<path/to/root.sln>",
          "backend": "src/backend/<BackendSolution>.sln"
       }
       ```
    - **Interactive prompt**: if neither the env vars nor `.solutions.json` exist, the agent will prompt you to enter the two paths. The agent will not assume solution names automatically without explicit confirmation.
 - **Fallback (not recommended)**: if the user does not provide values, the agent may attempt to detect `*.sln` files under the repo root and `src/backend/`, but it will always ask you to confirm the chosen files before using them.
 - **Build locally**: run (replace `<root-sln-path>` with the provided root solution path)

    ```powershell
    dotnet restore <root-sln-path>
    dotnet build <root-sln-path> -c Debug
    ```
 - **Run API**: run the backend API project (replace `<backend-project-path>` with the provided backend project path under `src/backend`, e.g. `src/backend/<BackendSolution>.API`)

    ```powershell
    dotnet run --project <backend-project-path>
    ```
- **Key patterns to follow**: No `MediatR`/Mediator; prefer vertical-slice Handlers under `<BackendSolution>.Application/UseCases/*` and the Result pattern in `<BackendSolution>.Application/Common`.
- **Important interfaces/implementations**: `<BackendSolution>.Domain/Interfaces/IUsuarioRepository.cs` and `<BackendSolution>.Infrastructure/Repositories/UsuarioRepository.cs` (repository pattern + EF Core).
- **DI registration**: check `Extensions/ServiceCollectionExtensions.cs` in API and Application for handler and repository registrations.
- **When updating code**: preserve Clean Architecture boundaries — Domain must not reference Infrastructure; Application coordinates use cases; Infrastructure contains EF Core and repository implementations.
- **Where to look first**: `Program.cs`, `Controllers/v1/UsuariosController.cs`, `Application/UseCases/Usuario/*`, `Domain/Entities/Usuario.cs`, `Infrastructure/Repositories/UsuarioRepository.cs`.
- **If you need SDK pinning info**: look for `global.json` (none present currently) and update CI/Dockerfiles if they pin .NET SDK.

If any of these targets or files are out-of-date, ask before making cross-cutting changes.
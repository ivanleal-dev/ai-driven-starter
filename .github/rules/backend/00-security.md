# Regras de Segurança - Backend

> Este arquivo centraliza **TODAS** as regras de segurança para operações do backend.

---

## 🔒 Princípios Gerais

1. **Confirmação explícita** para operações destrutivas
2. **Não assumir** configurações sensíveis
3. **Documentar** comandos antes de executar

---

## ⚠️ Migrations - REGRA CRÍTICA

A LLM **NUNCA** deve executar comandos de migration automaticamente.

### Fluxo Obrigatório

1. **Gerar código da migration** (apenas arquivos `.cs`)
2. **Notificar o usuário**:
   ```
   ⚠️ ATENÇÃO: Migration criada mas NÃO aplicada ao banco de dados.
   
   📋 Para aplicar manualmente:
   cd src/backend/<SolutionName>.Infrastructure
   dotnet ef migrations add {NomeMigration} --startup-project ../<SolutionName>.API
   dotnet ef database update --startup-project ../<SolutionName>.API
   
   ❓ Deseja que eu execute estes comandos agora? (Requer confirmação explícita)
   ```
3. **Aguardar confirmação explícita** do usuário

### Comandos Proibidos Sem Confirmação

| Comando | Risco |
|---------|-------|
| `dotnet ef migrations add` | Cria arquivos no projeto |
| `dotnet ef database update` | Altera schema do banco |
| `dotnet ef migrations remove` | Remove arquivos de migration |
| `dotnet ef database drop` | **DESTRUTIVO** - apaga banco inteiro |

### Exceções (Permitidos)

- ✅ Gerar arquivos de configuração EF (`*Configuration.cs`)
- ✅ Criar classes de entidade (`.cs`)
- ✅ Documentar comandos de migration (sem executar)

---

## 🏭 Criação de Solution

Antes de criar qualquer projeto backend, **DEVE**:

### Verificação

```powershell
# Procurar *.sln em src/backend/
Get-ChildItem -Path src/backend -Filter *.sln* -Recurse
```

### Se Solution NÃO Existir

```
🆕 Nenhuma solution backend detectada.

📋 Para criar a estrutura completa do backend, preciso do nome da solution.

❓ Qual o nome do projeto? (Ex: AgenteViagem, ControleEstoque, SistemaVendas)

Este nome será usado para:
- <SolutionName>.sln
- <SolutionName>.Domain
- <SolutionName>.Application
- <SolutionName>.Infrastructure
- <SolutionName>.API
```

### Confirmação Obrigatória

```
✅ Confirma criação da solution "<SolutionName>"? (sim/não)

Estrutura a ser criada:
src/backend/
├── <SolutionName>.sln
├── <SolutionName>.Domain/
├── <SolutionName>.Application/
├── <SolutionName>.Infrastructure/
└── <SolutionName>.API/
```

### Comandos que Requerem Confirmação

- ❌ `dotnet new sln`
- ❌ `dotnet new classlib`
- ❌ `dotnet new webapi`

### Se Solution Existir

- ✅ Usar o nome detectado automaticamente
- ✅ Informar: `📂 Solution detectada: <NomeEncontrado>`

---

## 🗄️ Operações DDL (Banco de Dados)

Qualquer operação que modifique o schema do banco requer aprovação prévia:

| Operação | Confirmação |
|----------|-------------|
| `CREATE TABLE` | ✅ Obrigatória |
| `ALTER TABLE` | ✅ Obrigatória |
| `DROP TABLE` | ✅ Obrigatória + Aviso DESTRUTIVO |
| `CREATE INDEX` | ✅ Obrigatória |
| `DROP INDEX` | ✅ Obrigatória |
| `TRUNCATE` | ✅ Obrigatória + Aviso DESTRUTIVO |

---

## ✅ Resumo de Permissões

| Ação | Permitido | Requer Confirmação |
|------|-----------|-------------------|
| Gerar código `.cs` | ✅ | ❌ |
| Gerar configurações EF | ✅ | ❌ |
| Documentar comandos | ✅ | ❌ |
| Criar solution/projetos | ✅ | ✅ |
| Executar migrations | ✅ | ✅ |
| Operações DDL | ✅ | ✅ |
| `database drop` | ⚠️ DESTRUTIVO | ✅✅ (dupla) |

---

## Referências

- Setup inicial: `07-setup.md`
- Database patterns: `05-database.md`

# Exercício 1 — Fundamentos do Lakebase

🎯 **Modo Demo:** obrigatório

Base para todos os outros exercícios. Aqui você cria a instância Lakebase, conecta, faz operações CRUD básicas, explora branching + PITR e registra no Unity Catalog.

## O que você vai aprender

- Hierarquia Lakebase: **Project → Branch → Endpoint**
- Como provisionar uma instância via UI em ~2 min
- Conectar via Databricks SQL Editor com OAuth (sem senha estática)
- Criar database, schema e tabelas
- Usar **branches** como "git para dados"
- Restaurar dados com **PITR** (Point-in-Time Recovery)
- Registrar o Lakebase no Unity Catalog para governança unificada

## Pré-requisitos

- ✅ Workspace Databricks com Lakebase habilitado
- ✅ Permissão para criar instância Lakebase
- ✅ Catalog disponível no Unity Catalog (default: `dbacademy`)

---

## Parte 1 — Criar a instância Lakebase

### Objetivo

Criar uma instância Lakebase — o "projeto" base que vamos usar nas próximas partes. Tudo via UI, sem terminal.

### 🖱️ Passo 1: Acessar o Lakebase

1. No menu lateral do workspace, clique em **Compute**
2. Na barra de abas no topo, clique em **Lakebase**

> Se a aba não aparecer, seu workspace pode não ter Lakebase habilitado. Confirme com o admin que é um workspace **serverless** com Lakebase ativado.

### 🖱️ Passo 2: Criar a instância

1. Clique no botão **Create** (canto superior direito)
2. Em **Display name**, digite `lakebase-demo-<seu_database>` (substitua `<seu_database>` pelo seu identificador único)
3. Clique em **Create**

### ⏳ Passo 3: Aguardar o provisionamento

A instância fica pronta em **alguns segundos**. Quando o compute aparecer como `Active`, ela está pronta para receber conexões.

> 💡 Por baixo dos panos, a Databricks criou três coisas:
> - **Projeto** `lakebase-demo-<seu_database>` (o container)
> - **Branch** `production` (a base de dados primária — marcada como `Default`)
> - **Compute** `primary` (o compute que aceita conexões)

### 🖱️ Passo 4: Explorar o Project Dashboard

Clique no nome `lakebase-demo-<seu_database>` na lista para abrir a página **Project dashboard**. Você verá 3 painéis:

- **Monitoring** — gráfico de uso de CU e RAM ao longo do tempo, com seletor de branch e compute
- **1 Branch** — lista das branches existentes (só `production` por enquanto), capacidade do compute e quem criou
- **Project settings** — informações da instância:
  - **Region** — onde está rodando (ex: `AWS (us-east-1)`)
  - **Default compute size** — capacidade default (ex: `2 ↔ 4 CU`)
  - **History retention** — janela de PITR (padrão `7 days`)
  - **Postgres version** — versão do Postgres (atualmente `17`)

### 🖱️ Passo 5: Obter a string de conexão

1. Clique no botão **Connect** (canto superior direito)
2. Uma janela vai abrir com a string de conexão e as credenciais (OAuth token) para o compute `primary`
3. Não precisa anotar nada agora — vamos voltar aqui na próxima parte quando formos conectar via SQL Editor

### O que esperar

✅ **Sucesso:** Instância `lakebase-demo-<seu_database>` com status `ACTIVE`, branch `production`, endpoint `primary`.

❌ **Possíveis problemas:**

| Erro | Causa | Solução |
|---|---|---|
| Botão "Create" não aparece | Sem permissão | Pedir grant de criar instância Lakebase ao admin |
| `Lakebase not available in this workspace` | Workspace não habilitado | Workspace precisa ser tipo serverless com Lakebase ON |
| Status fica `FAILED` | Quota da conta esgotada | Verificar limites em Account Console |

### 🧪 Try it yourself (modo treinamento)

**Exercício:** Tente clicar em **Endpoints → Create endpoint**. Que tipos de endpoint estão disponíveis?

<details>
<summary>Resposta</summary>

`READ_WRITE` (read-write, só 1 por branch) e `READ_ONLY` (read replica, vários por branch). Usaremos `READ_ONLY` no Exercício 5 pra mostrar escala horizontal de leitura.
</details>

---

## Parte 2 — Conectar e criar schema *(em construção)*

## Parte 3 — Branching + PITR *(em construção)*

## Parte 4 — Registrar no Unity Catalog *(em construção)*

---

## Próximo exercício

➡️ [`../02_seguranca/`](../02_seguranca/) — Row-Level Security + Column Masking via UC

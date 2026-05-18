# 01.01 — Criar a instância Lakebase

🎯 **Modo Demo:** obrigatório

## Objetivo

Criar uma instância Lakebase Autoscaling — o "projeto" base que vamos usar nos próximos cenários. Tudo via UI, sem terminal.

## O que você vai aprender

- Hierarquia Lakebase: **Project → Branch → Endpoint**
- Como criar uma instância em segundos via UI
- O que o autoscaling faz por trás dos panos

## Pré-requisitos

- ✅ Workspace Databricks com Lakebase habilitado
- ✅ Permissão para criar Database Instance

## Passo a passo

### 🖱️ Passo 1: Abrir a área de Database Instances

1. No menu lateral do workspace, clique em **Compute**
2. Na barra de abas no topo, clique em **Database Instances**

> Se a aba não aparecer, seu workspace pode não ter Lakebase habilitado. Confirme com o admin que é um workspace **serverless** com Lakebase ativado.

### 🖱️ Passo 2: Criar o projeto

1. Clique no botão **Create database instance** (canto superior direito)
2. Preencha:
   - **Name:** `lakebase-demo` (ou outro nome — anote para usar nos próximos passos)
   - **Tier:** `Autoscaling` (sempre prefira; o Provisioned é legado)
   - **Capacity (min / max CU):** `0.5 / 2`
     - 0.5 CU é o mínimo possível — para scale-to-zero futuro
     - 2 CU é o máximo para a demo (suficiente pra simular carga de pedidos)
3. Clique em **Create**

### ⏳ Passo 3: Aguardar o provisionamento

A instância leva **1 a 2 minutos** para ficar pronta. Você vai ver os estados:
- `CREATING` → `STARTING` → `ACTIVE`

Quando ficar `ACTIVE`, ela está pronta para receber conexões.

> 💡 Por baixo dos panos, a Databricks criou três coisas:
> - **Projeto** `lakebase-demo` (o container)
> - **Branch** `production` (a base de dados primária)
> - **Endpoint** `primary` do tipo READ_WRITE (o endpoint que aceita conexões)

### 🖱️ Passo 4: Verificar os detalhes

1. Clique no nome `lakebase-demo` na lista para abrir a página da instância
2. Você verá 4 abas principais:
   - **Overview** — status, capacidade, custo estimado
   - **Branches** — `production` (em breve criaremos `dev`)
   - **Endpoints** — `primary` (READ_WRITE, 0.5-2 CU)
   - **Settings** — escalar, deletar, configurações avançadas

### 🖱️ Passo 5: Anotar o hostname (vamos usar depois)

1. Na aba **Endpoints**, clique em `primary`
2. Copie o **Host** (algo como `lakebase-demo.xxxxxx.databricks.com`)
3. Guarde em algum lugar — vamos colar no próximo guia

## O que esperar

✅ **Sucesso:** Instância `lakebase-demo` com status `ACTIVE`, branch `production`, endpoint `primary`.

❌ **Possíveis problemas:**

| Erro | Causa | Solução |
|---|---|---|
| Botão "Create" não aparece | Sem permissão | Pedir grant `CREATE DATABASE INSTANCE` ao admin |
| `Lakebase not available in this workspace` | Workspace não habilitado | Workspace precisa ser tipo serverless com Lakebase ON |
| Status fica `FAILED` | Quota da conta esgotada | Verificar limites em Account Console |

## 🧪 Try it yourself (modo treinamento)

**Exercício:** Tente clicar em **Endpoints → Create endpoint**. Que tipos de endpoint estão disponíveis?

<details>
<summary>Resposta</summary>

`READ_WRITE` (read-write, só 1 por branch) e `READ_ONLY` (read replica, vários por branch). Usaremos `READ_ONLY` no Cenário 5 pra mostrar escala horizontal de leitura.
</details>

## Próximo passo

➡️ [`02_schema_e_crud.md`](02_schema_e_crud.md) — Vamos conectar via SQL Editor e criar nosso primeiro database + tabelas.

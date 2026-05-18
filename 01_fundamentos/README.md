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
- ✅ Permissão para criar Database Instance
- ✅ Catalog disponível no Unity Catalog (default: `dbacademy`)

---

## Parte 1 — Criar a instância Lakebase

### Objetivo

Criar uma instância Lakebase Autoscaling — o "projeto" base que vamos usar nas próximas partes. Tudo via UI, sem terminal.

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
3. Guarde em algum lugar — vamos colar na próxima parte

### O que esperar

✅ **Sucesso:** Instância `lakebase-demo` com status `ACTIVE`, branch `production`, endpoint `primary`.

❌ **Possíveis problemas:**

| Erro | Causa | Solução |
|---|---|---|
| Botão "Create" não aparece | Sem permissão | Pedir grant `CREATE DATABASE INSTANCE` ao admin |
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

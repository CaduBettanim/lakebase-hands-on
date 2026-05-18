# Exercício 1 — Fundamentos do Lakebase

🎯 **Modo Demo:** obrigatório

Base para todos os outros exercícios. Aqui você cria a instância Lakebase, conecta, faz operações CRUD básicas, explora branching + PITR e registra no Unity Catalog.

## O que você vai aprender

- Hierarquia Lakebase: **Project → Branch → Compute**
- Como provisionar uma instância via UI em segundos
- Conectar via Databricks SQL Editor
- Criar database, schema e tabelas
- Usar **branches** como "git para dados" (copy-on-write)
- Restaurar dados com **PITR** (Point-in-Time Restore)
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
> - **Database** `databricks_postgres` (database Postgres default criado dentro da branch)

### 🖱️ Passo 4: Explorar o Project dashboard

Clique no nome `lakebase-demo-<seu_database>` na lista para abrir a página **Project dashboard**. Você verá 3 painéis:

- **Monitoring** — gráfico de uso de CU e RAM ao longo do tempo, com seletor de branch e compute
- **1 Branch** — lista das branches existentes (só `production` por enquanto), capacidade do compute e quem criou
- **Project settings** — informações da instância:
  - **Region** — onde está rodando (ex: `AWS (us-east-1)`)
  - **Default compute size** — capacidade default (ex: `2 ↔ 4 CU`)
  - **History retention** — janela de PITR (padrão `7 days`, configurável de 2 a 30)
  - **Postgres version** — versão do Postgres (atualmente `17`)

### 🖱️ Passo 5: Conhecer o botão Connect

1. Clique no botão **Connect** (canto superior direito)
2. Uma janela vai abrir com a string de conexão e as credenciais para o compute `primary`
3. Aqui você pode ver formatos prontos para psql, JDBC, Python e outras ferramentas
4. Não precisa anotar nada agora — vamos voltar aqui na próxima parte

### O que esperar

✅ **Sucesso:** Instância `lakebase-demo-<seu_database>` com status `Active`, branch `production`, compute `primary`.

❌ **Possíveis problemas:**

| Erro | Causa | Solução |
|---|---|---|
| Botão "Create" não aparece | Sem permissão | Pedir grant de criar instância Lakebase ao admin |
| `Lakebase not available in this workspace` | Workspace não habilitado | Workspace precisa ser tipo serverless com Lakebase ON |
| Status fica `FAILED` | Quota da conta esgotada | Verificar limites em Account Console |

---

## Parte 2 — Conectar e criar schema

### Objetivo

Conectar ao seu Lakebase via Databricks SQL Editor (sem terminal, sem cliente externo) e criar um schema básico — um pequeno modelo de `customers` que vamos usar nas próximas partes.

### 🖱️ Passo 1: Abrir o SQL Editor do projeto

Ainda na página **Project dashboard** da sua instância Lakebase:

1. No menu lateral do projeto, clique em **SQL Editor**
2. No topo do editor, selecione:
   - **Branch:** `production`
   - **Database:** `databricks_postgres` (database default criado pela Databricks)

> 💡 O SQL Editor do Lakebase reutiliza sua identidade Databricks automaticamente — sem precisar gerar token OAuth manual. É o jeito mais rápido de rodar SQL contra o Postgres.

### 💻 Passo 2: Verificar a conexão

Cole o SQL abaixo e clique em **Run**:

```sql
SELECT version();
```

Você deve ver algo como `PostgreSQL 17.x ...` — confirmação de que está realmente conectado a um Postgres 17.

### 💻 Passo 3: Criar schema e tabela

```sql
CREATE SCHEMA IF NOT EXISTS demo;

CREATE TABLE IF NOT EXISTS demo.customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    signed_up_at TIMESTAMPTZ DEFAULT NOW()
);
```

Resultado esperado: `CREATE SCHEMA` e `CREATE TABLE` sem erros.

### 💻 Passo 4: Inserir dados

```sql
INSERT INTO demo.customers (name, email) VALUES
    ('Alice Silva', 'alice@example.com'),
    ('Bruno Souza', 'bruno@example.com'),
    ('Carla Mendes', 'carla@example.com');

SELECT * FROM demo.customers;
```

Você deve ver 3 linhas com IDs gerados automaticamente (1, 2, 3) e timestamps preenchidos.

### 🧪 Try it yourself (modo treinamento)

**Exercício 1** — Use o meta-comando psql `\dt demo.*` para listar todas as tabelas do schema `demo`. O SQL Editor do Lakebase suporta meta-comandos psql (`\dt`, `\d`, `\l`).

**Exercício 2** — Clique em **Explain** ao invés de **Run** para a query `SELECT * FROM demo.customers`. Veja o plano de execução do Postgres. Depois clique em **Analyze** — agora vê tempo real de execução por nó do plano.

---

## Parte 3 — Branching + Point-in-Time Restore

### Objetivo

Provar duas das features mais únicas do Lakebase: **branches copy-on-write** (criar cópia da base em segundos) e **PITR** (voltar a base no tempo).

### 🖱️ Passo 1: Criar uma branch chamada `dev`

1. No menu lateral do projeto, clique em **Branches**
2. Clique no botão **New branch**
3. Configure:
   - **Name:** `dev`
   - **Parent branch:** `production`
   - **Initialize from:** *current state* (ou escolha um ponto no passado se preferir)
4. Clique em **Create branch**

A branch é criada **instantaneamente**, copy-on-write:

> 💡 A branch `dev` herda schema e dados de `production` mas compartilha o storage subjacente. Só quando você modificar dados em `dev`, novos blocos são escritos. Isso permite criar branch de uma base de 1TB em segundos, sem custo extra de storage até divergir.

### 💻 Passo 2: Modificar dados na branch `dev` (sem afetar `production`)

Volte ao **SQL Editor** e mude o seletor de **Branch** para `dev`:

```sql
DELETE FROM demo.customers WHERE name = 'Alice Silva';
INSERT INTO demo.customers (name, email) VALUES ('Diego Lima', 'diego@example.com');

SELECT * FROM demo.customers ORDER BY id;
```

Agora você tem Bruno, Carla, Diego em `dev`.

### 💻 Passo 3: Confirmar que `production` está intacta

Mude o seletor de **Branch** de volta para `production` e rode:

```sql
SELECT * FROM demo.customers ORDER BY id;
```

Você ainda vê Alice, Bruno, Carla. **A modificação em `dev` não vazou para `production`.** Essa é a mágica.

### 🖱️ Passo 4: Point-in-Time Restore — restaurar `production` para 5 minutos atrás

Suponha que alguém acabou de apagar uma linha por engano em `production`. Vamos testar a restauração:

1. Em `production`, rode: `DELETE FROM demo.customers WHERE name = 'Bruno Souza';`
2. Confirme: `SELECT * FROM demo.customers;` — Bruno sumiu
3. No menu lateral do projeto, clique em **Backup & Restore**
4. Na seção **Restore from history**:
   - **Source branch:** `production`
   - **Date/time:** selecione ~5 minutos atrás (antes do DELETE)
5. Clique em **Preview data** para confirmar que Bruno volta a aparecer
6. Clique em **Restore** → confirme em **Restore** novamente

> ⚠️ **Importante:** O PITR cria uma **nova branch raiz** com os dados restaurados — sua branch `production` original não é sobrescrita. Você precisa migrar os dados ou apontar suas aplicações para a nova branch.
> 
> Limite: até 3 branches raiz por projeto.

### 🧪 Try it yourself (modo treinamento)

**Exercício** — Em **Project settings**, clique em **Manage** e ajuste o **History window** para 14 dias. Note como isso aumenta o storage estimado.

---

## Parte 4 — Registrar no Unity Catalog

### Objetivo

Tornar as tabelas Lakebase visíveis no Unity Catalog — para que possam ser consultadas via Databricks SQL, recebam políticas de governança (RLS, column masking) e apareçam na audit log unificada.

### 🖱️ Passo 1: Abrir o Catalog Explorer

No menu lateral do workspace (não do projeto), clique em **Catalog**.

### 🖱️ Passo 2: Registrar o Lakebase como Database Catalog

1. No topo do Catalog Explorer, clique no botão **+** ou **Create** → **Database Catalog**
2. Configure:
   - **Catalog name:** `lakebase_demo_<seu_database>` *(será o nome dentro do UC)*
   - **Lakebase instance:** selecione `lakebase-demo-<seu_database>` no dropdown
   - **Lakebase database:** `databricks_postgres`
3. Clique em **Create**

> ⚠️ A localização exata do botão pode variar conforme a versão do Catalog Explorer. Se não encontrar "Database Catalog", procure por "External Data" → "Postgres" ou pelo menu de **Add data**.

### 🖱️ Passo 3: Validar no Catalog Explorer

1. No painel esquerdo do Catalog Explorer, navegue até `lakebase_demo_<seu_database>`
2. Expanda o catalog → deve aparecer o schema `demo`
3. Expanda `demo` → deve aparecer a tabela `customers`
4. Clique em `customers` para ver a aba **Sample Data**

### 💻 Passo 4: Consultar via Databricks SQL

Abra um novo **SQL Editor** (do menu lateral do workspace, não do projeto Lakebase) e rode:

```sql
SELECT * FROM lakebase_demo_<seu_database>.demo.customers;
```

A mesma tabela que está no Postgres aparece como se fosse uma tabela Delta — mesma sintaxe, mesma UC, mesma governança.

### 🧪 Try it yourself (modo treinamento)

**Exercício** — No Catalog Explorer, clique em **Permissions** na tabela `customers` e dê `SELECT` para algum grupo. Depois revogue. Note como o controle é o mesmo de qualquer tabela Delta.

---

## Próximo exercício

➡️ [`../02_seguranca/`](../02_seguranca/) — Row-Level Security + Column Masking via UC

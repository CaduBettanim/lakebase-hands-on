# Exercício 2 — Segurança no Lakebase

🎯 **Modo Demo:** obrigatório

Aprenda a proteger dados sensíveis em Lakebase com dois mecanismos complementares: **Row-Level Security (RLS)** para filtrar linhas por usuário/tenant e **Column Masking** para esconder valores de colunas (PII) de quem não pode vê-los. Veja como ambos se integram ao Unity Catalog.

## O que você vai aprender

- Aplicar **Row-Level Security** nativa do Postgres em tabelas Lakebase
- Por que Databricks identities têm `BYPASSRLS` por default e como contornar com `SET ROLE`
- Mascarar colunas sensíveis usando **VIEWs do Postgres** + `GRANT/REVOKE`
- Como uma VIEW criada no Lakebase fica automaticamente disponível no Unity Catalog
- Limitação atual: UC Column Masks nativos **não** se aplicam a tabelas `MANAGED_ONLINE_CATALOG` (Lakebase) — a abordagem certa é via VIEW

## Pré-requisitos

- ✅ [Exercício 1](../01_fundamentos/) completo (instância Lakebase + schema `demo` + tabela `demo.customers` registrada no UC)
- ✅ SQL Warehouse rodando (para consultar via UC ao final)

---

## Parte 1 — Row-Level Security

### Objetivo

Aplicar RLS na tabela `demo.orders` para que cada **account manager** só veja seus próprios pedidos.

### 💻 Passo 1: Criar tabela `orders` com coluna de owner

No **SQL Editor do projeto Lakebase** (branch `production`, database `databricks_postgres`):

```sql
CREATE TABLE IF NOT EXISTS demo.orders (
    id SERIAL PRIMARY KEY,
    customer_id INT,
    amount NUMERIC(10,2),
    account_manager TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

TRUNCATE demo.orders RESTART IDENTITY;

INSERT INTO demo.orders (customer_id, amount, account_manager) VALUES
    (1, 100.00, 'alice@databricks.com'),
    (2, 200.00, 'alice@databricks.com'),
    (3, 300.00, 'bob@databricks.com'),
    (1, 400.00, 'bob@databricks.com');

SELECT * FROM demo.orders;
```

Você deve ver 4 pedidos, divididos entre Alice (2) e Bob (2).

### 💻 Passo 2: Habilitar RLS + criar policy

```sql
-- Habilita RLS
ALTER TABLE demo.orders ENABLE ROW LEVEL SECURITY;

-- Força RLS também para o table owner (sem isso, owner ignora a policy)
ALTER TABLE demo.orders FORCE ROW LEVEL SECURITY;

-- Cria a policy: usuário só vê linhas onde account_manager = setting de sessão
CREATE POLICY orders_by_manager ON demo.orders
    FOR ALL
    USING (account_manager = current_setting('app.current_manager', true));
```

> 💡 **Como funciona:** A função `current_setting('app.current_manager', true)` lê uma variável de sessão. O app cliente faz `SET app.current_manager = '<email>'` antes de cada query, e o Postgres filtra automaticamente. Isso é o padrão multi-tenant em Postgres.

### 💻 Passo 3: Verificar que sua identidade tem `BYPASSRLS`

```sql
SELECT current_user, rolbypassrls
FROM pg_roles WHERE rolname = current_user;
```

Resultado esperado: `rolbypassrls = t` (true). **Sua identidade Databricks tem privilégio de bypass.** Isso é por design no Lakebase para que admins sempre consigam acessar dados, mas significa que você não verá o efeito da policy ao fazer SELECT direto.

### 💻 Passo 4: Criar role de aplicação sem `BYPASSRLS`

```sql
CREATE ROLE demo_app NOINHERIT;
GRANT demo_app TO current_user;
GRANT USAGE ON SCHEMA demo TO demo_app;
GRANT SELECT ON demo.orders TO demo_app;
```

A role `demo_app` representa o "user da aplicação" — ela honra RLS porque não tem `BYPASSRLS`.

### 💻 Passo 5: Testar a policy

```sql
-- Trocar para o role demo_app na sessão
SET ROLE demo_app;

-- Como Alice
SET app.current_manager = 'alice@databricks.com';
SELECT * FROM demo.orders;  -- só os pedidos da Alice

-- Como Bob
SET app.current_manager = 'bob@databricks.com';
SELECT * FROM demo.orders;  -- só os pedidos do Bob

-- Sem identificação
RESET app.current_manager;
SELECT * FROM demo.orders;  -- 0 linhas

-- Voltar para sua identidade original
RESET ROLE;
```

✅ **Resultado esperado:** Alice 2 linhas, Bob 2 linhas, sem setting 0 linhas.

### 🧪 Try it yourself

**Exercício** — Modifique a policy para que `account_manager = current_user` (usar o próprio usuário Postgres conectado, em vez de uma session variable). Que tipo de cenário esse modelo serve melhor?

<details>
<summary>Resposta</summary>

Quando cada account manager tem o próprio Postgres role e conecta diretamente — comum em apps internos com SSO repassado para o banco. O modelo via session variable (que usamos) é melhor quando a app é multi-tenant e conecta com um único role compartilhado, repassando contexto via `SET`.
</details>

---

## Parte 2 — Column Masking via VIEW + Unity Catalog

### Objetivo

Mascarar a coluna `email` da tabela `demo.customers` para usuários sem permissão de PII — e fazer isso de forma que a UC enxergue a view mascarada como qualquer outra tabela.

### ⚠️ Por que não usar `ALTER TABLE ... SET MASK` direto?

A sintaxe canônica do UC para column masking é:

```sql
-- ❌ Isso NÃO funciona em tabelas Lakebase hoje
ALTER TABLE lakebase_demo_<seu_database>.demo.customers
ALTER COLUMN email SET MASK <catalog>.<schema>.mask_email;
```

Tabelas tipo `MANAGED_ONLINE_CATALOG` (Lakebase via UC) **só permitem alterar campos de owner** — você não pode aplicar masks/tags via ALTER. Erro retornado:

```
UPDATE_OR_DELETE_FORBIDDEN_FOR_FOREIGN_SECURABLE — Securable of type
TABLE_MANAGED_POSTGRESQL cannot have [column_infos] fields updated
```

**A abordagem certa hoje:** criar uma VIEW no Postgres que aplica o masking via `CASE`, dar `GRANT` na view, `REVOKE` na tabela base. A view aparece automaticamente no UC e a transformação é aplicada antes do dado sair do Lakebase.

### 💻 Passo 1: Criar a view com masking

No **SQL Editor do projeto Lakebase**:

```sql
CREATE OR REPLACE VIEW demo.customers_safe AS
SELECT
    id,
    name,
    CASE
        WHEN current_setting('app.is_pii_viewer', true) = 'true' THEN email
        ELSE regexp_replace(email, '^(.{2}).*(@.*)$', '\1***\2')
    END AS email,
    signed_up_at
FROM demo.customers;
```

A view aplica o mask por padrão. Quem souber a "chave" (`app.is_pii_viewer = true`) consegue ver o email completo.

### 💻 Passo 2: Bloquear acesso direto à tabela base

```sql
REVOKE SELECT ON demo.customers FROM demo_app;
GRANT SELECT ON demo.customers_safe TO demo_app;
```

Agora `demo_app` só consegue ler a view (mascarada) e não consegue dar `SELECT * FROM demo.customers` direto.

### 💻 Passo 3: Testar como app sem permissão de PII

```sql
SET ROLE demo_app;

-- Email mascarado (default)
SELECT * FROM demo.customers_safe;

-- Habilitar PII flag e ver email completo
SET app.is_pii_viewer = 'true';
SELECT * FROM demo.customers_safe;

-- Tentar a tabela base direto -> permission denied
SELECT * FROM demo.customers LIMIT 1;

RESET ROLE;
```

✅ **Resultado esperado:**
| Cenário | Email retornado |
|---|---|
| `demo_app` sem flag | `al***@example.com` |
| `demo_app` com `app.is_pii_viewer='true'` | `alice@example.com` |
| `demo_app` em `demo.customers` direto | ❌ `permission denied` |

### 💻 Passo 4: Confirmar que a UC enxerga a view

Abra um **novo SQL Editor** (do menu lateral do workspace, não do projeto Lakebase) conectado a um **SQL Warehouse**:

```sql
SELECT * FROM lakebase_demo_<seu_database>.demo.customers_safe ORDER BY id;
```

O dado vem **já mascarado**, sem precisar passar `app.is_pii_viewer`. Por que? Porque a conexão do SQL Warehouse para o Lakebase não passa a flag, então o `CASE` retorna o ramo padrão (mascarado).

> 💡 Esse é o ponto-chave: o mask **acontece no Lakebase**, antes do dado sair. UC só repassa o resultado. Não há jeito de "vazar" PII desligando o mask no Databricks SQL.

### 🧪 Try it yourself

**Exercício** — Crie outra view `demo.customers_internal` que mascara `email` mas mostra apenas se o domínio é interno (`@databricks.com`). Use `split_part(email, '@', 2)` para extrair o domínio.

---

## Próximo exercício

➡️ [`../03_htap/`](../03_htap/) — OLTP + Analytics no mesmo dado, sem ETL

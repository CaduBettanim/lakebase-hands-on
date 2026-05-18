# Lakebase — Demo + Treinamento

Material end-to-end de **Databricks Lakebase** (Postgres serverless integrado ao Lakehouse). Funciona como demo executiva (~1h15) ou treinamento self-paced (~2h45), com o mesmo conteúdo.

## Como usar este material

- **Demo executiva:** percorra os guias na ordem, executando só as células marcadas como obrigatórias. Pule as seções "Try it yourself".
- **Treinamento self-paced:** execute tudo, incluindo exercícios "Try it yourself" e seção opcional do Cenário 5.
- **Reprodutível em qualquer workspace Databricks** (incluindo workspaces de cliente) — não há dependência externa nem credencial fixa.

## Estrutura

```
lakebase-hands-on/
├── 00_overview/             Visão geral, comparativo, quando usar
├── 01_fundamentos/          Criar Lakebase, CRUD, branching, PITR, UC
├── 02_seguranca/            Row-Level Security + Column Masking via UC
├── 03_htap/                 OLTP + sync Lakebase→Delta + dashboard
├── 04_agent_memory/         Memória de longo prazo para agentes (pgvector)
├── 05_reverse_etl_app/      Synced Delta→Lakebase + Databricks App (opcional)
└── 99_cleanup/              Derruba todos os recursos criados
```

## Pré-requisitos

| Requisito | Como obter |
|---|---|
| Workspace Databricks com Lakebase habilitado | Em ambiente Databricks: confira em **Compute → Database Instances**. Se não aparecer, fale com o admin do workspace. |
| Permissão para criar Database Instance | Role `account admin` ou `workspace admin`, ou grant específico de Lakebase |
| Permissão para criar Unity Catalog objects | `CREATE CATALOG` ou usar um catalog existente com `USE CATALOG` |
| Cluster ou compute serverless ativo | Qualquer DBR 15.4+ ou SQL Warehouse Serverless |

## Convenções

- **🖱️ UI** sinaliza passo via interface gráfica
- **💻 SQL Editor** sinaliza SQL pra colar no Databricks SQL Editor
- **📓 Notebook** sinaliza célula em notebook Python
- **🎯 Modo Demo** = obrigatório no caminho rápido
- **🧪 Try it yourself** = opcional, pra aprofundar (modo treinamento)

## Variáveis de ambiente

Todo o material usa as variáveis abaixo. Ajuste-as ao seu workspace antes de começar — elas aparecem como placeholders nos SQLs e como widgets nos notebooks.

| Variável | Default | Descrição |
|---|---|---|
| `${catalog}` | `dbacademy` | Unity Catalog onde os objetos serão criados |
| `${schema}` | `lakebase_demo` | Schema dentro do catalog |
| `${lakebase_project}` | `lakebase-demo` | Nome da instância Lakebase criada no Cenário 1 |
| `${lakebase_db}` | `demo` | Database lógico dentro da instância Lakebase |

> Os notebooks declaram essas variáveis via **widgets** (`dbutils.widgets.text`) — você muda o valor uma vez no topo do notebook e ele propaga.
> Os arquivos SQL usam a sintaxe `${var}` (Databricks SQL Editor expande automaticamente quando o parâmetro está configurado).

## Ordem sugerida

1. **00_overview** — leia o `visao_geral.md` (10 min)
2. **01_fundamentos** — base de tudo (~30 min)
3. **02_seguranca** — RLS e column masking (~25 min)
4. **03_htap** — OLTP + Analytics (~30 min)
5. **04_agent_memory** — agente com memória (~35 min)
6. **05_reverse_etl_app** — serving + app opcional (~30 min)
7. **99_cleanup** — derruba tudo (5 min)

## Suporte

Dúvida durante execução? Cada guia tem uma seção **Troubleshooting comum** no fim. Se persistir, abra uma issue na pasta `99_cleanup/issues_notas.md`.

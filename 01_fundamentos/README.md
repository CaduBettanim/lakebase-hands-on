# Cenário 1 — Fundamentos do Lakebase

🎯 **Modo Demo:** obrigatório

Base para todos os outros cenários. Aqui você cria a instância Lakebase, conecta, faz operações CRUD básicas, explora branching + PITR e registra no Unity Catalog.

## O que você vai aprender

- Hierarquia Lakebase: **Project → Branch → Endpoint**
- Como provisionar uma instância via UI em ~2 min
- Conectar via Databricks SQL Editor com OAuth (sem senha estática)
- Criar database, schema e tabelas
- Usar **branches** como "git para dados"
- Restaurar dados com **PITR** (Point-in-Time Recovery)
- Registrar o Lakebase no Unity Catalog para governança unificada

## Passos

1. [`01_criar_lakebase.md`](01_criar_lakebase.md) — Criar a instância via UI
2. *(em construção)* `02_schema_e_crud.md` — Conectar e criar tabelas
3. *(em construção)* `03_branching_e_pitr.md` — Branches + restauração no tempo
4. *(em construção)* `04_unity_catalog.md` — Registrar no UC

## Pré-requisitos

- Workspace Databricks com Lakebase habilitado
- Permissão para criar Database Instance
- Catalog disponível no Unity Catalog (default: `dbacademy`)

## Próximo cenário

➡️ [`../02_seguranca/`](../02_seguranca/) — Row-Level Security + Column Masking via UC

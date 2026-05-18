# 00 — Visão Geral do Lakebase

⏱️ **Tempo de leitura:** 10 min (demo) / 15 min (treinamento)

## O que é Lakebase

**Lakebase é Postgres 17 serverless gerenciado pela Databricks, nativamente integrado ao Lakehouse.**

Em uma frase: é o único banco transacional que vive *dentro* do Unity Catalog — mesmo controle de acesso, mesma governança, mesma audit log que tabelas Delta.

### Os 3 pilares

1. **Postgres de verdade** — Postgres 17 compatível 100%. Seu app Django/Rails/Spring funciona sem mudar uma linha.
2. **Serverless** — autoscaling de 0.5 a 32 CU, scale-to-zero quando idle. Sem provisionar nada.
3. **Integrado ao Lakehouse** — registrado no Unity Catalog, sincronização bidirecional com Delta, governança unificada.

## Por que existe (o problema que resolve)

Hoje empresas têm o "split Lakehouse / OLTP":

```
┌─────────────────┐         ┌──────────────────┐
│ App operacional │ ──ETL─→ │ Data Lake/Warehouse │
│ (RDS, Aurora)   │         │ (Databricks, etc)  │
└─────────────────┘         └──────────────────┘
   • Credenciais separadas      • Pipeline custom (Kafka/CDC)
   • Governança separada        • Latência de minutos/horas
   • PII em dois lugares        • Time de dados ≠ time de app
```

Lakebase **elimina o split**:

```
┌────────────────────────────────────────────┐
│           Unity Catalog (governança)        │
├─────────────────────┬──────────────────────┤
│  Lakebase (OLTP)    │  Delta Lake (Analytics)│
│  - Pedidos          │  - Pedidos agregados  │
│  - Sessões          │  - Métricas BI        │
│  - Memória agente   │  - Features ML        │
└─────────────────────┴──────────────────────┘
        ↕ Synced Tables (segundos)
```

## Comparativo competitivo

| Capacidade | Lakebase | RDS Postgres | Aurora | Snowflake Hybrid Tables |
|---|---|---|---|---|
| Postgres 100% compat | ✅ | ✅ | ✅ | ❌ (subset) |
| Serverless / scale-to-zero | ✅ | ❌ | ⚠️ (Aurora Serverless v2 com baseline) | ⚠️ |
| Branching copy-on-write | ✅ | ❌ | ❌ (clone existe, lento) | ❌ |
| Governança unificada c/ Lakehouse | ✅ (UC nativo) | ❌ | ❌ | ⚠️ (mesma plataforma só) |
| Sync nativo com Delta/Lakehouse | ✅ | ❌ (DMS, ETL) | ❌ | ⚠️ |
| Audit unificada c/ dados analíticos | ✅ | ❌ | ❌ | ⚠️ |
| pgvector | ✅ | ✅ | ✅ | ❌ |

**Conclusão:** RDS/Aurora competem em "ser Postgres". Lakebase compete em "ser o Postgres do Lakehouse" — é categoria diferente.

## Quando usar Lakebase

✅ **Use Lakebase para:**
- Backend operacional de Databricks Apps
- Memory store de agentes IA (curto + longo prazo)
- Feature serving online (<50ms) a partir de modelos batch
- Apps internos onde governança UC importa
- Reverse ETL (Delta → app operacional)
- Sistemas OLTP que precisam de analytics em tempo real (HTAP)

❌ **NÃO use Lakebase para:**
- Workloads puramente analíticos (Delta + Photon é melhor)
- Cargas massivas de batch ETL (Lakeflow/DLT)
- Volumes >10TB em uma única tabela transacional

## Arquitetura interna (resumida)

- **Storage layer separada da compute** — storage "bottomless" no S3/GCS/Azure Blob
- **Compute layer** — endpoints autoscale independentes, podem ter múltiplos por branch
- **Branches** — copy-on-write no storage layer (criar branch de 10TB = segundos, custo zero até divergir)

Resultado: você pode ter `production`, `staging`, `dev-feature-x`, `dev-feature-y` — todos do mesmo dado de produção, isolados, baratos.

## Os 5 exercícios da demo (mapa)

| Exercício | Mostra | Pra quem é |
|---|---|---|
| **1. Fundamentos** | Criar, conectar, fazer CRUD, branching, PITR, registrar no UC | Todos |
| **2. Segurança** | Row-Level Security + Column Masking via UC | Compliance, Sec, CDO |
| **3. HTAP** | Pedidos chegando no Lakebase, aparecendo em dashboard analítico em segundos | Arquitetos, CDO |
| **4. Agent Memory** | Agente IA com pgvector + JSONB, memória persistente cross-session | ML Lead, AI Builders |
| **5. Reverse ETL + App** | Features ML servidas em <50ms, opcionalmente via Databricks App | Eng. de plataforma, MLOps |

## Próximo passo

➡️ Vá para [`01_fundamentos/01_criar_lakebase.md`](../01_fundamentos/01_criar_lakebase.md)

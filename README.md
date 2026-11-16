# Customer360 AI

AI-powered customer intelligence and retention platform for insurance, built with Snowflake + Cortex AI + Streamlit.

## Live Demo

- **Streamlit app:** https://app.snowflake.com/lexmgqw/eu51827/#/streamlit-apps/CUSTOMER_360_DB.C360.CUSTOMER360_AI
- **Read-only demo login:** username `DEMO_VIEWER`, password `Customer360Demo!2026` (role `CUSTOMER360_VIEWER`)

## Architecture

```
Source Tables (5)          Dynamic Tables (3)              Streamlit App (17 files)
CUSTOMERS ──────┐
POLICIES ───────┤
CLAIMS ─────────┼──→ CUSTOMER_360_VIEW ──┬──→ app.py (entry point)
INTERACTIONS ───┤         (churn scoring) ├──→ views/ (5 pages)
TRANSCRIPTS ────┘                        ├──→ components/ (6 widgets)
                 CHURN_SIGNALS_EXPLAINED ─┘   utils/ (DB + formatting)
                 NEXT_BEST_ACTION ───────┘

AI Functions (4) — SQL UDFs calling AI_COMPLETE (llama3.1-70b)
```

## Setup

Upload the `sql/` files to a stage and run them in order:

```sql
-- 1. Database, schema, and 5 source tables
EXECUTE IMMEDIATE FROM @CUSTOMER360_APP_STAGE/setup_sql/01_tables.sql;

-- 2. Synthetic data (deterministic: 500 customers, ~780 policies, ~700 claims,
--    ~2,400 interactions, ~1,100 call transcripts)
EXECUTE IMMEDIATE FROM @CUSTOMER360_APP_STAGE/setup_sql/02_generate_data.sql;

-- 3. 3 dynamic tables (10-minute target lag) + 4 AI functions
EXECUTE IMMEDIATE FROM @CUSTOMER360_APP_STAGE/setup_sql/03_dynamic_tables_and_functions.sql;
```

If your region doesn't host `llama3.1-70b`, enable cross-region inference:
`ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION';`

Then deploy the Streamlit app:

```sql
CREATE STAGE IF NOT EXISTS CUSTOMER_360_DB.C360.CUSTOMER360_APP_STAGE;
-- Upload all Python files to @CUSTOMER360_APP_STAGE/CUSTOMER360_AI/

CREATE STREAMLIT IF NOT EXISTS CUSTOMER_360_DB.C360.CUSTOMER360_AI
  ROOT_LOCATION = '@CUSTOMER_360_DB.C360.CUSTOMER360_APP_STAGE/CUSTOMER360_AI'
  MAIN_FILE = 'app.py'
  QUERY_WAREHOUSE = COMPUTE_WH
  TITLE = 'Customer360 AI';
```

## Pages

| Page | Description |
|------|-------------|
| Executive Dashboard | KPIs, risk distribution, high-priority customer list |
| Customer 360 | Full customer profile with 6 detail tabs |
| Ask Customer 360 | Natural language Q&A over customer data |
| Action Center | Filterable priority work queue |
| Customer Analytics | Management-level analytical charts |

## Built With

- **Snowflake** — Data warehouse, dynamic tables, SQL functions
- **Cortex AI** — `AI_COMPLETE` with `llama3.1-70b`
- **Streamlit-in-Snowflake** — Application framework
- **Cortex Code (CoCo)** — AI-powered IDE that generated all code

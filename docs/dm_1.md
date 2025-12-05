# Stage 4: Data Mart (DM) Aggregation

This final transformation aggregates the detailed DW data into summary tables for reporting and dashboards.

## Source
- **Database:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (DW tables)

## Destination
- **Database:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (DM tables)

## Transformations
1. **Manifestacao Aggregation (`dm_Manifestacao`)**
    - **Source:** `dw_Manifestacao`
    - **Filter Logic:** Selects records created in the last 3 months (rolling window optimization).
    - **Grouping Keys:**
        - `Data` (normalized to the 1st of the month)
        - `TipoAtendimentoManifestacaoId`
        - `PerfilManifestacaoId`
        - `ClassificaoManifestacaoId`
        - `MotivoManifestacaoId`
        - `EstabelecimentoTratado`
    - **Metrics:** `Total` (Sum of count).
    - **Output:** `dm_Manifestacao` table is updated with these aggregated snapshots.

2. **Melhorias Aggregation (`dm_melhorias`)**
    - Similar logic applied for "Melhorias" (Improvements) if applicable, aggregating counts based on categorization.

## Purpose
These tables are optimized for direct consumption by Power BI or other visualization tools, providing pre-aggregated monthly totals by various dimensions.

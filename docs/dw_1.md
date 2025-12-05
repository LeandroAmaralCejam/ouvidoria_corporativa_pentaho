# Stage 3: Data Warehouse (DW) Transformation

This transformation takes the raw data from the Staging area, cleans it, and structures it for the Data Warehouse.

## Source
- **Database:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `oc` (Staging tables)

## Destination
- **Database:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (DW tables)

## ETL Process
The process involves reading from the Staging tables, applying sort operations (required for some Pentaho steps or just for organization), and loading into `dw_` tables.

| Staging Table | Target DW Table | Description |
| :--- | :--- | :--- |
| `stage_PerguntaPadrao` | `dw_PerguntaPadrao` | Standard Questions |
| `stage_perguntaresposta` | `dw_perguntaresposta` | QA pairs |
| `stage_formulario` | `dw_formulario` | Forms |
| `stage_ClassificacaoManifestacao` | `dw_ClassificacaoManifestacao` | Classifications |
| `stage_MotivoManifestacao` | `dw_MotivoManifestacao` | Reasons |
| `stage_Manifestacao` | `dw_Manifestacao` | Manifestations |
| `stage_Regionais_MedicSys` | `dw_Regionais_MedicSys` | Regionals |

## Key Logic
- **Sorting:** All streams are sorted by their respective IDs (e.g., `Id`, `Id_Medicsys`) to ensure ordered insertion or to facilitate downstream merges/joins if needed.
- **Manifestacao Handling:** 
    - The transformation reads 3 months of history for `Manifestacao` to update recent records while maintaining older history constant, optimizing performance.
    - *Note: There is a commented-out logic for full truncation vs sliding window updates.*

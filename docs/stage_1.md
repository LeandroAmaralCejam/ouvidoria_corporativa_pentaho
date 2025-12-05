# Stage 1: Extraction from Helper System (MySQL)

This transformation is responsible for extracting the raw data from the operational database (MySQL) and loading it into the Staging Area (SQL Server).

## Source System
- **Database Type:** MySQL
- **Schema:** `cejam`
- **Connection:** `con_mysql_medicsys`

## Destination System (Staging)
- **Database Type:** SQL Server
- **Schema:** `oc` (implied Staging schema)
- **Connection:** `con_db_medicsys_cejam`

## ETL Process
The process extracts data from the following tables and loads them into their respective `stage_` tables. Some tables use an incremental load strategy based on dates, while others are full loads.

| Source Table | Destination Table | Load Type | Description |
| :--- | :--- | :--- | :--- |
| `ClassificacaoManifestacao` | `stage_ClassificacaoManifestacao` | Full | Classification types |
| `Manifestacao` | `stage_Manifestacao` | Incremental | Main manifestation records |
| `MotivoManifestacao` | `stage_MotivoManifestacao` | Full | Reasons for manifestations |
| `PerfilManifestacao` | `stage_PerfilManifestacao` | Full | Profiles |
| `TipoAtendimentoManifestacao` | `stage_TipoAtendimentoManifestacao` | Full | Types of attendance |
| `formulario` | `stage_formulario` | Full | Forms definitions |
| `formulariopergunta` | `stage_formulariopergunta` | Full | Questions in forms |
| `formularioresposta` | `stage_formularioresposta` | Full | Answers in forms |
| `estabelecimento` | `stage_estabelecimento` | Full | Establishments |
| `perguntapadrao` | `stage_perguntapadrao` | Full | Standard questions |
| `perguntaresposta` | `stage_perguntaresposta` | Full | Question answers |
| `pesquisaresposta` | `stage_pesquisaresposta` | Incremental | Survey responses |
| `pesquisaestabelecimento` | `stage_pesquisaestabelecimento` | Incremental | Survey establishments |

## Transformation Logic
- **Full Refresh:** Truncates the destination table and reloads all data.
- **Incremental:** Uses `max_date` lookup to only fetch records created or modified after the last successful load.

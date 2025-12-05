# Stage 2: Extraction from Excel (Regionals)

This transformation extracts auxiliary regional classification data from a shared Excel file.

## Source
- **Type:** Excel File (`.xlsx`)
- **Path:** `G:\Drives compartilhados\INFORMAÇÃO\03. Dashboards\Institucional\Ouvidoria Corporativa\DIM_REGIONAIS _MEDICSYS\DIM_REGIONAIS_MEDICSYS.xlsx`
- **Sheet:** `MedicSys`

## Destination
- **Database:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `oc`
- **Table:** `stg_Regionais_MedicSys`

## Fields Mapped
- `Unidade` -> `Unidade`
- `Nível de Atenção` -> `Nível de Atenção`
- `Município` -> `Município`
- `Tipo de Unidade` -> `Tipo de Unidade`
- `Id_Medicsys` -> `Id_Medicsys`
- `Regional` -> `Regional`

## Logic
- **Truncate & Load:** The destination table (`stg_Regionais_MedicSys`) is truncated before inserting the rows from the Excel file (Full Refresh).
- **Sorting:** Rows are sorted by `Id_Medicsys` before loading.

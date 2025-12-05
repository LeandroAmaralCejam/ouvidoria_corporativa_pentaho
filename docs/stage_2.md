# Estágio 2: Extração de Excel (Regionais)

Esta transformação extrai dados auxiliares de classificação regional de um arquivo Excel compartilhado.

## Origem
- **Tipo:** Arquivo Excel (`.xlsx`)
- **Caminho:** `G:\Drives compartilhados\INFORMAÇÃO\03. Dashboards\Institucional\Ouvidoria Corporativa\DIM_REGIONAIS _MEDICSYS\DIM_REGIONAIS_MEDICSYS.xlsx`
- **Planilha:** `MedicSys`

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `oc`
- **Tabela:** `stg_Regionais_MedicSys`

## Campos Mapeados
- `Unidade` -> `Unidade`
- `Nível de Atenção` -> `Nível de Atenção`
- `Município` -> `Município`
- `Tipo de Unidade` -> `Tipo de Unidade`
- `Id_Medicsys` -> `Id_Medicsys`
- `Regional` -> `Regional`

## Lógica
- **Truncar e Carregar:** A tabela de destino (`stg_Regionais_MedicSys`) é truncada (limpa) antes de inserir as linhas do arquivo Excel (Atualização Completa).
- **Ordenação:** As linhas são ordenadas por `Id_Medicsys` antes do carregamento.


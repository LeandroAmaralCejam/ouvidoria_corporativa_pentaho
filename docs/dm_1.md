# Estágio 4: Agregação Data Mart (DM)

Esta transformação final agrega os dados detalhados do DW em tabelas de resumo para relatórios e dashboards.

## Origem
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DW)

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DM)

## Transformações
## Transformações Detalhadas

### 1. DM Manifestação
O processamento de Manifestações no Data Mart ocorre em duas etapas principais: limpeza (truncate) da janela deslizante e carga agregada.

- **[Ver SQL: Limpeza (Truncate 3 Meses)](sql/dm_1_manifestacao_truncate.md)**
    - Remove os dados dos últimos 3 meses da tabela `dm_Manifestacao` para evitar duplicação antes da recarga incremental.
- **[Ver SQL: Carga e Agregação](sql/dm_1_manifestacao_load.md)**
    - Realiza a leitura do DW, aplica o a janela de tempo e agrega os totais mensais.

### 2. DM Melhorias e Pesquisa
Processamento complexo para dados de melhorias e respostas textuais.

- **[Ver SQL: Limpeza (Truncate 3 Meses)](sql/dm_1_melhorias_truncate.md)**
    - Limpa a tabela alvo `dm_melhorias`.
- **[Ver SQL: Carga Lógica Complexa](sql/dm_1_melhorias_load.md)**
    - Script extenso utilizando tabelas temporárias (`#E1`, `#E2`, etc.) para transformar respostas textuais e relacioná-las com perguntas padrão.




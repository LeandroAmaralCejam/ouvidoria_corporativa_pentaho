# Estágio 3.2: Transformação Data Warehouse - Pesquisa Resposta

Esta transformação é dedicada ao processamento das respostas da pesquisa de satisfação, consolidando dados de respostas e estabelecimentos.

## Origem
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `oc` (Tabelas Staging: `stg_pesquisaresposta`, `stg_pesquisaestabelecimento`)

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc`
- **Tabela:** `dw_FATO_PESQUISA_RESPOSTA` (implícito pelo nome do KTR)

## Processo ETL
A transformação realiza um join entre as respostas e os estabelecimentos, aplicando filtros específicos de formulários e perguntas, além de substituições de texto (ex: "Pessimo" para "Péssimo").

## Lógica Principal
- **Carga Incremental:** Utiliza a data máxima (`stamp`) da tabela de destino para carregar apenas novos registros.
- **Filtros de Negócio:**
    - Exclusão de formulários específicos (`46, 48, ...`).
    - Exclusão de estabelecimentos inativos ou específicos da Ouvidoria (`106, 112`).
    - Filtro de perguntas padrão (`PerguntaPadraoId`).

## Consultas SQL Utilizadas

### Lookup de Data Máxima (Incremental)
```sql
SELECT MAX(stamp) AS MAX_DATE FROM medicsys.oc.dw_FATO_PESQUISA_RESPOSTA WHERE Id IS NOT NULL
```

### Extração e Transformação (`stage_pesquisa_resposta`)
- **[Ver SQL: Extração Detalhada](sql/dw_2_pesquisaresposta.md)**
- **Descrição:** Extrai respostas de pesquisa incementalmente, aplicando filtros rigorosos de formulários e perguntas específicas para garantir a qualidade dos dados.


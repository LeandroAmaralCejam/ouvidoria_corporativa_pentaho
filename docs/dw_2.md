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
```sql
SELECT
    r.Id
    ,CONVERT(DATE, r.Criado_Dt) AS Data
	,e.EstabelecimentoId
    ,e.FormularioId
    ,r.PesquisaEstabelecimentoId
    ,r.PerguntaPadraoId
    ,r.PerguntaRespostaId
    ,r.NumeroResposta
	,REPLACE(r.TextoResposta,'Pessimo','Péssimo') AS TextoResposta
	,r.Alterado_Dt as stamp
FROM medicsys.oc.stg_pesquisaresposta r
INNER JOIN medicsys.oc.stg_pesquisaestabelecimento e ON r.PesquisaEstabelecimentoId = e.Id
WHERE 
    r.TextoResposta IS NOT NULL
	AND e.EstabelecimentoId NOT IN (106, 112) -- Retirados pela Ouvidoria
    AND e.IsAtivo = 1
    AND e.FormularioId NOT IN (46, 48, 53, 60, ... ) -- IDs de formulários ignorados
    AND r.PerguntaPadraoId IN (1,2, ... ) -- IDs de perguntas padrão
AND r.Alterado_Dt > ? -- INCREMENTAL
```

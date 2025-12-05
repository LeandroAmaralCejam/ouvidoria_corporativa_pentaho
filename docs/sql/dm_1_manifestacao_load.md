# SQL: Carga Manifestação (Agregação)
**Transformação:** `dm_1.ktr`
**Passo:** `Manifestacao`

## Objetivo
Extrai dados do Data Warehouse (`dw_Manifestacao`), filtra pela janela de 3 meses, e agrega as contagens por dimensões chave (Data, TipoAtendimento, Perfil, Classificação, Motivo, Estabelecimento).

## Script SQL
```sql
WITH
E1 AS
(
SELECT
*

FROM
medicsys.oc.dw_Manifestacao

WHERE
Id IS NOT NULL
--------
/*
O TRECHO ABAIXO GARANTE QUE APENAS OS ULTIMOS 3 MESES SEJAM ATUALIZADOS,
PARA TRUNCAR TODA A TABELA, COMENTE A LINHA E MARQUE TRUNCAR NO OUTPUT TABLE NO STEP FINAL
*/
AND Criado_Dt >= DATEADD(DAY, 1, EOMONTH(GETDATE(), -4)) -- VOLTA 3 MESES E RETORNA O PRIEMEIRO DIA DO MES RESULTANTE
--------

)

,E2 AS
(
SELECT
Id
,CAST(CONCAT(LEFT(CAST(Criado_Dt AS DATE),8),'01') AS DATE) AS Data
,Protocolo
,TipoAtendimentoManifestacaoId
,EstabelecimentoId
,EstabelecimentoEnvioId
,PerfilManifestacaoId
,ClassificaoManifestacaoId
,MotivoManifestacaoId
,Status
--,TempoResposta
,EstabelecimentoTratado
,1 as Total

FROM
E1
)


,E3 AS
(
SELECT
Data
--,Protocolo
,TipoAtendimentoManifestacaoId
--,EstabelecimentoId
--,EstabelecimentoEnvioId
,PerfilManifestacaoId
,ClassificaoManifestacaoId
,MotivoManifestacaoId
--,Status
,EstabelecimentoTratado
,SUM(Total) AS Total

FROM
E2

GROUP BY
Data
--,Protocolo
,TipoAtendimentoManifestacaoId
--,EstabelecimentoId
--,EstabelecimentoEnvioId
,PerfilManifestacaoId
,ClassificaoManifestacaoId
,MotivoManifestacaoId
--,Status
,EstabelecimentoTratado
)


-- ACRESCENTA O STAMP DE ATUALIZAÇÃO DA LINHA
SELECT
Data
,TipoAtendimentoManifestacaoId
,PerfilManifestacaoId
,ClassificaoManifestacaoId
,MotivoManifestacaoId
,EstabelecimentoTratado
,Total
,GETDATE() AS stamp

FROM
E3
```

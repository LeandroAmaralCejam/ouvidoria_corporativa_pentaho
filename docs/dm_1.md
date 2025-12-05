# Estágio 4: Agregação Data Mart (DM)

Esta transformação final agrega os dados detalhados do DW em tabelas de resumo para relatórios e dashboards.

## Origem
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DW)

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DM)

## Transformações
1. **Agregação Manifestação (`dm_Manifestacao`)**
    - **Origem:** `dw_Manifestacao`
    - **Lógica de Filtro:** Seleciona registros criados nos últimos 3 meses (janela deslizante).
    - **Chaves de Agrupamento:**
        - `Data` (normalizada para o dia 1 do mês)
        - `TipoAtendimentoManifestacaoId`
        - `PerfilManifestacaoId`
        - `ClassificaoManifestacaoId`
        - `MotivoManifestacaoId`
        - `EstabelecimentoTratado`
    - **Métricas:** `Total` (Soma da contagem).
    - **Saída:** A tabela `dm_Manifestacao` é atualizada com esses snapshots agregados.

2. **Agregação Melhorias (`dm_melhorias`)**
    - Lógica similar aplicada para "Melhorias" se aplicável, agregando contagens baseadas em categorização.

## Propósito
Estas tabelas são otimizadas para consumo direto pelo Power BI ou outras ferramentas de visualização, fornecendo totais mensais pré-agregados por várias dimensões.

## Consulta SQL da Agregação

### Agregação de Manifestações (`dm_Manifestacao`)
A consulta utiliza CTEs para filtrar, transformar e agrupar os dados.

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



# SQL: Dados Regionais
**Transformação:** `dw_1.ktr`
**Passo:** `dw_Regionais_MedicSys`

## Objetivo
Enriquece a tabela de regionais com o tipo de contrato do estabelecimento.

## Script SQL
```sql
WITH
E1 AS
(
SELECT
id
,CASE 
    WHEN ContratoId IN (8, 11, 18, 21, 28, 33, 35, 36) THEN 'Convenio'
    WHEN ContratoId = 2 THEN 'Sede'
    WHEN ContratoId IN (1, 3, 4, 5, 6, 7, 9, 10, 12, 13, 14, 15, 16, 17, 19, 20, 22, 23, 24, 25, 26, 27, 30, 31, 32, 34) THEN 'Contrato'
    ELSE 'Outro'
END AS TipoContrato

FROM
medicsys.oc.stg_estabelecimento
 
WHERE
Id IS NOT NULL
-- AND IsAtivo = 1
)



SELECT
A.Unidade
,A.[Nível de Atenção]
,A.Município
,A.[Tipo de Unidade]
,A.Id_Medicsys
,A.Regional
,B.TipoContrato AS Tipo

FROM
medicsys.oc.stg_Regionais_MedicSys A

LEFT JOIN
E1 B ON
A.Id_Medicsys = B.id
```

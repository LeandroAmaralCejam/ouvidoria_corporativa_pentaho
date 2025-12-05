# SQL: Classificação Manifestação
**Transformação:** `dw_1.ktr`
**Passo:** `stage_ClassificacaoManifestacao`

## Objetivo
Carrega a tabela de classificação das manifestações, aplicando uma regra de negócio para definir o `PesoManifestacao` (Positiva ou Negativa).

## Script SQL
```sql
SELECT
Id
,ClassificacaoManifestacaoDesc
,CASE
	WHEN Id IN ('2','5','6') THEN '1.Positiva'
	ELSE '2.Negativa'
END AS PesoManifestacao


FROM
medicsys.oc.stg_ClassificacaoManifestacao

WHERE
Id IS NOT NULL
-- AND IsAtivo = 1
```

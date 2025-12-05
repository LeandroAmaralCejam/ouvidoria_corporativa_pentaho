# SQL: Truncar Melhorias (3 Meses)
**Transformação:** `dm_1.ktr`
**Passo:** `Trunca Melhorias - 3 meses`

## Objetivo
Remove registros da tabela de destino (`dm_melhorias`) referentes aos últimos 3 meses para permitir o reprocessamento.

## Script SQL
```sql
-- TRUNC A TABELA ALVO EM 3 MESES

DELETE FROM 
medicsys.oc.dm_melhorias

WHERE
Data >= DATEADD(DAY, 1, EOMONTH(GETDATE(), -4)) -- VOLTA 3 MESES E RETORNA O PRIMEIRO DIA DO MES RESULTANTE
;
```

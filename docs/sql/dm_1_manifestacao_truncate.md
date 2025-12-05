# SQL: Truncar Manifestação (3 Meses)
**Transformação:** `dm_1.ktr`
**Passo:** `Trunca Manifestacao - 3 meses`

## Objetivo
Remove registros da tabela de destino (`dm_Manifestacao`) referentes aos últimos 3 meses para permitir o reprocessamento (carga incremental via janela deslizante).

## Script SQL
```sql
-- TRUNC A TABELA ALVO EM 3 MESES

DELETE FROM 
medicsys.oc.dm_Manifestacao

WHERE
Data IS NOT NULL
AND Data >= DATEADD(DAY, 1, EOMONTH(GETDATE(), -4)) -- VOLTA 3 MESES E RETORNA O PRIEMEIRO DIA DO MES RESULTANTE
;
```

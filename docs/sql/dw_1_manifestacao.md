# SQL: Manifestação (Enriquecida)
**Transformação:** `dw_1.ktr`
**Passo:** `stage_Manifestacao`

## Objetivo
Extrai os dados principais de manifestações da Stage. Aplica lógica para corrigir o `EstabelecimentoTratado` quando o estabelecimento original é administrativo (IDs 3 ou 142).

## Script SQL
```sql
SELECT
    Id
    ,Criado_Dt
    ,Protocolo
    ,TipoAtendimentoManifestacaoId
    ,EstabelecimentoId
    ,EstabelecimentoEnvioId
    ,PerfilManifestacaoId
    ,ClassificaoManifestacaoId
    ,MotivoManifestacaoId
    ,Status
    ,TempoResposta
    ,CASE 
        WHEN EstabelecimentoId IN (3, 142) 
             THEN ISNULL(EstabelecimentoEnvioId, EstabelecimentoId)
        ELSE EstabelecimentoId
    END AS EstabelecimentoTratado
FROM
    medicsys.oc.stg_Manifestacao
WHERE
    Id IS NOT NULL
    AND IsAtivo = 1;
```

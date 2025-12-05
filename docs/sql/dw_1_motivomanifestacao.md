# SQL: Agrupamento de Motivos
**Transformação:** `dw_1.ktr`
**Passo:** `dw_MotivoManifestacao`

## Objetivo
Classifica os motivos de manifestação em grupos macro para análise categorizada.

## Script SQL
```sql
SELECT
Id,
MotivoManifestacaoDesc,
CASE 
	WHEN Id = 1 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 2 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 3 THEN 'Elogio'
    WHEN Id = 4 THEN 'Elogio'
    WHEN Id = 5 THEN 'Clima organizacional'
    WHEN Id = 6 THEN 'Folha de Pagamento'
    WHEN Id = 7 THEN 'Processo Seletivo'
    WHEN Id = 8 THEN 'Agendamento de consulta'
    WHEN Id = 9 THEN 'Informações gerais'
    WHEN Id = 10 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 11 THEN 'Clima organizacional'
    WHEN Id = 12 THEN 'Agendamento de consulta'
    WHEN Id = 13 THEN 'Agendamento de exames'
    WHEN Id = 14 THEN 'Demora no atendimento'
    WHEN Id = 15 THEN 'Agendamento de cirurgia'
    WHEN Id = 16 THEN 'Falta de medicamentos'
    WHEN Id = 17 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 18 THEN 'Limpeza e manutenção'
    WHEN Id = 19 THEN 'Limpeza e manutenção'
    WHEN Id = 20 THEN 'Informações gerais'
    WHEN Id = 21 THEN 'Limpeza e manutenção'
    WHEN Id = 22 THEN 'Informações gerais'
    WHEN Id = 23 THEN 'Informações gerais'
    WHEN Id = 24 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 25 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 26 THEN 'Comportamento inadequado do profissional'
    WHEN Id = 27 THEN 'Processo Seletivo'
    WHEN Id = 28 THEN 'Folha de Pagamento'
    WHEN Id = 29 THEN 'Processo Seletivo'
    WHEN Id = 30 THEN 'Informações gerais'
    WHEN Id = 31 THEN 'Informações gerais'
    WHEN Id = 32 THEN 'Clima organizacional'
    WHEN Id = 33 THEN 'Dificuldade de atendimento telefônico.'
	WHEN Id = 34 THEN 'Fluxo de Atendimento'
END AS MotivoGrupo

FROM
medicsys.oc.stg_MotivoManifestacao

WHERE
Id IS NOT NULL
-- AND IsAtivo = 1
```

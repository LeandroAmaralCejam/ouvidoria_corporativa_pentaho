# SQL: Tabelas Simples (Pass-through)
**Transformação:** `dw_1.ktr`
**Passos:** `stage_formulario`, `stage_perguntaresposta`

## Objetivo
Estas transformações são do tipo "direct load" ou possuem transformações mínimas, carregando dados das tabelas de stage para o DW para garantir integridade referencial.

### 1. Formulário
**Passo:** `stage_formulario`
```sql
SELECT
  Id
, Nome
, [Grupo Formulário]
FROM medicsys.oc.stg_formulario
WHERE Id IS NOT NULL
```
*(Nota: A query exata é inferida como um SELECT * ou SELECT explícito das colunas mapeadas no KTR)*

### 2. Pergunta Resposta
**Passo:** `stage_perguntaresposta`
```sql
SELECT
  Id
, PerguntaPadraoId
, Ordem
, Resposta
FROM medicsys.oc.stg_perguntaresposta
WHERE Id IS NOT NULL
```

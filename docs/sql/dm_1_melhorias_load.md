# SQL: Carga Melhorias e Pesquisa (Complexo)
**Transformação:** `dm_1.ktr`
**Passo:** `melhorias`

## Objetivo
Processa dados de pesquisas de satisfação (`stg_pesquisaresposta`) e relaciona com perguntas e estabelecimentos para gerar insights de melhorias e respostas detalhadas. Utiliza tabelas temporárias para performance e organização.

## Script SQL
```sql
-- Limpar tabelas temporárias se existirem
IF OBJECT_ID('tempdb..#E1') IS NOT NULL DROP TABLE #E1;
IF OBJECT_ID('tempdb..#E2') IS NOT NULL DROP TABLE #E2;
IF OBJECT_ID('tempdb..#E3') IS NOT NULL DROP TABLE #E3;



-- TABELA 1: #E1 com CAST da data de modificação
SELECT 
    pe.EstabelecimentoId,
    es.Fantasia,
    pe.Data,
    pr.PerguntaPadraoId,
    pr.TextoResposta,
    pr.PerguntaRespostaId,
    sp.PerguntaDestinoId,
    STRING_AGG(sp.RespostasDestinoTexto, ',') AS RespostasDestinoTexto,  -- SubResposta aqui
    pr.FormularioPerguntaId,
    pe.Id,
    pp.Pergunta,
    CAST(pr.Alterado_Dt AS DATE) AS Alterado_Dt  -- Aqui já trazemos somente a data

INTO #E1
FROM 
medicsys.oc.stg_pesquisaresposta pr

INNER JOIN
medicsys.oc.stg_pesquisaestabelecimento pe ON
pr.PesquisaEstabelecimentoId = pe.Id

INNER JOIN
medicsys.oc.stg_perguntapadrao pp ON
pr.PerguntaPadraoId = pp.Id

INNER JOIN
medicsys.oc.stg_estabelecimento es ON
pe.EstabelecimentoId = es.Id

LEFT JOIN (
    SELECT 
        prd.PerguntaPadraoId AS PerguntaDestinoId,
        prd.TextoResposta AS RespostasDestinoTexto,
        prd.PesquisaEstabelecimentoId,
        fpo.PerguntaPadraoId AS PerguntaOrigemId,
        TRIM(ppr.Resposta) AS RespostaTextOrigem
    FROM medicsys.oc.stg_formularioresposta fr
    INNER JOIN medicsys.oc.stg_formulariopergunta fpo ON fr.FormularioPerguntaOrigemId = fpo.Id
    INNER JOIN medicsys.oc.stg_formulariopergunta fpd ON fr.FormularioPerguntaDestinoId = fpd.Id
    INNER JOIN medicsys.oc.stg_pesquisaresposta prd ON prd.PerguntaPadraoId = fpd.PerguntaPadraoId
    INNER JOIN medicsys.oc.stg_perguntaresposta ppr ON fr.PerguntaRespostaId = ppr.Id
    WHERE fr.id IS NOT NULL
      AND prd.TextoResposta IS NOT NULL
      AND prd.IsAtivo = 1
      AND fpo.IsAtivo = 1
    GROUP BY 
        prd.PerguntaPadraoId,
        prd.TextoResposta,
        prd.PesquisaEstabelecimentoId,
        fpo.PerguntaPadraoId,
		TRIM(ppr.Resposta)
) sp 
    ON sp.PerguntaOrigemId = pr.PerguntaPadraoId 
    AND sp.PesquisaEstabelecimentoId = pr.PesquisaEstabelecimentoId 
    AND sp.RespostaTextOrigem = TRIM(pr.TextoResposta)
WHERE 
    pr.id IS NOT NULL 
--------
/*
O TRECHO ABAIXO GARANTE QUE APENAS OS ULTIMOS 3 MESES SEJAM ATUALIZADOS,
PARA TRUNCAR TODA A TABELA, COMENTE A LINHA E MARQUE TRUNCAR NO OUTPUT TABLE NO STEP FINAL
*/  
AND pr.Criado_Dt >= DATEADD(DAY, 1, EOMONTH(GETDATE(), -4)) -- VOLTA 3 MESES E RETORNA O PRIEMEIRO DIA DO MES RESULTANTE
--------
AND pr.IsAtivo = 1
AND pr.PerguntaPadraoId NOT IN (
    SELECT DISTINCT fps.PerguntaPadraoId 
    FROM medicsys.oc.stg_formularioresposta frs 
    INNER JOIN medicsys.oc.stg_formulariopergunta fps ON frs.FormularioPerguntaDestinoId = fps.Id 
    WHERE fps.FormularioId = 2
)

AND (
    pr.FormularioPerguntaId IS NULL 
    OR pr.FormularioPerguntaId NOT IN (
        SELECT fr.FormularioPerguntaDestinoId 
        FROM medicsys.oc.stg_formularioresposta fr
    )
)
GROUP BY 
    pe.EstabelecimentoId,
    es.Fantasia,
    pe.Data,
    pr.PerguntaPadraoId,
    pr.TextoResposta,
    pr.PerguntaRespostaId,
    sp.PerguntaDestinoId,
    pr.FormularioPerguntaId,
    pe.Id,
    pp.Pergunta,
    CAST(pr.Alterado_Dt AS DATE);  -- Incluindo no GROUP BY já com CAST


-- TABELA 2: #E2, levando SubResposta e Alterado_Dt
SELECT 
    CAST(tmp_distinct.Data AS DATE) AS Data,
    tmp_distinct.PerguntaPadraoId AS Id,
    tmp_distinct.Pergunta,
    STRING_AGG(CONCAT(tmp_distinct.PerguntaDestinoId, tmp_distinct.PerguntaPadraoId), ',') AS StrSubPergunta,
    tmp_distinct.PerguntaDestinoId,
    tmp_distinct.TextoResposta,
    tmp_distinct.RespostasDestinoTexto AS SubResposta,
    tmp_distinct.EstabelecimentoId,
    tmp_distinct.Alterado_Dt
INTO #E2
FROM (
    SELECT DISTINCT 
        tmp_proc_report.PerguntaPadraoId,
        tmp_proc_report.PerguntaDestinoId,
        tmp_proc_report.Pergunta,
        tmp_proc_report.Data,
        tmp_proc_report.TextoResposta,
        tmp_proc_report.RespostasDestinoTexto,
        tmp_proc_report.EstabelecimentoId,
        tmp_proc_report.Alterado_Dt
    FROM #E1 tmp_proc_report
) AS tmp_distinct
WHERE tmp_distinct.PerguntaDestinoId IS NOT NULL
AND tmp_distinct.PerguntaDestinoId IN (171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 266, 267, 268, 269, 270, 271, 272, 273, 274, 275, 276, 277, 278, 279)
GROUP BY 
    tmp_distinct.PerguntaPadraoId,
    tmp_distinct.Pergunta,
    tmp_distinct.PerguntaDestinoId,
    tmp_distinct.Data,
    tmp_distinct.TextoResposta,
    tmp_distinct.RespostasDestinoTexto,
    tmp_distinct.EstabelecimentoId,
    tmp_distinct.Alterado_Dt;

-- TABELA 3: #E3
SELECT
    A.Data,
    A.Pergunta,
    B.Pergunta AS SubPergunta,
    A.TextoResposta,
    A.SubResposta,
    A.EstabelecimentoId,
    1 AS Total
INTO #E3
FROM #E2 A
LEFT JOIN medicsys.oc.stg_perguntapadrao B ON A.PerguntaDestinoId = B.id
WHERE 
    A.Data IS NOT NULL
AND (B.Pergunta LIKE 'Ao que atribui%' OR B.Pergunta LIKE 'Ao que você atribui%')
ORDER BY 
    A.Data;

-- RESULTADO FINAL
SELECT
[Data]
,EstabelecimentoId
,Pergunta
,CASE
	WHEN TextoResposta = 'Pessimo' THEN 'Péssimo'
	ELSE TextoResposta
END AS Resposta
,SubPergunta
,SubResposta   
,SUM(Total) AS Total
    
INTO #E4
FROM
#E3

GROUP BY 
[Data]
,Pergunta
,SubPergunta
,TextoResposta
,SubResposta
,EstabelecimentoId

ORDER BY
Data
;


-- ACRESCENTA O STAMP DE ATUALIZAÇÃO DA LINHA
SELECT
Data
,EstabelecimentoId
,Pergunta
,Resposta
,SubPergunta
,SubResposta
,Total
,GETDATE() AS stamp

FROM
#E4
```

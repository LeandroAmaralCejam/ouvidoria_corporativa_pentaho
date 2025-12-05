# SQL: Padronização de Perguntas
**Transformação:** `dw_1.ktr`
**Passo:** `dw_PerguntaPadrao`

## Objetivo
Padroniza e categoriza as perguntas do formulário utilizando Common Table Expressions (CTEs) e `CASE` statements.

## Script SQL
```sql
WITH
E1 AS
(
SELECT
Id,
Pergunta,
CASE
	WHEN Id in (94, 98, 143, 114, 091, 102, 105, 251) THEN 'Atendimento - Enfermagem?'
	    WHEN Id in (96, 100, 116, 119, 104, 107) THEN 'Atendimento - Equipe Multiprofissional?'
	    WHEN Id in (142, 148) THEN 'Atendimento - Farmácia?'
	    WHEN Id in (95, 99, 144, 115, 118, 103, 106, 249, 258, 261) THEN 'Atendimento - Médico?'
	    WHEN Id in (139, 145, 253) THEN 'Atendimento - Psicólogo?'
	    WHEN Id in (93, 97, 101, 244, 259, 262) THEN 'Atendimento - Recepção?'
	WHEN Id in (141, 147) THEN 'Atendimento - Serviço Social?'
	    WHEN Id in (140, 146, 256) THEN 'Atendimento - Terapeuta Ocupacional?'
	    WHEN Id in (16, 113, 217, 264) THEN 'Como você avalia este Serviço de Saúde?'
	    WHEN Id in (109, 149, 108, 120, 117) THEN 'Serviço - Instalações?'
	    WHEN Id in (35) THEN 'Atendimento - Triagem?'
	WHEN Id in (36) THEN 'Atendimento - Abertura de Ficha?'
	    WHEN Id in (38) THEN 'Atendimento - Confiança aos serviços prestados?'
	    WHEN Id in (260, 263) THEN 'Atendimento - Administrativo?'
	WHEN Id in (257) THEN 'Atendimento - Assistente Social?'
	    WHEN Id in (245) THEN 'Atendimento - Telefônico?'
	    WHEN Id in (252) THEN 'Atendimento - Fisioterapia?'
	    WHEN Id in (250) THEN 'Atendimento - Educador Físico?'
	    WHEN Id in (255) THEN 'Atendimento - Nutricionista?'
	    WHEN Id in (254) THEN 'Atendimento - Fonoterapia?'
	WHEN Id in (110, 246) THEN 'Serviço - Limpeza?'
	    WHEN Id in (112) THEN 'Serviço - Refeição?'
	    WHEN Id in (167) THEN 'Teleatendimento - Médico' 
	WHEN Id in (111, 247) THEN 'Serviço - Segurança?'
	    WHEN Id in (33, 248) THEN 'Serviço - Acessibilidade?'
	ELSE Pergunta
END AS PerguntaPadronizado

FROM
medicsys.oc.stg_perguntapadrao

WHERE
id NOT IN ('131','132','133','134','135','136','137','138','159',
'233','234','235','236','237','238','239','240','241','242','243',
'283','284','285','286','287','288','289','290','291','292','293','294','295','296','297','298','299','300','301','302',
'303','304','305','306','307','308','309','310','311','312','313','314','315','316','317','318','319','320','321','322',
'323','324','325','326','327','328','329','330','331','332','333','334','335','336','337','338','339','340','341','342',
'343','344','345','346','347','348','349','350','351','352','353','354','355','356','357','358','359','360','361','362',
'363','364','365','366','367','368','369','370','371','372','373','374','375','376','377','378','379','380','381','382',
'383','384','385','386','387','388','389','390','391'

)


-- AND IsAtivo = 1
)


,E2 AS
(
SELECT
Id
,Pergunta
,PerguntaPadronizado
,CASE
	WHEN PerguntaPadronizado = 'Como você avalia este Serviço de Saúde?' THEN 'Atendimento'
	WHEN PerguntaPadronizado = 'Como você avalia este Serviço de Saúde?' THEN '% Global de Satisfação'
	WHEN PerguntaPadronizado = 'Qual a chance de você recomendar o serviço desta Unidade?' THEN 'Recomendação'
	WHEN PerguntaPadronizado LIKE 'Serviço%' THEN 'Serviço'
	WHEN PerguntaPadronizado LIKE 'Ao que%' THEN 'Subpergunta'	
	WHEN Id = '280' THEN 'Serviço'
	WHEN Id = '281' THEN 'Serviço'
	ELSE 'Atendimento'
END AS 'Categoria Pergunta'

,CASE
	WHEN Pergunta LIKE 'Ao que atribui como péssim%' THEN 'Péssimo'
	WHEN Pergunta LIKE 'Ao que atribui como ruim%' THEN 'Ruim'
	WHEN Pergunta LIKE 'Ao que você atribui como péssim%' THEN 'Péssimo'
	WHEN Pergunta LIKE 'Ao que você atribui como ruim%' THEN 'Ruim'
	ELSE NULL
END AS 'Categoria Subpergunta'


FROM
E1
)


SELECT
Id
,Pergunta AS PerguntaExtenso
,PerguntaPadronizado
,[Categoria Pergunta]
,[Categoria Subpergunta]
,CASE
	WHEN [Categoria Pergunta] LIKE 'Atendimento%' THEN '1'
	WHEN [Categoria Pergunta] LIKE 'Serviço%' THEN '2'
	WHEN [Categoria Pergunta] LIKE '% Global%' THEN '3'
	WHEN [Categoria Pergunta] LIKE 'Recomendação%' THEN '4'
	ELSE 5
END AS 'Indice'

,REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(PerguntaPadronizado
,'Atendimento - ','')
,'Serviço - ','')
,'Técnico de Enfermagem?','Enfermagem?')
,'Atendimento do Enfermeiro?','Atendimento - Enfermeiro?')
,'Atendimento das copeiras?','Atendimento - Copeiras?')
,' Rouparia?','Rouparia?')
AS Pergunta


FROM
E2
```

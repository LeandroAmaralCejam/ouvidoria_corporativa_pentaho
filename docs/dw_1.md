# Estágio 3: Transformação Data Warehouse (DW)

Esta transformação recebe os dados brutos da área de Staging, limpa-os e estrutura-os para o Data Warehouse.

## Origem
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `oc` (Tabelas Staging)

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DW)

## Processo ETL
O processo envolve ler das tabelas de Staging, aplicar operações de ordenação (necessárias para alguns passos do Pentaho ou apenas para organização) e carregar nas tabelas `dw_`.

| Tabela Staging | Tabela DW de Destino | Descrição |
| :--- | :--- | :--- |
| `stage_PerguntaPadrao` | `dw_PerguntaPadrao` | Perguntas Padrão |
| `stage_perguntaresposta` | `dw_perguntaresposta` | Pares de Pergunta/Resposta |
| `stage_formulario` | `dw_formulario` | Formulários |
| `stage_ClassificacaoManifestacao` | `dw_ClassificacaoManifestacao` | Classificações |
| `stage_MotivoManifestacao` | `dw_MotivoManifestacao` | Motivos |
| `stage_Manifestacao` | `dw_Manifestacao` | Manifestações |
| `stage_Regionais_MedicSys` | `dw_Regionais_MedicSys` | Regionais |

## Lógica Principal
- **Ordenação:** Todos os fluxos são ordenados por seus respectivos IDs (ex: `Id`, `Id_Medicsys`) para garantir inserção ordenada ou facilitar junções/merges futuros se necessário.
- **Tratamento de Manifestação:**
    - A transformação lê 3 meses de histórico para `Manifestacao` para atualizar registros recentes enquanto mantém o histórico mais antigo constante, otimizando o desempenho.
    - *Nota: Existe uma lógica comentada para truncamento total vs atualizações de janela deslizante.*

## Consultas SQL Utilizadas

### Padronização de Perguntas (`dw_PerguntaPadrao`)
Esta etapa utiliza uma consulta complexa com CTEs (`WITH`) para limpar e categorizar as perguntas do formulário.

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

### Agrupamento de Motivos (`dw_MotivoManifestacao`)
Agrupa motivos detalhados em categorias macro.
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

### Dados Regionais (`dw_Regionais_MedicSys`)
Realiza um join com a tabela de estabelecimentos para determinar o tipo de contrato.
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



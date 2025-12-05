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
WITH E1 AS (
    SELECT
        Id,
        Pergunta,
        CASE
            WHEN Id in (94, 98, 143, ...) THEN 'Atendimento - Enfermagem?'
            -- ... (múltiplos CASES para padronização)
            ELSE Pergunta
        END AS PerguntaPadronizado
    FROM medicsys.oc.stg_perguntapadrao
    WHERE id NOT IN ('131', ...) -- Exclusão de IDs legados
),
E2 AS (
    SELECT
        Id,
        Pergunta,
        PerguntaPadronizado,
        CASE
            WHEN PerguntaPadronizado LIKE 'Serviço%' THEN 'Serviço'
            ELSE 'Atendimento'
        END AS 'Categoria Pergunta',
        -- ...
    FROM E1
)
SELECT * FROM E2;
```

### Agrupamento de Motivos (`dw_MotivoManifestacao`)
Agrupa motivos detalhados em categorias macro.
```sql
SELECT
    Id,
    MotivoManifestacaoDesc,
    CASE 
        WHEN Id IN (1, 2, 10, ...) THEN 'Comportamento inadequado do profissional'
        WHEN Id IN (3, 4) THEN 'Elogio'
        -- ...
    END AS MotivoGrupo
FROM medicsys.oc.stg_MotivoManifestacao
WHERE Id IS NOT NULL
```

### Dados Regionais (`dw_Regionais_MedicSys`)
Realiza um join com a tabela de estabelecimentos para determinar o tipo de contrato.
```sql
SELECT
    A.Unidade,
    A.[Nível de Atenção],
    A.Município,
    A.[Tipo de Unidade],
    A.Id_Medicsys,
    A.Regional,
    B.TipoContrato AS Tipo
FROM medicsys.oc.stg_Regionais_MedicSys A
LEFT JOIN (
    SELECT id, 
    CASE 
        WHEN ContratoId IN (8, 11, ...) THEN 'Convenio'
        WHEN ContratoId = 2 THEN 'Sede'
        ELSE 'Contrato'
    END AS TipoContrato
    FROM medicsys.oc.stg_estabelecimento
) B ON A.Id_Medicsys = B.id
```



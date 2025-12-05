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


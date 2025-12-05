# Estágio 4: Agregação Data Mart (DM)

Esta transformação final agrega os dados detalhados do DW em tabelas de resumo para relatórios e dashboards.

## Origem
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DW)

## Destino
- **Banco de Dados:** SQL Server (`con_db_medicsys_cejam`)
- **Schema:** `medicsys.oc` (Tabelas DM)

## Transformações
1. **Agregação Manifestação (`dm_Manifestacao`)**
    - **Origem:** `dw_Manifestacao`
    - **Lógica de Filtro:** Seleciona registros criados nos últimos 3 meses (janela deslizante).
    - **Chaves de Agrupamento:**
        - `Data` (normalizada para o dia 1 do mês)
        - `TipoAtendimentoManifestacaoId`
        - `PerfilManifestacaoId`
        - `ClassificaoManifestacaoId`
        - `MotivoManifestacaoId`
        - `EstabelecimentoTratado`
    - **Métricas:** `Total` (Soma da contagem).
    - **Saída:** A tabela `dm_Manifestacao` é atualizada com esses snapshots agregados.

2. **Agregação Melhorias (`dm_melhorias`)**
    - Lógica similar aplicada para "Melhorias" se aplicável, agregando contagens baseadas em categorização.

## Propósito
Estas tabelas são otimizadas para consumo direto pelo Power BI ou outras ferramentas de visualização, fornecendo totais mensais pré-agregados por várias dimensões.


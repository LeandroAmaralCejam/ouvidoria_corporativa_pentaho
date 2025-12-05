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

## Consultas SQL Detalhadas

Para facilitar a manutenção e leitura, as consultas SQL complexas foram documentadas em páginas separadas:

### 1. Padronização de Perguntas
- **[Ver SQL: Padronização de Perguntas](sql/dw_1_perguntapadrao.md)**
- **Transformação:** `dw_PerguntaPadrao`
- **Descrição:** Utiliza CTEs e `CASE` statements extensos para normalizar textos de perguntas e agrupar variações.

### 2. Agrupamento de Motivos
- **[Ver SQL: Agrupamento de Motivos](sql/dw_1_motivomanifestacao.md)**
- **Transformação:** `dw_MotivoManifestacao`
- **Descrição:** Mapeia centenas de motivos detalhados para macro-grupos de análise.

### 3. Dados Regionais
- **[Ver SQL: Enriquecimento Regional](sql/dw_1_regionais.md)**
- **Transformação:** `dw_Regionais_MedicSys`
- **Descrição:** Join estratégico para determinar o tipo de contrato (Sede, Convênio, Contrato) baseado em IDs.

### 4. Manifestação (Fato)
- **[Ver SQL: Manifestação Enriquecida](sql/dw_1_manifestacao.md)**
- **Transformação:** `dw_Manifestacao`
- **Descrição:** Tabela principal de fatos. Inclui lógica de negócio para correção de estabelecimento (`EstabelecimentoTratado`).

### 5. Classificação de Manifestação
- **[Ver SQL: Classificação](sql/dw_1_classificacaomanifestacao.md)**
- **Transformação:** `dw_ClassificacaoManifestacao`
- **Descrição:** Define o peso da manifestação (Positiva/Negativa).

### 6. Tabelas Auxiliares
- **[Ver SQL: Tabelas Simples](sql/dw_1_simples.md)**
- **Transformação:** `dw_formulario`, `dw_perguntaresposta`
- **Descrição:** Cargas diretas para tabelas de domínio.




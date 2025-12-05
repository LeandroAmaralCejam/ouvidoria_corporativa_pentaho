# Estágio 1: Extração do Sistema Helper (MySQL)

Esta transformação é responsável por extrair os dados brutos do banco de dados operacional (MySQL) e carregá-los na Área de Staging (SQL Server).

## Sistema de Origem
- **Tipo de Banco de Dados:** MySQL
- **Schema:** `cejam`
- **Conexão:** `con_mysql_medicsys`

## Sistema de Destino (Staging)
- **Tipo de Banco de Dados:** SQL Server
- **Schema:** `oc` (schema implícito de Staging)
- **Conexão:** `con_db_medicsys_cejam`

## Processo ETL
O processo extrai dados das seguintes tabelas e os carrega em suas respectivas tabelas `stage_`. Algumas tabelas usam uma estratégia de carga incremental baseada em datas, enquanto outras são cargas completas (full loads).

| Tabela de Origem | Tabela de Destino | Tipo de Carga | Descrição |
| :--- | :--- | :--- | :--- |
| `ClassificacaoManifestacao` | `stage_ClassificacaoManifestacao` | Completa | Tipos de classificação |
| `Manifestacao` | `stage_Manifestacao` | Incremental | Registros principais de manifestação |
| `MotivoManifestacao` | `stage_MotivoManifestacao` | Completa | Motivos para manifestações |
| `PerfilManifestacao` | `stage_PerfilManifestacao` | Completa | Perfis |
| `TipoAtendimentoManifestacao` | `stage_TipoAtendimentoManifestacao` | Completa | Tipos de atendimento |
| `formulario` | `stage_formulario` | Completa | Definições de formulários |
| `formulariopergunta` | `stage_formulariopergunta` | Completa | Perguntas dos formulários |
| `formularioresposta` | `stage_formularioresposta` | Completa | Respostas dos formulários |
| `estabelecimento` | `stage_estabelecimento` | Completa | Estabelecimentos |
| `perguntapadrao` | `stage_perguntapadrao` | Completa | Perguntas padrão |
| `perguntaresposta` | `stage_perguntaresposta` | Completa | Respostas das perguntas |
| `pesquisaresposta` | `stage_pesquisaresposta` | Incremental | Respostas de pesquisa |
| `pesquisaestabelecimento` | `stage_pesquisaestabelecimento` | Incremental | Estabelecimentos de pesquisa |

## Lógica de Transformação
- **Atualização Completa (Full):** Trunca (limpa) a tabela de destino e recarrega todos os dados.
- **Incremental:** Usa um lookup de `max_date` para buscar apenas registros criados ou modificados após a última carga bem-sucedida.


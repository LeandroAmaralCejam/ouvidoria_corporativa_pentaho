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

## Consultas SQL Utilizadas

### Extração de Manifestação (Incremental)
```sql
SELECT
Id
,Criado_Dt
,Criado_Por
,Alterado_Dt
,Alterado_Por
,IsAtivo
,IsDeleted
,Anonimo
,Sigilo
,CNPJ
,RazaoSocial
,NomeFantasia
,CPF
,DATE_FORMAT(DataNascimento, '%Y-%m-%d %H:%i:%s') AS DataNascimento
,Nome
,NomeSocial
,Email
,CEP
,Endereco
,Numero
,Complemento
,Bairro
,CidadeId
,Estado
,Telefone
,TelefoneContato
,Celular
,ManifestacaoDescricao
,Evidencia
,Providencia
,Justificativa
,TipoLocalEnvio
,Status
,DataOcorrencia
,TipoAtendimentoManifestacaoId
,EstabelecimentoId
,EstabelecimentoEnvioId
,EstabeleciomentoOcorrenciaId
,Protocolo
,EstadoEstab
,MunicipioEstab
,MidiaSocialId
,GeneroId
,DepartamentoId
,TempoResposta
,Avaliacao
,PerfilManifestacaoId
,ClassificaoManifestacaoId
,MotivoManifestacaoId
,IsTermo
FROM cejam.Manifestacao
WHERE id IS NOT NULL
AND Alterado_Dt > ? -- Parâmetro injetado pelo passo de lookup
```

### Consultas Padrão (Carga Completa)
Para a maioria das tabelas auxiliares, a consulta segue o padrão:
```sql
SELECT * FROM cejam.NomeDaTabela;
```
Exemplos:
- `SELECT * FROM cejam.ClassificacaoManifestacao;`
- `SELECT * FROM cejam.MotivoManifestacao;`
- `SELECT * FROM cejam.PerfilManifestacao;`

### Consultas de Lookup (Max Date)
Utilizadas para determinar o ponto de corte da carga incremental.
```sql
SELECT MAX(Alterado_Dt) AS MAX_DATE FROM medicsys.oc.stg_manifestacao WHERE id IS NOT NULL
```
*(Similar para `stg_pesquisaestabelecimento` e `stg_pesquisaresposta`)*



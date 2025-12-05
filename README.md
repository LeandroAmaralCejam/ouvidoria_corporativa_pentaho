# Ouvidoria Corporativa - ETL Pentaho

Este projeto contém os processos de ETL (Extração, Transformação, Carga) para o sistema de "Ouvidoria Corporativa", implementados utilizando **Pentaho Data Integration (PDI)**.

## Visão Geral do Projeto
O objetivo principal deste projeto é consolidar dados de várias fontes (bancos de dados operacionais MySQL, arquivos Excel) em um Data Warehouse e subsequentemente em um Data Mart para suportar dashboards de Business Intelligence.

O projeto tem como fonte principal o sistema **MedicSys**, que é alimentado nas unidades de saúde, através de pesquisas de satisfação de usuário.

## Requisitos do Sistema

Este projeto foi desenvolvido e validado utilizando o **Pentaho Data Integration (PDI) versão 9.3** (Build `9.3.0.0-428`).

> [!IMPORTANT]
> É um **requisito mínimo** utilizar esta versão (ou superior compatível) para garantir que todos os jobs e transformações, bem como o versionamento dos arquivos XML do PDI, funcionem perfeitamente após clonar o repositório. Versões anteriores podem causar conflitos na leitura dos arquivos `.kjb` e `.ktr`.

## Configuração e Instalação

Para configurar o projeto localmente, clone o repositório no diretório de produção usando o seguinte comando:

```bash
git clone https://github.com/LeandroAmaralCejam/ouvidoria_corporativa.git C:\github\prod\ouvidoria_corporativa
```

> [!NOTE]
> Certifique-se de que a pasta de destino `C:\github\prod\ouvidoria_corporativa` esteja vazia, ou, não exista antes de executar o comando de clone.

## Conceitos Fundamentais

Para facilitar o entendimento do fluxo de dados, o projeto está estruturado em três camadas lógicas padrão de mercado:

- **Stage Area (Área de Palco):** É a camada de entrada. Sua função é extrair e armazenar uma cópia bruta dos dados das fontes (Sistemas, Planilhas), sem aplicar regras de negócio complexas. Isso garante que o processamento pesado ocorra dentro do ambiente de DW, sem impactar a performance dos sistemas de origem.
- **Data Warehouse (DW):** É a camada de integração e limpeza. Aqui os dados da Stage são padronizados, corrigidos e historicizados. É a "fonte da verdade" dos dados tratados.
- **Data Mart (DM):** É a camada de apresentação. Os dados do DW são refinados, agregados e modelados (geralmente em Schemas Estrela) para atender a perguntas de negócio específicas e alimentar painéis de BI.

## Arquitetura ETL

O processo de ETL segue a sequência dessas camadas, detalhado abaixo:

### 1. Camada Stage (Extração)
A extração é dividida em duas etapas para organizar melhor as diferentes naturezas das fontes de dados. Ambas carregam dados para a área de *Staging* do SQL Server.

*   **[Stage 1 - Dados Operacionais (MySQL)](docs/stage_1.md)**
    *   Extrai dados do sistema principal da Ouvidoria.
    *   **Contém:** Tabelas fundamentais como `Manifestacao`, `MotivoManifestacao`, `Classificacao`, `PesquisaResposta`, entre diversas outras tabelas transacionais.
*   **[Stage 2 - Dados Auxiliares (Excel)](docs/stage_2.md)**
    *   Processa arquivos complementares geridos manualmente ou por outros setores.
    *   **Contém:** Tabelas de `Metas`, `Unidades` e de-paras de `Regionais`.

### 2. Camada Data Warehouse (Transformação)
Responsável por transformar os dados brutos em informações estruturadas.

*   **[DW 1 - Corporativo](docs/dw_1.md)**
    *   Processo principal que trata as manifestações e seus relacionamentos.
    *   Padroniza perguntas, agrupa motivos e enriquece os dados com informações geográficas e de contrato.
*   **[DW 2 - Pesquisa de Satisfação](docs/dw_2.md)**
    *   Processo focado especificamente na modelagem das respostas das pesquisas para análise de qualidade.

### 3. Camada Data Mart (Agregação)
Gera as tabelas finais para consumo dos Dashboards.

*   **[DM 1 - Analytics Ouvidoria](docs/dm_1.md)**
    *   Lê do DW e cria tabelas agregadas e fatos otimizadas.
    *   Gera visões analíticas de volumetria, tempos de resposta e satisfação do usuário.

### 4. Controle de Carga
O projeto utiliza um mecanismo de "Stamps" para garantir cargas incrementais confiáveis:
- **`stamp_stage`**: Marca temporal da extração na origem.
- **`stamp_job`**: Controle de execução do pacote ETL.


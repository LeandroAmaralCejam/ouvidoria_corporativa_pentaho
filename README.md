# Ouvidoria Corporativa - ETL Pentaho

Este projeto contém os processos de ETL (Extração, Transformação, Carga) para o sistema de "Ouvidoria Corporativa", implementados utilizando **Pentaho Data Integration (PDI)**.

## Visão Geral do Projeto
O objetivo principal deste projeto é consolidar dados de várias fontes (bancos de dados operacionais MySQL, arquivos Excel) em um Data Warehouse e subsequentemente em um Data Mart para suportar dashboards de Business Intelligence.

## Configuração e Instalação

Para configurar o projeto localmente, clone o repositório no diretório de produção usando o seguinte comando:

```bash
git clone https://github.com/LeandroAmaralCejam/ouvidoria_corporativa_pentaho.git C:\github\prod\ouvidoria_corporativa_pentaho
```

> [!NOTE]
> Certifique-se de que a pasta de destino `C:\github\prod\ouvidoria_corporativa_pentaho` não exista antes de executar o comando de clone, ou esteja vazia.

## Arquitetura ETL

O processo de ETL é dividido em estágios lógicos, cada um responsável por uma parte específica do pipeline de dados.

### 1. Estágio 1: Extração de Dados Operacionais
- **Origem:** MySQL (schema `cejam`)
- **Destino:** Área de Staging SQL Server
- **Detalhes:** [Ver Documentação Técnica](docs/stage_1.md)

### 2. Estágio 2: Extração de Dados Auxiliares
- **Origem:** Arquivos Excel (Sharepoint/Drive de Rede)
- **Destino:** Área de Staging SQL Server
- **Detalhes:** [Ver Documentação Técnica](docs/stage_2.md)

### 3. Transformação Data Warehouse (DW)
- **Origem:** Área de Staging
- **Destino:** Data Warehouse (`medicsys.oc`)
- **Detalhes:** [Ver Documentação Técnica](docs/dw_1.md)

### 4. Agregação Data Mart (DM)
- **Origem:** Data Warehouse
- **Destino:** Data Mart (Tabelas Agregadas)
- **Detalhes:** [Ver Documentação Técnica](docs/dm_1.md)


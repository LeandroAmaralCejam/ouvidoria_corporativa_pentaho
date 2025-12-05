# Ouvidoria Corporativa - ETL Pentaho

This project contains the ETL (Extract, Transform, Load) processes for the "Ouvidoria Corporativa" system, implemented using **Pentaho Data Integration (PDI)**.

## Project Overview
The main goal of this project is to consolidate data from various sources (MySQL operational databases, Excel files) into a Data Warehouse and subsequently a Data Mart to support Business Intelligence dashboards.

## Setup & Installation

To set up the project locally, clone the repository to the production directory using the following command:

```bash
git clone https://github.com/LeandroAmaralCejam/ouvidoria_corporativa_pentaho.git C:\github\prod\ouvidoria_corporativa
```

> [!NOTE]
> Ensure that the destination folder `C:\github\prod\ouvidoria_corporativa` does not exist before running the clone command, or is empty.

## ETL Architecture

The ETL process is divided into logical stages, each responsible for a specific part of the data pipeline.

### 1. Stage 1: Operational Data Extraction
- **Source:** MySQL (`cejam` schema)
- **Destination:** SQL Server Staging Area
- **Details:** [View Technical Documentation](docs/stage_1.md)

### 2. Stage 2: Auxiliary Data Extraction
- **Source:** Excel Files (Sharepoint/Network Drive)
- **Destination:** SQL Server Staging Area
- **Details:** [View Technical Documentation](docs/stage_2.md)

### 3. Data Warehouse (DW) Transformation
- **Source:** Staging Area
- **Destination:** Data Warehouse (`medicsys.oc`)
- **Details:** [View Technical Documentation](docs/dw_1.md)

### 4. Data Mart (DM) Aggregation
- **Source:** Data Warehouse
- **Destination:** Data Mart (Aggregated Tables)
- **Details:** [View Technical Documentation](docs/dm_1.md)

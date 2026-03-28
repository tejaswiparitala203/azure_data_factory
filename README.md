# Azure Data Factory — ETL Pipeline Project

An end-to-end data engineering project built on **Azure Data Factory** that automates ingestion, transformation, and orchestration of CSV data using Azure Blob Storage, parameterised pipelines, data flows, and CI/CD via GitHub integration.

---

## Project Overview

This project demonstrates a production-style ADF setup with a fully orchestrated pipeline hierarchy, event-driven and scheduled triggers, reusable parameterised datasets, a mapping data flow for CSV transformation, and ARM template-based deployment for repeatability.

The factory (`tejaswinifactory`) reads raw CSV files from Azure Blob Storage, applies transformations using a mapping data flow, and writes processed output back to Blob Storage — all automated via triggers and managed through GitHub source control.

---

## Architecture

```
Azure Blob Storage (raw files)
        │
        ▼
┌─────────────────────────────┐
│        prodpipeline         │  ◄── mantrigger (Blob event)
│     (Top-level orchestrator)│  ◄── selectedfilestrigger (Schedule)
└──────────┬──────────────────┘
           │
     ┌─────┴──────┐
     ▼            ▼
pipelinemanager   selectedfiles
     │                 │
     ├─ Copy CSV        ├─ GetMetadata
     ├─ Delete source   ├─ ForEach (loop files)
     └─ Execute ──►     └─ Execute ──►
        pipelinegit        Varpipeline + transformcsv (Data Flow)
        (GitHub copy)
           │
           ▼
Azure Blob Storage (processed output)
```

---

## Pipelines

| Pipeline | Activities | Purpose |
|---|---|---|
| `prodpipeline` | ExecutePipeline × 2 | Top-level orchestrator — fans out to child pipelines |
| `pipelinemanager` | Copy, Delete, ExecutePipeline | Copies CSV to destination, deletes source file, triggers GitHub pipeline |
| `selectedfiles` | GetMetadata, ForEach, ExecuteDataFlow | Loops through files, applies data flow transformation |
| `pipelinegit` | Copy | Copies data sourced from GitHub (HTTP linked service) |
| `Varpipeline` | GetMetadata, SetVariable | Reads file metadata and stores values as pipeline variables |

---

## Data Flow

**`transformcsv`** — a mapping data flow that reads from a CSV source dataset, applies transformations, and writes to a sink dataset. Used inside `selectedfiles` pipeline via a ForEach loop.

---

## Datasets

All datasets are `DelimitedText` (CSV) format backed by Azure Blob Storage:

| Dataset | Role |
|---|---|
| `csvsource` / `csv` | Source CSV files for ingestion |
| `dataflowsource` / `dataflow_sink` | Input and output for the `transformcsv` data flow |
| `metadata` / `metadatads` | Metadata inspection datasets |
| `parametrisedsource` | Parameterised source — demonstrates dynamic dataset binding |
| `reporting` / `reporting_sink` | Reporting layer output |
| `DelimitedText1` / `DelimitedText2` | General-purpose text datasets |

---

## Linked Services

| Linked Service | Type | Purpose |
|---|---|---|
| `AzureBlobStorage1` | Azure Blob Storage | Primary storage connection |
| `datafactorylinkedservice` | Azure Blob Storage | Secondary storage connection |
| `linked_git` | HTTP Server | Reads raw files from GitHub (`raw.githubusercontent.com`) |

---

## Triggers

| Trigger | Type | Description |
|---|---|---|
| `mantrigger` | Blob Events Trigger | Fires automatically when a new file lands in Blob Storage |
| `selectedfilestrigger` | Schedule Trigger | Runs the pipeline on a defined schedule |

---

## Integration Runtime

| Runtime | Type | Purpose |
|---|---|---|
| `azureintegrationRuntime` | Self-Hosted | Handles data movement in a custom network environment |

---

## Deployment

This project uses ARM templates for repeatable deployments across environments.

### Prerequisites

- Azure subscription
- Azure Data Factory instance
- Azure Blob Storage account
- Azure CLI installed



---

## Repository Structure

```
azure_data_factory/
├── ARMTemplateForFactory.json           # Main ARM template (all resources)
├── ARMTemplateParametersForFactory.json # Parameters file
├── linkedTemplates/
│   ├── ArmTemplate_master.json          # Master linked template
│   ├── ArmTemplateParameters_master.json
│   ├── ArmTemplate_0.json               # Linked services, datasets, pipelines
│   └── ArmTemplate_1.json               # prodpipeline and triggers
└── factory/
    ├── tejaswinifactory_ARMTemplateForFactory.json
    └── tejaswinifactory_ARMTemplateParametersForFactory.json
```

---

## Key Concepts Demonstrated

- **Pipeline orchestration** — parent/child pipeline pattern with `ExecutePipeline`
- **Event-driven architecture** — Blob storage event trigger for real-time processing
- **Parameterised datasets** — dynamic source binding via pipeline parameters
- **Mapping data flows** — code-free CSV transformation using `transformcsv`
- **Metadata-driven pipelines** — `GetMetadata` + `ForEach` for dynamic file looping
- **Source control integration** — ADF connected to GitHub for full CI/CD
- **ARM template deployment** — infrastructure-as-code using linked templates
- **Self-hosted integration runtime** — custom network data movement

---

## Technologies Used

- **Azure Data Factory** — pipeline orchestration and ETL
- **Azure Blob Storage** — raw and processed data storage
- **ARM Templates** — infrastructure as code
- **GitHub** — source control and CI/CD integration
- **Azure Integration Runtime (Self-Hosted)** — custom runtime environment

---

## Author

**Tejaswini Paritala**  

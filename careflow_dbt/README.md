
# CareFlow - dbt Data Transformation

## Overview

This dbt project is part of the CareFlow Clinical Pathway Process Mining system.

It is responsible for preparing and standardising hospital event-log data stored in Google BigQuery. The staging layer provides a consistent structure for downstream data analysis and process-mining workflows.

## Project Configuration

| Setting | Value |
|---|---|
| Project name | `careflow_dbt` |
| dbt version | 1.12.6 |
| dbt adapter | dbt-bigquery 1.12.1 |
| Database | Google BigQuery |
| Dataset | `careflow_analytics` |
| Location | US |
| Model materialisation | View |

## Data Source

The staging model reads from the following BigQuery table:

```text
careflow-analytics-98234.careflow_analytics.event_logs
```

The source contains hospital event records associated with patient encounters.

Important source fields include:

- `event_id`
- `case_id`
- `activity`
- `timestamp`
- `department`
- `event_sequence`
- `resource_role`
- `resource_id`
- `event_detail`
- `arrival_mode`
- `disposition`

## Staging Model

The current staging model is:

`models/staging/stg_event_logs.sql`

It is configured as a BigQuery view.

### Transformations performed

| Transformation | Purpose |
|---|---|
| Cast `event_id` to STRING | Standardise event identifiers |
| Cast `case_id` to STRING | Standardise patient-case identifiers |
| Trim `activity` | Remove unwanted surrounding whitespace |
| SAFE_CAST timestamp | Safely convert timestamp values |
| Replace blank department with `Unknown` | Handle empty department values |
| Retain workflow attributes | Preserve event sequence and operational information |

The resulting model provides standardised event data for further analysis.

## Data Quality Tests

The staging model has six configured schema tests.

| Column | Test |
|---|---|
| `case_id` | not_null |
| `event_id` | not_null |
| `event_id` | unique |
| `activity` | not_null |
| `event_timestamp` | not_null |
| `department` | not_null |

These tests are intended to check identifier completeness, event uniqueness, activity completeness, timestamp validity, and department availability.

**Execution status:** The dbt project successfully passed parsing. An attempted execution of the six staging tests encountered a BigQuery permission error. Successful execution and deployment of the staging view have not yet been confirmed.

## Project Structure

```text
careflow_dbt/
|
|-- models/
|   |-- staging/
|   |   |-- stg_event_logs.sql
|   |   |-- schema.yml
|   |
|   |-- example/
|
|-- analyses/
|-- macros/
|-- seeds/
|-- snapshots/
|-- tests/
|
|-- dbt_project.yml
|-- README.md
```

## Running the dbt Project

Run commands from the main repository directory.

### 1. Check the dbt configuration

```bash
dbt debug --project-dir careflow_dbt --profiles-dir ~/.dbt
```

### 2. Parse the project

```bash
dbt parse --project-dir careflow_dbt --profiles-dir ~/.dbt
```

### 3. Build the staging model

```bash
dbt run --project-dir careflow_dbt --profiles-dir ~/.dbt --select stg_event_logs
```

### 4. Run staging tests

```bash
dbt test --project-dir careflow_dbt --profiles-dir ~/.dbt --select stg_event_logs
```

Successful execution requires a valid BigQuery connection and the necessary permissions.


## Role in the CareFlow Pipeline

The dbt staging layer is designed to support the broader CareFlow workflow:

1. Hospital event records are stored in BigQuery.
2. dbt standardises the source records through the staging model.
3. Data quality tests check important event-log fields.
4. Prepared event data can support downstream process analysis.
5. Process-mining techniques help explore activities, transitions, and repeated workflow steps.

**Current integration note:** The Streamlit dashboard currently reads directly from the original BigQuery event table or a CSV upload. It does not yet consume the dbt staging view.

## Important Notes

- The staging model is configured as a view, not a physical table.
- `SAFE_CAST` prevents invalid timestamp conversions from causing the transformation query to fail, but invalid values can still become NULL.
- Data quality tests must be executed successfully before treating the model as fully validated.
- BigQuery credentials and secrets must not be committed to the repository.

## Project Goal

The goal of this dbt component is to establish a reliable and standardised event-data preparation layer for CareFlow, supporting consistent data quality checks and future integration with clinical pathway process-mining analysis.

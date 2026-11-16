
# FreshCart Data Engineering Pipelines

A comprehensive Data Engineering project demonstrating Git-based collaboration, CI/CD automation, Snowflake ELT workflows, Mage-based machine learning pipelines, and real-time data streaming using RabbitMQ.

## Project Directory Structure

- `elt/` – Contains the FreshCart ELT workflow and Snowflake-related scripts.
- `iris_project/` – Implements the Iris classification pipeline using Mage.
- `stream/` – Includes RabbitMQ message dispatchers and consumer drivers.
- `tests/` – Contains automated test cases for validating project functionality.
- `.github/workflows/` – Stores GitHub Actions configurations for CI automation.

## FreshCart ELT Pipeline (Part B)

The FreshCart ELT workflow is executed through the main orchestration module `elt.run_pipeline`.

Before running the pipeline, configure your Snowflake credentials and other required settings in the `.env` file. Use `.env.example` as a reference for the required configuration variables.

### Running the Pipeline

**1. Run the baseline pipeline with mock data**

Execute the following command to run the workflow using simulated source data:

```powershell
python -m elt.run_pipeline --mode mock
```

**2. Run transformations on previously loaded data**

To rerun the transformation process without extracting and loading an additional batch of data, execute:

```powershell
python -m elt.run_pipeline --mode transform-only
```

This is the default execution mode.

**3. Execute a specific transformation stage**

Individual transformation stages can also be executed separately. For example, to run the Gold layer:

```powershell
python -m elt.run_pipeline --stage gold
```

### Additional Pipeline Information

- **Pincode Validation:** Deterministic mock responses are used by default. To retrieve actual API responses, enable the `--live-pincodes` option.
- **SQL Transformations:** All SQL scripts used for data processing are located inside `elt/sql/`.
- **Automated Testing:** Python-based business rules and logic tests can be executed using:

```powershell
python -m pytest
```

- **Live Extraction:** Actual source data extraction requires the appropriate service URLs and authentication tokens to be configured in `.env`. Refer to `.env.example` for the required variable names.

### Bronze Layer File Ingestion

Bronze file ingestion is controlled through the additional SQL migration:

`elt/sql/01_ingestion_files.sql`

The ingestion mechanism maintains a tracking ledger to prevent duplicate file processing and improve recovery from failed loads.

The main features include:

1. **File Identification:** A SHA-256 hash is generated using the normalized source and staged `source/date/filename` path.
2. **Checksum Validation:** The compressed file's content checksum is used to identify changes in files that already have an existing identity.
3. **Loading Tracking:** Each file is registered with a `LOADING` status before the PUT and COPY operations begin.
4. **Successful Ingestion:** The status is updated to `LOADED` only after COPY results and Bronze source-file row counts have been verified.
5. **Failure Management:** Failed ingestion attempts are marked `FAILED` and can be retried after the necessary checks.

### Recovery and Concurrency Management

An unexpected process termination may leave a file with a `LOADING` status. Before manually updating its status:

- Inspect the corresponding Bronze `SRC_FILE` row count.
- Ensure that no other loader is currently active.
- Reconcile the ledger status based on the actual ingestion outcome.

The tracking ledger is implemented as a standard Snowflake table for compatibility with trial accounts. Its `FILE_ID` column does not enforce uniqueness, so ingestion must follow a single-writer approach.

A host-level lock prevents multiple local processes from running simultaneously. Jobs running on different machines, including scheduled workflows and manual loaders, must also be coordinated to avoid concurrent writes.

The loader checks for duplicate ledger entries and blocks ambiguous legacy Bronze records that have no corresponding ledger entry. These records must be reconciled before retrying ingestion.

**Important:** Snowflake PUT, COPY, and ledger DML operations are not part of a single atomic transaction. Consequently, failed or interrupted ingestion attempts require status verification and row-count reconciliation before another attempt is made.

## Assignment Components

The complete assignment consists of the following sections:

- **Part A – Git and CI/CD:** Version control and continuous integration workflows.
- **Part B – FreshCart ELT on Snowflake:** Data extraction, loading, and transformation.
- **Part C – ML Pipeline with Mage:** Iris dataset classification workflow.
- **Part D – Food-Order Streaming with RabbitMQ:** Real-time message dispatching and processing.
- **Part E – Final Report and Viva:** Project documentation and technical demonstration.

---

**Repository:** `freshcart-pipelines`


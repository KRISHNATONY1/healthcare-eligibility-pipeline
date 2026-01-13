
# Healthcare Eligibility Pipeline

## Overview
This project implements a Spark-based data pipeline that ingests healthcare eligibility files from multiple partner sources, standardizes the data into a common schema, and outputs a unified eligibility dataset. The pipeline is designed to be easily extensible as new partners are onboarded.

## How to Run the Pipeline

### Prerequisites
* Databricks workspace with Unity Catalog enabled
* Access to the `workspace` catalog
* Python / PySpark runtime

### Setup

1. Upload partner input files into a Unity Catalog volume:

  /Volumes/workspace/default/eligibility_data/
   ├── acme/acme.txt
   └── bettercare/bettercare.xlsx

2. Ensure the repository is cloned into Databricks using the Git integration.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Execution

Run the pipeline by executing the `eligibility_pipeline.py` notebook:

```This part of the code : python
final_df = run_pipeline()
final_df.show(truncate=False)
```

The pipeline will:

* Read each partner file
* Normalize column names and data types
* Standardize the schema
* Add a partner identifier
* Combine all partner data into a single DataFrame

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Output Schema

The final unified dataset follows this structure:

| Column Name  | Description                        |
| ------------ | ---------------------------------- |
| external_id  | Partner-specific member identifier |
| first_name   | Member first name                  |
| last_name    | Member last name                   |
| dob          | Date of birth                      |
| email        | Member email address               |
| phone        | Member phone number                |
| partner_code | Source partner identifier          |

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## How to Add a New Partner

To onboard a new partner, follow these steps:

1. **Add partner data**

   * Place the partner file under:

     ```
     Data/<partner_name>/
     ```
   * Supported formats can be extended as needed (CSV, TXT, Excel, etc.).

2. **Define partner-specific mapping**

   * Add a new reader section in `eligibility_pipeline.py` to handle the file format.
   * Map partner column names to the standard schema.

3. **Register the partner in the pipeline**

   * Include the new partner in the pipeline runner so it is processed automatically.
   * Assign a unique `partner_code` for identification.

4. **Validate**

   * Run the pipeline and verify the output matches the standard schema.

This modular design allows new partners to be added with minimal changes to the existing code.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Bonus: Data Validation and Error Handling

### Validation

The pipeline validates the presence of `external_id`, which is required to uniquely identify a member across partner systems.

* Rows where `external_id` is missing or null are dropped during processing.
* This ensures the final dataset only contains valid, traceable member records.

This validation step prevents downstream issues such as duplicate records or unidentifiable members.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Error Handling

The pipeline is designed to handle malformed data gracefully without failing the entire job.

* Date fields (such as `dob`) are parsed using tolerant parsing logic (`try_to_date`), allowing invalid or malformed dates to be converted to `NULL` instead of causing runtime failures.
* Non-critical data issues (e.g., unexpected formats or missing optional fields) do not stop the pipeline execution.
* This approach allows ingestion to continue while preserving visibility into data quality issues.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Design Rationale

These validation and error-handling choices ensure the pipeline is:

* **Resilient** to partner data quality issues
* **Safe for production-scale ingestion**
* **Extensible** as additional partners are onboarded

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------








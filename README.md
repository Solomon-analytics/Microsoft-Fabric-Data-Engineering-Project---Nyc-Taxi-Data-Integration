**Microsoft Fabric Data Engineering Project - NYC Taxi Data Integration**
---
**Project Overview:**

This project demonstrates a complete end-to-end data engineering solution using Microsoft Fabri Lakehouse.

The goal is to integrate the Yellow and Green NYC Taxi datasets into a unified analytical data model following the medallion architecture (Raw --> Landing --> Bronze --> Silver --> Gold).

The pipeline supports incremental ingestion, transformation, enrichment, and dimensional modelling, creating a fact table that combines both taxi datasets with detailed location and time-based insights.

---
**Architecture Summary**

The project follows the Medallion Architecture pattern, ensuring clean data progression, high data quality, and scalability.

| Layer                | format used  |          Description                                                    | Key Actions                                                       |
| -------------------- | ------------ | ------------------------------------------------------------------------|-------------------------------------------------------------------|                              
| Raw                  | Parquet      | Original source data exactly as received                                | Data upload from source site                                      |
| Landing              | Parquet      | Staging area for incremental loads using date parameter                 | Add processing_date and store partitioned data                    |
| Bronze               | Delta        | Dataset stored exactly as recieved including parameter (processing_date)| Schema enforcement and consistent formatting                      |
| Silver               | Delta        | Business transformed and enriched data                                  | Data quality checks, lookups, and derived columns                 |
| Gold                 | Delta        | Analytical layer with fact and dimenstion table                         | Unified fact table with dimensional model for analytics reporting |

                                                     

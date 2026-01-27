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

---

**Data Flow Summary**

---

**Step 1: Raw --> Landing**

Source: NYC Yellow and Green Taxi datasets stored in:

      - /Files/Raw/nyctaxiyellow
      - /Files/Raw/nyctaxigreen
      - Added a new column processing_date to track data load batches
      - Data read using PySpark and written to Landing layer with partitioning by processing_date:
      - /Files/Landing/nyctaxiyellow
      - /Files/Landing/nyctaxigreen
      

**Step 2: Landing --> Bronze**

      - Landing data loaded using today_date parameter
      - stored data into Bronze Delta tables
      - nyctaxiyellow_bronze
      - nyctaxigreen_bronze


**Step 3: Bronze --> Silver**

      - Applied basic schema checks, transformation and standardisation
      - ntroduced data validation and business rules
      - Dropped null rows in key columns
      - Derived trip duration, pickup time bucket, day of the week, etc
      - Created business and natural keys using SHA2 hashing
      - Enhanced with mappable dimensions
      - Stored results in Silver Delta tables:
      - yellowtaxi_silver
      - greentaxi_silver


  **Step 4: Silver --> Gold**

      - Combined yellow and green silver tables into a unified Fact Trip Table
      - Create dimension table dim_location from taxi zone lookup
      - Created a surrogate key in dim_location using Window functions
      - Joined dim_location to the combination of (yellow and green silver) to replace each of the pickup_location_id and dropoff_location_id in (yellow and silver combination) with newly created surrogate key in dim_location
      - Created a Fact Table (fact_nyc_taxi) with full upsert (merge) logic to support incremental updates

---

**Key Functional Logic**

---

**Data Quality**

      - Invalid or missing trips flagged before load
      - Null-critical fields (Vendor, pickup_datetime, dropoff_datetime) removed
      - Logical check: Pickup_datetime must not be later than dropoff_datetime


**Data Enrichement**

      - Derived additional columns for deeper analytics
      - trip duration minutes
      - pickup day of the week
      - pickup time bucket (morning peak, evening peak, off peak)
      - pickup week day type (weekday/weekend)


**Surrogate and Natural Keys**

      - trip_business_key: composed of location ids, vendor, pickup_datetime and dropoff_datetime
      - trip_natural_key - SHA2 hash of business key for uniqueness across datasets


---

**Fact and Dimension Design**

---

**Dimension Table - dim_location**

| Column               | Description                                      |
| -------------------- | -------------------------------------------------|                            
| location_sk          | Surrogate key: derived using (window function)   | 
| location_id          | Original location id                             | 
| borough              | NYC borough                                      |                                    
| zone                 | NYC Taxi zone                                    | 
| service_zone         | NYC service area of the trip                     | 


**Fact Table - fact_nyc_taxi**

| Column                                         | Description                                                                                    |
| -------------------------------------          |------------------------------------------------------------------------------------------------|                            
| vendor_id, vendor_name                         | Taxi vendor                                                                                    | 
| pickup_datetime, dropoff_datetime              | Trip timestamps                                                                                | 
| trip_distance                                  | Distance travelled                                                                             |                                    
| fare_amount, total_amount                      | Cost metrics                                                                                   | 
| pickup_location_sk, dropoff_location_sk        | Replace inital pickup_location and dropoff_location_id with the location_sk in dim_location    | 
| trip_duration_minutes                          | compute travel duration                                                                        |
| pickup_time_bucket                             | Categorised travel time range                                                                  |
| trip_natural_key                               | Unique hash key for row identification                                                         |


---

**Pipeline Orchestration**

Dataset for year 2025 ingested using **Mircosoft Fabric Pipeline**

      - Each notebook (Raw --> Landing --> Bronze --> Silver --> Gold) is parameterised with processed_date and today_date
      - The pipeline automates incremental ingestion, transformation, and loading.
      - Applied to full 2025 dataset (42.5 million rows) with manual triggers applied.

---

**Performance and Monitoring**

Implemented **Delta Table History** to monitor update, insert, and delete metrics.

      - numTargetRowsinserted
      - numTargetRowsupdated
      - numTargetRowsdeleted
      - Display total impacted rows after each merge
      - successfully loaded 42.5 million records with efficient merge operations.

      

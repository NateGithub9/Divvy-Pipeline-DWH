# Data Warehouse for Cross-Analysis Divvy - Chicago: Urban Mobility and Meteorological Impact

## 1. "Why" and "What"?

Divvy is the Chicago region's public bicycle sharing system, serving as a critical component of the city's urban 
mobility infrastructure. As a public service, Divvy generates massive amounts of granular trip data daily, representing
complex patterns of commuter behavior, seasonal usage, and station demand.

This abundance of raw data is invaluable. This data engineering project was initiated to transform Divvy's raw operational 
logs into a trusted, clean, and analytical dataset.

By creating a reliable, high-performance OLAP (Online Analytical Processing) system, this project offers transportation 
planners and city officials a reliable asset to model demand, optimize resource allocation (e.g., bike rebalancing), and improve Chicago's transportation network efficiency.
Specifically, the resulting governed dataset enables the following critical insights:

* **Demand Forecasting & Staffing:** Analyzing trip volume by day and season to establish baseline demand.
* **Station Optimization:** Determining the most popular stations to identify infrastructure bottlenecks and expansion sites.
* **Resource Allocation & Inventory:** Identifying seasonal and hourly usage patterns to proactively manage bike inventory.
* **Operational Resilience:** Establishing the correlation between weather conditions and ridership to build resilient operational models.

### The datasets 

Divvy Bike Trip Dataset:
https://divvy-tripdata.s3.amazonaws.com/index.html

![](https://drive.usercontent.google.com/download?id=1_ZRTKA5qzwS2J8x00IE-6dvHiElz91NS)

Chicago Weather Database:
https://www.kaggle.com/datasets/curiel/chicago-weather-database?resource=download

![](https://drive.usercontent.google.com/download?id=1HCdgrv2u41cHW91jeEPo2VB508jlNGvO)

### Conceptual Design (ERD Design)

The conceptual data model was initially drafted using **Looping** (https://www.looping-mcd.fr/) software. 
This crucial first step established the core entities (e.g., Trip, Rider, Station, Date, Weather) 
and their relationships without concern for the final database platform (Da). This approach ensured alignment with
the stakeholders and analysts on what the data means before moving to implementation.

### Dimensional Modeling Approach

The logical and physical models, built using a Dimensional Modeling paradigm with a Star Schema, optimize analytical and
reporting performance. This model, which originated from the 2022-2023 Divvy Tripdata and Chicago weather dataset, 
comprises a central Fact Table and several surrounding Dimension Tables:

* Fact Table (Fact_Trip): Stores granular, atomic trip records, linking all dimensions and capturing key performance
measures such as Trip Duration and Distance.
* Dimension Tables (DIM_Date_day, DIM_Date_hour, DIM_Station, DIM_Rider, DIM_Weather): Contain descriptive attributes 
essential for filtering, grouping, and drilling down into the data. This structure ensures data consistency and reduces 
redundancy.

![](https://drive.usercontent.google.com/download?id=1L2BQhCGOgCMRx4oGlWqgJBolwrgTIo2H)

Plan in Databricks:

![](https://drive.usercontent.google.com/download?id=1B_N_iRBa7xlF_bI4KOgnPA3miwrU7mH5)

## 2. Azure Services Provisioning

The foundation of the project required provisioning the core cloud resources within Microsoft Azure and 
securing the raw datasets in the landing zone.

The following essential resources were provisioned in Azure to host the Data Lakehouse architecture:

* **Resource Group:** A dedicated resource group was created to logically organize and manage all related services, simplifying governance and cost tracking.
* **Azure Blob Storage Account with dedicated container**: Provisioned to serve as the Source Data landing zone. This storage account provides a durable and scalable, hierarchically-named 
location for the raw, compressed Divvy trip data and the Chicago weather data.

Blob storage:
![](https://drive.usercontent.google.com/download?id=1bRKcvChYHRpV85T6p3gCB9uJpEkQyNhU)

Container:
![](https://drive.usercontent.google.com/download?id=1Rrob0_nBgg7Nmy7bjxZdo7Np239Drk_E)

* **Azure Databricks Workspace:** The core engine for the entire data warehouse architecture.

![](https://drive.usercontent.google.com/download?id=1yinHR8nzULdh88pmGLs2kuAMfv-wHK5-)

## 3. Data warehouse 
### 1. Bronze Ingestion: Raw Data Acquisition 

The Bronze layer ingests the core data assets from Azure Blob Storage.

* **Decompression Stage:** Compressed ZIP archives of divvy-tripdata were programmatically decompressed via 
a Python notebook (see "Decompression.ipynb"), and the extracted data was transferred to the Divvy container (blob storage) for further  processing.

![](https://drive.usercontent.google.com/download?id=1IiKgzLbItCV1Wh4K9dtm5R9tfntdisNn)

* **Divvy trip data:** The raw divvy-tripdata was ingested and named "bronze_trip_data". This data was 
sourced as a CSV, requiring the use of DLT's Auto Loader feature to efficiently handle the file format, schema inference, and incremental loading.
*For more details, see "01 - Ingestion.ipynb"* 

![](https://drive.usercontent.google.com/download?id=13Aa-5n5TebyyPkmIce9GSCfkfLHjQII7)

* **Chicago Weather Data:** The complementary dataset containing daily and hourly weather metrics was also ingested to 
support later correlation analysis. This process, named "bronze_weather_data," 
*For more details, see "01 - Ingestion.ipynb"*

![](https://drive.usercontent.google.com/download?id=1VCFNg714BmmWvkmgEUhYUyHtiAAYhOrK)

### 2. Table Creation
#### Dimensions tables creation
* Created two time type dimensions: "DIM_day_date" & "DIM_day_hour"

![](https://drive.usercontent.google.com/download?id=1NLiQNhFoNnKLfc8AuPl5RewOmjZG3ojr)

![](https://drive.usercontent.google.com/download?id=1Fn_OJfTw4Shp-0QSgml9UCH1MoJyo3Po)
  
* Created DIM Station:
The dim_station table is maintained using a Slowly Changing Dimension (Type 1) strategy. A staging process first
aggregates all unique Start and End stations from the trip data. This dataset is then merged into the target dimension: 
existing stations have their names or coordinates updated (overwritten) if changes are detected, while entirely new stations are inserted.
* Created DIM Rider: 
The dim_rider table serves as a reference for equipment types. The process extracts distinct rideable_type values 
from the silver_trip_data to ensure all bike variations are accounted for in the analytical layer.
* Created DIM Weather: To facilitate categorical analysis, the dim_weather process transforms continuous weather metrics into discrete buckets. A temporary staging view applies business logic to classify Temperature, Humidity, Precipitation, and Wind Speed into both text-based categories (e.g., "Freezing", "Light Rain") and binary integer codes for efficient sorting and querying.

*For more details, see "02 - Creation.ipynb" & "04 - Insertion.ipynb"* 

#### Fact table creation
* Created FACT_trip table: The central fact_trip table acts as the core transactional repository. The loading process resolves Foreign Keys by joining trip records against the Rider, Station, and Weather dimensions. It further populates quantitative metrics (Duration, Distance) and temporal keys (Date and Hour) to enable detailed slicing by time and magnitude.

*For more details, see "02 - Creation.ipynb" & "04 - Insertion.ipynb"* 

![](https://drive.usercontent.google.com/download?id=1x_CshzJZLmRZZjASHnDSbsGJohk0c1YN)

### 3. Silver Transformation: Trip data
* Dropped records that contained schema drift or corruption, diverting them to the _rescued_data column and prioritizing data quality.

* Cast the raw "started_at" and "ended_at" STRING columns to the TIMESTAMP data type, naming them "trip_start_ts" and "trip_end_ts".

* Applied COALESCE to replace NULL station IDs/names with standardized values ('N/A', 'UNKNOWN'), preparing the data for DIM table joins.

*For more details, see "03 - Transformation.ipynb"*
### 4. Silver Transformation: Weather data

* "Trip_Duration_Min" (DOUBLE) was calculated using TIMESTAMPDIFF, including a check to ensure end_ts was greater than start_ts.

* "Trip_Distance_Km" (DOUBLE) was calculated by implementing the Haversine Formula with native SQL math functions (ASIN, SQRT), assuming the Earth's radius of 6371 km.

* "Full_Date" (DATE) was extracted from the start timestamp to serve as the Foreign Key to the DIM_Date table.

* A final filter was implemented in the WHERE clause, excluding invalid records where Trip_Distance_Km was less than or equal to 0.05 or Trip_Duration_Min was NULL.

*For more details, see "03 - Transformation.ipynb"*

### 5. Fact Table data insertion

The fact_trip table is populated through a segmented process that first resolves dimensional relationships by joining 
trip records against the Rider, Station, and Weather tables to generate the necessary surrogate keys. 

Subsequent insertion 
steps independently ingest quantitative metrics, specifically trip duration and distance, directly from the silver
source to support performance analysis. 

To enable granular time-series reporting, the final phase derives Start 
and End date keys by linking parsed timestamps to the Date and Hour dimension tables. 

This logic currently executes as four separate insertions, which creates fragmented rows for dimensions, measures,
and dates rather than a single unified trip record.


*For more details, see "04 - Insertion.ipynb"*

## 4. Notebook automation and jobs

To move beyond manual execution and ensure data freshness, the ETL process is orchestrated using Azure Databricks Workflows. This transforms the individual notebooks into a cohesive, automated pipeline.

A multi-task Job was configured with linear dependencies to ensure data integrity. If an upstream task fails, the downstream tasks are halted to prevent data corruption.

1. **Task 1: Decompression:** Triggers the logic to unzip raw files from the landing zone.
2. **Task 2: Ingestion (Bronze):** Runs 01 - Ingestion.ipynb to load raw CSVs into the Data Lakehouse.
3. **Task 3: Schema Creation:** Runs 02 - Creation.ipynb to validate table structures if they don't exist.
4. **Task 4: Silver Transformation:** Runs 03 - Transformation.ipynb to clean, type-cast, and filter the data.
5. **Task 5: Gold Loading:** Runs 04 - Insertion.ipynb to populate the Fact and Dimension tables.

##### Scheduling

* *Trigger:* The job was set to run on a scheduled trigger (e.g., daily or weekly) to mimic a batch processing environment suitable for reporting requirements.
* *Notifications:* Email alerts are configured to notify the Data Engineering team in case of job failure or timeout.

## 5. Final Deliverable 

The fully governed, high-performance scalable Data Warehouse hosted on Databricks is ready for connection with BI tools like Power BI.

**Business Value Delivered: Analysts can now run simple SQL queries to get complex insights.**




## 6. Future Improvements
* **Migration to Delta Live Tables (DLT):** Transitioning the standard notebook workflow to DLT will enable declarative pipeline management and automated dependency resolution. This provides significant operational advantages, including native cluster auto-scaling for optimized cost and performance, alongside mandatory data quality enforcement using Expectations.
* **H3 Geospatial Indexing:** Implementing Uber’s H3 Hexagonal Hierarchical Spatial Indexing. This would significantly enhance geospatial analysis by allowing for efficient bucketing of rides into hexagonal grid cells. This enables high-performance heatmapping and proximity clustering.
## 7. Conclusion

This project provided the opportunity to implement a robust and scalable cloud-native data engineering solution from scratch. It allowed me to practically apply the theoretical knowledge acquired during my training and validate skills aligned with my Azure and Databricks certifications.

Thank you for reading and if you have any questions please reach out: nathaniel.akingbade@gmail.com 

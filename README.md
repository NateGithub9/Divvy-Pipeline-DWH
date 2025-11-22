# Data Warehouse for Cross-Analysis Divvy - Chicago: Urban Mobility and Meteorological Impact

## 1. "Why" and "What"?

Divvy is the Chicago region's public bicycle sharing system, serving as a critical component of the city's urban 
mobility infrastructure. As a public service, Divvy generates massive amounts of granular trip data daily, representing
complex patterns of commuter behavior, seasonal usage, and station demand.
This abundance of raw data is invaluable. This data engineering pipeline was initiated to transform Divvy's raw operational 
logs into a trusted, clean, and analytical dataset.
By creating a reliable, high-performance OLAP (Online Analytical Processing) system, this project offers transportation 
planners and city officials a reliable asset to model demand, optimize resource allocation (e.g., bike rebalancing), and improve Chicago's transportation network efficiency.
Specifically, the resulting governed dataset enables the following critical insights:

* **Demand Forecasting & Staffing:** Analyzing trip volume by day and season to establish baseline demand.
* **Station Optimization:** Determining the most popular stations to identify infrastructure bottlenecks and expansion sites.
* **Resource Allocation & Inventory:** Identifying seasonal and hourly usage patterns to proactively manage bike inventory.
* **Operational Resilience:** Establishing the correlation between weather conditions and ridership to build resilient operational models.

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

![](https://drive.usercontent.google.com/download?id=1r10ehTDroab7K5oUSeA7iuPY3vpO1Sp8)

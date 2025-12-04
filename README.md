# NoSQLClimateProject

Climate Monitoring Data Hub (MongoDB)

This repository contains the Data Definition Language (DDL) and analytical aggregation pipelines (queries) for a MongoDB database designed to store and analyze real-time climate monitoring data.

**I. Project Overview**

This project establishes a flexible NoSQL structure for managing three core types of climate data:

Staff: Records of personnel involved in the project
Monitoring: Metadata about the deployed devices (buoys, balloons, surface monitors)
Readings: High-volume time-series data captured by the devices (CO2, temperature, pressure, etc.)
Monitor Maintenance: Records of maintenance events and costs for the equipment



**II. Data Model & Denormalization Strategy**

Unlike traditional SQL databases, this MongoDB schema utilizes denormalization to prioritize read speed for analytical queries.

<img width="1260" height="166" alt="image" src="https://github.com/user-attachments/assets/3b1f72c6-46af-4a68-b4e5-6b12764f77f3" />



**III. Data Setup (DDL)**

The script for the NOSQLClimateDDL handles the entire database setup process.

Script Steps:
1. Object IDs: Defines static ObjectId variables for all staff members.
2. Clean-up: Drops existing collections (staff, readings, monitoring, monitormaintenance) for a clean start.
3. Data Insertion: Populates all four core collections with AI-generated mock data.
4. Indexing: Creates essential indexes to optimize query performance.

**Indexes:**

idx_equipment_id_unique (on monitoring.equipment_id)
idx_equipment_timestamp (on readings)
idx_type_timestamp (on readings)
idx_country (on readings.country)
idx_equipment_date (on monitormaintenance)



**IV. Analytical Queries**

The AnalyticalQueries file contains three essential aggregation pipelines that extract high-value insights from climate data. These pipelines are simplified and optimized to retain a denormalized data structure. NoSQL queries are run in stages to manage distributed workloads across multiple servers, a process that is important when handling large datasets and ensuring resource reliability.

**Analytical Query 1:** CO2 Averages and Global Comparison
Purpose: Determine which countries or regions have CO2 levels that are higher or lower than the overall global average.

Stage I. $match is used to filter data to a specific time window
Stage II. $group is used to calculate the average CO2 per country using the denormalized country field
Stage III. $group is used to calculate the global average CO2
Stage IV. $unwind and $project are used to compare each country's average against the global average and assign a comparison status based on the number (e.g., ABOVE_GLOBAL_AVERAGE)


**Analytical Query 2:** Anomaly Detection/High-Level Alerting
Purpose: Flags any critical readings that exceed a predefined safety threshold (e.g., CO2 > 415 ppm) for immediate action

Stage I. $match is used to filter data to a recent time window
Stage II. $match is used to filter for any document where co2_ppm is greater than 415
Stage III. $project outputs any necessary alert fields if the number meets a certain threshold


**Analytical Query 3:** Data Volume Analysis
Purpose: Counts the total number of readings from each country to aid in infrastructure auditing and resource allocation

Stage I. $group is used to organize all documents by the country field and uses '$sum: 1' to calculate the total number of readings
Stage II. $sort arranges the results to show countries with the highest contributions first (descending after)



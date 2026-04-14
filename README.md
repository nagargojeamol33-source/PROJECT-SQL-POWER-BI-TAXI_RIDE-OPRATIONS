# PROJECT-SQL-POWER-BI-TAXI_RIDE-OPRATIONS
Comprehensive data analysis of NYC taxi trip records to optimize fleet operations and maximize sales revenue. Features SQL-based data cleaning, Analysis, and interactive dashboarding.
Comprehensive data analysis of NYC taxi trip records to optimize fleet operations and maximize sales revenue.
Features SQL-based data cleaning, Analysis, and interactive dashboarding.
 NYC Taxi Operations & Sales Analysis

 Project Structure
├── SQL_Scripts/All SQL queries (Cleaning & Analysis)
├── Dashboard/          
 .pbix file and Dashboard
├── Data/Documentation about the dataset source
└── README.md
   PROJECT OVERVIEW 
This project analyzes the NYC Taxi and Limousine Commission (TLC) dataset to optimize fleet operations and sales performance.
Using SQLfor heavy data lifting and Power BI for interactive storytelling, the project identifies key revenue drivers and operational 
bottlenecks in New York's transport system.

  KEY OBJECTIVE 
Sales Intelligence: Analyzing fare components (tips, tolls, surcharges) to increase total revenue.
Demand Hotspots: Using location data to identify high-demand zones for better vehicle deployment.
Operational Efficiency: Calculating average trip distances and durations to optimize driver shifts.
Payment Trends: Evaluating the shift between cash, online and credit card transactions.

  TECHNICAL
atabase: SQL (SQL Server / MySQL / PostgreSQL) - Used for Data Cleaning, Joins, and Aggregations.
Visualization: Power BI - Used for creating interactive DAX-powered dashboards.

   SQL Implementation
Data Cleaning: Handled null values and filtered out outliers (e.g., zero-distance trips or negative fares).
Feature Engineering: Created time-based columns (Morning, Afternoon, Evening, Night) using SQL Case statements.
KPI Extraction: Wrote complex queries to calculate 'Average Revenue per Mile' and 'Peak Hour Frequency'.

   POWER BI DASHBOARD 
High-level view of Total Revenue, Total Trips, and Avg Tip Amount.
Geospatial Map Interactive map showing pickup/drop-off density across NYC Boroughs.
Time-Series Analysis: Hourly and weekly trends to visualize peak demand periods.
Slicers  Dynamic filtering by Borough, Payment Type, and Vendor.

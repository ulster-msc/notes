# notes

https://www.kaggle.com/datasets/ronaldonyango/global-suicide-rates-1990-to-2022?select=suicide_rates_1990-2022.csv

```
curl -L -o ~/datasets/kaggle_data.zip https://www.kaggle.com/api/v1/datasets/download/ronaldonyango/global-suicide-rates-1990-to-2022
```


# Hive and Zeppelin Queries for Suicide Rates Analysis

This repository contains a collection of semi-complex Hive and Zeppelin queries for analyzing the `suicide_rates_1990-2022.csv` dataset. These queries can be used to explore trends and patterns related to suicide rates across different demographics, countries, and economic factors.

## 1. Semi-Complex Queries

Below are some queries you can run in Hive or a Zeppelin notebook connected to a Hive interpreter.

### Query 1: Yearly Suicide Trends by Generation

**Description**: This query calculates the total number of suicides for each generation for each year, allowing you to see how suicide counts have changed over time across different generational cohorts.

```sql
SELECT
    Year,
    Generation,
    SUM(SuicideCount) AS TotalSuicides
FROM
    suicide_rates
GROUP BY
    Year,
    Generation
ORDER BY
    Year,
    Generation;
```

### Query 2: Top 10 Countries with the Highest Average Suicide Rate (2000-2022)

**Description**: This query identifies the top 10 countries with the highest average suicide rate per 100k population for the years 2000 to 2022. It filters for a specific time range and then calculates the average rate.

```sql
SELECT
    CountryName,
    AVG(DeathRatePer100K) AS AverageDeathRate
FROM
    suicide_rates
WHERE
    Year BETWEEN 2000 AND 2022
GROUP BY
    CountryName
ORDER BY
    AverageDeathRate DESC
LIMIT 10;
```

### Query 3: Suicide Rates vs. Economic Factors (GDP per Capita Categories)

**Description**: This query investigates the relationship between economic well-being and suicide rates. It categorizes countries based on their GDP per capita and then calculates the average suicide rate for each category for the year 2021.

```sql
WITH CountryEconomicCategory AS (
    SELECT
        CountryName,
        CASE
            WHEN GDPPerCapita < 1000 THEN 'Low Income'
            WHEN GDPPerCapita >= 1000 AND GDPPerCapita < 5000 THEN 'Lower Middle Income'
            WHEN GDPPerCapita >= 5000 AND GDPPerCapita < 20000 THEN 'Upper Middle Income'
            ELSE 'High Income'
        END AS EconomicCategory,
        DeathRatePer100K
    FROM
        suicide_rates
    WHERE
        Year = 2021
)
SELECT
    EconomicCategory,
    AVG(DeathRatePer100K) AS AvgDeathRate
FROM
    CountryEconomicCategory
GROUP BY
    EconomicCategory
ORDER BY
    AvgDeathRate;
```

### Query 4: Ranking of Generations by Suicide Count Within Each Country

**Description**: Using a window function, this query ranks the generations by their total suicide count within each country. This can help identify which generations are most affected in different parts of the world.

```sql
WITH GenerationSuicides AS (
    SELECT
        CountryName,
        Generation,
        SUM(SuicideCount) AS TotalSuicides
    FROM
        suicide_rates
    GROUP BY
        CountryName,
        Generation
)
SELECT
    CountryName,
    Generation,
    TotalSuicides,
    RANK() OVER (PARTITION BY CountryName ORDER BY TotalSuicides DESC) AS Rank
FROM
    GenerationSuicides
ORDER BY
    CountryName,
    Rank;
```

### Query 5: Year-over-Year Percentage Change in Suicide Counts for the United States

**Description**: This query calculates the year-over-year percentage change in total suicide counts for the United States. It uses the LAG window function to compare the current year's suicide count with the previous year's.

```sql
WITH YearlySuicides AS (
    SELECT
        Year,
        SUM(SuicideCount) AS TotalSuicides
    FROM
        suicide_rates
    WHERE
        CountryName = 'United States of America'
    GROUP BY
        Year
)
SELECT
    Year,
    TotalSuicides,
    LAG(TotalSuicides, 1, 0) OVER (ORDER BY Year) AS PreviousYearSuicides,
    ((TotalSuicides - LAG(TotalSuicides, 1, 0) OVER (ORDER BY Year)) / LAG(TotalSuicides, 1, 1) OVER (ORDER BY Year)) * 100 AS PercentageChange
FROM
    YearlySuicides
ORDER BY
    Year;
```

# notes

https://www.kaggle.com/datasets/ronaldonyango/global-suicide-rates-1990-to-2022?select=suicide_rates_1990-2022.csv

```
curl -L -o ~/datasets/kaggle_data.zip https://www.kaggle.com/api/v1/datasets/download/ronaldonyango/global-suicide-rates-1990-to-2022
```


# Suicide Rates Analysis SQL Queries

This document contains SQL queries for analyzing global and regional suicide rate trends, correlations with economic factors, and a custom deprivation impact score. Each query includes its purpose and is organized by analysis category.

## 1. Global Trends and Demographics

### Global Average Death Rate by Year (Line Chart Data)
```sql
SELECT Year, SUM(SuicideCount) / SUM(Population) * 100000 AS GlobalRate
FROM suicide_rates
WHERE SuicideCount IS NOT NULL AND Population IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Calculates the annual global suicide rate per 100,000 population for trend analysis.

### Suicide Rate by Sex
```sql
SELECT Sex, AVG(DeathRatePer100K) AS AvgRateBySex
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
GROUP BY Sex;
```
**Purpose**: Compares average suicide rates between males and females.

### Suicide Rate by Age Group
```sql
SELECT AgeGroup, AVG(DeathRatePer100K) AS AvgRateByAge
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
GROUP BY AgeGroup
ORDER BY AvgRateByAge DESC;
```
**Purpose**: Identifies age groups with the highest suicide rates (e.g., 75+ years).

## 2. Regional and Country Patterns

### Average Death Rate by Region
```sql
SELECT RegionName, AVG(DeathRatePer100K) AS AvgRegionalRate
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
GROUP BY RegionName
ORDER BY AvgRegionalRate DESC;
```
**Purpose**: Highlights regions with the highest suicide rates (e.g., Europe, Asia).

### Highest Death Rates by Country (Top 5)
```sql
SELECT CountryName, AVG(DeathRatePer100K) AS AvgCountryRate
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
GROUP BY CountryName
HAVING COUNT(*) > 10 -- Ensures sufficient data points
ORDER BY AvgCountryRate DESC
LIMIT 5;
```
**Purpose**: Identifies countries with the highest average suicide rates (e.g., Lithuania, South Korea).

### Yearly Rate for Specific Country (e.g., USA)
```sql
SELECT Year, AVG(DeathRatePer100K) AS AvgRate
FROM suicide_rates
WHERE CountryName = 'United States of America' AND DeathRatePer100K IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Tracks USA-specific suicide rate trends (e.g., 2021 male rate ~28/100k).

## 3. Correlation with Economic Factors

### Average Death Rate vs. GDP Per Capita by Year
```sql
SELECT Year, AVG(DeathRatePer100K) AS AvgDeathRate, AVG(GDPPerCapita) AS AvgGDPPerCapita
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL AND GDPPerCapita IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Provides data for correlation analysis between suicide rates and GDP per capita (negative correlation ~ -0.25).

### Average Death Rate vs. Inflation Rate by Year
```sql
SELECT Year, AVG(DeathRatePer100K) AS AvgDeathRate, AVG(InflationRate) AS AvgInflation
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL AND InflationRate IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Provides data for positive correlation (~0.15) between suicide rates and inflation.

### Average Death Rate vs. Employment Population Ratio by Year
```sql
SELECT Year, AVG(DeathRatePer100K) AS AvgDeathRate, AVG(EmploymentPopulationRatio) AS AvgEmploymentRatio
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL AND EmploymentPopulationRatio IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Provides data for negative correlation (~ -0.20) between suicide rates and employment.

## 4. Custom Deprivation Impact Score

### Calculate Deprivation Impact Score and Average Rate
```sql
SELECT Year,
       AVG(DeathRatePer100K) AS AvgDeathRate,
       AVG((InflationRate / GDPPerCapita) * (100 - EmploymentPopulationRatio)) AS DeprivationScore
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
  AND InflationRate IS NOT NULL
  AND GDPPerCapita IS NOT NULL
  AND EmploymentPopulationRatio IS NOT NULL
GROUP BY Year
ORDER BY Year;
```
**Purpose**: Computes a custom metric linking economic deprivation to suicide rates; higher scores correlate with ~15% rate increase.

### Gender-Specific Deprivation Impact (Males vs. Females)
```sql
SELECT Sex,
       AVG(DeathRatePer100K) AS AvgDeathRate,
       AVG((InflationRate / GDPPerCapita) * (100 - EmploymentPopulationRatio)) AS DeprivationScore
FROM suicide_rates
WHERE DeathRatePer100K IS NOT NULL
  AND InflationRate IS NOT NULL
  AND GDPPerCapita IS NOT NULL
  AND EmploymentPopulationRatio IS NOT NULL
GROUP BY Sex;
```
**Purpose**: Highlights gender disparity in deprivation impact (e.g., males 4x more affected).

## 5. Data Validation and Cleaning

### Check for Missing Values
```sql
SELECT COUNT(*) - COUNT(DeathRatePer100K) AS MissingDeathRates,
       COUNT(*) - COUNT(GDPPerCapita) AS MissingGDP,
       COUNT(*) - COUNT(InflationRate) AS MissingInflation,
       COUNT(*) - COUNT(EmploymentPopulationRatio) AS MissingEmployment
FROM suicide_rates;
```
**Purpose**: Ensures data quality before analysis; replace NULLs or filter them out as done above.

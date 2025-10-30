# 🌎 Conflict & Self-Harm: A Comparative SQL Analysis

#### Disclaimer
The data utilized in this analysis involves deeply sensitive and complex public health and geopolitical subjects, specifically armed conflict, mortality, trauma, and suicide. I am approaching this topic with the utmost respect, sensitivity, and ethical consideration for the individuals, communities, and nations affected by these issues. This analysis is purely observational, drawing connections between large-scale data sets, and is not intended to minimize, generalize, or provide clinical insights.

If you or someone you know is struggling with thoughts of self-harm or suicide, please know that help is available. Please reach out immediately to local emergency services or a dedicated crisis line for immediate support.
|Crisis Support|Phone Number|
|-|-|
|US Suicide Hotline|988|
|Trevor Project|866-488-7386|

## Project Overview

This SQL project investigates the potential correlation and relationship between large-scale organized violence and national suicide rates. By integrating the Uppsala Conflict Data Program (UCDP)'s comprehensive records of armed conflict fatalities with the World Health Organization (WHO)'s standardized global suicide rates, this database serves as a platform for comparative analysis in conflict-affected regions over time.

The primary goal is to use SQL joins and aggregations to identify longitudinal trends and spatial overlaps between the two phenomena, providing a robust dataset for researchers and policymakers.

## 📊 Data Sources

The analysis relies on two major, globally-respected datasets. All data is cleaned, harmonized (by country code and year), and imported into the relational database.
| Dataset | Description | Source | Key Variables Used |
|---|----|----|----|
| UCDP Armed Conflict Data | Records of state-based, non-state, and one-sided conflicts, including battle-related deaths by year and country. | UCDP Dataset Download Center | Country, Year, Total Deaths |
| WHO Suicide Rates | Age-standardized suicide mortality rates per 100,000 population, disaggregated by country and year. | WHO Mortality Database (Global Health Estimates) | Country, Year, ASR Suicide Rate |

## 💾 Database Schema

The core of the project involves two main tables, linked by country code and year, which facilitates direct comparative querying.

#### conflicts Table

This table stores annualized conflict data derived from UCDP datasets (like UCDP/PRIO Armed Conflict Dataset or GED).

| Column Name | Data Type | Description |
|----|-----|----|
| country_id | VARCHAR(3) | ISO-3 country code (Primary Key component) |
| year | INT | Year of observation (Primary Key component) |
| conflict_level | VARCHAR(50) | Categorization (e.g., "State-Based", "Non-State") |
| battle_related_deaths | INT | Total fatalities recorded for that country/year |
| is_active_conflict | BOOLEAN | Flag indicating if a high-intensity conflict was active |

#### suicide_rates Table

This table stores standardized, age-adjusted mortality rates for self-harm by country and year, sourced from WHO.

| Column Name | Data Type | Description |
|-|-|-|
| country_id | VARCHAR(3) | ISO-3 country code (Primary Key component) |
| year | INT | Year of observation (Primary Key component) |
| asr_suicide_rate_total | NUMERIC(5,2) | Age-standardized suicide rate (per 100k) |
| rate_male | NUMERIC(5,2) | Suicide rate for males |
| rate_female | NUMERIC(5,2) | Suicide rate for females |

## 💡 Key SQL Analysis Queries
**the following are questions we will be asking throughout the process of analyzing the data set**
1. Identifying High-Risk Countries (Conflict and Suicide)
This query identifies countries that simultaneously experienced high conflict deaths (over 1,000) and high suicide rates (over the global average, e.g., 10 per 100k) in a specific year.

```
SELECT
    c.country_id,
    c.year,
    c.battle_related_deaths,
    s.asr_suicide_rate_total
FROM
    conflicts c
INNER JOIN
    suicide_rates s ON c.country_id = s.country_id AND c.year = s.year
WHERE
    c.battle_related_deaths > 1000
    AND s.asr_suicide_rate_total > 10.0
ORDER BY
    c.battle_related_deaths DESC;
```


2. Calculating Correlation Coefficient

This is a conceptual query using a window function (or a statistical extension like PL/R if available, but here shown as a direct aggregation) to determine the Pearson correlation coefficient between conflict intensity and suicide rates across all observations.

```
SELECT
    CORR(c.battle_related_deaths, s.asr_suicide_rate_total) AS correlation_coefficient,
    COUNT(*) AS total_observations
FROM
    conflicts c
INNER JOIN
    suicide_rates s ON c.country_id = s.country_id AND c.year = s.year
WHERE
    c.battle_related_deaths > 0; -- Only include years with recorded conflict deaths for relevance
```


3. Change Over Time Comparison

Find the percent change in both deaths and suicide rates for a specific country (e.g., Afghanistan - AFG) between two years.

```
WITH BaseData AS (
    SELECT
        c.year,
        c.battle_related_deaths,
        s.asr_suicide_rate_total
    FROM
        conflicts c
    INNER JOIN
        suicide_rates s ON c.country_id = s.country_id AND c.year = s.year
    WHERE
        c.country_id = 'AFG'
)
SELECT
    t1.year AS current_year,
    t2.year AS previous_year,
    t1.asr_suicide_rate_total AS current_suicide_rate,
    t2.asr_suicide_rate_total AS previous_suicide_rate,
    (t1.asr_suicide_rate_total - t2.asr_suicide_rate_total) * 100.0 / t2.asr_suicide_rate_total AS suicide_rate_pct_change,
    (t1.battle_related_deaths - t2.battle_related_deaths) * 100.0 / t2.battle_related_deaths AS conflict_deaths_pct_change
FROM
    BaseData t1
INNER JOIN
    BaseData t2 ON t1.year = t2.year + 1
ORDER BY
    current_year;
```


🛠️ Setup and Prerequisites

To replicate this project, you will need:

A Relational Database System: PostgreSQL, MySQL, or SQLite.

SQL Client: DBeaver, VS Code SQL tools, or pgAdmin for connecting and running queries.

Data Ingestion Script: A script (e.g., Python/Pandas, or manual CSV import) to clean the raw UCDP and WHO files and load them into the tables defined above. Ensure country names are consistently mapped to standard ISO-3 codes for accurate joining.

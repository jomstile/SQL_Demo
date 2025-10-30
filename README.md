# 🌍 Conflict & Despair: Correlating Armed Conflict Intensity with Global Suicide Rates

Project Overview

This project uses SQL to investigate the potential correlation between sustained periods of organized armed conflict and subsequent changes in national suicide mortality rates. The core hypothesis is that countries experiencing high-intensity conflict (as defined by battle-related deaths) will exhibit a statistically significant increase in suicide rates compared to periods of peace or low-intensity conflict within the same country, or when compared to stable, conflict-free nations.

This analysis leverages two major global datasets—the Uppsala Conflict Data Program (UCDP) and World Health Organization (WHO) Global Health Estimates—to create a unified, temporal dataset for cross-national comparison.

Data Sources

The project relies on two primary data sources, which require significant cleaning and standardization (especially country names and temporal aggregation) for successful joining.

1. UCDP/PRIO Armed Conflict Dataset (ACD)

The UCDP data provides annual information on organized armed conflicts globally. We will use the conflict-year unit of analysis, focusing on the intensity of violence.

Column Name

Description

Source

conflict_id

Unique ID for the armed conflict.

UCDP

country

The country affected by the conflict. (Requires standardization)

UCDP

year

Calendar year of the conflict observation.

UCDP

intensity

Classification of conflict intensity (e.g., 1 for minor, 2 for war).

UCDP

battle_deaths_total

The estimated total number of battle-related deaths in the conflict-year.

UCDP

2. WHO Suicide Mortality Data

This data provides global, age-standardized suicide mortality rates (per 100,000 population) collected through national health reporting systems.

Column Name

Description

Source

country

The reporting country. (Requires standardization)

WHO

year

Year of the mortality rate observation.

WHO

suicide_rate_per_100k

Age-standardized suicide mortality rate per 100,000 people.

WHO

sex

Disaggregation by sex (e.g., Male, Female, Both sexes).

WHO

SQL Database Schema (Conceptual)

The final database structure will use three main tables:

dim_countries: A lookup table to ensure consistent country naming across datasets.

ucdp_conflict_data: The imported and cleaned UCDP data.

who_suicide_data: The imported and cleaned WHO data.

The primary joining keys for analysis will be country_id (from the dimension table) and year.

Table: dim_countries

Data Type

Key

country_id

INT

PRIMARY KEY

ucdp_name

VARCHAR



who_name

VARCHAR



standard_name

VARCHAR



Table: ucdp_conflict_data

Data Type

Key

country_id

INT

FOREIGN KEY (to dim_countries)

year

INT



conflict_status

VARCHAR

(e.g., 'No Conflict', 'Minor Conflict', 'War')

battle_deaths

INT



Table: who_suicide_data

Data Type

Key

country_id

INT

FOREIGN KEY (to dim_countries)

year

INT



sex

VARCHAR



suicide_rate

DECIMAL(5,2)



Methodology & Key Analysis Steps

The project's primary deliverable is a set of SQL queries designed to generate analytical views.

1. Data Cleaning and Standardization

Country Mapping (Crucial Step): Create the dim_countries table to map variant country names (e.g., "Syrian Arab Republic" vs. "Syria") between the two source datasets to a single country_id.

Conflict Status Classification: Translate the UCDP data (often involving multiple conflict entries per country-year) into a single, comprehensive conflict status (e.g., MAX(intensity)) per country-year.

2. Primary Join and Correlation

The main query will perform a FULL OUTER JOIN on the cleaned UCDP and WHO tables based on country_id and year. This creates a comprehensive view of conflict status and suicide rate over time and space.

3. Analytical SQL Objectives

Average Rates by Conflict Status: Calculate the average suicide rate (overall and by sex) across all countries, grouped by conflict status (No Conflict, Minor Conflict, War).

-- Example Query Objective
SELECT
    t1.conflict_status,
    t2.sex,
    AVG(t2.suicide_rate) AS avg_suicide_rate,
    COUNT(DISTINCT t1.country_id) AS num_countries_in_group
FROM ucdp_conflict_data t1
JOIN who_suicide_data t2 ON t1.country_id = t2.country_id AND t1.year = t2.year
GROUP BY 1, 2;


Lagged Effect Analysis: Investigate the lagged correlation by joining conflict data with suicide data from subsequent years (e.g., joining 2005 conflict data with 2008 suicide data) to test for long-term mental health impacts.

Case Study Identification: Identify specific countries that show the highest increase in suicide rates in the five years following the termination or de-escalation of a major conflict.

Trend Visualization Data: Generate an output table showing country_name, year, conflict_intensity, and suicide_rate for easy visualization in tools like Tableau or Power BI.

Expected (Hypothesized) Findings

The analysis is anticipated to find a complex, often lagged, relationship.

Positive Correlation (War to Despair): A significant positive correlation is expected between the presence of War (Intensity 2) and elevated suicide rates in the same or immediately subsequent years, particularly for specific demographics (e.g., young males, returning veterans, or civilian populations experiencing trauma).

Lagged Impact: The mental health impact may be most pronounced 2-5 years after the conflict has subsided, reflecting the delayed onset of PTSD, depression, and social disintegration.

Regional Differences: The strength of the correlation is expected to vary significantly based on regional factors, cultural stigma, and the availability of mental health infrastructure.

Technology Stack

Database: SQL (Standard ANSI SQL, compatible with PostgreSQL or MySQL).

Tools: Standard SQL environment (e.g., DBeaver, VS Code with SQL extension).

Data Cleaning: Python (Pandas) may be required for initial data ingestion and country name standardization before loading into the database.

```
CREATE VIEW Annual_Conflict_Summary AS
SELECT
    country_id,
    country_name,
    year,
    SUM(best_fatality_estimate) AS total_annual_deaths
FROM
    UCDP_Conflict_Deaths
GROUP BY
    country_id,
    country_name,
    year;
```

```
CREATE VIEW Combined_Analysis_Data AS
SELECT
    ACS.country_name,
    ACS.year,
    ACS.total_annual_deaths,
    WSR.crude_suicide_rate
FROM
    Annual_Conflict_Summary AS ACS
INNER JOIN
    WHO_Suicide_Rates AS WSR
ON
    ACS.country_id = WSR.country_id
    AND ACS.year = WSR.year
WHERE
    WSR.sex = 'Both Sexes'  -- Focus on the aggregate rate for the primary analysis
    AND ACS.total_annual_deaths IS NOT NULL
    AND WSR.crude_suicide_rate IS NOT NULL;
```

```
SELECT
    CASE
        WHEN total_annual_deaths >= 100 THEN 'High Conflict (>= 100 Deaths)'
        ELSE 'Low/No Conflict (< 100 Deaths)'
    END AS Conflict_Intensity,
    COUNT(*) AS Total_Observations,
    AVG(crude_suicide_rate) AS Average_Suicide_Rate
FROM
    Combined_Analysis_Data
GROUP BY
    Conflict_Intensity;
```

```
SELECT
    country_name,
    year,
    total_annual_deaths,
    crude_suicide_rate
FROM
    Combined_Analysis_Data
WHERE
    total_annual_deaths > 0  -- Filter for country-years with active conflict
ORDER BY
    crude_suicide_rate DESC
LIMIT 5;
```

```
CREATE VIEW Conflict_Type_Summary AS
SELECT
    country_id,
    country_name,
    year,
    conflict_type,
    SUM(best_fatality_estimate) AS type_annual_deaths
FROM
    UCDP_Conflict_Deaths
GROUP BY
    country_id,
    country_name,
    year,
    conflict_type;
```

```
SELECT
    CTS.country_name,
    CTS.year,
    CTS.conflict_type,
    CTS.type_annual_deaths,
    WSR.crude_suicide_rate
FROM
    Conflict_Type_Summary AS CTS
INNER JOIN
    WHO_Suicide_Rates AS WSR
ON
    CTS.country_id = WSR.country_id
    AND CTS.year = WSR.year
WHERE
    WSR.sex = 'Both Sexes'  -- Maintain focus on the aggregate rate
    AND CTS.type_annual_deaths > 0 -- Only include active conflict-years
ORDER BY
    CTS.country_name, CTS.year, CTS.conflict_type;
```

```
SELECT
    conflict_type,
    COUNT(*) AS Total_Observations,
    AVG(crude_suicide_rate) AS Average_Suicide_Rate_for_Type,
    AVG(type_annual_deaths) AS Average_Deaths_for_Type
FROM
    (
    -- Subquery to ensure we only include observations where the suicide rate is not null
    SELECT
        CTS.conflict_type,
        WSR.crude_suicide_rate,
        CTS.type_annual_deaths
    FROM
        Conflict_Type_Summary AS CTS
    INNER JOIN
        WHO_Suicide_Rates AS WSR
    ON
        CTS.country_id = WSR.country_id
        AND CTS.year = WSR.year
    WHERE
        WSR.sex = 'Both Sexes'
        AND WSR.crude_suicide_rate IS NOT NULL
        AND CTS.type_annual_deaths > 0
    ) AS T
GROUP BY
    conflict_type
ORDER BY
    Average_Suicide_Rate_for_Type DESC;
```

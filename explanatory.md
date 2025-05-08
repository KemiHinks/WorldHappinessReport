# **QUESTIONS**

1) What was the average happiness score across all countries in 2020?  
2) Determine what percentage of countries were above the average score and what percentage were below in 2020.  
3) Compare the happiness averages and percentages in 2024 against that of 2020.  Which year had the highest average happiness score recorded and which had the lowest?  
4) Compare the happiness averages and percentages across the 5 year period.  Which year had the highest average happiness score recorded and which had the lowest?  
5) Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries according to happiness score in 2020 (The year with the lowest average happiness score).    
6) Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries in 2022 (The year with the highest average happiness score).   
7) Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries over the 5 year period.   
8) a) Identify which 15 countries have the highest incline in happiness according to its happiness score from 2020 to 2024. (Only include countries that are present in both data sets)  
b) Amongst these countries which variable experienced the greatest incline?    
9) a)Identify which 15 countries had the smallest incline/greatest decline in happiness according to its happiness score from 2020 to 2024.  
b) Amongst these countries which variable experienced the greatest decline?  


# **SOLUTIONS**  

1. What was the average happiness score across all countries in 2020?  

```sql
SELECT '2020' AS year, ROUND(AVG(happiness_score),3) FROM happiness_2020
```
| year | round |
|------|-------|
| 2020 | 5.473 |


2. Determine what percentage of countries had a score above average and below in 2020.  
```sql
WITH above
AS
(SELECT COUNT(*) AS count_above_average FROM happiness_2020
WHERE happiness_score > (SELECT ROUND(AVG(happiness_score), 3) from happiness_2020)),

 below AS
(SELECT COUNT(*) AS count_below_average FROM happiness_2020
WHERE happiness_score < (SELECT ROUND(AVG(happiness_score), 3) from happiness_2020))

SELECT '2020' AS year, ROUND(AVG(happiness_score), 3) AS average_happiness_score,
(SELECT * FROM above),
ROUND (((SELECT * FROM above) / (SELECT COUNT(*) FROM happiness_2020) ::float
* 100)::numeric, 2) AS percent_above,
(SELECT * FROM below),
ROUND(((SELECT * FROM below) / (SELECT COUNT(*) FROM happiness_2020) ::float
* 100)::numeric, 2) AS percent_below
FROM happiness_2020

```

| year | average_happiness_score | count_above_average | percent_above | count_below_average | percent_below |
|------|-------------------------|---------------------|---------------|---------------------|---------------|
| 2020 | 5.473                   | 80                  | 52.29         | 73                  | 47.71         |


3. Compare the average happiness score and percentages of 2024 against that of 2020.  Which year had the highest average happiness score recorded and which had the lowest?  
```sql
WITH yearly_data AS (
SELECT '2020' AS year, happiness_score FROM happiness_2020
    UNION ALL
    SELECT '2024', happiness_score FROM happiness_2024),
averages AS (
    SELECT year, ROUND(AVG(happiness_score), 3) AS avg_score
    FROM yearly_data
    GROUP BY year),
counts AS (
    SELECT y.year,
    COUNT(*) AS total_count,
    SUM(CASE WHEN y.happiness_score > a.avg_score THEN 1 ELSE 0 END) AS above_count,
    SUM(CASE WHEN y.happiness_score < a.avg_score THEN 1 ELSE 0 END) AS below_count
    FROM yearly_data y
    JOIN averages a ON y.year = a.year
    GROUP BY y.year)

SELECT 
    c.year,
    a.avg_score AS average_happiness_score,
    c.above_count,
    ROUND((c.above_count::numeric / c.total_count) * 100, 2) AS percent_above,
    c.below_count,
    ROUND((c.below_count::numeric / c.total_count) * 100, 2) AS percent_below
FROM counts c
JOIN averages a ON c.year = a.year
ORDER BY c.year

```
| year | average_happiness_score | above_count | percent_above | below_count | percent_below |
|------|-------------------------|-------------|---------------|-------------|---------------|
| 2020 | 5.473                   | 80          | 52.29         | 73          | 47.71         |
| 2024 | 5.531                   | 79          | 56.43         | 61          | 43.57         |


4. Compare the average happiness score and percentages across the 5 year period. Which year had the highest average happiness score recorded and which had the lowest?
   
```sql
WITH yearly_data AS (
    SELECT '2020' AS year, happiness_score FROM happiness_2020
    UNION ALL
    SELECT '2021', happiness_score FROM happiness_2021
    UNION ALL
    SELECT '2022', happiness_score FROM happiness_2022
    UNION ALL
    SELECT '2023', happiness_score FROM happiness_2023
    UNION ALL
    SELECT '2024', happiness_score FROM happiness_2024
),
averages AS (
    SELECT year, ROUND(AVG(happiness_score), 3) AS avg_score
    FROM yearly_data
    GROUP BY year
),
counts AS (
    SELECT 
        y.year,
        COUNT(*) AS total_count,
        SUM(CASE WHEN y.happiness_score > a.avg_score THEN 1 ELSE 0 END) AS above_count,
        SUM(CASE WHEN y.happiness_score < a.avg_score THEN 1 ELSE 0 END) AS below_count
    FROM yearly_data y
    JOIN averages a ON y.year = a.year
    GROUP BY y.year
)
SELECT 
    c.year,
    a.avg_score AS average_happiness_score,
    c.above_count,
    ROUND((c.above_count::numeric / c.total_count) * 100, 2) AS percent_above,
    c.below_count,
    ROUND((c.below_count::numeric / c.total_count) * 100, 2) AS percent_below
FROM counts c
JOIN averages a ON c.year = a.year
ORDER BY c.year

```
| year | average_happiness_score | above_count | percent_above | below_count | percent_below |
|------|-------------------------|-------------|---------------|-------------|---------------|
| 2020 | 5.473                   | 80          | 52.29         | 73          | 47.71         |
| 2021 | 5.533                   | 75          | 50.34         | 74          | 49.66         |
| 2022 | 5.554                   | 74          | 50.68         | 72          | 49.32         |
| 2023 | 5.540                   | 74          | 54.01         | 63          | 45.99         |
| 2024 | 5.531                   | 79          | 56.43         | 61          | 43.57         |


5. Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries according to happiness score in 2020 (The year with the lowest average happiness score).

    
```sql
SELECT
'2020' as year, 'happiest' as group,
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2020
ORDER BY happiness_score DESC
LIMIT 30)
UNION 
SELECT
'2020', 'unhappiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2020
ORDER BY happiness_score ASC
LIMIT 30)

```
| year | group      | gdp_correlation     | social_correlation | healthy_correlation | freedom_correlation | generosity_correlation | corruption_correlation |
|------|------------|---------------------|--------------------|---------------------|---------------------|------------------------|------------------------|
| 2020 | happiest   | 0.44720570343417376 | 0.6910602007592295 | 0.5237755331684959  | 0.621276126510321   | 0.5510348911807854     | 0.7650950711024582     |
| 2020 | unhappiest | 0.17642222546707262 | 0.4829469098034706 | 0.2854590280979354  | 0.13878906781129777 | 0.06959128455041882    | -0.151738340374521     |


6. Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries in 2022 (The year with the highest average happiness score). 
```sql
SELECT
'2022' as year, 'happiest' as group,
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2022
ORDER BY happiness_score DESC
LIMIT 30)
UNION 
SELECT
'2022', 'unhappiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2022
ORDER BY happiness_score ASC
LIMIT 30)


```
| year | group      | gdp_correlation    | social_correlation | healthy_correlation | freedom_correlation | generosity_correlation | corruption_correlation |
|------|------------|--------------------|--------------------|---------------------|---------------------|------------------------|------------------------|
| 2022 | happiest   | 0.3910018076520407 | 0.6274227549784788 | 0.4715178205040063  | 0.45356996494673624 | 0.48103000978194616    | 0.5780130850518874     |
| 2022 | unhappiest | 0.1372054593519768 | 0.5253286793070645 | 0.20637566988698597 | 0.2041766905915833  | 0.17787028194835933    | -0.07575221513764577   |


7. Determine the coefficient of correlation between the happiness score and each key variable using the 30 happiest and least happiest countries over the 5 year period.
```sql
SELECT
'2020' as year, 'happiest' as group,
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2020
ORDER BY happiness_score DESC
LIMIT 30)
UNION 
SELECT
'2021', 'happiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2021
ORDER BY happiness_score DESC
LIMIT 30)
UNION 
SELECT
'2022', 'happiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2022
ORDER BY happiness_score DESC
LIMIT 30)
UNION 
SELECT
'2023', 'happiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2023
ORDER BY happiness_score DESC
LIMIT 30)
UNION
SELECT
'2024', 'happiest',
CORR(happiness_score, GDP_per_cap) AS gdp_correlation,
CORR(happiness_score, social_support) AS social_correlation,
CORR(happiness_score, healthy_life_expectancy) AS healthy_correlation,
CORR(happiness_score, freedom_of_choice) AS freedom_correlation,
CORR (happiness_score, generosity) AS generosity_correlation,
CORR(happiness_score, perceptions_of_corruption) AS corruption_correlation
FROM
(SELECT * FROM happiness_2024
ORDER BY happiness_score DESC
LIMIT 30)

```
| year | group    | gdp_correlation     | social_correlation | healthy_correlation | freedom_correlation | generosity_correlation | corruption_correlation |
|------|----------|---------------------|--------------------|---------------------|---------------------|------------------------|------------------------|
| 2020 | happiest | 0.44720570343417376 | 0.6910602007592295 | 0.5237755331684959  | 0.621276126510321   | 0.5510348911807854     | 0.7650950711024582     |
| 2021 | happiest | 0.39855535539402326 | 0.6229441566511705 | 0.4757212499482636  | 0.4904061339114231  | 0.3645717880531196     | 0.7617468228212575     |
| 2022 | happiest | 0.3910018076520407  | 0.6274227549784788 | 0.4715178205040063  | 0.45356996494673624 | 0.48103000978194616    | 0.5780130850518874     |
| 2023 | happiest | 0.3274797355771989  | 0.5833069170992449 | 0.5421764858110251  | 0.45613107326094526 | 0.4924020439976215     | 0.5635662746665031     |
| 2024 | happiest | 0.23865324090537937 | 0.6917145000523751 | 0.4626006349912459  | 0.405365482970521   | 0.2585887305406691     | 0.4464292802956362     |


8a) Comparing the data from 2020 and 2024, identify which 15 countries have the greatest incline in happiness according to its happiness score. (Only including countries that are present in both data sets)
```sql
SELECT 
c.country,
h20.happiness_rank as happiness_rank_20, 
h20.happiness_score as happiness_score20, 
h24.happiness_rank as happiness_rank_24,
h24.happiness_score as hapiness_score24, 
ROUND(((h24.happiness_score - h20.happiness_score) / h20.happiness_score) * 100, 2) as happiness_increase_perc
FROM (SELECT HP.country FROM (SELECT * FROM happiness_2024
UNION ALL
SELECT * FROM happiness_2020) AS HP
GROUP BY HP.country
HAVING COUNT (*) = 2
ORDER BY HP.COUNTRY) as c
LEFT JOIN happiness_2020 h20
ON c.country = h20.country
LEFT JOIN happiness_2024 h24
ON c.country = h24.country
ORDER BY happiness_increase_perc DESC
LIMIT 15

```
| country      | happiness_rank_20 | happiness_score20 | happiness_rank_24 | hapiness_score24 | happiness_increase_perc |
|--------------|-------------------|-------------------|-------------------|------------------|-------------------------|
| China        | 94                | 5.120             | 60                | 5.973            | 16.66                   |
| Armenia      | 116               | 4.680             | 82                | 5.455            | 16.56                   |
| Kuwait       | 48                | 6.100             | 13                | 6.951            | 13.95                   |
| India        | 144               | 3.570             | 126               | 4.054            | 13.56                   |
| Vietnam      | 83                | 5.350             | 54                | 6.043            | 12.95                   |
| Mozambique   | 120               | 4.620             | 90                | 5.216            | 12.90                   |
| South Africa | 109               | 4.810             | 83                | 5.422            | 12.72                   |
| Malaysia     | 82                | 5.380             | 59                | 5.975            | 11.06                   |
| Georgia      | 117               | 4.670             | 91                | 5.185            | 11.03                   |
| Venezuela    | 99                | 5.050             | 79                | 5.607            | 11.03                   |
| Serbia       | 64                | 5.780             | 37                | 6.411            | 10.92                   |
| Lithuania    | 41                | 6.220             | 19                | 6.818            | 9.61                    |
| Albania      | 105               | 4.880             | 87                | 5.304            | 8.69                    |
| Tanzania     | 148               | 3.480             | 131               | 3.781            | 8.65                    |
| Iraq         | 110               | 4.780             | 92                | 5.166            | 8.08                    |


b) Amongst these countries which Indicator experienced the greatest incline? 
```sql
 SELECT c.country,
  ROUND(((h24.happiness_score - h20.happiness_score) / 
  NULLIF(h20.happiness_score, 0)) * 100, 2) AS happiness_increase_perc,
  ROUND(((h24.gdp_per_cap - h20.gdp_per_cap) / 
  NULLIF(h20.gdp_per_cap, 0)) * 100, 2) AS gdp_perc,
  ROUND(((h24.social_support - h20.social_support) / 
  NULLIF(h20.social_support, 0)) * 100, 2) AS support_perc,
  ROUND(((h24.healthy_life_expectancy - h20.healthy_life_expectancy) / 
  NULLIF(h20.healthy_life_expectancy, 0)) * 100, 2) AS healthy_life_perc,
  ROUND(((h24.freedom_of_choice - h20.freedom_of_choice) / 
  NULLIF(h20.freedom_of_choice, 0)) * 100, 2) AS freedom_perc,
  ROUND(((h24.generosity - h20.generosity) / NULLIF(h20.generosity, 0)) * 100, 2) AS generosity_perc,
  ROUND(((h24.perceptions_of_corruption - h20.perceptions_of_corruption) / 
  NULLIF(h20.perceptions_of_corruption, 0)) * 100, 2) AS corruption_perc
FROM (
  SELECT HP.country
  FROM (
    SELECT * FROM happiness_2024
    UNION ALL
    SELECT * FROM happiness_2020
  ) AS HP
  GROUP BY HP.country
  HAVING COUNT(*) = 2
) AS c
LEFT JOIN happiness_2020 h20 ON c.country = h20.country
LEFT JOIN happiness_2024 h24 ON c.country = h24.country
ORDER BY happiness_increase_perc DESC
LIMIT 15;

```
| country      | happiness_increase_perc | gdp_perc | support_perc | healthy_life_perc | freedom_perc | generosity_perc | corruption_perc |
|--------------|-------------------------|----------|--------------|-------------------|--------------|-----------------|-----------------|
| China        | 16.66                   | 51.21    | 9.65         | -27.70            | 17.33        | 65.00           | 36.67           |
| Armenia      | 16.56                   | 78.27    | 12.04        | -22.69            | 71.05        | -53.64          | 73.00           |
| Kuwait       | 13.95                   | 29.93    | 10.00        | -15.26            | 45.09        | 53.85           | 56.36           |
| India        | 13.56                   | 59.73    | 2.03         | -22.78            | 32.24        | -27.50          | 10.91           |
| Vietnam      | 12.95                   | 84.86    | 1.36         | -34.27            | 29.69        | -32.86          | 77.78           |
| Mozambique   | 12.90                   | 211.11   | -8.02        | -51.25            | 30.00        | -28.18          | 22.50           |
| South Africa | 12.72                   | 54.33    | 8.65         | -21.46            | 24.88        | -40.00          | -43.33          |
| Malaysia     | 11.06                   | 40.68    | -2.31        | -31.65            | 38.17        | -16.30          | 98.33           |
| Georgia      | 11.03                   | 72.59    | 35.62        | -24.06            | 38.78        | -100.00         | 2.35            |
| Venezuela    | 11.03                   | -100.00  | -2.15        | -36.23            | 91.85        | 113.33          | 43.33           |
| Serbia       | 10.92                   | 55.35    | 4.59         | -29.52            | 65.75        | 33.33           | 68.33           |
| Lithuania    | 9.61                    | 48.40    | 1.68         | -25.25            | 26.90        | -12.00          | 45.00           |
| Albania      | 8.69                    | 58.02    | 11.33        | -24.94            | 50.00        | -18.82          | 63.33           |
| Tanzania     | 8.65                    | 78.26    | -18.85       | -13.64            | 39.02        | -29.26          | 28.50           |
| Iraq         | 8.08                    | 27.45    | -1.39        | -6.04             | 51.79        | -6.00           | -31.43          |



9a) Comparing the data from 2020 and 2024, identify which 15 countries had the greatest decline in happiness according to its happiness score. (Only include countries that are present in both data sets)

```sql
SELECT 
c.country,
h20.happiness_rank as happiness_rank_20, 
h20.happiness_score as happiness_score20, 
h24.happiness_rank as happiness_rank_24,
h24.happiness_score as hapiness_score24, 
ROUND(((h24.happiness_score - h20.happiness_score) / h20.happiness_score) * 100, 2) as happiness_increase_perc
FROM (SELECT HP.country FROM (SELECT * FROM happiness_2024
UNION ALL
SELECT * FROM happiness_2020) AS HP
GROUP BY HP.country
HAVING COUNT (*) = 2
ORDER BY HP.COUNTRY) as c
LEFT JOIN happiness_2020 h20
ON c.country = h20.country
LEFT JOIN happiness_2024 h24
ON c.country = h24.country
ORDER BY happiness_increase_perc ASC
LIMIT 15

```
| country          | happiness_rank_20 | happiness_score20 | happiness_rank_24 | hapiness_score24 | happiness_increase_perc |
|------------------|-------------------|-------------------|-------------------|------------------|-------------------------|
| Lebanon          | 111               | 4.770             | 142               | 2.707            | -43.25                  |
| Afghanistan      | 153               | 2.570             | 143               | 1.721            | -33.04                  |
| Congo (Kinshasa) | 131               | 4.310             | 139               | 3.295            | -23.55                  |
| Bangladesh       | 107               | 4.830             | 129               | 3.886            | -19.54                  |
| Pakistan         | 66                | 5.690             | 108               | 4.657            | -18.15                  |
| Sierra Leone     | 139               | 3.930             | 140               | 3.245            | -17.43                  |
| Comoros          | 134               | 4.290             | 132               | 3.566            | -16.88                  |
| Ghana            | 91                | 5.150             | 120               | 4.289            | -16.72                  |
| Benin            | 86                | 5.220             | 116               | 4.377            | -16.15                  |
| Lesotho          | 143               | 3.650             | 141               | 3.186            | -12.71                  |
| Mali             | 114               | 4.730             | 122               | 4.232            | -10.53                  |
| Cambodia         | 106               | 4.850             | 119               | 4.341            | -10.49                  |
| Sri Lanka        | 130               | 4.330             | 128               | 3.898            | -9.98                   |
| Jordan           | 119               | 4.630             | 125               | 4.186            | -9.59                   |
| Ethiopia         | 136               | 4.190             | 130               | 3.861            | -7.85                   |



b) Amongst these countries which variable experienced the greatest decline?

```sql
SELECT c.country,
  ROUND(((h24.happiness_score - h20.happiness_score) / 
  NULLIF(h20.happiness_score, 0)) * 100, 2) AS happiness_increase_perc,
  ROUND(((h24.gdp_per_cap - h20.gdp_per_cap) / 
  NULLIF(h20.gdp_per_cap, 0)) * 100, 2) AS gdp_perc,
  ROUND(((h24.social_support - h20.social_support) / 
  NULLIF(h20.social_support, 0)) * 100, 2) AS support_perc,
  ROUND(((h24.healthy_life_expectancy - h20.healthy_life_expectancy) / 
  NULLIF(h20.healthy_life_expectancy, 0)) * 100, 2) AS healthy_life_perc,
  ROUND(((h24.freedom_of_choice - h20.freedom_of_choice) / 
  NULLIF(h20.freedom_of_choice, 0)) * 100, 2) AS freedom_perc,
  ROUND(((h24.generosity - h20.generosity) / NULLIF(h20.generosity, 0)) * 100, 2) AS generosity_perc,
  ROUND(((h24.perceptions_of_corruption - h20.perceptions_of_corruption) / 
  NULLIF(h20.perceptions_of_corruption, 0)) * 100, 2) AS corruption_perc
FROM (
  SELECT HP.country
  FROM (
    SELECT * FROM happiness_2024
    UNION ALL
    SELECT * FROM happiness_2020
  ) AS HP
  GROUP BY HP.country
  HAVING COUNT(*) = 2
) AS c
LEFT JOIN happiness_2020 h20 ON c.country = h20.country
LEFT JOIN happiness_2024 h24 ON c.country = h24.country
ORDER BY happiness_increase_perc ASC
LIMIT 15;

```
| country          | happiness_increase_perc | gdp_perc | support_perc | healthy_life_perc | freedom_perc | generosity_perc | corruption_perc |
|------------------|-------------------------|----------|--------------|-------------------|--------------|-----------------|-----------------|
| Lebanon          | -43.25                  | 54.72    | -51.51       | -29.62            | -8.95        | -57.50          | 45.00           |
| Afghanistan      | -33.04                  | 109.33   | -100.00      | -10.37            | NULL         | -35.00          | NULL            |
| Congo (Kinshasa) | -23.55                  | 790.00   | -19.88       | -6.43             | 31.39        | -24.40          | -10.00          |
| Bangladesh       | -19.54                  | 100.36   | -71.38       | -25.65            | 29.17        | -22.22          | -7.22           |
| Pakistan         | -18.15                  | 72.42    | -31.03       | -31.70            | 32.20        | -37.39          | -38.33          |
| Sierra Leone     | -17.43                  | 172.50   | -24.53       | 26.50             | 23.42        | -30.38          | 6.00            |
| Comoros          | -16.88                  | 113.33   | -54.44       | -15.91            | -4.44        | -50.77          | 60.00           |
| Ghana            | -16.72                  | 85.69    | -22.99       | -16.28            | 29.79        | -29.62          | -53.33          |
| Benin            | -16.15                  | 147.03   | -63.43       | -13.94            | 38.29        | -44.00          | 93.85           |
| Lesotho          | -12.71                  | 71.33    | -21.93       | -100.00           | 27.56        | -18.00          | 70.00           |
| Mali             | -10.53                  | 113.43   | -29.07       | 16.09             | 54.21        | -29.41          | 50.00           |
| Cambodia         | -10.49                  | 87.22    | -4.77        | -25.08            | 28.81        | -26.09          | 1.43            |
| Sri Lanka        | -9.98                   | 51.22    | -0.92        | -25.82            | 10.00        | -42.40          | -38.00          |
| Jordan           | -9.59                   | 59.75    | -13.77       | -23.85            | 41.19        | -34.44          | 26.00           |
| Ethiopia         | -7.85                   | 147.50   | -8.50        | -12.50            | 7.56         | 17.39           | -15.83          |

# gamelofts_demo
a demo for a job I am applying at gamelofts

# 5 Areas of Interest
Borrowing from the concepts of e-commerce, I assume these five areas would be most important as the outcome of analysis.
* GROWTH
* RETENTION
* STOCK MANAGEMENT
* ENGAGEMENT
* PRICING

SQL key words | meaning | MySQL support
--- | --- | ---
UPSERT | 1. if row exists, update <br/> 2. if not, insert | no
COALESCE | Return the first non-null value in a list | yes
LEAST | return smallest value | yes
GREATEST | return biggest value | yes
DATEDIFF | applied to "yyyy-mm-dd" | yes
WHEN THEN (ELSE) | a.k.a. The SQL CASE Statement | yes
SUM (WHERE condition)| returns the total sum of a numeric column | yes
COUNT (WHERE condition) | returns the number of rows that matches a specified criterion | yes
INNER JOIN | rows which intersect both tables | yes
LEFT OUTER JOIN | same as LEFT JOIN. LEFT = FROM table1, RIGHT = JOIN table2 | yes
FULL OUTER JOIN | same as OUTER JOIN | no
ON | preceded by JOIN | yes
GROUP BY | aggregate rows with the same values | yes

## Retention Studies

suppose we are continuously upserting into dim_customer (which contains the AcquisitionDate),
from the daily_activity table\
we could use the following code:

```sql
UPSERT INTO dim_customer
SELECT
 COALESCE(dc.CustomerId, ac.CustomerId) AS CustomerId, 
 LEAST(dc.AcquisitionDate, ac.ActivityDate) AS AcquisitionDate
FROM dim_customers dc
FULL OUTER JOIN (
 SELECT 
  CustomerId,
  ActivityDate
 FROM daily_activity 
 WHERE 
  ActivityType = 'signup'
  AND ActivityDate = '<RUN_DATE>'
 GROUP BY 1, 2
) ac 
 ON dc.CustomerId = ac.CustomerId
GROUP BY 1, 2
```
where <RUN_DATE> is the incremental date (do this every day)

# footnotes

word | meaning
--- | ---
COALESCE | to come together to form one larger group, substance, etc
SKU | Stock Keeping Unit

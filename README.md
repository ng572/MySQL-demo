# gamelofts_demo
a demo for a job I am applying at gamelofts

# 5 Areas of Interest
Borrowing from the concepts of e-commerce, I assume these five areas would be most important as the outcome of analysis.
* GROWTH
* RETENTION
* STOCK MANAGEMENT
* ENGAGEMENT
* PRICING

## Data Generation

For the purpose of demo-ing the SQL codes I will be generating fake e-commerce data using Python.

A separate set of data will be generated for each task. (meaning independence)

table | description
--- | ---
daily_activity | table containing user activities such as 'sign-up', 'sign-in', 'purchase'
dim_customer | a cohort table to be computed with SQL

see the detailed python code [here](generation.ipynb).

## Loading Data Into MySQL

```sql
CREATE TABLE daily_activity (
	`index` INT,
	`CustomerId` INT,
	`ActivityDate` DATE,
	`ActivityType` INT
)
```

```sql
LOAD DATA INFILE "C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/data.csv" INTO TABLE daily_activity
	FIELDS TERMINATED BY ','
	IGNORE 1 LINES;
```

## Retention Studies

suppose we are continuously upserting into dim_customer (which contains the AcquisitionDate),
from the daily_activity table\
we could use the following code:

```sql
CREATE TABLE dim_customer (
	`CustomerId` INT KEY,
	`AcquisitionDate` DATE
)
```

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

```sql
SELECT
	DATEDIFF(ac.ActivityDate, dc.AcquisitionDate) AS DaySinceAcquisition,
	COUNT(DISTINCT ac.CustomerId) AS D1ActiveCustomers
FROM dim_customer dc
LEFT OUTER JOIN (
	SELECT
		CustomerId, 
		ActivityDate
	FROM daily_activity
	WHERE 
		ActivityType IN ('signup', 'signin')
		AND ActivityDate >= '<MIN_DATE>'
	GROUP BY 1,2
) ac 
	ON dc.CustomerId = ac.CustomerId 
	AND dc.AcquisitionDate <= ac.ActivityDate
WHERE 
	dc.AcquisitionDate => '<MIN_DATE>'
GROUP BY 1
```

# footnotes

word | meaning
--- | ---
COALESCE | to come together to form one larger group, substance, etc
SKU | Stock Keeping Unit

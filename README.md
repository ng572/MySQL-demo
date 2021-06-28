# Areas of Interest
I am trying to instantiate the code and concept written in [this article](https://medium.com/analytics-and-data/e-commerce-analysis-data-structures-and-applications-6420c4fa65e7)\
Borrowing from the concepts, I assume these five areas would be most important as the outcome of e-commerce analysis.
* GROWTH
* RETENTION
* STOCK MANAGEMENT
* ENGAGEMENT
* PRICING

While lacking experience in the field, I opt to demonstrate some skills in Python and SQL that may make me stand out a little.\
I will only be doing a little code that will show customer retention rate.

## Data Generation

For the purpose of demo-ing the SQL codes I will be generating fake user activity data using Python.

It is assumed that each user will initially sign-up for goods / services at a certain date, before the subsequent sign-ins and purchase activities.

Data between 2017-01-01 and 2017-12-31 will be generated.

### Encoding

Code | Activity
--- | ---
0 | sign-up
1 | sign-in
2 | purchase

### Action

see the detailed python code and result [here](generation.ipynb).

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
For some reason there are a few missing CustomerID, but we will let that go for the sake of time.
```sql
>> SELECT count(distinct CustomerID) FROM daily_activity
9889
```

## Retention Studies

suppose we are using a table named `dim_customer` to keep track of each customer's acquisition date.

suppose we are continuously upsert-ing into dim_customer (which contains the AcquisitionDate),
from the daily_activity table\
we could use the following code:

```sql
CREATE TABLE dim_customer (
	`CustomerId` INT KEY,
	`AcquisitionDate` DATE
)
```
```sql
delimiter //

CREATE DEFINER=`root`@`%` PROCEDURE `update_dim_customer`(`date` DATE)
BEGIN
	INSERT IGNORE INTO dim_customer
	SELECT
	 COALESCE(dc.CustomerId, ac.CustomerId) AS CustomerId, 
	 ac.ActivityDate AS AcquisitionDate
	FROM dim_customer dc
	RIGHT JOIN (
	 SELECT 
	  CustomerId,
	  ActivityDate
	 FROM daily_activity 
	 WHERE 
	  ActivityType = 0
	  AND ActivityDate = `date`
	 GROUP BY 1, 2
	) ac 
	 ON dc.CustomerId = ac.CustomerId
	GROUP BY 1, 2;
END
```
```sql
delimiter //

CREATE PROCEDURE doiterate()
BEGIN
  SET @running_date = '2017-01-01';
  SET @num = 0;
  label1: LOOP
    SET @num = @num + 1;
	SET @running_date = DATE_ADD(@running_date, INTERVAL 1 DAY);
	CALL update_dim_customer(@running_date);
    IF @num < 365 THEN
      ITERATE label1;
    END IF;
    LEAVE label1;
  END LOOP label1;
END;
```
Finally, calling the doiterate function
```sql
CALL doiterate()
```
```sql
SELECT * FROM demo.dim_customer ORDER BY AcquisitionDate;
```
CustomerId | AcquisitionDate
--- | ---
3418 | 2017-01-01
1641 | 2017-01-01
5642 | 2017-01-01
4307 | 2017-01-01
7302 | 2017-01-01
... | ...

We have exactly one entry for each distinct CustomerID in daily_activity
```sql
>> SELECT COUNT(*) FROM demo.dim_customer;
9889
```
Let's run the below code for deducing the customer activity level since sign-up\
by replacing <MIN_DATE> with '2017-01-01'
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
		ActivityType IN (0, 1)
		AND ActivityDate >= '<MIN_DATE>'
	GROUP BY 1,2
) ac 
	ON dc.CustomerId = ac.CustomerId 
	AND dc.AcquisitionDate <= ac.ActivityDate
WHERE 
	dc.AcquisitionDate >= '<MIN_DATE>'
GROUP BY 1
```
DaySinceAcquisition | D1ActiveCustomers
--- | ---
0 | 9889
1 | 665
2 | 673
3 | 708
4 | 700
5 | 739

Now let's see the decay pattern, which we expect to be constant (uniformly distributed and not really decaying)\
And it looks ugly but is exactly what we expected. In real life, it should display an actual decaying pattern.

<img src="decay_percent.png" width=60%>

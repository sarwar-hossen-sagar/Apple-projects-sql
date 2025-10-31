# Apple-projects-sql

# 🧮 SQL Analytics Project — Store, Sales & Warranty Insights

This project demonstrates advanced SQL analytical queries for retail data analysis, focusing on **sales performance**, **warranty tracking**, **store insights**, and **product trends**.
Each query explores a different real-world business question using **PostgreSQL** functions and concepts such as **aggregations**, **joins**, **window functions**, and **date/time analysis**.

---

## 📂 Dataset Overview

The dataset consists of the following key tables:

* **STORES** – Store information including country and store details
* **SALES** – Transactional sales data (date, quantity, total price, etc.)
* **PRODUCTS** – Product details (name, price, launch date, category)
* **WARRANTY** – Warranty claim records
* **CATEGORY** – Product category information

---

## 🧠 Analytical Queries

### 1️⃣ Number of Stores per Country

Find how many stores are available in each country.

```sql
SELECT
	COUNTRY,
	COUNT(*) AS NUMBER_OF_COUNTRY
FROM
	STORES
GROUP BY
	1
ORDER BY
	2 DESC;
```

---

### 2️⃣ Total Units Sold by Each Store

Calculate the total number of units sold per store.

```sql
SELECT
	ST.STORE_NAME,
	SUM(S.QUANTITY) AS TOTAL_NUM_SOLD
FROM
	SALES S
	JOIN STORES ST ON S.STORE_ID = ST.STORE_ID
GROUP BY
	1
ORDER BY
	2 DESC;
```

---

### 3️⃣ Total Sales in December 2023

Identify how many sales occurred in **December 2023**.

```sql
SELECT  
    TO_CHAR(SALE_DATE, 'FMMonth-YYYY') AS MONTH,
    SUM(QUANTITY) AS TOTAL_NUMBER_SOLD
FROM Sales
WHERE TO_CHAR(SALE_DATE, 'FMMonth-YYYY') = 'December-2023'
GROUP BY 1;
```

---

### 4️⃣ Stores Without Any Warranty Claims

```sql
SELECT 
    COUNT(*) AS stores_without_warranty
FROM STORES st
WHERE NOT EXISTS (
    SELECT 1
    FROM SALES s
    JOIN WARRANTY w ON s.sale_id = w.sale_id
    WHERE s.store_id = st.store_id
);
```

---

### 5️⃣ CALCULATE THE PERCENTAGE OF WARRANTY CLAIMS MARKED AS 'WARRANTY VOID'

```sql
SELECT
	ROUND(
		COUNT(*)::NUMERIC / (
			SELECT
				COUNT(*)
			FROM
				WARRANTY
		)::NUMERIC * 100,
		2
	) AS PERCENTAGE
FROM
	WARRANTY
WHERE
	REPAIR_STATUS = 'Warranty Void'
GROUP BY
	REPAIR_STATUS;
```

---

### 6️⃣ IDENTIFY WHICH STORES HAS HIGHHEST TOTAL UNIT SOLD IN THELAST YEAR

```sql
SELECT
	STORE_ID,
	SUM(QUANTITY) AS TOTAL_SALE
FROM
	SALES
WHERE
	SALE_DATE >= CURRENT_DATE - INTERVAL '1 YEARS'
GROUP BY
	1
ORDER BY
	2 DESC
LIMIT
	1;
```

---

### 7️⃣ HOW MANY WARRANTY CLAIMS WERE  FILLED IN 2020

```sql
SELECT EXTRACT(
	YEAR
	FROM
		CLAIM_DATE
) AS YEAR,
COUNT(*) TOTAL_CLAIM
FROM
	WARRANTY
WHERE
	EXTRACT(
		YEAR
		FROM
			CLAIM_DATE
	) = 2020
GROUP BY
	1;
```

---

### 8️⃣ FOR EACH STORE, IDENTIFY BEST SELLING DAYS BASED ON HIGHEST QUANTITY  SOLD

```sql
SELECT
	*
FROM
	(
		SELECT
			STORE_ID,
			TO_CHAR(SALE_DATE, 'DAY') AS DAY,
			SUM(QUANTITY) TOTAL_SALE,
			DENSE_RANK() OVER (
				PARTITION BY
					STORE_ID
				ORDER BY
					SUM(QUANTITY)
			) AS RANK
		FROM
			SALES
		GROUP BY
			1,
			2
		ORDER BY
			3
	) AS T1
WHERE
	RANK = 1;
```

---

### 9️⃣ IDENTIFY THE LEAST SELLING PRODUCT IN EACH COUNTRY FOR EACH YEAR BASED ON THE TOTAL NUMBER SOLD

```sql
SELECT
	*
FROM
	(
		SELECT
			P.PRODUCT_NAME,
			ST.COUNTRY,
			EXTRACT(
				YEAR
				FROM
					S.SALE_DATE
			) AS YEAR,
			SUM(S.QUANTITY) AS TOTAL_SALE,
			RANK() OVER (
				PARTITION BY ST.COUNTRY,EXTRACT(YEAR FROM S.SALE_DATE
					)
				ORDER BY
					SUM(S.QUANTITY) DESC
			) AS RANK
		FROM
			PRODUCTS P
			JOIN SALES S ON P.PRODUCT_ID = S.PRODUCT_ID
			JOIN STORES ST ON S.STORE_ID = ST.STORE_ID
		GROUP BY
			1,2,3
	) AS T
WHERE
	RANK = 1
ORDER BY
	2,3;
```

---

### 🔟CALCULATE HOW MANY WARRANTY CLAIMS WERE FILLED WITHHIN LAST 180 DAYS ON A PRODUCT SALE

```sql
SELECT COUNT(*) FROM WARRANTY W JOIN SALES S ON W.SALE_ID=S.SALE_ID
WHERE W.CLAIM_DATE-S.SALE_DATE<=180;
```

---

### 11️⃣ Claims for Products Launched in Last 2 Years

```sql
SELECT P.PRODUCT_NAME, COUNT(*)
FROM PRODUCTS P
JOIN SALES S ON P.PRODUCT_ID = S.PRODUCT_ID
JOIN WARRANTY W ON W.SALE_ID = S.SALE_ID
WHERE P.LAUNCH_DATE >= CURRENT_DATE - INTERVAL '2 YEARS'
GROUP BY 1
ORDER BY 2 DESC;
```

---

### 12️⃣ Months (Last 3 Years) Where USA Sales > 5000 Units

```sql
SELECT TO_CHAR(S.SALE_DATE, 'YYYY-MM') AS MONTH, SUM(S.QUANTITY) AS TOTAL_UNITS
FROM STORES ST
JOIN SALES S ON ST.STORE_ID = S.STORE_ID
WHERE ST.COUNTRY = 'USA'
  AND S.SALE_DATE >= CURRENT_DATE - INTERVAL '3 YEARS'
GROUP BY 1
HAVING SUM(S.QUANTITY) > 5000
ORDER BY 2 DESC;
```

---

### 13️⃣ Product Category with Most Warranty Claims (3 Years)

```sql
SELECT C.CATEGORY_NAME, COUNT(W.REPAIR_STATUS)
FROM CATEGORY C
JOIN PRODUCTS P ON P.CATEGORY_ID = C.CATEGORY_ID
JOIN SALES S ON P.PRODUCT_ID = S.PRODUCT_ID
LEFT JOIN WARRANTY W ON S.SALE_ID = W.SALE_ID
WHERE S.SALE_DATE >= CURRENT_DATE - INTERVAL '3 YEARS'
GROUP BY 1
ORDER BY 2 DESC;
```

---

### 14️⃣ Country-wise Warranty Risk Percentage

```sql
SELECT
	ST.COUNTRY,
	COALESCE(COUNT(W.CLAIM_ID)::NUMERIC, 0) AS TOTAL_CLAIM,
	SUM(S.QUANTITY) AS TOTAL_UNIT_SOLD,
	ROUND(
		COALESCE(COUNT(W.CLAIM_ID)::NUMERIC, 0) / NULLIF(SUM(S.QUANTITY), 0)::NUMERIC * 100,
		2
	) AS RISK
FROM STORES ST
JOIN SALES S ON S.STORE_ID = ST.STORE_ID
JOIN WARRANTY W ON S.SALE_ID = W.SALE_ID
GROUP BY 1
ORDER BY 2 DESC;
```

---

### 15️⃣ Year-over-Year Growth Ratio by Store

```sql
WITH INCOME_DIFF AS (
	SELECT
		ST.STORE_NAME,
		EXTRACT(YEAR FROM S.SALE_DATE) AS YEAR,
		SUM(S.TOTAL_PRICE) AS TOTAL_SELL,
		LAG(SUM(S.TOTAL_PRICE), 1) OVER (PARTITION BY ST.STORE_NAME ORDER BY EXTRACT(YEAR FROM S.SALE_DATE)) AS PREVIOUS_YEAR_SELL
	FROM STORES ST
	JOIN SALES S ON ST.STORE_ID = S.STORE_ID
	GROUP BY 1,2
)
SELECT
	STORE_NAME,
	YEAR,
	TOTAL_SELL,
	PREVIOUS_YEAR_SELL,
	ROUND((TOTAL_SELL - PREVIOUS_YEAR_SELL)::NUMERIC / PREVIOUS_YEAR_SELL, 2) AS GROWTH_RATIO
FROM INCOME_DIFF
WHERE PREVIOUS_YEAR_SELL IS NOT NULL
	AND YEAR <> EXTRACT(YEAR FROM CURRENT_DATE);
```

---

### 16️⃣ Correlation Between Product Price & Warranty Claims

Segmented by Price Range.

```sql
SELECT
	CASE
		WHEN P.PRICE < 500 THEN 'LOW_RANGE_PRICE'
		WHEN P.PRICE BETWEEN 500 AND 1000 THEN 'MID_RANGE_PRODUCT'
		ELSE 'HIGH_RANGE_PRODUCT'
	END AS PRICE_SEGMENTS,
	COUNT(W.CLAIM_ID) AS NO_OF_WARRANTY_CLAIM
FROM PRODUCTS P
JOIN SALES S ON P.PRODUCT_ID = S.PRODUCT_ID
JOIN WARRANTY W ON S.SALE_ID = W.SALE_ID
WHERE S.SALE_DATE >= CURRENT_DATE - INTERVAL '5 YEARS'
GROUP BY 1;
```

---

### 17️⃣ Store with Highest Percentage of “Paid Repaired” Claims

```sql
SELECT
	ST.STORE_ID,
	ST.STORE_NAME,
	COUNT(W.REPAIR_STATUS) AS TOTAL_REPAIR,
	COUNT(CASE WHEN W.REPAIR_STATUS = 'Paid Repaired' THEN 1 END) AS PAID_REPAIR,
	ROUND(
		COUNT(CASE WHEN W.REPAIR_STATUS = 'Paid Repaired' THEN 1 END)::NUMERIC /
		(SELECT COUNT(*) FROM WARRANTY)::NUMERIC * 100,
		2
	) AS PERCENTAGE_OF_CLAIM
FROM STORES ST
JOIN SALES S ON S.STORE_ID = ST.STORE_ID
JOIN WARRANTY W ON S.SALE_ID = W.SALE_ID
GROUP BY 1,2
ORDER BY 5 DESC;
```

---

### 18️⃣ Monthly Running Total Sales (Last 4 Years)

```sql
WITH MONTHLY_SALES AS (
	SELECT
		ST.STORE_ID,
		SUM(S.TOTAL_PRICE) AS MONTHLY_TOTAL,
		TO_CHAR(S.SALE_DATE, 'MON-YYYY') AS MONTH
	FROM STORES ST
	JOIN SALES S ON ST.STORE_ID = S.STORE_ID
	GROUP BY 1,3
)
SELECT
	STORE_ID,
	MONTH,
	MONTHLY_TOTAL,
	SUM(MONTHLY_TOTAL) OVER (PARTITION BY STORE_ID ORDER BY TO_DATE(MONTH, 'MON-YYYY')
	ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RUNNING_TOTAL
FROM MONTHLY_SALES
ORDER BY 1, TO_DATE(MONTH, 'MON-YYYY') DESC;
```

---

### 19️⃣ Product Sales Trends by Lifecycle Periods

(Sales from launch to 6 months, 6–12, 12–18, and beyond 18 months).
*(Query in progress – to be finalized.)*

---

## ⚙️ Technologies Used

* PostgreSQL 15+
* SQL Window Functions
* CTEs (Common Table Expressions)
* Aggregations & Joins
* Date/Time Functions

---

## ✍️ Author

**Md. Sarwar Hossen Sagar**
📧 [sarwarhsagar@gmail.com](mailto:sarwarhsagar@gmail.com)
🌐 [Portfolio](https://www.datascienceportfol.io/sarwarhsagar)

---

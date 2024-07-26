<h1>Case Study #1 - Danny's Diner</h1>

<h5> Link for the case study#1: https://8weeksqlchallenge.com/case-study-1/ </h5>

<h4>Brief Introduction</h4>
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

<h4><h4>Questions and Solutions</h4></h4>

<h5>1. What is the total amount each customer spent at the restaurant?</h5>

```sql
SELECT 
    SUM(PRICE)TOTAL_AMOUNT,CUSTOMER_ID
FROM DANNYS_DINER.MENU M JOIN DANNYS_DINER.SALES S
ON M.PRODUCT_ID=S.PRODUCT_ID
GROUP BY CUSTOMER_ID;
```
![Q1](Case Study #1/q1.png)

<h5>2. How many days has each customer visited the restaurant?</h5>

```sql
SELECT
    CUSTOMER_ID,COUNT(DISTINCT ORDER_DATE) VISITS
FROM DANNYS_DINER.SALES
GROUP BY CUSTOMER_ID;
```

<h5>3. What was the first item from the menu purchased by each customer?</h5>

```sql
SELECT  
    T.CUSTOMER_ID, T.PRODUCT_NAME
FROM
(
    SELECT
        ROW_NUMBER() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY ORDER_DATE) R,PRODUCT_NAME, CUSTOMER_ID
    FROM DANNYS_DINER.MENU M JOIN DANNYS_DINER.SALES S
    ON M.PRODUCT_ID=S.PRODUCT_ID
) T
WHERE T.R=1;
```

<h5>4. What is the most purchased item on the menu and how many times was it purchased by all customers?</h5>

```sql
SELECT 
  	COUNT(*) CT,M.PRODUCT_NAME
FROM
	DANNYS_DINER.SALES S JOIN
 	DANNYS_DINER.MENU M
ON S.PRODUCT_ID=M.PRODUCT_ID
GROUP BY M.PRODUCT_NAME
ORDER BY CT DESC
LIMIT 1 ;
```

<h5>5. Which item was the most popular for each customer?</h5>

```sql
WITH MOST_POPULAR AS 
(
    SELECT 
        SALES.CUSTOMER_ID, 
        MENU.PRODUCT_NAME, 
        COUNT(MENU.PRODUCT_ID) AS ORDER_COUNT,
        DENSE_RANK() OVER (
        PARTITION BY SALES.CUSTOMER_ID 
        ORDER BY COUNT(*) DESC
    ) AS RANK
    FROM DANNYS_DINER.MENU
    INNER JOIN DANNYS_DINER.SALES
    ON MENU.PRODUCT_ID = SALES.PRODUCT_ID
    GROUP BY SALES.CUSTOMER_ID, MENU.PRODUCT_NAME
)

SELECT 
    CUSTOMER_ID, 
    PRODUCT_NAME, 
    ORDER_COUNT
FROM MOST_POPULAR 
WHERE RANK = 1;
```

<h5>6. Which item was purchased first by the customer after they became a member?</h5>

```sql
WITH CTE AS
(
    SELECT
        S.PRODUCT_ID,S.CUSTOMER_ID,S.ORDER_DATE,
    DENSE_RANK() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE ASC) AS R
    FROM DANNYS_DINER.SALES S JOIN DANNYS_DINER.MEMBERS M
    ON
    S.CUSTOMER_ID=M.CUSTOMER_ID
    WHERE M.JOIN_DATE<S.ORDER_DATE
ORDER BY S.ORDER_DATE ASC)
SELECT 
    CTE.CUSTOMER_ID,PRODUCT_NAME
FROM CTE JOIN DANNYS_DINER.MENU M 
ON CTE.PRODUCT_ID=M.PRODUCT_ID
WHERE R=1;
```

</h5>7. Which item was purchased just before the customer became a member?</h5>

```sql
WITH CTE AS
(
    SELECT
        S.PRODUCT_ID,S.CUSTOMER_ID,S.ORDER_DATE,
        ROW_NUMBER() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE DESC) AS R
    FROM DANNYS_DINER.SALES S JOIN DANNYS_DINER.MEMBERS M
    ON
    S.CUSTOMER_ID=M.CUSTOMER_ID
    WHERE M.JOIN_DATE>S.ORDER_DATE
    ORDER BY S.ORDER_DATE ASC)
SELECT 
    CTE.CUSTOMER_ID,PRODUCT_NAME
FROM CTE JOIN DANNYS_DINER.MENU M 
ON CTE.PRODUCT_ID=M.PRODUCT_ID
WHERE R=1;
```

<h5>8. What is the total items and amount spent for each member before they became a member?
What is the total items and amount spent for each member before they became a member?</h5>

```sql
SELECT 
    COUNT(S.PRODUCT_ID)
    ,SUM(PRICE)
    ,S.CUSTOMER_ID
FROM
DANNYS_DINER.SALES S JOIN DANNYS_DINER.MEMBERS M
ON
S.CUSTOMER_ID=M.CUSTOMER_ID
JOIN 
DANNYS_DINER.MENU MN
ON S.PRODUCT_ID=MN.PRODUCT_ID
WHERE S.ORDER_DATE<M.JOIN_DATE
GROUP BY S.CUSTOMER_ID;
```

<h5>9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
</h5>

```sql
SELECT
    CUSTOMER_ID,
    SUM(CASE WHEN M.PRODUCT_NAME='sushi' THEN 20*M.PRICE    
    ELSE M.PRICE*10 END) POINTS
FROM
DANNYS_DINER.SALES S JOIN DANNYS_DINER.MENU M
ON S.PRODUCT_ID=M.PRODUCT_ID
GROUP BY CUSTOMER_ID;
```

<h5>10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
</h5>

```sql
WITH DATES_CTE AS 
(
    SELECT 
        CUSTOMER_ID, 
        JOIN_DATE, 
        JOIN_DATE + INTERVAL '6 DAYS' AS VALID_DATE
    FROM DANNYS_DINER.MEMBERS
)
SELECT 
    SALES.CUSTOMER_ID, 
    SUM(
    CASE
      WHEN MENU.PRODUCT_NAME = 'SUSHI' THEN 2 * 10 * MENU.PRICE
      WHEN SALES.ORDER_DATE BETWEEN DATES.JOIN_DATE AND DATES.VALID_DATE THEN 2 * 10 * MENU.PRICE
      ELSE 10 * MENU.PRICE 
    END
  ) AS POINTS
FROM DANNYS_DINER.SALES
INNER JOIN DATES_CTE AS DATES
ON SALES.CUSTOMER_ID = DATES.CUSTOMER_ID
INNER JOIN DANNYS_DINER.MENU
ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
WHERE SALES.ORDER_DATE <= '2021-01-31'
GROUP BY SALES.CUSTOMER_ID;
```

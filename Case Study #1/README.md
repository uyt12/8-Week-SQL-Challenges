<h1>Case Study #1 - Danny's Diner</h1>

<h5> Link for the case study#1: https://8weeksqlchallenge.com/case-study-1/ </h5>

<h4>Brief Introduction</h4>
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

<h4>ER Diagram </h4>

![ER Diagram](https://github.com/user-attachments/assets/4c702cfc-15b0-4a3a-bf7e-f92e41064802)

<h4><h4>Questions and Solutions</h4></h4>

<h5>1. What is the total amount each customer spent at the restaurant?</h5>

```sql
SELECT 
    SUM(PRICE)TOTAL_AMOUNT,CUSTOMER_ID
FROM DANNYS_DINER.MENU M JOIN DANNYS_DINER.SALES S
ON M.PRODUCT_ID=S.PRODUCT_ID
GROUP BY CUSTOMER_ID;
```
<h5>Output:</h5>

![q1](https://github.com/user-attachments/assets/2c659a6b-a856-4c6f-9671-5121eaf0d338)

<h5>2. How many days has each customer visited the restaurant?</h5>

```sql
SELECT
    CUSTOMER_ID,COUNT(DISTINCT ORDER_DATE) VISITS
FROM DANNYS_DINER.SALES
GROUP BY CUSTOMER_ID;
```
<h5>Output:</h5>

![q2](https://github.com/user-attachments/assets/3882e631-7b7f-4c81-99dc-f99d999eba69)

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
<h5>Output:</h5>

![q3](https://github.com/user-attachments/assets/96695200-ec5b-4634-b971-04b76c959b21)

<h5>4. What is the most purchased item on the menu and how many times was it purchased by all customers?</h5>

```sql
SELECT 
  	M.PRODUCT_NAME,COUNT(*) count
FROM
	DANNYS_DINER.SALES S JOIN
 	DANNYS_DINER.MENU M
ON S.PRODUCT_ID=M.PRODUCT_ID
GROUP BY M.PRODUCT_NAME
ORDER BY CT DESC
LIMIT 1 ;
```
<h5>Output:</h5>

![q4](https://github.com/user-attachments/assets/9f16266e-37cb-44f4-99ac-7be94518bc30)

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
<h5>Output:</h5>

![q5](https://github.com/user-attachments/assets/08e042b4-2382-4804-b907-b67f7fd87494)

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
<h5>Output:</h5>

![q6](https://github.com/user-attachments/assets/ceda3c8c-9e75-439c-b9ea-0cd01feb3877)

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
<h5>Output:</h5>

![q7](https://github.com/user-attachments/assets/6ced3d42-2c4c-4d03-9e14-818b0d89aa22)

<h5>8. What is the total items and amount spent for each member before they became a member?
What is the total items and amount spent for each member before they became a member?</h5>

```sql
SELECT 
	S.CUSTOMER_ID,
    COUNT(S.PRODUCT_ID) total_items
    ,SUM(PRICE) total_sales
    
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
<h5>Output:</h5>

![q8](https://github.com/user-attachments/assets/789f94dd-c5a1-44fc-85ed-2dedf5af0cc9)


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
<h5>Output:</h5>

![q9](https://github.com/user-attachments/assets/2664a71d-992c-432a-add7-daa8d0b54170)

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
<h5>Output:</h5>

![q10](https://github.com/user-attachments/assets/7735edaa-55f0-4d46-9079-d6d4a913739e)



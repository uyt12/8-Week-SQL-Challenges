<h1>Case Study #1</h1>
<h4>Brief Introduction</h4>
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

<h4><h4>Entity Relationship Diagram</h4></h4>
![ER Diagram](/Users/unnatitrivedi/Documents/Git/Images)

<h4><h4>Questions and Solutions</h4></h4>

<h5><h5>1.What is the total amount each customer spent at the restaurant?</h5><h5>
```sql
SELECT 
SUM(PRICE)TOTAL_AMOUNT,CUSTOMER_ID
FROM DANNYS_DINER.MENU M JOIN DANNYS_DINER.SALES S
ON M.PRODUCT_ID=S.PRODUCT_ID
GROUP BY CUSTOMER_ID
'''


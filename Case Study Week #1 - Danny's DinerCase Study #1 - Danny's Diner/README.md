# 🍜 Case Study #1: Danny's Diner SQL Challenge 💻
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
 - [Project Overview](#project-overview)
 - [Entity Relationship Diagram](#entity-relationship-diagram)
 - [Questions and Solution](#questions-and-solution)

This project analyzes the foundational case study from the 8 Weeks SQL Challenge.

***

## Project Overview
This project addresses a series of business questions using SQL to analyze a restaurant's sales data. The primary objective is to gain insights into customer behavior, spending habits, and popular menu items to support data-driven decision-making. The analysis covers a range of topics, including total customer spending, visit frequency, and loyalty program effectiveness.

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Questions and Solution

The queries for this analysis were written and executed using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).

If you have any questions or would like to connect, please feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/edgar-mblanco/).

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC; 
````

### Steps:
1. Combine the ````sales```` and ````menu```` tables using an **INNER JOIN** on ````product_id```` to link each sale to its price.
2. Use **SUM** to aggregate the ````price```` for each customer.
3. **GROUP BY** ````customer_id```` to ensure the sum is calculated individually for each customer.

#### Answer:

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |


- Customer A spent **$76**.
- Customer B spent **$74**.
- Customer C spent **$36**.

***

**2. How many days has each customer visited the restaurant?**

This query counts the unique dates each customer placed an order.

````sql
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
````
#### Steps:
- Use **COUNT(DISTINCT ````order_date````)** to count only the unique visit days for each customer, preventing multiple orders on the same day from being counted as separate visits.

- **GROUP BY** ````customer_id```` to get the visit count for each individual customer.

#### Result:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

This query identifies the first menu item(s) ordered by each customer, accounting for potential ties on the same day.

````sql
WITH ordered_sales AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
````
#### Steps:
- Create a **Common Table Expression (CTE)** called ````ordered_sales```.

- Within the CTE, use the ````DENSE_RANK()```` window function. This function assigns a unique rank to each row within a partition without gaps, which is perfect for handling ties (e.g., two items ordered on the same first day). The data is partitioned by ````customer_id```` and ordered by ````order_date````.

- The outer query selects the ````customer_id```` and ````product_name```` from the CTE, filtering for a rank of **1** to find the first order(s).

#### Result:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first order included both curry and sushi, as they were purchased on the same day.
- Customer B's first order was curry.
- Customer C's first order was ramen.

***
**4. What is the most purchased item on the menu and how many times was it purchased?**
This query identifies the top-selling menu item across all customers.

````sql
SELECT 
  menu.product_name,
  COUNT(sales.product_id) AS most_purchased_item
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
````
#### Steps:
- **JOIN** ````sales```` and ````menu```` tables.
- **COUNT** the number of ````product_id```` occurrences.
- **GROUP BY** ````product_name```` to count purchases for each item.
- **ORDER BY** the count in descending order to place the most popular item at the top.
- Use **LIMIT 1** to return only the single most purchased item.

#### Result:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |

- The most purchased item is **ramen**, with a total of **8** purchases.

***
**5. Which item was the most popular for each customer?**
This query finds each customer's favorite item, defined as the one they ordered most frequently.

````sql
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular 
WHERE rank = 1;
````
#### Steps:
- Create a CTE, ````most_popular````, to first count the number of times each customer ordered each product.

- Use the ````DENSE_RANK()```` window function within the CTE to rank the items for each customer based on their order count. This handles cases where a customer has a tie for their favorite item.

- The main query then filters for the top-ranked item(s) by selecting rows where ````rank```` equals **1**.

#### Result:

| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customers A and C's most popular item is **ramen**.
- Customer B shows equal preference for all three items: **sushi**, **curry**, and **ramen**.

***

**6. Which item was purchased first by the customer after they became a member?**

This query identifies the very first menu item purchased by each customer *after* they officially joined the loyalty program.

```sql
WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

#### Steps:
- Create a CTE, ````joined_as_member````, to filter for sales that occurred after a customer's join_date.

- Use the ````ROW_NUMBER()```` window function to assign a sequential number to each order for each customer, ordered by date. This is ideal for finding the single first purchase.

- The outer query **JOINs** this CTE with the ````menu```` table to get the product name and then filters for the rows where ````row_num```` is **1**.

#### Result:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first purchase as a member was ramen.
- Customer B's first purchase as a member was sushi.

***

**7. Which item was purchased just before the customer became a member?**
This query finds the very last item purchased by each customer *before* their membership began.

````sql
WITH purchased_prior_member AS (
  SELECT 
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date < members.join_date
)

SELECT 
  p_member.customer_id, 
  menu.product_name 
FROM purchased_prior_member AS p_member
INNER JOIN dannys_diner.menu
  ON p_member.product_id = menu.product_id
WHERE rank = 1
ORDER BY p_member.customer_id ASC;
````

#### Steps:
- Create a CTE, ````purchased_prior_member````, filtering for sales that occurred before the ````join_date````.
- Use ````ROW_NUMBER()```` partitioned by customer and ordered by ````order_date```` in descending order. This assigns ````rank = 1```` to the most recent purchase before joining.
- The outer query joins with the ````menu```` table and selects the rows where ````rank```` is **1**.

#### Result:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

- Both customers A and B's last purchase before becoming a member was sushi.

***

**8. What is the total items and amount spent for each member before they became a member?**
This query calculates the total number of items and the total amount spent for each customer who eventually became a member, considering only their non-member purchases.

```sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
#### Steps:
- **JOIN** ````sales```` and ````members```` tables, but include a filter to only consider sales where ````order_date```` is less than the ````join_date````.
- **JOIN** the result with the ````menu```` table to access the ````price````.
- **GROUP BY** ````customer_id```` and use **COUNT** and **SUM** to get the total number of items and the total spent for each customer.

#### Result:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

- Before becoming members, Customer A spent **$25** on **2** items.
- Before becoming members, Customer B spent **$40** on **3** items.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier, how many points would each customer have?**
This query calculates the total loyalty points for each customer, applying a special multiplier for sushi purchases.

```sql
WITH points_cte AS (
  SELECT 
    menu.product_id, 
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10 END AS points
  FROM dannys_diner.menu
)

SELECT 
  sales.customer_id, 
  SUM(points_cte.points) AS total_points
FROM dannys_diner.sales
INNER JOIN points_cte
  ON sales.product_id = points_cte.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
#### Steps:
- Create a CTE, ````points_cte````, to define the point value for each menu item using a CASE statement.
- The **CASE** statement checks if the ````product_id```` is **1** (sushi). If so, it multiplies the price by **20** (10 points * 2x multiplier); otherwise, it multiplies the price by **10**.
- The main query joins the ````sales```` table with the ````points_cte```` and sums the points for each customer.

#### Result:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Customer A has **860** points.
- Customer B has **940** points.
- Customer C has **360** points.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi. How many points do customer A and B have at the end of January?**
This query calculates total points for customers A and B, taking into account a special promotional period for new members.

```sql
WITH dates_cte AS (
  SELECT 
    customer_id, 
      join_date, 
      join_date + 6 AS valid_date, 
      DATE_TRUNC(
        'month', '2021-01-31'::DATE)
        + interval '1 month' 
        - interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
    WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
    ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
INNER JOIN dates_cte AS dates
  ON sales.customer_id = dates.customer_id
  AND dates.join_date <= sales.order_date
  AND sales.order_date <= dates.last_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
```

#### Steps:
- Create a CTE, ````dates_cte````, to determine the ````valid_date```` (the end of the first week of membership) and the ````last_date```` of January 2021.
- **JOIN** the ````sales````, ````members````, and ````menu```` tables with the new ````dates_cte````. The join conditions restrict the analysis to sales that occurred after the customer joined and before the end of January.
- Use a nested **CASE** statement to apply the correct point multiplier:
  - If the product is **'sushi'**, the price is multiplied by **20**.
  - If the order date falls within the first week of membership, the price is also multiplied by **20**.
  - Otherwise, the price is multiplied by **10**.
- The sum of these calculated points is then grouped by customer.

#### Result:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1020 |
| B           | 320 |

- Customer A's total points are **1,020**.
- Customer B's total points are **320**.

***

## BONUS QUESTIONS

**Join All The Things**

**Objective:** Recreate a comprehensive table that includes all order details, along with a column indicating whether the customer was a member at the time of purchase.

```sql
SELECT 
  sales.customer_id, 
  sales.order_date,  
  menu.product_name, 
  menu.price,
  CASE
    WHEN members.join_date > sales.order_date THEN 'N'
    WHEN members.join_date <= sales.order_date THEN 'Y'
    ELSE 'N' END AS member_status
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
ORDER BY members.customer_id, sales.order_date
```

#### Result: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***

**Rank All The Things**

**Objective:** Extend the previous table to include a ranking of customer purchases. The ranking should only be applied to orders made after a customer became a member, with non-member orders showing a NULL value.

```sql
WITH customers_data AS (
  SELECT 
    sales.customer_id, 
    sales.order_date,  
    menu.product_name, 
    menu.price,
    CASE
      WHEN members.join_date > sales.order_date THEN 'N'
      WHEN members.join_date <= sales.order_date THEN 'Y'
      ELSE 'N' END AS member_status
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  *, 
  CASE
    WHEN member_status = 'N' then NULL
    ELSE RANK () OVER (
      PARTITION BY customer_id, member_status
      ORDER BY order_date
  ) END AS ranking
FROM customers_data;
```

#### Result: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

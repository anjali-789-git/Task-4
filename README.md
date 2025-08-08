I’ll walk through each query step-by-step: what it returns, how it works (execution flow)

Query 1 — Total spent per customer

SELECT 
    c.Name AS CustomerName,
    SUM(o.TotalAmount) AS TotalSpent
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
GROUP BY c.CustomerID, c.Name;

What it returns

One row per customer who has at least one order.

TotalSpent is the sum of TotalAmount across that customer’s orders.


How it works (execution flow)

1. JOIN matches each order to its customer (this is an INNER JOIN, so customers without orders are excluded).


2. Rows are grouped by c.CustomerID, c.Name.


3. SUM(o.TotalAmount) aggregates the total per group

---

Query 2 — Number of orders per customer

SELECT 
    c.Name AS CustomerName,
    COUNT(o.OrderID) AS NumberOfOrders
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
GROUP BY c.CustomerID, c.Name;

What it returns

One row per customer who has at least one order, with how many orders they placed.


How it works

1. Inner join pairs orders with customers.


2. GROUP BY groups rows by customer.


3. COUNT(o.OrderID) counts rows per group. Since OrderID is a PK and never NULL, this equals the number of orders

---

Query 3 — Average product price per category

SELECT 
    cat.CategoryName,
    AVG(p.Price) AS AveragePrice
FROM Products p
JOIN ProductCategory pc ON p.ProductID = pc.ProductID
JOIN Categories cat ON pc.CategoryID = cat.CategoryID
GROUP BY cat.CategoryID, cat.CategoryName;

What it returns

One row per category showing the average (AVG) price of products assigned to that category.


How it works

1. Products → ProductCategory → Categories joins the many-to-many relationship.


2. Rows are grouped by category.


3. AVG(p.Price) computes the average price of the matched product rows.

---

Query 4 — Customers with total spent > 1000 (HAVING)

SELECT 
    c.Name AS CustomerName,
    SUM(o.TotalAmount) AS TotalSpent
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
GROUP BY c.CustomerID, c.Name
HAVING SUM(o.TotalAmount) > 1000;

What it returns

Only customers whose aggregated TotalSpent exceeds 1000 (filtering happens after grouping).


Key point: HAVING vs WHERE

WHERE filters rows before aggregation; HAVING filters groups after aggregation. You must use HAVING to filter by an aggregate result like SUM(...).

---

Query 5 — Max and Min stock per category

SELECT 
    cat.CategoryName,
    MAX(p.Stock) AS MaxStock,
    MIN(p.Stock) AS MinStock
FROM Products p
JOIN ProductCategory pc ON p.ProductID = pc.ProductID
JOIN Categories cat ON pc.CategoryID = cat.CategoryID
GROUP BY cat.CategoryName;

What it returns

For each category, the largest (MAX) and smallest (MIN) Stock value found among that category’s products.


NULL handling

SQL aggregate functions MAX and MIN ignore NULLs.

If a category has at least one non-NULL stock value, MIN/MAX will consider only those non-NULL values.

If all stock values in the category are NULL, the result for MAX and MIN will be NULL.



Practical notes

It's better to group by cat.CategoryID, cat.CategoryName for clarity and to satisfy strict SQL modes.

If you want numeric stock defaults instead of NULL, wrap with COALESCE, e.g. COALESCE(MIN(p.Stock),0).


Sample result (given your data)

CategoryName	MaxStock	MinStock

Electronics	50	50
Fashion	200	200
Groceries	NULL	NULL


(Groceries shows NULL because its only product has Stock = NULL.)


---

Final tips & checks

Include IDs in GROUP BY (e.g., GROUP BY c.CustomerID, c.Name) to avoid ambiguity and to be safe with ONLY_FULL_GROUP_BY.

Use LEFT JOIN + COALESCE when you want to include rows that have no matches in the joined table (e.g., customers with zero orders).

For monetary results, consider ROUND(SUM(...),2) or format functions when presenting results.

For category-level aggregates from a many-to-many (Products ↔ Categories), be careful about duplicate/accidental rows — dedupe if necessary.

Indexes that help these queries: Orders(CustomerID), ProductCategory(ProductID, CategoryID), OrderDetails(OrderID, ProductID).



---

If you want, I can:

produce a single “Task 4” report that contains SUM, COUNT, AVG, MAX, MIN, and HAVING examples together;

or give LEFT JOIN variants so customers with zero orders are included;

or show the exact SQL output by running these on a local SQLite/MySQL dump (I can produce the SQL file). Which would you like next?

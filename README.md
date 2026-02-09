# SQL Portfolio: E-Commerce Business Intelligence
This project focuses on analyzing the "thelook_ecommerce" public dataset on Google BigQuery. The goal is to extract business insights regarding sales performance, product trends, and user demographics for the year 2024.

---

## Project: TheLook Ecommerce Analysis (2024)

### 1. Top 10 Products (Alphabetical Order)
**Goal:** Identifying unique products sold in 2024 with 'Complete' status.
```sql
SELECT DISTINCT
  p.name AS product_name
FROM `bigquery-public-data.thelook_ecommerce.order_items` o
JOIN `bigquery-public-data.thelook_ecommerce.products` p
  ON o.product_id = p.id
WHERE EXTRACT(YEAR FROM o.created_at) = 2024
  AND o.status = 'Complete'
ORDER BY product_name ASC
LIMIT 10;
```

### 2. Top 10 Product Categories by Revenue
**Goal:** Identifying categories with the highest total sales in 2024.
```sql
SELECT p.category AS product_category,
      SUM(o.sale_price) total_sales
FROM `bigquery-public-data.thelook_ecommerce.order_items` o
JOIN `bigquery-public-data.thelook_ecommerce.products` p
  ON o.product_id = p.id
WHERE EXTRACT(YEAR FROM o.created_at) = 2024
  AND o.status = 'Complete'
GROUP BY p.category
ORDER BY total_sales DESC
LIMIT 10;
```

### 3. Monthly Sales: Outerwear & Coats Category
**Goal:** Tracking monthly revenue trends for specific categories in 2024.
```sql
SELECT EXTRACT(MONTH FROM o.created_at) AS Month,
        SUM(o.sale_price) as total_sales
FROM `bigquery-public-data.thelook_ecommerce.order_items` o
JOIN `bigquery-public-data.thelook_ecommerce.products` p
  ON o.product_id = p.id
WHERE EXTRACT(YEAR FROM o.created_at) = 2024
      AND o.status = 'Complete'
      AND p.category = 'Outerwear & Coats'
GROUP BY Month
ORDER BY Month ASC;
```

### 4. Top 10 High-Frequency Users
**Goal:** Identifying customers with the highest number of completed orders.
```sql
SELECT CONCAT(u.first_name, ' ', u.last_name) AS full_name,
      COUNT(o.id) AS total_orders
FROM `bigquery-public-data.thelook_ecommerce.order_items` o
JOIN `bigquery-public-data.thelook_ecommerce.users` u
  ON o.user_id = u.id
WHERE EXTRACT(YEAR FROM o.created_at) = 2024
  AND o.status = 'Complete'
GROUP BY full_name
ORDER BY total_orders DESC
LIMIT 10;
```

### 5. Demographic Insight: Brasilian Users (Above Average Age)
**Goal:** Analyzing specific demographics based on geographic and age filters.
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name,
            Age
FROM `bigquery-public-data.thelook_ecommerce.users`
WHERE country = 'Brasil'
    AND age > (SELECT AVG(age) FROM `bigquery-public-data.thelook_ecommerce.users`)
ORDER BY age ASC
LIMIT 10;
```

# SQL Portfolio: Music Store Sales & Customer Insights
This project analyzes  music store database (PostgreSQL) to provide actionable insights for Sales, Marketing, and Product teams.

---

## Sales & Customer Performance Analysis

### 1. Top 10 Countries by Invoice Volume
**Goal:** Identifying geographic markets with the highest transaction frequency.*
```sql
SELECT "BillingCountry" AS Country, COUNT("InvoiceId") AS Total_Invoice
FROM "Invoice"
GROUP BY "BillingCountry"
ORDER BY Total_Invoice DESC, Country ASC 
LIMIT 10;
```

### 2. Top 10 Genres by Revenue
**Goal:** Determining which music genres generate the most sales value.
```sql
SELECT gn."Name" AS Genre_Name, SUM(il."Quantity"*il."UnitPrice") AS total_sales
FROM "InvoiceLine" il
JOIN "Track" tr ON il."TrackId" = tr."TrackId"
JOIN "Genre" gn ON tr."GenreId" = gn."GenreId"
GROUP BY Genre_Name
ORDER BY total_sales DESC
LIMIT 10;
```

### 3. Top 10 High-Spending Customers
**Goal:** Recognizing VIP customers based on their total historical spending.
```sql
SELECT CONCAT(c."FirstName", ' ', c."LastName") AS full_name, c."Email", 
       SUM(i."Total") AS Total_Spending 
FROM "Customer" c
JOIN "Invoice" i ON c."CustomerId" = i."CustomerId"
GROUP BY c."FirstName", c."LastName", c."Email"
ORDER BY Total_Spending DESC
LIMIT 10;
```

### 4. City-Level Insights for Top Markets
**Goal:** Identifying the most active cities within the top performing countries.
```sql
WITH Top_10_Country AS (
    SELECT "BillingCountry", COUNT("InvoiceId") AS Total_Invoice
    FROM "Invoice"
    GROUP BY "BillingCountry"
    ORDER BY Total_Invoice DESC
    LIMIT 10
)
SELECT "BillingCountry" AS Country, "BillingCity" AS City, COUNT("InvoiceId") AS Total_Invoices
FROM "Invoice"
WHERE "BillingCountry" IN (SELECT "BillingCountry" FROM Top_10_Country)
GROUP BY "BillingCountry", "BillingCity"
ORDER BY Total_Invoices DESC;
```

## Product & Marketing Strategy

### 5. UK Market Expansion: Genre Popularity
**Goal:** Recommending 4 tracks for the UK market based on local genre popularity.
```sql
SELECT gn."Name" AS Genre, COUNT(il."Quantity") AS sold_tracks 
FROM "Genre" gn
JOIN "Track" tr ON gn."GenreId" = tr."GenreId"
JOIN "InvoiceLine" il ON tr."TrackId" = il."TrackId"
JOIN "Invoice" i ON il."InvoiceId" = i."InvoiceId"
WHERE i."BillingCountry" = 'United Kingdom'
GROUP BY Genre
ORDER BY sold_tracks DESC;
```

### 6. Top 10 Popular Albums in the USA
**Goal:** Identifying top-selling albums in the USA for potential international marketing.
```sql
SELECT al."Title" AS Album_title, COUNT(il."Quantity") AS Unit_solds
FROM "Album" al
JOIN "Track" tr ON al."AlbumId" = tr."AlbumId"
JOIN "InvoiceLine" il ON tr."TrackId" = il."TrackId"
JOIN "Invoice" i ON il."InvoiceId" = i."InvoiceId"
WHERE i."BillingCountry" = 'USA'
GROUP BY Album_title
ORDER BY Unit_solds DESC
LIMIT 10;
```

### 7. Global Sales Segmentation (The 'Other' Category)
**Goal:** Aggregating countries with single customers into an 'Other' group for cleaner reporting.
```sql
WITH sales AS (
    SELECT 
        CASE WHEN COUNT(DISTINCT "CustomerId") = 1 THEN 'Other' ELSE "BillingCountry" END AS "Country",
        COUNT(DISTINCT "CustomerId") AS Total_Customer,
        SUM("Total") AS Total_sales,
        COUNT("InvoiceId") AS total_invoice
    FROM "Invoice" 
    GROUP BY "BillingCountry"
)
SELECT "Country", SUM(Total_Customer) AS TotalCustomer, SUM(Total_sales) AS TotalSales, 
       ROUND(SUM(Total_sales) / NULLIF(SUM(Total_Customer), 0), 3) AS AverageSalesPerCustomer,
       ROUND(SUM(Total_sales) / NULLIF(SUM(total_invoice), 0), 3) AS AverageOrderValue
FROM sales
GROUP BY "Country"
ORDER BY TotalCustomer DESC, TotalSales DESC, "Country" ASC;
```

### 8. Underperforming Genres in the USA
**Goal:** Identifying genres that need marketing boosts or promotions.
```sql
SELECT gn."Name" AS Genre, COUNT(il."Quantity") AS quantity, 
       SUM(il."Quantity"*il."UnitPrice") AS total_sales
FROM "Genre" gn
JOIN "Track" tr ON gn."GenreId" = tr."GenreId"
JOIN "InvoiceLine" il ON tr."TrackId" = il."TrackId"
JOIN "Invoice" i ON il."InvoiceId" = i."InvoiceId"
WHERE i."BillingCountry" = 'USA'
GROUP BY Genre
ORDER BY total_sales ASC;
```

### 9. Customer Personalization: Favorite Genre by Spending
**Goal:** Finding the top genre for each customer to enable targeted advertising.
```sql
WITH top_genre AS (
    SELECT c."CustomerId", c."FirstName", c."LastName", gn."Name" AS genre, 
           SUM(il."UnitPrice"*il."Quantity") AS total_spent
    FROM "Customer" c 
    JOIN "Invoice" i ON c."CustomerId" = i."CustomerId"
    JOIN "InvoiceLine" il i."InvoiceId" = il."InvoiceId"
    JOIN "Track" tr ON il."TrackId" = tr."TrackId"
    JOIN "Genre" gn ON tr."GenreId" = gn."GenreId"
    GROUP BY 1, 2, 3, 4
),
genre_top AS (
    SELECT *, RANK() OVER(PARTITION BY "CustomerId" ORDER BY total_spent DESC) ranked
    FROM top_genre
)
SELECT "CustomerId", "FirstName", "LastName", genre, total_spent
FROM genre_top
WHERE ranked = 1
ORDER BY total_spent DESC;
```

### 10. Top 10 High-Spending Countries
**Goal:** Assisting the Marketing team in allocating budget to high-value regions.
```sql
SELECT "BillingCountry" AS Country, SUM("Total") AS total_spending
FROM "Invoice" 
GROUP BY Country
ORDER BY total_spending DESC
LIMIT 10;
```

My biggest risk area was introducing errors when cleaning the data. The tables were extremely messy, and required lots of cleaning to make usable. Some of these queries were more complex than others, and so there was always a worry of deleting or inserting the wrong data. To ensure this didn't happen, I kept all the original tables intact and created copies to work with. I then consistently used queries to check my data against what I expected, and the original set.

- one example of this is using different joins to make sure I got the amount of rows I was expecting
  - inner join
```SQL
    SELECT s."productSKU", s."total_ordered", name, "stockLevel", "restockingLeadTime", "sentimentScore", "sentimentMagnitude", ratio 
    FROM sales_by_sku_original s
    JOIN sales_report_original r USING("productSKU")
```
- - left outer join
```SQL
    SELECT s."productSKU", s."total_ordered", name, "stockLevel", "restockingLeadTime", "sentimentScore", "sentimentMagnitude", ratio 
    FROM sales_by_sku_original s
    LEFT OUTER JOIN sales_report_original r USING("productSKU")
```
- - right outer join 
```SQL
    SELECT s."productSKU", s."total_ordered", name, "stockLevel", "restockingLeadTime", "sentimentScore", "sentimentMagnitude", ratio
    FROM sales_by_sku_original s
    LEFT OUTER JOIN sales_report_original r USING("productSKU")
```
- - and comparing to the original tables to see distinct row counts
```SQL
    SELECT distinct "productSKU"
    FROM sales_by_sku_original
```
```SQL
    SELECT distinct "productSKU"
    FROM sales_report_original
```
- I also used the following query as a sanity check before replacing null values, even though it wasn't necessary
```SQL
    SELECT "timeOnSite", coalesce("timeOnSite", 'N/A')
	FROM all_sessions_original
	WHERE "timeOnSite" != coalesce("timeOnSite", 'N/A')
```
- After calculating `productRevenue` in `all_sessions`, I used this query to ensure there were no rows where products were sold and the revenue was incorrect
```SQL
    SELECT "productQuantity", "productPrice", "productRevenue", coalesce("productRevenue", "productQuantity" * "productPrice")
	FROM all_sessions
	WHERE "productQuantity" > 0
```
- I used queries like this to make sure I was targetting the correct rows before either tidying or replacing  strings. I also ran them afterwards to ensure I got the output expected.
```SQL
    SELECT *
	FROM all_sessions
	WHERE "productName" !~ '^[[:upper:]]' AND "productName" !~ '^[[:digit:]]'
```
```SQL
    SELECT *
	FROM all_sessions
	WHERE "pagePathLevel1" !~ '^[/]'
```

- After removing duplicate data, I would run queries like this to ensure they were deleted. I would also compare the expected number of rows like mentioned above
```SQL
    SELECT "SKU", COUNT("SKU") 
    FROM products
    GROUP BY "SKU" 
    ORDER BY COUNT("SKU") DESC
```
- I used the following query to see if rows were capitalised before cleaning 
```SQL
    SELECT city 
    FROM all_sessions 
    WHERE city !~ '^[[:upper:]]'
```
The next thing I was worried about is making sure I correctly understood the data so that I wouldn't make erroneus calculations, or equate data incorectly. The biggest case of this was trying to relate information from the `analytics` table to other tables. It had the most rows of information, but it was missing key columns to easily relate the data. I was hoping to normalise the dataset so I was trying to relate the table using queries like below.
- With this query, I was trying to see if any orders matched between `all_sessions` and `analytics` by using the customer's `fullVisitorID` and the unix timestamp for when they first visited during that session in `visitID` as a primary key. I then checked to see if the order totals matched at all.
```SQL
    SELECT a."visitId" || a."fullVisitorID", s."visitID" || s."fullVisitorID"
	FROM analytics a
	JOIN all_sessions s ON a."visitId" || a."fullVisitorID" = s."visitID" || s."fullVisitorID"
	WHERE a.revenue = s."productRevenue"
```
- With this query, I tried to see if I could tie `fullVisitorID`'s to specific locations. The output showed that some IDs accessed the site from multiple countries, so I couldn't tie sales from `analytics` to a region.
```SQL
	SELECT a."fullVisitorID", a.country, b."fullVisitorID", b.country
	FROM all_sessions a
	JOIN all_sessions b USING("fullVisitorID")
	WHERE a."fullVisitorID" = b."fullVisitorID" AND a.country != b.country
```
- I used this query to figure out that there are 45 sales between `analytics` and `all_sessions` where the `visitID` and `fullVisitorID` match, which we can assume are from the same customer.`all_sessions` has rows for 33 of them. It was around this point that I saw a mentor mention that it may be impossible to link data between certain tables in the dataset, and realised that trying to normalise it would be out of the scope of the project.
```SQL
	SELECT DISTINCT a."visitId" || a."fullVisitorID", s."visitID" || s."fullVisitorID", a.revenue, units_sold, unit_price,s."productRevenue", "productQuantity", "productPrice"
	FROM analytics a
	JOIN all_sessions s ON a."visitId" || a."fullVisitorID" = s."visitID" || s."fullVisitorID"
	WHERE "productQuantity" != 0 AND revenue IS NOT NULL
```
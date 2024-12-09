### analytics 

- divided `unit_price` by 1,000,000
- divided `revenue` by 1,000,000
```SQL
    UPDATE analytics
    SET revenue = revenue/1000000
```
- for `time` I cast unix timestamp to time
```SQL
    UPDATE analytics
    SET "visitStartTime" = to_timestamp("visitStartTime"::int)::time
```
- converted timestamp to date for readability with local timezone
```SQL
    UPDATE analytics
    SET "visitStartTime" = to_timestamp("visitStartTime"::int)::date
```
- dropped column `userID` since it was enitrely null
- dropped column `socialEngagementType` since the entire column was "Not Socially Engaged"
```SQL
    ALTER TABLE analytics
    DROP COLUMN "userID",
    DROP COLUMN "socialEngagementType"
```

### all_sessions
- replaced `time` by casting the unix timestamp from 'visitID'
- I did the same for `date` so that it would be according to my local timezone
```SQL
    UPDATE all_sessions
    SET time = to_timestamp("visitID"::int)::time,
    SET date = to_timestamp("visitID"::int)::date
```
- divided `productPrice`, `productRevenue` and `transactionRevenue` by 1,000,000
```SQL
    UPDATE all_sessions
    SET "productPrice" = "productRevenue"/1000000,
    SET "productRevenue" = "productRevenue"/1000000,
    SET "transactionRevenue" = "transactionRevenue"/1000000
```
- removed `itemQuantity`, `itemRevenue`, `productRefundAmount`, `searchKeyword` since they were always null
- removed `currencyCode` since it was either null or always USD
- removed`productVariant` since there were only values for 40 rows
- removed `productName` since it was redundant with the same column from `products`
```SQL
    ALTER TABLE all_session
    DROP COLUMN "itemQuantity",
    DROP COLUMN "itemRevenue",
    DROP COLUMN "productRefundAmount",
    DROP COLUMN "searchKeyword",
    DROP COLUMN "currencyCode",
    DROP COLUMN "productVariant",
    DROP COLUMN "productName"
```
- removed duplicate rows
  - Here I created a temporary concatenation of the columns that would later be the primary key to make the following query easier
  - I also made a temporary serial primary key so I could identify rows
```SQL
	ALTER TABLE all_sessions
	ADD "temppk" varchar, 
	ADD id serial
```
```SQL
	UPDATE all_sessions
	SET "temppk" = "fullVisitorID" || "visitID"::varchar || "productSKU"
```	
- this query removed duplicate rows
```SQL
	DELETE FROM
    all_sessions a
    USING all_sessions b
	WHERE
    	a.id < b.id
    	AND a.temppk = b.temppk
```

- removed nulls in `sessionQualityDim`, `city` and `country`
```SQL
	UPDATE all_sessions
	SET "sessionQualityDim" = COALESCE("sessionQualityDim", 'N/A'),
	SET city = COALESCE(city, 'N/A'),
	SET country = COALESCE(country, 'N/A')
```
- replaced any filler for missing data in `city` and `country` with N/A to keep things standardised
- cities and countries were always capitalised in their respective columns, so any row that wasn't capitalised was a filler

```SQL
    UPDATE all_sessions
	SET city = CASE
			    WHEN city !~ '^[[:upper:]]' THEN 'N/A'
    			ELSE city 
				END,
	SET country = CASE
			    WHEN country !~ '^[[:upper:]]' THEN 'N/A'
    			ELSE country 
				END
```
- removed more null values from columns
  - for `productRevenue`, when data was available it was roughly equal to `productQuantity * productPrice` so I used that where possible
   - for most of the `integer` or `real` columns, I left them null because I did not want to affect the result of calculations
   - for `productQuantity` I did replace null values with 0 since that likely meant the customer didn't order anything
```SQL
    UPDATE all_sessions
    SET "timeOnSite" =  COALESCE("timeOnSite", 'N/A'),
    SET "productquantity" = COALESCE("productquantity", 0)
```
```SQL
    UPDATE all_sessions
    SET "productRevenue" =  COALESCE("productRevenue", "productQuantity" * "productPrice"),
```
- dropped `transactionRevenue` because there were only 4 values and they were equal to `totalTransactionRevenue`
- dropped `transactionID` because there were only 9 values and I couldn't extrapolate for the other rows from that
- dropped `totalTransactionRevenue` because there only 81 columns and I couldn't use the data extrapolate for the same, or even different columns
```SQL
    ALTER TABLE all_sessions
    DROP COLUMN "transactionRevenue",
    DROP COLUMN "transactionID",
    DROP COLUMN "totalTransactionRevenue"
```

### products

- replaced the names with the ones from `all_sessions` since they were more descriptive and better formatted
- also took SKUs that were in `all_sessions` but not `products`
```SQL
    CREATE TABLE products AS(
    	SELECT DISTINCT  "SKU","productSKU", name, "productName", "productPrice" as price,p."orderedQuantity", "stockLevel", "restockingLeadTime", "sentimentScore", 	"sentimentMagnitude"
    	FROM all_sessions s
    	FULL OUTER JOIN products_original p ON p."SKU"=s."productSKU"
	)
```
- This code replaced any missing product names or SKUs from one column in the new table into the other, creating one column for each that contained info from `products` and `all_sessions`
```SQL
    UPDATE products
	SET "SKU" = COALESCE("SKU", "productSKU"),
	SET "productName" = COALESCE("productName", name)
```
- Here I dropped the columns that were no longer necessary
```SQL
	ALTER TABLE products
	DROP COLUMN "productSKU",
	DROP COLUMN name
```	

- cleaned up 'productName' by removing any trailing and leading spaces
```SQL
	UPDATE products
	SET "productName" = TRIM(both ' ' from "productName")
```
- changed the name of 'productName'for personal preference
```SQL
    ALTER TABLE products
    RENAME "productName" TO name;
```
- Since `all_sessions` contained multiple prices for each SKU, I kept the higher price as that would most likely be the normal sale price
```SQL
	DELETE FROM products a
    USING products b
	WHERE
    	a.price < b.price
    	AND a."SKU" = b."SKU"
```
- Some SKUs ended up with multiple rows with different names. I kept the longer names since they were more descriptive
```SQL
	DELETE FROM products a
    USING products b
	WHERE
    	Length(a.name) < Length(b.name)
    	AND a."SKU" = b."SKU";
```	
- Left null values for the columns with integer and real values since replacing those would affect calculations
	

### sales_by_sku and sales_report
- combined both tables since a lot of information was redundant
```SQL
    CREATE TABLE sales_by_sku AS (
	    SELECT s."productSKU", s."total_ordered", name, "stockLevel", "restockingLeadTime", "sentimentScore", "sentimentMagnitude", ratio 
	    FROM sales_by_sku_original s
    	LEFT OUTER JOIN sales_report_original r USING("productSKU")
	)
```

- deleted SKUS that had no information in other columns
```SQL
		DELETE FROM sales_by_sku 
		WHERE "productSKU" NOT IN (
    	    SELECT "productSKU"
		    from sales_by_sku s
		    JOIN products p ON s."productSKU"=p."SKU"
		)
```

- dropped redundant columns found in the `products` table
```SQL
		ALTER TABLE sales_by_sku
		DROP COLUMN "name", 
		DROP COLUMN "stockLevel",
		DROP COLUMN "restockingLeadTime",
	    DROP COLUMN "sentimentScore",
		DROP COLUMN "sentimentMagnitude"
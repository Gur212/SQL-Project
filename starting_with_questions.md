# Question 1: Which cities and countries have the highest level of transaction revenues on the site?

##### SQL Queries

###### For Cities
``` SQL 
    SELECT city, SUM("productRevenue")
    FROM all_sessions
    WHERE city != 'N/A'
    GROUP BY city
    ORDER BY SUM("productRevenue") DESC
    LIMIT 5
```
###### For Countries
``` SQL
    SELECT country, SUM("productRevenue")
    FROM all_sessions
    WHERE country != 'N/A'
    GROUP BY country
    ORDER BY SUM("productRevenue") DESC
    LIMIT 5
```

##### Answer
The top cities by revenue are:
1. Salem
2. New York
3. Mountain View
4. Sunnyvale
5. San Francisco

The top countries by revenue are:
1. United States
2. Argentina
3. Ireland
4. Spain
5. Canada


# Question 2: What is the average number of products ordered from visitors in each city and country?

##### SQL Query  

###### For cities
``` SQL
    SELECT city, AVG("productQuantity")
    FROM all_sessions 
    WHERE "productQuantity" != 0
    GROUP BY city
    ORDER BY AVG("productQuantity") DESC
```
###### For countries
``` SQL
    SELECT country, AVG("productQuantity")
    FROM all_sessions 
    WHERE "productQuantity" != 0
    GROUP BY country
    ORDER BY AVG("productQuantity") DESC
```
##### Answer  

| **City**   | ** Average Products Ordered** |
|---------------|-------------------------------|
| Madrid        | 10                            |
| Salem         | 8                             |
| Atlanta       | 4                             |
| Houston       | 2                             |
| New York      | 1                             |
| Detroit       | 1                             |
| Dublin        | 1                             |
| Los Angeles   | 1                             |
| Mountain View | 1                             |
| Palo Alto     | 1                             |
| San Francisco | 1                             |
| San Jose      | 1                             |
| Seattle       | 1                             |
| Ann Arbor     | 1                             |
| Sunnyvale     | 1                             |
| Bengaluru     | 1                             |
| Chicago       | 1                             |
| Colombus      | 1                             |
| Dallas        | 1                             |

| **Country**   | **Average Products Ordered**  |
|---------------|-------------------------------|
| Spain         | 10                            |
| United States | 4                             |
| Colombia      | 1                             |
| Finland       | 1                             |
| France        | 1                             |
| Argentina     | 1                             |
| Ireland       | 1                             |
| Mexico        | 1                             |
| India         | 1                             |
| Canada        | 1                             |

# Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

##### SQL Query

###### For cities
``` SQL
    SELECT *
    FROM (
	    SELECT city, "productCategory", COUNT("productCategory") AS category_count
	    FROM all_sessions
	    WHERE "productQuantity" != 0
	    GROUP BY city, "productCategory"
    )
    ORDER BY city, category_count DESC
```

###### For countries
``` SQL
    SELECT *
    FROM (
    	SELECT country, "productCategory", COUNT("productCategory") AS category_count
	    FROM all_sessions
    	WHERE "productQuantity" != 0
    	GROUP BY country, "productCategory"
    )
    ORDER BY country, category_count DESC
```
##### Answer

There isn't enough data about orders from other countries, but Nest products seem to be popular in the United States. Unfortunately, there isn't enough data to glean any patterns about products ordered from each city.

# Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

##### SQL Query

###### For cities
``` SQL
    SELECT city, "productSKU", COUNT("productSKU") AS product_count	
    FROM all_sessions
    WHERE "productQuantity" != 0 AND city != 'N/A'
    GROUP BY city, "productSKU"
```

###### For countries
``` SQL
    SELECT country, "productSKU", name, COUNT("productSKU") AS product_count	
    FROM all_sessions a
	JOIN products p ON a."productSKU" = p."SKU"
    WHERE "productQuantity" != 0 AND country != 'N/A'
    GROUP BY country, "productSKU", name
```
##### Answer

There isn't enough information for cities or other countries, but the top selling products in the United States are the Nest Outdoor Camera, Indoor Camera and Thermostat


# Question 5: Can we summarize the impact of revenue generated from each city/country?

##### SQL Query

###### For cities

``` SQL
    SELECT city, SUM("productRevenue")
    FROM all_sessions
    WHERE city != 'N/A' and "productRevenue" != 0
    GROUP BY city
    ORDER BY SUM("productRevenue") DESC
```

###### For countries

``` SQL
    SELECT country, SUM("productRevenue")
    FROM all_sessions
    WHERE country != 'N/A' and "productRevenue" != 0
    GROUP BY country
    ORDER BY SUM("productRevenue") DESC
```
##### Answer
The impact of the top 10 cities are much higher than the rest, especially for the top 3. In terms of countries, the United States dwarfs the other countries in revenue generated.














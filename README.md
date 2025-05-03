### DATA CLEANING
1.	I dropped columns that would not be useful for the analysis eg. The separate date strings for shipment and order dates.
2.	Checked for duplicates and null values. There were none.

### DATA WRANGLING
1.	The order and shipment dates were in string form with each day, month and year in its own column. I concatenated the dates and converted them to a date data type which I used to create new columns; shipment and order date.
2.	I performed a merge between the warehouse order fulfillment sheet and the orders and shipments sheet
Sheet to create a new table called Fulfill. The data in the warehouse order fulfillment sheet contains the days a warehouse takes to begin shipment of an ordered item which would help in calculating the expected shipment date of a product
                      expected shipment = (order date + days it takes for shipment to begin).
3.	The actual shipment dates is already provided in the data which makes it easy in finding delays. 
4.	I created a new column called performance to know whether a shipment was on time or late. I achieved this by creating a function that checks whether the actual shipment day is less or equal to the expected shipment day. If it’s less I categorized it as “On Time” and if its greater I categorized it as “Late”

### DATA ANALYSIS
I categorized the analysis into;
 * Profits vs Sales (yearly vs monthly)
 * Market Analysis
 * 	Product Analysis
 * 	Performance Analysis
(The analysis was done using matplotlib using category bar charts, line charts, horizontal bar charts and a pie chart.)


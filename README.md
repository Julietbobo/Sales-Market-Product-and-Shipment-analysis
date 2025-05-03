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
- I categorized the analysis into;
   - Profits vs Sales (yearly vs monthly)
   - Market Analysis
   -	Product Analysis
   -	Performance Analysis
(The analysis was done using matplotlib using category bar charts, line charts, horizontal bar charts and a pie chart.).

- To perform the **monthly profits and sales analysis** I first extracted the months from the order date which I grouped in order to find the total profits and sales for each month.
- The grouped data would return a series which I sorted based on the index(in this case the months would be the index since I used the pd.CategoricalIndex to turn it into an index)
```
months= orders['Order_date'].dt.strftime('%b')
month_order = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

monthsales=orders.groupby(months)["Gross Sales"].sum()
monthsales.index=pd.CategoricalIndex(monthsales.index, categories=month_order, ordered=True)
monthsales=monthsales.sort_index()

```
- Below is a snippet of one of the codes used to plot the monthly sales and profit charts which is a combination of a bar and line chart
```
month_order = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
sales=monthsales.values
profits=monthprofit.values

fig,ax1=plt.subplots(figsize=(5,4))

ax1.bar(month_order,monthsales,color='b', label="Sales")
ax1.tick_params(axis='y', colors="b")


ax2=ax1.twinx()
ax2.plot(month_order,monthprofit, 'c-o', label="Profits")
ax2.tick_params(axis='y', colors="c")
ax2.set_title("Monthly Profits vs Sales")
lines1, labels1=ax1.get_legend_handles_labels()
lines2, labels2=ax2.get_legend_handles_labels()
ax1.legend(lines1+lines2, labels1+labels2, loc="upper right")
plt.show()
```
- I applied the same steps I used in the monthly for the yearly analysis of profits and sales.
- For the **market analysis**  I analysed orders by markets, orders by region and profits and sales by market
- Below is a snippet of how I found orders by market and plotted it. The main thing was grouping the markets and doing a count of orders per market
```
mkt=orders.groupby('Customer Market')['Order ID'].count()
mktlist=list(orders.groupby('Customer Market').groups.keys())
mktvalues=mkt.values

plt.bar(mktlist, mktvalues, color='#730e5b')
plt.title('Orders by Market',color='#965a0b')
plt.xticks(color='#965a0b')
plt.yticks(color='#965a0b')
plt.show()

```


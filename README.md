### DATA SOURCE AND LOADING
- The file is an excel file with several sheets sourced from [here](https://herdataproject.gumroad.com/l/supply_chain).
- I first worked with data from the orders_and_fulfillment sheet which I called orders

```
import pandas as pd
file=pd.read_excel('/content/drive/MyDrive/Colab Notebooks/Supply Chain Dataset.xlsx', sheet_name=None)
orders=file.get('orders_and_shipments')
orders.columns=orders.columns.str.strip()
```
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
5.	I re-ordered the orders data frame to reflect the changes I had made
```
new_order=['Order ID',
 'Order Item ID',
 'Order_date',
 'Order Time',
 'Order Quantity',
 'Product Department',
 'Product Category',
 'Product Name',
 'Customer ID',
 'Customer Market',
 'Customer Region',
 'Customer Country',
 'Warehouse Country',
 'Shipment_date',
 'Shipment Mode',
 'Shipment Days - Scheduled',
 'Gross Sales',
 'Discount %',
 'Profit'
          ]
orders=orders[new_order]
```

### DATA ANALYSIS
- I categorized the analysis into;
   - Profits vs Sales (yearly vs monthly)
   - Market Analysis
   -	Product Analysis
   -	Performance Analysis
(The visualization was done using matplotlib and seaborn using category bar charts, line charts, horizontal bar charts, heatmap and a pie chart.).

 #### Profits and sales analysis
![ps monthly](https://github.com/user-attachments/assets/cf2a563f-1546-4174-8aa4-375114a608f1)
![ps year](https://github.com/user-attachments/assets/63f9f481-3f23-4ea5-b1d6-0aa81c98cc36)

- To perform the monthly profits and sales analysis I first extracted the months from the order date which I grouped in order to find the total profits and sales for each month.
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
 #### Market analysis
![ps by mkt](https://github.com/user-attachments/assets/a1ccf0ce-75e8-4cbe-a1d3-bc20839a00a4)
- I applied the same steps I used in the monthly for the yearly analysis of profits and sales.
- For the market analysis I analysed orders by markets, orders by region and profits and sales by market
![orders by reg](https://github.com/user-attachments/assets/b9c20861-97f7-4d5d-bccd-715b2bfe2584)
![orders by mkt](https://github.com/user-attachments/assets/f3f328cd-56cb-4025-aee3-2ff46938be2d)

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
 #### Product analysis
 ![TOPBOTTOM N](https://github.com/user-attachments/assets/ad125939-5f2b-4fa0-ae6b-4bb8b3dc9b80)
 - The product analysis was achived by grouping the product categories and finding the top 5 and bottom 5 products by profits and sales. I  used subplots to show the different categories and I thought it would be cleaner.

```
saleprof=orders.groupby('Product Category')[['Profit', 'Gross Sales']].sum()
profittop=saleprof.sort_values('Profit', ascending=False).head(5)
profitbottom=saleprof.sort_values('Profit', ascending=False).tail(5)

salestop=saleprof.sort_values('Gross Sales', ascending=False).head(5)
salesbottom=saleprof.sort_values('Gross Sales', ascending=False).tail(5)

products=orders.groupby('Product Category')['Order ID'].count().sort_values(ascending=False)
prodorderstop=products.head(5)
prodordersbottom=products.tail(5)


fig, axs=plt.subplots(6,1,figsize=(10,23))

profittop['Profit'].plot(kind='bar', ax=axs[0])
axs[0].set_title('Top 5 product categories by profit')
axs[0].set_xlabel('')
axs[0].tick_params(axis='x', rotation=360)

```
#### Shipment analysis
![pie chart](https://github.com/user-attachments/assets/9dbf14dc-09cd-4ad5-b1cf-bece1c304bfb)

- I created a new column called performance to know whether a shipment was on time or late. I achieved this by creating a function that checks whether the actual shipment day is less or equal to the expected shipment day. If it’s less I categorized it as “On Time” and if its greater I categorized it as “Late”. I used a pie chart to visualize this analysis.

```
fulfill=file.get('fulfillment')
fulfill.columns=fulfill.columns.str.strip()
orderfull = pd.merge(orders, fulfill, on='Product Name', how='left')
orderfull = orderfull.rename(columns={'Warehouse Order Fulfillment (days)': 'WOFD'})
orderfull = orderfull.rename(columns={'Shipment_date': 'Actual_Shipment_Day'})
orderfull['WOFD']=np.ceil(orderfull['WOFD'])

orderfull['Order_date']=pd.to_datetime(orderfull['Order_date'])
orderfull['Expected_shipment_date']=orderfull['Order_date']+pd.to_timedelta(orderfull['WOFD'], unit='D')
#orderfull[['Order_date', 'WOFD', 'Expected_shipment_date',  'Actual_Shipment_Day']]

def perform(row):
  if row['Actual_Shipment_Day'] <=row['Expected_shipment_date']:
    return 'On Time'
  else:
    return 'Late'
orderfull['Performance']=orderfull.apply(perform, axis=1)
grouped=orderfull['Performance'].value_counts()
labelling=grouped.index
cols=['#bf7811','#e6c595']
explodes=[0.05, 0.05]
plt.pie(grouped, labels=labelling, colors=cols, explode=explodes, autopct='%1.0f%%')
plt.title('Performance by shipment')
plt.show()
```
### Performance Analysis
- I created a heatmap to analyse how products were ordered based on the market.

```
  grouped=orders.groupby(['Customer Market','Product Category' ])['Order ID'].count().reset_index()
datas=grouped.pivot(index='Customer Market', columns='Product Category', values='Order ID')

mycols=['#bd291e', '#f0d28d', '#a4f08d']
segm = LinearSegmentedColormap.from_list('mycmap', mycols)
plt.figure(figsize=(18,8))
sb.heatmap(datas, annot=False, fmt='g', cmap=segm)

```
![heatmap](https://github.com/user-attachments/assets/4452add0-8d8c-4cf2-80ac-d9b3c52a5902)

- I also analysed how each market was performing based on its shipment schedules.

![barh](https://github.com/user-attachments/assets/3cc7b3cc-372b-4123-97ae-58f28ca36968)

### Findings
1. July and August have the highest sales
2. Europe and LATAM has the largest customer base and therefore the highest profits and sales
3. Cleats, fishing, cardio equipment, women's apparel and indoor/outdoor games are the top 5 profitable product categories
4. Fishing, Cleats, Camping and outdoor, cardio equipment and women's apparel have the highest revenue
5. Cleats, men's footwear, women's apparel, indoor/outdoor and fishing are the top five ordered products
6. 84% of the shipments are early and 16% are late
7. There are product categories that have very minimal order in all markets, some are just ordered in one region minimally while others are well ordered in all most markets



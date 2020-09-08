# Question 1

##The Fault In Our AOV

###_The Problem_

#####On Shopify, we have exactly 100 sneaker shops, and each of these shops sells only one model of shoe. We want to do some analysis of the average order value (AOV). When we look at orders data over a 30 day window, we naively calculate an AOV of \$3145.13. Given that we know these shops are selling sneakers, a relatively affordable item, something seems wrong with our analysis.

#####1. Think about what could be going wrong with our calculation. Think about a better way to evaluate this data.

#####2. What metric would you report for this dataset?

#####3. What is its value?

###_Analysis_

#####Let's take a peek at the data first.

In [102]:

    import pandas as pd

    df = pd.read_csv('https://docs.google.com/spreadsheets/d/e/2PACX-1vSvtQCCuNgG8S30A7fhO0ndjSxzAyL154_NUNCcJ5dwAV_DPBC67dxx3AM7jiQuVmPVVoyw18OyVBVF/pub?gid=0&single=true&output=csv')
    df.set_index('order_id', inplace=True)
    print(df.head())

              shop_id  user_id  order_amount  total_items payment_method  \
    order_id
    1              53      746           224            2           cash
    2              92      925            90            1           cash
    3              44      861           144            1           cash
    4              18      935           156            1    credit_card
    5              18      883           156            1    credit_card

                       created_at
    order_id
    1         2017-03-13 12:36:56
    2         2017-03-03 17:38:52
    3          2017-03-14 4:23:56
    4         2017-03-26 12:43:37
    5          2017-03-01 4:35:11

#####We can see that each row corresponds to one order. Let's look at a summary of relevant data.

In [75]:

    order_details_df = df[['order_amount', 'total_items']]

    orders_description = order_details_df.describe()
    print(orders_description)

            order_amount  total_items
    count    5000.000000   5000.00000
    mean     3145.128000      8.78720
    std     41282.539349    116.32032
    min        90.000000      1.00000
    25%       163.000000      1.00000
    50%       284.000000      2.00000
    75%       390.000000      3.00000
    max    704000.000000   2000.00000

#####Immediately from the description of the data frame, we can tell that the average order value (AOV) was calculated in the same way as the mean value of order amount, which is `$`3145.13.

#####And we can see that the maximum value of the order amount `$`704,000.00 is abnormally higher than the average. This is most likely the reason why we got such an absurd figure for the AOV. Let's investigate further by looking at orders with out-of-ordinary values (higher than `$`10,000).

In [77]:

    abnormal_df = df[df['order_amount'] > 10000]
    print(abnormal_df.head())

              shop_id  user_id  order_amount  total_items payment_method  \
    order_id
    16             42      607        704000         2000    credit_card
    61             42      607        704000         2000    credit_card
    161            78      990         25725            1    credit_card
    491            78      936         51450            2          debit
    494            78      983         51450            2           cash

                       created_at
    order_id
    16         2017-03-07 4:00:00
    61         2017-03-04 4:00:00
    161        2017-03-12 5:56:57
    491       2017-03-26 17:08:19
    494       2017-03-16 21:39:35

#####From here we can see that high value orders are placed at certain shops. There are 2 of those shops it seems. Let's confirm by counting unique shop ID's.

In [43]:

    print(pd.DataFrame(abnormal_df['shop_id']).apply(pd.Series.nunique))

    shop_id    2
    dtype: int64

#####There are indeed 2 shops where high value orders occur. Though the circumstances of orders placed at each shop don't seem to be similar. Let's look at them separately. First, shop 42.

In [61]:

    shop_42_df = abnormal_df.loc[abnormal_df['shop_id'] == 42].copy()
    shop_42_df['unit_price'] = shop_42_df['order_amount'] / shop_42_df['total_items']
    print(shop_42_df)

              shop_id  user_id  order_amount  total_items payment_method  \
    order_id
    16             42      607        704000         2000    credit_card
    61             42      607        704000         2000    credit_card
    521            42      607        704000         2000    credit_card
    1105           42      607        704000         2000    credit_card
    1363           42      607        704000         2000    credit_card
    1437           42      607        704000         2000    credit_card
    1563           42      607        704000         2000    credit_card
    1603           42      607        704000         2000    credit_card
    2154           42      607        704000         2000    credit_card
    2298           42      607        704000         2000    credit_card
    2836           42      607        704000         2000    credit_card
    2970           42      607        704000         2000    credit_card
    3333           42      607        704000         2000    credit_card
    4057           42      607        704000         2000    credit_card
    4647           42      607        704000         2000    credit_card
    4869           42      607        704000         2000    credit_card
    4883           42      607        704000         2000    credit_card

                      created_at  unit_price
    order_id
    16        2017-03-07 4:00:00       352.0
    61        2017-03-04 4:00:00       352.0
    521       2017-03-02 4:00:00       352.0
    1105      2017-03-24 4:00:00       352.0
    1363      2017-03-15 4:00:00       352.0
    1437      2017-03-11 4:00:00       352.0
    1563      2017-03-19 4:00:00       352.0
    1603      2017-03-17 4:00:00       352.0
    2154      2017-03-12 4:00:00       352.0
    2298      2017-03-07 4:00:00       352.0
    2836      2017-03-28 4:00:00       352.0
    2970      2017-03-28 4:00:00       352.0
    3333      2017-03-24 4:00:00       352.0
    4057      2017-03-28 4:00:00       352.0
    4647      2017-03-02 4:00:00       352.0
    4869      2017-03-22 4:00:00       352.0
    4883      2017-03-25 4:00:00       352.0

#####The abnormal orders placed at shop 42 are rather curious. The were all made by the same user, of the same quantity and at the same time of the day (4:00:00).

#####I believe these orders occur too often to be just erroneous data. The other possibility I could think of is that the user ordered these sneakers in large quantities for resale purposes. And he or she set up an automatic re-ordering for re-stocking.

#####Let us then look at the shop 78.

In [60]:

    shop_78_df = abnormal_df.loc[abnormal_df['shop_id'] == 78].copy()
    shop_78_df['unit_price'] = shop_78_df['order_amount'] / shop_78_df['total_items']
    print(shop_78_df.head())

              shop_id  user_id  order_amount  total_items payment_method  \
    order_id
    161            78      990         25725            1    credit_card
    491            78      936         51450            2          debit
    494            78      983         51450            2           cash
    512            78      967         51450            2           cash
    618            78      760         51450            2           cash

                       created_at  unit_price
    order_id
    161        2017-03-12 5:56:57     25725.0
    491       2017-03-26 17:08:19     25725.0
    494       2017-03-16 21:39:35     25725.0
    512        2017-03-09 7:23:14     25725.0
    618       2017-03-18 11:18:42     25725.0

###_Answers_

#####We can see that the occurrences of high value orders at shop 78 were simply due to the fact that the shop sells an expensive brand of sneaker, priced at \\\$25725.00 a pair.

#####In both cases, the data was legitimate. So I don't think it would be appropriate to simply drop them. Perhaps we should group the data and have separate metrics for difference groups of data. There are several ways to do this:

######1.) Grouping the data by shops. I think this method is feasible in this case because the outliers occur due to the uncommon natures of some shops (sales of high-priced products and offering wholesale service).

In [103]:

    print(round(df.groupby('shop_id')['order_amount'].mean(), 2).head())

    shop_id
    1    308.82
    2    174.33
    3    305.25
    4    258.51
    5    290.31
    Name: order_amount, dtype: float64

######2.) Grouping the data by types of orders, retail orders vs. wholesale orders. For the purpose of classifying the orders, we define orders with quantity less than 10 as retail orders and those with 10 or more as wholesale orders.

In [90]:

    retail_df = df[df['total_items'] < 10]
    wholesale_df = df[df['total_items'] >= 10]


    print('Retail AOV: $' + str(round(retail_df['order_amount'].mean(), 2)))
    print('Wholesale AOV: $' + str(round(wholesale_df['order_amount'].mean(), 2)))

    Retail AOV: $754.09
    Wholesale AOV: $704000.0

######3.) Grouping the data by users. This approach could provide us insights into the total dollar value of each user. This could be helpful for when audience-targeting related desicions need to be made (discriminatory pricing, targeted ads etc.).

In [98]:

    print(round(df.groupby('user_id')['order_amount'].mean(), 2).head())

    user_id
    607    704000.00
    700       299.38
    701       397.08
    702       406.62
    703       380.69
    Name: order_amount, dtype: float64

#####However if we are required to come up with a SINGLE-value metric for the dataset, the first thing comes to mind is median. In the presence of large outliers, median is better than mean but it definitely leaves some information in the dataset unused.

In [114]:

    print('Order amount median: $' + str(round(df['order_amount'].median(), 2)))

    Order amount median: $284.0

#####I don't think mode would be a very appropriate metric in this case. As shown below, "Order amounts" has large standard deviation and min-max difference, which means the value of a single order varies a lot. There are 258 unique values for order amount. If one of them happens to ocur the most often (`$`153.00), it does not represent the overall data that well.

In [119]:

    print(df['order_amount'].describe())
    print()
    print(pd.DataFrame(df['order_amount']).apply(pd.Series.nunique))
    print('\nOrder amount mode: $' + str(round(df['order_amount'].mode()[0], 2)))

    count      5000.000000
    mean       3145.128000
    std       41282.539349
    min          90.000000
    25%         163.000000
    50%         284.000000
    75%         390.000000
    max      704000.000000
    Name: order_amount, dtype: float64

    order_amount    258
    dtype: int64

    Order amount mode: $153

#####In my opinion, each of the mean (AOV), median, and mode provides different insights about the dataset. For example, the mean (AOV) leans towards the higher value side, which means higher value orders represent most of the revenue. And the median combined with mean, tells us that the distribution is skewed to the right, in other words, there are more high value orders than low value ones. I don't think we could pick one of them as THE metric. Instead, we need to consider every one and the combination of different metrics when evaluating the dataset.

---

# Question 2

####a. How many orders were shipped by Speedy Express in total?

```sql
SELECT count(*)
FROM Orders
JOIN Shippers
USING (ShipperID)
WHERE Shippers.ShipperName = 'Speedy Express'
```

#####<i>Answer</i>: 54
<br>

####b. What is the last name of the employee with the most orders?

```sql
SELECT LastName
FROM Employees
WHERE EmployeeID =
(SELECT eID
 FROM
 (SELECT eID, max(numberOfOrders)
  FROM
  (SELECT EmployeeID as eID, count(*) as numberOfOrders
   FROM Orders
   GROUP BY EmployeeID)))
```

#####<i>Answer</i>: Peacock
<br>

####c. What product was ordered the most by customers in Germany?

```sql
SELECT ProductName
FROM Products
WHERE ProductID in
(SELECT ProductID
 FROM
 (SELECT max(numberOfOrders) as maxNumberOfOrders, ProductID
  FROM
  (SELECT ProductID, count(*) as numberOfOrders
   FROM OrderDetails
   WHERE OrderID in
   (SELECT OrderID
    FROM Orders
    WHERE CustomerID in
    (SELECT customerID
     FROM customers
     WHERE country = 'Germany'))
     GROUP BY ProductID)))
```

#####<i>Answer</i>: Gorgonzola Telino

---

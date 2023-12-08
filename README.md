# Python_Ecommerce_Analysis
This repository depicts the detailed analysis of E-commerce Platform using given excel data

# Data Analysis with Python Course Project
***

In this project, you'll be analysing listings data from an e-commerce Platform. 

The dataset is stored in the `data/project_data.xlsx` file. It contains listing information posted on the platform.

One single listing corresponds to one row in the dataset.

The dataset has 12 columns, and 464433 rows. 

Here are the brief descriptions of each column:
- `itemid`: a unique ID of the product
- `shopid`: a unique ID of the shop
- `item_name`: product title  
- `item_description`: detailed  product description
- `item_variation`: stores variations of a product (e.g. different colours or sizes, in the format like {variation 1 name: variation 1 price, variation 2 name: variation 2 price})
- `price`: how much does the item sold
- `stock`: how many stocks left 
- `category`: which category does the product belongs to 
- `cb_option`: 1 indicates the product is sold by a cross border shop
- `is_preferred`: 1 indicates the product is sold by a preferred shop
- `sold_count`: how many products have been sold 
- `item_creation_date`: when are the product uploaded by the seller

### Questions
First step is uploading the necessary file to conduct analysis on.
```{py}
 import pandas as pd
import numpy as np

df = pd.read_excel('C:/Users/Uesr/Downloads/project_data.xlsx',
                   sheet_name= 'listing_data',
                   dtype= {'itemid': str, 'shopid': str,
                           'cb_option': str, 'is_preferred': str})
df.info()
df.head()
```

## 1. How many unique shops are in the dataset?

We use set() function to count unique shopids
```{py}
len(set(df['shopid']))  
```

OR we can use unique() function
```{py} len(df['shopid'].unique())
```

## 2. How many unique preferred and cross border shops are in the dataset?
Boolean indexing
```{py}
len(df[(df.cb_option=='1') & (df.is_preferred=='1')]['shopid'].unique())
```
.loc + np.where
```{py}
print(np.where((df.cb_option=='1') & (df.is_preferred=='1')))
```

```{py} len(df.loc[np.where((df.cb_option=='1') & (df.is_preferred=='1'))]['shopid'].unique())
```
## 3. How many products have zero sold count?
`unique()` function

len(df[(df.sold_count==0)]['itemid'].unique())
`np.where()` + `unique()` function
len(df.loc[np.where(df.sold_count==0)]['itemid'].unique())
## 4. How many products were created in the year 2018?
from datetime import datetime
Using `strftime` we can extract year or date from datetime format data
df['year'] = [i.strftime("%Y") for i in df['item_creation_date']]

df.head(5)
Now we can filter the items created in 2018
len(df[df.year=='2018']['itemid'].unique())
## 5. Show Top 3 Preferred shops’ shopid that have the largest number of unique products
- First > filter out preferred shops
- Second > group them into shopid category
- Third > sort value in descending order
- Lastly > pull out TOP 3 results using index
df[df['is_preferred']=='1'].groupby(['shopid'])['itemid'].count().sort_values(ascending=False)[0:3]
## 6. Show Top 3 Categories that have the largest number of unique cross-border products
- Filter to cross-border products, 'cb_option'== 1
- Groupby them by 'itemid' and count
- Sort value and pull out TOP 3
df[df['cb_option']=='1'].groupby(['category'])['shopid'].count().sort_values(ascending=False)[0:3]
## 7. Find Top 3 shopid with the highest revenue (Assumption: the product price has not been changed.)
- We can add another column named 'revenue' that represents the revenue of each item 
- Sort it in descending order 
df['revenue'] = df['sold_count']*df['price']
df.head()
df.groupby(['shopid'])['revenue'].sum().sort_values(ascending=False)[0:3]
## 8. Find number of products that have more than 3 variations (do not include products with 3 or fewer variations)
- First we create 'variation_count' column to determine how many variation each item has
- Then we filter out the products that have more than 3 variations
- Finally we count the filtered products
df['variation_count'] = [i.count(':') for i in df['item_variation']]
df[['item_variation', 'variation_count']].head(10)


df[df['variation_count']>3]['itemid'].count()
## 9. Identify duplicated listings within each shop (If listing A and B in shops have the exactly same product title, product detailed description, and price, both listing A and B are considered as duplicated listings)
To do so we need to concatenate product title, description and price into one new column. If the new column items have the same value, then the they are the duplicated listings 
df['listing_details'] = df['item_name'].astype(str) + df['item_description'].astype(str) + df['price'].astype(str)

df.head(7)
shopid_listing_count = df.groupby(['listing_details'])['shopid'].count().reset_index()
shopid_listing_count.head()
shopid_duplicates = shopid_listing_count[shopid_listing_count.shopid>1]

shopid_duplicates.head(9)
shopid_duplicates.columns = ['listing_details', 'shopid_count']
shopid_duplicates.head(9)
## 10. Mark those duplicated listings with True otherwise False and store the marking result in a new column named “is_duplicated”
- We have to merge the original dataframe with the new dataframe 'shopid_duplicates'
- Then we can mark the listing that have value >=2 `True`, and `False` others
df_new = pd.merge(df, shopid_duplicates, on = 'listing_details', how='left')
df_new.head(9) 
df_new['is_duplicated'] = [True if not pd.isnull(i) else False for i in df_new['shopid_count']]

df_new.head(9)
df_new.info()
## 11. Find duplicate listings that has less than 2 sold count and store the result in a new excel file named “duplicated_listings.xlsx”
duplicated_listing = df_new[(df_new['is_duplicated']==True) & (df_new['sold_count']<2)]
duplicated_listing.head()
duplicated_listing.info()
duplicated_listing.to_excel('../Python/duplicated_listing.xlsx', index=False)
## 12. Find the preferred shop shopid that have the most number of duplicated listings
df_new[df_new['is_preferred']=='1'].groupby(['shopid'])['is_duplicated'].count().sort_values(ascending=False)[0:10]

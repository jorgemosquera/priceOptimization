# Using Data Analytics for Price Optimization.
When selling a used car, it’s difficult to know where to start. With so many options like trade ins, internet dealers or private, it is hard to decide, especially when the objective is to get as much money as possible for a well-maintained vehicle.

But exactly how much money can I get for my used car? According to Carfax.ca, the key when selling a used car, is to set a competitive and accurate asking price in order to sell it as quickly as possible and for the best amount. If the price is too high, there will be no interest but if the price is too low, I may not get as much as it’s worth.

In order to set a competitive and accurate asking price, the key as always, is to have as much information as possible to make a well-informed decision. So, the goal of this project was given a car with its characteristics, define what will be the optimal asking price in order to sell it as quickly as possible and for the best amount.

## Understanding the Problem
First, it was important to understand the factors that could impact the value of a used car. According to the experts, the value of a used car depends basically in the model year , car kilometers and overall condition. Accident history is also important. Other factors include colour, number of previous owners, service history and after-market features.  

Second, I needed to find sources of data from the used car market. But exactly how much data can I find?
The first place to look for used car data was the local car magazine. In the case of Colombia, the most popular magazine is Revista Motor that weekly prints out a reference table based on the following assumptions:

  - Average asking price of the Bogota vehicles.
  - 20Km per year.
  - The year range goes from 08 to 19.

The second place was online dealer websites. Looking at the most popular website from Colombia, I was able to find different offers according to the model, year and kilometers of my car. The offers also specified detailed information like number of previous owners, colour, and after-market features.

## Data Gathering
Using Python, I created a web scraper in order to get all the data available to my car from this website..

To do this, I created a function download, that download the content of the dealer webpage, a function get_address_list that checks and saves all the pages related to the search and a get_information function that checks all the items and stores them in a data frame.

Following the results of the web scraper:

<img src="/images/getData.png">

## Clean data

Next, it was time to clean the data and to do this, I created the function prepare data, that performs the following tasks:
1. Drop unnecessary columns.
2. Rename columns.
3. Convert attributes from string to numeric.
4. Scale price attribute.
5. Extract version car from the attributes.
6. Filter data by the version required.

```python
def prepareData(df):
    # Rename columns
    df.rename(columns={'Año':'year2',
                   'Marca':'brand',
                   'Modelo':'model',
                   'Versión':'version',
                   'Cilindrada':'engine',
                   'Kilómetros':'kms',
                   'Puertas':'doors',
                   'Tipo de combustible':'gasoline',
                   'Placa':'plate'},
          inplace=True)

    # Drop unnecessary Columns
    df.drop(['url','year','ID','brand','model','engine','doors','gasoline','plate'], axis=1, inplace=True)
    df.rename(columns={'year2':'year'}, inplace=True)

    # Transform kms to numeric
    df.kms = df.kms.str.split(' ').str[0].str.replace('.','').astype(int)

    # Transform price to numeric and scale
    df.price = df.price.str.replace('.','').astype(int)
    df.price = df.price/1000000

    # Filter data by VERSION
    df_version = df[df.name.str.contains(pat = VERSION)].copy()
    df_version.drop(['name','version'], axis=1, inplace=True)

    # Reset Index
    df_version.reset_index(inplace= True, drop = True)

    return df_version    
```

Following the results of the data preparation:

<img src="/images/prepareData.png">

## Data Assessment

Next it was time to explore the data and get insights. In this step my objective was to understand the data, discover patters and fix anomalies. I was able to do this, following these steps:

1. Check the distribution of the attributes. Look for Outliers.

<img src="/images/distributions.png">

The distributions graphs allow us to spot outliers and the shape of our attributes.

Box Plots of Price by Years

<img src="/images/pricePerYear.png">

In this graph we can see how the asking price increase according to the year model. Also the variance inside the groups.

Box Plots of Kms over Year

<img src="/images/kmsPerYear.png">

 In this graph we can see how the kilometers driven decrease with the newer models. Also the variance inside the groups.

 As this is a small dataset, it was very important to handle outliers because they can influence the results. I removed the outliers using IQR score.

 After removing the outliers, we had a consistent dataset.

<p align="center">
 <img src="/images/pairplot.png" width="500">
</p>

 And we can plot the relationship between Price and Kms using colour to identify the year model.

<p align="center">
 <img src="/images/correlation1.png" width="500">
</p>

2. Check Correlations.

Next, I checked the correlations between the attributes. As we only had three attributes, I plotted it in a Heatmap.

<p align="center">
<img src="/images/heatmap.png" width="500">
</p>

Here, we can see a strong positive correlation between price and year and a strong negative correlation between price and kms and year and kms.

We can draw a linear regression curve to verify the distribution.

<p align="center">
<img src="/images/linear.png" width="500">
</p>

It doesn't fit well at values near 0 or greater than 120k. Looks more like an exponential curve. Another way is to plot the residuals:

<p align="center">
<img src="/images/residuals.png" width="500">
</p>

To fix this, we can take the log of price.

<p align="center">
<img src="/images/correlation_log.png" width="500">
</p>

And we can see the correlations with the new attribute price_log:

<p align="center">
<img src="/images/heatmap2.png" width="500">
</p>

Finally, we can plot a regression line to se its fit:

<p align="center">
<img src="/images/linear_log.png" width="500">
</p>

And the new residuals:

<p align="center">
<img src="/images/residuals_log.png" width="500">
</p>

We can finish our transformation creating a function that:
1. Remove outliers using IQR score.
2. Creating a new feature that is the log of the Price.

```python
def transformData(df):
    # Remove Outliers using IQR Score
    Q1 = df.quantile(0.25)
    Q3 = df.quantile(0.75)
    IQR = Q3 - Q1
    df_iqr = df[~((df < (Q1 - 1.5 * IQR)) |(df > (Q3 + 1.5 * IQR))).any(axis=1)]

    #Add logPrice column
    df_iqr = df_iqr.assign(price_log = lambda x: np.log(x.price))

    return df_iqr
```

## Regressions

I was ready to make regression calculations on the clean and transform dataset. I wanted to test two Regression: Linear regression, using categorical data year model and a non-linear regression that will only use kms and price.

### Linear Multiple Regression

I will use LinearRegression from Sklearn to obtain the line that best fits our data.

```python
from sklearn.linear_model import LinearRegression

# Create linear regression object.
mlr= LinearRegression()

# Fit linear regression.
mlr.fit(df[['kms','year']], df['price_log'])

# Get the slope and intercept of the line best fit.
print(mlr.intercept_)
# -158.192497671487

print(mlr.coef_)
# [-2.02620015e-06  8.06647554e-02]
```

Now as we saw in the Data assessment, the data can have outliers. And this is normal, basically a seller can put the price that he wants. Also we can have cars with a lot of kilometers. Because of this is better to use mean absolute error as our performance measure.

```python
from sklearn.metrics import mean_absolute_error
estimated_prices = np.exp(mlr.predict(df[['kms','year']]))
lin_mae = mean_absolute_error(df['price'], estimated_prices)

lin_mae
#3.7660973862461753
```

With this I was ready to make some predictions:

```python
# Predictions using scikit learn.
KMS = 40000
YEAR_MODEL = 2009

print(np.exp(mlr.predict([[KMS,YEAR_MODEL]])))
#43.9014732
```

### Non-Linear Regression

I also wanted to test a non-linear regression model. To do this, I will used only kms and price features.

```python
df = df.sort_values('kms')
X = df.iloc[:,2:3]
y = df.iloc[:,3]

# Fitting Polynomial Regression to the dataset
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree = 3)
X_poly = poly.fit_transform(X)

poly.fit(X_poly, y)
lin2 = LinearRegression()
lin2.fit(X_poly, y)
```

The resulting line of the non-linear regression:

<p align="center">
<img src="/images/plr2.png" width="500">
</p>

The mean absolute error:

```python
from sklearn.metrics import mean_absolute_error
estimated_prices = np.exp(lin2.predict(poly.fit_transform(X)))
lin_mae = mean_absolute_error(df['price'], estimated_prices)
lin_mae
#8.161210767006
```

We can see the difference of the predictions using both models in the data:

<p align="center">
<img src="/images/results.png" width="500">
</p>

I believed the linear regression captures in a better way the mix of the year model and kms attributes to estimate a price.

## Conclusions
Data analytics is a powerful tool to make well-informed decisions. Using the process of setting a problem, understanding the background, gather, clean and analyze data, it is easy to draw well-informed decisions.

Using these tools I was able to:

1. Gather data from an external source using a webscraper.
2. Clean the data and extract attributes.
3. Understand the data.
4. Transform data.
5. Create two models to predict prices.
6. Get Results.

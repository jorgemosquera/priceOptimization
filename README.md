# Using Data Analytics for Price Optimization.
When selling a used car, it’s difficult to know where to start. With so many options like trade ins, internet dealers or private, it is hard to decide, especially when the objective is to get as much money as possible for a well-maintained vehicle.

But exactly how much money can I get for my used car? According to Carfax.ca, the key when selling a used car, is to set a competitive and accurate asking price in order to sell it as quickly as possible and for the best amount. If the price is too high, there will be no interest but if the price is too low, I may not get as much as it’s worth.

In order to set a competitive and accurate asking price, the key as always, is to have as much information as possible to make a well-informed decision. So, the goal of this project is given a car with its characteristics, define what will be the optimal asking price in order to sell it as quickly as possible and for the best amount.

## Understanding the Problem
First, it was important to understand the factors that could impact the value of a used car. According to the experts, the value of a used car depends basically in the model year , car kilometers and overall condition. Accident history is also important. Other factors include colour, number of previous owners, service history and after-market features.  

Second, I needed to find sources of data from the used car market. But exactly how much data can I find?
The first place to look for used car data is the local car magazine. In the case of Colombia, the most popular magazine is Revista Motor that weekly prints out a reference table based on the following assumptions:

  - Average asking price of the Bogota vehicles.
  - 20Km per year
  - The year range goes from 08 to 19.

The data is available in a pdf file that I will need to scrape according to the car model and year.

The second place to look for used car data is at online dealer websites. Looking at the most popular, I was able to find different offers according to the model, year and kilometers of my car. The offers also specified detailed information like number of previous owners, colour, and after-market features.

## Data Gathering
Using Python, I created a web scraper in order to get all the data available to my car.

To do this, I created a function download, that download the content of the dealer webpage, a function get_address_list that checks and saves all the pages related to the search and a get_information function that checks all the items and stores them in a data frame.

Following the results of the web scraper.

<img src="/images/getData.png">

## Clean data

Now, it is time to clean our data and I will do this with the function prepare data, that basically will do the following tasks:
1. Drop unnecessary columns.
2. Rename columns.
3. Transform attributes from string to numeric.
4. Scale price attribute.
5. Extract version car from the attributes.
6. Filter data by the version required.

We can see the results of the data preparation in the following image:

<img src="/images/prepareData.png">

## Data Assessment

Now it was time to explore our data and get insights of what the data can provide us. In this step my objective was to understand the data, discover patters and fix anomalies.

1. Check the distribution of the attributes. Look for Outliers.

Distributions of Attributes

<img src="/images/distributions.png">

The distributions graphs allow us to spot outliers and the shape of our attributes.

Box Plots of Price over Years

<img src="/images/pricePerYear.png">

In this graph we can see how the price increase according to the year model. Also the variance inside the groups.

Box Plots of Kms over Year

<img src="/images/kmsPerYear.png">

 In this graph we can see how the kilometers driven decrease with the newer models. Also the variance inside the groups.

 So, as we have few points of data that can be easily influence by outlier, we will remove outliers based on IQR score.

 After removing the outliers, we have a consistent dataset

 <img src="/images/pairplot.png">

 And we can plot the relationship between Price and Kms identifying years.

 <img src="/images/correlation1.png">

2. Check Correlations.

Next we can check the correlations between the attributes. As we only have three attributes, we can plot them in a Heatmap

<img src="/images/heatmap.png">

We see a strong positive correlation between price and year and a strong negative correlation between price and kms and year and kms.

We can draw a linear regression curve to verify the distribution.

<img src="/images/linear.png">

It doesn't fit well at values near 0 or greater than 120k. Looks more like an exponential curve.

<img src="/images/residuals.png">

To fix this, we can take the log of price.

<img src="/images/correlation_log.png">

And we can see the correlations with the new attribute price_log

<img src="/images/heatmap2.png">

Finally, we can plot a regression line to se its fit.

<img src="/images/linear_log.png">

And the new residuals

<img src="/images/residuals_log.png">

One important thing to realize is that we didn't have enough data for a Machine Learning Algorithm.


## Conclusions
Data analytics is a powerful tool to make well-informed decisions. Using the process of setting a problem, understanding the background, gather, clean and analyze data, it is easy to draw well-informed decisions.

# Data-visuals
from kaggle; unit 2 lesson 4 project 4

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline

fed = pd.read_csv('kaggle.csv')


# First 3 columns broken down into Year, Month, Day
# Combine into one column, Date, to use as index
fed['Date'] = fed[fed.columns[:3]].apply(lambda x: '-'.join(x.dropna().astype(int).astype(str)), axis=1)

# Changing type of fed['Date'] to_datetime()
fed['Date'] = pd.to_datetime(fed['Date'])

# Setting date as index
fed = fed.set_index('Date')


# Changing nan, mainly through interpolate or ffill

# Creating column 'gdp' that takes 'Real GDP' column with ffill
fed['gdp'] = fed['Real GDP (Percent Change)'].fillna(method='pad')

# Creating column 'fedrate' with interpolate 
fed['fedrate'] = fed['Effective Federal Funds Rate'].interpolate()

# Unemployment using interpolate
fed['unemployment'] = fed['Unemployment Rate'].interpolate()

# Inflation; first interpolate
fed['inflation'] = fed['Inflation Rate'].interpolate()
# then, change the chunk of nan at the beginning of the dataframe to 0
fed.loc['1954-7-1':'1957-12-1', ('inflation')] = 0


# New df with updated values for nan
fed2 = fed[['fedrate', 'gdp', 'unemployment', 'inflation']]



plt.figure(figsize=(15, 10))
bins = np.linspace(-10, 20, 100)
for a in fed2:
    plt.hist(fed2[a], bins, alpha=0.3, label=str(a))
    
plt.title('Too many variables')
plt.xlabel('Percentage (%)')
plt.legend()

fed2.hist(layout=(2, 2), sharex=True, sharey=True, bins=50)

plt.show()

The first histogram has all of the variables stacked on top of another, and though it is a good idea to do so for comparison, occasionally the visuals end up looking ambiguous or just plain confusing.
I was better served with creating separate histograms.  What struck out was the high count of __0__ in fedrate.  Going into the data, it turns out that there is not even a single count of _0_ in fedrate, and that what appears to be _0_ is just an extremely small decimal number.  Changing the number of bins is able to portray the amount of unique numbers in that cluster.



plt.figure(figsize=(15, 10))
bins = np.linspace(-10, 20, 200)
for a in fed2:
    plt.hist(fed2[a], bins, alpha=0.3, label=str(a))
plt.title('Too many variables')
plt.xlabel('Percentage (%)')
plt.legend()
fed2.hist(layout=(2, 2), sharex=True, sharey=True, bins=100)
plt.show()

Furthermore, _gdp_ has what appears to be the most normally distributed dataset amongst the variables.  This may be a strong enough argument to base any hypothesis around this variable.



plt.figure(figsize=(22, 13))  
for a in fed2:
    plt.plot(fed2[a])
plt.title('Line Chart', fontsize='xx-large') 
plt.ylabel(
    'Percentage (%)', 
    fontsize='xx-large', 
    horizontalalignment='right', 
    rotation='horizontal'
)
plt.legend(loc='upper right', fontsize='xx-large')

plt.figure(figsize=(12, 7))

x=0
for a in fed2:
    x += 1
    plt.subplot(2, 2, x)
    plt.plot(fed2[a])
    plt.title(str(a).capitalize(), fontsize='small')
    plt.ylabel('Percentage (%)')

plt.show()

A basic line plot is also a good starting point in extracting information from datasets.  My immediate response is to check for correlation.  A scatter plot might be better served, but a line plot can give you a good sense.
Look for abnormalities.  To me, there are 2 or 3 sections of the graph that are particularly abnormal: just before the midpoint where there is a giant spike/dip, and towards the end where the variables start to diverge somewhat.
- The past is a good indicator of the future, so it is important to study the past when it comes to the economy.  At the same time, times change.  If it were to easy to use the past to predict the future, we'd all be rich.  However, due to the housing market crash, there has definitely been some foreign influences affecting the data and is worth investigating.



def scatter_plot3(series, xaxis=fed2['fedrate']):
    # Scatter plot with fedrate as axis
    # series should contain list of series
    # Incomplete, trying to extract name of columns for title, etc
    colors = ['red', 'cyan', 'blue', 'green']
    i = 0
    for a in series:
        j = i + 1
        try:
            plt.subplot(2, 2, j)
            plt.scatter(
                x=xaxis,
                y=series[a],
                color=colors[i],
                alpha=0.7,
                marker='x',
                s=5
            )
            plt.title(str(a).capitalize())
            i+=1
        except ValueError:
            pass
    
plt.figure(figsize=(12, 7))
scatter_plot3(fed2)

plt.figure(figsize=(12, 7))
scatter_plot3(fed2, xaxis=fed2['gdp'])

plt.figure(figsize=(12, 7))
scatter_plot3(fed2, xaxis=fed2['unemployment'])

plt.figure(figsize=(12,7))
scatter_plot3(fed2, fed2['inflation'])



plt.figure(figsize=(12, 7))

x=0
for a in fed2:
    x += 1
    plt.subplot(2, 2, x)
    plt.boxplot(fed2[a])
    plt.title(str(a).capitalize(), fontsize='small')

plt.show()

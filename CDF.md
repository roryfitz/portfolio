# Using a normed cumulative density function (CDF) to analyze data distribution:

```python
# plot CDF
df.plot(y='rt', kind='hist', cumulative=True, density=True)

# add lines at the median, 25th percentile, and 75th percentile
plt.axvline(df['rt'].describe()['25%'], 0, 1, color='turquoise', linestyle='--')
plt.axvline(df['rt'].median(), 0, 1, color='cyan', linestyle='-')
plt.axvline(df['rt'].describe()['75%'], 0, 1, color='turquoise', linestyle='--')

# show plot
plt.show()
```




    
![png](Assignment_3_files/Assignment_3_78_0.png)

## Data Analysis

The CDF shows that 60% of reaction times were less than the median, and 40% were above the median. Further, 80% were in the first three quartiles, and only 20% were in the fourth and final quartile. From this we can see that the data are skewed to the right.


```python

```

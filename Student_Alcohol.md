# Student Alcohol Consumption: Math vs Portugese
Here I analyzed the difference in average alcohol consumption between math and portugese students

```python
# read in the data

math = pd.read_csv('math_student_alcohol.csv')
por = pd.read_csv('por_student_alcohol.csv')
```

#### I was only interested in total alcohol consumption for this test, so I combined the weekday ('Dalc') and weekend ('Walc') consumption values for both groups

```python
# combine weekday and weekend consumption

math['cons'] = math['Dalc'] + math['Walc']
por['cons'] = por['Dalc'] + por['Walc']
```

#### I performed a t test to determine if there is a significant difference in student alcohol consumption between math and portugese students

```python
# calculate means

math_avg = math['cons'].mean()
por_avg = por['cons'].mean()

print('math average = ' + str(math_avg) + ', portugese average = ' + str(por_avg))
```

math average = 3.7721518987341773, portugese average = 3.782742681047766



```python
# calculate standard deviations

math_std = math['cons'].std()
por_std = por['cons'].std()

print('math standard dev = ' + str(math_std) + ', portugese standard dev = ' + str(por_std))
```

math standard dev = 1.9843893758892996, portugese standard dev = 1.9924110332337173



```python
# calculate n values

n_math = len(math['cons'])
n_por = len(por['cons'])

print('math n value = ' + str(n_math) + ', portugese n value = ' + str(n_por))
```

math n value = 395, portugese n value = 649



```python
# calculate standard errors

math_se = math_std/mat.sqrt(n_math)
por_se = por_std/mat.sqrt(n_por)

print('math standard err = ' + str(math_se) + ', portugese standard err = ' + str(por_se))
```

math standard err = 0.09984546534383723, portugese standard err = 0.0782089741948411



```python
# calculate standard error on the difference between math and portugese samples

sed = mat.sqrt(math_se**2 + por_se**2)

print('standard error between the samples = ' + str(sed))
```

standard error between the samples = 0.12682965187343503



```python
# calculate the t statistic

t_statistic = (por_avg - math_avg) / sed

print('t statistic = ' + str(t_statistic))
```

t statistic = 0.08350399261646918



```python
# calculate degrees of freedom

df = n_math + n_por - 2

print('degrees of freedom = ' + str(df))
```

degrees of freedom = 1042



```python
# calculate critical value

alpha = 0.05
critval = norm.ppf(1.0 - alpha, df)

print('critical value = ' + str(critval))
```

critical value = 1043.6448536269515



```python
# calculate p value

pval = (1 - norm.cdf(abs(t_statistic), df)) * 2

print('p value = ' + str(pval))
```

p value = 2.0

#### At the end, I printed all important values that I would need for my data analysis

```python
print('t = ' + str(t_statistic) + ', df = ' + str(df) + ', cv = ' + str(critval) + ', p = ' + str(pval))
```

t = 0.08350399261646918, df = 1042, cv = 1043.6448536269515, p = 2.0

Note: code written in python

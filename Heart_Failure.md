# Heart Failure
Here I compared the creatinine phoshpokinase levels in female and male smokers and nonsmokers

```
# read in the data
hf = pd.read_csv('heart_failure.csv')
```

I converted the binary and boolean values of the sex and smoking conditions to strings in order to make the resulting table more readable
```
dict1 = {0:'female', 1:'male'}
hf['sex'] = hf['sex'].replace(dict1)
dict2 = {0:'no', 1:'yes'}
hf['smoking'] = hf['smoking'].replace(dict2)
```

### Determining effect of sex and smoking on creatinine phosphokinase levels

I created a table to show the mean creatinine levels for four conditions: female non-smoker, female smoker, male non-smoker, and male smoker
```
print('Table 1: Mean Creatinine for Sex and Smoking Conditions')
hf.groupby(['sex', 'smoking'])[['creatinine_phosphokinase']].mean().rename(columns={'creatinine_phosphokinase':'mean creatinine'})
```

Table 1: Mean Creatinine for Sex and Smoking Conditions

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>mean creatinine</th>
    </tr>
    <tr>
      <th>sex</th>
      <th>smoking</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">female</th>
      <th>no</th>
      <td>487.702970</td>
    </tr>
    <tr>
      <th>yes</th>
      <td>201.000000</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">male</th>
      <th>no</th>
      <td>671.843137</td>
    </tr>
    <tr>
      <th>yes</th>
      <td>601.956522</td>
    </tr>
  </tbody>
</table>
</div>



I then created a second table that displayed the standard deviations that corresponded with the means from the first table
```
print('Table 2: Standard Deviations for Table 1 Means')
hf.groupby(['sex', 'smoking'])[['creatinine_phosphokinase']].std().rename(columns={'creatinine_phosphokinase':'standard deviation'})
```

Table 2: Standard Deviations for Table 1 Means

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>standard deviation</th>
    </tr>
    <tr>
      <th>sex</th>
      <th>smoking</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">female</th>
      <th>no</th>
      <td>620.628658</td>
    </tr>
    <tr>
      <th>yes</th>
      <td>111.684078</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">male</th>
      <th>no</th>
      <td>1187.625301</td>
    </tr>
    <tr>
      <th>yes</th>
      <td>1033.529735</td>
    </tr>
  </tbody>
</table>
</div>



# Heart Failure


```
# read in the data
hf = pd.read_csv('heart_failure.csv')
```


```
dict1 = {0:'female', 1:'male'}
hf['sex'] = hf['sex'].replace(dict1)
dict2 = {0:'no', 1:'yes'}
hf['smoking'] = hf['smoking'].replace(dict2)
```

### Determining effect of sex and smoking on creatinine phosphokinase levels


```
print('Table 1: Mean Creatinine for Sex and Smoking Conditions')
hf.groupby(['sex', 'smoking'])[['creatinine_phosphokinase']].mean().rename(columns={'creatinine_phosphokinase':'mean creatinine'})
```

    Table 1: Mean Creatinine for Sex and Smoking Conditions





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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




```
print('Table 2: Standard Deviations for Table 1 Means')
hf.groupby(['sex', 'smoking'])[['creatinine_phosphokinase']].std().rename(columns={'creatinine_phosphokinase':'standard deviation'})
```

    Table 2: Standard Deviations for Table 1 Means





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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



# Determining Accuracy of Mean and Standard Deviation Values from Reaction Time Data

First I read in the data.

```python
files = pd.DataFrame(glob('spid[31-51]*'))
files = files + '/' + files + '_data.txt'
files = files.stack().tolist()

data = []

for name in files:
    x = pd.read_csv(name, sep='\t')
    data.append(x)

all_data = pd.concat(data)
```
I then removed missing data and the practice trials.

```python
all_data = all_data.dropna(how='any')

all_data = all_data[all_data.block != 'practice']
```

I then added the condition information to the dataframe.

```python
# first we create a set of conditions — the different possible mappings
conditions = [
    (all_data['mapping'] == 0) & (all_data['target'] == 'black') & (all_data['targetLocation'] == 'left'),
    (all_data['mapping'] == 0) & (all_data['target'] == 'black') & (all_data['targetLocation'] == 'right'),
    (all_data['mapping'] == 0) & (all_data['target'] == 'white') & (all_data['targetLocation'] == 'left'),
    (all_data['mapping'] == 0) & (all_data['target'] == 'white') & (all_data['targetLocation'] == 'right'),
    (all_data['mapping'] == 1) & (all_data['target'] == 'black') & (all_data['targetLocation'] == 'left'),
    (all_data['mapping'] == 1) & (all_data['target'] == 'black') & (all_data['targetLocation'] == 'right'),
    (all_data['mapping'] == 1) & (all_data['target'] == 'white') & (all_data['targetLocation'] == 'left'),
    (all_data['mapping'] == 1) & (all_data['target'] == 'white') & (all_data['targetLocation'] == 'right')
    ]

# create mappings that align with the order of the entries in the conditions list above
mapping = ['incongruent', 
           'congruent', 
           'congruent', 
           'incongruent', 
           'congruent', 
           'incongruent', 
           'incongruent', 
           'congruent'
          ]

# create a new column called "simon" based on the conditions and mapping lists above
all_data['simon'] = np.select(conditions, mapping, default='none')
```

I then converted the reaction times into milliseconds.

```python
#Put the reaction time into ms to better display the data:

all_data.rt = all_data.rt * 1000
all_data.head()
```

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
      <th>id</th>
      <th>year</th>
      <th>month</th>
      <th>day</th>
      <th>hour</th>
      <th>minute</th>
      <th>mapping</th>
      <th>messageViewingTime</th>
      <th>block</th>
      <th>trialNum</th>
      <th>targetLocation</th>
      <th>target</th>
      <th>flankers</th>
      <th>rt</th>
      <th>response</th>
      <th>error</th>
      <th>anticipation</th>
      <th>feedbackResponse</th>
      <th>targetOnError</th>
      <th>simon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>144</th>
      <td>spid39</td>
      <td>2015.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.407595</td>
      <td>1</td>
      <td>1.0</td>
      <td>right</td>
      <td>white</td>
      <td>congruent</td>
      <td>687.385241</td>
      <td>white</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>0.069286</td>
      <td>incongruent</td>
    </tr>
    <tr>
      <th>145</th>
      <td>spid39</td>
      <td>2015.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.407595</td>
      <td>1</td>
      <td>1.0</td>
      <td>right</td>
      <td>white</td>
      <td>congruent</td>
      <td>687.385241</td>
      <td>white</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>0.069286</td>
      <td>incongruent</td>
    </tr>
    <tr>
      <th>146</th>
      <td>spid39</td>
      <td>2015.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.407595</td>
      <td>1</td>
      <td>1.0</td>
      <td>right</td>
      <td>white</td>
      <td>congruent</td>
      <td>687.385241</td>
      <td>white</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>0.069286</td>
      <td>incongruent</td>
    </tr>
    <tr>
      <th>147</th>
      <td>spid39</td>
      <td>2015.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.407595</td>
      <td>1</td>
      <td>2.0</td>
      <td>left</td>
      <td>white</td>
      <td>congruent</td>
      <td>345.519429</td>
      <td>white</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>0.068559</td>
      <td>congruent</td>
    </tr>
    <tr>
      <th>148</th>
      <td>spid39</td>
      <td>2015.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.407595</td>
      <td>1</td>
      <td>2.0</td>
      <td>left</td>
      <td>white</td>
      <td>congruent</td>
      <td>345.519429</td>
      <td>white</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>0.068559</td>
      <td>congruent</td>
    </tr>
  </tbody>
</table>
</div> 

I then calculated the mean reaction times for each condition
```python
#Mean (RT):

print('Table 1: Mean RTs for Each Condition')
all_data.groupby(['flankers', 'simon'])[['rt']].mean().rename(columns={'rt': 'mean_rt(ms)'})
```

    Table 1: Mean RTs for Each Condition


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
      <th>mean_rt(ms)</th>
    </tr>
    <tr>
      <th>flankers</th>
      <th>simon</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">congruent</th>
      <th>congruent</th>
      <td>403.236213</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>424.046921</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">incongruent</th>
      <th>congruent</th>
      <td>465.160221</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>463.644580</td>
    </tr>
  </tbody>
</table>
</div>

I then calculated the standard deviations for each mean.

```python
#Standard Deviation (RT):

print('Table 2: Standard Deviations for Each Condition')
all_data.groupby(['flankers', 'simon'])[['rt']].std().rename(columns={'rt':'standard_deviation'})
```

    Table 2: Standard Deviations for Each Condition

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
      <th>standard_deviation</th>
    </tr>
    <tr>
      <th>flankers</th>
      <th>simon</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">congruent</th>
      <th>congruent</th>
      <td>98.817574</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>90.166940</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">incongruent</th>
      <th>congruent</th>
      <td>115.213379</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>96.629932</td>
    </tr>
  </tbody>
</table>
</div>


Finally, I calculated the accuracy for each condition.

```python
#Accuracy (Error):

print('Table 3: Accuracy for Each Condition')
(((all_data.groupby(['flankers', 'simon'])[['error']].count()) - (all_data.groupby(['flankers', 'simon'])[['error']].sum()))/(all_data.groupby(['flankers', 'simon'])[['error']].count())).rename(columns={'error': 'accuracy_rate'})
```

    Table 3: Accuracy for Each Condition

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
      <th>accuracy_rate</th>
    </tr>
    <tr>
      <th>flankers</th>
      <th>simon</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">congruent</th>
      <th>congruent</th>
      <td>0.967218</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>0.941672</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">incongruent</th>
      <th>congruent</th>
      <td>0.879961</td>
    </tr>
    <tr>
      <th>incongruent</th>
      <td>0.888889</td>
    </tr>
  </tbody>
</table>
</div>

Note: code written in python

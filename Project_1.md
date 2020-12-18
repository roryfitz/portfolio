# Project 1 Submission
## NESC 3505

This Jupyter notebook file is where you should put your final submission. You may want to work in one or more other notebook files as you're working on the assignment. Having a few on the go would allow different people to work on different parts of the assignment, although naturally there is some logical progression to what has to be done. You're also welcome to work directly in this file if you want, although it might be safer to make a backup copy first, so you have a "clean" version if you need it later.

The critical thing is, this file is linked to the CoCalc assignment for project 1, so this file is the only one out of your shared project that we will be able to collect and grade.

Below we've provide a few Markdown cells with headings to help you structure your submission. 

---

# **CODEZI**
![](images/codezi.png)

## Team Members

List the names of your team's members below.

Justine David,
Emily Gosselin,
Caroline Damus,
Nour Mohamed,
Unity Cooper,
Andrew Dixon

## Contributions

List each team member's contribution to the project. It's fine if multiple people all contributed to a particular aspect of the project. 

Importing and Cleaning Data: Emily Gosselin and Caroline Damus<br>
Derive and Encode Simon Condition: Nour Mohamed and Unity Cooper<br>
Exploratory Data Analysis: Justine David<br>
Interpretations and Conclusion: Justine David and Emily Gosselin<br>
Review: Andrew Dixon

## Packages you may need


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from glob import glob
```

## Read in the data 

In this assignment you'll get experience starting with the raw data and extracting the necessary information yourself. Read in the data file for each participant. Ultimately you will want to combine them into one big pandas DataFrame. Call that DataFrame `df`

The naming convention for this data set was `[subjectID]_[date]_[time]`, with the date represented as YYYY_MM_DD and the time (when the experiment was started) as HH_MM.

Each subject's data file is in a sub-directory with that subject's name and the date they were run. These are listed in the `subjects` list below for you. The sub-directory contains three files, all named with the naming convention above, but with different extensions (remember, extensions are the bit after the period in a filename, that indicates what type of file it is).  The three files are:
- `[ID]_code.py` - the python file that ran the experiment - i.e., presented the stimuli and collected responses
- `[ID]_data.txt` - the data file you will want to open
- `[ID]_trigger.txt` - an additional data file not of interest to us

We've included the "extra" files and complicated directory structure because this is what real study data typically looks like. So, it's good experience for you to see this, and learn how to read in just the specific file you need. 

<font color='red'> Notes: <br>
    - `[31-51]`>`[31-51]` not scalable<br> 
    - could simplify with glob: `files = glob('spid*/*_data.txt')`</font>


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

## Clean the data

The next things we need to do are:
- decide how to deal with the `NaN`s
- remove the redundant lines

In terms of the missing data, it turns out these are just trials on which the participant didn't make a button press response. We have (at least) three options:
1. Leave these as-is. 
2. Remove all rows with missing data
3. Replace (impute) the missing values with actual values. 

Option 3 was covered in the DataCamp lessons. Imputation of missing data is sometimes done in psychology and neuroscience studies, especially if we have lots of variables, and only one data point per subject (e.g., a score on a standardized test completed by each subject). Usually the reason for imputing data is that statistical methods such as ANOVAs do not allow for missing data, so without imputation we might have to discard a subject's data, even if they are only missing data from one test among many that were administered. 

However, in the current case, it doesn't make sense to "make up" a reaction time on a trial when a participant didn't make a response at all. So we could remove those trials entirely. On the other hand, we might want to report how many trials, on average, our participants failed to respond to, or we might want to treat those the same as errors. 

Although missing data is problematic for things like ANOVAs, it is not an issue for EDA summary statistics in pandas. Indeed, [pandas' documentation explicitly states](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html#calculations-with-missing-data) that missing data is ignored in computing values such as the mean and standard deviation.  

So, we could probably safely keep the NaNs in the data. On the other hand, you learned in *Manipulating DataFrames with pandas* how to use the `.dropna()` method. And as they say, "when you have a hammer, everything looks like a nail". So let's drop the any rows containing any NaN values:

<font color='green'>Reassigned to itself. Nice.</font><br>
<font color='red'></font>


```python
all_data = all_data.dropna(how='any')
```

## Remove Practice Trials

<font color='red'>This presupposes that "practice" is always spelled correctly. You can check by calling `.unique()` on the `block` column first.</font><br>


```python
all_data = all_data[all_data.block != 'practice']
```

<font color='red'>didn't drop duplicates or remove outliers</font>

## Derive and encode Simon condition

One confusing thing about the way these data are stored is that, while we have one column clearly indicating whether our flankers were congruent or not (the intuitively-named `flankers` column), there is no column that uniquely and clearly tells us whether each trial was congruent or not for the Simon task component. This is encoded across three columns: `target`, `targetLocation`, and `mapping`. That is, once we know the mapping between response hand and colour, AND which side the stimulus was presented on, we will know whether a value of "black" in the `target` column is Simon-congruent or Simon-incongruent. 

| mapping | left button  | right button | 
| ------- | ----- | ----- |
| 0       | white | black |
| 1       | black | white |

So you need to flex those Python muscles and create a new column in the DataFrame, called `simon`, which encodes whether each trial was Simon-congruent or Simon-incongruent (note I'm explicitly saying "Simon-congruent or Simon-incongruent" here to try to avoid confusion with flanker-congruent and flanker-incongruent). 

This is more advanced than what has been covered in your lessons so far, but it's a common and useful thing to know. So, I've provided most of the code for you, but left bits for you to fill in, which will hopefully help you to understand what you're doing. 

We use `np.select` to do this; I suggest you [read the API](https://numpy.org/doc/stable/reference/generated/numpy.select.html?highlight=select#numpy.select) to help you understand what's going on. 

Below, fill in the missing bits after the `==` symbols on all the lines where information is missing. I've done the first three lines for you. Note as well how I'm systematically changing the three variables (`mapping`, `target`, and `targetLocation`) — the rightmost one (`targetLocation`) changes fastest, as we go down the list (`left`-`right`-`left` etc), the middle one changes next most quickly (`black`-`black`-`white` etc) and the leftmost one changes slowest. It's always good to follow systematic patterns like this; it makes for more intuitive, readable code, and also means you're less likely to make errors or forget to include a condition.


```python
# first we create a set of conditions — in our cases, the different possible mappings
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

# The mappings below align with the order of the entries in the conditions list above
mapping = ['incongruent', 
           'congruent', 
           'congruent', 
           'incongruent', 
           'congruent', 
           'incongruent', 
           'incongruent', 
           'congruent'
          ]

# This line creates a new column called "simon" based on the conditions and mapping lists above
all_data['simon'] = np.select(conditions, mapping, default='none')
```

## Exploratory Data Analysis

This is the heart of the project. Derive and display your data in tables and graphics that address the hypotheses listed in the *Instructions* document.


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



<font color='green'> Great `.groupby()` syntax, and renaming the columns in the same line is both elegant and helpful. </font><br>
<font color='orange'> Round ms--two decimal places is common </font><br>
<font color='red'>Means and stds are slightly off due to outliers.</font><br> 
<font color='red'>Means and stds in the same table would be much easier to read -- this can readily be done with the `.agg()` multi-aggregate method.<br>
Instructions indicate RT calculations are to focus on error-free trials only.</font><br> 


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



<font color='green'> Elegant use of `.groupby()` to generate the plots. </font><br>
<font color='orange'> Plots could be improved by colour or some other differentiation. While using the pandas `.plot` method is easy, as noted in a past assignment, you gain much power and flexibility in using matplotlib or seaborn instead. This could allow you to label the plots individually, more control over colours and appearance, etc. </font><br>
<font color='red'> Plot titles on the plots themselves would be better than having to scroll.<br>
Side-by-side boxplots would be easier to compare. As it is, you have not made it at all easy for your audience to make visual comparisons between conditions.<br>
Note the outliers on the plots.</font>


```python
#Graphs:

print('Order of Box Plots:')
print('1. Congruent-Congruent')
print('2. Congruent-Incongruent')
print('3. Incongruent-Congruent')
print('4. Incongruent-Incongruent')

all_data.groupby(['flankers', 'simon'])[['rt']].plot(kind='box', ylabel='rt (ms)', title='Reaction Time for Corresponding Condition')
plt.show()
```

    Order of Box Plots:
    1. Congruent-Congruent
    2. Congruent-Incongruent
    3. Incongruent-Congruent
    4. Incongruent-Incongruent





    
![png](graded-Project_1_Submission%20%28Codezi%29_files/graded-Project_1_Submission%20%28Codezi%29_28_1.png)
    






    
![png](graded-Project_1_Submission%20%28Codezi%29_files/graded-Project_1_Submission%20%28Codezi%29_28_2.png)
    






    
![png](graded-Project_1_Submission%20%28Codezi%29_files/graded-Project_1_Submission%20%28Codezi%29_28_3.png)
    






    
![png](graded-Project_1_Submission%20%28Codezi%29_files/graded-Project_1_Submission%20%28Codezi%29_28_4.png)
    



## Interpretations and Conclusion

Provide an interpretation of the data, with reference to how the EDA results support or do not support the hypotheses, and why.




<font color='red'> Helpful to include numerical results (e.g., actual means, stdevs, accuracy) within interpretation write-ups. The reader should not have to scroll up and down to link them.<br></font>

Based on the EDA that we completed, some of the hypotheses were supported, but others were not supported. <font color='red'>This kind of "throwaway" sentence should be avoided. It tells us nothing specific. </font>

The hypothesis stating simon-congruent reaction times would be faster than simon-incongruent reaction times in the flanker compatible condition is supported by our findings. This can be seen in Table 1 above, where the mean reaction time for the congruent-congruent condition is shorter than the congruent-incongruent condition. The related hypothesis stating the simon-congruent accuracy rate would be higher than the simon-incongruent accuracy rate in the flanker compatible condition is also supported by our findings. In Table 3 above, it can be seen that the accuracy rate for congruent-congruent is higher than congruent-incongruent, however only by a small margin.

The hypothesis stating simon-congruent reaction times would be slower than simon-incongruent reaction times in the flanker incompatible condition is supported by our findings. However, as seen in Table 1, the difference is quite small, about 2ms. The variance, shown in Table 2, for the incongruent-congruent condition is relatively larger than that of the other conditions, so although our results were slightly in-line with the hypothesis, further testing should be done to determine if this small difference in reaction times is normal or an exceptional result in our study. The related hypothesis stating the simon-congruent accuracy rate would be lower than the simon-incongruent accuracy rate in the flanker incompatible condition is supported by our findings; however, in Table 3, the difference between the accuracy rates is minimal, with incongruent-incongruent having slightly higher accuracy than incongruent-congruent.
<font color='green'>Great job of thinking critically about the size of the effect, and considering the variance and not just the mean!</font>

The last two hypotheses state that the reaction times will be faster and the accuracy will be higher for the incompatible-incompatible trials, than either flanker incongruent - congruent or Simon incompatible - compatible contrasts. Our results do not support this hypothesis. As was previously stated above, the incongruent-incongruent condition had a faster reaction time and higher accuracy rate compared to the incongruent-congruent condition; however, the incongruent-incongruent condition did not have a faster reaction time or higher accuracy rate when compared to the congruent-incongruent condition. Once again, this conclusion is supported by the data displayed in Table 1 and Table 3. 

The box plots displayed show the data of each condition to be relatively normal, the median is towards the center of the box, and displays the center of the each conditions data quite well. Additionally, the variance of each conditions data can also be seen, with the incongruent-congruent condition showing more variance than the other conditions.

Our data indicates an interaction between the flanker and Simon conditions, with under-additivity between the two conditions. To explain, for the processes to have been independent of each other, the incongruent-incongruent condition would have to have been about 83 ms <font color='red'> Where does 83 ms come from? Show your calculations for all steps.</font> slower in reaction time than the congruent-congruent condition; however, the incongruent-incongruent condition was only about 60 ms slower than the congruent-congruent condition. Because of this, the incongruent-incongruent condition is under-additive and there is evidence the processes of the two effects overlap.

As an aside, it appears that the flanker condition's stimulus-stimulus (S-S) compatibility effects were responsible for the majority of the effect and have a greater impact than the simon condition's stimulus response (S-R) compatibility effects. For, when the flanker condition was incongruent there was a greater increase in reaction time then when it was absent. It also appears to contribute the most to the magnitude of the incongruent-incongruent effect as this effect was closer to the magnitude of the flanker-incongruent simon-congruent condition. <font color='orange'>Good job in thinking about this! Your logic is mostly good, though one criticism would be that since the flanker and Simon effects were not additive, we actually can't tell how much each contributed to the incompatible-incongruent condition</font>

# 

# Team Codezi Evaluation

<font color='green'> green = 1 </font><br>
<font color='orange'> orange = 0.5 </font><br>
<font color='red'> red = 0 </font>

### Completeness
Criterion: Objectives are met<br>
Modifier: colour * 1

Import data: <font color='green'> Met </font>

Clean Data<br>
- Scalable: <font color='red'> Not met </font>
- Outliers removed at individual subject level: <font color='red'> Not met </font>
- RT analyses on "Correct" trials only: <font color='red'> Not met </font>

EDA<br>
- Tables -- RT means/SDs, and error rates for each condition: <font color='green'> Met </font>
- Figure(s) -- Appropriate, informative, and sufficient: <font color='orange'> Mostly met -- No visualization for accuracy  </font>
- Hypotheses statements (support/refute): <font color='orange'> Mostly met -- contrast calculations not shown</font>

Conclusion Statement: <font color='orange'> Met</font>


```python
completeness = [1., 0., 0., 0., 1., .5, 1., .5]
len(completeness)
```




    8



### Accuracy
Criterion: End results are correct<br>
Modifier: colour * 4

Import data: <font color='green'> Correct </font>

Clean Data<br>
- Scalable: <font color='red'> Incorrect </font>
- Outliers removed at individual subject level: <font color='red'>  Absent</font>
- RT analyses on "Correct" trials only: <font color='red'> Absent </font>
    
EDA<br>
- Tables -- RT means/SDs, and error rates for each condition:  <font color='green'>  Correct</font>
- Figure(s) -- Appropriate, informative, and sufficient: <font color='green'> Correct </font>
- Hypotheses statements (support/refute): <font color='green'> Incorrect due to outlier inclusion, but that falls into the "carry-down error" category. As such, points were docked at the beginning of the error and not thereafter.</font>
   
Conclusion statement: <font color='green'> Correct </font>



```python
accuracy = [1., 0., 0., 0., 1., 1., 1., 1.]
len(accuracy)
```




    8



### Pythonicity
Criterion: Code is pythonic — follows the syntax guidelines of Python and especially The Zen of Python. For the most part you should be able to complete this project without using loops.<br>
Modifier: colour * 2.5

Import data: <font color='orange'> Mostly pythonic -- can be done without a loop </font><br>

Clean Data<br>
- Scalable: <font color='green'> What was done was done pythonically </font>
- Outliers removed at individual subject level: <font color='red'>  Absent</font>
- RT analyses on "Correct" trials only: <font color='red'> Absent </font>
    
EDA<br>
- Tables -- RT means & SDs for each condition: <font color='orange'> Mostly pythonic -- means and stdevs (if not accuracy as well) would have been better if provided together on one table.  </font>
- Figure(s) -- Appropriate, informative, and sufficient: <font color='green'> Pythonic </font>
- Hypotheses statements (support/refute): <font color='orange'> Mostly pythonic -- Blocks of text should be broken up with Markdown texture (e.g. sub-header separation, bolding/italicizing important info)  </font> 
   
Conclusion statement: <font color='orange'> Again, no differentiation among text </font>


```python
pythonicity = [.5, 1., 0., 0., .5, 1., .5, .5]
len(pythonicity)
```




    8



### Presentation Quality
Criteria: The end product (a Jupyter notebook) presents the code, data, and explanations clearly and elegantly. Data is presented and discussed thoroughly, and the data is presented in ways that are relevant to the research question. All material is relevant to addressing the experimental hypotheses (no off-topic discussion or gratuitous visualizations).<br>
Modifier: colour * 2.5

Import data: <font color='green'> Clear and elegant </font><br>

Clean Data<br>
- General (colour * 3): <font color='green'> Clear and elegant </font>
    
EDA<br>
- Tables -- RT means & SDs for each condition: <font color='green'>  Clear and elegant</font>
- Figure(s) -- Appropriate, informative, and sufficient: <font color='green'> Clear and elegant </font><br>
- Hypotheses statements (support/refute): <font color='green'> (Mostly) clear and elegant -- same reasons as above, but no benefit in penalizing twice for the same issues. I mention it here more for your consideration.   
</font>
   
Conclusion statement: <font color='orange'> Mostly clear and elegant -- Main research question referred to as "the last two hypotheses", leading to the most important conclusions not standing out as such.  </font>


```python
presentation = [1., 1., 1., 1., 1., 1., 1., .5]
len(presentation)
```




    8



### Adjustors:
Special values: green = 3, yellow = 1, red = -1<br>
<font color='green'>Critical thinking -- You actually noticed and thought about effect sizes, and commented that a difference seemed small in relation to the SD. </font><br>
<font color='red'>   </font>


```python
adjustors = [3.]
```

### Grade


```python
Comp = sum(completeness) * 1
Acc = sum(accuracy) * 4
Pyth = sum(pythonicity) * 2.5
Pres = sum(presentation) * 2.5
Adjust = sum(adjustors)
```


```python
Total = Comp + Acc + Pyth + Pres + Adjust
Max = len(completeness) * 1 + len(accuracy) * 4 + len(pythonicity) * 2.5 + len(presentation) * 2.5
```


```python
Grade = Total / Max
XP = Grade * 6000
```


```python
print("Grade = " + str(round(Grade * 100, 2)) + "%")
print(str(int(Grade * 6000)) + " XP")
```

    Grade = 69.69%
    4181 XP


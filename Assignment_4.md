# Assignment 4
## NESC 3505

## Single Unit Data


```python
blue = []
```

### Data Source
These data were provided courtesy of Dr. Nathan Crowder, Department of Psychology & Neuroscience, Dalhousie University. Data are from 23 mouse primary visual cortex neurons; for simplicity we are going to work with data firstly from only one neuron, then we'll load data from 3 neurons to get experience working with a larger data set. Note that these are *not* data from a multielectrode array, as were described in the textbook. Data were recorded from single electrodes that were embedded directly into single neurons, a technique called [**patch clamping**](https://www.leica-microsystems.com/science-lab/the-patch-clamp-technique). We have data from multiple neurons because Dr. Crowder's team painstakingly recorded from several neurons in each of a few animals. One of the important differences between this data set, and multielectrode array data, is that multielectrode arrays record from many neurons *simultaneously*, whereas patch clamping records from only one neuron at a time, so each neuron's data were recorded at different times. But for the purposes of this assignment, we can work with the data from multiple neurons in much the same ways as shown in the textbook. 

Prior to completing this assignment, you should read the [single unit data](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/introduction.html) chapter of the textbook.

### Experimental Design
The experiment involved presenting drifting (oscillating) sine wave gratings to mice while recording from electrodes in the primary visual cortex. The gratings oscillated (drifted) at a speed of 2 Hz. The contrast of the stimuli were varied systematically over ten levels from 4 (virtually no contrast between the brightest and darkest parts of the sine wave grating, which would appear as uniform grey) to 100% (gratings varying between black and white, as in the example grating in the [textbook](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/intro_multielec_data#stimuli)). **Contrast** is therefore one of the experimental variables; the levels of contrast used were 4, 8, 12, 16, 24, 32, 48, 64, 84, and 100%. 

A second variable is **condition**:the stimuli were presented under 2 experimental **conditions**: control (CTRL) and adaptation (ADAPT):
- In the **CTRL** condition, each trial began with a 2000 ms blank grey screen, then the oscillating sine wave grating at the trial's contrast level (see below) for 1000 ms, then 1000 ms blank grey screen.
- In the **ADAPT** condition, each trial started with the adaptation stimulus (a 50% contrast sine wave grating for 2000 ms), then the sine wave grating at the trial's contrast level (see below), then 1000 ms blank grey screen.

A total of 8 trials were presented at each contrast level, in each condition. Each trial was 4000 ms in duration, and data were sampled at 1000 Hz, meaning that we have a data point every 1 ms. 

### Predictions 
- In the control condition, latency to first spike is expected to decrease with increasing contrast.
- Some primary visual cortex cells are phase sensitive (so-called **simple cells**), so in response to a drifting sine wave grating they will oscillate between excitation and inhibition at the same temporal frequency of the stimulus (2 Hz in this case). 

## Load Necessary Packages

There's a couple of new packages here; the functions we need will be explained as we use them.


```python
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
```

### Define experiment parameters

Things we know about the experimental design that are useful to hard-code. In the next cell we define these as variables that we'll use later in the script, but here we'll also use them to visualize the experimental design.


```python
# Conditions
cond_labels = ['CTRL', 'ADAPT'] 

# Labels for the levels of stimulus contrast
contr_labels = [4, 8, 12, 16, 24, 32, 48, 64, 84, 100]

# Labels for repetitions (trials)
rep_labels = list(np.arange(1, 8))
num_reps = len(rep_labels)

# Time point labels - each trial is 4000 ms long
time_labels = list(np.arange(4000))

# Stimulus timing
stim_on_time = 2000
stim_off_time = stim_on_time + 1000

adapt_on_time = 0
adapt_off_time = adapt_on_time + 2000
```

### Plot Experimental Design

Here we plot the experimental conditions, and when the stimuli (adaptation and target gratings) were shown. This code is very similar to what was described in the [Ten Intensities](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/ten_intensities) chapter of the textbook, so refer to that for an explanation.

You should use this code as a starting point to plot the data below, because it basically provides a template to which you can add the raster plots and PSTHs.


```python
fig = plt.figure(figsize=[12,18])

subplot_counter = 1 # used to track subplots
for contr in contr_labels:
    for cond in cond_labels:
        ax = fig.add_subplot(len(contr_labels), len(cond_labels), subplot_counter)
        for rep in rep_labels:
            plt.axhline(rep - .5, 0, 4000, color='black', linestyle='--', linewidth=.75)

        # Show adaptation grating at 50% contrast
        if cond == 'ADAPT':
            plt.axvspan(0, stim_on_time-1, alpha=0.5, color='grey')

        # shading indicates stimulus on period
        plt.axvspan(stim_on_time, stim_off_time, alpha=contr/100, color='grey')

        # Pretty formatting
        plt.xlim([0, max(time_labels)+1])
        plt.ylim([0, len(rep_labels)])
        plt.title(cond + ' ' + str(contr) + '% contrast')
        plt.xlabel('Time (ms)')
        plt.ylabel('Repetition',)
        plt.yticks([x + 0.5 for x in range(num_reps)], [str(x + 1) for x in range(num_reps)], size=8) 
        plt.tight_layout()
        subplot_counter += 1

plt.show()
```




    
![png](Assignment_4_files/Assignment_4_9_0.png)
    



### Data Structure

Data are provided in two CSV files - one for the single neuron, and one that we'll load later, for three neurons.

The single-neuron data set is 4 dimensional. This is far less sci-fi than it sounds! It just means that we call each variable a "dimension" along which the data vary in a systematic way. So our dimensions are condition (2 levels) x contrast level (10) x repetitions (8) x time (4000 levels). This means we can compute the expected number of data points to be:


```python
2 * 10 * 8 * 4000
```




    640000



Whoa... 640,000 data points for a single neuron! 

One last thing to note, that follows from the 4000 time points, is that these data are not in the form of spike times, but rather times series in which each time point is represented as a 0 (not spiking) or 1 (spiking).

## Read in single neuron data

Data are saved in a CSV file, which we load into a pandas DataFrame:


```python
df = pd.read_csv('crowder_1_neuron.csv')
```

We should, naturally, look at the data frame and see how it's structured:


```python
df
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>m6_11</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>m6_11</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>m6_11</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>m6_11</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>m6_11</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>639995</th>
      <td>m6_11</td>
      <td>3995</td>
      <td>8</td>
      <td>100</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>639996</th>
      <td>m6_11</td>
      <td>3996</td>
      <td>8</td>
      <td>100</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>639997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>8</td>
      <td>100</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>639998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>8</td>
      <td>100</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>639999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>8</td>
      <td>100</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>640000 rows × 6 columns</p>
</div>



The text at the bottom of the output shows that we have 640,000 rows, which is the number we expect based on the math above. It's always good practice to confirm that what. you get is what you expect! 

Also, although we hard-coded labels for the experimental conditions and other parameters, we should check that what's in teh data matches what we expect. The `.unique()` method is useful for printing out the unique values in a particular column of a DataFrame:


```python
df['repetition'].unique()
```




    array([1, 2, 3, 4, 5, 6, 7, 8])



Do this now to see the unique values in the `contrast` column:

<font color='red'></font>


```python
blue += [1.]
```


```python
df['contrast'].unique()
```




    array([  4,   8,  12,  16,  24,  32,  48,  64,  84, 100])



Another useful thing to check is that we in fact have 4000 time points per trial. Printing out the unique values of time would yield excessive output (4000 values), but we can check in other ways. For example, below print out the max value of `time`:

<font color='red'></font>


```python
blue += [1.]
```


```python
print(df['time'].max())
```

    3999


Explain why the max is 3999 and not 4000:

<font color='red'></font>


```python
blue += [1.]
```

Because the first value for time is 0, not 1

Now use the pandas `.iloc[]` method to slice the DataFrame indices to show the rows of the DataFrame corresponding to the end of the first repetition of the first contrast level and the start of the second repetition of the first contrast level (for the first neuron, and the first condition). Show at least two rows before and after the transition from repetition 1 to 2. Hint: each repetition is 4000 time points, i.e., 4000 rows.

<font color='red'> Your output is correct, as Python will implicitly select all columns if none are specified. However, explicit is better than implicit (see The Zen of Python, linked in the course textbook here: https://dalpsychneuro.github.io/NESC_3505_textbook/2/python.html). It's best to add the "`, :`" after your row selection to explicitly select all columns. Same goes for the few cells below. <br> That being said, DataCamp teaches the implicit approach, so it's fine to use it here. Just know that it's even better (e.g., lower risk of systematic error) to be explicit. </font>


```python
blue += [1.]
```


```python
df.iloc[3997:4003]
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4000</th>
      <td>m6_11</td>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4001</th>
      <td>m6_11</td>
      <td>1</td>
      <td>2</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4002</th>
      <td>m6_11</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Show the rows of the DataFrame corresponding to the end of the 8th repetition of the first contrast level, and the 1st repetition of the second contrast level (for the first neuron, and the first condition). Show at least two rows before and after the transition.

You can use some multiplication (time points times repetitions) to figure out which rows those would be in the data frame.

<font color='red'></font>


```python
blue += [1.]
```


```python
df.iloc[35997:36003]
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>35997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>1</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>1</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>1</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36000</th>
      <td>m6_11</td>
      <td>0</td>
      <td>2</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36001</th>
      <td>m6_11</td>
      <td>1</td>
      <td>2</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36002</th>
      <td>m6_11</td>
      <td>2</td>
      <td>2</td>
      <td>8</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Show the rows of the DataFrame corresponding to the end of the CTRL condition (i.e., after all time points of all trials, of all contrast levels) and the start of the ADAPT condition, for the first neuron. Show at least two rows before and after the transition.

<font color='red'></font>


```python
blue += [1.]
```


```python
df.iloc[319997:320003]
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>319997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>320000</th>
      <td>m6_11</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>320001</th>
      <td>m6_11</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>320002</th>
      <td>m6_11</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>ADAPT</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Show the total number of spikes in the DataFrame. This will involve counting the number of rows where the value of the `spike` column is equal to 1. 

<font color='red'>`.sum()` works here because the values being counted are 1s, however `.count()` with a conditional statement (`df[df['spike']==1]['spike'].count()`) would be better as it would ignore any other numeric value that may happen to be in that column (accidents happen). </font>


```python
blue += [.75]
```


```python
print(df['spike'].sum())
```

    1955


Below you'll need to write code that selects, and plots, individual contrast levels, trials, and conditions. So it's good to refresh your memory of how to select a subset of rows of a DataFrame, based on values in certain columns. For example, to select only the rows for the control condition, we'd use:


```python
df[df['condition']=='CTRL']
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>m6_11</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>m6_11</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>m6_11</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>m6_11</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>m6_11</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>319995</th>
      <td>m6_11</td>
      <td>3995</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319996</th>
      <td>m6_11</td>
      <td>3996</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>319999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>8</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>320000 rows × 6 columns</p>
</div>



If we want to select based on multiple columns, we use the Boolean `&` operator. But we also have to contain each separate Boolean "selector" statement in parentheses. So if we want to select only the lowest contrast level (4) from only control trials, we'd use this code:


```python
df[(df['contrast']==4) & (df['condition']=='CTRL')]
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>m6_11</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>m6_11</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>m6_11</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>m6_11</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>m6_11</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>31995</th>
      <td>m6_11</td>
      <td>3995</td>
      <td>8</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31996</th>
      <td>m6_11</td>
      <td>3996</td>
      <td>8</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>8</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>8</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>8</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>32000 rows × 6 columns</p>
</div>



## Raster plots

This is a complex experiment so we'll need lots of subplots to see all trials for all contrast levels, and both conditions. But, as in the Ten Intensities example in the textbook, it's easiest to build up the plot routine by starting with a single trial, condition, and contrast level.

First, write code that will print out only the rows of the DataFrame corresponding to the first trial, the highest contrast level (100), and the control condition: 

<font color='red'></font>


```python
blue += [1.]
```


```python
df[(df['repetition'] == 1) & (df['contrast'] == 100) & (df['condition'] == 'CTRL')] 
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>288000</th>
      <td>m6_11</td>
      <td>0</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>288001</th>
      <td>m6_11</td>
      <td>1</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>288002</th>
      <td>m6_11</td>
      <td>2</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>288003</th>
      <td>m6_11</td>
      <td>3</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>288004</th>
      <td>m6_11</td>
      <td>4</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>291995</th>
      <td>m6_11</td>
      <td>3995</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>291996</th>
      <td>m6_11</td>
      <td>3996</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>291997</th>
      <td>m6_11</td>
      <td>3997</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>291998</th>
      <td>m6_11</td>
      <td>3998</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>291999</th>
      <td>m6_11</td>
      <td>3999</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>4000 rows × 6 columns</p>
</div>



One challenge in plotting this data is that a raster plot shows the times at which spikes occurred. In the Ten Intensities data set, the data were already stored as spike times. However, in the present data set, we have all time points, with a `1` in the `spike` column at the times when spikes occurred. So, we need to further filter our DataFrame, not just based on the labels you used in the cell above, but to select only the rows where the `spike` column has a value of `1`. Then, we need to pull out the values of the `time` column for those rows, to know when the spikes occurred. 

So first, copy and paste your code from the above answer, and add another Boolean statement to select only the rows where `spike` is equal to `1`:

<font color='red'>Instructions don't call for variable assignment. Not only does this add another variable that chews up some memory, it also requires the additional line of code to output said variable.</font>


```python
blue += [1.]
```


```python
df_raster = df[(df['repetition'] == 1) & (df['contrast'] == 100) & (df['condition'] == 'CTRL') & (df['spike'] == 1)] 
df_raster
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>288797</th>
      <td>m6_11</td>
      <td>797</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290153</th>
      <td>m6_11</td>
      <td>2153</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290362</th>
      <td>m6_11</td>
      <td>2362</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290420</th>
      <td>m6_11</td>
      <td>2420</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290439</th>
      <td>m6_11</td>
      <td>2439</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290473</th>
      <td>m6_11</td>
      <td>2473</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290500</th>
      <td>m6_11</td>
      <td>2500</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290557</th>
      <td>m6_11</td>
      <td>2557</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290633</th>
      <td>m6_11</td>
      <td>2633</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290657</th>
      <td>m6_11</td>
      <td>2657</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290715</th>
      <td>m6_11</td>
      <td>2715</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290759</th>
      <td>m6_11</td>
      <td>2759</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>290897</th>
      <td>m6_11</td>
      <td>2897</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>291050</th>
      <td>m6_11</td>
      <td>3050</td>
      <td>1</td>
      <td>100</td>
      <td>CTRL</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Now add to that statement to select only the `time` column:


```python

```

<font color='red'>Same here, but now I see that you're using said variable to minimize your typing--a different type of efficiency. Let's call it even this time. </font>


```python
blue += [1.]
```


```python
df_rast_time = pd.DataFrame(df_raster.iloc[:, 1])
df_rast_time
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
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>288797</th>
      <td>797</td>
    </tr>
    <tr>
      <th>290153</th>
      <td>2153</td>
    </tr>
    <tr>
      <th>290362</th>
      <td>2362</td>
    </tr>
    <tr>
      <th>290420</th>
      <td>2420</td>
    </tr>
    <tr>
      <th>290439</th>
      <td>2439</td>
    </tr>
    <tr>
      <th>290473</th>
      <td>2473</td>
    </tr>
    <tr>
      <th>290500</th>
      <td>2500</td>
    </tr>
    <tr>
      <th>290557</th>
      <td>2557</td>
    </tr>
    <tr>
      <th>290633</th>
      <td>2633</td>
    </tr>
    <tr>
      <th>290657</th>
      <td>2657</td>
    </tr>
    <tr>
      <th>290715</th>
      <td>2715</td>
    </tr>
    <tr>
      <th>290759</th>
      <td>2759</td>
    </tr>
    <tr>
      <th>290897</th>
      <td>2897</td>
    </tr>
    <tr>
      <th>291050</th>
      <td>3050</td>
    </tr>
  </tbody>
</table>
</div>



Now you have the data that you need in order to make a raster plot. In the cell below, add a line that assigns the output of the selector statement you produced in the cell above, to the variable `spike_times`. If you do this correctly, the code below should produce a raster plot

<font color='red'></font>


```python
blue += [1.]
```


```python
spike_times = df_rast_time

fig = plt.figure()
plt.vlines(spike_times, 0, 1);
```




    
![png](Assignment_4_files/Assignment_4_67_0.png)
    



We want to get to the point where we're looping through trials (repetitions), contrast levels, and conditions to generate the full set of subplots. This will involve loops, and assigning values of things like repetition number to variables, instead of hard-coding a specific level of repetition, contrast, and condition as you did above. So as the first step in doing this, in the cell below, first assign the correct values to variables called `rep`, `contr`, and `cond`. The, modify the selector code you used above to use these variables rather than hard-coding the values. For example, for repetition your solution code should use `rep` rather than `1`. The resulting plot should look identical to the one above.

<font color='red'>It's now becoming redundant, and no longer efficient. You're better off simply assigning the selector code to the `spike_times` variable and skipping the `pd.DataFrame` step altogether.</font>


```python
blue += [.75]
```


```python
rep = 1
contr = 100
cond = 'CTRL'

spike = df[(df['repetition'] == rep) & (df['contrast'] == contr) & (df['condition'] == cond) & (df['spike'] == 1)] 
spike_times = pd.DataFrame(spike.iloc[:, 1])

fig = plt.figure()
plt.vlines(spike_times, 0, 1);
```




    
![png](Assignment_4_files/Assignment_4_71_0.png)
    



Now we can start making the plots more pretty, and also prepare to plot the rasters for every trial, rather than just one. Add the same working code (selector statement) from above, into the corresponding spot below:

<font color='red'>Same as above, and same for below. However, because these are copy-and-paste incremental steps, we consider it a carry-down error (dinged for the first occurrance, but not thereafter). </font>


```python
blue += [1.]
```


```python
rep = 1
contr = 100
cond = 'CTRL'

spike = df[(df['repetition'] == rep) & (df['contrast'] == contr) & (df['condition'] == cond) & (df['spike'] == 1)] 
spike_times = pd.DataFrame(spike.iloc[:, 1])

fig = plt.figure()
plt.vlines(spike_times, 0, 1);

# shading indicates stimulus on period
plt.axvspan(stim_on_time, stim_off_time, alpha=contr/100, color='grey')

# Pretty formatting
plt.xlim([0, max(time_labels)+1])
plt.ylim([0, len(rep_labels)])
plt.title(cond + ' ' + str(contr) + '% contrast')
plt.xlabel('Time (ms)')
plt.ylabel('Repetition',)
plt.yticks([x + 0.5 for x in range(num_reps)], [str(x + 1) for x in range(num_reps)], size=8) 
plt.tight_layout()
```




    
![png](Assignment_4_files/Assignment_4_75_0.png)
    



You can see we've added things like axis labels, shading when the stimulus is on, and now the spikes are centered on the *y*-axis tick corresponding to trial `1`.

The next thing to do is to put the plotting code inside a loop, in order to plot each trial. we do this by looping through the `rep_labels` list that we hard-coded at the top of this file. For now, we'll stick with only plotting the highest contrast level for the control condition.

Below, you need to write the first line of the `for` loop, so that `rep` is updated for every value of `rep_labels`. Then, inside the loop, paste in the same selector code you used in your previous answer. The values of `contr`, and `cond` should still be set in memory from running the above code. All the indented lines of code provided below should be inside your `for` loop.

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure()

rep = 1
contr = 100
cond = 'CTRL'

for x in rep_labels:
    rep = x
    spike = df[(df['repetition'] == rep) & (df['contrast'] == contr) & (df['condition'] == cond) & (df['spike'] == 1)] 
    spike_times = pd.DataFrame(spike.iloc[:, 1])

    # Code below will format your plot nicely
    plt.vlines(spike_times, rep-1, rep);

    # shading indicates stimulus on period
    plt.axvspan(stim_on_time, stim_off_time, alpha=contr/100, color='grey')

    # Pretty formatting
    plt.xlim([0, max(time_labels)+1])
    plt.ylim([0, len(rep_labels)])
    plt.title(cond + ' ' + str(contr) + '% contrast')
    plt.xlabel('Time (ms)')
    plt.ylabel('Repetition',)
    plt.yticks([x + 0.5 for x in range(num_reps)], [str(x + 1) for x in range(num_reps)], size=8) 
    plt.tight_layout()

```




    
![png](Assignment_4_files/Assignment_4_79_0.png)
    



Now we're ready to generate a whole set of subplots for the different contrast levels and conditions. Below is a copy of the code from near the top of this file, under the heading "Plot Experimental Design". This has everything you need to generate the subplots, and label things appropriately. 

All you need to do is add in the code you wrote above to loop through repetitions, and to select the spike times for a particular combination of parameters (repetition, contrast, condition). Be careful, though, that the `plt.vlines()` command is *inside* the loop through reps, but all the lines of code after that are *outside* the loop.

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure(figsize=[12,18])

subplot_counter = 1 # used to track subplots

for contr in contr_labels:
    for cond in cond_labels:

        ax = fig.add_subplot(len(contr_labels), len(cond_labels), subplot_counter)

        # Show adaptation grating at 50% contrast
        if cond == 'ADAPT':
            plt.axvspan(0, stim_on_time-1, alpha=0.5, color='grey')

        # Shading indicates stimulus on period
        plt.axvspan(stim_on_time, stim_off_time, alpha=contr/100, color='grey')

        for x in rep_labels:
            rep = x
            spike = df[(df['repetition'] == rep) & (df['contrast'] == contr) & (df['condition'] == cond) & (df['spike'] == 1)] 
            spike_times = pd.DataFrame(spike.iloc[:, 1])

            plt.vlines(spike_times, rep-1, rep);

        # Formatting
        plt.xlim([0, max(time_labels)+1])
        plt.ylim([0, len(rep_labels)])
        plt.title(cond + ' ' + str(contr) + '% contrast')
        plt.xlabel('Time (ms)')
        plt.ylabel('Repetition',)
        plt.yticks([x + 0.5 for x in range(num_reps)], [str(x + 1) for x in range(num_reps)], size=8) 
        plt.tight_layout()
        subplot_counter += 1
        
plt.show()
```




    
![png](Assignment_4_files/Assignment_4_83_0.png)
    



## PSTHs

You can adapt the code above to generate peri-stimulus time histogram (PTSH) plot rather than rasters. However, because the code is pretty dense, it can again be helpful to start simple, and figure out how to generate the PSTH for a single subplot: one conditino (control) and contrast level (100%).

To start with, we'll define that we want the histogram bins for the PSTH to be 50 ms wide, and use this to create an array of the edges (start/end times) of the bins; you'll need this later in creating the histograms:


```python
# Define bins for histograms
hist_bin_width = 50 
time_bins = np.arange(0, max(time_labels), hist_bin_width)
print(time_bins)
```

    [   0   50  100  150  200  250  300  350  400  450  500  550  600  650
      700  750  800  850  900  950 1000 1050 1100 1150 1200 1250 1300 1350
     1400 1450 1500 1550 1600 1650 1700 1750 1800 1850 1900 1950 2000 2050
     2100 2150 2200 2250 2300 2350 2400 2450 2500 2550 2600 2650 2700 2750
     2800 2850 2900 2950 3000 3050 3100 3150 3200 3250 3300 3350 3400 3450
     3500 3550 3600 3650 3700 3750 3800 3850 3900 3950]


Next you can generate a single PSTH. 

First, use selector code (similar to what you used for the raster plots above) to store the spike times for 100% contrast trials in the control condition to `spike_times`. You'll want to include lines to assign values to `contr` and `cond`, then run the selector code using those variables. Importantly, because PSTHs represent the count of spikes in each time bin *across trials*, you should not include a selector for repetition. 

Next, copy the code for plotting a PSTH from the first PSTH example in the [Ten Intensities textbook chapter](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/ten_intensities.html). You'll need two lines of code specifically: the one that runs the `np.histogram()` command (which computes the histogram, and stores the result in two variables), and the `plt.bar()` command that comes after it. 

You'll need to modify the code that you copy a bit. In the `np.histogram` command, the `bins=` argument should take the `time_bins` variable defined in the cell above. Then in the `plt.bar` command, include the argument `width=hist_bin_width` along with the two arguments shown in the Ten Intensities chapter.

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure()

contr = 100
cond = 'CTRL'
spike_cont = df[(df['contrast'] == contr) & (df['spike'] == 1) & (df['condition'] == cond)]
spike_times = spike_cont.iloc[:, 1]

spike_count, bins = np.histogram(spike_times, bins=time_bins)
plt.bar(bins[:-1], spike_count, width=hist_bin_width)

plt.show()
```




    
![png](Assignment_4_files/Assignment_4_89_0.png)
    



One addition modification to make is to *standardize* the histogram; that is, plot in terms of the probability of a spike rather than the count of spikes, in each time bin. To do this, you need to divide `spike_count` by the number of repetitions (`num_reps`, which was already set near the top of this script). Copy and paste your working code from the cell above, and add the code to standardize the height of the bars.

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure()

contr = 100
cond = 'CTRL'
spike_cont = df[(df['contrast'] == contr) & (df['spike'] == 1) & (df['condition'] == cond)]
spike_times = spike_cont.iloc[:, 1]

spike_count, bins = np.histogram(spike_times, bins=time_bins)
plt.bar(bins[:-1], spike_count/num_reps, width=hist_bin_width)

plt.show()
```




    
![png](Assignment_4_files/Assignment_4_93_0.png)
    



If you look at the result, you should see something surprising: the tallest bar has a height of 2. This is surprising because the maximum probability of a spike occurring should be 1.0 — which corresponds to a 100% probability of a spike at that time. Indeed, if you look at the *y* axis of the first PSTH above, the tallest bar has a *y* value of 16, which doesn't really make sense since there were only 8 trials. The issue is that our histogram bins are 50 ms wide, and data were recorded every millisecond. This means that even on a single trial, a bin could conceivably contain up to 50 spikes, not just one. Therefore, to properly compute the probability of a spike occurring in each 50 ms time bin, we need to divide by the number of repetitions (8) multiplied by the number of time points in that bin (50). 

In the cell below, paste your code for plotting a standardized PSTH from the cell above, but modify it to divide by `(num_reps * hist_bin_width)`. The result should have a *y* axis with a maximum value of less than 0.1.

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure()

contr = 100
cond = 'CTRL'
spike_cont = df[(df['contrast'] == contr) & (df['spike'] == 1) & (df['condition'] == cond)]
spike_times = spike_cont.iloc[:, 1]

spike_count, bins = np.histogram(spike_times, bins=time_bins)
plt.bar(bins[:-1], spike_count/(num_reps*hist_bin_width), width=hist_bin_width)

plt.show()
```




    
![png](Assignment_4_files/Assignment_4_97_0.png)
    



Now you're ready to use this code in loops to generate the full set of PSTHs for all contrast levels and conditions, in a similar format to the rasters above. 

Start by copying and pasting your code from above that prints all of the raster plots. The modify it to create PSTHs. To do this, you'll need to:
- remove the loop through reps
- modify the `spike_times` selector code so that it doesn't select a particular repetition (i.e., you will take all data from that contrast and condition, regardless of which repetition it was)
- remove the `plt.vline()` code that generates the raster "ticks"
- add the two lines of code you generated in the previous cell, for `np.histogram()` and `plt.bar()`
- change the *y*-axis label from "Repetition" to "Spike Probability"
- change the *y* axis limit (`plt.ylim()` so that its maximum is 0.05 (5% probability), rather than the number of trials (`len(rep_labels)`). Although 5% probability might seem low, it's not in this context.
- remove the line starting with `plt.yticks`. This was used to hard-code a tick on the *y* axis for every repetition, but that' doesn't make sense now that the *y* axis represents spike probability

<font color='red'></font>


```python
blue += [1.]
```


```python
fig = plt.figure(figsize=[12,18])

subplot_counter = 1 # used to track subplots

for contr in contr_labels:
    for cond in cond_labels:

        ax = fig.add_subplot(len(contr_labels), len(cond_labels), subplot_counter)

        # Show adaptation grating at 50% contrast
        if cond == 'ADAPT':
            plt.axvspan(0, stim_on_time-1, alpha=0.5, color='grey')

        # shading indicates stimulus on period
        plt.axvspan(stim_on_time, stim_off_time, alpha=contr/100, color='grey')

        spike = df[(df['contrast'] == contr) & (df['condition'] == cond) & (df['spike'] == 1)] 
        spike_times = pd.DataFrame(spike.iloc[:, 1])
        
        spike_count, bins = np.histogram(spike_times, bins=time_bins)
        plt.bar(bins[:-1], spike_count/(num_reps*hist_bin_width), width=hist_bin_width)

        # Pretty formatting
        plt.xlim([0, max(time_labels)+1])
        plt.ylim([0, 0.05])
        plt.title(cond + ' ' + str(contr) + '% contrast')
        plt.xlabel('Time (ms)')
        plt.ylabel('Spike Probability',)
        plt.tight_layout()
        subplot_counter += 1
        
plt.show()
```




    
![png](Assignment_4_files/Assignment_4_101_0.png)
    



## Heat maps
    
Plot heat maps showing the PTSHs, with colour indicating the spike count. The great thing about heat maps is how condensed they are. We don't really need to use an interactive plot, because one plot includes all contrast levels. So plot this with two columns (CTRL, ADAPT) and 1 row.
    
For each heat map, put time on the *x* axis, contrast on the *y* axis, and colour indicating the number of spikes per bin. 

For heat maps you need to compute the histograms for all contrast levels, and store these temporarily in an object such as a pandas DataFrame, list, dictionary, or numpy array. The textbook chapter on heat maps showed how to do this with a dictionary, but we suggest that instead you adapt the code from the section on [Pre-Computing Histograms](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/pandas_multielec#pre-computing-histograms).


First, create a pandas DataFrame with two columns: `condition` and `contrast`. We will append this DataFrame to the DataFrame of histograms, and these columns will serve as labels. You can use nested list comprehension to create the columns. The first 10 rows should be for the `CTRL` condition, and the last 10 for the `ADAPT` condition. The `contrast` column should cycle through the levels of contrast, once for the control condition, and then a second time for the adaptation condition. 

<font color='red'>Ultimately the correct output, but you didn't use the nested list comprehension as specified in the instructions. Your solution is much more labour intensive.</font>


```python
blue += [.5]
```


```python
cond_labels_1 = ['CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'CTRL', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT', 'ADAPT']
contr_labels_1 = [4, 8, 12, 16, 24, 32, 48, 64, 84, 100, 4, 8, 12, 16, 24, 32, 48, 64, 84, 100]

condition_labels = pd.DataFrame({'condition':cond_labels_1, 'contrast':contr_labels_1})
```


```python
# This will print out the resulting DataFrame so we can check it
condition_labels
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
      <th>condition</th>
      <th>contrast</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CTRL</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CTRL</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CTRL</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CTRL</td>
      <td>16</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CTRL</td>
      <td>24</td>
    </tr>
    <tr>
      <th>5</th>
      <td>CTRL</td>
      <td>32</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CTRL</td>
      <td>48</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CTRL</td>
      <td>64</td>
    </tr>
    <tr>
      <th>8</th>
      <td>CTRL</td>
      <td>84</td>
    </tr>
    <tr>
      <th>9</th>
      <td>CTRL</td>
      <td>100</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ADAPT</td>
      <td>4</td>
    </tr>
    <tr>
      <th>11</th>
      <td>ADAPT</td>
      <td>8</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ADAPT</td>
      <td>12</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ADAPT</td>
      <td>16</td>
    </tr>
    <tr>
      <th>14</th>
      <td>ADAPT</td>
      <td>24</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ADAPT</td>
      <td>32</td>
    </tr>
    <tr>
      <th>16</th>
      <td>ADAPT</td>
      <td>48</td>
    </tr>
    <tr>
      <th>17</th>
      <td>ADAPT</td>
      <td>64</td>
    </tr>
    <tr>
      <th>18</th>
      <td>ADAPT</td>
      <td>84</td>
    </tr>
    <tr>
      <th>19</th>
      <td>ADAPT</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
</div>



Next, create a DataFrame in which each row is the PSTH for a given condition and contrast level. First loop through conditions and contrasts to create a list of DataFrames for each condition/contrast. The order of the PTSHs should match the order of the labels above, so be sure you run your `for` loops in the same order below as you did for the labels above. 

<font color='red'></font>


```python
blue += [1.]
```


```python
psth_df_list = []
for cond in cond_labels:
    for contr in contr_labels:
        spike_times = df[(df['condition']==cond) & (df['contrast']==contr) & (df['spike']==1)]['time']
        hist, bins = np.histogram(spike_times, bins=time_bins)
        psth_df_list.append(hist)
```

Now, concatenate all the DataFrames in the list into one DataFrame, called `psth_df`. 


```python
### BEGIN SOLUTION
psth_df = pd.concat([condition_labels, pd.DataFrame(psth_df_list)], axis=1)       
### END SOLUTION
```

Finally, set the `condition` and `contrast` columns to be indexes.

<font color='red'></font>


```python
blue += [1.]
```


```python
psth_df = psth_df.set_index(['condition', 'contrast'])
```


```python
# This again will print the result so we can confirm it looks right
psth_df
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
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>69</th>
      <th>70</th>
      <th>71</th>
      <th>72</th>
      <th>73</th>
      <th>74</th>
      <th>75</th>
      <th>76</th>
      <th>77</th>
      <th>78</th>
    </tr>
    <tr>
      <th>condition</th>
      <th>contrast</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">CTRL</th>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>48</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>100</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">ADAPT</th>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>7</td>
      <td>9</td>
      <td>4</td>
      <td>4</td>
      <td>10</td>
      <td>8</td>
      <td>6</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>5</td>
      <td>9</td>
      <td>10</td>
      <td>8</td>
      <td>11</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2</td>
      <td>0</td>
      <td>5</td>
      <td>3</td>
      <td>6</td>
      <td>1</td>
      <td>8</td>
      <td>6</td>
      <td>15</td>
      <td>6</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>9</td>
      <td>7</td>
      <td>8</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>0</td>
      <td>0</td>
      <td>7</td>
      <td>9</td>
      <td>4</td>
      <td>2</td>
      <td>14</td>
      <td>7</td>
      <td>8</td>
      <td>8</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>7</td>
      <td>3</td>
      <td>15</td>
      <td>7</td>
      <td>5</td>
      <td>9</td>
      <td>9</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>48</th>
      <td>0</td>
      <td>0</td>
      <td>8</td>
      <td>9</td>
      <td>8</td>
      <td>10</td>
      <td>6</td>
      <td>10</td>
      <td>8</td>
      <td>15</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>0</td>
      <td>0</td>
      <td>6</td>
      <td>4</td>
      <td>4</td>
      <td>8</td>
      <td>12</td>
      <td>9</td>
      <td>6</td>
      <td>9</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>0</td>
      <td>1</td>
      <td>6</td>
      <td>6</td>
      <td>4</td>
      <td>9</td>
      <td>12</td>
      <td>8</td>
      <td>10</td>
      <td>12</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>100</th>
      <td>0</td>
      <td>0</td>
      <td>6</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
      <td>5</td>
      <td>5</td>
      <td>10</td>
      <td>9</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>20 rows × 79 columns</p>
</div>



Now you can write a `for` loop through conditions to plot the heat maps — remember to use two subplots in one row, with separate heat maps (subplots) for each of the two conditions.

As in the textbook, use the `plt.imshow()` function to generate the heat maps. You will need to add one argument to the command, though, that's not shown in the textbook: `aspect=5`. This is needed to get the aspect ratio (height to width ratio) of the plot to look good, given the range of contrast levels and times.

Be mindful of your choice of color map and interpolation method.

The code provided will help make your plot clear and interpretable. 

<font color='red'>Colour scheme (`cmap`) does not reflect the textbook material (https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/heat_maps.html), which states why "hot" is problematic.</font>


```python
blue += [.75]
```


```python
fig = plt.figure(figsize=[15,30])
subplot_counter = 1 # used to track subplots


for cond in cond_labels:
    ax = fig.add_subplot(2, 2, subplot_counter)
    aa = plt.imshow(psth_df.loc[cond], origin='lower', cmap='hot', interpolation='bilinear', aspect=5)

    cb = fig.colorbar(aa, shrink=0.33)
    cb.ax.set_ylabel('Spike Probability')
    
    plt.title(cond)

    xticks = range(0, len(time_bins), 10)
    plt.xticks(xticks,
           time_bins[xticks])
    plt.xlabel('Time (ms)')

    plt.yticks([x for x,y in enumerate(contr_labels)], 
               [x for x in contr_labels])
    if cond == cond_labels[0]:
        plt.ylabel('Stimulus Contrast')

    subplot_counter += 1

plt.show()

```




    
![png](Assignment_4_files/Assignment_4_122_0.png)
    



Provide a rationale for your choice of color map and interpolation method for the heat maps above.

<font color='red'>`"hot"` cmap was referenced at the beginning of the textbook section, however the section went on to describe better options.</font>


```python
blue += [.75]
```

I used the same color map as the textbook, not sure of reasoning other than it looks nice and has a good contrast

The bilinear interpolation allows for a smaller time scale for each pixel, creating a less blocky style of heat map

Compare the heat maps to the PSTHs you plotted above. Do you think that they tell a consistent "story" about the data, or do you see different things in each? Elaborate. 

<font color='red'> In addition to these similarities between PSTHs and heat maps, what about the benefit of the colour intensity in the latter? What does that tell us, and can it be found in the histograms? Also, note the efficiency benefit of two heat map plots versus 20 histograms. </font>


```python
blue += [.75]
```

the two maps seem consistent with the PSTHs: for the CTRL condition the higher spike probability is more rare and concentrated at the later time intervals, whereas for the ADAPT condition the higher spike probability is more common and concentrated at the earlier time intervals. This can be seen in both the PSTHs and the heat maps.

## Working with multiple neurons

The CoCalc cloud service is great in many ways, but each student doesn't get a lot of RAM to use. This limits how many neurons' data we can work with at once — we can't get anywhere hear the 90+ neurons used in the textbook example! 

But we can load in data from three neurons, allowing you to demonstrate your ability to work with subplots, as well as to compare data between different neurons. We won't repeat all of the steps you performed above, but instead go straight to plotting heat maps, since they make it easiest to compare many neurons at once.

The CSV file `crowder_3_neurons.csv` contains data from three neurons, from the same experiment as the data we've worked with so far. Load it as a pandas DataFrame called `df3`:

<font color='red'></font>


```python
blue += [1.]
```


```python
df3 = pd.read_csv('crowder_3_neurons.csv')
```

View the head of `df_3neurons`

<font color='red'></font>


```python
blue += [1.]
```


```python
df3.head()
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
      <th>neuron</th>
      <th>time</th>
      <th>repetition</th>
      <th>contrast</th>
      <th>condition</th>
      <th>spike</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>m1_6</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>m1_6</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>m1_6</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>m1_6</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>m1_6</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>CTRL</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Use the `.unique()` method to assign the names of the three neurons in this data to a list called `neuron_labels`:

<font color='red'></font>


```python
blue += [1.]
```


```python
neuron_labels = sorted(df3['neuron'].unique())

print(neuron_labels)
```

    ['m1_6', 'm3_11', 'm6_3a2']


We're going to create a DataFrame of PSTHs as we did above. This time, however, the DataFrame will include data from all three neurons, so it will have 3x as many rows as before, and an extra index column labeling which neuron each PSTH came from. 

We'll create the index columns first. Copy and paste the code you used above under **Heat Maps** to create a DataFrame called `condition_labels`. Add to the code to create another column, *before* the `condition` and `contrast` columns. Call the additional column `neuron` and fill it with neuron names. 

<font color='red'>First three variable assignments unnecessary. These values are already assigned in cells 145, 8, and 8 (respectively).</font>


```python
blue += [.75]
```


```python
neuron_labels = sorted(df3['neuron'].unique())
cond_labels_2 = df3['condition'].unique()
contr_labels_2 = sorted(df3['contrast'].unique())

condition_labels_2 = pd.DataFrame([[neur, cond, contr] for neur in neuron_labels for cond in cond_labels_2 for contr in contr_labels_2], columns=['neuron', 'condition', 'contrast'])
condition_labels_2
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
      <th>neuron</th>
      <th>condition</th>
      <th>contrast</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>16</td>
    </tr>
    <tr>
      <th>4</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>24</td>
    </tr>
    <tr>
      <th>5</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>32</td>
    </tr>
    <tr>
      <th>6</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>48</td>
    </tr>
    <tr>
      <th>7</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>64</td>
    </tr>
    <tr>
      <th>8</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>84</td>
    </tr>
    <tr>
      <th>9</th>
      <td>m1_6</td>
      <td>CTRL</td>
      <td>100</td>
    </tr>
    <tr>
      <th>10</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>4</td>
    </tr>
    <tr>
      <th>11</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>8</td>
    </tr>
    <tr>
      <th>12</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>12</td>
    </tr>
    <tr>
      <th>13</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>16</td>
    </tr>
    <tr>
      <th>14</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>24</td>
    </tr>
    <tr>
      <th>15</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>32</td>
    </tr>
    <tr>
      <th>16</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>48</td>
    </tr>
    <tr>
      <th>17</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>64</td>
    </tr>
    <tr>
      <th>18</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>84</td>
    </tr>
    <tr>
      <th>19</th>
      <td>m1_6</td>
      <td>ADAPT</td>
      <td>100</td>
    </tr>
    <tr>
      <th>20</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>4</td>
    </tr>
    <tr>
      <th>21</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>8</td>
    </tr>
    <tr>
      <th>22</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>12</td>
    </tr>
    <tr>
      <th>23</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>16</td>
    </tr>
    <tr>
      <th>24</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>24</td>
    </tr>
    <tr>
      <th>25</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>32</td>
    </tr>
    <tr>
      <th>26</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>48</td>
    </tr>
    <tr>
      <th>27</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>64</td>
    </tr>
    <tr>
      <th>28</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>84</td>
    </tr>
    <tr>
      <th>29</th>
      <td>m3_11</td>
      <td>CTRL</td>
      <td>100</td>
    </tr>
    <tr>
      <th>30</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>4</td>
    </tr>
    <tr>
      <th>31</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>8</td>
    </tr>
    <tr>
      <th>32</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>12</td>
    </tr>
    <tr>
      <th>33</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>16</td>
    </tr>
    <tr>
      <th>34</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>24</td>
    </tr>
    <tr>
      <th>35</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>32</td>
    </tr>
    <tr>
      <th>36</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>48</td>
    </tr>
    <tr>
      <th>37</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>64</td>
    </tr>
    <tr>
      <th>38</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>84</td>
    </tr>
    <tr>
      <th>39</th>
      <td>m3_11</td>
      <td>ADAPT</td>
      <td>100</td>
    </tr>
    <tr>
      <th>40</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>4</td>
    </tr>
    <tr>
      <th>41</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>8</td>
    </tr>
    <tr>
      <th>42</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>12</td>
    </tr>
    <tr>
      <th>43</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>16</td>
    </tr>
    <tr>
      <th>44</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>24</td>
    </tr>
    <tr>
      <th>45</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>32</td>
    </tr>
    <tr>
      <th>46</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>48</td>
    </tr>
    <tr>
      <th>47</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>64</td>
    </tr>
    <tr>
      <th>48</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>84</td>
    </tr>
    <tr>
      <th>49</th>
      <td>m6_3a2</td>
      <td>CTRL</td>
      <td>100</td>
    </tr>
    <tr>
      <th>50</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>4</td>
    </tr>
    <tr>
      <th>51</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>8</td>
    </tr>
    <tr>
      <th>52</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>12</td>
    </tr>
    <tr>
      <th>53</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>16</td>
    </tr>
    <tr>
      <th>54</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>24</td>
    </tr>
    <tr>
      <th>55</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>32</td>
    </tr>
    <tr>
      <th>56</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>48</td>
    </tr>
    <tr>
      <th>57</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>64</td>
    </tr>
    <tr>
      <th>58</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>84</td>
    </tr>
    <tr>
      <th>59</th>
      <td>m6_3a2</td>
      <td>ADAPT</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
</div>



Now create a DataFrame of PSTHs, called `psth_df3_list`, looping through neurons, conditions, and contrasts. Again, you can copy and paste similar code from above, and nest that all inside an additional `for` loop to cycle through neurons. Remember to change all of the occurrences of `df` in the code you copy to `df3`, and all occurrences of `psth_df_list` to `psth_df3_list`. Again, double-check that the order of your loops here is the same as used above in generating the index columns.

<font color='red'></font>


```python
blue += [1.]
```


```python
psth_df3_list = []
for neur in neuron_labels:
    for cond in cond_labels_2:
        for contr in contr_labels_2:
            spike_times = df3[(df3['neuron']==neur) & (df3['condition']==cond) & (df3['contrast']==contr) & (df3['spike']==1)]['time']
            hist, bins = np.histogram(spike_times, bins=time_bins)
            psth_df3_list.append(hist)
```

<font color='red'></font>


```python
blue += [1.]
```

Finally, concatenate `condition_labels` with `psth_df3_list`, assign the result to `psth_df3`, and set `neuron`, `condition`, and `contrast` as indexes of `psth_df3`.


```python
psth_df3 = pd.concat([condition_labels_2, pd.DataFrame(psth_df3_list)], axis=1) 
psth_df3 = psth_df3.set_index(['neuron', 'condition', 'contrast'])
psth_df3
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
      <th></th>
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>69</th>
      <th>70</th>
      <th>71</th>
      <th>72</th>
      <th>73</th>
      <th>74</th>
      <th>75</th>
      <th>76</th>
      <th>77</th>
      <th>78</th>
    </tr>
    <tr>
      <th>neuron</th>
      <th>condition</th>
      <th>contrast</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="20" valign="top">m1_6</th>
      <th rowspan="10" valign="top">CTRL</th>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>5</td>
      <td>2</td>
      <td>1</td>
      <td>5</td>
      <td>2</td>
      <td>...</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
      <td>3</td>
      <td>5</td>
      <td>1</td>
      <td>...</td>
      <td>2</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>5</td>
      <td>0</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>5</td>
      <td>1</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>...</td>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>2</td>
      <td>12</td>
      <td>5</td>
      <td>1</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>32</th>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2</td>
      <td>5</td>
      <td>5</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>1</td>
      <td>5</td>
      <td>1</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>100</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">ADAPT</th>
      <th>4</th>
      <td>4</td>
      <td>0</td>
      <td>13</td>
      <td>4</td>
      <td>17</td>
      <td>10</td>
      <td>14</td>
      <td>16</td>
      <td>10</td>
      <td>9</td>
      <td>...</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>5</td>
      <td>16</td>
      <td>10</td>
      <td>14</td>
      <td>8</td>
      <td>12</td>
      <td>11</td>
      <td>9</td>
      <td>12</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>6</td>
      <td>4</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2</td>
      <td>1</td>
      <td>9</td>
      <td>9</td>
      <td>13</td>
      <td>5</td>
      <td>10</td>
      <td>7</td>
      <td>8</td>
      <td>7</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3</td>
      <td>6</td>
      <td>14</td>
      <td>11</td>
      <td>16</td>
      <td>11</td>
      <td>12</td>
      <td>9</td>
      <td>10</td>
      <td>11</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3</td>
      <td>6</td>
      <td>13</td>
      <td>10</td>
      <td>17</td>
      <td>12</td>
      <td>9</td>
      <td>13</td>
      <td>10</td>
      <td>6</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>1</td>
      <td>2</td>
      <td>12</td>
      <td>7</td>
      <td>11</td>
      <td>11</td>
      <td>9</td>
      <td>11</td>
      <td>8</td>
      <td>7</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>1</td>
      <td>4</td>
      <td>12</td>
      <td>7</td>
      <td>15</td>
      <td>14</td>
      <td>11</td>
      <td>14</td>
      <td>11</td>
      <td>9</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>64</th>
      <td>1</td>
      <td>1</td>
      <td>14</td>
      <td>7</td>
      <td>14</td>
      <td>13</td>
      <td>10</td>
      <td>9</td>
      <td>8</td>
      <td>8</td>
      <td>...</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>4</td>
      <td>2</td>
      <td>18</td>
      <td>10</td>
      <td>14</td>
      <td>8</td>
      <td>6</td>
      <td>10</td>
      <td>11</td>
      <td>7</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>100</th>
      <td>1</td>
      <td>5</td>
      <td>11</td>
      <td>9</td>
      <td>9</td>
      <td>8</td>
      <td>11</td>
      <td>9</td>
      <td>11</td>
      <td>8</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th rowspan="20" valign="top">m3_11</th>
      <th rowspan="10" valign="top">CTRL</th>
      <th>4</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>6</td>
      <td>2</td>
      <td>6</td>
      <td>2</td>
      <td>1</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>2</td>
      <td>5</td>
      <td>3</td>
    </tr>
    <tr>
      <th>8</th>
      <td>7</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>5</td>
      <td>3</td>
      <td>3</td>
      <td>...</td>
      <td>3</td>
      <td>7</td>
      <td>3</td>
      <td>4</td>
      <td>8</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>4</td>
      <td>5</td>
      <td>5</td>
      <td>8</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>5</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>16</th>
      <td>9</td>
      <td>6</td>
      <td>5</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>...</td>
      <td>4</td>
      <td>2</td>
      <td>6</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>6</td>
      <td>2</td>
      <td>1</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>8</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>32</th>
      <td>5</td>
      <td>3</td>
      <td>5</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>4</td>
      <td>7</td>
      <td>...</td>
      <td>5</td>
      <td>2</td>
      <td>4</td>
      <td>7</td>
      <td>4</td>
      <td>3</td>
      <td>6</td>
      <td>4</td>
      <td>8</td>
      <td>6</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>5</td>
      <td>7</td>
      <td>2</td>
      <td>4</td>
      <td>6</td>
      <td>4</td>
      <td>4</td>
      <td>...</td>
      <td>2</td>
      <td>8</td>
      <td>6</td>
      <td>4</td>
      <td>5</td>
      <td>6</td>
      <td>1</td>
      <td>7</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>64</th>
      <td>8</td>
      <td>4</td>
      <td>0</td>
      <td>4</td>
      <td>7</td>
      <td>3</td>
      <td>2</td>
      <td>10</td>
      <td>8</td>
      <td>6</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>7</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>84</th>
      <td>7</td>
      <td>0</td>
      <td>4</td>
      <td>2</td>
      <td>8</td>
      <td>1</td>
      <td>1</td>
      <td>6</td>
      <td>5</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>100</th>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>2</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>6</td>
      <td>3</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>6</td>
      <td>4</td>
      <td>6</td>
      <td>4</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">ADAPT</th>
      <th>4</th>
      <td>5</td>
      <td>6</td>
      <td>15</td>
      <td>20</td>
      <td>21</td>
      <td>17</td>
      <td>23</td>
      <td>19</td>
      <td>13</td>
      <td>18</td>
      <td>...</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>7</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3</td>
      <td>2</td>
      <td>24</td>
      <td>21</td>
      <td>25</td>
      <td>16</td>
      <td>15</td>
      <td>16</td>
      <td>15</td>
      <td>21</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>2</td>
      <td>19</td>
      <td>23</td>
      <td>22</td>
      <td>14</td>
      <td>18</td>
      <td>15</td>
      <td>17</td>
      <td>15</td>
      <td>...</td>
      <td>6</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>6</td>
      <td>8</td>
      <td>3</td>
      <td>5</td>
      <td>9</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>5</td>
      <td>7</td>
      <td>25</td>
      <td>23</td>
      <td>15</td>
      <td>24</td>
      <td>16</td>
      <td>18</td>
      <td>23</td>
      <td>17</td>
      <td>...</td>
      <td>2</td>
      <td>4</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>7</td>
      <td>5</td>
      <td>18</td>
      <td>21</td>
      <td>27</td>
      <td>22</td>
      <td>12</td>
      <td>18</td>
      <td>16</td>
      <td>18</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>5</td>
    </tr>
    <tr>
      <th>32</th>
      <td>5</td>
      <td>5</td>
      <td>24</td>
      <td>19</td>
      <td>23</td>
      <td>18</td>
      <td>19</td>
      <td>17</td>
      <td>15</td>
      <td>13</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>48</th>
      <td>3</td>
      <td>5</td>
      <td>24</td>
      <td>22</td>
      <td>20</td>
      <td>27</td>
      <td>18</td>
      <td>24</td>
      <td>19</td>
      <td>23</td>
      <td>...</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>4</td>
      <td>5</td>
      <td>5</td>
      <td>9</td>
      <td>8</td>
    </tr>
    <tr>
      <th>64</th>
      <td>0</td>
      <td>3</td>
      <td>18</td>
      <td>23</td>
      <td>19</td>
      <td>20</td>
      <td>18</td>
      <td>22</td>
      <td>14</td>
      <td>18</td>
      <td>...</td>
      <td>6</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>84</th>
      <td>5</td>
      <td>5</td>
      <td>18</td>
      <td>21</td>
      <td>23</td>
      <td>23</td>
      <td>18</td>
      <td>16</td>
      <td>10</td>
      <td>17</td>
      <td>...</td>
      <td>2</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>5</td>
      <td>4</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <th>100</th>
      <td>2</td>
      <td>7</td>
      <td>21</td>
      <td>22</td>
      <td>19</td>
      <td>18</td>
      <td>16</td>
      <td>17</td>
      <td>15</td>
      <td>16</td>
      <td>...</td>
      <td>6</td>
      <td>2</td>
      <td>5</td>
      <td>8</td>
      <td>5</td>
      <td>1</td>
      <td>7</td>
      <td>11</td>
      <td>3</td>
      <td>11</td>
    </tr>
    <tr>
      <th rowspan="20" valign="top">m6_3a2</th>
      <th rowspan="10" valign="top">CTRL</th>
      <th>4</th>
      <td>4</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>2</td>
      <td>7</td>
      <td>6</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>8</th>
      <td>5</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>10</td>
      <td>...</td>
      <td>4</td>
      <td>7</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>4</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>...</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>6</td>
      <td>5</td>
      <td>5</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>8</td>
      <td>0</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>5</td>
      <td>3</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3</td>
      <td>4</td>
      <td>0</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
      <td>6</td>
      <td>3</td>
      <td>4</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>9</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>4</td>
      <td>...</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
      <td>5</td>
      <td>5</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>0</td>
      <td>7</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
      <td>6</td>
      <td>5</td>
      <td>4</td>
      <td>6</td>
      <td>0</td>
      <td>4</td>
    </tr>
    <tr>
      <th>64</th>
      <td>4</td>
      <td>4</td>
      <td>6</td>
      <td>3</td>
      <td>4</td>
      <td>0</td>
      <td>7</td>
      <td>3</td>
      <td>2</td>
      <td>5</td>
      <td>...</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>84</th>
      <td>4</td>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>6</td>
      <td>4</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>100</th>
      <td>7</td>
      <td>4</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
      <td>9</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">ADAPT</th>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>11</td>
      <td>11</td>
      <td>13</td>
      <td>15</td>
      <td>21</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3</td>
      <td>3</td>
      <td>18</td>
      <td>12</td>
      <td>12</td>
      <td>14</td>
      <td>18</td>
      <td>13</td>
      <td>7</td>
      <td>10</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>9</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>12</th>
      <td>7</td>
      <td>5</td>
      <td>16</td>
      <td>5</td>
      <td>12</td>
      <td>21</td>
      <td>18</td>
      <td>15</td>
      <td>12</td>
      <td>9</td>
      <td>...</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>4</td>
      <td>5</td>
      <td>17</td>
      <td>6</td>
      <td>21</td>
      <td>14</td>
      <td>13</td>
      <td>19</td>
      <td>10</td>
      <td>13</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>6</td>
      <td>7</td>
      <td>13</td>
      <td>7</td>
      <td>10</td>
      <td>21</td>
      <td>10</td>
      <td>19</td>
      <td>9</td>
      <td>8</td>
      <td>...</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>5</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>32</th>
      <td>1</td>
      <td>0</td>
      <td>10</td>
      <td>15</td>
      <td>10</td>
      <td>13</td>
      <td>19</td>
      <td>12</td>
      <td>12</td>
      <td>6</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2</td>
      <td>2</td>
      <td>18</td>
      <td>9</td>
      <td>15</td>
      <td>12</td>
      <td>21</td>
      <td>22</td>
      <td>12</td>
      <td>8</td>
      <td>...</td>
      <td>5</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>64</th>
      <td>2</td>
      <td>6</td>
      <td>8</td>
      <td>10</td>
      <td>19</td>
      <td>16</td>
      <td>18</td>
      <td>15</td>
      <td>9</td>
      <td>18</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>6</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>84</th>
      <td>1</td>
      <td>1</td>
      <td>11</td>
      <td>7</td>
      <td>9</td>
      <td>16</td>
      <td>12</td>
      <td>11</td>
      <td>12</td>
      <td>7</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>100</th>
      <td>1</td>
      <td>2</td>
      <td>10</td>
      <td>11</td>
      <td>16</td>
      <td>20</td>
      <td>19</td>
      <td>11</td>
      <td>15</td>
      <td>15</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>60 rows × 79 columns</p>
</div>



Now plot heat maps for all three neurons in one figure. Again you can adapt the code you used above for heat maps. Here, the figure should still have two columns (for conditions) but now three rows — one for each neuron. Again, don't forget that your data are now called `psth_df3`, so you need to update the code after you copy it to use the correct DataFrame, otherwise you will get errors.

<font color='red'>The PerformanceWarning you're receiving is due to a missing colon. Your `plt.imshow()` currently contains `psth_df3.loc[neuron, cond]`, but it should read `psth_df3.loc[neuron, cond, :]` to index all, and only, the relevant data. <br><br>Otherwise, aside from the `hot` cmap issue (mentioned above), these plots look great!</font>


```python
blue += [1.]
```


```python
fig = plt.figure(figsize=[15,15])
subplot_counter = 1 # used to track subplots

for neur in neuron_labels:
    for cond in cond_labels_2:
        ax = fig.add_subplot(len(neuron_labels), len(cond_labels_2), subplot_counter)
        aa = plt.imshow(psth_df3.loc[neur, cond], origin='lower', cmap='hot', interpolation='bilinear', aspect=5)
        
        cb = fig.colorbar(aa, shrink=0.33)
        cb.ax.set_ylabel('Spike Probability')

        # Nice formatting
        plt.title('Neuron ' + neur + ': ' + cond)

        xticks = range(0, len(time_bins), 10)
        plt.xticks(xticks,
                   time_bins[xticks])
        if neur == neuron_labels[-1]:
            plt.xlabel('Time (ms)')

        plt.yticks([x for x,y in enumerate(contr_labels)], 
                   [x for x in contr_labels])

        if cond == cond_labels[0]:
            plt.ylabel('Stimulus Contrast')
        subplot_counter += 1

plt.show()

```

    <ipython-input-75-29a62bc5e9f5>:7: PerformanceWarning: indexing past lexsort depth may impact performance.
      aa = plt.imshow(psth_df3.loc[neur, cond], origin='lower', cmap='hot', interpolation='bilinear', aspect=5)





    
![png](Assignment_4_files/Assignment_4_158_1.png)
    



## Interpretation Questions

In the control condition, do you see a relationship between stimulus presentation and spiking for any of the neurons? For now, look for any relationship *regardless* of contrast level. That is, does spike rate increase in a systematic way relative to stimulus onset? (remember when looking at heat maps that stimulus onset was at 2000 ms).

<font color='red'></font>


```python
blue += [1.]
```

The spike probability for the control condition is highest just after the stimulus onset (2000 ms), thus there does appear to be a relationship between the two.

One specific prediction from the top of the assignment is that in the control condition, latency to first spike is expected to decrease with increasing contrast. Do you see evidence of that for the first neuron you worked with (the one you plotted PSTHs for)?

<font color='red'>Note that the question asks about contrast levels' effect on latency to fist spike (time domain), not probability. You already spoke to the latter in the previous question.</font>


```python
blue += [0.]
```

Yes, I do see evidence of this for the first neuron, as the occurrence of a spike immediately after stimulus onset becomes much more probable as stimulus contrast increases

Do you see evidence that the stimulus contrast affects spike rates in any other way? If so, what is the evidence?

<font color='red'></font>


```python
blue += [1.]
```

In the second and third neurons, there appear to be two areas concentrated with high spike probability in close temporal proximity to each other rather than just the one

Is there a *linear* or *nonlinear* relationship between contrast and spike rate? A linear relationship would be one where each step in contrast generates an equal-sized increase in spike rate. A nonlinear relationship would be one in which the change in spike rate from one contrast level to the next, is different at different contrast levels. 

<font color='red'></font>


```python
blue += [1.]
```

The relationship appears nonlinear, as the increase in spike probability is not equal to the increase in stimulus intensity, despite their positive correlation

You should see that in all the neurons, spiking starts sooner in the adaptation condition than in the control condition. Why do you think that is? 

<font color='red'></font>


```python
blue += [1.]
```

I think this is because in the adaptation condition, before stimulus onset, the participant was shown an image similar to the stimulus image, only at a lower contrast.  The neurons would have responded to that stimulus, thus showing spikes before the onset of the real stimulus.

In the adaptation condition, note that the neurons do not show a similar or systematic response to the onset of the "main" stimulus (the one that starts at 2000 ms) as in the control condition. Why do you think this is?

<font color='red'>Excellent.</font>


```python
blue += [1.]
```

 I think this is because in the adaptation condition, the "pre-stimulus" period contained a stimulus of its own, despite being at a lower contrast. The neurons got used to the initial, lighter stimulus, and were not subjected to as sharp of a change as in the control condition. you can see especially in the third neuron that as the pre-stimulus continued, the firing probability decreased as the neuron adapted to the stimullus and stopped firing as much.

One of the predictions at the top of this assignment was that some primary visual cortex cells are phase sensitive (so-called **simple cells**), so in response to a drifting sine wave grating they will oscillate between excitation and inhibition at the same temporal frequency of the stimulus (2 Hz in this case; rem,ember that 2 Hz means two peaks per second). Do you see any simple cells among the neurons you worked with here? Which neuron(s), and why? 

<font color='red'>Good observation. Well done throughout. </font>


```python
blue += [1.]
```

Neuron m6_3a2 could pootentially be a simple cell as it exhibits multiple spike times in what appears to be a repeating pattern.

# THE END

# 

## Grade


```python
print(blue)
print(len(blue))
print(np.sum(blue))
```

    [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.75, 1.0, 1.0, 1.0, 1.0, 0.75, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.5, 1.0, 1.0, 0.75, 0.75, 0.75, 1.0, 1.0, 1.0, 0.75, 1.0, 1.0, 1.0, 1.0, 0.0, 1.0, 1.0, 1.0, 1.0, 1.0]
    39
    36.0



```python
Grade = np.sum(blue)/len(blue)
print('Grade: ' + str(round(Grade*100, 2)))
print('XP: ' + str(int(Grade * 3000)))
```

    Grade: 92.31
    XP: 2769



```python

```

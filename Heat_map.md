# Heat Maps to Visualize Neuron Temporal Response Data

I started by reading data from three mouse visual cortex neurons into a dataframe.

```python
df3 = pd.read_csv('crowder_3_neurons.csv')
```

I assigned the names of the three neurons, as well as the condition and contrast labels to lists.


```python
# assign condition/contrast labels
cond_labels_2 = df3['condition'].unique()
contr_labels_2 = sorted(df3['contrast'].unique())

# assign neuron labels
neuron_labels = sorted(df3['neuron'].unique())

print(neuron_labels)
```

    ['m1_6', 'm3_11', 'm6_3a2']


I created a new dataframe that contained the neuron, condition, and contrast labels, using the lists created above.

```python


# create dataframe that combines the 
condition_labels_2 = pd.DataFrame([[neur, cond, contr] for neur in neuron_labels for cond in cond_labels_2 for contr in contr_labels_2], columns=['neuron', 'condition', 'contrast'])
```

I created a DataFrame of PSTHs from the neuronal response data.

```python
psth_df3_list = []
for neur in neuron_labels:
    for cond in cond_labels_2:
        for contr in contr_labels_2:
            spike_times = df3[(df3['neuron']==neur) & (df3['condition']==cond) & (df3['contrast']==contr) & (df3['spike']==1)]['time']
            hist, bins = np.histogram(spike_times, bins=time_bins)
            psth_df3_list.append(hist)
```

Finally, I concatenated the conditions dataframe with the response dataframe, setting the conditions as indexes.


```python
psth_df3 = pd.concat([condition_labels_2, pd.DataFrame(psth_df3_list)], axis=1) 
psth_df3 = psth_df3.set_index(['neuron', 'condition', 'contrast'])
```
    
I then used the dataframe to plot heat maps for all three neurons at CTRL and ADAPT conditions.

```python
fig = plt.figure(figsize=[15,15])
subplot_counter = 1 # used to track subplots

for neur in neuron_labels:
    for cond in cond_labels_2:
        ax = fig.add_subplot(len(neuron_labels), len(cond_labels_2), subplot_counter)
        aa = plt.imshow(psth_df3.loc[neur, cond, :], origin='lower', cmap='hot', interpolation='bilinear', aspect=5)
        
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
<img src="Heat-map.png" width="640" />

Note: code written in python; data courtesy of Dr. Nathan Crowder, Department of Psychology & Neuroscience, Dalhousie University.

I started by creating a list of subject IDs from the data files.

```python
subjects = sorted(glob.glob('nagl*'))
```

I then used this list of IDs to read in each subject's data into a dictionary.

```python
evoked = {}
for subject in subjects:
    evoked[subject] = mne.read_evokeds(subject + '/' + subject + '-ave.fif')

print(evoked)
```

    {'nagl002': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=21), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl003': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl004': [<Evoked | 'Quiet/Control/Correct' (average, N=31), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl005': [<Evoked | 'Quiet/Control/Correct' (average, N=10), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=10), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl006': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl007': [<Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl008': [<Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=28), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl009': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl010': [<Evoked | 'Quiet/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl016': [<Evoked | 'Quiet/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl017': [<Evoked | 'Quiet/Control/Correct' (average, N=29), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=29), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=24), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl018': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl019': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=31), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=27), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl020': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl021': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl026': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl028': [<Evoked | 'Quiet/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl029': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl030': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=18), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl031': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=22), [-0.19922, 1] sec, 64 ch, ~479 kB>]}

I created a list of the conditions for the experiment.

```python
conditions  = [x.comment for x in evoked[subjects[0]]]

conditions
```
 ```
    ['Quiet/Control/Correct',
     'Quiet/Violation/Correct',
     'Noise/Control/Correct',
     'Noise/Violation/Correct']
```
I then calculated the grand averages for each condition.
```python
gavg = {}

for key, val in enumerate(conditions):
    gavg[val] = mne.grand_average([evoked[subj][key] for subj in evoked.keys()])

gavg
```

    {'Quiet/Control/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Quiet/Violation/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Noise/Control/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Noise/Violation/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>}

I then generated a waveform plot of the ERP data

```python
Cz_idx = gavg['Quiet/Control/Correct'].ch_names.index('Cz')

mne.viz.plot_compare_evokeds(gavg, picks=Cz_idx)
plt.show()
```
![png](Project_2_files/Project_2_34_0.png)

I then plotted a topographic map of each condition's ERP response at 0.500 seconds

```python
for condition in gavg:
    gavg[condition].plot_topomap(.500, average=.200, vmin=-3, vmax=3, title = condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_40_0.png)
    






    
![png](Project_2_files/Project_2_40_1.png)
    






    
![png](Project_2_files/Project_2_40_2.png)
    






    
![png](Project_2_files/Project_2_40_3.png)

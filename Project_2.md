# Project 2
## NESC 3505

## Team Members and Contributions

In the cell below, list each team member and the contributions they made to completing this assignment

| <strong> Team Member </strong>     | <strong>Contribution</strong> |
| ----------- | ----------- |
| <strong>Anna Torres</strong>  | Reading in and examining the data, creating grand averages, vizualizing: ERP Waveforms and Topographic maps, plotting mean amplitudes over time windows of interest, interpreting data and proof-reading  |
| <strong>Caroline Damus</strong>  | Creating difference wave dictionary, visualizing difference waves: ERP Waveforms and Topographic maps, statistics, discussion, interpreting data, and proof-reading  |
| <strong>Emily Gosselin</strong>  | Creating difference wave dictionary, visualizing difference waves: ERP Waveforms and Topographic maps, statistic, interpreting data and proof-reading |
| <strong>Noa Maianski</strong>  | Reading in and examining the data, creating grand averages, vizualizing: ERP Waveforms and Topographic maps, plotting mean amplitudes over time windows of interest, interpreting data and proof-reading   |
| <strong>Thomas Pope</strong>  | Reading in and examining the data, creating grand averages, vizualizing: ERP Waveforms and Topographic maps, Discussion, interpreting data and proof-reading|

# EEG/ERP data

For general background on EEG/ERP data, please read the [textbook chapter](https://dalpsychneuro.github.io/NESC_3505_textbook/eeg/introduction.html). If you're rushed for time, you can skip the Preprocessing sections, because these steps have already been applied to the data you'll be working with.

## N400 – Artificial Grammar Learning (NAGL) Experiment

The data for this project are from an EEG/ERP study run in my lab. In the full study, we collected data in two experimental paradigms (N400 and artificial grammar learning), with the aim of finding out whether individual differences in brain activity across the two paradigms were correlated. For this project, we will work only with the N400 data.

The N400 is an ERP component associated with processing the meanings of words (*semantic* information). It was first discovered more than 40 years ago (Kutas & Hillyard, 1980), contrasting sentences that ended with a *semantic violation* (a word that doesn't make sense given the prior context of the sentence, e.g., *I take my coffee with milk and **dog**.*) with meaningful control sentences (e.g., *I take my coffee with milk and **sugar**.*). The N400 manifests as a more negative ERP potential, particularly over the top (vertex) of the head, for violation than control sentences, with the largest difference usually around 400-600 ms after the onset of the violation word. 

The N400 has been studied extensively since then (Kutas & Federmeier, 2011) and the general consensus is that it reflects brain activity associated with integrating incoming information into one's ongoing "context". That is, as we're reading or listening to language, we create a mental representation of the topic (the context), and try to integrate the meaning of each new word into that context. Words that don't make sense are harder (or impossible) to integrate, so our brains engage in more of this "semantic integration" activity, leading to a larger N400. 

In the present study, we were interested in whether semantic integration changes when there is background noise that makes the sentences harder to hear and understand. So, we presented auditory sentences to participants, with half of the sentences ending with semantic violations. In the first half of the experiment, the sentences were presented with no background noise, but in the second half of the experiment, the sentences were presented with background noise ("multi-talker babble"; the sound of several people talking at once, without any of the background speech being understandable). 

While the study that these data are from involved another paradigm and some complex predictions, for the purpose of your present project, we will state the hypotheses as follows:

- there will be an N400 in the quiet condition, with violation sentences eliciting more negative ERPs than control sentences between 400-600 ms 

- there will also be an N400 in the quiet condition, with violation sentences eliciting more negative ERPs than control sentences between 400-600 ms 

- the N400 in response to violations will be significantly smaller in the noise than quiet conditions, because more cognitive effort will be directed to filtering out noise, leaving fewer resources to process the semantic anomaly.

### Your Tasks

Test the hypotheses above using
- topographic scalp maps in time windows of interest
- waveform plots at electrodes of interest
- Seaborn plots of mean amplitudes for conditions/contrasts within ROIs
- t-tests at selected electrodes

### MNE

MNE is a Python package for working with EEG and MEG data, and is described in the [textbook](https://dalpsychneuro.github.io/NESC_3505_textbook/eeg/mne-python.html). You will also find the [MNE API](https://mne.tools/stable/python_reference.html) useful in doing this project; the API has a search function which makes it easy to look up the commands that you're instructed to use in the steps below. You won't have to find/use any MNE commands other than the ones explicitly mentioned in this project, but you may have to consult the API to perform some of the steps below.

Beyond the general introduction to MNE in the textbook, and the MNE API for specifci commands you're provided with here, you should find the documentation in this project (combined with your past knowledge of Python) sufficient to complete it. There is no expectation that you will need to learn MNE or read its documentation beyond what's stated here. 

### References

Kutas, M., Federmeier, K. (2011). Thirty Years and Counting: Finding Meaning in the N400 Component of the Event-Related Brain Potential (ERP). *Annual Review of Psychology*  62(1), 621 647. https://dx.doi.org/10.1146/annurev.psych.093008.131123

Kutas, M., Hillyard, S. (1980). Reading senseless sentences: brain potentials reflect semantic incongruity. *Science*  207(4427), 203-205. https://dx.doi.org/10.1126/science.7350657

#### NOTE:

Some MNE functions produce *lots* of output. On CoCalc, you can force the output of a cell to be a small box wiht a scroll bar, rather than a gigantic box. To do this, with the cell selected, go to the *Cell* menu and selected *Toggle scrolled*.</p>



```python
import numpy as np
import pandas as pd
import os, glob

import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns

import mne
```

## Read in the data

There is one data file per subject, named in the format [subjID]-ave.fif. For example, the first subject is `nagl001` and their data file is `nagl001-ave.fif`. Each data file is in a folder named for that subject (e.g., `nagl002/nagl001-ave.fif`).

Create a list of the subject IDs (i.e., the directory names), using `glob.glob()` 


```python
subjects = sorted(glob.glob('nagl*'))
```

The following will print a sorted list of subject IDs that were found:


```python
sorted(subjects)
```




    ['nagl002',
     'nagl003',
     'nagl004',
     'nagl005',
     'nagl006',
     'nagl007',
     'nagl008',
     'nagl009',
     'nagl010',
     'nagl016',
     'nagl017',
     'nagl018',
     'nagl019',
     'nagl020',
     'nagl021',
     'nagl026',
     'nagl028',
     'nagl029',
     'nagl030',
     'nagl031']



Print the length of `subjects`, which will tell you how many data sets you'll be working with


```python
len(subjects)
```




    20



Define an empty dictionary called `evoked`.

Loop through subjects, and for each, read in the subject's data file using `mne.read_evokeds()`. You'll need to specify the whole path and input file name using regular expressions or getting it with `glob.glob()`.

Assign each data file that you read to an entry in the `evoked` dictionary. The key for each dictionary entry should be the subject ID (e.g., `nagl001`). 



```python
evoked = {}
for subject in subjects:
    evoked[subject] = mne.read_evokeds(subject + '/' + subject + '-ave.fif')
```

    Reading nagl002/nagl002-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 37 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 21 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl003/nagl003-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 26 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl004/nagl004-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 31 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 32 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 20 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl005/nagl005-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 10 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 10 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 30 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl006/nagl006-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 37 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 30 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl007/nagl007-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 41 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 35 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl008/nagl008-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 41 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 28 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl009/nagl009-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 32 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 26 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl010/nagl010-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 34 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl016/nagl016-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 26 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl017/nagl017-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 29 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 29 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 24 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 25 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl018/nagl018-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 34 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 26 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl019/nagl019-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 31 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 27 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl020/nagl020-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 34 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 25 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl021/nagl021-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 37 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 30 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 35 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl026/nagl026-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 38 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl028/nagl028-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 32 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 35 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 25 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 26 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl029/nagl029-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 40 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 35 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl030/nagl030-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 37 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 36 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 34 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 18 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
    Reading nagl031/nagl031-ave.fif ...
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Control/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Quiet/Violation/Correct)
            0 CTF compensation matrices available
            nave = 39 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Control/Correct)
            0 CTF compensation matrices available
            nave = 33 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied
        Found the data of interest:
            t =    -199.22 ...    1000.00 ms (Noise/Violation/Correct)
            0 CTF compensation matrices available
            nave = 22 - aspect type = 100
    No projector specified for this dataset. Please consider the method self.add_proj.
    No baseline correction applied


## Examine the data structure

Show the `evoked` dictionary:


```python
print(evoked)
```

    {'nagl002': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=21), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl003': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl004': [<Evoked | 'Quiet/Control/Correct' (average, N=31), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl005': [<Evoked | 'Quiet/Control/Correct' (average, N=10), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=10), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl006': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl007': [<Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl008': [<Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=28), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl009': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl010': [<Evoked | 'Quiet/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl016': [<Evoked | 'Quiet/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl017': [<Evoked | 'Quiet/Control/Correct' (average, N=29), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=29), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=24), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl018': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl019': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=31), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=27), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl020': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl021': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=30), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl026': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl028': [<Evoked | 'Quiet/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=25), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=26), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl029': [<Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=35), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl030': [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=36), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=34), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=18), [-0.19922, 1] sec, 64 ch, ~479 kB>], 'nagl031': [<Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Quiet/Violation/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>, <Evoked | 'Noise/Violation/Correct' (average, N=22), [-0.19922, 1] sec, 64 ch, ~479 kB>]}


So you have a dictionary whose keys are subject IDs, and the value of each dictionary entry is the data for that subject. This means that you can select the data for a particular subject using the subject's ID code (a string):


```python
evoked['nagl002']
```




    [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Noise/Violation/Correct' (average, N=21), [-0.19922, 1] sec, 64 ch, ~479 kB>]



However, the value of each subject's dictionary entry is a list of Evoked objects, one for each condition. 

Importantly, this means that although you can access an individual subject's data by their ID code (the dictionary key), the data for each condition is in a list, so you need to access individual condition's data using list indices (integers).

Show the values for the first subject in the `evoked` dictionary. Rather than hard-coding a specific subject ID, you should use the first entry in the `subjects` list.


```python
evoked[subjects[0]]
```




    [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Violation/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Noise/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Noise/Violation/Correct' (average, N=21), [-0.19922, 1] sec, 64 ch, ~479 kB>]



This shows you that each subject's data consists of a list of MNE `Evoked` objects. There is one list entry for each experimental condition; the name of the condition comes after the "pipe" `|` symbol, in quotes. Each entry also lists the number of experimental trials that contributed to the averaged data here, as well as the time range of the epoch (with negative numbers indicating times prior to stimulus onset), the number of channels (electrodes) we have data for, and the size of the data in kiloBytes.

Conveniently (/by design), the conditions are listed in the same order for every subject, which makes it easy to iterate over subjects to access the data for a specific condition.

You can't see the contents of an Evoked object just by typing it's name; if you try that below for the first list entry (condition) for the first subject, you'll see you just get output like in the list of all conditions for that subject above. Try it anyway (ask for the first item in the first subject's dictionary entry):


```python
evoked[subjects[0]][0]
```




    <Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>



The Evoked object does have some properties that you can view though. For example, the condition name is stored as the `.comment` property. Show the condition name for the first condition, for the first subject:


```python
evoked[subjects[0]][0].comment
```




    'Quiet/Control/Correct'



Since the value of each subject's dictionary entry is a list, we can get the number of conditions by asking for the length, `len()`, of the list, for the first subject. Do this below:


```python
len(evoked[subjects[0]])
```




    4



Create a list of condition names, which will come in handy later. Call the list `conditions`.

You can use the `.comment` properties of all the list entries for one subject, to create a list of experimental condition names. You can pick any subject, e.g., `subject[0]`.

Use list comprehension to create this list. The `for` part of the list comprehension should iterate through the list of Evokeds for one subject. You can use the `range()` function combined with the length of the list of conditions (the cell right above this one) to do this iteration.

Whereas above (your answer two cells earlier) you should have used a numerical index to get the first list entry for the first subject, in list comprehension you'll replace this by a variable that you iterate through. 


```python
conditions  = [x.comment for x in evoked[subjects[0]]]

conditions
```




    ['Quiet/Control/Correct',
     'Quiet/Violation/Correct',
     'Noise/Control/Correct',
     'Noise/Violation/Correct']



This makes it easy to make a list of conditions, which we can then iterate over as necessary. Create a list called `conditions`, using list comprehension, that contains the condition labels.

## Grand Averages
In ERPs, a "grand average" is an average across subjects, for each condition.

The function to create a grand average is `mne.grand_average()`, with the argument being a list of Evoked objects, from each subject, for one condition. For example: 

    mne.grand_average([evoked['nagl001'][0], 
                       evoked['nagl002'][0], 
                       evoked['nagl003'][0]
                       ]
                      )

We want to create grand averages for each condition, saving the results in a dictionary called `gavg` that is keyed by condition label. 

To do this, we need to loop through conditions, and for each condition we need to give `mne.grand_average()` a list of all the `Evoked` objects corresponding to that condition. 

We'll do this in steps, so it's not too overwhelming. 

First, figure out how to use list comprehension to get the list of all `Evoked` objects for *one* condition (i.e., loop through all subjects and select the data for that condition). 

Below is some sample code that will create a list of all the `Evoked` objects for the first condition: 


```python
[evoked[subj][0] for subj in evoked.keys()]
```




    [<Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=31), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=10), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=41), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=33), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=38), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=29), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=32), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=40), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=37), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     <Evoked | 'Quiet/Control/Correct' (average, N=39), [-0.19922, 1] sec, 64 ch, ~479 kB>]



You need to put this code in a `for` loop that loops through the list of condition labels. Since the entries for each subject are a list, you need to index the conditions in that list numerically; we can't use their names. So  use `enumerate()` to get both the position in the list (index) and the condition label. If you need hints on using `enumerate()`, try [this link](https://realpython.com/python-enumerate/#using-pythons-enumerate).

Inside the `for` loop, use the list comprehension above as the input to `mne.grand_average()`. You'll need to change the `[0]` in the list comprehension above to instead be the index of your `for` loop. Assign the output of mne.grand_average to a dictionary entry in `gavg`, whose key is the condition label.


```python
gavg = {}

for key, val in enumerate(conditions):
    gavg[val] = mne.grand_average([evoked[subj][key] for subj in evoked.keys()])

gavg
```

    Identifying common channels ...
    Identifying common channels ...
    Identifying common channels ...
    Identifying common channels ...





    {'Quiet/Control/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Quiet/Violation/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Noise/Control/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Noise/Violation/Correct': <Evoked | 'Grand average (n = 20)' (average, N=20), [-0.19922, 1] sec, 64 ch, ~479 kB>}



## Visualization: ERP Waveforms

Now that we have grand averages, we can visualize them! MNE provides a number of functions for visualizing ERPs. We'll start with `mne.viz.plot_compare_evokeds()`, which plots ERPs as waveforms: voltage on the *y* axis and time on the *x* axis.

We'll plot the waveforms for a single electrode, Cz, which is located at the vertex (top) of the head. The N400 is typically largest at this location, so it is a good one to plot. Somewhat annoyingly,  `mne.viz.plot_compare_evokeds()` does not allow you to specify electrodes by name, but only by their index in the list of channels. So the first line below finds this value and assigns it to `Cz_idx`, which we use in this plot, and that you can use in others below.

For reference (you will find this useful in doing the work below), [the API for `mne.viz.plot_compare_evokeds()` is here](https://mne.tools/stable/generated/mne.viz.plot_compare_evokeds.html?highlight=plot_compare_evokeds#mne.viz.plot_compare_evokeds).

For now, just generate the plot with the code below, and note the differences between conditions. Below, you'll need to adapt this code for another waveform plot.


```python
Cz_idx = gavg['Quiet/Control/Correct'].ch_names.index('Cz')

mne.viz.plot_compare_evokeds(gavg, picks=Cz_idx)
plt.show()
```




    
![png](Project_2_files/Project_2_34_0.png)
    



## Visualization: Topographic maps

A second way of viewing ERP data is how the voltage is distributed across the scalp. Above, with the waveform plot we visualized the data from one electrode, over the entire time period of the ERP. In a "topo map", we view all electrodes, but for a specific point in time (or, averaged over a time range). Both types of visualization are useful: waveform plots can help you see when the effects are largest (and especially, when differences occur) over time, but you have to know which electrode(s) to look at. Topo maps, in contrast, allow you to figure out where on the scalp the effects are biggest, but you have to specify time. Or, you can plot a series of topo maps over a range of times. 

First, we'll plot a single topo map. Since the N400 above is largest from approximately 400-600 ms, we will average over that time range. 

We use MNE's `Evoked.plot_topomap()` method to create topo maps. Note that, while `mne.viz.plot_compare_evokeds()` is a *function*, `Evoked.plt_topomap()` is a *method*. We write it with `Evoked`  capitalized, to indicate that `Evoked` is a class and not a function name (if you want a refreshed on the use of capitalization in Python, refer back to this [Style Note](https://dalpsychneuro.github.io/NESC_3505_textbook/single_unit/intro_spike_trains.html) in the textbook). So to make a topo map, you will need to replace `Evoked` with the specification of the dictionary entry from `gavg` that you wish to plot. 

[The API for `Evoked.plot_topomap()` is here](https://mne.tools/stable/generated/mne.Evoked.html?highlight=plot_topomap#mne.Evoked.plot_topomap). But in brief, the first (and only required) argument to the command is the time (or set of times) that you want to plot the data at. This is specfied in *seconds*, so 500 ms would be `.500`. Optionally, you can add a `average=` argument, which will cause MNE to plot the average voltage over the time range you specify (again, in seconds). So below we plot the data from 400-600 ms, by specifying the  midpoint time in that range (.500) and averaging over 200 ms:


```python
gavg['Quiet/Control/Correct'].plot_topomap(.500, average=.200)
plt.show()
```




    
![png](Project_2_files/Project_2_36_0.png)
    



Note that MNE's default colour map for topo plots is `seismic`. This is an excellent choice in this case, because `seismic` has white at the centre of its range, with positive values mapping to increasingly dark reds, and negative values mapping to increasingly dark blues. In this way, the brain naturally sees the areas of low/zero voltage as 'insignificant' (white), and positive and negative voltages are clearly differentiated by distinct colours. 

Below, put the plot command from above inside a loop, that will cycle through the keys of `gavg` and plot topo maps for each of the 4 conditions (Quiet/Control, Quiet/Violation, Noise/Controk, and Noise/Violation). Note that with MNE, there is no need to define a figure ahead of time, or for subplot specifications, as long as you don't mind each plot below the other.


```python
for condition in gavg:
    gavg[condition].plot_topomap(.500, average=.200, title = condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_38_0.png)
    






    
![png](Project_2_files/Project_2_38_1.png)
    






    
![png](Project_2_files/Project_2_38_2.png)
    






    
![png](Project_2_files/Project_2_38_3.png)
    



One issue with the plot above is that each topo map is scaled differently, which you can see by the different numbers on the colour bar. This makes it hard to comapre the plots, because colours map to voltages of different sizes in each plot. We can specify the minimum and maximum values for each plot to be the same, using the `vmin` and `vmax` arguments (see the API for details).

In the cell below, copy and paste your code from the previous cell, and re-plot all four conditions with a range of -3 to 3 microVolts.


```python
for condition in gavg:
    gavg[condition].plot_topomap(.500, average=.200, vmin=-3, vmax=3, title = condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_40_0.png)
    






    
![png](Project_2_files/Project_2_40_1.png)
    






    
![png](Project_2_files/Project_2_40_2.png)
    






    
![png](Project_2_files/Project_2_40_3.png)
    



You can also plot a series of time points with `Evoked.plot_topomap`. To do this, you simply specify the first argument as a list of times, rather than a single value. Below, copy your plotting loop from the previous cell, and adapt it so that each plot is a series of times from 0 to 900 ms (*inclusive*, meaning you should include a plot at 900 ms), in 100 ms increments. The best way to do this is to define a list with the time points you want to plot, before looping through conditions, nad use this list as the first argument to `.plot_topomap()`. Use the `np.arange()` function to do this, as the `range()` function only works with integers, not floats. 

You only need to specify one value for `average=`, since you will want to average over the same amount of time for each time point. However, in this case average over 100 ms rather than 200 ms


```python
time = np.arange(0, 1.000, 0.100)

for condition in gavg:
    gavg[condition].plot_topomap(time, average=.100, vmin=-3, vmax=3, title=condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_42_0.png)
    






    
![png](Project_2_files/Project_2_42_1.png)
    






    
![png](Project_2_files/Project_2_42_2.png)
    






    
![png](Project_2_files/Project_2_42_3.png)
    



## Difference waves

Key to testing our hypotheses are contrasts between conditions. Specifically, the N400 is defined as Violation - Control, so we want to subtract the control grand average from the violation grand average, for each condition (Quiet and Noise). This way, we can visualize the *difference waveform* rather than just each condition separately.

The dictionary below specifies the contrasts of interest.


```python
contrasts = {'Quiet_V-C' : ['Quiet/Violation/Correct', 'Quiet/Control/Correct'],
             'Noise_V-C' : ['Noise/Violation/Correct', 'Noise/Control/Correct']
            }
```

Create a dictionary called `gavg_diff` that contains the *difference waves* for each contrast. Use `mne.combine_evoked()`, with the first argument being a list of the two conditions you want to compare (with a minus sign preceding the second item in the list, so that it is subtracted from the first), and the second argument being `weights='equal'`. The skeleton of this code has been provided to help guide you.


```python
gavg_diff = {}

for value in contrasts:
    gavg_diff[value] = mne.combine_evoked([gavg[contrasts[value][0]], -gavg[contrasts[value][1]]], weights='equal')

gavg_diff
```




    {'Quiet_V-C': <Evoked | '0.500 × Grand average (n = 20) + 0.500 × -Grand average (n = 20)' (average, N=40.0), [-0.19922, 1] sec, 64 ch, ~479 kB>,
     'Noise_V-C': <Evoked | '0.500 × Grand average (n = 20) + 0.500 × -Grand average (n = 20)' (average, N=40.0), [-0.19922, 1] sec, 64 ch, ~479 kB>}



## Visualizing Difference Waves

Now that you have the difference waves, you can use them as the input to `mne.viz.plot_compare_evokeds()` and `Evokped.plot_topomap()`. 

In the cell below, plot the waveforms for the two differences (for the Quiet and Noise conditions) in a single waveform plot, at electrode Cz:


```python
mne.viz.plot_compare_evokeds(gavg_diff, picks=Cz_idx, title = "Waveform Differences for Quiet and Noise Conditions at Electrode Cz")
plt.show()
```




    
![png](Project_2_files/Project_2_48_0.png)
    



In the cell below, plot the topomaps for each difference wave, averaged over the time range 400-600 ms:


```python
for condition in gavg_diff:
    gavg_diff[condition].plot_topomap(0.500, average=0.200, vmin=-3, vmax=3, title=condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_50_0.png)
    






    
![png](Project_2_files/Project_2_50_1.png)
    




```python
for condition in gavg_diff:
    gavg_diff[condition].plot_topomap(0.500, average=0.200, vmin=-1.5, vmax=1.5, title=condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_51_0.png)
    






    
![png](Project_2_files/Project_2_51_1.png)
    



In the cell below, plot the difference waveforms for each of the two conditions, as a series of times from 0 to 900 ms (*inclusive*, meaning you should include a plot at 900 ms), in 100 ms increments, with each averaged over a 100 ms time window.


```python
time = np.arange(0, 1.000, 0.100)

for condition in gavg_diff:
    gavg_diff[condition].plot_topomap(time, average=.100, vmin=-3, vmax=3, title=condition)
    plt.show()
```




    
![png](Project_2_files/Project_2_53_0.png)
    






    
![png](Project_2_files/Project_2_53_1.png)
    



---
## Plot mean amplitudes over time windows of interest

While we can eyeball the differences using topomaps and waveform plots, averaging over the time windows of interest and plotting the means as box plots or bar graphs can be useful as well. This is easy to do using Seaborn, as you have done in the past with data in pandas DataFrames. Fortunately, MNE provides the `Evoked.to_data_frame()` method to convert MNE Evoked data to a pandas DataFrame. 

### Extract data to Pandas dataframe
In the cell below, write a loop to cycle through subjects and conditions in `evoked` and extract the data from each as a pandas DataFrame. You'll need to use a pair of nested `for` loops, first to go through subjects, then the second to enumerate through conditions (as you did above in creating `gavg`). Use the following arguments to `Evoked.to_data_frame`: `time_format='ms', picks=Cz_idx, long_format=True`. 

Inside the loop, after you create each subject/condition DataFrame, you'll want to add two columns, `subject` and `condition`, to indicate which subject and condition the data is from. Note that pandas will broadcast a single value to an entire column, which makes this step easy.  

Append the DataFrames for each subject/condition to `df_list` and then, once you have looped through all subjects and conditions, concatenate them into one DataFrame called `df`. 


```python
df_list = []

# looping through subjects and conditions
for subject in subjects:
    for key, val in enumerate(conditions):
        #creating df per subject-key
        df_ = (evoked[subject][key].to_data_frame(time_format = 'ms', picks = Cz_idx, long_format=True))
        df_['subject'] = subject; df_['condition'] = val
        df_list.append(df_)

df = pd.concat(df_list)
```

    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...
    Converting "channel" to "category"...
    Converting "ch_type" to "category"...


Use the pandas `.sample()` method to show 10 random rows from `df`, to get a sense of what your result looks like. You should see columns for time, channel, ch_type, value, subject, and condition.


```python
df.sample(n=10)
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
      <th>channel</th>
      <th>ch_type</th>
      <th>value</th>
      <th>subject</th>
      <th>condition</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>485</th>
      <td>748</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-2.986966</td>
      <td>nagl017</td>
      <td>Noise/Violation/Correct</td>
    </tr>
    <tr>
      <th>177</th>
      <td>146</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.866771</td>
      <td>nagl004</td>
      <td>Quiet/Violation/Correct</td>
    </tr>
    <tr>
      <th>116</th>
      <td>27</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-1.378665</td>
      <td>nagl020</td>
      <td>Quiet/Violation/Correct</td>
    </tr>
    <tr>
      <th>299</th>
      <td>385</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.177851</td>
      <td>nagl016</td>
      <td>Noise/Violation/Correct</td>
    </tr>
    <tr>
      <th>598</th>
      <td>969</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-0.616956</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
    </tr>
    <tr>
      <th>204</th>
      <td>199</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-3.928756</td>
      <td>nagl007</td>
      <td>Quiet/Violation/Correct</td>
    </tr>
    <tr>
      <th>156</th>
      <td>105</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-1.725997</td>
      <td>nagl009</td>
      <td>Quiet/Control/Correct</td>
    </tr>
    <tr>
      <th>539</th>
      <td>854</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-1.418990</td>
      <td>nagl007</td>
      <td>Noise/Control/Correct</td>
    </tr>
    <tr>
      <th>456</th>
      <td>691</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-0.388106</td>
      <td>nagl020</td>
      <td>Noise/Control/Correct</td>
    </tr>
    <tr>
      <th>492</th>
      <td>762</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-0.694198</td>
      <td>nagl020</td>
      <td>Quiet/Violation/Correct</td>
    </tr>
  </tbody>
</table>
</div>



The `condition` column isn't really in the form we want. This experiment used a 2 x 2 design, with the variables sentence type (Control, Violation) and background condition (Quiet, Noise). So to properly visualize the data, we'll want to group by each of those variables separately, which is not easy to do when both variables are encoded in one column.

Fortunately, pandas has a `.str.split()` method which allows you to "split" the strings in a column based on a particular character. For example, to split on the `/` character, we can use `df['condition'].str.split('/')`. By default, the output is a list for each row, with the number of entries equal to the number of strings separated by `/`s. However, if we add the `expand=True` argument, then the output is a column for each part of the split:


```python
df['condition'].str.split('/', expand=True)
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>610</th>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>611</th>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>612</th>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>613</th>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>614</th>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
  </tbody>
</table>
<p>49200 rows × 3 columns</p>
</div>



Since this generates a DataFrame, with the same number of rows as `df`, we can merge the output of `.str.split()` with `df` to create a single DataFrame. Do this below. First, split the `condition` column and assign the output to a new DataFrame called `df_labels`. Then, rename the columns from 0, 1, 2 to `background`, `sen_type`, and `correct`. Then, concatenate `df` and `df_labels`, by columns, with the output being assigned to a new DataFrame called `df_all`. Finally, print a sample of 10 rows from `df_all` to confirm things worked right.


```python
#creating extracting conditions into new df
df_labels = df['condition'].str.split('/', expand=True)
df_labels.columns = ['background', 'sen_type', 'correct']

#concatenating dfs
df_all = pd.concat([df, df_labels], axis=1, join='inner')

df_all
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
      <th>channel</th>
      <th>ch_type</th>
      <th>value</th>
      <th>subject</th>
      <th>condition</th>
      <th>background</th>
      <th>sen_type</th>
      <th>correct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-199</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.068257</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-197</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.056280</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-195</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.022480</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-193</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-0.031840</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-191</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>-0.104097</td>
      <td>nagl002</td>
      <td>Quiet/Control/Correct</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>610</th>
      <td>992</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.697847</td>
      <td>nagl031</td>
      <td>Noise/Violation/Correct</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>611</th>
      <td>994</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.822720</td>
      <td>nagl031</td>
      <td>Noise/Violation/Correct</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>612</th>
      <td>996</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>0.963199</td>
      <td>nagl031</td>
      <td>Noise/Violation/Correct</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>613</th>
      <td>998</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>1.121566</td>
      <td>nagl031</td>
      <td>Noise/Violation/Correct</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
    <tr>
      <th>614</th>
      <td>1000</td>
      <td>Cz</td>
      <td>eeg</td>
      <td>1.298880</td>
      <td>nagl031</td>
      <td>Noise/Violation/Correct</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>Correct</td>
    </tr>
  </tbody>
</table>
<p>49200 rows × 9 columns</p>
</div>



### Compute mean amplitude over a time window

Create a new DataFrame called `df_means` that contains the mean ERP amplitude value for each subject,sentence type, and background condition, averaged over the N400 time window of interest (400-600 ms; we've defined this time window for you below as a tuple).

To do this, you'll need to use chaining. You can use `.loc()` on the `time` column to specify a combination of Boolean selectors to get the time range. You can then chain this with `.groupby()` to group the data by specific columns that matter. Finally, the `.mean()` method will compute 

You'll want to drop the original 'time' column as it will no longer be informative, and could be confusing. 


```python
N400_tw = (.400, .600)
```


```python
df_means = df_all.loc[(df_all['time'] > N400_tw[0]*1000) & (df_all['time'] < N400_tw[1]*1000)].groupby(['subject', 'background', 'sen_type']).mean('value')
df_means = df_means.drop(columns = 'time')
df_means
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
      <th>value</th>
    </tr>
    <tr>
      <th>subject</th>
      <th>background</th>
      <th>sen_type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">nagl002</th>
      <th rowspan="2" valign="top">Noise</th>
      <th>Control</th>
      <td>1.183809</td>
    </tr>
    <tr>
      <th>Violation</th>
      <td>-0.979825</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">Quiet</th>
      <th>Control</th>
      <td>1.187312</td>
    </tr>
    <tr>
      <th>Violation</th>
      <td>-1.691669</td>
    </tr>
    <tr>
      <th>nagl003</th>
      <th>Noise</th>
      <th>Control</th>
      <td>1.228835</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>nagl030</th>
      <th>Quiet</th>
      <th>Violation</th>
      <td>-1.108669</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">nagl031</th>
      <th rowspan="2" valign="top">Noise</th>
      <th>Control</th>
      <td>-3.292339</td>
    </tr>
    <tr>
      <th>Violation</th>
      <td>-3.030141</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">Quiet</th>
      <th>Control</th>
      <td>-2.466758</td>
    </tr>
    <tr>
      <th>Violation</th>
      <td>-5.946990</td>
    </tr>
  </tbody>
</table>
<p>80 rows × 1 columns</p>
</div>



Show a sample of 10 random rows from `df_means`:


```python
df_means.sample(n=10)
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
      <th>value</th>
    </tr>
    <tr>
      <th>subject</th>
      <th>background</th>
      <th>sen_type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>nagl016</th>
      <th>Quiet</th>
      <th>Control</th>
      <td>0.196358</td>
    </tr>
    <tr>
      <th>nagl008</th>
      <th>Noise</th>
      <th>Control</th>
      <td>-3.629049</td>
    </tr>
    <tr>
      <th>nagl002</th>
      <th>Noise</th>
      <th>Violation</th>
      <td>-0.979825</td>
    </tr>
    <tr>
      <th>nagl018</th>
      <th>Noise</th>
      <th>Violation</th>
      <td>-3.280324</td>
    </tr>
    <tr>
      <th>nagl026</th>
      <th>Noise</th>
      <th>Violation</th>
      <td>-3.145489</td>
    </tr>
    <tr>
      <th>nagl020</th>
      <th>Noise</th>
      <th>Violation</th>
      <td>-2.254707</td>
    </tr>
    <tr>
      <th>nagl031</th>
      <th>Quiet</th>
      <th>Violation</th>
      <td>-5.946990</td>
    </tr>
    <tr>
      <th>nagl006</th>
      <th>Quiet</th>
      <th>Control</th>
      <td>1.233813</td>
    </tr>
    <tr>
      <th>nagl030</th>
      <th>Quiet</th>
      <th>Control</th>
      <td>-3.805278</td>
    </tr>
    <tr>
      <th>nagl029</th>
      <th>Noise</th>
      <th>Violation</th>
      <td>-11.110585</td>
    </tr>
  </tbody>
</table>
</div>



If done correctly, you will see that the result is a DataFrame with subject, background, and sen_type as indexes, and value as the only data column. This `value` column is the mean ERP amplitude across all trials, for each subject, channel, and combination of conditions.

The `.groupby()` method cuased pandas to use the columns specified there as indexes. In some cases, this is desirable, but it's actually not what we want for the steps below; we just want these treated as normal columns. So, the line below will convert these columns back to "normal", and index the DataFrame numerically:


```python
df_means.reset_index(inplace=True)
```

Show a sample of 10 random rows from `df_means` to confirm the indexes were reset:


```python
df_means.sample(n=10)
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
      <th>subject</th>
      <th>background</th>
      <th>sen_type</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>69</th>
      <td>nagl029</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>-11.110585</td>
    </tr>
    <tr>
      <th>39</th>
      <td>nagl016</td>
      <td>Quiet</td>
      <td>Violation</td>
      <td>-4.163883</td>
    </tr>
    <tr>
      <th>51</th>
      <td>nagl019</td>
      <td>Quiet</td>
      <td>Violation</td>
      <td>0.062284</td>
    </tr>
    <tr>
      <th>17</th>
      <td>nagl006</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>-5.164039</td>
    </tr>
    <tr>
      <th>5</th>
      <td>nagl003</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>-4.113631</td>
    </tr>
    <tr>
      <th>29</th>
      <td>nagl009</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>-3.659548</td>
    </tr>
    <tr>
      <th>43</th>
      <td>nagl017</td>
      <td>Quiet</td>
      <td>Violation</td>
      <td>-0.376076</td>
    </tr>
    <tr>
      <th>30</th>
      <td>nagl009</td>
      <td>Quiet</td>
      <td>Control</td>
      <td>4.328665</td>
    </tr>
    <tr>
      <th>35</th>
      <td>nagl010</td>
      <td>Quiet</td>
      <td>Violation</td>
      <td>-4.700094</td>
    </tr>
    <tr>
      <th>25</th>
      <td>nagl008</td>
      <td>Noise</td>
      <td>Violation</td>
      <td>0.523079</td>
    </tr>
  </tbody>
</table>
</div>



## Seaborn plots

Generate a boxplot of values from `means`, with amplitude on the *y* axis and sentence type on the *x* axis. Use separate columns for the two background conditions.


```python
sns.boxplot(x=df_means['sen_type'], y=df_means['value'], data=df_means)
plt.ylabel('Amplitude')
plt.xlabel('Sentence Type')
plt.title('Difference in Mean N400 ERP Amplitude for Control vs Violation Sentence Conditions')
plt.show()
```




    
![png](Project_2_files/Project_2_72_0.png)
    



---
# Stats
## *t*-tests over N400 time window

Use `scipy.stats.ttest_rel` ([API here](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_rel.html)) and the data in `df_mean`, perform a pairwise *t*-test between Violation and Control, separately for each Background Condition. 

Print the *t* and *p* values for each contrast, with *t* rounded to two decimal places, and *p* rounded to 4 decimal places. Be sure each output line specifies which Background Condition the result is for.


```python
from scipy.stats import ttest_rel
```


```python
noise_cont = df_means[(df_means['background'] == 'Noise') & (df_means['sen_type'] == 'Control')]
noise_viol = df_means[(df_means['background'] == 'Noise') & (df_means['sen_type'] == 'Violation')]
quiet_cont = df_means[(df_means['background'] == 'Quiet') & (df_means['sen_type'] == 'Control')]
quiet_viol = df_means[(df_means['background'] == 'Quiet') & (df_means['sen_type'] == 'Violation')]

noise_tstat, noise_pval = ttest_rel(noise_cont['value'], noise_viol['value'])
quiet_tstat, quiet_pval = ttest_rel(quiet_cont['value'], quiet_viol['value'])

print('Control vs Violation in Noise background: t statistic = ' + str(round(noise_tstat, 2)) + ', p value = ' + str(round(noise_pval, 4)))
print('Control vs Violation in Quiet background: t statistic = ' + str("%.2f" % quiet_tstat) + ', p value = ' + str(round(quiet_pval, 4)))
```

    Control vs Violation in Noise background: t statistic = 3.62, p value = 0.0018
    Control vs Violation in Quiet background: t statistic = 3.60, p value = 0.0019


Now do a *t*-test comparing only the violation conditions, between quiet and noise backgrounds.


```python
viol_tstat, viol_pval = ttest_rel(noise_viol['value'], quiet_viol['value'])

print('Quiet vs Noise background in the Violation condition: t statistic = ' + str(round(viol_tstat, 2)) + ', p value = ' + str(round(viol_pval, 4)))
```

    Quiet vs Noise background in the Violation condition: t statistic = -0.14, p value = 0.8887


---

## Discussion

Based on the results, provide a conclusion for each of the three hypotheses. In your answers, be specific about what evidence you're using to draw your conclusions. That is, make reference to the definition of the N400 in the introduction, and refer to specific plots, and features of those plots. It is not enough simply to say "there is an N400, as shown in the waveform plot"; instead, you need to talk about the definition of the N400 (timing, where it's biggest on the scalp), and specific details in your various plots that support that. You should refer to details in all of your results (waveform plots, topo maps, box plots, and *t*-tests) in addressing each hypothesis.

Our three hypotheses were:
- "there will be an N400 in the quiet condition, with violation sentences eliciting more negative ERPs than control sentences between 400-600 ms"
- "there will also be an N400 in the noise condition, with violation sentences eliciting more negative ERPs than control sentences between 400-600 ms"
- "the N400 in response to violations will be significantly smaller in the noise than quiet conditions, because more cognitive effort will be directed to filtering out noise, leaving fewer resources to process the semantic anomaly"

In the ERP waveform plot (cell 35), there is a clear difference between the N400 amplitude for the control vs violation conditions in both the quiet and noisy background conditions. The violation sentence condition in both quiet and noise background conditions has a more negative N400 amplitude than the control sentence conditions. The topographic plots in cells 41 and 43 also support this result as they show there is a more negative N400 potential over the vertex of the head for quiet/noise violation conditions as compared to quiet/noise control. Finally, the box plot of mean values in cell 72 also shows a stronger negative N400 amplitude in the violation sentence condition compared to control. This data supports the first two hypotheses; the N400 amplitude was more negative (i.e. produced a larger negative effect) in the violation condition for both background conditions. 

In the waveform plot (cell 49), you can see the N400 amplitude over time, which shows that the noise condition had a larger negative amplitude for most of the time frame (except for a short blip around 0.500s). This result is also exemplified in the topographic maps (cell 53) at timestamp 0.400-0.600s. This indicates that this difference was exacerbated by the added distraction of background noise, which is contrary to what was expected.

The p values comparing control and violation sentence conditions was lower than the alpha value (0.05) for both background conditions, Noise (0.0018) and Quiet(0.0019). This means the difference between the N400 amplitudes was statistically significant, which supports the first two hypotheses.

The p value for the comparison of noise and quiet background conditions was greater than the alpha value (0.05), therefore the result that the N400 amplitude was more negative in the noise condition was not statistically significant.

In conclusion, our data supported our first two hypotheses, but not the third hypothesis. For the third hypothesis, there was no significant difference between the conditions.


```python

```

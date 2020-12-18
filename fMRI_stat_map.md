## Statistical Mapping of Experimental fMRI Data

I started by loading in a statistical Z map, the results of an analysis of a single fMRI scan from an experiment designed to activate area V5 of the visual cortex in response to an alternating pattern of moving and stationary visual stimuli.


```python
fmri_zstat = nib.load('V5_loc1_zstat1.nii.gz')
```

### Access and Visualize the Data

We can get a sense of the range of values in the data by finding the min and max values. The image contains positive *Z* scores, which represent stronger activation in the moving than stationary condition, as well as negative values, which represent stronger activation in the stationary condition. The latter are not of particular interest to us, but such "deactivations" commonly occur in just about every fMRI study.


```python
print('Max z value = ' + str(fmri_zstat_data.max().round(2)))
print('Min z value = ' + str(fmri_zstat_data.min().round(2)))
```

    Max z value = 8.05
    Min z value = -5.96


I plotted 9 slices through the image, and used the seismic colour map to make the positive and negative values more easily distinguishable from both each other and zero.

   
```python
fig = plt.figure(figsize=[8, 12])
subplot_counter = 1

for x in range(0, 27, 3):
    plt.subplot(3, 3, subplot_counter)
    im1 = fmri_zstat_data[:, :, x]
    plt.imshow(im1, cmap='seismic', vmin=-8, vmax=8)
    plt.axis('off')
    subplot_counter = subplot_counter + 1
plt.tight_layout()
plt.show()
```

![png](Assignment_5_files/Assignment_5_84_0.png)
    


### Thresholding the statistical image

Since our objective is to identify V5 and any other brain area that responds more strongly to moving than stationary stimuli, we will next *threshold* the image to only show the *Z* scores above our statistical threshold. 

Statistical thresholding in fMRI is actually a big topic, because it's more complex than in other areas of psychology or neuroscience that you may have encountered. Our fMRI image has a shape of 64 x 64 x 27 voxels, which is a total of 110 592, and we're performing a statistical test at *each* of those voxels. So a conventional *p* threshold of .05 — which means that 5% of "significant" findings would occur by chance — would result in over 5 000 false positives just by chance. In "real" fMRI work we use sophisticated routines to correct for multiple comparisons. Here, for simplicity we'll just apply a threshold of *p* < .0001. This is more conservative (fewer false positives) than *p* < .05, but not as fancy as what would be used in real fMRI research. 

`scipy.stats` provides a convenient function to determine the *Z* value corresponding to any *p* value. Note that we need to use the inverse of our desired *p* threshold (1 - *p*):


```python
z_thresh = scipy.stats.norm.ppf(1 - .0001)
print(z_thresh)
```

    3.719016485455709


Copy and paste your code from above that showed 9 slices through the *Z* map with the seismic colour map, and this time plot the image thresholded to show only voxels with values above `zthresh`. To do this, you will first need to make a mask using `np.where()`. Assign the mask to `thresh_zstat`. Voxels above threshold in `thresh_zstat` should take the values of `fmri_zstat_data`, and values below threshold should be set to `np.nan` 

<font color='red'>Here too. </font>


```python
blue += [1.]
```


```python
fig = plt.figure(figsize=[8, 12])
subplot_counter = 1
thresh_zstat = np.where(fmri_zstat_data > z_thresh, fmri_zstat_data, np.nan)
for x in range(0, 27, 3):
    plt.subplot(3, 3, subplot_counter)
    im1 = thresh_zstat[:, :, x]
    plt.imshow(im1, cmap='seismic', vmin=-8, vmax=8)
    plt.axis('off')
    subplot_counter = subplot_counter + 1
plt.tight_layout()
plt.show()
```
    
![png](Assignment_5_files/Assignment_5_90_0.png)

Of course, those "blobs" don't tell us much without being able to visualize them relative to the brain anatomy. To do this, we can overlay the thresholded *Z* map on a greyscale brain image. The easiest one to use is a "raw" fMRI image from the same scan, because this has the same resolution and orientation as our statistical map. There are ways to use a higher-quality anatomical scan, but we won't add that compelxity to our code here.

The cell below loads the image we'll use as the anatomical underlay:


```python
func = nib.load('V5_loc1_example_func.nii.gz')
underlay = func.get_fdata()
```

Now, copy and paste the code from above that plotted`thresh_zstat`. The only change you need to make is to add one line of code right *above* the `plt.imshow()` line that plots `thresh_zstat`, but the image it plots should be `underlay`, and the colourmap should be grey (don't add any other arguments). Be sure that you're using the same slicing code on the data in both `plt.imshow()` commands, to ensure you're plotting the same slice for the overlay and underlay.

<font color='red'>Your `im1` variable has three dimensions, but your `underlay1` variable only has two. Python can't overlay them, so it breaks. </font>


```python
blue += [0.]
```


```python
fig = plt.figure(figsize=[8, 12])
subplot_counter = 1
thresh_zstat = np.where(fmri_zstat_data > z_thresh, fmri_zstat_data, np.nan)
for x in range(0, 27, 3):
    plt.subplot(3, 3, subplot_counter)
    im1 = thresh_zstat[:, :, x]
    underlay1 = underlay[:, :, x]
    plt.imshow(underlay1, cmap='gray')
    plt.imshow(im1, cmap='seismic', vmin=-8, vmax=8)
    plt.axis('off')
    subplot_counter = subplot_counter + 1
plt.tight_layout()
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_96_0.png)
    



For reference, the 6th slice (2nd row, 3rd column) shows area V5 in each hemisphere. Due to the way the images are oriented, these are the blobs of red nearer to the back of the head (left side of image) in each hemisphere. There are other areas of significant activation as well; some of these may be real, since other brain areas might be engaged by this task (e.g., attention, or related to pressing different buttons in the two conditions). However, some of that is likely noise as well. In a typical experiment, we would do several fMRI scans to obtain a more robust estimate of activation in each individual, and we would also include many individuals in the study to determine the average response across people. 

# 

## Grade


```python
print(blue)
print(len(blue))
print(np.sum(blue))
```


```python
print("Grade = " + str(round(np.sum(blue)/len(blue) * 100, 2)) + "%")
print("XP = " + str(int((np.sum(blue)/len(blue)) * 1000)))
```

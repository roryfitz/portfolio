## Masking an Image

I started by reading in a 3D volume of 160 DICOM files


```python
vol = imageio.volread('5.T1_GRE_3D_AXIAL')
```

    Reading DICOM (examining files): 1/161 files (0.6%)...............161/161 files (100.0%)
      Found 1 correct series.
    Reading DICOM (loading data): 160/160  (100.0%)


### Visualize a slice

In this image, there are 160 slices. Visualize the middle slice, using `plt.imshow()`. To do this, you will need to slice the volume (using list slicing), specifying a single slice, and then all rows/columns within the slice.  

<font color='red'>This worked, but its important to be explicit with your code. Here, python is correctly interpreting which plane your `[80]` is meant to represent, but we'll see other situations below. </font>


```python
blue += [.75]
```


```python
plt.imshow(vol[80], cmap='gray')
plt.axis('off')
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_38_0.png)
    



### View multiple slices

Since one slice is not very representative of the brain, we can use subplots to visualize a number of slices through the brain volume. 
Since there are 160 slices, the math works well to plot every 10th slice, in a 4 x 4 array of subplots. 

Write a `for` loop to do this. A few hints:
- the `for` line should use the `range` function to go from 0 to the total number of slices (size of the slice dimension in the image), with the third argument to `range` (the step) used to specify every 10th slice.
- use the same commands as in previous plots above to show the image, and turn the axes off
- also include the line: `plt.tight_layout()` after you draw each slice in the loop. This makes less spacing between each subplot
- don't forget to increment `subplot_counter` each time through the loop

<font color='red'> Nice and explicit there. Good.</font>


```python
blue += [1.]
```


```python
fig = plt.figure(figsize=[8, 12])
subplot_counter = 1

for x in range(0, 160, 10):
    plt.subplot(4, 4, subplot_counter)
    im = vol[x, :, :]
    plt.imshow(im, cmap='gray')
    plt.axis('off')
    subplot_counter = subplot_counter + 1
plt.tight_layout()
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_42_0.png)
    



Another cool thing we can do with a 3D image is visualize a slice in some other dimension. That is, although the image was aquired as 160 slices in the axial plane, we can "reslice" the image to view the image from a different perspective. Below, show the middle slice of the image in the sagittal plane (specify a number in the third dimension, with the other two dimensions shown as `:`).


```python
plt.imshow(ndi.rotate(vol[:, :, 96], 180), cmap='gray')
plt.axis('off')
plt.show()
```

    
<img src="cell 51.png" width="640" />
    

I then generated a histogram to identify the intensity ranges of the various tissue types present in the image (e.g. CSF, grey matter, white matter)


```python
hist = ndi.histogram(brain_img, min=0, max=500, bins=256)
plt.plot(hist)
plt.show()
```
    
<img src="cell 55 - hisogram.png" width="640" />
    


Upon analysis of the histogram, I noted that the brain tissue (grey and white matter) intensity values lay in the 150-200 range, represented by the second peak. Within that peak, it was difficult to distinguish between the grey and white intensity values. I used this range of values to create a mask that would isolate the brain tissue in the image, so that further distinction between grey and white matter could be made.


```python
brain_mask = np.where(brain_img > 125, 1, 0)
plt.imshow(brain_mask, cmap='gray')
plt.axis('off')
plt.show()
```

<img src="cell 59 - mask.png" width="640" />
    


I then used this brain mask to exclude the data from voxels not part of the mask and create a histogram of just the intensities of voxels within the mask. This allowed for me to disstinguish between the intensity ranges of grey matter voxels and white matter voxels.

```python
brain_mask_1 = np.where(brain_mask == 1, brain_img, np.nan)
hist_brain = ndi.histogram(brain_mask_1, 0, 300, 300)
plt.plot(hist_brain)
plt.show()
```
    
<img src="cell 63 - mask hist.png" width="640" />
    

This histogram showed one peak just below 150 and another at around 180. The 150 peak corresponds to grey matter and the 180 to white matter.

Note: this code was written in python

## Masking an Image

I started by loading the DICOM image using imageio.imread


```python
brain_img = imageio.imread('Anat001.20040930.142737.2.MPRAGE_T1_SAGITTAL.0080.dcm')
```


```python
plt.imshow(brain_img, cmap='gray')
plt.axis('off')
plt.show()
```

    
<img src="cell 23.png" width="640" />
    

I then generated a histogram to identify the intensity ranges of the various tissue types present in the image (e.g. CSF, grey matter, white matter)


```python
hist = ndi.histogram(brain_img, min=0, max=500, bins=256)
plt.plot(hist)
plt.show()
```
    
<img src="cell 55 - histogram.png" width="640" />
    


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

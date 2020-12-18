# Assignment 5
## NSEC 3505
## Working with MRI Data

This assignment is based on the DataCamp lesson, [Biomedical Image Analysis in Python](https://learn.datacamp.com/courses/biomedical-image-analysis-in-python). To complete this assignment, you can rely on chapters 1 and 2 from that lesson. I suggest that you download the slides from those chapters and refer to them as you work through this lesson.


First we'll load all the packages we'll need for this assignment:


```python
blue = []
```


```python
import imageio
import scipy.ndimage as ndi
import scipy.stats
import nibabel as nib
import numpy as np
import matplotlib.pyplot as plt
```

Use the `imageio.imread` function to load in the DICOM file named `Anat001.20040930.142737.2.MPRAGE_T1_SAGITTAL.0080.dcm`. Assign the object to `brain_img` (i.e., use `brain_img = ` followed by the load command)

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_img = imageio.imread('Anat001.20040930.142737.2.MPRAGE_T1_SAGITTAL.0080.dcm')
```

As you learned in the lesson, you can view the metadata for a DICOM file, which is stored in the `.meta` property of the object. Show all the metadata

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_img.meta
```




    Dict([('TransferSyntaxUID', '1.2.840.10008.1.2'),
          ('SOPClassUID', '1.2.840.10008.5.1.4.1.1.4'),
          ('SOPInstanceUID', '1.3.12.2.1107.5.2.13.20590.4.0.3354588020772092'),
          ('StudyDate', '20040930'),
          ('SeriesDate', '20040930'),
          ('AcquisitionDate', '20040930'),
          ('ContentDate', '20040930'),
          ('StudyTime', '142047.390000 '),
          ('SeriesTime', '142737.546000 '),
          ('AcquisitionTime', '142142.387518 '),
          ('ContentTime', '142805.234000 '),
          ('Modality', 'MR'),
          ('Manufacturer', 'SIEMENS'),
          ('InstitutionName', 'UNIVERSITY OF ROCHESTER'),
          ('StudyDescription', 'ACHTMAN^FMRI'),
          ('SeriesDescription', 'MPRAGE T1 SAGITTAL'),
          ('PatientName', 'Anat001'),
          ('PatientID', '04.09.30-14:20:32-DST-1.3.12.2.1107.5.2.13.20590'),
          ('PatientBirthDate', '19730905'),
          ('PatientSex', 'M '),
          ('PatientAge', '031Y'),
          ('PatientWeight', 77.18),
          ('StudyInstanceUID', '1.3.12.2.1107.5.2.13.20590.4.0.3352789032273021'),
          ('SeriesInstanceUID', '1.3.12.2.1107.5.2.13.20590.4.0.3354137163342650'),
          ('SeriesNumber', 2),
          ('AcquisitionNumber', 1),
          ('InstanceNumber', 80),
          ('ImagePositionPatient', (-3.2118645, -186.98305, 100.20339)),
          ('ImageOrientationPatient', (0.0, 1.0, 0.0, 0.0, 0.0, -1.0)),
          ('SamplesPerPixel', 1),
          ('Rows', 256),
          ('Columns', 256),
          ('PixelSpacing', (1.0, 1.0)),
          ('BitsAllocated', 16),
          ('BitsStored', 12),
          ('HighBit', 11),
          ('PixelRepresentation', 0),
          ('PixelData',
           b'Data converted to numpy array, raw data removed to preserve memory'),
          ('shape', (256, 256)),
          ('sampling', (1.0, 1.0))])



You can also view a specific field/property of the metadata, by specifying it in square brackets. Show the *modality* of `brain_img`. (In imaging, *modality* refers to the type of imaging, such as CT, PET, MRI, EEG, MEG, etc.)

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_img.meta['Modality']
```




    'MR'



The image you've loaded is a single 2D "slice" through the brain. What is the shape of the image (size along each dimension)?

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_img.shape
```




    (256, 256)



Display the image. As in the lesson, use the `gray` colour map (American spelling :) and turn off the axes.

<font color='red'> </font>


```python
blue += [1.]
```


```python
plt.imshow(brain_img, cmap='gray')
plt.axis('off')
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_22_0.png)
    



## A 3D Volume

Now we're going to load in a 3D volume, i.e., a whole series of slices that cover the entire brain. This is stored in a set of 160 DICOM files, each a slice through the brain. They are stored inside a folder called `5.T1_GRE_3D_AXIAL`. Use `imageio.volread()` to load in this data set and assign it to `vol`.

<font color='red'> </font>


```python
blue += [1.]
```


```python
vol = imageio.volread('5.T1_GRE_3D_AXIAL')
```

    Reading DICOM (examining files): 1/161 files (0.6%)161/161 files (100.0%)
      Found 1 correct series.
    Reading DICOM (loading data): 160/160  (100.0%)


What is the shape of this image?

<font color='red'> </font>


```python
blue += [1.]
```


```python
vol.shape
```




    (160, 256, 192)



Another thing that's often nice to know about the image is its spatial resolution. That is, how big are the *voxels* in each dimension? **Voxels** are what we call pixels when they are 3D — i.e., volume pixels. This is stored in the metadata property `sampling`. Show the voxel dimensions of this image (note, they are in mm).

<font color='red'> </font>


```python
blue += [1.]
```


```python
vol.meta['sampling']
```




    (1.0, 1.0, 1.0)



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
    



Another cool thing we can do with a 3D image is visualize a slice in some other dimension. That is, although the image was aquired as 160 slices in the axial plane, we can "reslice" the image to view the image from a different perspective. Below, show the middle slice of the image in the sagittal plane (specify a number in the third dimension, with the other two dimensions shown as `:`). The orientation of the image will be upside down.

<font color='red'> </font>


```python
blue += [1.]
```


```python
plt.imshow(vol[:, :, 96], cmap='gray')
plt.axis('off')
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_46_0.png)
    



We can turn the image right side up for the plot, using the `ndi.rotate()` function. This is described in chapter 4 of the DataCamp lesson, slide 8. Copy and paste your code from the previous cell below, and embed the data specification (your slice of `vol`) inside the `ndi.rotate()` command.

<font color='red'> </font>


```python
blue += [1.]
```


```python
plt.imshow(ndi.rotate(vol[:, :, 96], 180), cmap='gray')
plt.axis('off')
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_50_0.png)
    



### Histograms and tissue types

In structural/anatomical MRI brain images, the **contrast** is difference in intensities, that allows you to distinguish white matter from grey matter from the cerebralspinal fluid (CSF) around the brain and in the ventricles. The primary source of contrast in MRI is between fat and water. The MRI image you are working with has what is called **T1 weighting**, in which water appears dark and fat appears bright. Since CSF is mostly water, it appears black, and white matter is brightest, because it is primarily composed of axons connecting different brain areas. The myelin sheath around the axons is primarily fat. Grey matter (the outer surface of the cortex, and some inner areas like the basal ganglia) has intensity values between CSF and white matter, because the tissue is more dense than CSF, but less fatty than white matter.

Because different tissues have different intensities, a histogram of the image can help us identify the intensity ranges of different tissue types. 

Switching back to the first image we loaded, `brain_img`, use the `ndi` package to plot a histogram of `brain_img`. Use a minimum of 0 and maximum of 500, with 500 bins.

<font color='red'> </font>


```python
blue += [1.]
```


```python
hist = ndi.histogram(brain_img, min=0, max=500, bins=256)
plt.plot(hist)
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_54_0.png)
    



### Mask the image

The histogram above shows a large peak close to zero. This means there are a large number of voxels in teh image with values close to zero. This is because much of the 3D image volume is outside of the brain (the black background area), and as well there are dark areas inside the brain, where CSF is. There is a second peak between intensity values of approximately 150-200. This peak represents the "brain" voxels. Grey and white matter have pretty similar intensities in this image, so we don't see distinct peaks in the histogram for each of these, but the histogram tells us that "brain" voxels generally have intensities between 150-200. 

As in the lesson, use `np.where()` to create a mask to isolate the brain in `brain_img`, using a threshold of 125. Make values above 125 equal to 1, and values below the threshold equal to zero. Assign this to `brain_mask`. Then plot `brain_mask` using similar commands that you used above for `brain_img`. 

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_mask = np.where(brain_img > 125, 1, 0)
plt.imshow(brain_mask, cmap='gray')
plt.axis('off')
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_58_0.png)
    



### Masked Histogram

One use of creating a brain mask like this is that we can use it to exclude data from voxels outside of the mask. In the histogram above, as we noted there are only two obvious peaks: one corresponding to non-brain areas of the image, and one for brain tissue. However, since grey and white matter have different intensities, logically they *should* have different peaks in the histogram. The issue above is that they are too "squished together" to distinguish. 

So we can use the brain mask to compute and plot a histogram *only* of voxels inside the brain mask. Do this below. To mask `brain_img`, you'll first need to use `np.where()` to select voxels in `brain_mask` with a value of 1 (the "brain" voxels). In the output image, voxels with a value of 1 in the mask should take the values of `brain_img`, while voxels not in the brain mask should take values of `np.nan`. It's important to use `np.nan` and not 0, because NaN values are ignored when making the histogram, which is what we want to do with non-brain voxels.

Once you've assigned the masked brain image to a new variable, compute and plot a histogram of the result. Use the same command and parameters as your previous histogram, just with a different (masked) input image. For the histogram, use a minimum of 0 (since there are no voxels below that value), a maximum of 300, and 300 bins. This will help "zoom in" on the range of values we're interested in.

<font color='red'> </font>


```python
blue += [1.]
```


```python
brain_mask_1 = np.where(brain_mask == 1, brain_img, np.nan)
hist_brain = ndi.histogram(brain_mask_1, 0, 300, 300)
plt.plot(hist_brain)
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_62_0.png)
    



This does allow us to see more fine-grained structure in the histogram, including one peak just below 150 and another at around 180. It turns out that for this particular brain image, the contrast between grey and white matter is not that strong, so the peaks in the histogram are not that distinct; but hopefully this conveys a sense of how and why you would use masking. 

## Working with fMRI Statistical Maps

Now we'll see how we can apply these approaches to fMRI data. We're going to load in a statistical map, the results of an analysis of a single fMRI scan from an experiment. This experiment was designed to activate area V5 of the visual cortex, which is particularly sensitive to visual motion. During the scan, the stimuli alternated between moving and stationary dots every 15.4 s. During moving blocks, the participant had to press a button whenever the dot speed briefly increased; during the stationary blocks, the dots occasionally dimmed in brightness, and the participant had to press the button when they detected dimming. 

To analyze the data, we correlated the time series at each voxel with a time series that represents the experimental design. The experimental design could be drawn as a *square wave*, like this example where "on" would be moving stimuli and "off" would be stationary stimuli:

<img src='fmri_block_design.jpg' width=450px>

 
The brain doesn't actually respond in an instant on-off fashion, but rather gradually ramping up and down again. Formally this is known as the **hemodynamic response function**, and we correlated our fMRI data with a variant of the square wave above that is more rounded. The details aren't important for now, however. What we are going to work with now is the *output* of such a correlation (also called **regression**) analysis. 

The image that we load below is called a **statistical parametric map**. It's a 3D volume, like other MRI images we've worked with, but here the intensity value of each voxel is a statistical *Z* score representing the strength of that correlation (which you can also think of as the difference in fMRI signal between moving and stationary blocks), divided by its variance. *Z* scores are *standardized*, meaning that a particular *Z* value is always associated with a particular *p* value. For example, *Z* = 2.33 equates to a *p* = 0.01.

As a side note, these are in a different file format. The above images were in DICOM, but these are in [NIfTI](https://nifti.nimh.nih.gov) format. *NIfTI* stands for "neuroimaging informatics technology initiative", and was sponsored by the the National Institute of Mental Health and the National Institute of Neurological Disorders and Stroke with the aim of providing a standard file format that could be used across different software packages and labs working in neuroimaging research.

SciPy's `imageio` package does not read NIfTI files, so we need to import and use a different package, [NiBabel](https://nipy.org/nibabel/), which was written specifically for reading a variety of neuroimaging data formats (in spite of NIfTI's being an effort to standardize file formats, there are still a number of other formats in fairly widespread use). We imported NiBabel at the top of this file as `nib`.


```python
fmri_zstat = nib.load('V5_loc1_zstat1.nii.gz')
```

`fmri_zstat` is stored as a Python *object*, of the class `nibabel.nifti1.Nifti1Image`. So it's a complex data structure, not just the data comprising a brain image:


```python
type(fmri_zstat)
```




    nibabel.nifti1.Nifti1Image



One important component of the NIfTI image is its header, which contains metadata for the image. We can see the header by printing it:


```python
print(fmri_zstat.header)
```

    <class 'nibabel.nifti1.Nifti1Header'> object, endian='<'
    sizeof_hdr      : 348
    data_type       : b''
    db_name         : b''
    extents         : 0
    session_error   : 0
    regular         : b'r'
    dim_info        : 0
    dim             : [ 3 64 64 27  1  1  1  1]
    intent_p1       : 0.0
    intent_p2       : 0.0
    intent_p3       : 0.0
    intent_code     : z score
    datatype        : float32
    bitpix          : 32
    slice_start     : 0
    pixdim          : [1.   3.75 3.75 4.   1.   0.   0.   0.  ]
    vox_offset      : 0.0
    scl_slope       : nan
    scl_inter       : nan
    slice_end       : 0
    slice_code      : unknown
    xyzt_units      : 10
    cal_max         : 8.046828
    cal_min         : -5.9577875
    slice_duration  : 0.0
    toffset         : 0.0
    glmax           : 0
    glmin           : 0
    descrip         : b'FSL5.0'
    aux_file        : b''
    qform_code      : scanner
    sform_code      : scanner
    quatern_b       : 0.0
    quatern_c       : 0.0
    quatern_d       : 0.0
    qoffset_x       : -121.725
    qoffset_y       : -87.425
    qoffset_z       : -52.0
    srow_x          : [   3.75     0.       0.    -121.725]
    srow_y          : [  0.      3.75    0.    -87.425]
    srow_z          : [  0.   0.   4. -52.]
    intent_name     : b''
    magic           : b'n+1'


The header is quite complicated! But some of the most important information is relatively easy to see above. For example, the `dim` field tells us the dimensions of the MRI image, which contains 27 slices with each slice a 64 x 64 square matrix. Likewise, the `pixdim` field shows the dimensions of the image in millimetres: the slides are 4.0 mm thick, and 3.75 x 3.75 mm in-plane (i.e., within each slice). 

Since the image is a "map" of the *Z* scores, it's commonly called a *Z* map. Importantly, because the *Z* scores represent the correlations over the whole time series, we no longer have a 4D image, but just a 3D one:


```python
fmri_zstat.shape
```




    (64, 64, 27)



### Access and Visualize the Data

We can use the `.get_fdata()` method to access just the data stored in the object. This is a NumPy array:


```python
fmri_zstat_data = fmri_zstat.get_fdata()
type(fmri_zstat_data)
```




    numpy.ndarray



We can get a sense of the range of values in the data by finding the min and max values. The image contains positive *Z* scores, which represent stronger activation in the moving than stationary condition, as well as negative values, which represent stronger activation in the stationary condition. The latter are not of particular interest to us, but such "deactivations" commonly occur in just about every fMRI study.


```python
print('Max z value = ' + str(fmri_zstat_data.max().round(2)))
print('Min z value = ' + str(fmri_zstat_data.min().round(2)))
```

    Max z value = 8.05
    Min z value = -5.96


Plot a series of slices through this image. Copy your code from above that generated a 4 x 4 set of subplots for the anatomical volume, and adapt it to plot a series of slices through the fMRI image. Because the fMRI data is lower resolution, we have only 27 slices, so plot every *third* slice rather than every tenth, in a 3 x 3 grid of subplots.

You will see that although the shape of the image still looks brain-like, the intensity values make it look more "mottled" (blobs of black and white and grey). This reflects the fact that the brain is organized into distinct functional regions, that responded differently to the experimental manipulation.

<font color='red'>Interestingly, your slice selection is explicit, however your variable is in the wrong plane. Put it in the third plane (`fmri_zstat_data[:, :, x]`) and your output would be perfect. Small error, big effect.</font>


```python
blue += [.5]
```


```python
fig = plt.figure(figsize=[8, 12])
subplot_counter = 1

for x in range(0, 27, 3):
    plt.subplot(3, 3, subplot_counter)
    im1 = fmri_zstat_data[:, :, x]
    plt.imshow(im1, cmap='gray')
    plt.axis('off')
    subplot_counter = subplot_counter + 1
plt.tight_layout()
plt.show()
```




    
![png](Assignment_5_files/Assignment_5_80_0.png)
    



This is a case where a different colour map can help with visualization. Since the data have positive and negative values, with 0 meaning "no activation difference between conditions", it makes sense to use a colour map that has different colours for positive and negative, and no colour close to zero. The `seismic` colour map in Matplotlib is designed for exactly this purpose.

We also provide the arguments `vmin` and `vmax` to the plot command, to set the upper and lower ranges that the colours map to. This is important to ensure that all of the panels are scaled the same; you'll notice above that when we don't do this, the background (non-brain areas) intensity varies in each plot, suggesting that something is not right about how we're plotting the data. 

In the cell below, copy and paste your code from the previous cell to again plot 9 slices through the image. Make the following changes to the code:
- use the `seismic` colour map in `plt.imshow()`
- also as arguments to `plt.imshow()`, use `vmin=-8` and `vmax=8`

<font color='red'> Here too, and below, but these are now carry-down errors. Everything else looks great.</font>


```python
blue += [1.]
```


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

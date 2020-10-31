```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
```


```python
# read in image and convert to greyscale

orig = cv2.imread('CT_scan.jpg')
orig_grey = cv2.cvtColor(orig, cv2.COLOR_BGR2GRAY)
```


```python
# convert image to CLAHE object

clahe = cv2.createCLAHE(tileGridSize=(2,2))
orig_cl = clahe.apply(orig_grey)
```


```python
# create new equalized image from the clahe original

new = cv2.equalizeHist(orig_cl)
```


```python
# Stack the original image and new image side-by-side

res = np.hstack((orig_grey,new))
cv2.imwrite('res.png', res)
plt.imshow(res)
plt.axis('off')
```




    (-0.5, 1543.5, 465.5, -0.5)






    
![png](CT_equalizing_files/CT_equalizing_4_1.png)
    



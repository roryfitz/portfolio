# Alzheimer's MRI Study

```python
# read in the data
long = pd.read_csv('oasis_longitudinal.csv')
cross = pd.read_csv('oasis_cross-sectional.csv')
```


```python
#Convert M/F to Male,Female
dict = {'F':'Female', 'M':'Male'}
long['M/F'] = long['M/F'].replace(dict)
```

### Comparison of Age to Dementia Status and Brain Volume


```python
fig1 = plt.subplot(2,1,1)
sns.swarmplot(x='Group', y='Age', data=long, hue='M/F')
plt.title('Dementia Status vs Age', color='navy', size=14)
plt.xlabel('Group', color='navy', size=12)
plt.ylabel('Age', color='navy', size=12)


fig2 = plt.subplot(2,1,2)
sns.swarmplot(x='Group', y='nWBV', data=long, hue='M/F')
plt.title('Dementia Status vs Normalized Whole Brain Volume', color='navy', size=14)
plt.xlabel('Group', color='navy', size=12)
plt.ylabel('nWBV', color='navy', size=12)

plt.tight_layout()
plt.show()
```




    
![png](Alzheimer%27s%20MRI_files/Alzheimer%27s%20MRI_4_0.png)
    

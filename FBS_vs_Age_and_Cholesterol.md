# Heart Disease
Here I produced 2 plots, one comparing fasting blood sugar to age, and the other comparing fasting blood sugar to serum cholesterol.

Both plots also indicate sex of the participants.


```python
# read in data
heart = pd.read_csv('heart.csv')
```

In order to have the legends read "Male" and "Female" rather than "0" and "1", I decided to convert the values within the dataset rather than edit the legends themselves. This woould ensure that upon further data exploration/visualization I would not run into the same issue.


```python
# Convert sex from 0,1 values to Female,Male values
dict = {0:'Female', 1:'Male'}
heart['sex'] = heart['sex'].replace(dict)
```

Next, and finally, I created the plots!


```python
# Create first subplot, comparing fasting blood sugar to age
fig1 = plt.subplot(1,2,1)
sns.violinplot(x='fbs', y='age', data=heart, inner=None, color='lightgray')
sns.stripplot(x='fbs', y='age', data=heart, size=4, jitter=True, hue='sex')
plt.xlabel('Fasting Blood Sugar')
plt.ylabel('Age')
plt.title('Fasting Blood Sugar vs Age')

# Create second subplot, comparing fasting blood sugar to cholesterol
fig2 = plt.subplot(1,2,2)
sns.violinplot(x='fbs', y='chol', data=heart, inner=None, color='lightgray')
sns.stripplot(x='fbs', y='chol', data=heart, size=4, jitter=True, hue='sex')
plt.xlabel('Fasting Blood Sugar')
plt.ylabel('Serum Cholesterol (mg/dl)')
plt.title('Fasting Blood Sugar vs Serum Cholesterol')

# Edit x tick values
labels1 = [item.get_text() for item in fig1.get_xticklabels()]
labels2 = [item.get_text() for item in fig2.get_xticklabels()]
labels1 = labels2 = ['< 120mg/dl', '> 120mg/dl']

fig1.set_xticklabels(labels1)
fig2.set_xticklabels(labels2)

# Show plot
plt.tight_layout()
plt.show()
```




    
![png](FBS%20vs%20age%20and%20cholesterol_files/FBS%20vs%20age%20and%20cholesterol_6_0.png)
    



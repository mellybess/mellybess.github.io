---
layout: post
title:  "Getting started"
date:   2017-12-06 13:05:46 -0800
#Scategories: jekyll update
---
For my first project, I wanted to learn some basics: cleaning and combining some data and making some simple graphs. So I found a small data set about gun ownership and murder in the United States and I downloaded another small dataset with state related data to use with it.

To be clear: the dataset I was using is questionable. I think I found it on Kaggle and there was very little information on it. It has categories like `GunOwnership` with values between 0 and 1, but there wasn't any information on what that means. I decided to use it since it was pretty small and therefore I could focus more on learning the tools as opposed to fretting over the data.

### Goals

For this exercise I wanted to:
- Get comfortable with adding values and columns
- Make some different graphs with Matplotlib

nothin' fancy


### What I did

First I imported the libraries I needed, added some columns from the `states.csv` dataset to the main dataset, and added in a few values that were missing from Washington DC.

There were columns `Murder`, `Gun Murders`, and `GunOwnership` but I wasn't exactly sure how those compared to each other. I also wanted to see the number of murders and gun murders relative to each state's population. So I created columns for Murders and Gun Murders per 100k people and, assuming `GunOwnership` was the rate of gun ownership in the state, I multiplied that by 100k as well to get Gun ownership per 100k.


```python
df['Murders per 100k']= (df['Murders']/df['Population'])* 100000
df['Gun Murders per 100k']= (df['GunMurders']/df['Population'])* 100000
df['Gun ownership per 100k'] = (df['GunOwnership']*100000)
```

My assumption was that the rate of murders or at least the rate of gun murders would be in some way linked to the rate of Gun ownership.

I looked through a few examples and tutorials for Matplotlib. I first made a couple graphs comparing Murders and Gun Murders to GunOwnership using the matplotlib.pyplot module.

```python
%matplotlib inline
plt.subplot(1,2,1)
plt.plot(df['Murders per 100k'], df['Gun ownership per 100k'], 'b.')
plt.xlabel('Murders')
plt.ylabel('Gun Ownership')
plt.title('Murders v Gun Ownership')

plt.subplot(1,2,2)
plt.plot(df['Gun Murders per 100k'], df['Gun ownership per 100k'], 'r.')
plt.xlabel('Gun Murders')
plt.ylabel('Gun Ownership')
plt.title('Gun Murders v Gun Ownership')

plt.tight_layout()
```


![png](/assets/jekyll_test_5_0.png)

Hmmm. It seemed like my initial assumption wasn't quite accurate. There were all kinds of different murder and gun murder rates in states with similar gun ownership rates. And in fact, DC had a massive murder and gun murder rate, but a very low gun ownership rate. It looked like murders and gun murders were pretty correlated though.

From here, I wanted to try to recommended approach to Matplotlib, using their Object Oriented API.  My first figure compared Population Density and Gun ownershp. Unsurprisingly, there are a lot more guns per capita in less dense states. This makes a lot of sense, since it seems like a person would be more likely to own guns if they had land to shoot them on, and felt they needed a safety precaution if police or sheriffs are not likely to be close by.

```python
fig= plt.figure()
axes= fig.add_axes([0.1,0.1,0.8,0.8])

axes.plot(df['PopulationDensity'],df['GunOwnership'],'b.')
axes.set_xlabel('Population Density')
axes.set_ylabel("Gun Ownership")
axes.set_title('Density vs Gun Ownership rate');
```
![png](/assets/jekyll_test_6_0.png)

Then I wanted to try out having multiple graphs on a figure. So I compared a bunch of stuff relatively randomly.

```python
fig,axes= plt.subplots(nrows=2,ncols=2, figsize=(15,5))

axes[1,0].plot(df['Murders'], df['GunMurders'], 'b.')
#axes[1,0].text('Murders','Gun Murders')
axes[1,0].set_title('Murders v Gun Murders')
axes[0,1].plot(df['Murders per 100k'], df['Gun Murders per 100k'], 'b.')
axes[0,1].set_title('Murders v Gun Murders per 100k')
axes[0,0].plot(df['Population'], df['Murders per 100k'], 'r.')
axes[0,0].set_title('Population v Murders per 100k')
axes[1,1].plot(df['Population'], df['Gun ownership per 100k'], 'r.')
axes[1,1].set_title('Population v Gun Ownership per 100k')

plt.tight_layout()
```


![png](/assets/jekyll_test_7_0.png)

Finally I made the figure that I had been thinking about when I first looked at the data. I compared Gun ownership across all the states as a line, and Gun murders per 100k per state as a bar graph. I was stuck for a while since the two y-axes have very different scales, but the `twinx()` function allows you to plot on the x-axis regardless of the scale of your different y-axes.

```python
from __future__ import division
from matplotlib import colors as mcolors


colors = list(mcolors.CSS4_COLORS.keys())[20:29]
color_labels = dict(zip(color_labels, colors))

df1=df

df1["Colors"]=df1["Region"].apply(lambda x: color_labels[x])

x = df1['Abbreviation']
z = df1["Region"]

fig, ax1 = plt.subplots(figsize=(15,5))
ax1.plot(df1['Gun ownership per 100k'], color="purple", marker="o")
ax1.set_ylabel('Gun ownership per 100k', fontsize=15, color="purple")
ax1.set_xticklabels(x, rotation='vertical')
for label in ax1.get_yticklabels():
    label.set_color("purple")

ax2 = ax1.twinx()
ax2.bar(x,df1['Gun Murders per 100k'], color=df1["Colors"], alpha=0.5, label=df1["Region"])
ax2.set_ylabel("Gun murders per 100k", fontsize=15)
ax1.set_title("Gun Ownership and Murders by State", fontsize=20);


```


![png](/assets/jekyll_test_8_0.png)


I also wanted to group the states by region. For whatever reason I could not re order them on the x-axis to show up in groups...but regardless I think it was a fine afternoon of learning.

### Final thoughts

It's interesting that Gun ownership seemed to be strongly correlated to gun murders. There are many reasons why this could be:

- I was only comparing 2-3  variables at a time, maybe if I had combined them more cleverly a pattern would have appeared
- The data could be flawed. I know little about it, and maybe I made the wrong assumption on what Gun Ownership meant. For example, if gun ownership only took into account registered weapons, it wouldn't accurately represent the number of guns present in an area.
- This is looking at whole states. Some states have multiple major cities with lots of people, and a proportionally high amount of murders. Some states have few to no major cities. But this data only looked at the state level.

Also as I was reading, most sources indicated that Seaborn has easier and nicer ways to visualize data. So I'll learn that next and see if I can make some nicer visualizations of this data. Then I'll move on to another larger dataset.

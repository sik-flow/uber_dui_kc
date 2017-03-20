
### Have Drunk Driving Incidents Reduced since Uber has Arrived in Kansas City?

Lets see if Uber has reduced the number of DUIs in Kansas City. 

#### Load Libraries and Data


```python
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
import seaborn as sns
import matplotlib
import datetime
import urllib
from bokeh.plotting import *
from bokeh.models import HoverTool
from collections import OrderedDict
% matplotlib inline
plt.style.use('fivethirtyeight')
import warnings
warnings.filterwarnings('ignore')
```


```python
# 2016 Data
query = ("https://data.kcmo.org/resource/c6e8-258d.json?$limit=500000")
df_2016 = pd.read_json(query)
```


```python
# 2015 Data
query = ("https://data.kcmo.org/resource/geta-wrqs.json?$limit=500000")
df_2015 = pd.read_json(query)
```


```python
# 2014 Data
query = ("https://data.kcmo.org/resource/nsn9-g8a4.json?$limit=500000")
df_2014 = pd.read_json(query)
```


```python
# 2013 Data
query = ("https://data.kcmo.org/resource/ff6a-bhbu.json?$limit=500000")
df_2013 = pd.read_json(query)
```


```python
# 2012 Data
query = ("https://data.kcmo.org/resource/xwdv-8y2g.json?$limit=500000")
df_2012 = pd.read_json(query)
```


```python
# 2011 Data
query = ("https://data.kcmo.org/resource/5u8g-kq4k.json?$limit=500000")
df_2011 = pd.read_json(query)
```


```python
# 2010 Data
query = ("https://data.kcmo.org/resource/c3qq-bxi5.json?$limit=500000")
df_2010 = pd.read_json(query)
```


```python
# 2009 Data
query = ("https://data.kcmo.org/resource/3u3f-44ew.json?$limit=500000")
df_2009 = pd.read_json(query)
```

I downloaded data from 2009 to 2016.  I started at 2009 because that was as far back as I could find on the [Kansas City Open Data Platform.](https://data.kcmo.org/) Next, I need to sort through this data to only include the drunk driving incidents.  First I'm going to take a look at my data.


```python
df_2016.head()
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/df_2016.png'/>

It looks like I can do a search of the description column to find the drunk driving incidents; which is what I wil do next and concatenate all my dataframes together.


```python
duis = df_2016[df_2016['description'].str.contains('Driving')]
duis.description.unique()
```




    array([u'Driving Under Influe'], dtype=object)




```python
dui_2016 = df_2016[df_2016['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2015 = df_2015[df_2015['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2014 = df_2014[df_2014['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2013 = df_2013[df_2013['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2012 = df_2012[df_2012['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2011 = df_2011[df_2011['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2010 = df_2010[df_2010['description'] == 'Driving Under Influe'][['description', 'reported_date']]
dui_2009 = df_2009[df_2009['description'] == 'Driving Under Influe'][['description', 'reported_date']]
```


```python
dui = pd.concat([dui_2016, dui_2015, dui_2014, dui_2013, dui_2012, dui_2011, dui_2010, dui_2009])
```


```python
dui.shape
```




    (7883, 2)



So, we have 7,883 drunk driving incidents in Kansas City from 2009 to 2016!  Kinda scary when you think about it! I only including 2 columns in my main dataframe (dui) because I only really needed the date to use the resample command in pandas.  To use the resample command, I need to make sure that my 'reported_date' column is in a form that pandas can handle (i.e. date/time).  So let's check it out.


```python
dui.dtypes
```




    description      object
    reported_date    object
    dtype: object




```python
dui.head()
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/dui_head_1.png'/>

Naturally, the date is showing up as an object (aka string) and not a date. To convert 'reported_date' to a date/time, I first did a split at the 'T' charachter in  the column and then converted this to the datetime index.


```python
dui.reset_index(inplace = True, drop = True)
phantom_df = pd.DataFrame(dui['reported_date'].str.split('T').tolist(), columns = ['date', 'time'])
dui['date'] = phantom_df['date']
dui['date'] = pd.to_datetime(dui['date'], format = '%Y-%m-%d')
dui.drop('reported_date', axis =1, inplace =True)
```


```python
dui.head()
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/dui_head_2.png'/>

Everything is looking good so far, lets set the index to the 'date' column, since I am going  to be using the resample command and start making some plots!


```python
dui.set_index('date', inplace = True)
```

#### DUIs by Year

I first want to look at the total number of DUIs per year.


```python
drouin = dui.resample('A')['description'].count().plot(kind = 'bar', figsize = [10,6], color = 'navy', fontsize = 14)
drouin.set_xticklabels([2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016])
drouin.set_xlabel('')
drouin.set_ylabel('Number of DUIs')
drouin.set_title('DUIs by Year')

# Make labels
rects = drouin.patches
labels = dui.resample('A')['description'].count()
for rect, label in zip(rects, labels):
    height = rect.get_height()
    drouin.text(rect.get_x() + rect.get_width()/2, height + 10, label, ha='center', va='bottom')
    
# Make Mean Line
mean_duis = drouin.axhline(y=dui.resample('A')['description'].count().mean(), label = 'Mean: 985', c='y',
                           linewidth = 2)
drouin.legend(handles = [mean_duis], loc = 'upper left', fontsize = 14)
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/DUIs_by_Year.png'/>

This is not good!  Expected some slight variation but nothing what I am seeing here.  2013 to 2016 are consistent, but before 2013 is it not consistent at all.  

#### Deciding on Which Years to Include

I am not sure why 2009 and 2010 have such low numbers of DUIs.  I did a search in the description column for 2009 and 2010 to see if DUIs were labeled something differently.  In 2009 I found another description called 'driving under the in', however there was only 2 other incidents.  In 2010 I did not find any other options for DUIs in the description column.  Due to both of these years having such different results, I am going to eliminate them from my further analysis.  I also was curious about the big spike in the number of DUIs in 2011 and 2012.  I did some research and found that the city of Kansas City purchased a $395,000 [Mobile DUI Enforcment Center](http://kcmo.gov/wp-content/uploads/sites/2/2013/10/2annual2011final1.pdf) in 2011 and believe this explains some of the raised number of DUIs in 2011 and 2012.  For this reason, I will drop 2011 and 2012 from my dataset as well.  Since Uber came in May 2014, I will use 2013 through 2015.  This will give me 16 months of data to look at before Uber (January 2013 - April 2014) and and 18 months of data to look at after Uber arrived (June 2014 - December 2015).  

Let's make my new dataset.  Since I have the dates as an index, I can just use the ix command to select which years I want to include.


```python
dui_df = dui.ix['2013': '2015']
```

#### DUIs Per Month


```python
yogi = dui_df.resample('M')['description'].count().plot(kind='line', figsize=[10,4], color = 'orange', fontsize = 14)
yogi.axvline(x = '2014-05', color = 'k', linewidth = 3)
yogi.set_ylabel('Number of DUIs')
yogi.text('2014-06', 62, 'Uber Arrival')
yogi.set_xlabel('')
yogi.set_title('Number of DUIs in KC by Month')
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/dui_by_month.png' />

I have included a vertical line for Uber's arrival of May 2014 (Uber arrived in KC on 5/9/2014 according to this [article](http://www.pitch.com/news/article/20565177/uber-has-officially-arrived-in-kansas-city)).  Everything appears to be pretty similar except for December 2013. For this reason, I'm going to make a seperate dataset and check to see if I have the full data for December 2013.

My dataset is called df_check and I'm first going to convert 'reported_date' to a date/time type.


```python
df_check = df_2013[['description', 'reported_date']]
```


```python
phantom = pd.DataFrame(df_check['reported_date'].str.split('T').tolist(), columns = ['date', 'time'])
df_check['date'] = phantom['date']
df_check['date'] = pd.to_datetime(phantom['date'], format = '%Y-%m-%d')
df_check.drop('reported_date', axis =1, inplace =True)
```

Next, I want select only the month of December and do a resample to see the number of incidents per day.


```python
dec_check = df_check.ix['2013-12-01': '2013-12-31']
dec_2013 = dec_check.resample('D')['description'].count().plot(kind = 'line', figsize = [10,6])
dec_2013.set_title('Number of Incidents per Day for December 2013')
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/december.png' />

As you can see the data stops on 12/22, so the data is incomplete.  For this reason, I will not include the December data due to it being incomplete.

Next, I want to compare monthly averages before and after Uber came to Kansas City.  


```python
dui_before = dui_df['2013': '2014-04']
dui_after = dui_df['2014-06': '2015']
```

Next, I want to make a new dataframe with the mean number of DUIs per month before and after Uber arrived.


```python
# Reset Index to Get Month and Year
dui_before.reset_index(inplace = True)
dui_after.reset_index(inplace = True)
# Get Month 
dui_before['month'] = dui_before['date'].dt.month
dui_after['month'] = dui_after['date'].dt.month
# Get Year
dui_before['year'] = dui_before['date'].dt.year
dui_after['year'] = dui_after['date'].dt.year
# Get Total DUIs Per Month
total_before = dui_before.groupby('month')['month'].count()
total_after = dui_after.groupby('month')['month'].count()
# Get Number of Months (to get mean of each month) and calculate mean
dui_before_months = dui_before.groupby(['month'])['year'].nunique()
dui_mean_before = total_before.astype('float') / dui_before_months
dui_after_months = dui_after.groupby('month')['year'].nunique()
dui_mean_after = total_after.astype('float') / dui_after_months
# Make new Dataframe with calculated data of mean DUIs per month for before and after Uber
data = {'Before Uber': dui_mean_before, 'After Uber': dui_mean_after}
df = pd.DataFrame(data, columns = ['Before Uber', 'After Uber'])
# Drop December due to incomplete data
df.drop([12], inplace = True)
```

If you're still with me, we can finally plot!!

#### Number of DUIs per Month BU (Before Uber) and AU (After Uber)


```python
mike_modano = df.plot(kind = 'line', figsize = [12, 8], fontsize = 14, rot=0)
mike_modano.set_title('Number of DUIs per Month Before and After Uber Arrived')
mike_modano.set_xlabel('Month')
mike_modano.set_ylabel('Number of DUIs')
```

<img src = 'https://raw.githubusercontent.com/sik-flow/uber_dui_kc/master/Pics/before_after.png' />

January and February both had more DUIs after Uber arrived, but after those months there was less DUIs after Uber arrived.  Perhaps due to people not willing to stand outside in the cold and wait for an Uber? What I find most interesting is the trends for DUIs - increases over the summer months and drops down in the fall months.

Finally lets calculate the percent difference of DUIs before and after Uber arrived.


```python
sum_after = df['After Uber'].sum()
sum_before = df['Before Uber'].sum()

percent_diff = round(((abs(sum_after - sum_before)) / sum_before) *100, 2)
percent_diff
```




    7.65



#### Key Takeaways

After Uber arrived DUIs went down 7.65% per month.  Remember that this data is only from January 2013 through November 2015 due to inconsistent data.  Additionally, I removed December 2013, December 2014, and December 2015 also due to inconsistent data.  This analysis could be improved by getting a more consistent data set and looking at other ride sharing programs.  Last thing I want to look at is the average age of someone that is arrested for a DUI.


```python
age_2015 = df_2015[df_2015['description'] == 'Driving Under Influe'][['description', 'reported_date', 'age_1']]
age_2014 = df_2014[df_2014['description'] == 'Driving Under Influe'][['description', 'reported_date', 'age_1']]
age_2013 = df_2013[df_2013['description'] == 'Driving Under Influe'][['description', 'reported_date', 'age']]
age = pd.concat([age_2015, age_2014, age_2013])
age = age.fillna(age['age_1'])
age['age'].describe()
```




    count    805.000000
    mean      33.617391
    std       10.971478
    min       16.000000
    25%       25.000000
    50%       30.000000
    75%       40.000000
    max       76.000000
    Name: age, dtype: float64



Mean age is 33.6 and median age is 30.  I feel that one of the reasons ride sharing programs have an impact on DUIs is because 40% of Uber users are between the ages of 25 and 34 ([click here for source](https://www.globalwebindex.net/blog/the-demographics-of-ubers-us-users)) and, as you can see, the mean and median age of DUIs arrests is of a similar age group.

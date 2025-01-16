# An analysis of Vision Zero in Road Safety

## Introduction

In 1997, the Swedish Parliament decided that Vision Zero should be the basis for road safety work in Sweden. In the long term, no one should be killed or seriously injured in the road transport system. Despite significant progress in recent decades, challenges remain to be addressed in order to achieve this ambitious goal.

This analysis examines traffic data from 2003 to 2023, including trends in the number of road accidents, the contribution of different transport modes to accidents and serious injuries, and seasonal variations. The results aim to identify where interventions are needed to reduce accidents and improve road safety.

The data represents the year period 2003-2023 from Transportstyrelsen's side. 

Data has been collected from Transportstyrelsen and been analyzed to get an understanding for: 

- Number of deaths and serious injuries over the years.

- Which types of vehicles and accident types contribute most to the statistics.

- Seasonal variations and other patterns.

In this project, I aim to study whether Sweden is moving towards the Vision Zero in road safety. I will analyze traffic data to identify trends in fatalities and severe injuries over time. I will examine the most common causes of accidents, as well as which modes of transport and accident types contribute most to the statistics. The goal is to gain insights into where improvements are needed to further reduce fatalities and injuries on the roads.

Source: https://www.transportstyrelsen.se/sv/om-oss/statistik-och-analys/statistik-inom-vagtrafik/olycksstatistik/statistik-over-vagtrafikolyckor/

#### Import data


```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from scipy.stats import zscore
```


```python
file_path = "/Users/alvinalindqvist/Desktop/nationell_arstatistik.xlsx"
data = pd.ExcelFile(file_path)
print('Sheets in the excel:', data.sheet_names)
```

    Sheets in the excel: ['Info', 'Omkomna', 'Skadade (P+S)', 'Skadade (RPMI)', 'Skadade gående (RPMI)']


## Fatalities in traffic


```python
fatalities_data = pd.read_excel(file_path, sheet_name = "Omkomna", header = 8)

fatalities_data = fatalities_data.dropna(axis = 1, how = "all")
fatalities_data = fatalities_data.dropna(axis = 0, how = 'any')

fatalities_data['År'] = pd.to_numeric(fatalities_data['År'], errors='coerce').astype(int)

print('Number of fatalities')
fatalities_data.head()
```

    Number of fatalities





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
      <th>År</th>
      <th>Jan</th>
      <th>Feb</th>
      <th>Mar</th>
      <th>Apr</th>
      <th>Maj</th>
      <th>Jun</th>
      <th>Jul</th>
      <th>Aug</th>
      <th>Sep</th>
      <th>Okt</th>
      <th>Nov</th>
      <th>Dec</th>
      <th>Summa total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2003</td>
      <td>31</td>
      <td>29</td>
      <td>36</td>
      <td>39</td>
      <td>43</td>
      <td>55</td>
      <td>55</td>
      <td>53</td>
      <td>40</td>
      <td>39</td>
      <td>42</td>
      <td>62</td>
      <td>524.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004</td>
      <td>25</td>
      <td>29</td>
      <td>26</td>
      <td>36</td>
      <td>37</td>
      <td>64</td>
      <td>45</td>
      <td>57</td>
      <td>39</td>
      <td>47</td>
      <td>29</td>
      <td>46</td>
      <td>480.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>30</td>
      <td>32</td>
      <td>38</td>
      <td>30</td>
      <td>28</td>
      <td>43</td>
      <td>49</td>
      <td>34</td>
      <td>41</td>
      <td>41</td>
      <td>28</td>
      <td>46</td>
      <td>440.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2006</td>
      <td>21</td>
      <td>28</td>
      <td>26</td>
      <td>38</td>
      <td>33</td>
      <td>50</td>
      <td>50</td>
      <td>49</td>
      <td>39</td>
      <td>28</td>
      <td>35</td>
      <td>48</td>
      <td>445.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2007</td>
      <td>33</td>
      <td>28</td>
      <td>27</td>
      <td>30</td>
      <td>39</td>
      <td>50</td>
      <td>57</td>
      <td>51</td>
      <td>50</td>
      <td>32</td>
      <td>34</td>
      <td>40</td>
      <td>471.0</td>
    </tr>
  </tbody>
</table>
</div>



An overview of the data we can see that there are many people who have lost ther life in traffic. The highest yearly total is 524 fatalities in 2003, and the lowest is 204 in 2020, which may be because of reduced travel during the pandemic. Since 2016, the yearly fatalities have stayed around 200–270, indicating some progress in reducing fatalities compared to earlier years. However, in 2022 and 2023, fatalities increased again to 227 and 229, raising concerns about the recent trend.

### Data prep - Fatalities Categorized by Modes of Travel


```python
fatalities_modesoftravel = pd.read_excel(file_path, sheet_name = "Omkomna", header = 84, nrows = 21)
fatalities_modesoftravel = fatalities_modesoftravel.dropna(axis = 1, how = 'all')
fatalities_modesoftravel.isnull().sum()
```




    År                  0
    Cykel               0
    Gående              0
    MC                  0
    Moped               0
    Övrigt              0
    Personbil           0
    Lastbil (lätt)      0
    Lastbil (tung)      0
    Lastbil (okänd)    11
    Buss                5
    Summa total         0
    dtype: int64



We can see that the column "Lastbil (okänd)" (Truck unknown) has 11 missing values, which means that the majority of its rows are empty. The column "Buss" (Bus) has 5 missing values.


```python
fatalities_modesoftravel.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 21 entries, 0 to 20
    Data columns (total 12 columns):
     #   Column           Non-Null Count  Dtype  
    ---  ------           --------------  -----  
     0   År               21 non-null     int64  
     1   Cykel            21 non-null     int64  
     2   Gående           21 non-null     int64  
     3   MC               21 non-null     int64  
     4   Moped            21 non-null     int64  
     5   Övrigt           21 non-null     int64  
     6   Personbil        21 non-null     int64  
     7   Lastbil (lätt)   21 non-null     int64  
     8   Lastbil (tung)   21 non-null     int64  
     9   Lastbil (okänd)  10 non-null     float64
     10  Buss             16 non-null     float64
     11  Summa total      21 non-null     int64  
    dtypes: float64(2), int64(10)
    memory usage: 2.1 KB


Columns like "År" (Year), "Cykel" (Bicycle), "Gående" (Pedestrian), etc., have 21 non-empty values. The column "Lastbil (okänd)" (Truck - unknown) has only 10 non-empty values, meaning that the remaining 11 rows contain missing values. The column "Buss" (Bus) has 16 non-empty values, so 5 rows have missing values.


```python
fatalities_modesoftravel = fatalities_modesoftravel.fillna(0)

fatalities_modesoftravel['År'] = pd.to_numeric(fatalities_modesoftravel['År'], errors='coerce').astype(int)

print('Fatalities Categorized by Modes of Travel')
fatalities_modesoftravel.head()
```

    Fatalities Categorized by Modes of Travel





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
      <th>År</th>
      <th>Cykel</th>
      <th>Gående</th>
      <th>MC</th>
      <th>Moped</th>
      <th>Övrigt</th>
      <th>Personbil</th>
      <th>Lastbil (lätt)</th>
      <th>Lastbil (tung)</th>
      <th>Lastbil (okänd)</th>
      <th>Buss</th>
      <th>Summa total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2003</td>
      <td>33</td>
      <td>55</td>
      <td>46</td>
      <td>9</td>
      <td>5</td>
      <td>344</td>
      <td>13</td>
      <td>6</td>
      <td>3.0</td>
      <td>10.0</td>
      <td>524</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004</td>
      <td>27</td>
      <td>67</td>
      <td>56</td>
      <td>18</td>
      <td>10</td>
      <td>284</td>
      <td>5</td>
      <td>5</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>480</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>38</td>
      <td>50</td>
      <td>46</td>
      <td>8</td>
      <td>7</td>
      <td>271</td>
      <td>11</td>
      <td>6</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>440</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2006</td>
      <td>26</td>
      <td>55</td>
      <td>55</td>
      <td>15</td>
      <td>7</td>
      <td>261</td>
      <td>9</td>
      <td>7</td>
      <td>0.0</td>
      <td>10.0</td>
      <td>445</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2007</td>
      <td>33</td>
      <td>58</td>
      <td>60</td>
      <td>14</td>
      <td>6</td>
      <td>276</td>
      <td>9</td>
      <td>6</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>471</td>
    </tr>
  </tbody>
</table>
</div>



### Data prep - Fatitities Categorized by Accident Type


```python
fatalities_accident_type = pd.read_excel(file_path, sheet_name = "Omkomna", header = 109, nrows = 22)

fatalities_accident_type = fatalities_accident_type.dropna(axis = 1, how = 'all')
fatalities_accident_type = fatalities_accident_type.fillna(0)

print('Fatalities Categorized by Accident Type')
fatalities_accident_type.head(3)
```

    Fatalities Categorized by Accident Type





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
      <th>År</th>
      <th>Av-svängande</th>
      <th>Cykel-moped</th>
      <th>Gående (påkörningsolyckor)</th>
      <th>Korsande kurs</th>
      <th>Möte</th>
      <th>Omkörning</th>
      <th>Singel</th>
      <th>Upp-hinnande</th>
      <th>Vilt</th>
      <th>Övrigt</th>
      <th>Summa total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2003</td>
      <td>20</td>
      <td>25</td>
      <td>54</td>
      <td>72</td>
      <td>112</td>
      <td>5.0</td>
      <td>163</td>
      <td>14</td>
      <td>6</td>
      <td>53</td>
      <td>524</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004</td>
      <td>9</td>
      <td>32</td>
      <td>60</td>
      <td>37</td>
      <td>89</td>
      <td>0.0</td>
      <td>160</td>
      <td>10</td>
      <td>12</td>
      <td>71</td>
      <td>480</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>15</td>
      <td>29</td>
      <td>51</td>
      <td>22</td>
      <td>100</td>
      <td>5.0</td>
      <td>158</td>
      <td>9</td>
      <td>8</td>
      <td>43</td>
      <td>440</td>
    </tr>
  </tbody>
</table>
</div>



For the data regarding fatalities based on modes of travel and accident types, I handled NaN values by replacing them with 0's. Upon examining the data in the Excel file, I observed that the NaN values represented empty entries. I assume these values indicate 0 (no fatalities), as there are no actual 0 values anywhere else in the dataset. However, if there had been 0's in other parts of the dataset, I would have investigated further to determine whether these NaN values might instead signify missing data for those specific years and categories.

### Plotting Fatalities Data


```python
fig, ax = plt.subplots(2, 2, figsize=(18, 13))  

# Första diagrammet
fatalities_data = fatalities_data[(fatalities_data['År'] >= 2019)]

years = fatalities_data['År']
totals = fatalities_data['Summa total']

ax[0,0].plot(years, totals, marker='o')

ax[0,0].set_xticks(years) 
ax[0,0].set_xticklabels(years, rotation=45) 
ax[0,0].set_title('Total Traffic Fatalities Per Year', fontsize=14)
ax[0,0].set_xlabel('Year', fontsize=12)
ax[0,0].set_ylabel('Number of Fatalities', fontsize=12)
ax[0,0].grid(True, linestyle='--', alpha=0.6)


# Andra diagrammet
monthly_data = fatalities_data.iloc[:, 1:13] 
monthly_data.columns = fatalities_data.columns[1:13]  
monthly_data.index = fatalities_data['År'] 

monthly_data.T.plot(ax=ax[0, 1], legend=True, alpha=0.7)
ax[0,1].set_title('Monthly Traffic Fatalities Over the Years', fontsize=14)
ax[0,1].set_xlabel('Month', fontsize=12)
ax[0,1].set_ylabel('Number of Fatalities', fontsize=12)
ax[0,1].grid(True, linestyle='--', alpha=0.6)
ax[0,1].legend(loc='upper left', bbox_to_anchor=(1.02, 1), fontsize=10, title="Year")


# Tredje diagrammet
fatalities_modesoftravel = fatalities_modesoftravel[(fatalities_modesoftravel['År'] >= 2019)]
years1 = fatalities_modesoftravel['År']

for column in fatalities_modesoftravel.columns:
    if column != 'År' and column != 'Summa total':  
        ax[1, 0].plot(years1, fatalities_modesoftravel[column], marker='o', label=column)

ax[1,0].set_xticks(years1)
ax[1,0].set_xticklabels(years1, rotation=45)
ax[1,0].set_title('Total Traffic Fatalities by Mode of Travel (2019-2023)', fontsize=14)
ax[1,0].set_xlabel('Year', fontsize=12)
ax[1,0].set_ylabel('Number of Fatalities', fontsize=12)
ax[1,0].grid(True, linestyle='--', alpha=0.6)
ax[1,0].legend(title='Mode of Travel', fontsize=10, loc='upper left', bbox_to_anchor=(1.02, 1))

# Fjärde diagrammet
fatalities_accident_type = fatalities_accident_type[(fatalities_accident_type['År'] >= 2019)]
years2 = fatalities_accident_type['År']

for column in fatalities_accident_type.columns:
    if column != 'År' and column != 'Summa total':  
        ax[1, 1].plot(years2, fatalities_accident_type[column], marker='o', label=column)

ax[1,1].set_xticks(years2)
ax[1,1].set_xticklabels(years2, rotation=45)
ax[1,1].set_title('Total Traffic Fatalities by Accident Type (2019-2023)', fontsize=14)
ax[1,1].set_xlabel('Year', fontsize=12)
ax[1,1].set_ylabel('Number of Fatalities', fontsize=12)
ax[1,1].grid(True, linestyle='--', alpha=0.6)
ax[1,1].legend(title='Type of Accident', fontsize=10, loc='upper left', bbox_to_anchor=(1.02, 1))

plt.tight_layout()
plt.show()
```


    
![png](Project_code_files/Project_code_22_0.png)
    


#### Total Traffic Fatalities Per Year
Fatalities decreased significantly in 2020, likely due to reduced travel during the pandemic. However, numbers have increased steadily since then, with 2022 and 2023 showing concerning trends.

#### Monthly Traffic Fatalities Over the Years
Fatalities are higher in Januari month, and also the summer months, likely due to increased travel and outdoor activities. It is interesting that the month Mars has so few fatalities.

#### Total Traffic Fatalities by Mode of Travel
Car users account for the majority of fatalities, while bicycles, pedestrians, and motorcycles contribute less but remain relatively stable over the years. However, it is alarming that motorcycle fatalities remain high, especially considering they are mostly used during the summer months. Also fatalities involving bicycles appear to be on the rise. Light trucks accidents seems to fall, so that is possitive. 

#### Total Traffic Fatalities by Accident Type
Single accidents and head-on collisions are the most common causes of fatalities. However, head-on collisions has a negative trend in 2023 but single accidents seems to increse. Cycle-moped and Other is also an accident type that seems to have an increasing trend over the years, but luckily has a negative trend in 2023.  

## Plotting Fatalities Data, 2023


```python
fig, ax = plt.subplots(1, 2, figsize = (18,8))

# Fig 1
fatalities_modesoftravel_23 = fatalities_modesoftravel[fatalities_modesoftravel['År'] == 2023]

data_2023 = fatalities_modesoftravel_23.drop(columns=['År', 'Summa total'])

data_2023.T.plot(kind='bar', ax=ax[0], legend=False, width=0.8, color = 'Skyblue')  

ax[0].set_xticklabels(data_2023.columns, rotation=45) 


ax[0].set_title("Mode of Travel for 2023")
ax[0].set_xlabel("Transport Type")
ax[0].set_ylabel("Number of Fatilities")
ax[0].grid(axis='y')

#Fig 2
fatalities_accident_type23 = fatalities_accident_type[fatalities_accident_type['År'] == 2023]

data_2023 = fatalities_accident_type23.drop(columns=['År', 'Summa total'])

# Plotta histogram i den första subplott-axeln
data_2023.T.plot(kind='bar', ax=ax[1], legend=False, width=0.8, color = 'Lightblue') 

ax[1].set_xticklabels(data_2023.columns, rotation=45) 
ax[1].set_title("Accident Type for 2023")
ax[1].set_xlabel("AccidentType")
ax[1].set_ylabel("Number of Fatilities")
ax[1].grid(axis='y')

plt.show()
```


    
![png](Project_code_files/Project_code_25_0.png)
    


The plot from 2023 shows that car accidents account for the majority of fatalities. This is not surprising, as cars are in use throughout the day and year. Pedestrians, motorcyclists and cyclists also contribute significantly to the fatality numbers, reflecting their vulnerability in traffic.

Regarding accident types, single accidents are the most common, followed by head-on collisions. To reduce fatalities, we could focus on better road design, teaching drivers to be safer, and making sure traffic rules are followed more strictly. It would be interesting to investigate why Single accidents are at such a high level. Could alcohol or other factors, like road conditions, be a cause?

## Severely injured

When examining the severely injured, I will select the sheet 'Skadade (RPMI)' instead of "Skadade (P+S)" because it is more closely related to the vision zero concept. The injury numbers are not whole numbers because they are averages calculated using RPMI1%. This makes the data easier to compare across years and categories but does not reflect exact counts of injured people.


```python
Severe_injured = pd.read_excel(file_path, sheet_name='Skadade (RPMI)', header=7, nrows=8).iloc[:, :14]
Severe_injured.fillna(0, inplace=True)  

very_Severe_injured = pd.read_excel(file_path, sheet_name="Skadade (RPMI)", header=7, nrows=8).iloc[:, 15:]
very_Severe_injured.columns = [col.replace('.1', '') for col in very_Severe_injured.columns]
very_Severe_injured.fillna(0, inplace=True)

if 'index' in Severe_injured.columns:
    Severe_injured.drop(columns=['index'], inplace=True)
if 'index' in very_Severe_injured.columns:
    very_Severe_injured.drop(columns=['index'], inplace=True)

Severe_injured.set_index("År", inplace=True)
very_Severe_injured.set_index("År", inplace=True)

total_injured = Severe_injured.add(very_Severe_injured, fill_value=0)

total_injured["Summa total"] = total_injured.iloc[:, :12].sum(axis=1)

total_injured.reset_index(inplace=True)

columns_order = ["År"] + list(Severe_injured.columns) 
total_injured = total_injured[columns_order]

total_injured
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
      <th>År</th>
      <th>Jan</th>
      <th>Feb</th>
      <th>Mar</th>
      <th>Apr</th>
      <th>Maj</th>
      <th>Jun</th>
      <th>Jul</th>
      <th>Aug</th>
      <th>Sep</th>
      <th>Okt</th>
      <th>Nov</th>
      <th>Dec</th>
      <th>Summa total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016</td>
      <td>280.070996</td>
      <td>258.434230</td>
      <td>243.090509</td>
      <td>322.461796</td>
      <td>537.661806</td>
      <td>509.769130</td>
      <td>474.463875</td>
      <td>483.662666</td>
      <td>471.730603</td>
      <td>388.719236</td>
      <td>336.200039</td>
      <td>285.921332</td>
      <td>4592.186219</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>302.464690</td>
      <td>232.206605</td>
      <td>289.115723</td>
      <td>291.609141</td>
      <td>457.820830</td>
      <td>467.427598</td>
      <td>448.833534</td>
      <td>480.412283</td>
      <td>389.028083</td>
      <td>398.631107</td>
      <td>418.615169</td>
      <td>304.644701</td>
      <td>4480.809463</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018</td>
      <td>284.870577</td>
      <td>209.537856</td>
      <td>201.683213</td>
      <td>306.815017</td>
      <td>514.480367</td>
      <td>467.339039</td>
      <td>404.142177</td>
      <td>389.922469</td>
      <td>358.440359</td>
      <td>361.970113</td>
      <td>289.135763</td>
      <td>251.142324</td>
      <td>4039.479272</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019</td>
      <td>262.879708</td>
      <td>188.848542</td>
      <td>213.901726</td>
      <td>306.898500</td>
      <td>381.292386</td>
      <td>451.944360</td>
      <td>390.951424</td>
      <td>469.482319</td>
      <td>367.217556</td>
      <td>356.343730</td>
      <td>287.597266</td>
      <td>245.962670</td>
      <td>3923.320187</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020</td>
      <td>221.605605</td>
      <td>226.426501</td>
      <td>186.500675</td>
      <td>245.374453</td>
      <td>320.681707</td>
      <td>459.576129</td>
      <td>348.950650</td>
      <td>420.050868</td>
      <td>352.747523</td>
      <td>280.978193</td>
      <td>236.700151</td>
      <td>170.265133</td>
      <td>3469.857588</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2021</td>
      <td>205.799644</td>
      <td>151.229909</td>
      <td>175.654799</td>
      <td>227.552460</td>
      <td>290.880163</td>
      <td>447.646706</td>
      <td>664.265634</td>
      <td>512.549546</td>
      <td>458.992373</td>
      <td>467.242455</td>
      <td>390.363906</td>
      <td>311.442368</td>
      <td>4303.619964</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2022</td>
      <td>289.447000</td>
      <td>302.959000</td>
      <td>332.157670</td>
      <td>387.619001</td>
      <td>542.507084</td>
      <td>648.845462</td>
      <td>635.146240</td>
      <td>614.698003</td>
      <td>507.243181</td>
      <td>473.608136</td>
      <td>382.147308</td>
      <td>305.286707</td>
      <td>5421.664792</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2023</td>
      <td>313.896883</td>
      <td>245.608820</td>
      <td>308.616447</td>
      <td>367.402267</td>
      <td>562.420357</td>
      <td>691.981862</td>
      <td>537.907598</td>
      <td>533.825044</td>
      <td>577.249220</td>
      <td>405.163884</td>
      <td>348.795760</td>
      <td>242.396143</td>
      <td>5135.264285</td>
    </tr>
  </tbody>
</table>
</div>




```python
Severe_injured_modeoftravel = pd.read_excel(file_path, sheet_name = 'Skadade (RPMI)', header = 49, nrows = 8).iloc[:,:12]
Very_Severe_injured_modeoftravel = pd.read_excel(file_path, sheet_name = 'Skadade (RPMI)', header = 49, nrows = 8).iloc[:,15:27]
```


```python
Very_Severe_injured_modeoftravel.columns = [col.replace(".1", "") for col in Very_Severe_injured_modeoftravel.columns]

if 'index' in Severe_injured_modeoftravel.columns:
    Severe_injured_modeoftravel.drop(columns=['index'], inplace=True)
    
if 'index' in Very_Severe_injured_modeoftravel.columns:
    Very_Severe_injured_modeoftravel.drop(columns=['index'], inplace=True)

Severe_injured_modeoftravel.set_index("År", inplace=True)
Very_Severe_injured_modeoftravel.set_index("År", inplace=True)

total_injured_modeoftravel = Severe_injured_modeoftravel.add(Very_Severe_injured_modeoftravel, fill_value=0)
total_injured_modeoftravel
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
      <th>Buss</th>
      <th>Cykel</th>
      <th>Gående</th>
      <th>Lastbil (lätt)</th>
      <th>Lastbil (okänd)</th>
      <th>Lastbil (tung)</th>
      <th>MC</th>
      <th>Moped</th>
      <th>Personbil</th>
      <th>Övrigt</th>
      <th>Summa total</th>
    </tr>
    <tr>
      <th>År</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016</th>
      <td>67.633647</td>
      <td>2084.106369</td>
      <td>230.458068</td>
      <td>32.314719</td>
      <td>6.061577</td>
      <td>20.952074</td>
      <td>269.346645</td>
      <td>245.815643</td>
      <td>1548.547712</td>
      <td>86.949765</td>
      <td>4592.186219</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>75.172767</td>
      <td>2103.966625</td>
      <td>213.801222</td>
      <td>29.007337</td>
      <td>8.063079</td>
      <td>21.518557</td>
      <td>238.514208</td>
      <td>227.672713</td>
      <td>1468.701736</td>
      <td>94.391219</td>
      <td>4480.809463</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>68.006540</td>
      <td>1977.225884</td>
      <td>199.266484</td>
      <td>24.583694</td>
      <td>10.967819</td>
      <td>18.343119</td>
      <td>211.039597</td>
      <td>221.953084</td>
      <td>1241.208970</td>
      <td>66.884081</td>
      <td>4039.479272</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>79.152114</td>
      <td>1949.809051</td>
      <td>192.226757</td>
      <td>22.748623</td>
      <td>8.303510</td>
      <td>12.362567</td>
      <td>192.881031</td>
      <td>227.770242</td>
      <td>1072.375046</td>
      <td>165.691246</td>
      <td>3923.320187</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>41.468231</td>
      <td>1930.681661</td>
      <td>142.503617</td>
      <td>15.090011</td>
      <td>2.742523</td>
      <td>11.658384</td>
      <td>207.622503</td>
      <td>206.436281</td>
      <td>805.506199</td>
      <td>106.148177</td>
      <td>3469.857588</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>45.888449</td>
      <td>2363.121408</td>
      <td>198.693690</td>
      <td>21.852761</td>
      <td>4.987820</td>
      <td>23.983724</td>
      <td>240.157174</td>
      <td>242.433644</td>
      <td>1072.859796</td>
      <td>89.641498</td>
      <td>4303.619964</td>
    </tr>
    <tr>
      <th>2022</th>
      <td>51.418204</td>
      <td>3118.983040</td>
      <td>304.693209</td>
      <td>23.075371</td>
      <td>6.393111</td>
      <td>19.979666</td>
      <td>306.983534</td>
      <td>283.672365</td>
      <td>1195.689239</td>
      <td>110.777054</td>
      <td>5421.664792</td>
    </tr>
    <tr>
      <th>2023</th>
      <td>71.452955</td>
      <td>2847.976931</td>
      <td>289.702190</td>
      <td>21.563760</td>
      <td>6.063687</td>
      <td>20.449861</td>
      <td>307.246313</td>
      <td>246.889829</td>
      <td>1203.685354</td>
      <td>120.233406</td>
      <td>5135.264285</td>
    </tr>
  </tbody>
</table>
</div>




```python
Severe_injured_accident_type = pd.read_excel(file_path, sheet_name = 'Skadade (RPMI)', header = 63, nrows = 8).iloc[:,:12]
Very_Severe_injured_accident_type = pd.read_excel(file_path, sheet_name = 'Skadade (RPMI)', header = 63, nrows = 8).iloc[:,15:27]
```


```python
Very_Severe_injured_accident_type.columns = [col.replace(".1", "") for col in Very_Severe_injured_accident_type.columns]

if 'index' in Severe_injured_accident_type.columns:
    Severe_injured_accident_type.drop(columns=['index'], inplace=True)
    
if 'index' in Very_Severe_injured_accident_type.columns:
    Very_Severe_injured_accident_type.drop(columns=['index'], inplace=True)

Severe_injured_accident_type.set_index("År", inplace=True)
Very_Severe_injured_accident_type.set_index("År", inplace=True)

total_injured_accident_type = Severe_injured_accident_type.add(Very_Severe_injured_accident_type, fill_value=0)
total_injured_accident_type
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
      <th>Av-svängande</th>
      <th>Cykel-moped</th>
      <th>Gående (påkörningsolyckor)</th>
      <th>Korsande kurs</th>
      <th>Möte</th>
      <th>Omkörning</th>
      <th>Singel</th>
      <th>Upp-hinnande</th>
      <th>Vilt</th>
      <th>Övrigt</th>
      <th>Summa total</th>
    </tr>
    <tr>
      <th>År</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016</th>
      <td>111.386141</td>
      <td>2286.451239</td>
      <td>233.680008</td>
      <td>170.655064</td>
      <td>142.259137</td>
      <td>43.230269</td>
      <td>741.414338</td>
      <td>585.760478</td>
      <td>76.990552</td>
      <td>200.358991</td>
      <td>4592.186219</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>96.318221</td>
      <td>2289.171873</td>
      <td>216.378661</td>
      <td>149.351299</td>
      <td>137.849685</td>
      <td>42.980866</td>
      <td>681.864460</td>
      <td>572.559148</td>
      <td>74.825490</td>
      <td>219.509758</td>
      <td>4480.809463</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>94.687734</td>
      <td>2172.497657</td>
      <td>196.636296</td>
      <td>138.707011</td>
      <td>138.405545</td>
      <td>40.203443</td>
      <td>537.006064</td>
      <td>471.003165</td>
      <td>70.309075</td>
      <td>180.023281</td>
      <td>4039.479272</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>77.986423</td>
      <td>2143.723271</td>
      <td>197.647143</td>
      <td>119.750470</td>
      <td>115.145439</td>
      <td>31.934718</td>
      <td>445.697590</td>
      <td>451.248495</td>
      <td>68.973694</td>
      <td>271.212945</td>
      <td>3923.320187</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>66.852547</td>
      <td>2096.516721</td>
      <td>145.391743</td>
      <td>98.396555</td>
      <td>64.783812</td>
      <td>23.941272</td>
      <td>440.296763</td>
      <td>289.801199</td>
      <td>50.305498</td>
      <td>193.571479</td>
      <td>3469.857588</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>78.085344</td>
      <td>2537.808226</td>
      <td>155.333276</td>
      <td>113.358714</td>
      <td>110.064961</td>
      <td>27.378685</td>
      <td>569.516118</td>
      <td>364.616856</td>
      <td>66.293751</td>
      <td>281.164032</td>
      <td>4303.619964</td>
    </tr>
    <tr>
      <th>2022</th>
      <td>90.560077</td>
      <td>3323.994331</td>
      <td>221.632033</td>
      <td>115.995299</td>
      <td>125.873175</td>
      <td>26.810474</td>
      <td>705.630322</td>
      <td>380.692200</td>
      <td>59.463370</td>
      <td>371.013514</td>
      <td>5421.664792</td>
    </tr>
    <tr>
      <th>2023</th>
      <td>86.839286</td>
      <td>3022.304630</td>
      <td>213.807009</td>
      <td>118.503450</td>
      <td>168.974227</td>
      <td>22.310866</td>
      <td>677.449834</td>
      <td>387.976672</td>
      <td>70.139447</td>
      <td>366.958864</td>
      <td>5135.264285</td>
    </tr>
  </tbody>
</table>
</div>



## Plotting (Very) Severely Injured Data


```python
fig, ax = plt.subplots(2, 2, figsize=(18, 13)) 

# Fig 1
total_injured = total_injured[total_injured['År'].astype(int) >= 2019]

years3 = total_injured['År']
totals3 = total_injured['Summa total']

ax[0,0].set_xticks(years3)  # Sätt hela år som tick-värden
ax[0,0].plot(years3, totals3, marker='o')
ax[0,0].set_xlabel('År')
ax[0,0].set_ylabel('Number of Injured')
ax[0,0].set_title('Total of (Very) Serverly injured per year, 2019-2023')
ax[0,0].grid(True)

# Fig 2
monthly_data1 = total_injured.iloc[:, 1:13]  
monthly_data1.columns = total_injured.columns[1:13]  
monthly_data1.index = total_injured['År']  

monthly_data1.T.plot(ax=ax[0, 1], legend=True, alpha=0.7)
ax[0,1].set_title('Monthly Total of (Very) Serverly injured Over the Years', fontsize=14)
ax[0,1].set_xlabel('Month', fontsize=12)
ax[0,1].set_ylabel('Number of Injured', fontsize=12)
ax[0,1].grid(True, linestyle='--', alpha=0.6)
ax[0,1].legend(loc='upper left', bbox_to_anchor=(1.02, 1), fontsize=10, title="Year")

#Fig 3
total_injured_modeoftravel = total_injured_modeoftravel.reset_index()
total_injured_modeoftravel = total_injured_modeoftravel[(total_injured_modeoftravel['År'] >= 2019)]
years4 = total_injured_modeoftravel['År']

for column in total_injured_modeoftravel.columns:
    if column != 'År' and column != 'Summa total':  # ta bort "År" och "Summa total"
        ax[1, 0].plot(years4, total_injured_modeoftravel[column], marker='o', label=column)

ax[1,0].set_xticks(years4)
ax[1,0].set_xticklabels(years4, rotation=45)
ax[1,0].set_title('Total of (Very) Serverly injured per year by Mode of Travel, 2019-2023', fontsize=14)
ax[1,0].set_xlabel('Year', fontsize=12)
ax[1,0].set_ylabel('Number of Injured', fontsize=12)
ax[1,0].grid(True, linestyle='--', alpha=0.6)
ax[1,0].legend(title='Mode of Travel', fontsize=10, loc='upper left', bbox_to_anchor=(0.7, 1))

#Fig 4
total_injured_accident_type = total_injured_accident_type.reset_index()
total_injured_accident_type = total_injured_accident_type[(total_injured_accident_type['År'] >= 2019)]
years5 = total_injured_accident_type['År']

for column in total_injured_accident_type.columns:
    if column != 'År' and column != 'Summa total':  # ta bort "År" och "Summa total"
        ax[1, 1].plot(years5, total_injured_accident_type[column], marker='o', label=column)

ax[1,1].set_xticks(years5)
ax[1,1].set_xticklabels(years5, rotation=45)
ax[1,1].set_title('Total of (Very) Serverly injured per year by Accident Type, 2019-2023', fontsize=14)
ax[1,1].set_xlabel('Year', fontsize=12)
ax[1,1].set_ylabel('Number of Injured', fontsize=12)
ax[1,1].grid(True, linestyle='--', alpha=0.6)
ax[1,1].legend(title='Accident Type', fontsize=10, loc='upper left', bbox_to_anchor=(1.02, 1))

plt.show()
```


    
![png](Project_code_files/Project_code_35_0.png)
    


#### Total of (Very) Severely Injured per Year (2019–2023)
The first plot highlights a concerning trend. There is though a noticeable dip in 2020, likely due the pandemic, but the numbers have risen significantly in the following years. Especially 2022 was a dark year. Despite a slight improvement in 2023, the overall trend is moving in the wrong direction.

#### Monthly Total of (Very) Severely Injured Over the Years
The second plot shows a seasonal pattern in the injuries, with peaks during the summer months. Increased outdoor activities and travel during these months may contribute to higher injury rates. January also shows higher numbers, which may be attributed to the weather conditions or post-holiday traffic. 

#### Total of (Very) Severely Injured per Year by Mode of Travel (2019–2023)
The third plot shows that car users account for the majority of severe injuries, which aligns with their dominance on the roads. However, pedestrians and cyclists also represent a substantial proportion. unfortantly the numbers for cyclists appear to be increasing.

#### Total of (Very) Severely Injured per Year by Accident Type (2019–2023)
The fourth plot shows that cycles-moped accidents are the most common cause of severe injuries. It is interesting to note that single accidents and head-on collisions have a very high fatality rate, yet the injury level is relatively low. In contrast, cycle-moped accidents result in a very high number of injuries butfewer fatalities. This may suggest that single accidents and head-on accidents are often more severe, with fewer people surviving them, while cycle-moped accidents, despite causing many injuries, may be less fatal.

## Combine Fatilities and Total injured

I will be using merge() to combine data. 


```python
fatalities_total = fatalities_data[['År', 'Summa total']].rename(columns={'Summa total': 'Number of Fatilities'})
injuries_total = total_injured[['År', 'Summa total']].rename(columns={'Summa total': 'Number of Serverly injured'})

combined_totals = pd.merge(fatalities_total, injuries_total,on='År',how='inner')

combined_totals
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
      <th>År</th>
      <th>Number of Fatilities</th>
      <th>Number of Serverly injured</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019</td>
      <td>221.0</td>
      <td>3923.320187</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020</td>
      <td>204.0</td>
      <td>3469.857588</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021</td>
      <td>210.0</td>
      <td>4303.619964</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022</td>
      <td>227.0</td>
      <td>5421.664792</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2023</td>
      <td>229.0</td>
      <td>5135.264285</td>
    </tr>
  </tbody>
</table>
</div>




```python
combined_totals['Fatalities Change (%)'] = combined_totals['Number of Fatilities'].pct_change() * 100
combined_totals['Injured Change (%)'] = combined_totals['Number of Serverly injured'].pct_change() * 100
combined_totals
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
      <th>År</th>
      <th>Number of Fatilities</th>
      <th>Number of Serverly injured</th>
      <th>Fatalities Change (%)</th>
      <th>Injured Change (%)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019</td>
      <td>221.0</td>
      <td>3923.320187</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020</td>
      <td>204.0</td>
      <td>3469.857588</td>
      <td>-7.692308</td>
      <td>-11.558134</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021</td>
      <td>210.0</td>
      <td>4303.619964</td>
      <td>2.941176</td>
      <td>24.028720</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022</td>
      <td>227.0</td>
      <td>5421.664792</td>
      <td>8.095238</td>
      <td>25.979172</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2023</td>
      <td>229.0</td>
      <td>5135.264285</td>
      <td>0.881057</td>
      <td>-5.282520</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

While the long-term trend shows improvement, with fatalities decreasing from 300–500 per years ago to just over 200 in recent years, the short-term trend is worrying. Fatalities and severe injuries have risen steadily since 2020, and 2022 being particularly concerning. Although there was a bit improvement in 2023, the numbers remain high and far from the Vision Zero goal.

Cyclists and pedestrians are increasingly vulnerable, while single-vehicle and head-on collisions remain the deadliest accident types. Seasonal patterns were clear, with injuries peaking in summer months, likely due to increased travel and outdoor activities. Interestingly, January also showed relatively high numbers, potentially linked to post-holiday traffic and winter conditions.

When examining transportation modes, car users accounted for the majority of fatalities and injuries, reflecting their activness on the roads. Cyclists and pedestrians also represented a significant share, and the increasing injuries among cyclists are concerning. Meanwhile, accident types revealed that single accidents and head-on collisions are the deadliest, with high fatality rates but lower injury numbers. In contrast, cycle-moped accidents result in many injuries but fewer fatalities, suggesting differences in severity and survivability.

To reverse the recent negative trend and make further progress, targeted actions such as improving road safety for vulnerable groups, addressing the causes of single accidents, and strengthening enforcement of traffic regulations are crucial. Achieving Vision Zero will require consistent and innovative efforts to make traffic safer for all road users.

Recommendations could be
- Improve infrastructure with safer crossings and separate bike lanes.
- Educate motorists and vulnerable road users on safety.
- Use technology like automatic braking systems.
- Focus on seasonal safety measures for summer and winter.
- Enforce stricter regulations, like speed limits and seat belt use.

Targeted actions aligned with Vision Zero can make traffic safer for all.


```python
!jupyter nbconvert --to markdown hProject.code-Copy1.ipynb
```

    [NbConvertApp] WARNING | pattern 'hProject.code-Copy1.ipynb' matched no files
    This application is used to convert notebook files (*.ipynb) to various other
    formats.
    
    WARNING: THE COMMANDLINE INTERFACE MAY CHANGE IN FUTURE RELEASES.
    
    Options
    =======
    The options below are convenience aliases to configurable class-options,
    as listed in the "Equivalent to" description-line of the aliases.
    To see all configurable class-options for some <cmd>, use:
        <cmd> --help-all
    
    --debug
        set log level to logging.DEBUG (maximize logging output)
        Equivalent to: [--Application.log_level=10]
    --generate-config
        generate default config file
        Equivalent to: [--JupyterApp.generate_config=True]
    -y
        Answer yes to any questions instead of prompting.
        Equivalent to: [--JupyterApp.answer_yes=True]
    --execute
        Execute the notebook prior to export.
        Equivalent to: [--ExecutePreprocessor.enabled=True]
    --allow-errors
        Continue notebook execution even if one of the cells throws an error and include the error message in the cell output (the default behaviour is to abort conversion). This flag is only relevant if '--execute' was specified, too.
        Equivalent to: [--ExecutePreprocessor.allow_errors=True]
    --stdin
        read a single notebook file from stdin. Write the resulting notebook with default basename 'notebook.*'
        Equivalent to: [--NbConvertApp.from_stdin=True]
    --stdout
        Write notebook output to stdout instead of files.
        Equivalent to: [--NbConvertApp.writer_class=StdoutWriter]
    --inplace
        Run nbconvert in place, overwriting the existing notebook (only 
        relevant when converting to notebook format)
        Equivalent to: [--NbConvertApp.use_output_suffix=False --NbConvertApp.export_format=notebook --FilesWriter.build_directory=]
    --clear-output
        Clear output of current file and save in place, 
        overwriting the existing notebook.
        Equivalent to: [--NbConvertApp.use_output_suffix=False --NbConvertApp.export_format=notebook --FilesWriter.build_directory= --ClearOutputPreprocessor.enabled=True]
    --no-prompt
        Exclude input and output prompts from converted document.
        Equivalent to: [--TemplateExporter.exclude_input_prompt=True --TemplateExporter.exclude_output_prompt=True]
    --no-input
        Exclude input cells and output prompts from converted document. 
        This mode is ideal for generating code-free reports.
        Equivalent to: [--TemplateExporter.exclude_output_prompt=True --TemplateExporter.exclude_input=True]
    --allow-chromium-download
        Whether to allow downloading chromium if no suitable version is found on the system.
        Equivalent to: [--WebPDFExporter.allow_chromium_download=True]
    --log-level=<Enum>
        Set the log level by value or name.
        Choices: any of [0, 10, 20, 30, 40, 50, 'DEBUG', 'INFO', 'WARN', 'ERROR', 'CRITICAL']
        Default: 30
        Equivalent to: [--Application.log_level]
    --config=<Unicode>
        Full path of a config file.
        Default: ''
        Equivalent to: [--JupyterApp.config_file]
    --to=<Unicode>
        The export format to be used, either one of the built-in formats
        ['asciidoc', 'custom', 'html', 'latex', 'markdown', 'notebook', 'pdf',
        'python', 'rst', 'script', 'slides', 'webpdf'] or a dotted object name that
        represents the import path for an `Exporter` class
        Default: ''
        Equivalent to: [--NbConvertApp.export_format]
    --template=<Unicode>
        Name of the template to use
        Default: ''
        Equivalent to: [--TemplateExporter.template_name]
    --template-file=<Unicode>
        Name of the template file to use
        Default: None
        Equivalent to: [--TemplateExporter.template_file]
    --writer=<DottedObjectName>
        Writer class used to write the  results of the conversion
        Default: 'FilesWriter'
        Equivalent to: [--NbConvertApp.writer_class]
    --post=<DottedOrNone>
        PostProcessor class used to write the results of the conversion
        Default: ''
        Equivalent to: [--NbConvertApp.postprocessor_class]
    --output=<Unicode>
        overwrite base name use for output files. can only be used when converting
        one notebook at a time.
        Default: ''
        Equivalent to: [--NbConvertApp.output_base]
    --output-dir=<Unicode>
        Directory to write output(s) to. Defaults to output to the directory of each
        notebook. To recover previous default behaviour (outputting to the current
        working directory) use . as the flag value.
        Default: ''
        Equivalent to: [--FilesWriter.build_directory]
    --reveal-prefix=<Unicode>
        The URL prefix for reveal.js (version 3.x). This defaults to the reveal CDN,
        but can be any url pointing to a copy  of reveal.js.
        For speaker notes to work, this must be a relative path to a local  copy of
        reveal.js: e.g., "reveal.js".
        If a relative path is given, it must be a subdirectory of the current
        directory (from which the server is run).
        See the usage documentation
        (https://nbconvert.readthedocs.io/en/latest/usage.html#reveal-js-html-
        slideshow) for more details.
        Default: ''
        Equivalent to: [--SlidesExporter.reveal_url_prefix]
    --nbformat=<Enum>
        The nbformat version to write. Use this to downgrade notebooks.
        Choices: any of [1, 2, 3, 4]
        Default: 4
        Equivalent to: [--NotebookExporter.nbformat_version]
    
    Examples
    --------
    
        The simplest way to use nbconvert is
        
        > jupyter nbconvert mynotebook.ipynb --to html
        
        Options include ['asciidoc', 'custom', 'html', 'latex', 'markdown', 'notebook', 'pdf', 'python', 'rst', 'script', 'slides', 'webpdf'].
        
        > jupyter nbconvert --to latex mynotebook.ipynb
        
        Both HTML and LaTeX support multiple output templates. LaTeX includes
        'base', 'article' and 'report'.  HTML includes 'basic' and 'full'. You
        can specify the flavor of the format used.
        
        > jupyter nbconvert --to html --template lab mynotebook.ipynb
        
        You can also pipe the output to stdout, rather than a file
        
        > jupyter nbconvert mynotebook.ipynb --stdout
        
        PDF is generated via latex
        
        > jupyter nbconvert mynotebook.ipynb --to pdf
        
        You can get (and serve) a Reveal.js-powered slideshow
        
        > jupyter nbconvert myslides.ipynb --to slides --post serve
        
        Multiple notebooks can be given at the command line in a couple of 
        different ways:
        
        > jupyter nbconvert notebook*.ipynb
        > jupyter nbconvert notebook1.ipynb notebook2.ipynb
        
        or you can specify the notebooks list in a config file, containing::
        
            c.NbConvertApp.notebooks = ["my_notebook.ipynb"]
        
        > jupyter nbconvert --config mycfg.py
    
    To see all available configurables, use `--help-all`.
    


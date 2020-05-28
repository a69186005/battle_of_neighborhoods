# Housing Sales Prices & Education Resource of Sydney

## 1. Introduce

### 1.1 Background

Sydeney is one of the biggest city in Austrilia. There are more than 5 million people live in the city. Sydney is a culturally diverse city, and immigrants from all over the world come to the city every year.

Immigrants who come to this city have a need to buy houses. In the process of buying a house, people often consider the education, business, life and transportation in the area where the house is located. Especially immigrants from East Asian countries, they want their children to have a good education, and they want to have good schools near where they live.

Therefore, this project will analyze the data of average house prices, educational resources, living facilities and public transport in various urban areas of Sydney, so as to provide housing purchase advice for new immigrants.

###  1.2 Data Description

The Sydney suburbs data is from "https://en.wikipedia.org/wiki/List_of_Sydney_suburbs".  
The average of housing sales prices are from "https://www.realestate.com.au/neighbourhoods/".  
The Ranking of school in Sydeny Data is from "https://bettereducation.com.au/results/hsc.aspx".  
I'll use Google Map, ‘Search Nearby’ option to get the center coordinates of the each suburb.   
I'll use Forsquare API to get the most common venues of given suburbs of Sydney. 

## 2. Data Processing

### 2.1 Collect Sydney suburbs data

#### Import libraries


```python
import pandas as pd
import numpy as np

import re
import requests
from bs4 import BeautifulSoup

print("import libraries success!")
```

    import libraries success!
    

#### Get Sydney suburbs' name


```python
## Make a suburb Dataframe
res = requests.get('https://en.wikipedia.org/wiki/List_of_Sydney_suburbs')
soup = BeautifulSoup(res.text, 'html.parser')

suburb_names = []
for title in soup.find_all('a'):
    if ', New South Wales' in str(title.get('title')):
        suburb_names.append(str(title.get('title')).replace(', New South Wales', ''))
suburb_names = list(set(suburb_names))
len(suburb_names)
```




    688




```python
suburb_scc = pd.read_csv('SSC_2016_AUST.csv')
suburb_scc = suburb_scc.drop_duplicates('SSC_CODE_2016')
suburb_scc = suburb_scc[suburb_scc['STATE_NAME_2016'].isin(['New South Wales'])]
suburb_scc.replace(' (\()(.*?)(\))', '', regex=True, inplace=True)
```


```python
suburb_scc = suburb_scc[suburb_scc['SSC_NAME_2016'].isin(suburb_names)]
suburb_scc = suburb_scc.drop(['MB_CODE_2016', 'STATE_CODE_2016', 'STATE_NAME_2016', 'AREA_ALBERS_SQKM'], axis=1)
suburb_scc.reset_index(drop=True, inplace=True)
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12162</td>
      <td>Kingswood</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11758</td>
      <td>Greendale</td>
    </tr>
    <tr>
      <th>3</th>
      <td>12921</td>
      <td>Nelson</td>
    </tr>
    <tr>
      <th>4</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>683</th>
      <td>13701</td>
      <td>Sutherland</td>
    </tr>
    <tr>
      <th>684</th>
      <td>12371</td>
      <td>Loftus</td>
    </tr>
    <tr>
      <th>685</th>
      <td>11422</td>
      <td>Engadine</td>
    </tr>
    <tr>
      <th>686</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
    </tr>
    <tr>
      <th>687</th>
      <td>14422</td>
      <td>Woronora Heights</td>
    </tr>
  </tbody>
</table>
<p>688 rows × 2 columns</p>
</div>




```python
scc_list = suburb_scc.iloc[:,0].values
```

#### Get the top three languages used most in all suburbs except English


```python
rlst_language = []
r2nd_language = []
r3rd_language = []
suburb_language = {}
for scc in scc_list:
    
    res = requests.get('https://quickstats.censusdata.abs.gov.au/census_services/getproduct/census/2016/quickstat/SSC'+ str(scc) +'?opendocument')
    suburb_soup = BeautifulSoup(res.text, 'html.parser')

    suburb_item = {}
    # suburb_item['suburb_name'] = suburb_soup.h2.text
    try:
        languageStr = str(suburb_soup.find_all('script')[-4])
        languageData = re.search(r'languageData = \[([^\]]+)\]', languageStr, flags=0)
        languageStr = str(languageData.group(1)).replace('\n\t\t\t', '')
        temps = re.split(r'\{([^\}]+)\}', languageStr)
        temparray = []
        for temp in temps:
            if temp !='' and temp !=',':
                temparray.append(temp)
        strList1 = temparray[0].split(',')
        content = re.search(r'\"([^\"]+)\"', strList1[0], flags=0).group(1)
        # content1 = re.search(r'\"([^\"]+)\"', strList1[1], flags=0).group(1)
        rlst_language.append(content)
        # suburb_item['1st_precent'] = content1
        strList2 = temparray[1].split(',')
        content = re.search(r'\"([^\"]+)\"', strList2[0], flags=0).group(1)
        # content1 = re.search(r'\"([^\"]+)\"', strList2[1], flags=0).group(1)
        r2nd_language.append(content)
        # suburb_item['2nd_precent'] = content1
        strList3 = temparray[2].split(',')
        content = re.search(r'\"([^\"]+)\"', strList3[0], flags=0).group(1)
        # content1 = re.search(r'\"([^\"]+)\"', strList3[1], flags=0).group(1)
        r3rd_language.append(content)
        # suburb_item['3st_precent'] = content1
    except:
        rlst_language.append(np.nan)
        r2nd_language.append(np.nan)
        r3rd_language.append(np.nan)

suburb_language['1st_language'] = rlst_language
suburb_language['2nd_language'] = r2nd_language
suburb_language['3rd_language'] = r3rd_language
```

#### Merge the suburb language data to suburb_scc dataframe


```python
suburb_scc['1st_language'] = suburb_language["1st_language"]
suburb_scc['2nd_language'] = suburb_language["2nd_language"]
suburb_scc['3rd_language'] = suburb_language["3rd_language"]
suburb_scc = suburb_scc.dropna()
suburb_scc.reset_index(drop=True, inplace=True)
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 5 columns</p>
</div>



#### Get suburb's geocode from google


```python
GOOGLE_KEY = 'AIzaSyDXx3Bap5YUZ1AiXms5nLh3O9EffbFHHug'
def get_coordinates(api_key, address, verbose=False):
    try:
        url = 'https://maps.googleapis.com/maps/api/geocode/json?key={}&address={}'.format(api_key, address)
        response = requests.get(url).json()
        if verbose:
            print('Google Maps API JSON result =>', response)
        results = response['results']
        geographical_data = results[0]['geometry']['location'] # get geographical coordinates
        lat = geographical_data['lat']
        lon = geographical_data['lng']
        return [lat, lon]
    except:
        return [None, None]
    
# address = 'Wisemans Ferry, New South Wales, Australia'
# Wisemans_center = get_coordinates('', address)
# print('Coordinate of {}: {}'.format(address, Wisemans_center))
```


```python
address_list = suburb_scc.iloc[:,1].values
base_address = " New South Wales, Australia"
scc_lat = []
scc_lon = []
scc_location = {}
for address in address_list:
    select_address = address + base_address
    address_coordinate = get_coordinates(GOOGLE_KEY, select_address)
    scc_lat.append(address_coordinate[0])
    scc_lon.append(address_coordinate[1])
    print(address_coordinate)
scc_location['Latitude'] = scc_lat
scc_location['Longitude'] = scc_lon
```

```python
suburb_scc['Latitude'] = scc_location['Latitude']
suburb_scc['Longitude'] = scc_location['Longitude']
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
      <td>-34.068333</td>
      <td>151.115833</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
      <td>-33.381944</td>
      <td>150.985000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
      <td>-33.890334</td>
      <td>151.196610</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-34.054620</td>
      <td>150.712500</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-31.783056</td>
      <td>152.534444</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
      <td>-34.029742</td>
      <td>151.059275</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.046500</td>
      <td>151.047100</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.065722</td>
      <td>151.012664</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
      <td>-34.055200</td>
      <td>151.032100</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
      <td>-34.030434</td>
      <td>151.030555</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 7 columns</p>
</div>



#### Crawl the top primary school data of sydney


```python
res = requests.get('https://bettereducation.com.au/school/Primary/nsw/nsw_top_primary_schools.aspx')
soup_school = BeautifulSoup(res.text, 'html.parser')
soup_school.find_all('td')
primary_schools = {}
school_areas = []
school_names = []
school_ranking = []
ranking = 1
for td in soup_school.find_all('td'):
    try:
        if 'School' in td.find('a').contents[0] or 'College' in td.find('a').contents[0]:
            school_str = td.find('a').contents[0]
            school_split = school_str.split(',')
            school_names.append(school_split[0])
            school_areas.append(school_split[1])
            school_ranking.append(ranking)
            ranking = ranking + 1
    except:
        pass
    
primary_schools['Primary_school_name'] = school_names[0:100]
primary_schools['Primary_school_area'] = school_areas[0:100]
primary_schools['Primary_school_ranking'] = school_ranking[0:100]
primary_dataframe = pd.DataFrame(columns=['Primary_school_name','SSC_NAME_2016','Primary_school_ranking'])
primary_dataframe['Primary_school_name'] = primary_schools['Primary_school_name']
primary_dataframe['SSC_NAME_2016'] = primary_schools['Primary_school_area']
primary_dataframe['Primary_school_ranking'] = primary_schools['Primary_school_ranking']
primary_dataframe['Primary_school_ranking'] = primary_dataframe['Primary_school_ranking'].apply(lambda x: ',' + str(x))
primary_dataframe1 = primary_dataframe.groupby(by='SSC_NAME_2016')['Primary_school_ranking'].sum()
primary_ranking = pd.DataFrame(primary_dataframe1)
primary_ranking['Primary_school_ranking'] = primary_ranking['Primary_school_ranking'].apply(lambda x :[x[1:]])
primary_ranking
suburb_scc = pd.merge(suburb_scc, primary_ranking, on='SSC_NAME_2016', how='left')
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 8 columns</p>
</div>



#### Crawl the top secondary school data of sydney


```python
res = requests.get('https://bettereducation.com.au/school/Secondary/nsw/nsw_top_secondary_schools.aspx')
soup_school = BeautifulSoup(res.text, 'html.parser')
soup_school.find_all('td')
secondary_schools = {}
school_areas = []
school_names = []
school_ranking = []
ranking = 1
for td in soup_school.find_all('td'):
    try:
        if 'School' in td.find('a').contents[0] or 'College' in td.find('a').contents[0] or 'Armidale' in td.find('a').contents[0] or 'House' in td.find('a').contents[0] or 'Kambala' in td.find('a').contents[0] or 'SCEGGS' in td.find('a').contents[0] or 'Loreto' in td.find('a').contents[0]:
            school_str = td.find('a').contents[0]
            school_split = school_str.split(',')
            school_names.append(school_split[0])
            school_areas.append(school_split[1])
            school_ranking.append(ranking)
            ranking = ranking + 1
    except:
        pass
    
secondary_schools['Secondary_school_name'] = school_names[0:100]
secondary_schools['Secondary_school_area'] = school_areas[0:100]
secondary_schools['Secondary_school_ranking'] = school_ranking[0:100]
secondary_dataframe = pd.DataFrame(columns=['Secondary_school_name','SSC_NAME_2016','Secondary_school_ranking'])
secondary_dataframe['Secondary_school_name'] = secondary_schools['Secondary_school_name']
secondary_dataframe['SSC_NAME_2016'] = secondary_schools['Secondary_school_area']
secondary_dataframe['Secondary_school_ranking'] = secondary_schools['Secondary_school_ranking']
secondary_dataframe['Secondary_school_ranking'] = secondary_dataframe['Secondary_school_ranking'].apply(lambda x: ',' + str(x))
secondary_dataframe1 = secondary_dataframe.groupby(by='SSC_NAME_2016')['Secondary_school_ranking'].sum()
secondary_ranking = pd.DataFrame(secondary_dataframe1)
secondary_ranking['Secondary_school_ranking'] = secondary_ranking['Secondary_school_ranking'].apply(lambda x :[x[1:]])
secondary_ranking
suburb_scc = pd.merge(suburb_scc, secondary_ranking, on='SSC_NAME_2016', how='left')
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
      <th>Secondary_school_ranking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 9 columns</p>
</div>



#### Crawl the top Yr12ATAR school data of sydney


```python
res = requests.get('https://bettereducation.com.au/Results/Hsc.aspx')
soup_school = BeautifulSoup(res.text, 'html.parser')
soup_school.find_all('td')
yr12_schools = {}
school_areas = []
school_names = []
school_ranking = []
ranking = 1
for td in soup_school.find_all('td'):
    try:
        if 'School' in td.find('a').contents[0] or 'College' in td.find('a').contents[0] or 'Armidale' in td.find('a').contents[0] or 'House' in td.find('a').contents[0] or 'Kambala' in td.find('a').contents[0] or 'SCEGGS' in td.find('a').contents[0] or 'Loreto' in td.find('a').contents[0]:
            school_str = td.find('a').contents[0]
            school_split = school_str.split(',')
            school_names.append(school_split[0])
        school_ranking.append(ranking)
        ranking = ranking + 1
    except:
        pass
    try:
        if 'NSW' in td.find('font').contents[0]:
            school_str = td.find('font').contents[0]
            school_split = school_str.split(',')
            school_areas.append(school_split[0])
    except:
        pass

yr12_schools['yr12_school_name'] = school_names[0:100]
yr12_schools['yr12_school_area'] = school_areas[0:100]
yr12_schools['yr12_school_ranking'] = school_ranking[0:100]
yr12_dataframe = pd.DataFrame(columns=['yr12_school_name','SSC_NAME_2016','yr12_school_ranking'])
yr12_dataframe['yr12_school_name'] = yr12_schools['yr12_school_name']
yr12_dataframe['SSC_NAME_2016'] = yr12_schools['yr12_school_area']
yr12_dataframe['yr12_school_ranking'] = yr12_schools['yr12_school_ranking']
yr12_dataframe['yr12_school_ranking'] = yr12_dataframe['yr12_school_ranking'].apply(lambda x: ',' + str(x))
yr12_dataframe1 = yr12_dataframe.groupby(by='SSC_NAME_2016')['yr12_school_ranking'].sum()
yr12_ranking = pd.DataFrame(yr12_dataframe1)
yr12_ranking['yr12_school_ranking'] = yr12_ranking['yr12_school_ranking'].apply(lambda x :[x[1:]])
suburb_scc = pd.merge(suburb_scc, yr12_ranking, on='SSC_NAME_2016', how='left')
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
      <th>Secondary_school_ranking</th>
      <th>yr12_school_ranking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 10 columns</p>
</div>



#### Crawl the average housing price of Sydney suburbs


```python
import json

f = open('./Sydney_postcode.txt')
lines = f.readlines()
price_suburb_name = []
housing_price_list = []
housing_price_data = {}
BASE_PRICE_URL = 'https://www.propertyvalue.com.au/map/'
for line in lines:
    items = line.split('\t')
    suburb_post = items[0].replace(' ', '-') + '-' + items[1] + '-' + items[2].replace('\n', '')
    suburb_name = items[0]
    try:
        
        send_headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36",
            "Connection": "keep-alive",
            "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
            "Accept-Language": "zh-CN,zh;q=0.8"}
        res = requests.get(BASE_PRICE_URL + suburb_post +'/buy', headers=send_headers)
        script = re.search(r'var searchInfoWinDetails = ".*"', res.text).group()
        price_data = re.search(r'".*"', script).group()

        price_json = json.loads(price_data)
        price_json = json.loads(price_json)
        housing_price = price_json['HOUSE']['medianValue'].replace('$', '')
    
        if 'm' in housing_price:
            housing_price = float(housing_price.replace('m', '')) * 1000
            print(housing_price)
            housing_price_list.append(housing_price)
            price_suburb_name.append(suburb_name)
        if 'k' in housing_price:
            housing_price = float(housing_price.replace('k', ''))
            print(housing_price)
            housing_price_list.append(housing_price)
            price_suburb_name.append(suburb_name)
    except:
        pass
    
housing_price_data['SSC_NAME_2016'] = price_suburb_name
housing_price_data['housing_price_list'] = housing_price_list
housing_price_data
```

```python
price_dataframe = pd.DataFrame(columns=['SSC_NAME_2016','The median of Housing price (k Dollars)'])
price_dataframe['SSC_NAME_2016'] = housing_price_data['SSC_NAME_2016']
price_dataframe['The median of Housing price (k Dollars)'] = housing_price_data['housing_price_list']
price_dataframe
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
      <th>SSC_NAME_2016</th>
      <th>The median of Housing price (k Dollars)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbotsbury</td>
      <td>927.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Abbotsford</td>
      <td>2200.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Airds</td>
      <td>451.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alexandria</td>
      <td>1600.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alfords Point</td>
      <td>1200.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>528</th>
      <td>Yagoona</td>
      <td>804.0</td>
    </tr>
    <tr>
      <th>529</th>
      <td>Yarrawarrah</td>
      <td>1000.0</td>
    </tr>
    <tr>
      <th>530</th>
      <td>Yellow Rock</td>
      <td>770.0</td>
    </tr>
    <tr>
      <th>531</th>
      <td>Yennora</td>
      <td>670.0</td>
    </tr>
    <tr>
      <th>532</th>
      <td>Yowie Bay</td>
      <td>1900.0</td>
    </tr>
  </tbody>
</table>
<p>533 rows × 2 columns</p>
</div>




```python
suburb_scc = pd.merge(suburb_scc, price_dataframe, on='SSC_NAME_2016', how='left')
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
      <th>Secondary_school_ranking</th>
      <th>yr12_school_ranking</th>
      <th>The median of Housing price (k Dollars)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>Italian</td>
      <td>German</td>
      <td>French</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2400.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>German</td>
      <td>Urdu</td>
      <td>Afrikaans</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>Tagalog</td>
      <td>Cantonese</td>
      <td>Afrikaans</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1400.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>660.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>Gaelic (Scotland)</td>
      <td>Irish</td>
      <td>Welsh</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>Mandarin</td>
      <td>Russian</td>
      <td>Cantonese</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1100.0</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1000.0</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>Mandarin</td>
      <td>Spanish</td>
      <td>Italian</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>917.0</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>Greek</td>
      <td>German</td>
      <td>Italian</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1000.0</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>Greek</td>
      <td>German</td>
      <td>Cantonese</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1100.0</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 11 columns</p>
</div>




```python
language_mapping = {
    'Afrikaans': 1,
    'Arabic': 2,
    'Armenian': 3,
    'Assyrian Neo-Aramaic': 4,
    'Auslan': 5,
    'Bengali': 6,
    'Cantonese': 7,
    'Chaldean Neo-Aramaic': 8,
    'Croatian': 9,
    'Danish': 10,
    'Dari': 11,
    'Dutch': 12,
    'Estonian': 13,
    'Filipino': 14,
    'French': 15,
    'Gaelic (Scotland)': 16,
    'German': 17,
    'Greek': 18,
    'Gujarati': 19,
    'Hindi': 20,
    'Hungarian': 21,
    'Indonesian': 22,
    'Irish': 23,
    'Italian': 24,
    'Japanese': 25,
    'Karen': 26,
    'Khmer': 27,
    'Korean': 28,
    'Macedonian': 29,
    'Malayalam': 30,
    'Maltese': 31,
    'Mandarin': 32,
    'Mongolian': 33,
    'Nepali': 34,
    'Persian (excluding Dari)': 35,
    'Polish': 36,
    'Portuguese': 37,
    'Punjabi': 38,
    'Russian': 39,
    'Samoan': 40,
    'Serbian': 41,
    'Spanish': 42,
    'Swedish': 43,
    'Tagalog': 44,
    'Tamil': 45,
    'Thai': 46,
    'Tibetan': 47,
    'Tongan': 48,
    'Turkish': 49,
    'Urdu': 50,
    'Vietnamese': 51,
    'Welsh': 52
}
suburb_scc['1st_language'] = suburb_scc['1st_language'].map(language_mapping)
suburb_scc['2nd_language'] = suburb_scc['2nd_language'].map(language_mapping)
suburb_scc['3rd_language'] = suburb_scc['3rd_language'].map(language_mapping)
suburb_scc
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
      <th>Secondary_school_ranking</th>
      <th>yr12_school_ranking</th>
      <th>The median of Housing price (k Dollars)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>24.0</td>
      <td>17.0</td>
      <td>15</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2400.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>17.0</td>
      <td>50.0</td>
      <td>1</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>44.0</td>
      <td>7.0</td>
      <td>1</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1400.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>16.0</td>
      <td>23.0</td>
      <td>52</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>660.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>16.0</td>
      <td>23.0</td>
      <td>52</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>32.0</td>
      <td>39.0</td>
      <td>7</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1100.0</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>32.0</td>
      <td>42.0</td>
      <td>24</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1000.0</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>32.0</td>
      <td>42.0</td>
      <td>24</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>917.0</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>18.0</td>
      <td>17.0</td>
      <td>24</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1000.0</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>18.0</td>
      <td>17.0</td>
      <td>7</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1100.0</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 11 columns</p>
</div>




```python
suburb_scc.fillna(0, inplace=True)
```


```python
suburb_scc.to_csv('./suburb_data.csv')
```

### Import some libraries to generate maps to visualize your neighborhoods and cluster them


```python
from geopy.geocoders import Nominatim
import matplotlib.cm as cm
import matplotlib.colors as colors
from sklearn.cluster import KMeans
import json
import folium # map rendering library
from pandas.io.json import json_normalize

print("libraries imported")
```

    libraries imported
    


```python
address = 'Sydney'

geolocator = Nominatim(user_agent="ny_explorer")
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of Sydney City are {}, {}.'.format(latitude, longitude))
```

    The geograpical coordinate of Sydney City are -33.8548157, 151.2164539.
    

### Create the neighborhoods marks on the map 


```python
# create map of New York using latitude and longitude values
map_sydney = folium.Map(location=[latitude, longitude], zoom_start=10)

# add markers to map
for lat, lng, suburb, price in zip(suburb_scc['Latitude'], suburb_scc['Longitude'], suburb_scc['SSC_NAME_2016'], suburb_scc['The median of Housing price (k Dollars)']):
    label = '{}, {}'.format(suburb, price)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_sydney)  
    
map_sydney
```
<img src="./1.png" height="300" width="600">



### Define Foursquare Credentials and Version


```python
CLIENT_ID = 'SMVI3FRU4PGES3H0FPRJZMTB4S5HIPK222NDSXWSULLUQPFM' # your Foursquare ID
CLIENT_SECRET = '2HTAV3YRQ0KIZ253M3WKPJ0D3LNQIHVENQMU1WZBIKNLXG3C' # your Foursquare Secret
VERSION = '20200508' # Foursquare API version

print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your credentails:
    CLIENT_ID: SMVI3FRU4PGES3H0FPRJZMTB4S5HIPK222NDSXWSULLUQPFM
    CLIENT_SECRET:2HTAV3YRQ0KIZ253M3WKPJ0D3LNQIHVENQMU1WZBIKNLXG3C
    

### Explore Sydney City


```python
LIMIT = 100

def getNearbyVenues(names, latitudes, longitudes, radius=1500):
    
    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
        print(name)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        try:
            results = requests.get(url).json()["response"]['groups'][0]['items']

            # return only relevant information for each nearby venue
            venues_list.append([(
                name, 
                lat, 
                lng, 
                v['venue']['name'], 
                v['venue']['location']['lat'], 
                v['venue']['location']['lng'],  
                v['venue']['categories'][0]['name']) for v in results])
        except:
            pass

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['SSC_NAME_2016', 
                  'Suburb Latitude', 
                  'Suburb Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```


```python
sydney_venues = getNearbyVenues(names=suburb_scc['SSC_NAME_2016'],
                                   latitudes=suburb_scc['Latitude'],
                                   longitudes=suburb_scc['Longitude']
                                  )
```


```python
print(sydney_venues.shape)
sydney_venues.head()
```

    (17494, 7)
    




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
      <th>SSC_NAME_2016</th>
      <th>Suburb Latitude</th>
      <th>Suburb Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Lilli Pilli</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>Lilli Pilli Point</td>
      <td>-34.067903</td>
      <td>151.115411</td>
      <td>Harbor / Marina</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Lilli Pilli</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>Lilli Pilli Pool</td>
      <td>-34.069622</td>
      <td>151.110933</td>
      <td>Pool</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Lilli Pilli</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>Port hacking</td>
      <td>-34.069645</td>
      <td>151.110258</td>
      <td>Boat or Ferry</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Lilli Pilli</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>Lilli Pilli Oval</td>
      <td>-34.062513</td>
      <td>151.119718</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Lilli Pilli</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>Lilli Pilli Patisserie Cafe</td>
      <td>-34.058971</td>
      <td>151.120923</td>
      <td>Bakery</td>
    </tr>
  </tbody>
</table>
</div>



### Analysis each suburb


```python
# one hot encoding
sydney_onehot = pd.get_dummies(sydney_venues[['Venue Category']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
sydney_onehot['SSC_NAME_2016'] = sydney_venues['SSC_NAME_2016'] 

# move neighborhood column to the first column
fixed_columns = [sydney_onehot.columns[-1]] + list(sydney_onehot.columns[:-1])
sydney_onehot = sydney_onehot[fixed_columns]

sydney_grouped = sydney_onehot.groupby('SSC_NAME_2016').mean().reset_index()
sydney_grouped
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
      <th>SSC_NAME_2016</th>
      <th>ATM</th>
      <th>Accessories Store</th>
      <th>Advertising Agency</th>
      <th>Afghan Restaurant</th>
      <th>Airport</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>American Restaurant</th>
      <th>Arcade</th>
      <th>...</th>
      <th>Warehouse Store</th>
      <th>Water Park</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
      <th>Zoo</th>
      <th>Zoo Exhibit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbotsbury</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Abbotsford</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Acacia Gardens</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Agnes Banks</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Airds</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>529</th>
      <td>Yarramundi</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>530</th>
      <td>Yarrawarrah</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>531</th>
      <td>Yennora</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>532</th>
      <td>Yowie Bay</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>533</th>
      <td>Zetland</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>534 rows × 369 columns</p>
</div>




```python
num_top_venues = 5

for hood in sydney_grouped['SSC_NAME_2016']:
    print("----"+hood+"----")
    temp = sydney_grouped[sydney_grouped['SSC_NAME_2016'] == hood].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```

    ----Abbotsbury----
                      venue  freq
    0                  Park  0.14
    1  Other Great Outdoors  0.14
    2     Convenience Store  0.14
    3    Italian Restaurant  0.07
    4           Supermarket  0.07
    
    
    ----Abbotsford----
                       venue  freq
    0                   Park  0.24
    1                   Café  0.15
    2  Australian Restaurant  0.06
    3        Thai Restaurant  0.06
    4     Italian Restaurant  0.03
    
    
    ----Acacia Gardens----
                      venue  freq
    0  Fast Food Restaurant  0.14
    1           Pizza Place  0.11
    2           Supermarket  0.08
    3        Sandwich Place  0.06
    4           Gas Station  0.06
    
    
    ----Agnes Banks----
                     venue  freq
    0           Campground   0.5
    1  Rental Car Location   0.5
    2          Opera House   0.0
    3      Paintball Field   0.0
    4         Outlet Store   0.0
    
    
    ----Airds----
                    venue  freq
    0      Baseball Field   0.2
    1         Supermarket   0.2
    2         Gas Station   0.2
    3  Chinese Restaurant   0.2
    4       Shopping Mall   0.2
    
    
    ----Alexandria----
                      venue  freq
    0                  Café  0.26
    1                   Pub  0.07
    2           Coffee Shop  0.06
    3    Italian Restaurant  0.05
    4  Gym / Fitness Center  0.05
    
    
    ----Alfords Point----
               venue  freq
    0  National Park  0.25
    1  Shopping Mall  0.25
    2     Playground  0.25
    3    Flea Market  0.25
    4            ATM  0.00
    
    
    ----Allambie Heights----
                      venue  freq
    0                  Café  0.11
    1      Department Store  0.11
    2  Fast Food Restaurant  0.11
    3                Bakery  0.11
    4        Sandwich Place  0.05
    
    
    ----Allawah----
                      venue  freq
    0  Fast Food Restaurant  0.08
    1                   Pub  0.06
    2    Chinese Restaurant  0.06
    3                Bakery  0.05
    4    Dim Sum Restaurant  0.05
    
    
    ----Ambarvale----
                      venue  freq
    0                  Café  0.18
    1  Fast Food Restaurant  0.09
    2           Supermarket  0.06
    3         Shopping Mall  0.06
    4          Dessert Shop  0.03
    
    
    ----Annandale----
                    venue  freq
    0                Café  0.31
    1  Italian Restaurant  0.08
    2                 Pub  0.06
    3        Liquor Store  0.05
    4    Sushi Restaurant  0.04
    
    
    ----Arcadia----
                   venue  freq
    0      Garden Center  0.25
    1  Food & Drink Shop  0.25
    2        Flower Shop  0.25
    3       Soccer Field  0.25
    4                ATM  0.00
    
    
    ----Arncliffe----
                    venue  freq
    0                Park  0.10
    1       Train Station  0.08
    2            Platform  0.05
    3  Chinese Restaurant  0.03
    4                 Pub  0.03
    
    
    ----Artarmon----
                     venue  freq
    0                 Café  0.14
    1      Thai Restaurant  0.06
    2  Japanese Restaurant  0.05
    3          Pizza Place  0.04
    4                  Pub  0.04
    
    
    ----Ashbury----
                     venue  freq
    0                 Café  0.06
    1                 Park  0.06
    2          Supermarket  0.06
    3  Dumpling Restaurant  0.05
    4    Convenience Store  0.05
    
    
    ----Ashcroft----
                      venue  freq
    0  Fast Food Restaurant  0.43
    1           Music Venue  0.14
    2     Convenience Store  0.14
    3        Sandwich Place  0.14
    4           Pizza Place  0.14
    
    
    ----Asquith----
                      venue  freq
    0  Fast Food Restaurant  0.22
    1         Bowling Alley  0.11
    2        Coffee Roaster  0.11
    3           Gas Station  0.11
    4          Soccer Field  0.11
    
    
    ----Auburn----
                           venue  freq
    0                Supermarket  0.09
    1  Middle Eastern Restaurant  0.09
    2              Grocery Store  0.06
    3                     Bakery  0.06
    4            Thai Restaurant  0.06
    
    
    ----Austral----
                       venue  freq
    0                   Park  0.25
    1  Australian Restaurant  0.25
    2     Athletics & Sports  0.25
    3      Martial Arts Dojo  0.25
    4                    ATM  0.00
    
    
    ----Avalon Beach----
                     venue  freq
    0                 Café  0.20
    1               Bakery  0.13
    2  Japanese Restaurant  0.07
    3          Coffee Shop  0.07
    4                Beach  0.07
    
    
    ----Badgerys Creek----
                       venue  freq
    0              Pet Store   1.0
    1                    ATM   0.0
    2                 Office   0.0
    3  Outdoors & Recreation   0.0
    4   Outdoor Supply Store   0.0
    
    
    ----Balgowlah----
               venue  freq
    0           Café  0.23
    1           Park  0.07
    2  Shopping Mall  0.05
    3          Beach  0.05
    4    Supermarket  0.05
    
    
    ----Balgowlah Heights----
                venue  freq
    0           Beach  0.22
    1            Park  0.17
    2  Scenic Lookout  0.09
    3   Grocery Store  0.09
    4      Restaurant  0.04
    
    
    ----Balmain----
             venue  freq
    0         Café  0.23
    1          Pub  0.09
    2         Park  0.06
    3          Bar  0.05
    4  Pizza Place  0.05
    
    
    ----Bangor----
                       venue  freq
    0   Fast Food Restaurant  0.10
    1          Grocery Store  0.10
    2     Italian Restaurant  0.05
    3  Australian Restaurant  0.05
    4                    Pub  0.05
    
    
    ----Banksia----
                      venue  freq
    0           Coffee Shop  0.07
    1                   Gym  0.05
    2           Supermarket  0.05
    3  Fast Food Restaurant  0.05
    4                Bakery  0.05
    
    
    ----Barangaroo----
              venue  freq
    0          Café  0.12
    1         Hotel  0.08
    2  Cocktail Bar  0.05
    3     Speakeasy  0.05
    4   Coffee Shop  0.04
    
    
    ----Barden Ridge----
              venue  freq
    0  Soccer Field  0.25
    1        Arcade  0.25
    2   Pizza Place  0.25
    3   Golf Course  0.25
    4           ATM  0.00
    
    
    ----Bardia----
               venue  freq
    0    Candy Store  0.25
    1  Train Station  0.25
    2           Café  0.25
    3    Gas Station  0.25
    4   Optical Shop  0.00
    
    
    ----Bardwell Park----
                      venue  freq
    0                  Café  0.24
    1         Train Station  0.10
    2           Pizza Place  0.07
    3                   Gym  0.07
    4  Fast Food Restaurant  0.07
    
    
    ----Bardwell Valley----
                      venue  freq
    0         Train Station  0.09
    1                  Café  0.09
    2  Fast Food Restaurant  0.07
    3              Platform  0.07
    4         Grocery Store  0.04
    
    
    ----Bass Hill----
                      venue  freq
    0  Fast Food Restaurant  0.17
    1                   Gym  0.11
    2         Shopping Mall  0.11
    3        Sandwich Place  0.06
    4        Ice Cream Shop  0.06
    
    
    ----Baulkham Hills----
                     venue  freq
    0                 Café  0.15
    1               Bakery  0.10
    2          Pizza Place  0.10
    3   Italian Restaurant  0.05
    4  Dumpling Restaurant  0.05
    
    
    ----Bayview----
                 venue  freq
    0  Harbor / Marina  0.25
    1             Park  0.12
    2    Boat or Ferry  0.12
    3            Trail  0.12
    4            Beach  0.12
    
    
    ----Beacon Hill----
                      venue  freq
    0  Fast Food Restaurant  0.15
    1                  Pool  0.08
    2            Restaurant  0.08
    3                  Café  0.08
    4                   Gym  0.08
    
    
    ----Beaumont Hills----
              venue  freq
    0          Park  0.18
    1          Café  0.18
    2           Pub  0.09
    3      Bus Stop  0.09
    4  Liquor Store  0.09
    
    
    ----Belfield----
             venue  freq
    0         Park  0.17
    1  Pizza Place  0.17
    2  Music Venue  0.08
    3  Gas Station  0.08
    4    BBQ Joint  0.08
    
    
    ----Bella Vista----
                    venue  freq
    0                Café  0.22
    1      Sandwich Place  0.09
    2  Italian Restaurant  0.07
    3     Thai Restaurant  0.07
    4         Pizza Place  0.04
    
    
    ----Bellevue Hill----
                     venue  freq
    0                 Café  0.17
    1                 Park  0.08
    2  Japanese Restaurant  0.06
    3                Beach  0.06
    4               Bakery  0.04
    
    
    ----Belrose----
               venue  freq
    0    Supermarket  0.13
    1     Restaurant  0.07
    2           Park  0.07
    3    Coffee Shop  0.07
    4  Shopping Mall  0.07
    
    
    ----Berala----
                       venue  freq
    0                   Park  0.20
    1          Train Station  0.13
    2                 Garden  0.07
    3  Vietnamese Restaurant  0.07
    4            Pizza Place  0.07
    
    
    ----Berowra----
                    venue  freq
    0            Platform  0.17
    1              Bakery  0.08
    2      Sandwich Place  0.08
    3                Park  0.08
    4  Chinese Restaurant  0.08
    
    
    ----Berowra Waters----
                       venue  freq
    0     Seafood Restaurant   0.4
    1        Harbor / Marina   0.2
    2          Boat or Ferry   0.2
    3  Australian Restaurant   0.2
    4           Optical Shop   0.0
    
    
    ----Berrilee----
                       venue  freq
    0                Stables   1.0
    1           Nudist Beach   0.0
    2  Outdoors & Recreation   0.0
    3   Outdoor Supply Store   0.0
    4      Other Repair Shop   0.0
    
    
    ----Beverly Hills----
                   venue  freq
    0               Café  0.14
    1        Pizza Place  0.07
    2               Park  0.07
    3                Pub  0.07
    4  Convenience Store  0.07
    
    
    ----Bexley----
                      venue  freq
    0     Convenience Store  0.10
    1  Fast Food Restaurant  0.10
    2                  Park  0.10
    3           Coffee Shop  0.07
    4                Bakery  0.07
    
    
    ----Bexley North----
               venue  freq
    0           Café  0.26
    1           Park  0.15
    2   Intersection  0.04
    3  Train Station  0.04
    4            Gym  0.04
    
    
    ----Bidwill----
                   venue  freq
    0  Convenience Store   0.4
    1               Park   0.2
    2        High School   0.2
    3                Bar   0.2
    4       Optical Shop   0.0
    
    
    ----Bilgola Beach----
                     venue  freq
    0                 Café  0.25
    1                Beach  0.12
    2          Coffee Shop  0.08
    3  Japanese Restaurant  0.04
    4           Restaurant  0.04
    
    
    ----Bilgola Plateau----
                     venue  freq
    0                 Café  0.25
    1                Beach  0.19
    2      Harbor / Marina  0.19
    3   Seafood Restaurant  0.06
    4  Japanese Restaurant  0.06
    
    
    ----Birchgrove----
                    venue  freq
    0                Café  0.21
    1                Park  0.10
    2                 Bar  0.06
    3                 Pub  0.06
    4  Italian Restaurant  0.05
    
    
    ----Birrong----
                    venue  freq
    0       Train Station  0.24
    1       Grocery Store  0.12
    2                 ATM  0.06
    3  Athletics & Sports  0.06
    4       Bowling Green  0.06
    
    
    ----Blackett----
                         venue  freq
    0     Fast Food Restaurant  0.17
    1            Shopping Mall  0.17
    2             Soccer Field  0.08
    3      Fried Chicken Joint  0.08
    4  Fruit & Vegetable Store  0.08
    
    
    ----Blair Athol----
                      venue  freq
    0  Fast Food Restaurant  0.09
    1                  Café  0.07
    2         Shopping Mall  0.05
    3           Supermarket  0.05
    4             Juice Bar  0.04
    
    
    ----Blairmount----
               venue  freq
    0         Bakery  0.25
    1   Liquor Store  0.25
    2  Moving Target  0.25
    3            Spa  0.25
    4    Opera House  0.00
    
    
    ----Blakehurst----
                     venue  freq
    0                 Park  0.11
    1                 Café  0.11
    2   Seafood Restaurant  0.11
    3  Sporting Goods Shop  0.06
    4          Pizza Place  0.06
    
    
    ----Bligh Park----
                      venue  freq
    0  Fast Food Restaurant  0.33
    1                  Park  0.17
    2     Convenience Store  0.17
    3    Basketball Stadium  0.17
    4         Grocery Store  0.17
    
    
    ----Bondi Junction----
                     venue  freq
    0                 Café  0.18
    1                  Pub  0.06
    2          Pizza Place  0.06
    3                 Park  0.06
    4  Japanese Restaurant  0.04
    
    
    ----Bonnet Bay----
                     venue  freq
    0                 Café  0.19
    1               Bakery  0.12
    2        Grocery Store  0.12
    3      Thai Restaurant  0.12
    4  Japanese Restaurant  0.06
    
    
    ----Bonnyrigg----
                      venue  freq
    0  Gym / Fitness Center  0.12
    1           Pizza Place  0.12
    2           Supermarket  0.12
    3  Fast Food Restaurant  0.06
    4            Food Truck  0.06
    
    
    ----Bonnyrigg Heights----
                      venue  freq
    0                  Park  0.09
    1  Gym / Fitness Center  0.09
    2        Sandwich Place  0.09
    3                   Gym  0.09
    4     Convenience Store  0.09
    
    
    ----Bossley Park----
               venue  freq
    0           Park  0.12
    1    Bus Station  0.08
    2  Grocery Store  0.08
    3    Pizza Place  0.08
    4      Preschool  0.04
    
    
    ----Bow Bowing----
                    venue  freq
    0         Pizza Place  0.14
    1            Platform  0.14
    2  Chinese Restaurant  0.14
    3    Basketball Court  0.07
    4   Convenience Store  0.07
    
    
    ----Box Hill----
              venue  freq
    0           Pub  0.25
    1     BBQ Joint  0.25
    2          Food  0.25
    3  Home Service  0.25
    4           ATM  0.00
    
    
    ----Breakfast Point----
                       venue  freq
    0                   Park  0.19
    1                   Café  0.11
    2  Australian Restaurant  0.06
    3          Boat or Ferry  0.06
    4        Thai Restaurant  0.06
    
    
    ----Brighton-le-Sands----
              venue  freq
    0          Café  0.09
    1           Gym  0.07
    2  Dessert Shop  0.07
    3   Supermarket  0.05
    4   Pizza Place  0.05
    
    
    ----Bronte----
                       venue  freq
    0                   Café  0.18
    1            Pizza Place  0.06
    2                  Beach  0.06
    3  Australian Restaurant  0.06
    4                    Pub  0.04
    
    
    ----Brookvale----
                      venue  freq
    0                  Café  0.07
    1  Fast Food Restaurant  0.07
    2      Department Store  0.07
    3           Supermarket  0.07
    4                Bakery  0.05
    
    
    ----Bundeena----
               venue  freq
    0          Beach  0.40
    1           Café  0.20
    2            Pub  0.07
    3           Pier  0.07
    4  Grocery Store  0.07
    
    
    ----Burraneer----
                 venue  freq
    0  Harbor / Marina   0.2
    1            Trail   0.2
    2             Park   0.2
    3            Motel   0.2
    4            Beach   0.2
    
    
    ----Burwood----
                    venue  freq
    0   Korean Restaurant  0.13
    1                Café  0.13
    2  Chinese Restaurant  0.08
    3     Thai Restaurant  0.07
    4         Coffee Shop  0.04
    
    
    ----Burwood Heights----
                    venue  freq
    0                Café  0.13
    1  Chinese Restaurant  0.09
    2         Pizza Place  0.06
    3        Noodle House  0.04
    4   Convenience Store  0.04
    
    
    ----Busby----
               venue  freq
    0    Music Venue  0.25
    1       Gym Pool  0.25
    2            Gym  0.25
    3  Shopping Mall  0.25
    4    Opera House  0.00
    
    
    ----Cabramatta----
                       venue  freq
    0  Vietnamese Restaurant  0.29
    1     Chinese Restaurant  0.15
    2                   Café  0.08
    3           Dessert Shop  0.04
    4   Fast Food Restaurant  0.04
    
    
    ----Caddens----
                      venue  freq
    0           Snack Place  0.25
    1                  Park  0.25
    2         Grocery Store  0.25
    3  Fast Food Restaurant  0.25
    4          Outlet Store  0.00
    
    
    ----Cambridge Gardens----
                venue  freq
    0   Grocery Store  0.15
    1            Café  0.15
    2           Hotel  0.08
    3  Baseball Field  0.08
    4            Park  0.08
    
    
    ----Cambridge Park----
                      venue  freq
    0     Convenience Store   0.2
    1  Fast Food Restaurant   0.2
    2         Grocery Store   0.2
    3      Basketball Court   0.1
    4               Stadium   0.1
    
    
    ----Camden----
                 venue  freq
    0             Café  0.21
    1              Bar  0.14
    2      Pizza Place  0.07
    3      Supermarket  0.07
    4  Thai Restaurant  0.07
    
    
    ----Camden South----
             venue  freq
    0          Pub  0.25
    1          Zoo  0.25
    2        Trail  0.25
    3         Farm  0.25
    4  Opera House  0.00
    
    
    ----Cammeray----
                     venue  freq
    0                 Café  0.19
    1  Japanese Restaurant  0.08
    2      Thai Restaurant  0.06
    3          Coffee Shop  0.05
    4                 Park  0.04
    
    
    ----Campbelltown----
                      venue  freq
    0  Fast Food Restaurant  0.11
    1      Department Store  0.06
    2         Grocery Store  0.06
    3        Sandwich Place  0.06
    4              Platform  0.06
    
    
    ----Camperdown----
                 venue  freq
    0             Café  0.17
    1             Park  0.04
    2  Thai Restaurant  0.04
    3   Ice Cream Shop  0.04
    4      Coffee Shop  0.04
    
    
    ----Canada Bay----
                      venue  freq
    0           Coffee Shop  0.09
    1                  Café  0.07
    2            Sports Bar  0.07
    3                  Park  0.05
    4  Fast Food Restaurant  0.04
    
    
    ----Canley Heights----
                               venue  freq
    0          Vietnamese Restaurant  0.27
    1             Chinese Restaurant  0.10
    2                           Café  0.07
    3                   Dessert Shop  0.06
    4  Vegetarian / Vegan Restaurant  0.05
    
    
    ----Canley Vale----
                       venue  freq
    0  Vietnamese Restaurant  0.30
    1     Chinese Restaurant  0.15
    2                   Café  0.06
    3   Fast Food Restaurant  0.04
    4        Bubble Tea Shop  0.04
    
    
    ----Canterbury----
                      venue  freq
    0                  Café  0.10
    1              Platform  0.07
    2  Fast Food Restaurant  0.05
    3     Korean Restaurant  0.05
    4    Chinese Restaurant  0.05
    
    
    ----Caringbah South----
                   venue  freq
    0               Café  0.14
    1       Liquor Store  0.09
    2  Convenience Store  0.09
    3        Coffee Shop  0.09
    4             Bakery  0.09
    
    
    ----Carlingford----
                      venue  freq
    0    Chinese Restaurant  0.10
    1  Fast Food Restaurant  0.10
    2          Soccer Field  0.06
    3           Pizza Place  0.06
    4           Supermarket  0.06
    
    
    ----Carlton----
                      venue  freq
    0                  Café  0.14
    1                  Park  0.07
    2                   Pub  0.07
    3           Pizza Place  0.07
    4  Fast Food Restaurant  0.05
    
    
    ----Carnes Hill----
                      venue  freq
    0  Fast Food Restaurant  0.18
    1           Supermarket  0.18
    2       Vacation Rental  0.09
    3    Italian Restaurant  0.09
    4                  Café  0.09
    
    
    ----Carramar----
                      venue  freq
    0    Italian Restaurant  0.17
    1         Bowling Alley  0.08
    2  Gym / Fitness Center  0.08
    3                   Gym  0.08
    4     Convenience Store  0.08
    
    
    ----Carss Park----
                      venue  freq
    0                  Park  0.15
    1    Seafood Restaurant  0.11
    2  Fast Food Restaurant  0.11
    3                  Café  0.11
    4           Pizza Place  0.11
    
    
    ----Cartwright----
                      venue  freq
    0  Fast Food Restaurant  0.18
    1         Shopping Mall  0.18
    2              Pharmacy  0.09
    3           Music Venue  0.09
    4        Sandwich Place  0.09
    
    
    ----Castle Cove----
                 venue  freq
    0             Café  0.15
    1             Park  0.10
    2           Bakery  0.08
    3    Grocery Store  0.06
    4  Thai Restaurant  0.06
    
    
    ----Castle Hill----
                      venue  freq
    0  Fast Food Restaurant  0.08
    1                  Café  0.08
    2      Department Store  0.06
    3            Food Court  0.04
    4         Movie Theater  0.04
    
    
    ----Castlecrag----
                     venue  freq
    0                 Café  0.22
    1                 Park  0.12
    2  Japanese Restaurant  0.06
    3        Grocery Store  0.06
    4          Supermarket  0.06
    
    
    ----Casula----
                    venue  freq
    0  Italian Restaurant  0.08
    1         Supermarket  0.08
    2  Seafood Restaurant  0.08
    3        Liquor Store  0.08
    4               Hotel  0.04
    
    
    ----Catherine Field----
                       venue  freq
    0                 Arcade   1.0
    1                    ATM   0.0
    2                 Office   0.0
    3           Outlet Store   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Cattai----
                 venue  freq
    0       Campground   0.5
    1    National Park   0.5
    2      Opera House   0.0
    3  Paintball Field   0.0
    4     Outlet Store   0.0
    
    
    ----Cawdor----
                    venue  freq
    0                 Pub  0.50
    1  Chinese Restaurant  0.25
    2         Pizza Place  0.25
    3         Opera House  0.00
    4        Outlet Store  0.00
    
    
    ----Cecil Hills----
                      venue  freq
    0         Shopping Mall  0.25
    1  Other Great Outdoors  0.25
    2  Fast Food Restaurant  0.25
    3    Italian Restaurant  0.12
    4                  Park  0.12
    
    
    ----Cecil Park----
                       venue  freq
    0       Sculpture Garden   1.0
    1                 Office   0.0
    2           Outlet Store   0.0
    3  Outdoors & Recreation   0.0
    4   Outdoor Supply Store   0.0
    
    
    ----Centennial Park----
                       venue  freq
    0                   Café  0.12
    1                    Bar  0.07
    2     Italian Restaurant  0.05
    3  Australian Restaurant  0.05
    4                    Pub  0.05
    
    
    ----Chatswood West----
               venue  freq
    0           Café  0.23
    1           Park  0.15
    2    Golf Course  0.15
    3    Sports Club  0.08
    4  National Park  0.08
    
    
    ----Cheltenham----
                    venue  freq
    0                Café  0.19
    1                Park  0.14
    2  Athletics & Sports  0.10
    3     Thai Restaurant  0.10
    4              Tunnel  0.05
    
    
    ----Cherrybrook----
              venue  freq
    0          Park  0.20
    1  Liquor Store  0.13
    2          Café  0.13
    3    Sports Bar  0.07
    4        Bakery  0.07
    
    
    ----Chester Hill----
                     venue  freq
    0         Soccer Field  0.18
    1                 Café  0.12
    2        Train Station  0.12
    3  Lebanese Restaurant  0.06
    4     Sushi Restaurant  0.06
    
    
    ----Chifley----
             venue  freq
    0         Café  0.17
    1  Supermarket  0.08
    2       Bakery  0.08
    3  Pizza Place  0.08
    4         Park  0.08
    
    
    ----Chippendale----
                     venue  freq
    0                 Café  0.17
    1                  Bar  0.08
    2                  Pub  0.06
    3      Thai Restaurant  0.05
    4  Japanese Restaurant  0.04
    
    
    ----Chipping Norton----
                    venue  freq
    0                Park  0.15
    1  Italian Restaurant  0.08
    2         Coffee Shop  0.08
    3      Farmers Market  0.08
    4     Thai Restaurant  0.08
    
    
    ----Church Point----
                 venue  freq
    0             Pier  0.17
    1            Beach  0.17
    2             Park  0.17
    3             Café  0.17
    4  Harbor / Marina  0.17
    
    
    ----Claremont Meadows----
                      venue  freq
    0       Automotive Shop  0.17
    1                  Park  0.17
    2           Gas Station  0.17
    3  Fast Food Restaurant  0.17
    4          Soccer Field  0.17
    
    
    ----Clareville----
                       venue  freq
    0        Harbor / Marina   0.4
    1                  Beach   0.2
    2  Australian Restaurant   0.2
    3                   Café   0.2
    4                    ATM   0.0
    
    
    ----Clemton Park----
                       venue  freq
    0                   Café  0.21
    1                    Gym  0.07
    2                   Park  0.07
    3  Portuguese Restaurant  0.04
    4            Supermarket  0.04
    
    
    ----Clontarf----
                   venue  freq
    0               Park  0.18
    1               Café  0.12
    2              Beach  0.09
    3     Scenic Lookout  0.06
    4  Indian Restaurant  0.06
    
    
    ----Clovelly----
             venue  freq
    0         Café  0.23
    1        Beach  0.07
    2         Park  0.04
    3  Pizza Place  0.03
    4    Juice Bar  0.03
    
    
    ----Collaroy----
                       venue  freq
    0                  Beach  0.16
    1                   Café  0.13
    2            Coffee Shop  0.06
    3             Board Shop  0.06
    4  Australian Restaurant  0.06
    
    
    ----Collaroy Plateau----
                       venue  freq
    0                   Café  0.10
    1                    Pub  0.07
    2  Australian Restaurant  0.07
    3             Board Shop  0.07
    4                  Beach  0.07
    
    
    ----Colyton----
                      venue  freq
    0  Fast Food Restaurant  0.21
    1     Convenience Store  0.11
    2    Chinese Restaurant  0.05
    3          Soccer Field  0.05
    4         Shopping Mall  0.05
    
    
    ----Como----
                 venue  freq
    0             Café  0.21
    1  Thai Restaurant  0.16
    2           Bakery  0.11
    3    Grocery Store  0.11
    4              Pub  0.05
    
    
    ----Concord----
                     venue  freq
    0                 Café  0.19
    1                 Park  0.12
    2      Thai Restaurant  0.07
    3   Italian Restaurant  0.07
    4  Japanese Restaurant  0.05
    
    
    ----Concord West----
                      venue  freq
    0                  Café  0.17
    1  Fast Food Restaurant  0.08
    2                  Park  0.06
    3    Dim Sum Restaurant  0.04
    4   Japanese Restaurant  0.04
    
    
    ----Condell Park----
                    venue  freq
    0          Sports Bar  0.08
    1         Pizza Place  0.08
    2      Sandwich Place  0.08
    3  Miscellaneous Shop  0.08
    4       Grocery Store  0.08
    
    
    ----Connells Point----
                     venue  freq
    0                 Park  0.19
    1   Italian Restaurant  0.06
    2    Indian Restaurant  0.06
    3   Falafel Restaurant  0.06
    4  Sporting Goods Shop  0.06
    
    
    ----Cremorne----
                     venue  freq
    0                 Café  0.19
    1  Japanese Restaurant  0.16
    2      Thai Restaurant  0.06
    3                  Pub  0.03
    4               Bakery  0.03
    
    
    ----Cremorne Point----
                     venue  freq
    0  Japanese Restaurant  0.13
    1                 Café  0.12
    2          Zoo Exhibit  0.08
    3                 Park  0.04
    4      Thai Restaurant  0.04
    
    
    ----Cromer----
                     venue  freq
    0                 Café  0.09
    1        Shopping Mall  0.09
    2         Liquor Store  0.09
    3  Japanese Restaurant  0.04
    4             Pharmacy  0.04
    
    
    ----Cronulla----
                    venue  freq
    0                Café  0.20
    1               Beach  0.09
    2                Park  0.09
    3                 Bar  0.06
    4  Mexican Restaurant  0.04
    
    
    ----Crows Nest----
                     venue  freq
    0                 Café  0.20
    1  Japanese Restaurant  0.09
    2                  Bar  0.05
    3      Thai Restaurant  0.05
    4          Coffee Shop  0.04
    
    
    ----Croydon----
                     venue  freq
    0                 Café  0.10
    1   Chinese Restaurant  0.07
    2          Supermarket  0.05
    3             Pharmacy  0.04
    4  Japanese Restaurant  0.04
    
    
    ----Croydon Park----
                   venue  freq
    0  Convenience Store  0.09
    1        Pizza Place  0.09
    2               Park  0.09
    3               Pool  0.05
    4   Malay Restaurant  0.05
    
    
    ----Curl Curl----
           venue  freq
    0       Café  0.21
    1  Surf Spot  0.07
    2       Park  0.07
    3      Beach  0.07
    4     Bakery  0.07
    
    
    ----Currans Hill----
                      venue  freq
    0  Fast Food Restaurant  0.19
    1  Gym / Fitness Center  0.12
    2              Pharmacy  0.12
    3         Grocery Store  0.12
    4         Shopping Mall  0.06
    
    
    ----Daceyville----
                       venue  freq
    0     Chinese Restaurant  0.12
    1  Indonesian Restaurant  0.10
    2                   Café  0.08
    3            Golf Course  0.04
    4        Thai Restaurant  0.04
    
    
    ----Dangar Island----
                    venue  freq
    0  Seafood Restaurant  0.15
    1            Platform  0.15
    2     Harbor / Marina  0.15
    3                 Pub  0.08
    4                 Bay  0.08
    
    
    ----Darling Point----
                       venue  freq
    0                   Café  0.18
    1     Italian Restaurant  0.08
    2                   Park  0.07
    3  Australian Restaurant  0.05
    4    Japanese Restaurant  0.04
    
    
    ----Darlington----
                 venue  freq
    0             Café  0.15
    1              Pub  0.06
    2              Bar  0.06
    3      Coffee Shop  0.06
    4  Thai Restaurant  0.05
    
    
    ----Davidson----
               venue  freq
    0          Trail   0.3
    1       Tea Room   0.1
    2        Brewery   0.1
    3  Women's Store   0.1
    4     Theme Park   0.1
    
    
    ----Dawes Point----
                       venue  freq
    0                   Café  0.11
    1                  Hotel  0.06
    2  Australian Restaurant  0.05
    3         Scenic Lookout  0.05
    4           Cocktail Bar  0.05
    
    
    ----Dee Why----
                    venue  freq
    0                Café  0.10
    1  Italian Restaurant  0.06
    2         Pizza Place  0.04
    3       Shopping Mall  0.04
    4         Coffee Shop  0.04
    
    
    ----Denham Court----
                            venue  freq
    0          Italian Restaurant   0.2
    1  Construction & Landscaping   0.2
    2                        Café   0.2
    3                 Gas Station   0.2
    4            Business Service   0.2
    
    
    ----Denistone----
                     venue  freq
    0   Chinese Restaurant  0.16
    1    Korean Restaurant  0.12
    2  Japanese Restaurant  0.09
    3          Supermarket  0.04
    4                 Café  0.04
    
    
    ----Denistone East----
                   venue  freq
    0               Café  0.11
    1               Park  0.08
    2  Korean Restaurant  0.05
    3        Supermarket  0.05
    4       Liquor Store  0.05
    
    
    ----Denistone West----
                     venue  freq
    0   Chinese Restaurant  0.13
    1    Korean Restaurant  0.10
    2  Japanese Restaurant  0.07
    3                 Café  0.07
    4          Pizza Place  0.04
    
    
    ----Dharruk----
                         venue  freq
    0     Fast Food Restaurant  0.18
    1            Shopping Mall  0.18
    2      Fried Chicken Joint  0.09
    3  Fruit & Vegetable Store  0.09
    4                 Gym Pool  0.09
    
    
    ----Dolans Bay----
                      venue  freq
    0                  Café   0.4
    1                Bakery   0.2
    2       Harbor / Marina   0.2
    3          Home Service   0.2
    4  Pakistani Restaurant   0.0
    
    
    ----Dolls Point----
                    venue  freq
    0                Café  0.24
    1               Beach  0.12
    2         Pizza Place  0.07
    3                Park  0.07
    4  Seafood Restaurant  0.05
    
    
    ----Doonside----
               venue  freq
    0    Zoo Exhibit  0.27
    1  Grocery Store  0.13
    2           Park  0.13
    3  Deli / Bodega  0.07
    4         Bakery  0.07
    
    
    ----Dover Heights----
                 venue  freq
    0             Café  0.16
    1             Park  0.12
    2  Thai Restaurant  0.06
    3    Deli / Bodega  0.06
    4      Coffee Shop  0.04
    
    
    ----Duffys Forest----
             venue  freq
    0  Golf Course  0.25
    1      Stadium  0.25
    2        Trail  0.25
    3         Café  0.25
    4          ATM  0.00
    
    
    ----Dundas Valley----
                    venue  freq
    0                Park  0.12
    1                 Pub  0.12
    2        Dessert Shop  0.12
    3  Chinese Restaurant  0.12
    4        Burger Joint  0.12
    
    
    ----Dural----
                          venue  freq
    0                      Café  0.23
    1                       Pub  0.08
    2        Athletics & Sports  0.08
    3  Mediterranean Restaurant  0.08
    4        Italian Restaurant  0.08
    
    
    ----Eagle Vale----
                venue  freq
    0   Shopping Mall  0.20
    1    Soccer Field  0.13
    2             Pub  0.07
    3             Gym  0.07
    4  Sandwich Place  0.07
    
    
    ----Earlwood----
               venue  freq
    0           Café  0.18
    1           Park  0.14
    2  Train Station  0.11
    3    Supermarket  0.07
    4            Gym  0.07
    
    
    ----East Hills----
                  venue  freq
    0              Park  0.18
    1          Platform  0.12
    2       Pizza Place  0.12
    3    Sandwich Place  0.06
    4  Business Service  0.06
    
    
    ----East Killara----
                   venue  freq
    0               Park  0.38
    1  Food & Drink Shop  0.12
    2               Café  0.12
    3      Grocery Store  0.12
    4      Shopping Mall  0.12
    
    
    ----East Lindfield----
                     venue  freq
    0                 Park  0.25
    1  Lebanese Restaurant  0.06
    2                 Café  0.06
    3    French Restaurant  0.06
    4      Thai Restaurant  0.06
    
    
    ----East Ryde----
                    venue  freq
    0                Park  0.11
    1  Chinese Restaurant  0.07
    2         Bus Station  0.07
    3                 Gym  0.07
    4     Thai Restaurant  0.07
    
    
    ----Eastgardens----
                      venue  freq
    0           Supermarket  0.10
    1           Coffee Shop  0.07
    2  Fast Food Restaurant  0.07
    3           Pizza Place  0.07
    4         Shopping Mall  0.05
    
    
    ----Eastlakes----
               venue  freq
    0    Golf Course  0.12
    1           Café  0.09
    2         Bakery  0.07
    3  Souvlaki Shop  0.07
    4  Grocery Store  0.05
    
    
    ----Eastwood----
                       venue  freq
    0     Chinese Restaurant  0.26
    1    Japanese Restaurant  0.07
    2  Vietnamese Restaurant  0.07
    3      Korean Restaurant  0.07
    4                    Pub  0.05
    
    
    ----Edensor Park----
                      venue  freq
    0  Other Great Outdoors  0.10
    1           Supermarket  0.10
    2                  Café  0.10
    3                  Park  0.10
    4         Shopping Mall  0.05
    
    
    ----Edgecliff----
                       venue  freq
    0                   Café  0.22
    1     Italian Restaurant  0.12
    2                   Park  0.06
    3                    Bar  0.05
    4  Australian Restaurant  0.05
    
    
    ----Edmondson Park----
                      venue  freq
    0         Grocery Store  0.33
    1  Fast Food Restaurant  0.33
    2         Train Station  0.17
    3              Pharmacy  0.17
    4                   ATM  0.00
    
    
    ----Elanora Heights----
               venue  freq
    0            Gym  0.14
    1           Park  0.14
    2           Café  0.14
    3  Grocery Store  0.14
    4  Shopping Mall  0.14
    
    
    ----Elderslie----
                    venue  freq
    0                 Pub  0.33
    1  Italian Restaurant  0.17
    2                Café  0.17
    3      Farmers Market  0.17
    4     Thai Restaurant  0.17
    
    
    ----Elizabeth Bay----
                       venue  freq
    0                   Café  0.17
    1     Italian Restaurant  0.11
    2                   Park  0.07
    3  Australian Restaurant  0.05
    4    Japanese Restaurant  0.05
    
    
    ----Elizabeth Hills----
                       venue  freq
    0          Shopping Mall   0.5
    1   Fast Food Restaurant   0.5
    2                    ATM   0.0
    3            Opera House   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Ellis Lane----
                            venue  freq
    0  Construction & Landscaping   1.0
    1                         ATM   0.0
    2                      Office   0.0
    3                Outlet Store   0.0
    4       Outdoors & Recreation   0.0
    
    
    ----Emu Heights----
                      venue  freq
    0  Fast Food Restaurant  0.33
    1           Pizza Place  0.17
    2     Convenience Store  0.17
    3                  Café  0.17
    4          Skating Rink  0.17
    
    
    ----Emu Plains----
                      venue  freq
    0           Pizza Place   0.1
    1           Supermarket   0.1
    2                  Café   0.1
    3  Fast Food Restaurant   0.1
    4                 River   0.1
    
    
    ----Enfield----
                 venue  freq
    0             Park  0.15
    1             Café  0.09
    2  Thai Restaurant  0.06
    3      Pizza Place  0.06
    4    Grocery Store  0.06
    
    
    ----Engadine----
                      venue  freq
    0                  Café  0.21
    1  Fast Food Restaurant  0.11
    2        Sandwich Place  0.11
    3      Sushi Restaurant  0.05
    4  Gym / Fitness Center  0.05
    
    
    ----Englorie Park----
                      venue  freq
    0                  Café  0.18
    1  Fast Food Restaurant  0.09
    2           Supermarket  0.06
    3         Shopping Mall  0.06
    4          Noodle House  0.03
    
    
    ----Enmore----
                venue  freq
    0            Café  0.16
    1             Bar  0.06
    2             Pub  0.05
    3          Bakery  0.04
    4  Ice Cream Shop  0.04
    
    
    ----Epping----
                       venue  freq
    0      Indian Restaurant  0.09
    1        Thai Restaurant  0.09
    2    Fried Chicken Joint  0.06
    3                   Park  0.06
    4  Vietnamese Restaurant  0.06
    
    
    ----Ermington----
                      venue  freq
    0                  Park  0.15
    1                  Café  0.07
    2  Fast Food Restaurant  0.07
    3    Chinese Restaurant  0.07
    4    Italian Restaurant  0.04
    
    
    ----Erskine Park----
                    venue  freq
    0    Asian Restaurant  0.14
    1  Italian Restaurant  0.14
    2                Café  0.14
    3         Supermarket  0.14
    4       Grocery Store  0.14
    
    
    ----Erskineville----
                    venue  freq
    0                Café  0.20
    1                 Bar  0.07
    2  Italian Restaurant  0.05
    3      Ice Cream Shop  0.04
    4         Coffee Shop  0.04
    
    
    ----Eschol Park----
                      venue  freq
    0         Grocery Store  0.12
    1          Soccer Field  0.12
    2         Shopping Mall  0.12
    3  Fast Food Restaurant  0.06
    4                   Pub  0.06
    
    
    ----Fairfield----
                        venue  freq
    0    Fast Food Restaurant  0.14
    1        Video Game Store  0.05
    2         Thai Restaurant  0.05
    3  Thrift / Vintage Store  0.05
    4          Sandwich Place  0.05
    
    
    ----Fairfield East----
                      venue  freq
    0  Fast Food Restaurant  0.17
    1              Platform  0.09
    2           Pizza Place  0.09
    3     Electronics Store  0.09
    4                   Gym  0.09
    
    
    ----Fairfield Heights----
                      venue  freq
    0           Pizza Place  0.10
    1           Supermarket  0.10
    2      Department Store  0.07
    3  Fast Food Restaurant  0.07
    4                  Café  0.03
    
    
    ----Fairfield West----
                           venue  freq
    0      Vietnamese Restaurant  0.18
    1                       Park  0.05
    2               Dessert Shop  0.05
    3                        Gym  0.05
    4  Middle Eastern Restaurant  0.05
    
    
    ----Fairlight----
                     venue  freq
    0                 Café  0.15
    1                Beach  0.09
    2                  Bar  0.06
    3          Coffee Shop  0.04
    4  Japanese Restaurant  0.04
    
    
    ----Five Dock----
                    venue  freq
    0                Park  0.11
    1                Café  0.09
    2       Grocery Store  0.06
    3                 Gym  0.06
    4  Italian Restaurant  0.06
    
    
    ----Forest Lodge----
                    venue  freq
    0                Café  0.18
    1                Park  0.05
    2     Thai Restaurant  0.05
    3  Italian Restaurant  0.05
    4                 Bar  0.04
    
    
    ----Forestville----
              venue  freq
    0          Park  0.17
    1   Pizza Place  0.17
    2           Gym  0.08
    3  Home Service  0.08
    4    Sports Bar  0.08
    
    
    ----Freemans Reach----
                       venue  freq
    0         Farmers Market   0.5
    1              Pet Store   0.5
    2                    ATM   0.0
    3            Opera House   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Frenchs Forest----
                   venue  freq
    0               Café  0.14
    1      Grocery Store  0.07
    2             Bakery  0.07
    3        Bus Station  0.07
    4  Convenience Store  0.07
    
    
    ----Freshwater----
              venue  freq
    0          Café  0.19
    1          Park  0.16
    2         Beach  0.10
    3  Burger Joint  0.06
    4     Surf Spot  0.06
    
    
    ----Galston----
              venue  freq
    0          Café  0.33
    1           Pub  0.11
    2  Liquor Store  0.11
    3      Gym Pool  0.11
    4          Park  0.11
    
    
    ----Georges Hall----
               venue  freq
    0           Park  0.18
    1        Airport  0.18
    2       Heliport  0.09
    3    Bus Station  0.09
    4  Shopping Mall  0.09
    
    
    ----Girraween----
                      venue  freq
    0  Fast Food Restaurant  0.12
    1                  Café  0.12
    2           Supermarket  0.06
    3              Platform  0.06
    4       Thai Restaurant  0.06
    
    
    ----Gladesville----
                       venue  freq
    0        Thai Restaurant  0.14
    1                    Gym  0.06
    2  Australian Restaurant  0.06
    3            Supermarket  0.06
    4                   Café  0.06
    
    
    ----Glen Alpine----
                      venue  freq
    0                   Pub  0.14
    1    Italian Restaurant  0.14
    2  Gym / Fitness Center  0.14
    3                  Park  0.14
    4         Shopping Mall  0.14
    
    
    ----Glendenning----
                      venue  freq
    0  Fast Food Restaurant  0.33
    1     Convenience Store  0.22
    2                  Park  0.11
    3        Sandwich Place  0.11
    4                   Gym  0.11
    
    
    ----Glenfield----
                      venue  freq
    0  Fast Food Restaurant  0.50
    1   Fried Chicken Joint  0.25
    2              Platform  0.25
    3          Nudist Beach  0.00
    4  Outdoor Supply Store  0.00
    
    
    ----Glenhaven----
                   venue  freq
    0               Café  0.17
    1      Grocery Store  0.08
    2  Other Repair Shop  0.08
    3              Trail  0.08
    4      Shopping Mall  0.08
    
    
    ----Glenmore Park----
                      venue  freq
    0  Fast Food Restaurant  0.50
    1        Sandwich Place  0.17
    2                  Park  0.17
    3        Shop & Service  0.17
    4                   ATM  0.00
    
    
    ----Glenorie----
                 venue  freq
    0           Bakery  0.25
    1      Supermarket  0.25
    2    Shopping Mall  0.25
    3              Bar  0.25
    4  Nature Preserve  0.00
    
    
    ----Glossodia----
                       venue  freq
    0            Pizza Place   1.0
    1           Nudist Beach   0.0
    2  Outdoors & Recreation   0.0
    3   Outdoor Supply Store   0.0
    4      Other Repair Shop   0.0
    
    
    ----Granville----
                      venue  freq
    0   Lebanese Restaurant  0.10
    1  Fast Food Restaurant  0.07
    2              Platform  0.07
    3         Train Station  0.07
    4          Dessert Shop  0.07
    
    
    ----Grasmere----
                      venue  freq
    0                  Café   0.5
    1  Arts & Entertainment   0.5
    2                   ATM   0.0
    3  Pakistani Restaurant   0.0
    4          Outlet Store   0.0
    
    
    ----Grays Point----
                    venue  freq
    0        Liquor Store   0.2
    1               Trail   0.2
    2          Playground   0.2
    3  Mexican Restaurant   0.2
    4        Home Service   0.2
    
    
    ----Green Valley----
                      venue  freq
    0  Fast Food Restaurant  0.22
    1      Asian Restaurant  0.22
    2           Coffee Shop  0.11
    3    Chinese Restaurant  0.11
    4     Convenience Store  0.11
    
    
    ----Greenacre----
                      venue  freq
    0  Fast Food Restaurant  0.21
    1                  Park  0.11
    2    Frozen Yogurt Shop  0.05
    3           Supermarket  0.05
    4             Pet Store  0.05
    
    
    ----Greenfield Park----
                    venue  freq
    0                Park  0.11
    1         Supermarket  0.11
    2       Grocery Store  0.11
    3  Italian Restaurant  0.05
    4  Athletics & Sports  0.05
    
    
    ----Greenhills Beach----
             venue  freq
    0    Surf Spot  0.33
    1  Golf Course  0.11
    2         Park  0.11
    3        Beach  0.11
    4          Gym  0.11
    
    
    ----Greenwich----
                 venue  freq
    0             Café  0.20
    1             Park  0.11
    2      Coffee Shop  0.05
    3       Restaurant  0.05
    4  Thai Restaurant  0.05
    
    
    ----Gregory Hills----
                      venue  freq
    0             Racetrack  0.25
    1                  Park  0.25
    2  Gym / Fitness Center  0.25
    3                Bistro  0.25
    4                   ATM  0.00
    
    
    ----Greystanes----
               venue  freq
    0           Park  0.38
    1            Pub  0.12
    2           Café  0.12
    3    Supermarket  0.12
    4  Shopping Mall  0.12
    
    
    ----Grose Vale----
                       venue  freq
    0                Stables   1.0
    1           Nudist Beach   0.0
    2  Outdoors & Recreation   0.0
    3   Outdoor Supply Store   0.0
    4      Other Repair Shop   0.0
    
    
    ----Guildford----
                      venue  freq
    0  Fast Food Restaurant  0.11
    1           Pizza Place  0.11
    2       Supplement Shop  0.05
    3               Library  0.05
    4    Miscellaneous Shop  0.05
    
    
    ----Guildford West----
                            venue  freq
    0                 Sports Club   0.2
    1           Convenience Store   0.2
    2  Construction & Landscaping   0.2
    3                        Café   0.2
    4                 Bus Station   0.2
    
    
    ----Gymea----
                      venue  freq
    0                  Café  0.12
    1      Department Store  0.07
    2     Convenience Store  0.05
    3  Fast Food Restaurant  0.05
    4        Sandwich Place  0.05
    
    
    ----Gymea Bay----
                    venue  freq
    0        Home Service   0.4
    1   Convenience Store   0.2
    2        Liquor Store   0.2
    3  Mexican Restaurant   0.2
    4                 ATM   0.0
    
    
    ----Haberfield----
                    venue  freq
    0                Café  0.17
    1  Italian Restaurant  0.09
    2         Pizza Place  0.06
    3         Coffee Shop  0.06
    4      Farmers Market  0.03
    
    
    ----Hammondville----
                      venue  freq
    0  Fast Food Restaurant  0.14
    1              Platform  0.14
    2                  Café  0.14
    3           Sports Club  0.07
    4                Bakery  0.07
    
    
    ----Harrington Park----
                  venue  freq
    0        Restaurant  0.12
    1               Gym  0.12
    2              Park  0.12
    3       Supermarket  0.12
    4  Department Store  0.12
    
    
    ----Harris Park----
                     venue  freq
    0                 Café  0.06
    1      Thai Restaurant  0.06
    2    Indian Restaurant  0.05
    3         Burger Joint  0.04
    4  Lebanese Restaurant  0.04
    
    
    ----Haymarket----
                 venue  freq
    0             Café  0.11
    1  Thai Restaurant  0.10
    2      Coffee Shop  0.08
    3   Ice Cream Shop  0.04
    4     Cocktail Bar  0.04
    
    
    ----Heathcote----
                      venue  freq
    0  Fast Food Restaurant  0.25
    1                   Pub  0.08
    2        Sandwich Place  0.08
    3                 Trail  0.08
    4         Train Station  0.08
    
    
    ----Hebersham----
                      venue  freq
    0         Shopping Mall  0.14
    1  Fast Food Restaurant  0.14
    2         Grocery Store  0.09
    3          Liquor Store  0.05
    4     Electronics Store  0.05
    
    
    ----Heckenberg----
                   venue  freq
    0        Music Venue  0.14
    1           Gym Pool  0.14
    2  Convenience Store  0.14
    3                Gym  0.14
    4      Shopping Mall  0.14
    
    
    ----Henley----
                 venue  freq
    0             Café  0.18
    1             Park  0.14
    2  Thai Restaurant  0.10
    3             Pier  0.06
    4              Gym  0.04
    
    
    ----Hillsdale----
                  venue  freq
    0       Supermarket  0.10
    1       Coffee Shop  0.08
    2       Pizza Place  0.06
    3  Department Store  0.06
    4              Café  0.06
    
    
    ----Hinchinbrook----
                    venue  freq
    0    Asian Restaurant   0.2
    1  Italian Restaurant   0.1
    2  Mexican Restaurant   0.1
    3   Convenience Store   0.1
    4         Gas Station   0.1
    
    
    ----Hobartville----
               venue  freq
    0    Supermarket  0.17
    1           Café  0.17
    2     Sports Bar  0.06
    3  Train Station  0.06
    4  Movie Theater  0.06
    
    
    ----Holroyd----
                      venue  freq
    0  Fast Food Restaurant  0.10
    1   Lebanese Restaurant  0.06
    2     Indian Restaurant  0.06
    3           Pizza Place  0.06
    4           Supermarket  0.04
    
    
    ----Holsworthy----
                      venue  freq
    0              Platform  0.12
    1  Fast Food Restaurant  0.12
    2                Bakery  0.12
    3         Shopping Mall  0.12
    4                   Bar  0.06
    
    
    ----Homebush----
                       venue  freq
    0                   Café  0.11
    1  Vietnamese Restaurant  0.07
    2          Grocery Store  0.04
    3      Indian Restaurant  0.04
    4      Korean Restaurant  0.03
    
    
    ----Homebush West----
                       venue  freq
    0  Vietnamese Restaurant  0.13
    1                   Café  0.06
    2           Outlet Store  0.04
    3      Electronics Store  0.04
    4          Track Stadium  0.04
    
    
    ----Horningsea Park----
                 venue  freq
    0             Café  0.15
    1      Supermarket  0.15
    2    Grocery Store  0.08
    3  Vacation Rental  0.08
    4     Home Service  0.08
    
    
    ----Hornsby----
                  venue  freq
    0              Café  0.16
    1               Gym  0.05
    2     Movie Theater  0.04
    3            Bakery  0.04
    4  Asian Restaurant  0.04
    
    
    ----Hornsby Heights----
                      venue  freq
    0                Bakery  0.14
    1     Food & Drink Shop  0.14
    2         Train Station  0.14
    3  Fast Food Restaurant  0.14
    4  Other Great Outdoors  0.14
    
    
    ----Horsley Park----
                    venue  freq
    0  Italian Restaurant  0.29
    1         Golf Course  0.14
    2                Café  0.14
    3         Gas Station  0.14
    4       Grocery Store  0.14
    
    
    ----Hoxton Park----
                      venue  freq
    0  Fast Food Restaurant  0.33
    1           Supermarket  0.22
    2    Italian Restaurant  0.11
    3                   Gym  0.11
    4                  Café  0.11
    
    
    ----Hunters Hill----
                venue  freq
    0            Café  0.27
    1            Park  0.20
    2            Pier  0.07
    3  Breakfast Spot  0.03
    4           River  0.03
    
    
    ----Huntleys Cove----
                       venue  freq
    0                   Café  0.19
    1        Thai Restaurant  0.12
    2                   Park  0.09
    3                   Pier  0.06
    4  Australian Restaurant  0.06
    
    
    ----Huntleys Point----
                venue  freq
    0            Café  0.25
    1            Park  0.21
    2            Pier  0.11
    3   Grocery Store  0.07
    4  Cricket Ground  0.04
    
    
    ----Hurlstone Park----
                      venue  freq
    0                  Café  0.14
    1                  Park  0.09
    2           Pizza Place  0.07
    3  Fast Food Restaurant  0.05
    4         Grocery Store  0.05
    
    
    ----Hurstville Grove----
               venue  freq
    0           Park  0.19
    1           Café  0.11
    2            Pub  0.07
    3         Bakery  0.07
    4  Train Station  0.07
    
    
    ----Illawong----
                    venue  freq
    0  Seafood Restaurant   0.2
    1   Convenience Store   0.2
    2                 Gym   0.2
    3                Café   0.2
    4       Shopping Mall   0.2
    
    
    ----Ingleside----
                       venue  freq
    0            Golf Course  0.25
    1                  Hotel  0.25
    2             Campground  0.25
    3            Flower Shop  0.25
    4  Outdoors & Recreation  0.00
    
    
    ----Jamisontown----
                        venue  freq
    0  Furniture / Home Store  0.11
    1                    Café  0.11
    2    Fast Food Restaurant  0.08
    3     Arts & Crafts Store  0.05
    4       Convenience Store  0.05
    
    
    ----Jannali----
                 venue  freq
    0           Bakery  0.17
    1             Café  0.08
    2  Thai Restaurant  0.08
    3      Pizza Place  0.08
    4    Shopping Mall  0.04
    
    
    ----Jordan Springs----
                  venue  freq
    0              Café  0.25
    1     Shopping Mall  0.25
    2    Baseball Field  0.12
    3  Department Store  0.12
    4               Gym  0.12
    
    
    ----Kangaroo Point----
                venue  freq
    0     Supermarket  0.13
    1  Sandwich Place  0.07
    2             Gym  0.07
    3          Bakery  0.07
    4            Park  0.07
    
    
    ----Kareela----
                     venue  freq
    0               Bakery  0.21
    1      Thai Restaurant  0.11
    2          Pizza Place  0.11
    3  Japanese Restaurant  0.05
    4          Supermarket  0.05
    
    
    ----Kearns----
                    venue  freq
    0            Pharmacy   0.1
    1  Athletics & Sports   0.1
    2                 Pub   0.1
    3      Sandwich Place   0.1
    4           BBQ Joint   0.1
    
    
    ----Kellyville----
                            venue  freq
    0                 Pizza Place   0.2
    1  Construction & Landscaping   0.2
    2        Fast Food Restaurant   0.2
    3          Chinese Restaurant   0.2
    4                Burger Joint   0.2
    
    
    ----Kellyville Ridge----
                     venue  freq
    0          Coffee Shop  0.10
    1  Japanese Restaurant  0.05
    2                 Park  0.05
    3        Grocery Store  0.05
    4          Supermarket  0.05
    
    
    ----Kemps Creek----
                      venue  freq
    0            Sports Bar  0.33
    1      Business Service  0.33
    2           Gas Station  0.33
    3          Nudist Beach  0.00
    4  Outdoor Supply Store  0.00
    
    
    ----Kensington----
                       venue  freq
    0                   Café  0.11
    1     Chinese Restaurant  0.10
    2  Indonesian Restaurant  0.10
    3       Malay Restaurant  0.06
    4        Thai Restaurant  0.04
    
    
    ----Kentlyn----
                       venue  freq
    0                  Trail   0.5
    1           Home Service   0.5
    2                    ATM   0.0
    3        Paintball Field   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Killara----
                   venue  freq
    0               Café  0.18
    1               Park  0.14
    2      Train Station  0.09
    3  Convenience Store  0.09
    4        Pizza Place  0.09
    
    
    ----Kings Langley----
                  venue  freq
    0       Supermarket  0.17
    1              Park  0.08
    2   Thai Restaurant  0.08
    3      Liquor Store  0.08
    4  Dentist's Office  0.08
    
    
    ----Kings Park----
                      venue  freq
    0  Fast Food Restaurant  0.12
    1                   Pub  0.12
    2           Gas Station  0.12
    3     Convenience Store  0.12
    4                  Park  0.06
    
    
    ----Kingsford----
                       venue  freq
    0                   Café  0.10
    1     Chinese Restaurant  0.08
    2        Thai Restaurant  0.06
    3  Indonesian Restaurant  0.06
    4            Bus Station  0.04
    
    
    ----Kingsgrove----
                     venue  freq
    0                 Café  0.22
    1                 Park  0.15
    2                  Pub  0.07
    3  Japanese Restaurant  0.04
    4               Bakery  0.04
    
    
    ----Kingswood----
                      venue  freq
    0  Fast Food Restaurant  0.19
    1           Supermarket  0.12
    2                   Pub  0.06
    3           Pizza Place  0.06
    4            Sports Bar  0.06
    
    
    ----Kirrawee----
                   venue  freq
    0               Café  0.16
    1                Pub  0.06
    2  Convenience Store  0.06
    3    Thai Restaurant  0.04
    4                Gym  0.04
    
    
    ----Kirribilli----
                       venue  freq
    0                   Café  0.14
    1         Scenic Lookout  0.08
    2                   Park  0.07
    3  Australian Restaurant  0.07
    4    Japanese Restaurant  0.05
    
    
    ----Kogarah----
                venue  freq
    0            Café  0.10
    1     Coffee Shop  0.08
    2             Gym  0.05
    3          Bakery  0.05
    4  Sandwich Place  0.05
    
    
    ----Kogarah Bay----
                      venue  freq
    0                  Park  0.13
    1                  Café  0.13
    2           Pizza Place  0.10
    3  Fast Food Restaurant  0.10
    4    Dim Sum Restaurant  0.07
    
    
    ----Kurnell----
                 venue  freq
    0            Beach  0.29
    1  Harbor / Marina  0.14
    2             Park  0.14
    3    Historic Site  0.14
    4             Café  0.14
    
    
    ----Kurraba Point----
                     venue  freq
    0                 Café  0.25
    1  Japanese Restaurant  0.10
    2                 Park  0.06
    3          Coffee Shop  0.05
    4                  Bar  0.04
    
    
    ----Kyeemagh----
                      venue  freq
    0        Airport Lounge  0.09
    1                  Café  0.07
    2       Thai Restaurant  0.05
    3  Fast Food Restaurant  0.05
    4          Dessert Shop  0.05
    
    
    ----Kyle Bay----
                    venue  freq
    0                Park  0.33
    1      Scenic Lookout  0.11
    2  Dim Sum Restaurant  0.11
    3      Farmers Market  0.11
    4                Café  0.11
    
    
    ----Lalor Park----
                      venue  freq
    0         Shopping Mall  0.12
    1     Convenience Store  0.12
    2  Gym / Fitness Center  0.06
    3          Liquor Store  0.06
    4    Athletics & Sports  0.06
    
    
    ----Lane Cove North----
                 venue  freq
    0             Café  0.11
    1      Pizza Place  0.11
    2              Gym  0.08
    3              Pub  0.05
    4  Thai Restaurant  0.05
    
    
    ----Lane Cove West----
                           venue  freq
    0             Sandwich Place  0.12
    1                       Café  0.12
    2                Bus Station  0.06
    3          Fish & Chips Shop  0.06
    4  Middle Eastern Restaurant  0.06
    
    
    ----Lansvale----
                    venue  freq
    0                Park  0.27
    1                Lake  0.18
    2  Seafood Restaurant  0.09
    3  Italian Restaurant  0.09
    4                Café  0.09
    
    
    ----Lavender Bay----
                     venue  freq
    0                 Café  0.27
    1                 Park  0.10
    2          Coffee Shop  0.05
    3                  Bar  0.05
    4  Japanese Restaurant  0.04
    
    
    ----Leichhardt----
                       venue  freq
    0                   Café  0.30
    1                    Pub  0.09
    2     Italian Restaurant  0.08
    3  Portuguese Restaurant  0.04
    4           Liquor Store  0.04
    
    
    ----Leonay----
               venue  freq
    0  Memorial Site  0.14
    1          River  0.14
    2     Sports Bar  0.14
    3            Gym  0.14
    4   Liquor Store  0.14
    
    
    ----Leppington----
               venue  freq
    0  Train Station  0.25
    1    Gas Station  0.25
    2       Platform  0.25
    3  Shopping Mall  0.25
    4            ATM  0.00
    
    
    ----Lewisham----
                       venue  freq
    0                   Café  0.27
    1                    Pub  0.05
    2            Pizza Place  0.04
    3            Coffee Shop  0.04
    4  Portuguese Restaurant  0.04
    
    
    ----Liberty Grove----
                        venue  freq
    0                    Café  0.17
    1    Fast Food Restaurant  0.06
    2             Coffee Shop  0.05
    3      Italian Restaurant  0.05
    4  Furniture / Home Store  0.03
    
    
    ----Lidcombe----
                       venue  freq
    0      Korean Restaurant  0.12
    1                   Park  0.12
    2                   Café  0.09
    3                 Bakery  0.06
    4  Vietnamese Restaurant  0.06
    
    
    ----Lilli Pilli----
                 venue  freq
    0           Bakery  0.17
    1  Harbor / Marina  0.17
    2             Pool  0.17
    3    Boat or Ferry  0.17
    4       Non-Profit  0.17
    
    
    ----Lilyfield----
                    venue  freq
    0                Café  0.24
    1  Italian Restaurant  0.09
    2                Park  0.06
    3                 Pub  0.05
    4       Grocery Store  0.05
    
    
    ----Lindfield----
                   venue  freq
    0               Café  0.17
    1               Park  0.17
    2  French Restaurant  0.11
    3  Convenience Store  0.09
    4        Coffee Shop  0.06
    
    
    ----Linley Point----
                   venue  freq
    0               Café  0.30
    1               Park  0.15
    2         Playground  0.05
    3        Bus Station  0.05
    4  Fish & Chips Shop  0.05
    
    
    ----Liverpool----
                      venue  freq
    0  Fast Food Restaurant  0.19
    1           Coffee Shop  0.07
    2                  Café  0.07
    3   Lebanese Restaurant  0.05
    4          Dessert Shop  0.05
    
    
    ----Loftus----
                    venue  freq
    0  Athletics & Sports  0.29
    1              Museum  0.14
    2    Basketball Court  0.14
    3       Train Station  0.14
    4                 Gym  0.14
    
    
    ----Londonderry----
                       venue  freq
    0              Pet Store   0.5
    1            Pizza Place   0.5
    2                    ATM   0.0
    3                 Office   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Long Point----
                       venue  freq
    0           Soccer Field   0.5
    1            Flower Shop   0.5
    2                    ATM   0.0
    3  Outdoors & Recreation   0.0
    4   Outdoor Supply Store   0.0
    
    
    ----Longueville----
                    venue  freq
    0                Pier  0.17
    1                Park  0.17
    2  Italian Restaurant  0.11
    3                Café  0.11
    4     Harbor / Marina  0.11
    
    
    ----Lovett Bay----
                 venue  freq
    0             Park  0.22
    1  Harbor / Marina  0.11
    2           Hostel  0.11
    3   Scenic Lookout  0.11
    4    Boat or Ferry  0.11
    
    
    ----Lower Portland----
                       venue  freq
    0                   Café   1.0
    1                    ATM   0.0
    2            Opera House   0.0
    3           Outlet Store   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Lugarno----
               venue  freq
    0         Bakery  0.18
    1           Café  0.18
    2  Shopping Mall  0.09
    3           Park  0.09
    4            Gym  0.09
    
    
    ----Lurnea----
                               venue  freq
    0           Fast Food Restaurant  0.18
    1                    Pizza Place  0.14
    2                           Park  0.09
    3  Paper / Office Supplies Store  0.05
    4                 Sandwich Place  0.05
    
    
    ----Macquarie Fields----
                      venue  freq
    0           Supermarket  0.12
    1  Fast Food Restaurant  0.12
    2                   Gym  0.06
    3                  Park  0.06
    4                Office  0.06
    
    
    ----Macquarie Links----
                   venue  freq
    0  Fish & Chips Shop   0.2
    1        Golf Course   0.2
    2      Train Station   0.2
    3           Platform   0.2
    4             Office   0.2
    
    
    ----Macquarie Park----
                venue  freq
    0            Café  0.23
    1  Clothing Store  0.04
    2             Gym  0.04
    3     Supermarket  0.04
    4            Park  0.03
    
    
    ----Malabar----
                     venue  freq
    0                 Café  0.22
    1          Golf Course  0.11
    2  Fried Chicken Joint  0.11
    3                 Park  0.11
    4                Beach  0.11
    
    
    ----Manly----
                    venue  freq
    0                Café  0.16
    1               Beach  0.11
    2                 Bar  0.09
    3      Ice Cream Shop  0.05
    4  Mexican Restaurant  0.04
    
    
    ----Manly Vale----
                     venue  freq
    0                 Café  0.17
    1                 Park  0.13
    2  Japanese Restaurant  0.07
    3         Liquor Store  0.07
    4        Shopping Mall  0.07
    
    
    ----Marayong----
                      venue  freq
    0  Fast Food Restaurant  0.12
    1     Convenience Store  0.12
    2           Supermarket  0.08
    3           Coffee Shop  0.08
    4                  Lake  0.04
    
    
    ----Maroota----
                       venue  freq
    0              Pet Store   1.0
    1                    ATM   0.0
    2                 Office   0.0
    3  Outdoors & Recreation   0.0
    4   Outdoor Supply Store   0.0
    
    
    ----Marsden Park----
                            venue  freq
    0  Construction & Landscaping  0.33
    1                   Pet Store  0.33
    2                Home Service  0.33
    3                         ATM  0.00
    4                 Opera House  0.00
    
    
    ----Marsfield----
                venue  freq
    0            Café  0.20
    1            Park  0.12
    2     Pizza Place  0.08
    3    Concert Hall  0.04
    4  Breakfast Spot  0.04
    
    
    ----Matraville----
             venue  freq
    0  Supermarket  0.08
    1         Park  0.08
    2         Café  0.08
    3  Bus Station  0.08
    4  Pizza Place  0.08
    
    
    ----Mays Hill----
                      venue  freq
    0    Chinese Restaurant  0.08
    1           Coffee Shop  0.05
    2  Fast Food Restaurant  0.05
    3                  Café  0.05
    4      Department Store  0.05
    
    
    ----McGraths Hill----
                               venue  freq
    0           Fast Food Restaurant  0.22
    1                 Sandwich Place  0.11
    2  Paper / Office Supplies Store  0.11
    3                    Supermarket  0.11
    4               Department Store  0.11
    
    
    ----Meadowbank----
                 venue  freq
    0             Café  0.14
    1      Pizza Place  0.07
    2  Thai Restaurant  0.07
    3      Supermarket  0.07
    4    Grocery Store  0.05
    
    
    ----Melrose Park----
                      venue  freq
    0                  Café  0.16
    1                  Park  0.10
    2  Fast Food Restaurant  0.06
    3         Shopping Mall  0.06
    4                  Pier  0.06
    
    
    ----Menai----
                       venue  freq
    0   Fast Food Restaurant  0.14
    1  Australian Restaurant  0.07
    2       Sushi Restaurant  0.07
    3              Juice Bar  0.07
    4                    Gym  0.07
    
    
    ----Menangle Park----
                       venue  freq
    0            Sports Club   0.2
    1  Australian Restaurant   0.2
    2          Train Station   0.2
    3              Gift Shop   0.2
    4              Racetrack   0.2
    
    
    ----Merrylands----
                      venue  freq
    0  Fast Food Restaurant  0.17
    1           Pizza Place  0.10
    2           Supermarket  0.07
    3                  Park  0.07
    4                  Café  0.03
    
    
    ----Merrylands West----
               venue  freq
    0            Pub  0.17
    1           Park  0.17
    2            Gym  0.08
    3  Train Station  0.08
    4    Supermarket  0.08
    
    
    ----Middle Cove----
                    venue  freq
    0                Café  0.19
    1                Park  0.14
    2     Thai Restaurant  0.08
    3              Bakery  0.08
    4  Italian Restaurant  0.06
    
    
    ----Middle Dural----
                   venue  freq
    0  Indian Restaurant   0.6
    1      Women's Store   0.2
    2        Flower Shop   0.2
    3                ATM   0.0
    4             Office   0.0
    
    
    ----Middleton Grange----
                            venue  freq
    0        Fast Food Restaurant  0.29
    1  Construction & Landscaping  0.14
    2          Mexican Restaurant  0.14
    3                         Gym  0.14
    4                 Gas Station  0.14
    
    
    ----Miller----
               venue  freq
    0            Gym  0.18
    1            Bar  0.09
    2    Social Club  0.09
    3       Gym Pool  0.09
    4  Shopping Mall  0.09
    
    
    ----Millers Point----
              venue  freq
    0          Café  0.12
    1   Coffee Shop  0.07
    2         Hotel  0.06
    3     Speakeasy  0.05
    4  Cocktail Bar  0.04
    
    
    ----Milperra----
                      venue  freq
    0  Fast Food Restaurant  0.36
    1           Golf Course  0.18
    2         Deli / Bodega  0.18
    3         Garden Center  0.09
    4           Supermarket  0.09
    
    
    ----Milsons Point----
                       venue  freq
    0                   Café  0.15
    1                   Park  0.10
    2  Australian Restaurant  0.06
    3                Theater  0.05
    4         Scenic Lookout  0.05
    
    
    ----Minchinbury----
                         venue  freq
    0        Convenience Store  0.17
    1              Sports Club  0.08
    2  Fruit & Vegetable Store  0.08
    3   Thrift / Vintage Store  0.08
    4                   Garden  0.08
    
    
    ----Minto----
                      venue  freq
    0  Fast Food Restaurant  0.27
    1      Basketball Court  0.09
    2        Sandwich Place  0.09
    3      Department Store  0.09
    4     Convenience Store  0.09
    
    
    ----Minto Heights----
                      venue  freq
    0      Business Service   0.5
    1          Home Service   0.5
    2                   ATM   0.0
    3  Pakistani Restaurant   0.0
    4          Outlet Store   0.0
    
    
    ----Miranda----
                      venue  freq
    0      Department Store  0.09
    1  Fast Food Restaurant  0.09
    2     Electronics Store  0.07
    3                  Café  0.07
    4             Juice Bar  0.04
    
    
    ----Mona Vale----
                        venue  freq
    0                    Café  0.27
    1                    Park  0.12
    2             Supermarket  0.09
    3     Japanese Restaurant  0.03
    4  Thrift / Vintage Store  0.03
    
    
    ----Monterey----
                  venue  freq
    0              Café  0.15
    1             Beach  0.12
    2       Pizza Place  0.06
    3  Greek Restaurant  0.06
    4              Park  0.06
    
    
    ----Moorebank----
                      venue  freq
    0  Fast Food Restaurant  0.25
    1           Pizza Place  0.17
    2                   Pub  0.08
    3     Convenience Store  0.08
    4    Chinese Restaurant  0.08
    
    
    ----Mortdale----
                    venue  freq
    0                Café  0.14
    1         Supermarket  0.06
    2                 Gym  0.06
    3  Italian Restaurant  0.06
    4                 Pub  0.06
    
    
    ----Mortlake----
                      venue  freq
    0                  Café  0.17
    1                  Park  0.11
    2           Golf Course  0.11
    3    Italian Restaurant  0.06
    4  Gym / Fitness Center  0.06
    
    
    ----Mosman----
                     venue  freq
    0                 Café  0.29
    1  Japanese Restaurant  0.07
    2               Bakery  0.06
    3                Beach  0.04
    4                  Pub  0.04
    
    
    ----Mount Colah----
                      venue  freq
    0                Bakery  0.14
    1          Liquor Store  0.14
    2         Train Station  0.14
    3  Other Great Outdoors  0.14
    4  Fast Food Restaurant  0.14
    
    
    ----Mount Druitt----
                      venue  freq
    0  Fast Food Restaurant  0.18
    1                   Gym  0.07
    2     Convenience Store  0.04
    3           Coffee Shop  0.04
    4              Platform  0.04
    
    
    ----Mount Kuring-Gai----
                venue  freq
    0  Scenic Lookout  0.25
    1     Bus Station  0.25
    2         Brewery  0.25
    3   Train Station  0.25
    4    Outlet Store  0.00
    
    
    ----Mount Pritchard----
                      venue  freq
    0     Convenience Store  0.22
    1            Food Truck  0.11
    2  Gym / Fitness Center  0.11
    3         Deli / Bodega  0.11
    4                  Park  0.11
    
    
    ----Mulgoa----
                      venue  freq
    0            Restaurant  0.17
    1  Gym / Fitness Center  0.17
    2                  Café  0.17
    3         Grocery Store  0.17
    4     Indian Restaurant  0.17
    
    
    ----Narellan Vale----
                      venue  freq
    0  Fast Food Restaurant  0.24
    1           Coffee Shop  0.10
    2                  Café  0.10
    3         Shopping Mall  0.10
    4           Pizza Place  0.05
    
    
    ----Naremburn----
                     venue  freq
    0                 Café  0.21
    1  Japanese Restaurant  0.07
    2      Thai Restaurant  0.06
    3                  Bar  0.04
    4          Coffee Shop  0.04
    
    
    ----Narrabeen----
                       venue  freq
    0                   Café  0.15
    1            Pizza Place  0.11
    2  Australian Restaurant  0.07
    3          Grocery Store  0.07
    4                  Beach  0.07
    
    
    ----Narraweena----
                      venue  freq
    0           Music Venue  0.25
    1           Rugby Pitch  0.25
    2         Shopping Mall  0.25
    3  Fast Food Restaurant  0.25
    4       Paintball Field  0.00
    
    
    ----Narwee----
                      venue  freq
    0                  Café  0.10
    1    Chinese Restaurant  0.10
    2                   Pub  0.06
    3           Pizza Place  0.04
    4  Fast Food Restaurant  0.04
    
    
    ----Neutral Bay----
                     venue  freq
    0                 Café  0.26
    1  Japanese Restaurant  0.14
    2                 Park  0.07
    3                  Bar  0.04
    4          Coffee Shop  0.03
    
    
    ----Newington----
                      venue  freq
    0                  Café  0.12
    1   Sporting Goods Shop  0.07
    2  Fast Food Restaurant  0.07
    3        Sandwich Place  0.05
    4   Japanese Restaurant  0.03
    
    
    ----Newport----
                 venue  freq
    0             Café  0.24
    1  Harbor / Marina  0.14
    2            Beach  0.10
    3      Pizza Place  0.10
    4      Supermarket  0.05
    
    
    ----Newtown----
                venue  freq
    0            Café  0.19
    1             Pub  0.05
    2             Bar  0.04
    3     Coffee Shop  0.04
    4  Ice Cream Shop  0.04
    
    
    ----Normanhurst----
                      venue  freq
    0           Pizza Place  0.12
    1  Gym / Fitness Center  0.12
    2                  Café  0.12
    3           Coffee Shop  0.06
    4  Fast Food Restaurant  0.06
    
    
    ----North Balgowlah----
                   venue  freq
    0               Café  0.25
    1        Golf Course  0.17
    2             Bakery  0.08
    3  Indian Restaurant  0.08
    4    Thai Restaurant  0.08
    
    
    ----North Curl Curl----
                    venue  freq
    0                Café  0.18
    1               Beach  0.08
    2  Italian Restaurant  0.08
    3        Burger Joint  0.05
    4         Coffee Shop  0.05
    
    
    ----North Epping----
                    venue  freq
    0                Park  0.17
    1  Italian Restaurant  0.11
    2  Athletics & Sports  0.11
    3      Baseball Field  0.06
    4        Soccer Field  0.06
    
    
    ----North Manly----
                      venue  freq
    0           Supermarket  0.09
    1                Bakery  0.07
    2      Department Store  0.07
    3  Fast Food Restaurant  0.07
    4                  Park  0.04
    
    
    ----North Narrabeen----
             venue  freq
    0         Café  0.12
    1         Park  0.09
    2         Lake  0.09
    3  Pizza Place  0.09
    4  Supermarket  0.06
    
    
    ----North Parramatta----
                       venue  freq
    0        Thai Restaurant  0.09
    1                   Café  0.06
    2                   Park  0.06
    3            Coffee Shop  0.06
    4  Australian Restaurant  0.04
    
    
    ----North Richmond----
                    venue  freq
    0                Café  0.22
    1          Sports Bar  0.11
    2      Sandwich Place  0.11
    3                Park  0.11
    4  Chinese Restaurant  0.11
    
    
    ----North Rocks----
                      venue  freq
    0           Golf Course  0.11
    1    Italian Restaurant  0.05
    2           Bus Station  0.05
    3  Gym / Fitness Center  0.05
    4       Thai Restaurant  0.05
    
    
    ----North Ryde----
             venue  freq
    0         Café  0.29
    1         Park  0.12
    2  Coffee Shop  0.06
    3        Hotel  0.03
    4  Sports Club  0.03
    
    
    ----North St Marys----
                      venue  freq
    0           Sports Club  0.11
    1      Football Stadium  0.11
    2      Indoor Play Area  0.11
    3     Convenience Store  0.11
    4  Fast Food Restaurant  0.11
    
    
    ----North Strathfield----
                      venue  freq
    0                  Café  0.17
    1     Korean Restaurant  0.04
    2        Clothing Store  0.04
    3  Fast Food Restaurant  0.04
    4    Chinese Restaurant  0.03
    
    
    ----North Sydney----
                     venue  freq
    0                 Café  0.23
    1                 Park  0.10
    2      Thai Restaurant  0.06
    3                  Bar  0.05
    4  Japanese Restaurant  0.05
    
    
    ----North Turramurra----
                    venue  freq
    0  Italian Restaurant  0.17
    1                Park  0.08
    2       Historic Site  0.08
    3   Food & Drink Shop  0.08
    4  Chinese Restaurant  0.08
    
    
    ----North Wahroonga----
              venue  freq
    0  Soccer Field  0.50
    1          Park  0.25
    2   Rugby Pitch  0.25
    3           ATM  0.00
    4   Opera House  0.00
    
    
    ----Northbridge----
                     venue  freq
    0                 Café  0.18
    1                 Park  0.13
    2      Harbor / Marina  0.04
    3      Thai Restaurant  0.04
    4  Japanese Restaurant  0.04
    
    
    ----Northmead----
                      venue  freq
    0           Coffee Shop  0.11
    1                  Café  0.07
    2  Fast Food Restaurant  0.07
    3                  Park  0.07
    4             Pet Store  0.07
    
    
    ----Northwood----
                    venue  freq
    0                Café  0.29
    1     Harbor / Marina  0.10
    2                Park  0.10
    3          Restaurant  0.05
    4  Seafood Restaurant  0.05
    
    
    ----Oakville----
                       venue  freq
    0            Auto Garage   1.0
    1                    ATM   0.0
    2            Opera House   0.0
    3           Outlet Store   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Oatlands----
               venue  freq
    0  Train Station   0.2
    1           Café   0.2
    2   Music School   0.1
    3    Golf Course   0.1
    4  Shopping Mall   0.1
    
    
    ----Old Guildford----
                      venue  freq
    0              Platform  0.16
    1     Electronics Store  0.11
    2  Fast Food Restaurant  0.11
    3           Pizza Place  0.11
    4         Train Station  0.05
    
    
    ----Oran Park----
                       venue  freq
    0          Shopping Mall   0.5
    1                 Lawyer   0.5
    2                    ATM   0.0
    3            Opera House   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Orchard Hills----
                    venue  freq
    0  Italian Restaurant   0.5
    1          Playground   0.5
    2                 ATM   0.0
    3              Office   0.0
    4        Outlet Store   0.0
    
    
    ----Oxford Falls----
                 venue  freq
    0             Park  0.50
    1            Trail  0.25
    2             Café  0.25
    3              ATM  0.00
    4  Paintball Field  0.00
    
    
    ----Oyster Bay----
                 venue  freq
    0             Café  0.19
    1  Thai Restaurant  0.11
    2           Bakery  0.11
    3    Grocery Store  0.07
    4      Pizza Place  0.07
    
    
    ----Paddington----
                       venue  freq
    0                   Café  0.17
    1     Italian Restaurant  0.08
    2    Japanese Restaurant  0.06
    3  Australian Restaurant  0.05
    4                 Bakery  0.05
    
    
    ----Padstow----
                    venue  freq
    0                Café  0.20
    1  Seafood Restaurant  0.12
    2            Platform  0.08
    3         Pizza Place  0.08
    4  Chinese Restaurant  0.04
    
    
    ----Padstow Heights----
               venue  freq
    0           Café  0.29
    1         Bakery  0.14
    2  Grocery Store  0.14
    3           Park  0.14
    4          River  0.14
    
    
    ----Palm Beach----
                       venue  freq
    0                  Beach  0.19
    1                   Café  0.12
    2                   Park  0.12
    3  Australian Restaurant  0.12
    4          Boat or Ferry  0.06
    
    
    ----Panania----
                    venue  freq
    0                Café  0.09
    1        Burger Joint  0.06
    2  Seafood Restaurant  0.06
    3            Platform  0.06
    4  Chinese Restaurant  0.06
    
    
    ----Parramatta----
                  venue  freq
    0   Thai Restaurant  0.07
    1              Café  0.06
    2       Coffee Shop  0.05
    3  Asian Restaurant  0.04
    4      Burger Joint  0.04
    
    
    ----Peakhurst----
                venue  freq
    0            Café  0.18
    1   Grocery Store  0.14
    2     Pizza Place  0.09
    3  Sandwich Place  0.05
    4         Exhibit  0.05
    
    
    ----Peakhurst Heights----
               venue  freq
    0           Park  0.33
    1         Bakery  0.11
    2  Deli / Bodega  0.11
    3           Café  0.11
    4  Grocery Store  0.11
    
    
    ----Pemulwuy----
                           venue  freq
    0             Sandwich Place  0.08
    1                Supermarket  0.08
    2                        Gym  0.08
    3  Middle Eastern Restaurant  0.08
    4            Thai Restaurant  0.08
    
    
    ----Pendle Hill----
                           venue  freq
    0       Fast Food Restaurant  0.14
    1                       Café  0.14
    2                Pizza Place  0.07
    3            Thai Restaurant  0.07
    4  Middle Eastern Restaurant  0.04
    
    
    ----Pennant Hills----
                      venue  freq
    0                  Café  0.19
    1    Chinese Restaurant  0.08
    2       Thai Restaurant  0.08
    3     Indian Restaurant  0.06
    4  Fast Food Restaurant  0.04
    
    
    ----Penrith----
                      venue  freq
    0                  Café  0.10
    1  Fast Food Restaurant  0.06
    2       Thai Restaurant  0.04
    3        Sandwich Place  0.04
    4      Department Store  0.04
    
    
    ----Penshurst----
                    venue  freq
    0                Café  0.16
    1  Chinese Restaurant  0.09
    2  Italian Restaurant  0.07
    3       Grocery Store  0.04
    4         Pizza Place  0.04
    
    
    ----Petersham----
                    venue  freq
    0                Café  0.30
    1                 Pub  0.07
    2  Italian Restaurant  0.05
    3         Supermarket  0.03
    4         Coffee Shop  0.03
    
    
    ----Picnic Point----
                      venue  freq
    0                  Park  0.25
    1           Pizza Place  0.25
    2         National Park  0.12
    3    Chinese Restaurant  0.12
    4  Fast Food Restaurant  0.12
    
    
    ----Pitt Town----
                       venue  freq
    0                    Pub  0.33
    1                   Café  0.33
    2          Grocery Store  0.33
    3            Opera House  0.00
    4  Outdoors & Recreation  0.00
    
    
    ----Pitt Town Bottoms----
                            venue  freq
    0                 Pizza Place  0.25
    1                        Café  0.25
    2  Construction & Landscaping  0.25
    3               Grocery Store  0.25
    4                 Opera House  0.00
    
    
    ----Pleasure Point----
               venue  freq
    0       Platform  0.33
    1            Pub  0.17
    2           Lake  0.17
    3           Park  0.17
    4  Train Station  0.17
    
    
    ----Port Hacking----
                 venue  freq
    0  Harbor / Marina  0.14
    1           Bakery  0.14
    2             Pool  0.14
    3            Trail  0.14
    4    Boat or Ferry  0.14
    
    
    ----Potts Point----
                       venue  freq
    0                   Café  0.19
    1     Italian Restaurant  0.08
    2                   Park  0.07
    3  Australian Restaurant  0.04
    4                Theater  0.03
    
    
    ----Prairiewood----
                      venue  freq
    0                  Café  0.12
    1  Fast Food Restaurant  0.10
    2                   Gym  0.05
    3       Thai Restaurant  0.05
    4     Convenience Store  0.05
    
    
    ----Prestons----
               venue  freq
    0          Hotel  0.14
    1       Pharmacy  0.14
    2            Gym  0.14
    3           Park  0.14
    4  Grocery Store  0.14
    
    
    ----Prospect----
                        venue  freq
    0          Sandwich Place  0.12
    1  Furniture / Home Store  0.12
    2       Convenience Store  0.12
    3             Supermarket  0.12
    4              Water Park  0.12
    
    
    ----Punchbowl----
                               venue  freq
    0            Lebanese Restaurant  0.13
    1  Paper / Office Supplies Store  0.13
    2                           Park  0.13
    3                     Skate Park  0.07
    4                            Bar  0.07
    
    
    ----Putney----
                    venue  freq
    0                Café  0.16
    1          Playground  0.08
    2                Park  0.08
    3                 Gym  0.08
    4  Italian Restaurant  0.04
    
    
    ----Pymble----
                 venue  freq
    0             Café  0.12
    1      Pizza Place  0.12
    2              Gym  0.08
    3  Thai Restaurant  0.08
    4           Bakery  0.08
    
    
    ----Pyrmont----
                 venue  freq
    0             Café  0.08
    1  Thai Restaurant  0.06
    2      Coffee Shop  0.06
    3        Bookstore  0.05
    4            Hotel  0.05
    
    
    ----Quakers Hill----
                       venue  freq
    0                   Café  0.10
    1            Pizza Place  0.10
    2       Department Store  0.05
    3  Australian Restaurant  0.05
    4            Gas Station  0.05
    
    
    ----Queens Park----
                     venue  freq
    0                 Café  0.12
    1          Pizza Place  0.06
    2                  Pub  0.06
    3  Japanese Restaurant  0.04
    4                  Gym  0.04
    
    
    ----Queenscliff----
                       venue  freq
    0                   Café  0.15
    1                   Park  0.13
    2                  Beach  0.09
    3              Surf Spot  0.04
    4  Australian Restaurant  0.04
    
    
    ----Raby----
                venue  freq
    0  Sandwich Place  0.12
    1       BBQ Joint  0.12
    2   Shopping Mall  0.12
    3  Discount Store  0.12
    4   Grocery Store  0.12
    
    
    ----Ramsgate----
                  venue  freq
    0              Café  0.19
    1              Park  0.09
    2             Beach  0.07
    3  Greek Restaurant  0.07
    4       Pizza Place  0.07
    
    
    ----Ramsgate Beach----
                    venue  freq
    0                Park  0.14
    1               Beach  0.10
    2                Café  0.10
    3         Pizza Place  0.07
    4  Seafood Restaurant  0.07
    
    
    ----Randwick----
                 venue  freq
    0             Café  0.18
    1  Thai Restaurant  0.09
    2              Pub  0.04
    3      Coffee Shop  0.04
    4      Pizza Place  0.04
    
    
    ----Redfern----
                     venue  freq
    0                 Café  0.16
    1  Japanese Restaurant  0.05
    2                  Bar  0.04
    3         Cocktail Bar  0.04
    4          Coffee Shop  0.04
    
    
    ----Regentville----
                        venue  freq
    0  Furniture / Home Store  0.17
    1                    Café  0.13
    2                   River  0.09
    3     Arts & Crafts Store  0.09
    4                    Park  0.04
    
    
    ----Revesby Heights----
                      venue  freq
    0         National Park   0.2
    1     Convenience Store   0.2
    2                  Café   0.2
    3    Chinese Restaurant   0.2
    4  Fast Food Restaurant   0.2
    
    
    ----Rhodes----
                      venue  freq
    0                  Café  0.09
    1                  Park  0.09
    2           Coffee Shop  0.06
    3  Fast Food Restaurant  0.05
    4           Supermarket  0.05
    
    
    ----Richmond----
                venue  freq
    0     Supermarket   0.3
    1      Sports Bar   0.1
    2             Pub   0.1
    3  Sandwich Place   0.1
    4            Café   0.1
    
    
    ----Riverstone----
                      venue  freq
    0        Sandwich Place  0.09
    1           Pizza Place  0.09
    2                   Bar  0.09
    3  Fast Food Restaurant  0.09
    4         Train Station  0.09
    
    
    ----Riverview----
                    venue  freq
    0                Café  0.14
    1         Pizza Place  0.10
    2  Chinese Restaurant  0.05
    3              Bakery  0.05
    4                 Gym  0.05
    
    
    ----Riverwood----
                       venue  freq
    0          Grocery Store  0.09
    1            Pizza Place  0.09
    2  Vietnamese Restaurant  0.06
    3     Italian Restaurant  0.06
    4         Sandwich Place  0.06
    
    
    ----Rockdale----
                   venue  freq
    0               Café  0.07
    1        Coffee Shop  0.07
    2                Gym  0.05
    3                Pub  0.05
    4  Convenience Store  0.05
    
    
    ----Rodd Point----
                    venue  freq
    0                Café  0.13
    1                Park  0.09
    2               Trail  0.07
    3  Italian Restaurant  0.07
    4         Coffee Shop  0.04
    
    
    ----Ropes Crossing----
                  venue  freq
    0     Grocery Store   0.2
    1          Pharmacy   0.1
    2  Asian Restaurant   0.1
    3     Train Station   0.1
    4    Shop & Service   0.1
    
    
    ----Rosehill----
                     venue  freq
    0    Indian Restaurant  0.16
    1                 Café  0.12
    2       Sandwich Place  0.06
    3  Lebanese Restaurant  0.06
    4                Hotel  0.06
    
    
    ----Roselands----
                      venue  freq
    0  Fast Food Restaurant  0.11
    1                  Park  0.08
    2                   Pub  0.06
    3           Pizza Place  0.06
    4                  Café  0.06
    
    
    ----Roseville----
                    venue  freq
    0                Café  0.12
    1     Thai Restaurant  0.07
    2  Chinese Restaurant  0.06
    3              Bakery  0.05
    4      Ice Cream Shop  0.05
    
    
    ----Roseville Chase----
               venue  freq
    0           Café  0.21
    1           Park  0.11
    2  Grocery Store  0.05
    3    Golf Course  0.05
    4      Pet Store  0.05
    
    
    ----Rossmore----
                            venue  freq
    0                  Campground  0.33
    1  Construction & Landscaping  0.33
    2            Business Service  0.33
    3                         ATM  0.00
    4             Organic Grocery  0.00
    
    
    ----Rouse Hill----
                      venue  freq
    0  Fast Food Restaurant  0.15
    1           Supermarket  0.08
    2      Department Store  0.05
    3           Coffee Shop  0.05
    4      Sushi Restaurant  0.03
    
    
    ----Rozelle----
             venue  freq
    0         Café  0.21
    1          Pub  0.07
    2         Park  0.06
    3  Pizza Place  0.05
    4          Bar  0.05
    
    
    ----Ruse----
                   venue  freq
    0      Grocery Store  0.33
    1             Bakery  0.17
    2  Convenience Store  0.17
    3             Garden  0.17
    4      Shopping Mall  0.17
    
    
    ----Rushcutters Bay----
                       venue  freq
    0                   Café  0.20
    1     Italian Restaurant  0.11
    2                   Park  0.06
    3  Australian Restaurant  0.05
    4    Japanese Restaurant  0.05
    
    
    ----Russell Lea----
                 venue  freq
    0             Café  0.15
    1             Park  0.10
    2            Trail  0.08
    3  Harbor / Marina  0.06
    4           Bakery  0.03
    
    
    ----Rydalmere----
                      venue  freq
    0          Liquor Store  0.09
    1                  Café  0.09
    2  Fast Food Restaurant  0.09
    3        Sandwich Place  0.09
    4                  Park  0.09
    
    
    ----Ryde----
               venue  freq
    0           Café  0.15
    1           Park  0.07
    2    Pizza Place  0.05
    3    Supermarket  0.05
    4  Grocery Store  0.04
    
    
    ----Sackville----
                  venue  freq
    0              Farm   0.5
    1  Business Service   0.5
    2               ATM   0.0
    3      Optical Shop   0.0
    4      Outlet Store   0.0
    
    
    ----Sackville North----
                       venue  freq
    0                   Park   0.5
    1       Business Service   0.5
    2                    ATM   0.0
    3            Opera House   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Sadleir----
                      venue  freq
    0  Fast Food Restaurant   0.2
    1             Pet Store   0.1
    2           Pizza Place   0.1
    3           Music Venue   0.1
    4                   Gym   0.1
    
    
    ----Sandringham----
                      venue  freq
    0                  Café  0.31
    1                 Beach  0.10
    2           Pizza Place  0.07
    3  Fast Food Restaurant  0.07
    4                  Park  0.07
    
    
    ----Sandy Point----
                       venue  freq
    0                   Park   0.6
    1             Sports Bar   0.2
    2           Burger Joint   0.2
    3                 Office   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Schofields----
               venue  freq
    0           Café   0.4
    1  Train Station   0.2
    2    Supermarket   0.2
    3   Soccer Field   0.2
    4            ATM   0.0
    
    
    ----Scotland Island----
                 venue  freq
    0  Harbor / Marina  0.25
    1             Pier  0.25
    2    Boat or Ferry  0.12
    3            Beach  0.12
    4             Café  0.12
    
    
    ----Seaforth----
                 venue  freq
    0             Café  0.21
    1  Harbor / Marina  0.21
    2             Park  0.11
    3      Coffee Shop  0.05
    4    Boat or Ferry  0.05
    
    
    ----Sefton----
                     venue  freq
    0        Train Station  0.17
    1  Lebanese Restaurant  0.08
    2          Supermarket  0.08
    3                  Gym  0.08
    4      Thai Restaurant  0.08
    
    
    ----Seven Hills----
                      venue  freq
    0  Fast Food Restaurant  0.17
    1     Convenience Store  0.10
    2         Shopping Mall  0.07
    3                  Pool  0.03
    4                   Gym  0.03
    
    
    ----Shalvey----
                       venue  freq
    0         Baseball Field   0.5
    1            Gas Station   0.5
    2                    ATM   0.0
    3                 Office   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Shanes Park----
                  venue  freq
    0       Wings Joint  0.25
    1              Café  0.25
    2  Business Service  0.25
    3       Golf Course  0.25
    4               ATM  0.00
    
    
    ----Silverwater----
                     venue  freq
    0                 Café  0.15
    1       Sandwich Place  0.08
    2  Sporting Goods Shop  0.08
    3    Electronics Store  0.08
    4                 Park  0.08
    
    
    ----Smithfield----
                      venue  freq
    0  Fast Food Restaurant  0.17
    1           Pizza Place  0.11
    2    Italian Restaurant  0.11
    3        Ice Cream Shop  0.06
    4      Dutch Restaurant  0.06
    
    
    ----South Coogee----
                    venue  freq
    0                Café  0.22
    1                Park  0.08
    2  Italian Restaurant  0.05
    3              Bakery  0.05
    4               Hotel  0.05
    
    
    ----South Granville----
              venue  freq
    0   Golf Course  0.29
    1  Soccer Field  0.14
    2  Tennis Court  0.14
    3        Garden  0.14
    4    Playground  0.14
    
    
    ----South Penrith----
                    venue  freq
    0       Bowling Alley  0.07
    1        Liquor Store  0.07
    2  Italian Restaurant  0.07
    3   Convenience Store  0.07
    4         Coffee Shop  0.07
    
    
    ----South Turramurra----
                    venue  freq
    0                Park  0.12
    1        Soccer Field  0.12
    2              Bakery  0.12
    3                Café  0.12
    4  Athletics & Sports  0.12
    
    
    ----South Wentworthville----
                      venue  freq
    0  Fast Food Restaurant  0.14
    1       Thai Restaurant  0.07
    2              Platform  0.07
    3           Pizza Place  0.07
    4     Indian Restaurant  0.07
    
    
    ----South Windsor----
                      venue  freq
    0           Sports Club   0.1
    1           Golf Course   0.1
    2         Bowling Alley   0.1
    3  Fast Food Restaurant   0.1
    4    Basketball Stadium   0.1
    
    
    ----Spring Farm----
                      venue  freq
    0           Supermarket   0.5
    1         Shopping Mall   0.5
    2                   ATM   0.0
    3          Nudist Beach   0.0
    4  Outdoor Supply Store   0.0
    
    
    ----St Andrews----
                    venue  freq
    0            Platform  0.22
    1           Pet Store  0.11
    2       Shopping Mall  0.11
    3  Chinese Restaurant  0.11
    4        Home Service  0.11
    
    
    ----St Clair----
                      venue  freq
    0  Fast Food Restaurant  0.15
    1           Pizza Place  0.15
    2          Soccer Field  0.08
    3           Supermarket  0.08
    4        Sandwich Place  0.08
    
    
    ----St Helens Park----
                      venue  freq
    0         Shopping Mall   0.4
    1           Gas Station   0.2
    2  Fast Food Restaurant   0.2
    3           Pizza Place   0.2
    4                   ATM   0.0
    
    
    ----St Johns Park----
                        venue  freq
    0              Sports Bar  0.08
    1       Convenience Store  0.08
    2  Thrift / Vintage Store  0.08
    3               Gastropub  0.08
    4             Pizza Place  0.08
    
    
    ----St Leonards----
                     venue  freq
    0                 Café  0.23
    1  Japanese Restaurant  0.06
    2      Thai Restaurant  0.05
    3                  Bar  0.04
    4          Coffee Shop  0.04
    
    
    ----St Peters----
                      venue  freq
    0                  Café  0.22
    1       Thai Restaurant  0.07
    2               Brewery  0.07
    3  Gym / Fitness Center  0.05
    4                   Pub  0.05
    
    
    ----Stanhope Gardens----
                      venue  freq
    0  Fast Food Restaurant  0.15
    1                  Café  0.07
    2           Bus Station  0.07
    3              Pharmacy  0.04
    4           Flea Market  0.04
    
    
    ----Stanmore----
             venue  freq
    0         Café  0.23
    1          Pub  0.06
    2       Bakery  0.04
    3          Bar  0.04
    4  Pizza Place  0.04
    
    
    ----Strathfield----
                      venue  freq
    0     Korean Restaurant  0.21
    1                  Café  0.12
    2  Fast Food Restaurant  0.07
    3     Indian Restaurant  0.07
    4       Thai Restaurant  0.05
    
    
    ----Strathfield South----
                      venue  freq
    0                  Park  0.16
    1                  Café  0.09
    2  Fast Food Restaurant  0.06
    3           Pizza Place  0.06
    4        Farmers Market  0.06
    
    
    ----Surry Hills----
                     venue  freq
    0                 Café  0.15
    1      Thai Restaurant  0.07
    2                  Pub  0.05
    3          Coffee Shop  0.05
    4  Japanese Restaurant  0.04
    
    
    ----Sutherland----
                 venue  freq
    0             Café  0.21
    1  Thai Restaurant  0.07
    2              Gym  0.07
    3              Pub  0.07
    4   Sandwich Place  0.03
    
    
    ----Sydenham----
                       venue  freq
    0                   Café  0.20
    1  Vietnamese Restaurant  0.13
    2                    Pub  0.05
    3                Brewery  0.05
    4                 Bakery  0.03
    
    
    ----Sydney Olympic Park----
                  venue  freq
    0              Café  0.21
    1             Hotel  0.08
    2           Stadium  0.06
    3  Football Stadium  0.04
    4              Park  0.04
    
    
    ----Sylvania----
                     venue  freq
    0   Seafood Restaurant  0.16
    1          Supermarket  0.11
    2  Japanese Restaurant  0.05
    3     Asian Restaurant  0.05
    4                  Pub  0.05
    
    
    ----Sylvania Waters----
               venue  freq
    0           Café  0.17
    1            Gym  0.07
    2   Intersection  0.03
    3         Bakery  0.03
    4  Big Box Store  0.03
    
    
    ----Tamarama----
                       venue  freq
    0                   Café  0.20
    1  Australian Restaurant  0.08
    2             Restaurant  0.06
    3                  Beach  0.04
    4                 Bakery  0.04
    
    
    ----Taren Point----
                               venue  freq
    0                           Café  0.11
    1                  Deli / Bodega  0.05
    2  Paper / Office Supplies Store  0.05
    3                  Big Box Store  0.05
    4                  Grocery Store  0.05
    
    
    ----Telopea----
                    venue  freq
    0       Grocery Store  0.15
    1                 Pub  0.08
    2         Pizza Place  0.08
    3  Seafood Restaurant  0.08
    4  Chinese Restaurant  0.08
    
    
    ----Tempe----
                       venue  freq
    0                   Café  0.14
    1                 Bakery  0.06
    2  Vietnamese Restaurant  0.06
    3   Fast Food Restaurant  0.04
    4          Train Station  0.03
    
    
    ----Tennyson Point----
                 venue  freq
    0             Café  0.19
    1             Park  0.09
    2  Thai Restaurant  0.09
    3              Pub  0.03
    4      Pizza Place  0.03
    
    
    ----Terrey Hills----
                    venue  freq
    0                Café  0.27
    1         Flower Shop  0.18
    2  Italian Restaurant  0.09
    3   German Restaurant  0.09
    4          Restaurant  0.09
    
    
    ----The Ponds----
                    venue  freq
    0       Metro Station   0.2
    1       Grocery Store   0.2
    2       Shopping Mall   0.2
    3  Athletics & Sports   0.2
    4         Coffee Shop   0.2
    
    
    ----The Rocks----
                       venue  freq
    0                   Café  0.11
    1                  Hotel  0.06
    2  Australian Restaurant  0.04
    3                Theater  0.04
    4              Hotel Bar  0.03
    
    
    ----Thornleigh----
                    venue  freq
    0                Café  0.16
    1  Chinese Restaurant  0.11
    2     Thai Restaurant  0.11
    3       Grocery Store  0.11
    4         Post Office  0.05
    
    
    ----Toongabbie----
                      venue  freq
    0         Grocery Store  0.10
    1                   Gym  0.10
    2  Fast Food Restaurant  0.10
    3           Bus Station  0.10
    4         Shopping Mall  0.05
    
    
    ----Tregear----
                         venue  freq
    0              Sports Club  0.17
    1         Football Stadium  0.17
    2         Indoor Play Area  0.17
    3  Fruit & Vegetable Store  0.17
    4                   Buffet  0.17
    
    
    ----Turramurra----
               venue  freq
    0           Café  0.17
    1         Bakery  0.09
    2  Grocery Store  0.09
    3   Liquor Store  0.09
    4    Gas Station  0.04
    
    
    ----Turrella----
               venue  freq
    0           Park  0.12
    1  Train Station  0.10
    2           Café  0.05
    3       Platform  0.05
    4       Pharmacy  0.03
    
    
    ----Ultimo----
                 venue  freq
    0             Café  0.11
    1  Thai Restaurant  0.07
    2              Pub  0.06
    3              Bar  0.06
    4      Coffee Shop  0.06
    
    
    ----Vaucluse----
            venue  freq
    0       Beach  0.21
    1        Park  0.15
    2  Lighthouse  0.12
    3      Bakery  0.06
    4        Café  0.06
    
    
    ----Vineyard----
               venue  freq
    0      Pet Store  0.50
    1           Pool  0.25
    2  Train Station  0.25
    3            ATM  0.00
    4         Office  0.00
    
    
    ----Voyager Point----
                venue  freq
    0            Park  0.29
    1        Platform  0.29
    2             Pub  0.14
    3   Train Station  0.14
    4  Baseball Field  0.14
    
    
    ----Wahroonga----
               venue  freq
    0           Park  0.11
    1  Train Station  0.11
    2           Café  0.08
    3    Coffee Shop  0.06
    4  Grocery Store  0.06
    
    
    ----Waitara----
                   venue  freq
    0               Café  0.17
    1                Gym  0.06
    2  Electronics Store  0.04
    3             Bakery  0.04
    4   Asian Restaurant  0.04
    
    
    ----Wakeley----
                       venue  freq
    0  Vietnamese Restaurant  0.09
    1                   Park  0.06
    2        Thai Restaurant  0.06
    3                    Gym  0.06
    4            Supermarket  0.06
    
    
    ----Wallacia----
                         venue  freq
    0  Health & Beauty Service  0.14
    1               Campground  0.14
    2      Austrian Restaurant  0.14
    3                     Lake  0.14
    4                Rest Area  0.14
    
    
    ----Wareemba----
                    venue  freq
    0                Café  0.14
    1                Park  0.08
    2         Pizza Place  0.08
    3  Seafood Restaurant  0.06
    4     Thai Restaurant  0.06
    
    
    ----Warrawee----
                 venue  freq
    0             Café  0.11
    1    Grocery Store  0.09
    2  Thai Restaurant  0.07
    3             Park  0.07
    4     Liquor Store  0.07
    
    
    ----Warwick Farm----
                      venue  freq
    0  Fast Food Restaurant  0.22
    1   Arts & Crafts Store  0.11
    2        Sandwich Place  0.11
    3            Steakhouse  0.11
    4      Department Store  0.11
    
    
    ----Waterfall----
                      venue  freq
    0              Platform   0.4
    1                 Trail   0.2
    2         Train Station   0.2
    3  Fast Food Restaurant   0.2
    4                   ATM   0.0
    
    
    ----Waterloo----
             venue  freq
    0         Café  0.18
    1       Bakery  0.07
    2  Coffee Shop  0.06
    3  Pizza Place  0.06
    4          Bar  0.05
    
    
    ----Watsons Bay----
                 venue  freq
    0   Scenic Lookout  0.12
    1            Beach  0.12
    2  Harbor / Marina  0.09
    3             Park  0.09
    4            Trail  0.09
    
    
    ----Wattle Grove----
                      venue  freq
    0                Bakery  0.17
    1  Fast Food Restaurant  0.17
    2         Shopping Mall  0.17
    3                   Bar  0.08
    4         Grocery Store  0.08
    
    
    ----Waverley----
              venue  freq
    0          Café  0.13
    1   Pizza Place  0.06
    2  Burger Joint  0.04
    3          Park  0.04
    4   Coffee Shop  0.03
    
    
    ----Waverton----
                     venue  freq
    0                 Café  0.27
    1                 Park  0.16
    2  Japanese Restaurant  0.07
    3                  Bar  0.05
    4                 Pier  0.05
    
    
    ----Wedderburn----
                       venue  freq
    0            Pizza Place   1.0
    1           Nudist Beach   0.0
    2  Outdoors & Recreation   0.0
    3   Outdoor Supply Store   0.0
    4      Other Repair Shop   0.0
    
    
    ----Wentworth Point----
                      venue  freq
    0                  Café  0.13
    1                  Park  0.09
    2  Fast Food Restaurant  0.05
    3           Supermarket  0.05
    4    Dim Sum Restaurant  0.04
    
    
    ----Wentworthville----
                      venue  freq
    0  Fast Food Restaurant  0.12
    1           Supermarket  0.10
    2                  Park  0.07
    3              Platform  0.07
    4            Steakhouse  0.05
    
    
    ----Werrington----
                      venue  freq
    0  Fast Food Restaurant  0.15
    1                   Bar  0.08
    2              Platform  0.08
    3                  Park  0.08
    4         Shopping Mall  0.08
    
    
    ----Werrington County----
                      venue  freq
    0  Fast Food Restaurant   0.2
    1         Grocery Store   0.2
    2                   Bar   0.1
    3     Convenience Store   0.1
    4      Basketball Court   0.1
    
    
    ----Werrington Downs----
                      venue  freq
    0  Fast Food Restaurant  0.18
    1         Grocery Store  0.18
    2        Baseball Field  0.09
    3        Sandwich Place  0.09
    4      Department Store  0.09
    
    
    ----West Hoxton----
                      venue  freq
    0                  Café  0.18
    1           Supermarket  0.18
    2    Italian Restaurant  0.09
    3      Department Store  0.09
    4  Fast Food Restaurant  0.09
    
    
    ----West Pennant Hills----
             venue  freq
    0        Trail  0.25
    1         Park  0.25
    2  Supermarket  0.25
    3         Café  0.25
    4          ATM  0.00
    
    
    ----West Pymble----
                    venue  freq
    0  Italian Restaurant  0.22
    1             Dog Run  0.11
    2       Grocery Store  0.11
    3       Shopping Mall  0.11
    4          Toll Plaza  0.11
    
    
    ----West Ryde----
                     venue  freq
    0                 Café  0.08
    1      Thai Restaurant  0.08
    2  Japanese Restaurant  0.05
    3            Pet Store  0.05
    4          Supermarket  0.05
    
    
    ----Westleigh----
                    venue  freq
    0  Chinese Restaurant  0.18
    1       Shopping Mall  0.12
    2                Café  0.12
    3              Garden  0.06
    4  Athletics & Sports  0.06
    
    
    ----Westmead----
                      venue  freq
    0           Coffee Shop  0.10
    1       Thai Restaurant  0.07
    2  Fast Food Restaurant  0.07
    3              Platform  0.07
    4     Indian Restaurant  0.07
    
    
    ----Wetherill Park----
                      venue  freq
    0                  Café  0.13
    1  Fast Food Restaurant  0.13
    2    Italian Restaurant  0.10
    3                   Gym  0.07
    4       Thai Restaurant  0.07
    
    
    ----Whalan----
                       venue  freq
    0   Fast Food Restaurant  0.15
    1    Fried Chicken Joint  0.10
    2            Coffee Shop  0.10
    3                    Gym  0.10
    4  Portuguese Restaurant  0.05
    
    
    ----Whale Beach----
            venue  freq
    0      Bakery   0.2
    1        Park   0.2
    2        Café   0.2
    3       Beach   0.2
    4  Restaurant   0.2
    
    
    ----Wheeler Heights----
                   venue  freq
    0                Pub  0.11
    1     Gymnastics Gym  0.11
    2               Park  0.11
    3  Food & Drink Shop  0.11
    4             Arcade  0.11
    
    
    ----Wiley Park----
                      venue  freq
    0  Fast Food Restaurant  0.16
    1           Supermarket  0.08
    2   Lebanese Restaurant  0.08
    3                  Park  0.05
    4           Pizza Place  0.05
    
    
    ----Willmot----
                  venue  freq
    0          Pharmacy  0.12
    1  Asian Restaurant  0.12
    2     Train Station  0.12
    3     Shopping Mall  0.12
    4              Café  0.12
    
    
    ----Willoughby----
                     venue  freq
    0                 Café  0.17
    1  Japanese Restaurant  0.08
    2      Thai Restaurant  0.06
    3                 Park  0.06
    4               Bakery  0.05
    
    
    ----Willoughby East----
                     venue  freq
    0                 Café  0.19
    1  Japanese Restaurant  0.07
    2          Pizza Place  0.07
    3                  Pub  0.05
    4                 Park  0.05
    
    
    ----Windsor----
                 venue  freq
    0             Café  0.11
    1      Pizza Place  0.07
    2             Park  0.07
    3      Supermarket  0.04
    4  Thai Restaurant  0.04
    
    
    ----Windsor Downs----
                       venue  freq
    0               Wine Bar   0.5
    1              Pet Store   0.5
    2                    ATM   0.0
    3                 Office   0.0
    4  Outdoors & Recreation   0.0
    
    
    ----Winston Hills----
                    venue  freq
    0       Shopping Mall  0.12
    1                Café  0.12
    2  Chinese Restaurant  0.12
    3         Supermarket  0.12
    4   Food & Drink Shop  0.06
    
    
    ----Wisemans Ferry----
               venue  freq
    0  Boat or Ferry  0.43
    1            Pub  0.14
    2           Park  0.14
    3          River  0.14
    4  Bowling Green  0.14
    
    
    ----Wolli Creek----
                      venue  freq
    0         Train Station  0.07
    1                  Café  0.07
    2                  Park  0.07
    3                 Hotel  0.04
    4  Gym / Fitness Center  0.03
    
    
    ----Wollstonecraft----
                 venue  freq
    0             Café  0.25
    1      Coffee Shop  0.05
    2             Park  0.04
    3              Bar  0.04
    4  Thai Restaurant  0.04
    
    
    ----Woodbine----
                      venue  freq
    0  Fast Food Restaurant  0.19
    1      Department Store  0.10
    2                 Hotel  0.05
    3    Miscellaneous Shop  0.05
    4         Grocery Store  0.05
    
    
    ----Woodcroft----
                      venue  freq
    0           Zoo Exhibit  0.17
    1  Fast Food Restaurant  0.10
    2         Grocery Store  0.07
    3              Pharmacy  0.03
    4    Burmese Restaurant  0.03
    
    
    ----Woodpark----
                           venue  freq
    0                       Park  0.18
    1                        Pub  0.09
    2  Middle Eastern Restaurant  0.09
    3          Convenience Store  0.09
    4                  BBQ Joint  0.09
    
    
    ----Woollahra----
                       venue  freq
    0                   Café  0.16
    1     Italian Restaurant  0.07
    2                   Park  0.05
    3  Australian Restaurant  0.05
    4                    Bar  0.05
    
    
    ----Woolloomooloo----
                       venue  freq
    0                   Café  0.19
    1     Italian Restaurant  0.07
    2                   Park  0.05
    3    Japanese Restaurant  0.04
    4  Australian Restaurant  0.04
    
    
    ----Woolooware----
                      venue  freq
    0                  Café  0.18
    1                  Park  0.05
    2                   Bar  0.04
    3  Fast Food Restaurant  0.03
    4    Seafood Restaurant  0.03
    
    
    ----Woolwich----
                    venue  freq
    0                Park  0.15
    1                Pier  0.15
    2                Café  0.09
    3  Italian Restaurant  0.06
    4                 Bar  0.06
    
    
    ----Woronora----
              venue  freq
    0          Park  0.22
    1          Café  0.22
    2           Pub  0.11
    3     Gastropub  0.11
    4  Burger Joint  0.11
    
    
    ----Woronora Heights----
               venue  freq
    0            Pub  0.17
    1          River  0.17
    2           Park  0.17
    3  Grocery Store  0.17
    4  Shopping Mall  0.17
    
    
    ----Yagoona----
                    venue  freq
    0                Café  0.12
    1              Bakery  0.12
    2              Buffet  0.08
    3  Chinese Restaurant  0.08
    4    Malay Restaurant  0.04
    
    
    ----Yarramundi----
              venue  freq
    0    Campground  0.33
    1  Food Service  0.33
    2  Home Service  0.33
    3           ATM  0.00
    4  Optical Shop  0.00
    
    
    ----Yarrawarrah----
                  venue  freq
    0       Pizza Place   0.4
    1              Café   0.2
    2  Sushi Restaurant   0.2
    3          Pharmacy   0.2
    4               ATM   0.0
    
    
    ----Yennora----
                            venue  freq
    0        Fast Food Restaurant  0.11
    1                    Platform  0.07
    2  Construction & Landscaping  0.07
    3   South American Restaurant  0.04
    4                Soccer Field  0.04
    
    
    ----Yowie Bay----
                    venue  freq
    0                Café  0.50
    1              Garden  0.25
    2  Mexican Restaurant  0.25
    3                 ATM  0.00
    4         Opera House  0.00
    
    
    ----Zetland----
                 venue  freq
    0             Café  0.17
    1           Bakery  0.08
    2      Coffee Shop  0.05
    3      Pizza Place  0.05
    4  Thai Restaurant  0.05
    
    
    

### Merge the data into suburb_scc


```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
```


```python
num_top_venues = 5

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['SSC_NAME_2016']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
suburb_venues_sorted = pd.DataFrame(columns=columns)
suburb_venues_sorted['SSC_NAME_2016'] = sydney_grouped['SSC_NAME_2016']

for ind in np.arange(sydney_grouped.shape[0]):
    suburb_venues_sorted.iloc[ind, 1:] = return_most_common_venues(sydney_grouped.iloc[ind, :], num_top_venues)

suburb_venues_sorted.head()
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
      <th>SSC_NAME_2016</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbotsbury</td>
      <td>Convenience Store</td>
      <td>Park</td>
      <td>Other Great Outdoors</td>
      <td>Garden Center</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Abbotsford</td>
      <td>Park</td>
      <td>Café</td>
      <td>Thai Restaurant</td>
      <td>Australian Restaurant</td>
      <td>Pool</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Acacia Gardens</td>
      <td>Fast Food Restaurant</td>
      <td>Pizza Place</td>
      <td>Supermarket</td>
      <td>Café</td>
      <td>Gas Station</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Agnes Banks</td>
      <td>Rental Car Location</td>
      <td>Campground</td>
      <td>Zoo Exhibit</td>
      <td>Farmers Market</td>
      <td>Eastern European Restaurant</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Airds</td>
      <td>Baseball Field</td>
      <td>Supermarket</td>
      <td>Shopping Mall</td>
      <td>Gas Station</td>
      <td>Chinese Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



### Cluster Neighborhoods


```python
kclusters = 7

sydney_grouped_clustering = sydney_grouped.drop('SSC_NAME_2016', 1)
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(sydney_grouped_clustering)
kmeans.labels_ 
```




    array([4, 6, 3, 4, 0, 1, 0, 4, 4, 1, 1, 4, 4, 1, 4, 3, 3, 4, 6, 1, 2, 1,
           6, 1, 4, 4, 1, 4, 1, 1, 4, 3, 1, 6, 3, 1, 4, 1, 1, 4, 6, 4, 4, 5,
           1, 4, 1, 6, 1, 1, 1, 4, 3, 4, 4, 4, 3, 1, 1, 4, 4, 4, 4, 4, 6, 4,
           1, 4, 1, 6, 4, 4, 0, 4, 3, 1, 3, 1, 4, 1, 3, 1, 4, 4, 4, 4, 1, 4,
           1, 3, 4, 4, 3, 1, 4, 1, 4, 4, 4, 4, 3, 4, 1, 1, 1, 6, 4, 1, 1, 4,
           6, 3, 1, 1, 6, 1, 1, 1, 3, 1, 1, 1, 4, 6, 1, 1, 4, 1, 1, 4, 4, 1,
           3, 4, 4, 1, 1, 4, 1, 4, 1, 4, 4, 4, 3, 1, 1, 4, 1, 1, 4, 1, 4, 1,
           6, 6, 6, 4, 4, 4, 4, 4, 1, 3, 4, 1, 1, 0, 4, 3, 4, 4, 1, 1, 1, 4,
           4, 4, 1, 4, 3, 3, 4, 4, 1, 4, 1, 4, 2, 1, 1, 1, 6, 4, 4, 3, 3, 3,
           1, 3, 0, 4, 4, 1, 4, 3, 3, 4, 1, 1, 6, 6, 5, 3, 1, 4, 4, 1, 3, 4,
           4, 1, 3, 3, 3, 1, 4, 4, 1, 4, 3, 4, 4, 1, 1, 3, 1, 3, 1, 1, 1, 1,
           6, 0, 4, 4, 4, 0, 4, 4, 4, 3, 4, 4, 4, 4, 1, 4, 3, 4, 1, 3, 1, 1,
           4, 1, 6, 1, 4, 6, 4, 4, 4, 6, 1, 1, 4, 0, 1, 1, 4, 4, 1, 1, 1, 3,
           4, 2, 4, 6, 6, 1, 1, 3, 3, 4, 1, 1, 1, 1, 3, 2, 4, 1, 4, 4, 3, 1,
           1, 3, 4, 3, 4, 1, 4, 3, 4, 1, 3, 1, 4, 3, 4, 4, 1, 1, 3, 1, 1, 1,
           3, 3, 4, 4, 1, 3, 1, 1, 3, 4, 1, 4, 1, 1, 4, 1, 1, 6, 4, 4, 4, 1,
           4, 1, 3, 1, 1, 4, 6, 1, 4, 1, 4, 1, 3, 0, 4, 6, 1, 1, 1, 1, 6, 4,
           4, 1, 6, 4, 3, 1, 4, 1, 1, 6, 1, 1, 6, 1, 1, 4, 4, 4, 4, 1, 4, 4,
           4, 4, 1, 4, 1, 6, 1, 1, 1, 3, 4, 4, 4, 1, 4, 4, 1, 4, 4, 4, 1, 1,
           4, 3, 1, 4, 1, 1, 4, 1, 4, 6, 3, 1, 6, 1, 1, 1, 4, 3, 4, 1, 1, 3,
           1, 6, 4, 1, 3, 4, 0, 4, 3, 0, 4, 1, 1, 3, 1, 4, 4, 1, 1, 1, 1, 4,
           1, 1, 4, 4, 1, 1, 1, 4, 1, 1, 3, 4, 1, 4, 1, 6, 2, 6, 4, 1, 4, 4,
           1, 4, 3, 3, 1, 6, 3, 1, 1, 4, 1, 3, 3, 3, 3, 1, 6, 4, 4, 4, 4, 1,
           3, 6, 4, 3, 4, 1, 1, 4, 2, 4, 6, 4, 1, 3, 4, 4, 1, 1, 1, 6, 1, 4,
           4, 4, 1, 3, 1, 1])




```python
suburb_venues_sorted.insert(0, 'Cluster Labels', kmeans.labels_)
sydney_merged = suburb_scc

sydney_merged = sydney_merged.join(suburb_venues_sorted.set_index('SSC_NAME_2016'), on='SSC_NAME_2016')
sydney_merged.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
sydney_merged
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
      <th>SSC_CODE_2016</th>
      <th>SSC_NAME_2016</th>
      <th>1st_language</th>
      <th>2nd_language</th>
      <th>3rd_language</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Primary_school_ranking</th>
      <th>Secondary_school_ranking</th>
      <th>yr12_school_ranking</th>
      <th>The median of Housing price (k Dollars)</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12321</td>
      <td>Lilli Pilli</td>
      <td>24.0</td>
      <td>17.0</td>
      <td>15</td>
      <td>-34.068333</td>
      <td>151.115833</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2400.0</td>
      <td>4.0</td>
      <td>Pool</td>
      <td>Non-Profit</td>
      <td>Harbor / Marina</td>
      <td>Athletics &amp; Sports</td>
      <td>Bakery</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14344</td>
      <td>Wisemans Ferry</td>
      <td>17.0</td>
      <td>50.0</td>
      <td>1</td>
      <td>-33.381944</td>
      <td>150.985000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>Boat or Ferry</td>
      <td>Pub</td>
      <td>Bowling Green</td>
      <td>Park</td>
      <td>River</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11213</td>
      <td>Darlington</td>
      <td>44.0</td>
      <td>7.0</td>
      <td>1</td>
      <td>-33.890334</td>
      <td>151.196610</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1400.0</td>
      <td>1.0</td>
      <td>Café</td>
      <td>Bar</td>
      <td>Pub</td>
      <td>Coffee Shop</td>
      <td>Thai Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11392</td>
      <td>Elderslie</td>
      <td>16.0</td>
      <td>23.0</td>
      <td>52</td>
      <td>-34.054620</td>
      <td>150.712500</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>660.0</td>
      <td>1.0</td>
      <td>Pub</td>
      <td>Farmers Market</td>
      <td>Café</td>
      <td>Italian Restaurant</td>
      <td>Thai Restaurant</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12273</td>
      <td>Lansdowne</td>
      <td>16.0</td>
      <td>23.0</td>
      <td>52</td>
      <td>-31.783056</td>
      <td>152.534444</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>641</th>
      <td>13701</td>
      <td>Sutherland</td>
      <td>32.0</td>
      <td>39.0</td>
      <td>7</td>
      <td>-34.029742</td>
      <td>151.059275</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1100.0</td>
      <td>1.0</td>
      <td>Café</td>
      <td>Pub</td>
      <td>Gym</td>
      <td>Thai Restaurant</td>
      <td>Athletics &amp; Sports</td>
    </tr>
    <tr>
      <th>642</th>
      <td>12371</td>
      <td>Loftus</td>
      <td>32.0</td>
      <td>42.0</td>
      <td>24</td>
      <td>-34.046500</td>
      <td>151.047100</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1000.0</td>
      <td>4.0</td>
      <td>Athletics &amp; Sports</td>
      <td>Gym</td>
      <td>Museum</td>
      <td>Light Rail Station</td>
      <td>Basketball Court</td>
    </tr>
    <tr>
      <th>643</th>
      <td>11422</td>
      <td>Engadine</td>
      <td>32.0</td>
      <td>42.0</td>
      <td>24</td>
      <td>-34.065722</td>
      <td>151.012664</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>917.0</td>
      <td>1.0</td>
      <td>Café</td>
      <td>Fast Food Restaurant</td>
      <td>Sandwich Place</td>
      <td>Bakery</td>
      <td>Bowling Alley</td>
    </tr>
    <tr>
      <th>644</th>
      <td>14485</td>
      <td>Yarrawarrah</td>
      <td>18.0</td>
      <td>17.0</td>
      <td>24</td>
      <td>-34.055200</td>
      <td>151.032100</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1000.0</td>
      <td>1.0</td>
      <td>Pizza Place</td>
      <td>Café</td>
      <td>Sushi Restaurant</td>
      <td>Pharmacy</td>
      <td>Fast Food Restaurant</td>
    </tr>
    <tr>
      <th>645</th>
      <td>14422</td>
      <td>Woronora Heights</td>
      <td>18.0</td>
      <td>17.0</td>
      <td>7</td>
      <td>-34.030434</td>
      <td>151.030555</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1100.0</td>
      <td>4.0</td>
      <td>Pub</td>
      <td>Park</td>
      <td>Pizza Place</td>
      <td>Shopping Mall</td>
      <td>Grocery Store</td>
    </tr>
  </tbody>
</table>
<p>646 rows × 17 columns</p>
</div>



### Cluster suburbs of Sydney City


```python
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=11)

x = np.arange(kclusters)
ys = [i + x + (i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

markers_colors = []
for lat, lon, poi, cluster, price in zip(sydney_merged['Latitude'], sydney_merged['Longitude'], sydney_merged['SSC_NAME_2016'], sydney_merged['Cluster Labels'], sydney_merged['The median of Housing price (k Dollars)']):
    if np.isnan(cluster):
        pass
    else:
        label = folium.Popup(str(poi) + ' Cluster ' + str(cluster) + ', Housing Price: ' + str(price) + 'k dollars', parse_html=True)
        folium.CircleMarker(
            [lat, lon],
            radius=5,
            popup=label,
            color=rainbow[int(cluster-1)],
            fill=True,
            fill_color=rainbow[int(cluster-1)],
            fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```
<img src="./2.png" height="300" width="600">


```python

```

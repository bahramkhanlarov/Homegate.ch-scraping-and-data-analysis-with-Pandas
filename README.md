# Web Scraping and Visualization of Rental Data in Geneva(Switzerland) with Python


This part is extracted from project "Analysis on Apartment rental prices between Switzerland and USA Markets" realized as a part my master studies in HSLU (Lucerne University of Applied Sciences) in collaboration with Aleksandra Bundovska (aleksandra.bundovska@stud.hslu.ch) and Yang Bai (yang.bai@stud.hslu.ch). 

In this article, I explain how you can:

- Get the rental data by web scraping and converting it to Pandas data frame
- Rental data analysis (For the rental data analysis see <ins>[this notebook](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Homegate_Data%20_Transformation%20&%20Data_Enrichment.ipynb)</ins>)
- Use the Geopandas library to convert maps which are in shapefile to Geopandas data frames and Geojson maps
- Make interactive Choropleth maps embedded with rental data, which looks like this:

![Choropleth](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/76f534623a9d4cc652040843fd38d6ce3fab6d47/Photos/visualization.svg)

## 1. Web Scraping for rental data

If available, it is easier/recommended to use an API (similar to twitter, reddit APIs) . However, most of the websites don't have public APIs or if they have they provide very limited public access. In such cases, web scraping might be necessary.

This was the case for me when I wanted to study rental data in Geneva. So, I decided to develop my own web crawler to get the data for analysis (not any business purposes) from Homegate.ch. Below I explain this web crawler. You can find the corresponding Jupyter notebook <ins>[here](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Homegate-Geneva%20scraping%20code.ipynb)</ins>.

After going to webpage <ins>[Homegate.ch](https://www.homegate.ch/)</ins> we select rent, type the city name in search box, and select language (eng) from top right:

![Homegate main page](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Photos/Screenshot%202022-07-26%20at%2022.59.09.png)

The information that i want to extract is the <ins>**price, size, number of rooms and the address**:</ins>

![Homegate main page](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Photos/Screenshot%202022-07-26%20at%2023.08.47.png)

1. First step is to import necessary packages and as well as beautifulsoup and requests modules for scraping, this is how you could do it:


```python
from bs4 import BeautifulSoup
import requests
import csv
import pandas as pd
```

2.  We define page number here for looping pagination

```python
cur_page = 1
```

3. After carefully inspecting website, realized apt rental advertisement goes in 2 categories: Premium one (paid subscription) and simple ones (without subscription) therefore create 2 empty list and then creating function for changing page numbers and While True loop has to run until it breaks:

```python
premium = []
simple = []


def getLink(page):
    return f"https://www.homegate.ch/rent/apartment/city-geneva/matching-list?ep={page}"
    
while True:
    print("Page ->", cur_page)
    link = getLink(cur_page)
    
    res = requests.get(link)
```

4. We parse HTML with Beautiful Soup (I strongly suggest you to take a look at python documentation of [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/):

```python
bs = BeautifulSoup(res.text, features='html.parser')
```

5. We define 2 variables-- a for premium and b for simple annoucements and with find_all() we returns all div containers with mentioned class names that match our filters:

```python

a = bs.find_all('div', {'class': 'ListItemTopPremium_item_K9dLF'})
b = bs.find_all('div', {'class': 'ListItem_item_1GcIZ'})
```
6. If we get zero results then we break the while loop defined earlier (we have 21 pages of results) otherwise we run for loop for findings in a and b then append according to the empty list created earlier:

```python
if len(a) == 0 and len(b) == 0:
        break
        
    for offer in a:
        premium.append(offer)
    for offer in b:
        simple.append(offer)
```

printing results and incrementing page number by one 

    print(len(premium), len(simple))
    cur_page += 1

7. Defining function block with result dic keys price,size,rooms,address and for each of them we use try except to get within span tag with mentioned class name info needed  and add them to created list:


```python
def extractPremiumInfo(block):
    result = {
        'price': None,
        'size': None,
        'rooms': None,
        'address': None
    }
    try:
        price = block.find('span', {'class': 'ListItemPrice_price_1o0i3'}).find_all('span')[1].text
        result['price'] = price
    except:
        pass

    try:
        m2 = block.find('span', {'class': 'ListItemLivingSpace_value_2zFir'}).text
        result['size'] = m2
    except:
        pass

    try:
        rn = block.find('span', {'class': 'ListItemRoomNumber_value_Hpn8O'}).text
        result['rooms'] = rn
    except:
        pass

    address = block.find('div', {'class': 'ListItemTopPremium_data_3i7Ca'})
    if address is None:
        address = block.find('div', {'class': 'ListItem_data_18_z_'})

    address = address.find_all('p')[1].text

    result['address'] = address

    return result
    
```
    
8. Again with for loop we go over simple and premium list and append results to fnish list:

```python

finish = []

for i in premium:
    finish.append(extractPremiumInfo(i))

for i in simple:
    finish.append(extractPremiumInfo(i))

print(f"Found {len(finish)}apartments")

```

9. Saving the extracted data into pandas dataframe and write to a CSV file:

```python
df = pd.DataFrame(finish)
df.to_csv('Geneva_listings_src.csv', index=False, encoding='utf-8')
```



## 2. Rental-Data Analysis

Once you have the rental data in the form of a Pandas dataframe you can do the usual data analysis pipeline. That is, you start by preprocessing the data (handling the missing data, outliers, etc.). For the data analysis, you can include new interesting features such as rent per room, rent per area, zip code of the apartments, etc. These are all done in <ins>[this notebook](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Homegate_Data%20_Transformation%20&%20Data_Enrichment.ipynb)</ins>. Perhaps, the most tricky part of the data analysis pipeline for this example is spotting and handling the outliers (which are indeed mostly due to wrong inputs from the users). Here is the first 5 elements of the resulting dataframe:


|  Price  |  Size | Rooms |              Address             |
|:-------:|------:|------:|:--------------------------------:|
| 4,150.– | 104m2 | 2.5rm | Rue de l'Athénée 38, 1206 Genf   |
| 1,250.– |  26m2 |   1rm | Rue de la Dôle 15, 1203 Genève   |
| 4,000.– |  90m2 | 2.5rm | Rue de l'Athénée 36, 1206 Genève |
| 3,100.– |  82m2 |   4rm | Rue Liotard, 1202 Geneva         |
| 1,580.– |   NaN | 2.5rm | Rue de Lyon, 1201 Genève         |

Let's say you are interested in rental prices distribution as a function of zip-code. Then you could use the `groupBy()` method of Pandas on the above dataframe as follows:

```python

zipVsRentMean = df[['ZipCode', 'RentPerArea', 'RentPerRoom', 'AreaPerRoom', 'SurfaceArea']]\
                .groupby(['ZipCode'], as_index = False).mean()
```
Here is `zipVsRentMean`:

| ZipCode | RentPerArea | RentPerRoom | AreaPerRoom | SurfaceArea |
|--------:|------------:|------------:|------------:|-------------|
|    1200 |   40.787924 |  899.814815 |   22.503367 |  106.666667 |
|    1201 |   41.403102 |  882.109565 |   21.283923 |   82.142857 |
|    1202 |   37.059230 |  818.934074 |   22.243254 |   85.266667 |
|    1203 |   37.527645 |  716.131490 |   19.320108 |   64.234043 |
|    1204 |   44.574250 | 1117.337317 |   25.277993 |   88.071429 |
|    1205 |   35.181856 |  698.735049 |   20.106478 |   75.918919 |
|    1206 |   39.905645 | 1103.584285 |   27.531853 |  143.326923 |
|    1207 |   41.646907 |  904.179500 |   21.860883 |  100.052632 |
|    1208 |   36.857806 |  909.245248 |   24.852548 |   88.071429 |
|    1209 |   37.278602 |  999.223665 |   27.183622 |  129.666667 |


## 3. Read Shapefiles and convert them to Geopandas dataframes

Next, we would like to show the results of the zip code table above on a map. To this end, we first should be able to read the maps in Python. Maps are usually available in the shapefile format *.shp. Let's first download this shapefile map, and then I discuss how you could read this in Python.

Download the Switzerland's zip- code shapefiles from <ins>[Swiss opendata](https://opendata.swiss/en/dataset/amtliches-ortschaftenverzeichnis-mit-postleitzahl-und-perimeter)</ins>. I have downloaded the PLZO_SHP_LV95 from <ins>[here](https://data.geo.admin.ch/ch.swisstopo-vd.ortschaftenverzeichnis_plz/PLZO_SHP_LV03.zip)</ins>). Extract the folder, and note the address where you saved the zip-code shapefile (called PLZO_PLZ.shp) . You can also get it <ins>[here](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/tree/Master/PLZO_SHP_LV03)</ins>.

Okay, now you have the shapefile. How would you read/manipulate this in Python? Luckily, the Geopandas library of Python, which is a powerful library used for geospatial data processing and analysis, has a method to convert shapefiles to geopandas dataframe:
```python

import geopandas as gpd

gdf = gpd.read_file('.../PLZO_SHP_LV95/PLZO_PLZ.shp')
```
The Coordinate Reference System (CRS) in which the data is displayed can be found by gdf.crs. I convert this to a more common CRS by the following command:
```python
gdf = gdf.to_crs({'init': 'espg:4326'})
```
Here is the first four elements of the geopandas dataframe gdf:

| 2635 | UUID                                   | OS_UUID                                |STATUS|INAEND| PLZ  | ZUSZIFF | Geometry
|-----:|---------------------------------------:|---------------------------------------:|-----:|-----:|-----:|--------:|--------------------------------------------------:|
| 3370 | {54A45D65-97A3-45A1-8DB2-FA3E6E540269} | {5DF8DDBE-8D41-42A3-8F30-F9E716E39C75} | real | nein | 1203 | 0       | POLYGON ((6.13514 46.20837, 6.13470 46.20798, ... |
| 3456 | {D924C540-1604-4E4A-9C30-A31E36299921} | {5DF8DDBE-8D41-42A3-8F30-F9E716E39C75} | real | nein | 1206 | 0       | POLYGON ((6.15383 46.17984, 6.15387 46.18019, ... |
| 3485 | {F97E72AA-A260-4075-B3AE-F87FEDE38726} | {5DF8DDBE-8D41-42A3-8F30-F9E716E39C75} | real | nein | 1205 | 0       | POLYGON ((6.13394 46.20368, 6.13408 46.20308, ... |
| 3531 | {B5EA9714-EF37-41F0-B481-F59A93221892} | {5DF8DDBE-8D41-42A3-8F30-F9E716E39C75} | real | nein | 1207 | 0       | POLYGON ((6.15741 46.20996, 6.15746 46.21001, ... |

The `geometry` column defines the shape of each polygon. Since we are only looking at the data in the city of Geneva, I extract the data of Geneva from `gdf` (note that `gdf` includes the data of the whole Switzerland):

First I create list of zip codes i have for Geneva:
```python
geneva = [1200, 1201, 1202,1203,1204,1205,1206,1207,1208,1209]
````
Then creating geopandas frame for Geneva with zipcodes in gdf are contained in geneva list:
```python
gdf_gen = gdf[gdf['PLZ'].isin(geneva)]
```
Now you can plot the zip-code map of Lausanne with the following code:

```python
gdf_gen.plot()
```
Which would result in the following figure:
![photo](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Photos/download.png)

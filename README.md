# Web Scraping and Visualization of Rental Data in Geneva(Switzerland) with Python


This part is extracted from project "Analysis on Apartment rental prices between Switzerland and USA Markets" realized as a part my master studies in HSLU (Lucerne University of Applied Sciences) in collaboration with Aleksandra Bundovska (aleksandra.bundovska@stud.hslu.ch) and Yang Bai (yang.bai@stud.hslu.ch). 

In this article, I explain how you can:

- Get the rental data by web scraping and converting it to Pandas data frame
- Use the Geopandas library to convert maps which are in shapefile to Geopandas data frames and Geojson maps
- Make interactive Choropleth maps embedded with rental data, which looks like this:

![Choropleth](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/76f534623a9d4cc652040843fd38d6ce3fab6d47/Photos/visualization.svg)

## 1. Web Scraping for rental data

If available, it is easier/recommended to use an API (similar to twitter, reddit APIs) . However, most of the websites don't have public APIs or if they have they provide very limited public access. In such cases, web scraping might be necessary.

This was the case for me when I wanted to study rental data in Geneva. So, I decided to develop my own web crawler to get the data for analysis (not any business purposes) from Immoscout24.ch. Below I explain this web crawler. You can find the corresponding Jupyter notebook <ins>[here](https://github.com/bkhan1820/Homegate.ch-scraping-and-data-analysis-with-Pandas/blob/Master/Homegate-Geneva%20scraping%20code.ipynb)</ins>.

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



# Web Scraping and Visualization of Rental Data in Switzerland with Python

In this article, I explain how you can:

* Get the rental data by web scraping and converting it to Pandas data frame
* Use the Geopandas library to convert maps which are in  [shapefile](https://en.wikipedia.org/wiki/Shapefile) to Geopandas data frames and Geojson maps 
* Make interactive Choropleth maps embedded with rental data, which looks like this (hover over the map to see average rent per room for each zip-code):

<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="https://cdn.jsdelivr.net/npm/vega@4"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-lite@3.0.0-rc3"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-embed@3"></script>
</head>
<body>  
  <div id="vis" style="text-align:center"></div>
  <script>
    var spec = "https://raw.githubusercontent.com/hamedrazavi/rental_analysis_switzerland_immoscout24/master/data/lausanne_zip_rental.json";
    var opt = {"actions": false,
               "padding": {left: 5, top: 5, right: 5, bottom: 5}};
  	vegaEmbed('#vis', spec, opt).catch(console.warn);
  </script>
</body>
</html>


## Web Scraping for rental data

If available, it is easier/recommended to use an API (similar to twitter, reddit APIs) . However, most of the websites don't have public APIs or if they have they provide very limited public access. In such cases, web scraping might be necessary. 

This was the case for me when I wanted to study rental data in Switzerland. So, I decided to develop my own web crawler to get the data for analysis (not any business purposes) from Immoscout24.ch. Below I explain this web crawler. You can find the corresponding Jupyter notebook [here](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/immoscoutHtml_to_text_once.ipynb). 

1. First step is to download the web html pages.  For instance, for city of Lausanne this is how you could do it:

   ```python
   import requests
   first_page = requests.get('https://www.immoscout24.ch/en/flat/rent/city-lausanne')
   ```

   This snippet downloads the first page of the rental data of the city of Lausanne. 

2. Use [```html2text```](https://github.com/aaronsw/html2text) script written by Aaron Swartz to convert the html page to a text file. This script gets rid of all html syntaxes:

   ```python
   import html2text
   h = html2text
   c = h.html2text(first_page.text)
   ```

   You can ``print(c)`` to see how it looks like. You can get meta info such as total number of pages from the first page data and then continue downloading all pages as explained in [this](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/immoscoutHtml_to_text_once.ipynb) Jupyter notebook. I have saved all pages in ``*.txt`` form. 

3. Split each txt file using regular expressions so that you can get each apartment listing: I call it ``singe_apt_listing`` which is simply a string containing one single apartment listing. Here is an example of that: 

   ```
   ### [3 rooms, 97 m²](/en/d/flat-rent-
   lausanne/5071748?s=2&t=1&l=2023&ct=609&ci=220&pn=10)
   
   ## «Ruchonnet 24 - Appt 3 pièces au rez-de-chaussée»
   
   Av. Ruchonnet 24, 1003 Lausanne, VD
   
   Close
   
   ### CHF 2,499.—
   
   Bookmark
   ```

4. Then I use ``regex`` in [this script](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/SingleApt.py) to extract the information such as number of rooms, area, etc. from the string above and convert it to Pandas dataframe. This step is a bit tricky as not all single apartment listings look as nice as the one above. So the script has to cover all different cases without outputing wrong data. 

5. Finally, I concatenate all single apartment data frames to a Pandas data frame which will be convenient for data analysis. 

   All these steps are integrated into a class `ImmoScout` which can be used as follows:

   ```python
   from ImmoScout import ImmoScout
   import pandas as pd
   
   # instantiate the ImmoScout class by setting the city name and listing type
   lausanne = ImmoScout(city_name = 'lausanne', list_type = 'rent')
   
   # if some data is already downloaded set 'in_path' to the already saved csv file,
   # otherwise just set the output path, i.e., out_path to save the converted data
   lausanne.to_csv(in_path = '', out_path='../data/lausanne.csv')
   
   # convert the csv to pandas data frame
   df = pd.read_csv('../data/lausanne.csv')
   ```

    

## Rental-Data analysis

Once you have the rental data in the form of a Pandas dataframe you can do the usual data analysis  pipeline. That is, you start by preprocessing the data (handling the missing data, outliers, etc.). For the data analysis, you can include new interesting features such as rent per room, rent per area, zip code of the apartments, etc. These are all done in [this notebook](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/rental_analysis_lausanne.ipynb). Perhaps, the most tricky part of the data analysis pipeline for this example is spotting and handling the outliers (which are indeed mostly due to wrong inputs from the users). Here is the first 5 elements of the resulting dataframe:



<div  style="overflow:auto;">
<table border="1">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>Id</th>
      <th>SurfaceArea</th>
      <th>NumRooms</th>
      <th>Type</th>
      <th>Address</th>
      <th>Description</th>
      <th>Rent</th>
      <th>Bookmark</th>
      <th>Link</th>
      <th>RentPerArea</th>
      <th>RentPerRoom</th>
      <th>AreaPerRoom</th>
      <th>ZipCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>4695812</td>
      <td>41.0</td>
      <td>2.0</td>
      <td>flat</td>
      <td>Av. de Cour 65, 1007 Lausanne, VD</td>
      <td>Appartement de 2 pièces au 5ème étage</td>
      <td>NaN</td>
      <td>New</td>
      <td>/en/d/flat-rent-lausanne/4695812?s=2&amp;t=1&amp;l=202...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>20.500000</td>
      <td>1007</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>5204125</td>
      <td>21.0</td>
      <td>1.5</td>
      <td>studio</td>
      <td>Route Aloys-Fauquez 122, 1018 Lausanne, VD</td>
      <td>Studio proche de toutes les commodités</td>
      <td>970.0</td>
      <td>New</td>
      <td>/en/d/studio-rent-lausanne/5204125?s=2&amp;t=1&amp;l=2...</td>
      <td>46.190476</td>
      <td>646.666667</td>
      <td>14.000000</td>
      <td>1018</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>5201717</td>
      <td>95.0</td>
      <td>4.0</td>
      <td>flat</td>
      <td>Av. de Morges 39, 1004 Lausanne, VD</td>
      <td>Joli appartement - centre ville proche commodités</td>
      <td>2085.0</td>
      <td>NaN</td>
      <td>/en/d/flat-rent-lausanne/5201717?s=2&amp;t=1&amp;l=202...</td>
      <td>21.947368</td>
      <td>521.250000</td>
      <td>23.750000</td>
      <td>1004</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>5201713</td>
      <td>29.0</td>
      <td>1.5</td>
      <td>flat</td>
      <td>Ch. du Devin 57, 1012 Lausanne, VD</td>
      <td>Quartier de Chailly, spacieux 1.5 pièce</td>
      <td>910.0</td>
      <td>NaN</td>
      <td>/en/d/flat-rent-lausanne/5201713?s=2&amp;t=1&amp;l=202...</td>
      <td>31.379310</td>
      <td>606.666667</td>
      <td>19.333333</td>
      <td>1012</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5195729</td>
      <td>11.0</td>
      <td>NaN</td>
      <td>single-room</td>
      <td>Chemin de Montolivet 19, 1006 Lausanne, VD</td>
      <td>1006 Lausanne - Montolivet 19 - Chambre meublé...</td>
      <td>560.0</td>
      <td>NaN</td>
      <td>/en/d/single-room-rent-lausanne/5195729?s=2&amp;t=...</td>
      <td>50.909091</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1006</td>
    </tr>
  </tbody>
</table>
</div>


Let's say you are interested in rental prices distribution as a function of zip-code. Then you could use the ``groupBy()`` method of Pandas on the above dataframe as follows:

```python
zipVsRentMean = df[['ZipCode', 'RentPerArea', 'RentPerRoom', 'AreaPerRoom', 'SurfaceArea']].groupby(['ZipCode'], as_index = False).mean()
```

Here is ``zipVsRentMean``:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ZipCode</th>
      <th>RentPerArea</th>
      <th>RentPerRoom</th>
      <th>AreaPerRoom</th>
      <th>SurfaceArea</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1000</td>
      <td>28.458524</td>
      <td>729.675634</td>
      <td>25.975658</td>
      <td>131.923077</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1003</td>
      <td>33.847367</td>
      <td>956.716565</td>
      <td>29.864104</td>
      <td>77.057471</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1004</td>
      <td>29.348282</td>
      <td>697.231981</td>
      <td>25.349916</td>
      <td>65.488971</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1005</td>
      <td>29.793665</td>
      <td>718.242015</td>
      <td>24.671657</td>
      <td>73.903226</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1006</td>
      <td>32.542634</td>
      <td>789.203604</td>
      <td>26.219773</td>
      <td>61.670213</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1007</td>
      <td>32.612209</td>
      <td>774.724010</td>
      <td>26.186800</td>
      <td>59.460000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1010</td>
      <td>28.715491</td>
      <td>714.434524</td>
      <td>26.260718</td>
      <td>76.546512</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1012</td>
      <td>30.620128</td>
      <td>789.353729</td>
      <td>27.079760</td>
      <td>69.865385</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1015</td>
      <td>23.448276</td>
      <td>616.428571</td>
      <td>24.857143</td>
      <td>87.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1018</td>
      <td>29.406521</td>
      <td>668.998769</td>
      <td>24.128802</td>
      <td>57.859504</td>
    </tr>
  </tbody>
</table>



For the moment, I will drop rows of zip code 1015  as I did not have enough data points. 



## Read Shapefiles and convert them to Geopandas dataframes

Next, we would like to show the results of the zip code table above on a map. To this end, we first should be able to read the maps in Python. Maps are usually available in the shapefile format ``*.shp``. Let's first download this shapefile map, and then I discuss how you could read this in Python. 

Download the Switzerland's zip- code shapefiles from [Swiss opendata](https://opendata.swiss/en/dataset/amtliches-ortschaftenverzeichnis-mit-postleitzahl-und-perimeter). I have downloaded the PLZO_SHP_LV95 from [here](http://data.geo.admin.ch/ch.swisstopo-vd.ortschaftenverzeichnis_plz/PLZO_SHP_LV95.zip). Extract the folder, and note the address where you saved the zip-code shapefile (called ``PLZO_PLZ.shp``) . I put it in my ``data`` folder. 

Okay, now you have the shapefile. How would you read/manipulate this in Python? Luckily, the [Geopandas](http://geopandas.org/) library of Python, which is a powerful library used for geospatial data processing and analysis, has a method to convert shapefiles to geopandas dataframe:

``` Python
import geopandas as gpd
gdf = gpd.read_file('../data/PLZO_SHP_LV95/PLZO_PLZ.shp')
```

The Coordinate Reference System (CRS) in which the data is displayed can be found by ``gdf.crs``. I convert this to a more common CRS by the following command:

```Python
gdf = gdf.to_crs({'init': 'espg:4326'})
```

Here is the first two elements of the geopandas dataframe ``gdf``: 

<div  style="overflow:auto;">
<table border="1">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>UUID</th>
      <th>OS_UUID</th>
      <th>STATUS</th>
      <th>INAEND</th>
      <th>PLZ</th>
      <th>ZUSZIFF</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>{0072F991-E46D-447E-A3BE-75467DB57FFC}</td>
      <td>{281807DC-9C0B-4364-9A55-0E8956876194}</td>
      <td>real</td>
      <td>nein</td>
      <td>3920</td>
      <td>0</td>
      <td>POLYGON ((7.575852862870608 45.98819190048114,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>{C3D3316F-1DFE-468E-BFC5-5C2308743B4C}</td>
      <td>{F065D58C-3F88-46EF-9AA0-DA0A96766333}</td>
      <td>real</td>
      <td>nein</td>
      <td>3864</td>
      <td>0</td>
      <td>POLYGON ((8.114458487426123 46.5465644007604, ...</td>
    </tr>
  </tbody>
</table>
</div>


The ``geometry`` column defines the shape of each polygon. Since we are only looking at the data in the city of Lausanne, I extract the data of Lausanne from ``gdf`` (note that ``gdf`` includes the data of the whole Switzerland):

```python
lausanne = [1000, 1003, 1004, 1005, 1006, 1007, 1010, 1010, 1012, 1015, 1018]
gdf_laus = gdf[gdf['PLZ'].isin(lausanne)]
```

Now you can plot the zip-code map of Lausanne with the following code:

```python
gdf_laus.plot()
```

Which would result in the following figure:

![gdf_laus](https://raw.githubusercontent.com/hamedrazavi/rental_analysis_switzerland_immoscout24/master/data/gdf_laus.jpg)

While ``geopandas`` can plot such minimal maps, I would like to have a Choropleth interactive map (where you can hover over the map see the rental results) that also looks a bit nicer than this one. To create such a map I decided to use the use the [Altair](https://github.com/hamedrazavi/altair_quick_tutorial) library. 

## Create interactive Choropleth map embedded with rental data

First off, let's merge the ``gdf_laus`` dataframe which only contains geographical data with ``zipVsRentMean`` Pandas dataframe which included the rental data for each zip-code in Lausanne:

```Python
gdf_laus = gdf_laus.merge(zipVsRentMean, left_on='PLZ', right_on='ZipCode')
```

This will simply add the columns of ``zipVsRentMean`` to the right of ``gdf_laus``. Okay, now we have a geopandas dataframe ``gdf_laus``, which includes both rental data and geographical information of Lausanne. Next, we want to visualize this on an interactive Choropleth map for which I use the Altair library. 

In order for the ``gdf_laus`` data to be readable by the Altair library, we need to do some preprocessing as follows:

```  Python
import json
import Altair as alt
json_laus = json.loads(gdf_laus.to_json())
alt_laus = alt.Data(values = json_laus['features'])
```

``alt_laus`` has the data form which is readable by Altair as follows:

```Python
alt_rentPerRoom = alt.Chart(alt_laus).mark_geoshape(
    stroke = 'white'
).encode(
    latitude = 'properties.y:Q',
    longitude = 'properties.x:Q',
    color = 'properties.RentPerRoom:Q'
).properties(
    width = 400,
    height = 500
)
alt_rentPerRoom # to display the map
```

Here is the result of the execution of this code:

<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="https://cdn.jsdelivr.net/npm/vega@4"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-lite@3.0.0-rc3"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-embed@3"></script>
</head>
<body>  
  <div id="vis_no_text" style="text-align:center"></div>
  <script>
    var spec = "https://raw.githubusercontent.com/hamedrazavi/rental_analysis_switzerland_immoscout24/master/data/lausanne_zip_rental_no_text.json";
    var opt = {"actions": false,
               "padding": {left: 5, top: 5, right: 5, bottom: 5}};
  	vegaEmbed('#vis_no_text', spec, opt).catch(console.warn);
  </script>
</body>
</html>

We are almost there, but we need to display the zip-code on the map too. To do this, I first get the *centroid* of each zip-code area by the following commands:

```python
gdf_laus['x'] = gdf_laus['geometry'].centroid.x
gdf_laus['y'] = gdf_laus['geometry'].centroid.y
```

Next, plot this with altair:

```python
text  = alt.Chart(alt_laus).mark_text(
        
).encode(
    longitude = 'properties.x:Q',
    latitude = 'properties.y:Q',
    text = 'properties.ZipCode:Q',
)
```

Finally, combine the two maps:

```python
chart = alt_rentPerRoom + text
chart
```

Here is the result. You can hover over the map to see the rent per room average in each zip-code area. We can do the same analysis for any other variable of interest (e.g., rent per surface area, etc.). 

<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="https://cdn.jsdelivr.net/npm/vega@4"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-lite@3.0.0-rc3"></script>
	<script src="https://cdn.jsdelivr.net/npm/vega-embed@3"></script>
</head>
<body>  
  <div id="vis_rent_per_room" style="text-align:center"></div>
  <script>
    var spec = "https://raw.githubusercontent.com/hamedrazavi/rental_analysis_switzerland_immoscout24/master/data/lausanne_zip_rental.json";
    var opt = {"actions": false,
               "padding": {left: 5, top: 5, right: 5, bottom: 5}};
  	vegaEmbed('#vis_rent_per_room', spec, opt).catch(console.warn);
  </script>
</body>
</html>

Head to [this notebook](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/rental_analysis_lausanne.ipynb) for the python code that I summarized above. 


# Web Scraping and Visualization of Rental Data in Switzerland with Python

In this article, I explain how you can:

* Get the rental data by web scraping and converting it to Pandas data frame
* Use the Geopandas library to get convert map [shapefiles](https://en.wikipedia.org/wiki/Shapefile) to Geopandas data frames and Geojson maps 
* Make interactive Choropleth maps embedded with rental data



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

4. Then I use ``regex`` in [this script](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/SingleAptListing_to_table.py) to extract the information such as number of rooms, area, etc. from the string above and convert it to Pandas dataframe. This step is a bit tricky as not all single apartment listings look as nice as the one above. So the script has to cover all different cases without outputing wrong data. 

5. Finally, I concatenate all single apartment data frames to a Pandas data frame which will be convenient for data analysis. 

   I will be completing this article, but for the moment you can see the [this notebook](https://github.com/hamedrazavi/rental_analysis_switzerland_immoscout24/blob/master/src/rental_analysis_lausanne.ipynb) for the analysis and resulting Choropleth rental maps. 


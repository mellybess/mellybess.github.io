---
layout: post
title:  "Getting some data"
date:   2017-01-23 13:05:46 -0800
#Scategories: jekyll update
---

For my first project, my goal was to get data on a known list of famous or infamous murderers in order to compare them to a the subjects of my favorite murder podcast, "My Favorite Murder" and map the subjects of those episodes. For a couple reasons, I changed my goal for this as I went through this exercise. I'll circle back to it once I have more data.

As I looked for a good source of known, infamous/famous murders, I remembered Murderperdia, a community site with a compendium of murderers from all over the worlds. Just like it sounds, Murderperdia is the wikipedia for murders.

Below are the steps I took to create a database of murderers and the location of their crimes. For this project I used BeautifulSoup for webscraping, Pandas for organizing and cleaning the data, and the Google Geocoding API for getting the coodinates of the murders.

As a note: I'm using the term murderers since it is convenient for the project. The data on Murderpedia can only be as accurate as the justice systems that prosecuted these individuals, so it is likely that some of them did not commit the crimes they were convicted of. 

Step 0: Import necessary libraries

I knew I'd need BeautifulSoup, Pandas, and Requests from the outset. Along the way, I also needed to import Re for regular expressions and String.

```python
from bs4 import BeautifulSoup
import pandas as pd
import requests
import re
import string
```

Step 1: Scraping for initial list of murderers

Murderpedia divided up the murderers by gender, and then by either last name or location, alphabetically. I decided it would be better to go through the alphabet than to go through a custom list of countries and states.

So my first step was to build the URLs for each of the pages I'd need to scrape to get the full list of murderer names, and to create an empty list to keep them in. With that I could use requests to get the html for each page:

```python
sexes = ["male", "female"]
murderers = []
letters = list(string.ascii_uppercase)
for sex in sexes:
   base = "http://murderpedia.org/" + sex
   for l in letters:
       url = base + "." + l + "/index." + l + ".htm"
       r = requests.get(url)
       c = r.content
```
Beautiful is an awesome library for webscraping. For well organized websites, it makes grabbing data based on elements and attributes simple and intuitive. Unfortunately for me, this simplicity requires that the website you are scraping is well organized and follows modern best practices for markup. Murderperdia, while a rich source of interesting information, does not fit that description.

When I inspected the html for Murderpedia on my browser, I found a ton of nested tables. There were no semantically useful classnames for these tables either, just undifferentiated table inside of table inside of table.

The data that I needed was inside of a table where each row had five cells:
- Cell[0] contained some media or an image, if it existed. I did not need that cell.
- Cell[1] had the murderer's name, and a relative URL to his or her individual page, both of which I needed.
- Cell[2] had the date, not formatted uniformly. For some it was a single date, for others (like serials killers) It would be a range of days, month or years when they were active. I needed this and could fiddle with it later.
- Cell[3] contained the number of victims. Again this one wasn't uniform, mostly it was a single integer, but for some it was a range like "8-200" if the body count wasn't certain...and unnervingly for some it was like "163+"
- Cell[4] contained the location of the murder. Unfortunately it was not very specific, only the country or state of the event. The individual murderder page contained a more precise location, which I could get later.

There was no way to separate out this useful table from the ones around it. Fortunately if I used BeautifulSoup to grab all of the `tr` elements from each page, the first row I was interested in was the 14th row on each page after all of the empty rows (why?) were removed. Some uniformity after all!

Once I had all of my useful rows, I could find each `td` I needed and, which a few tweeks to get rid of whitespace and such, I could create a dictionary with the information I needed and add it to my list of murderers.

```python
       soup = BeautifulSoup(c, "html.parser")
              allrows = soup.findAll("tr")
              rows = []
              for row in allrows:
                  if row.text.strip() != "":
                      rows.append(row)
              rows=rows[13:]
              for row in rows:
                  data = row.findAll("td")
                  links  = row.findAll("a")
                  if len(data) == 5:
                      dic = {}
                      dic["Name"] = data[1].text.strip().replace("\n","").replace("\t"," ")
                      dic["Date"] = data[2].text.strip().replace("\n","")
                      dic["Number of Victims"] = data[3].text.strip().replace("\n","")
                      dic["Location"] = data[4].text.strip().replace("\n","")
                      for x in links:
                          if re.match(r"%s/.*" % l, x["href"], re.I):
                              dic["Murderpedia URL"] = "http://murderpedia.org/" +sex +"."+ l+"/" +x["href"]
                      murderers.append(dic)
```

And then, there it was, a list with the basics information I needed. Phew!

From here I made a dataframe of the murderers, removed duplicate names, and saved it to a csv so that I wouldn't have to rescrape the whole site again.

```python
df1 = pd.DataFrame(murderers)
df1.drop_duplicates("Name")
df1.to_csv("all_murderers_raw.csv")
```

Step 2: Refining the location

The location provided in step 1 was helpful, but not specific enough for me to make maps with. However, if I went to the individual murderer page there was usually a list of information included a more specific location. Sometimes this would be a city or county if in the US, or a province/municipality/city if in another country. Sometimes there was no more specific data.

For this exercise I needed the correct URL for the individual page. I had gotten the URL in the previous step, but for some reason 1165 of these were `Nan`. This was due to an inconsistency in the URLs. If I were to redo the scraper, I would account for the inconsistency. For now though I just got rid of these rows. So sad, but this still left me with 3928	rows.

```python
df1 = df1.dropna(axis=0, how='any')
```

Next, I wrote a function to go to the individual page URL and get the location information for the murders from that page. I wanted to do this in batches, so that could go through my list of 4000 names more slowly, so as not to overwhelm Murderperdia.

```python
def get_location(data, rows):
    for item in rows:
        r = requests.get(data.loc[item,"Murderpedia URL"])
        c = r.content
        soup = BeautifulSoup(c, 'html.parser')
        x = soup.findAll("b")
        y = data.loc[item,"Location"].split()
        for i in x:
            if y[-1] in i.text:
                data.loc[item,"Location"] = i.text.strip().replace("\n","").replace("\t","")
                break
```

The vast majority of the results I got back from this were good. The problematic results were either obviously not locations ("Murder.uk.co"), partial locations like ", USA" or ambiguous but correct locations like "In the Sea." I'm sure it was in the sea, but that doesn't help me.

I exported this data as a csv and spent about an hour manually cleaning this in order to make sure the data was as good as I could get it. 

Step 3: Geocoding

Once that was done, I used the [`python_batch_geocode`](https://github.com/shanealynn/python_batch_geocode) script which took in my csv as input and output a new file with the location coordinated from Google's Geocoding API. I chose this option since me dataset was larger than the 2500 call limit that google has in place. It worked great!

So, at this point I had two files with all the information I needed, the name of the murderer, when they comitted the crime, the number of victims, and the location  and coordinates of the murders.

  




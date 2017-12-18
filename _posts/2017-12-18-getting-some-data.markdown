---
layout: post
title:  "Getting started"
date:   2017-12-06 13:05:46 -0800
#Scategories: jekyll update
---

##Scraping murder data

My next project is building a map with a bunch of known murder cases, and annotating that with information and resources. The problem I've been working on for the past week or so is how to get that data. My initial thought was to use the cases in episodes from my favorite murders podcast—"My Favorite Murder"—as the first set of data for the map. However, the only way I've found to get the cases covered in each episode is from the descriptions. The descriptions are usually a few sentences, and the cases can be named for the murderer (i.e. "Albert Fish" or "Son of Sam"), the victim like "Jonbenet Ramsey", or some other colloquial name for the case like "The Preppy Murders". 

So for now, in order to be able to extract the cases out of description I have to already have a known list of murderers and cases to compare it against. I found are some interesting datasets from the FBI and other crime reporting organizations, however most of them were either from too narrow a time period (1964-2015) or they didn't include the murderer or victims names. Interesting, but not useful to me. 

Then I remembered [Murderpedia](http://murderpedia.org/). Murderpedia is an awesome resource with over 5000 murder cases including names, dates, locations, details about each case, and tons of articles and information about each one. 

Disclaimer: Of course any resource on murder can only be as accurate as the data and sentences it is based on are. And with flawed criminal justice systems all over the world, I'm sure some of the people on Murderpedia are not guilty of committing murder. But for simplicity's sake, I'll refer to them as murderers...since adding clarification each time will be cumbersome.

I decided to scrape Murderpedia using [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/). I quickly flound that although Murderpedia's site has a ton of awesome content, their website is a bunch of nested tables with inline styling...which made it a little difficult for me to figure out how to extract the information I needed at first. But through a bunch of trial and error, I finally started getting somewhere. Here's how:

### First attempt

Murderpedia doesn't have just one page with all the murders listed on it. Instead there are two main sections: one for male murderers and one for female murderers. Inside each section, the murderers are either divided into pages based on the state or country of the murder OR by the first letter of their last name. Since the geographical divisions were a little fuzzy (there's a category called "Several States"), I decided to go with the alphabetical division.

First I imported the various libraries I'd need:

```
from bs4 import BeautifulSoup
import pandas as pd
import requests
import string
```

Then I had to write some loops to create the urls for to get each page, which has a list a murderers of one gender whose last names start with teh same letter:

```
sexes = ["male", "female"]
murderers = []
letters = list(string.ascii_uppercase)
for sex in sexes:
    base = "http://murderpedia.org/" + sex
    for l in letters:
        url = base + "." + l + "/index." + l + ".htm"
 ```
 
Then using `requests` and Beautiful Soup, I had to get each murder from each page. This was tricky, since there were any classes associated with the table or cells that contained the murderer's names. And there wasn't a uniform format for what element inside of the `<td>` elements contained the murderers' names. Some cells contained a `<p>` element, some had no `<p>`s but multiple `<font>` elements. After playing around for a while, I noticed that one thing each cell with a name has an `<a>` tag with an `"href"` that began with the first letter of the murderer's last name. 

So within the for loop the iterated though each letter of the alphabet I added a section that would find all `<a>` tags and add their text to my `murderers` list if the page they linked to started with that letter.

```
        r = requests.get(url)
        c = r.content
        soup = BeautifulSoup(c, "html.parser")
        links = soup.findAll('a')
        for x in links:
            if re.match(r"%s/.*" % l, x["href"], re.I):
                murderers.append(" ".join(x.text.split()))
```


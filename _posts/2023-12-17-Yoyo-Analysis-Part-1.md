---
layout: post
title: "Yo Data, Yo Analysis!"
subtitle: A Data Analysis of Yoyos! Part 1: Data Collection
gh-repo: kadenf17/kadenf17.github.io
tags: [Data Collect, Web Scraping, Web Crawling, Yoyo, Data Science]
comments: true
thumbnail-img: "assets/img/Space_Yoyo.jpg"
cover-img: "assets/img/Cover_Yoyo.jpg"
---

## Introduction:

I've spent the last 14 years as a dedicated yoyo enthusiast. I learned to throw as a little kid, and I have kept up with the hobby through the years. Currently, I’m serving as the president of the Yo-Yo Club at BYU. My passion extends beyond personal enjoyment; it's about sharing the thrill with others and contributing to the community.

I have been performing for larger crowds for a couple of years now, and it’s always so fun to see the look on people’s faces when I do crazy tricks! After shows, I usually have a lot of people come up and want to learn how to yoyo. I have a big collection of yoyos, and I always get questions like: What’s your best yoyo? Why are there so many different models? What’s the best yoyo to start out with?

To answer some of these questions, I initiated a data analysis project centered on yoyos. I collected information on over 500 yoyos and looked at different specs including price, material, weight, bearing type, and several other descriptors. The goal is to uncover trends and correlations that can offer insights into the diverse world of yoyos.

This post provides a sneak peek into the initial steps of my data-driven exploration, shedding light on the groundwork laid for deeper insights into the fascinating dynamics of yoyo specs. For the final analysis see part 2.

## The Beginning:

When I started this project, I knew I was going to have to create my own dataset. In my 14 years of purchasing yoyos, I had never come across a dataset of different yoyos and their specs. My best option was to pull information about the yoyos from an online store that describes and sells many different yoyos. My personal favorite is yoyoexpert.com, a website that sells yoyos from many different brands and is the largest online yoyo retailer I know of.

## Creeping, Crawling, and Scraping:

I first had to determine how to gather the data. I first checked to see if there was an API that I could use to get information. Unfortunately, there was no such thing. I then decided to scrape data using the Python library Beautiful Soup.

I was hoping there would be a list or a table with information on different yoyos, that would make things easy. Unfortunately, things were not easy. I realized that I needed to build a web crawler that would visit the product listing website for each yoyo and then scrape the information.

Ethical data scraping is an important consideration of mine. To make sure I was following best practices, I looked at yoyoexperts robots.txt to see if there were any regulations I should be informed of. I saw this:

~~~
User-agent:  *
Crawl-delay:  30
~~~

This crawl delay meant that I had to wait 30 seconds between each webpage while I was crawling. 30 seconds doesn’t seem like much until you have to wait 30 seconds 500 times over. The total wait time for the crawler to collect the raw data (Not including the catalog pages) was over 4 hours. This made it very tedious to make sure my crawler was working, but I got it working in the end.

## How it works:

My crawler takes advantage of lists and HTML parsing to collect the different product pages. I first initialize the crawler at the starting point. The starting point I used was a catalog of yoyos in the string trick collection. This collection is the biggest collection that includes the main type of yoyos. There are specialty yoyos that we will not be including in our data.

My code starts off with initializing necessary packages:

~~~
import requests
from bs4 import BeautifulSoup
import csv
import time
~~~

Then, I set my initial parameters:

~~~
starting_point = "https://shop.yoyoexpert.com/collections/1a-string-trick-yo-yos?view=listall"
base_site = "https://shop.yoyoexpert.com"
out_csv_file = "yoyo_raw_data.csv"
in_csv_file = "visited_urls.csv"
products = []
urls = [starting_point]
errors = 0
~~~

The urls variable is a list that keeps track of what URLs the scraper is going to visit. This includes other collection pages and product pages. 

I use two other files for my scraper. If these files are not already present, it creates them. The first is the out_csv_file. When my crawler/scraper finds information about a yoyo, it adds the data to this CSV file. The second file is the in_csv_file. This file is used to keep track of what URLs have already been visited. If the URL is a product page, it keeps track of it in this CSV. Because the file is read in when the program first runs, you can run the scraper again and again and it will only add new information it finds, it will not scrape the same URL twice.

My next bit of code creates a list of visited URLs:

~~~
try:
    with open(in_csv_file, mode='r', encoding='utf-8') as txt_file:
        visited_urls = [line.strip() for line in txt_file.readlines() if line.strip() != starting_point]
except FileNotFoundError:
    visited_urls = []
~~~

Then, I have a while loop that will continue to look at pages until the list of URLs to visit is 0. If a new yoyo is found, it gathers and saves the information in a dictionary. The keys of the dictionary are the different types of information I want to gather, and the values are the bits of information I find. If one of the specs is missing, I leave it as NA. I then write the write the new line of information to the out_csv_file. Here’s the code for the while loop:

~~~
while len(urls) > 0:
    current_url = urls.pop()
    print(f"Now attempting to visit: {current_url}")
    
    response = requests.get(current_url)
    soup = BeautifulSoup(response.content, "html.parser")
    
    # mark the current URL as visited
    visited_urls.append(current_url)

    # find other urls
    link_elements = soup.select("a[href]")
    
    for link_element in link_elements:
        url = link_element["href"]
        if "/collections/1a-string-trick-yo-yos?page=" in url:
            if base_site not in url:
                url = base_site+url
            if url not in visited_urls and url not in urls:
                urls.insert(0, url)

        if "/collections/1a-string-trick-yo-yos/products" in url:
            if base_site not in url:
                url = base_site+url
            if url not in visited_urls and url not in urls:
                urls.append(url)


    # if current_url is product page
    if "/products" in current_url:
        print('I found a new yoyo!')
        product = {}
        key_list = ["url", "name", "price", "brand", "diameter", "width", "gap width", "weight", "bearing size", "response", "material", "designed in", "made in", "machined in", "released"]

        for key in key_list:
            product[key] = "NA" # Not all productes have all specs, so initialize them with NAs

        product["url"] = current_url

        try:
            product["name"] = soup.select_one(".product-title").text.strip()
            print(f"Name: {soup.select_one('.product-title').text.strip()}")
        except:
            print("Name: Not found")
        
        try:
            product["price"] = soup.select_one(".hidden-xs.prices span.price").text.strip()
            print(f"Price: {soup.select_one('.hidden-xs.prices span.price').text.strip()}")
        except:
            print("Price: Not found")
        
        try:
            product["brand"] = soup.select_one(".product-vendor").text.strip()
            print(f"Brand: {soup.select_one('.product-vendor').text.strip()}")
        except:
            print("Brand: Not found")

        
        try:    # Extract table data
            table_data = {}
            for row in soup.select(".stats tr"):
                cells = row.select("td")
                if len(cells) == 2:
                    key = cells[0].text.strip().replace(":", "").lower().replace("release date", "released")
                    value = cells[1].text.strip().replace(",", "").replace(" ","").replace("\n","").replace("\r","")
                    if key in key_list:
                        table_data[key] = value
                    else:
                        print(f"   **Warning: {key} is not found in key list.")
                        errors += 1 # This error occurs if there is a new key that isn't in the key list

            # Add table data to product
            product.update(table_data)
        except:
            print("   ** Error Scraping Spec Table")
            errors += 1

        products.append(product)

        # Write to the CSV (adds lines, will not replace old lines)
        with open(out_csv_file, mode='a', newline='', encoding='utf-8') as file:
            fieldnames = key_list
            writer = csv.DictWriter(file, fieldnames=fieldnames)
            if file.tell() == 0:
                writer.writeheader()
            writer.writerow(product)

    else: print("No product found on this page.")

    # Save the visted urls to a csv (in_csv_file)
    if "/collections/1a-string-trick-yo-yos?page=" not in current_url and current_url != starting_point:
        with open(in_csv_file, mode='a', encoding='utf-8') as txt_file:
            txt_file.write(current_url + '\n')

    print(f"I have visited {len(visited_urls)-sv_url_len} new urls and {len(visited_urls)} urls total.")
    print(f"There are currently {len(urls)} urls in my queue to vist.")
    print(f"So far I have found {len(products)} new yoyos this run.")
    print(f"** {errors} errors found this run.")

    # Following their robots.txt we need to sleep 30 seconds

    print("Sleeping 30 sec (to comply with yoyoexpert's robots.txt)")
    time.sleep(30)
    print("\n")
~~~

## Cleaning the data:

The raw data I got was by no means perfect. It needed data cleaning. The raw data looked something like this line:

~~~
https://shop.yoyoexpert.com/collections/1a-string-trick-yo-yos/products/roppo-yoyo-by-shuriken,Roppo,$ 33.00,Shuriken - Stable and Powerful V-Shaped YoYo!,56.80 mm / 2.24 inches,46.80 mm / 1.84 inches,4.50 mm / 0.18 inches,65.3 grams,Size C (.250 x .500 x .187),"""Slim Pad"" Size 19mm OD",6061 Aluminum,Hong Kong,NA,NA,June 2023
~~~

I needed to make changes. Some changes include: removing the URL, changing the price, weight, width, diameter, etc. to numbers, coercing the bearing size and response pads to categorical groups, and other changes.

I used a Jupiter notebook to help me make these changes. Sometimes I had to use lots of string manipulation to help me make the data look right. All of the changes were pretty straightforward. I’m including one example below, but feel free to check out my GitHub repo with all the code for my data cleaning.

Changing price and weight to numerical values:

~~~
df['price'] = pd.to_numeric(df['price'].str.replace('$', '').str.replace(' ', ''))
df['weight'] = pd.to_numeric(df['weight'].str.replace('grams', '').str.replace(' ', '').str.replace('g',''))
~~~

After all this cleaning, my dataset was ready for analysis. The final table looks something like this:

~~~
name,price,brand,weight,bearing size,response,material,designed in,made in,machined in,released,description,diameter mm,diameter in,width mm,width in,gap width mm,gap width in,bearing type,material group
Roppo,33.0,Shuriken,65.3,Size C (.250 x .500 x .187),"""Slim Pad"" Size 19mm OD",6061 Aluminum,Hong Kong,,,2023-06-01,Stable and Powerful V-Shaped YoYo!,56.8,2.24,46.8,1.84,4.5,0.18,Size C,Aluminum
Da Vinci,100.0,yoyofriends,62.6,Size C (.250 x .500 x .187),"""Slim Pad"" Size 19mm OD",7068 Aluminum w/ Stainless Steel Rings,China,China,,2023-06-01,Tomoki Toyama Signature YoYo!,55.89,2.2,44.64,1.76,4.5,0.18,Size C,Aluminum Bi-Metal
Brass Exia,70.0,MK1 YoYos,66.3,Size C (.250 x .500 x .187),"""Slim Pad"" Size 19mm OD",7068 Aluminum w/ Brass Rings,USA,China,,2023-07-01,Inner Ring Bi-Metal YoYo!,55.6,2.19,47.3,1.86,4.5,0.18,Size C,Aluminum Bi-Metal
~~~

## The End of the Beginning

I’m excited to explore questions with you like:
-	What are the most expensive yoyos (what about by brand)
-	What effect does the material have on yoyo price?
-	Do certain manufacturers use better materials in their yoyos?
-	Does one brand of yoyo stand out above the rest?

Check out my next blog post to see these questions answered! Thanks for coming on this data analysis adventure with me!

---
layout: post
title: "Automatic Download of METAR Data using Web Scraping"
subtitle: "A Method to Download METAR Data using Web Scraping"
background: '/img/posts/web-scrapping/front.png'
---

## Background
Recently, I wanted to perform an analysis of METAR data from my station WAQT. However, I discovered that the database containing METAR data was not complete, and I faced difficulties as a result. Then, I remembered that our data is actually recorded and saved on aviation.bmkg.go.id, which we can access and copy from. But, a problem arose: if I only needed data for a short time range, like one day, it was manageable. However, since my data spans one month, it became practically impossible to do it manually. So, I wondered if there was a way to automate this process. In this tutorial, I will show you how to do just that!

EXPORT_METAR is a script for exporting METAR values from a specific station automatically using Python. The basic idea is to scrape content data from the website aviation.bmkg.go.id. The output is in CSV file format. There are two main parts to this script: the "can change" part and the "can't change" part. Please do not modify the "can't change" part (marked with #).

## Website Source of Data
To access the data from aviation.bmkg, you can use the following link:
```
https://aviation.bmkg.go.id/web/metar_speci.php
```

## Import Modules
First, we need to import some modules for this tutorial:
```
#import module
import pandas as pd
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.support.ui import Select
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
```
There are some modules, and you may already be familiar with some of them.

## Define the Date and Time
We must specify our desired time frame:
```
#CAN CHANGE PART (change station, st_date (start date), time_st (time start), ed_date (end date), time_ed (time end))
station = 'waqt' #icao index station
st_date = ['12', '07', '2023'] # day, month, year  | ex  : 4 june 2023
time_st = ['00', '00'] #hour, minute    | ex : hour 00 minute 00
ed_date = ['13', '07', '2023'] #same w/st_date
time_ed = ['01', '00'] #same with time_st
url = f'https://aviation.bmkg.go.id/web/metar_speci.php?icao={station}&sa=yes&fd={st_date[0]}%2F{st_date[1]}%2F{st_date[2]}&fh={time_st[0]}&fm={time_st[1]}&ud={ed_date[0]}%2F{ed_date[1]}%2F{ed_date[2]}&uh={time_ed[0]}&um={time_ed[1]}&f=raw_format'
#CAN CHANGE PART (UNTIL THIS)
```
As described in the script, you only need to change the date, specifically the st_date for the start date and ed_date for the end date. You need to fill in these values manually. The URL will be generated based on the information provided in st_date, time_st, ed_date, and time_ed.

## Function to Count Pages
Since the aviation website splits the data into pages, we need to count how many pages there are so that we can access them:
```
def page_count(url, page=None):
    driver = webdriver.Chrome(ChromeDriverManager().install())
    driver.get(url)
    driver.implicitly_wait(3)
    checkbox = driver.find_element(By.CSS_SELECTOR, "input[name='agreement']")
    checkbox.click()
    submit_button = driver.find_element(By.CSS_SELECTOR, "input[name='submit']")
    submit_button.click()

    # Find the dropdown element
    driver.implicitly_wait(3)
    dropdown = Select(driver.find_element(By.XPATH, './/*[@id="main_page"]/p[1]/label/select'))

    # Get the total number of pages
    total_pages = len(dropdown.options)
    #print(total_pages)

    if page is not None:
        page.append(total_pages)
```

## Function for Web Scraping and Downloading Data
In this step, we create a function for web scraping and downloading data:
```
def importmetar(url, index, page=None, metar_value=None):
    driver = webdriver.Chrome(ChromeDriverManager().install())
    driver.get(url)
    driver.implicitly_wait(3)
    checkbox = driver.find_element(By.CSS_SELECTOR, "input[name='agreement']")
    checkbox.click()
    submit_button = driver.find_element(By.CSS_SELECTOR, "input[name='submit']")
    submit_button.click()

    # Find the dropdown element
    driver.implicitly_wait(3)
    dropdown = Select(driver.find_element(By.XPATH, './/*[@id="main_page"]/p[1]/label/select'))

    dropdown_xpath = './/*[@id="main_page"]/p[1]/label/select'
    dropdown.select_by_index(index)
    
    #calculate span count
    x = 1
    elements_found = True
    span_count = 0

    while elements_found:
        xpath = f'//*[@id="main_page"]/span[{x}]'
        elements = driver.find_elements(By.XPATH, xpath)
    
        if len(elements) > 0:
            span_count += 1
            x += 1
        else:
            elements_found = False

    #print(f"Total span elements found: {span_count}")

    #Export metar value
    value = []
    for i in range(1,span_count):
        element = driver.find_element(By.XPATH,  f'.//*[@id="main_page"]/span[{i}]')
        value1 = element.text
        # Split the string at the newline character
        split_data = value1.split('\n') #for only take metar data, not with the header 

        # Get the value after the newline character
        value2 = split_data[1]
        value.append(value2)

    #print(value)

    if metar_value is not None:
        metar_value.append(value)

    driver.quit()
#CAN'T CHANGE PART (UNTIL THIS)
```

We start by using the Chrome web driver to open the URL. Next, we calculate the span count, which represents the number of rows on one page of data. For each span element on the page, we extract its text value and add it to the 'metar_value' list.

## Call the Above Function for Every Page
```
pages = []
page_count(url, page=pages)

metar_value = []
for i in range(pages[0]):
    importmetar(url,i, metar_value=metar_value)
    
f_metar_val = [item for sublist in metar_value for item in sublist] #this line for change the "" "" become " " list

```

## Save the data
FInally we save the data as csv file.
```
#reverse the list
series = pd.Series(f_metar_val)
reversed_series = series[::-1]
f_metar_val_rev = reversed_series.tolist()

df = pd.DataFrame(f_metar_val_rev, columns=['METAR'])
df.to_csv('export_metar.csv', sep=',', index=False)
print ('Script is finish, check the folder for the output file')
#CAN'T CHANGE PART
```

After run the script, this will be nottification like this:
![notif image](/img/posts/web-scrapping/result1.png)

And the result of data is like this:
![result image](/img/posts/web-scrapping/result2.png)

## Complete Code
To summarize, here's the combined script:  (or youc can acces this scrip in my github code : ) 

```
#import module
import pandas as pd
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.support.ui import Select
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait

#CAN CHANGE PART (change station, st_date (start date), time_st (time start), ed_date (end date), time_ed (time end))
station = 'waqt' #icao index station
st_date = ['12', '07', '2023'] # day, month, year  | ex  : 4 june 2023
time_st = ['00', '00'] #hour, minute    | ex : hour 00 minute 00
ed_date = ['13', '07', '2023'] #same w/st_date
time_ed = ['01', '00'] #same with time_st
url = f'https://aviation.bmkg.go.id/web/metar_speci.php?icao={station}&sa=yes&fd={st_date[0]}%2F{st_date[1]}%2F{st_date[2]}&fh={time_st[0]}&fm={time_st[1]}&ud={ed_date[0]}%2F{ed_date[1]}%2F{ed_date[2]}&uh={time_ed[0]}&um={time_ed[1]}&f=raw_format'
#CAN CHANGE PART (UNTIL THIS)

# CANT'CHANGE PART (FROM HERE)
def page_count(url, page=None):
    driver = webdriver.Chrome(ChromeDriverManager().install())
    driver.get(url)
    driver.implicitly_wait(3)
    checkbox = driver.find_element(By.CSS_SELECTOR, "input[name='agreement']")
    checkbox.click()
    submit_button = driver.find_element(By.CSS_SELECTOR, "input[name='submit']")
    submit_button.click()

    # Find the dropdown element
    driver.implicitly_wait(3)
    dropdown = Select(driver.find_element(By.XPATH, './/*[@id="main_page"]/p[1]/label/select'))

    # Get the total number of pages
    total_pages = len(dropdown.options)
    #print(total_pages)

    if page is not None:
        page.append(total_pages)
    
def importmetar(url, index, page=None, metar_value=None):
    driver = webdriver.Chrome(ChromeDriverManager().install())
    driver.get(url)
    driver.implicitly_wait(3)
    checkbox = driver.find_element(By.CSS_SELECTOR, "input[name='agreement']")
    checkbox.click()
    submit_button = driver.find_element(By.CSS_SELECTOR, "input[name='submit']")
    submit_button.click()

    # Find the dropdown element
    driver.implicitly_wait(3)
    dropdown = Select(driver.find_element(By.XPATH, './/*[@id="main_page"]/p[1]/label/select'))

    dropdown_xpath = './/*[@id="main_page"]/p[1]/label/select'
    dropdown.select_by_index(index)
    
    #calculate span count
    x = 1
    elements_found = True
    span_count = 0

    while elements_found:
        xpath = f'//*[@id="main_page"]/span[{x}]'
        elements = driver.find_elements(By.XPATH, xpath)
    
        if len(elements) > 0:
            span_count += 1
            x += 1
        else:
            elements_found = False

    #print(f"Total span elements found: {span_count}")

    #Export metar value
    value = []
    for i in range(1,span_count):
        element = driver.find_element(By.XPATH,  f'.//*[@id="main_page"]/span[{i}]')
        value1 = element.text
        # Split the string at the newline character
        split_data = value1.split('\n') #for only take metar data, not with the header 

        # Get the value after the newline character
        value2 = split_data[1]
        value.append(value2)

    #print(value)

    if metar_value is not None:
        metar_value.append(value)

    driver.quit()
#CAN'T CHANGE PART (UNTIL THIS)

#CAN'T CHANGE PART
pages = []
page_count(url, page=pages)

metar_value = []
for i in range(pages[0]):
    importmetar(url,i, metar_value=metar_value)
    
f_metar_val = [item for sublist in metar_value for item in sublist]

#reverse the list
series = pd.Series(f_metar_val)
reversed_series = series[::-1]
f_metar_val_rev = reversed_series.tolist()

df = pd.DataFrame(f_metar_val_rev, columns=['METAR'])
df.to_csv('export_metar.csv', sep=',', index=False)
print ('Script is finish, check the folder for the output file')
#CAN'T CHANGE PART

```
Thank you for reading this tutorial. I hope you find it helpful. If you have any questions or need further assistance, please feel free to ask. Your feedback is valuable.



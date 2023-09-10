---
layout: post
title: "Visualize RGB Satellite Data With Python"
subtitle: "A Basic Guide to Handling Satellite Data in Python and Making Improvements"
background: '/img/posts/01.jpg'
---

## Background
This tutorial is designed to assist individuals who want to learn how to visualize data in Python, specifically using NetCDF files. In this case, we will explore how to create an RGB composite from satellite data.

## Data
The data used in this tutorial is sourced from JAXA (Japan Aerospace Exploration Agency). We accessed the data via FTP using FileZilla. To access the data, please visit the following website:
```
https://www.eorc.jaxa.jp/ptree/index.html
```

## Installing Packages
First, we need to install some packages for this tutorial.
```
from netCDF4 import Dataset
import numpy as np
import matplotlib.pyplot as plt
import imageio
import matplotlib.image as im
from PIL import Image
from mpl_toolkits.basemap import Basemap
```
There are some modules, and you may already be familiar with some of them.

## Open Data
We use the NetCDF4 module with the Dataset function to open our data.

```
PATH = '/media/evan/A8242CC6242C9978/04092023/'
data = PATH+'NC_H08_20150825_0600_R21_FLDK.06001_06001.nc'
ds = Dataset(data,'r')
```
First, we need to specify the path to our data. This step is necessary not only for Python to find our data but also to specify the folder path for saving our output. We combine PATH with the filename of our data and assign it as 'data'. Finally, we open the data using the Dataset function and assign it to ds. The 'r' in the ds line is used to make our file readable in Python. In some cases, it may not be necessary, but in this tutorial, I'll use it.

## Assign The Variable
Now, we must assign the variables from our data so that we can process them.
```
b1 = ds.variables['albedo_01'][:]
b2 = ds.variables['albedo_02'][:]
b3 = ds.variables['albedo_03'][:]
lons = ds.variables['longitude'][:]
lats = ds.variables['latitude'][:]
```
In this example, I've taken band1, band2, and band3 from our data to create an RGB True Color image. In our JAXA Himawari data, the bands are named 'albedo_xx' where xx is the band number. (Note: Bands from 8 to the last band are named 'tbb_xx'). Don't forget to also assign the 'lons' and 'lats' variables. The symbol '[:]' at the end of each line indicates that we want to select 'all' of the values. In some cases, we can specify a range instead.

## Assigning Red, Green, and Blue Bands
In this step, we assign our variables to new variables called red, green, and blue.
```
red = b3
green = b2
blue = b1

def norm(band):
    band_min, band_max = np.nanpercentile(band,2), np.nanpercentile(band,98)
    return ((band-band_min)/(band_max-band_min))

r = norm(red.astype(float))
g = norm(green.astype(float))
b = norm(blue.astype(float))
```
After that, we perform an additional step known as normalizing the values of each band. This step is considered 'extra' because the script will still function correctly without it, but the output may not match our expectations (it will appear slightly darker). Therefore, we perform this step using the 'norm' function. The resulting normalized data is assigned to the 'r', 'g', and 'b' variables. For more details about this normalization process, you can refer to this link: 
```
https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)
```

## Stacking the Data (Creating RGB)
In Python, to create an RGB composite, we use the 'dstack' function from numpy.
```
rgb = np.dstack((r,g,b))
extent = [80, 200, -60, 60]
```
After performing the 'dstack' operation, we assign that variable and convert it to 'uint8' data type for proper display.

## Displaying the Basemap
For an aesthetically pleasing plot, we display the basemap.
```
below,above,left,right = -10, 10, 100, 130 #AOI
#below,above,left,right = -1, 4, 115, 120 #AOI
m = Basemap(projection='cyl',resolution='h',llcrnrlat=below,llcrnrlon=left,urcrnrlat=above,urcrnrlon=right)
m.drawcoastlines(color='white')
#m.drawparallels(np.arange(-90,90,3),labels=[1,0,0,1],dashes = [5,5], fontsize=8)
#m.drawmeridians(np.arange(-180,180,3),labels=[1,0,0,1],dashes = [5,5],fontsize=8)
```
First, we specify our Area of Interest (AOI). It's worth noting that if our plotting data is smaller than the AOI of our basemap, it's still acceptable as long as the plotting AOI is within the basemap's AOI. In the last two lines, I've commented out the latitude and longitude grid lines because I only want to focus on the RGB imagery. Feel free to uncomment those lines if you need them.

## Plotting
The final step is the plotting process.
```
plt.imshow(rgb, extent=extent)
plt.show()
```
## Output
![rgb image](/img/posts/visualize-rgb/truecolor.png)

## Building a Function for Performing RGB Composite Automatically
As an additional part of this tutorial, I've created a more concise function. The script above can become quite lengthy and less effective if you want to use it for other files or perform different types of RGB composites, such as false color. So, I've built this function (with a description similar to the one above).
```
# Program for plotting RGB composite in Himawari
# Note : Himawari data is from https://www.eorc.jaxa.jp/ptree/index.html

from netCDF4 import Dataset
import numpy as np
import matplotlib.pyplot as plt
import imageio
import matplotlib.image as im
from PIL import Image
from mpl_toolkits.basemap import Basemap
import pylab

PATH = input("Input PATH (ex : /home/evan/himawari/ ) :")
print("You entered:", PATH)
data = input("Input filename (ex : NC_H09_20230401_0610_R21_FLDK.02401_02401.nc ):")
print("You entered:", data)
data = PATH+data
ds = Dataset(data, 'r')
band1 = int(input("Input Band1 (ex: 3 )"))
band2 = int(input("Input Band2 (ex: 2 )"))
band3 = int(input("Input Band3 (ex: 1 )"))
title = input("Title : ")
below = int(input("Inpute coordinate below :"))
above = int(input("Input coordinate above :"))
left = int(input("Input coordinate left :"))
right = int(input("Input coordinate right :"))

def rgbcomposite(band1, band2, band3, below, above, left, right):
    i = [1,2,3,4,5,6]
    j = [7,8,9,10,11,12,13,14,15,16]
    if band1 in i:
        b1 = ds.variables['albedo_{:02d}'.format(band1)][:]
    if band1 in j:
        b1 = ds.variables['tbb_{:02d}'.format(band1)][:]
    if band2 in i:
        b2 = ds.variables['albedo_{:02d}'.format(band2)][:]
    if band2 in j:
        b2 = ds.variables['tbb_{:02d}'.format(band2)][:]
    if band3 in i:
        b3 = ds.variables['albedo_{:02d}'.format(band3)][:]
    if band3 in j:
        b3 = ds.variables['tbb_{:02d}'.format(band3)][:]
    lons = ds.variables['longitude'][:]
    lats = ds.variables['latitude'][:]

    #RGB Composition (for true color; BAND 3,2,1)
    red = b1
    green = b2
    blue = b3

    def norm(band):
        band_min, band_max = np.nanpercentile(band,2), np.nanpercentile(band,98)
        return ((band-band_min)/(band_max-band_min))

    r = norm(red.astype(float))
    g = norm(green.astype(float))
    b = norm(blue.astype(float))

    rgb = np.dstack((r,g,b))
    extent = [80, 200, -60, 60]

    m = Basemap(projection='cyl',resolution='h',llcrnrlat=below,llcrnrlon=left,urcrnrlat=above,urcrnrlon=right)
    m.drawcoastlines(color='white')
    #m.drawparallels(np.arange(-90,90,3),labels=[1,0,0,1],dashes = [5,5], fontsize=8)
    #m.drawmeridians(np.arange(-180,180,3),labels=[1,0,0,1],dashes = [5,5],fontsize=8)

    plt.imshow(rgb, extent=extent)
    plt.show()

# If you don't want to use input manually, you can directly call the function like this:
rgbcomposite(3, 2, 1, -10, 10, 100, 130)
```
Thank you for reading this tutorial. I hope you find it helpful. If you have any questions or need further assistance, please feel free to ask. Your feedback is valuable.



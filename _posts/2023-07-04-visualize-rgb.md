---
layout: post
title: "Visualize RGB Satelite With Python"
subtitle: "Basic for open satelite data in python and how to do some improvement in that data."
background: '/img/posts/01.jpg'
---

## Background
This tutorial was made with purpose to help some people that want to learn about visualize data in python (specific in netcdf file). In this case, we will take a look how to do RGB composite in satelite data.

## Data
For data that used in this tutorial is from jaxa. We use filezilla to acces the data via ftp jaxa.  Get the acces data via this site:
```
https://www.eorc.jaxa.jp/ptree/index.html
```

## Installing Packages
First step, we need to install some packages that we need in this tutorial.
```
from netCDF4 import Dataset
import numpy as np
import matplotlib.pyplot as plt
import imageio
import matplotlib.image as im
from PIL import Image
from mpl_toolkits.basemap import Basemap
import pylab
```

There are some module that maybe some of them is you guys already familiar with that.

## Open Data
We use Module NetCDF4 with Dataset Function to open our data.
```
PATH = '/home/evan/himawari/'
data = PATH+'NC_H09_20230401_0610_R21_FLDK.02401_02401.nc'
ds = Dataset(data,'r')
```
First we need write our path data. This step not only for python can find our data, but also to specify path folder to save our output. Combine PATH and name file of pur data, assign as 'data'. Last, open data with Dataset and assign as ds (The 'r' in ds line is for make our file become readable in python, in some case it is not really needed, but in this tutorial i'll use that)

## Assign The Variable
Now we must assign the variable of our data, so we can process that.
```
b1 = ds.variables['albedo_01'][:]
b2 = ds.variables['albedo_02'][:]
b3 = ds.variables['albedo_03'][:]
lons = ds.variables['longitude'][:]
lats = ds.variables['latitude'][:]
```
In this example, i take band1, band2, and band3 from our data to make RGB True Color. In our data from jaxa himawari, the band1 until band6 are name by 'albedo_xx' which is xx is the number of band (note: for band8 until last band are name by 'tbb_xx'). Don't forget to also assign the lons and lats variable. By the way, the symbol [:] in the end of every line is mean that we want to take 'all' of that value (in some case we can seleect or specif the range).

## Assign Red, Green, and Blue Band
In this step, we assign our variable to new variable called red, gree, and blue.
```
red = b3
green = b2
blue = b1

def norm(band):
    band_min, band_max = np.nanmin(band), np.nanmax(band)
    return ((band-band_min)/(band_max-band_min))

r = norm(red.astype(float))
g = norm(green.astype(float))
b = norm(blue.astype(float))
```
After that we do some extra step, called by normalization value of each band. This step is called 'extra' because even though we don't do that, the script is still work properly. But, the output is not realy like we wanted (the output will displaye a litle bit darker than we expect). So, we do this step with use norm function. The output data after get normalize is assign as r,g, and b variales. For more detail about that function, see this:
```
https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)
```

## Stack the Data (Make RGB)
In python, for make rgb composite, we use 'dstack' function from numpy.
```
rgb = np.dstack((r,g,b)) 
rgb_image = (rgb*255).astype('uint8')
imageio.imwrite(PATH+'rgbtruecolor1.tif', rgb_image)
```
after we perfrom dstack, we assign that variable to astype(uint8). Maybe some of you ask, why we apply the rgb with 255?, the answer is to make our data come back like usual. Because after perform norm function, the range value of our data become 0 until 1. In other words, we just need the norm function to apply in dstack function. Finally, we connvert our rgb_image to tiff file (why? this because tif file is more likely easy to read and plot, more detaill can see in step after basemap)

## Display Basemap
For nice plotting, we display the basemap.
```
below,above,left,right = -5, 5, 110, 120 #Area Of Interest
m = Basemap(projection='cyl',llcrnrlat=below,llcrnrlon=left,urcrnrlat=above,urcrnrlon=right,resolution='h',)
xv, yv = pylab.meshgrid(lons, lats) #using pylab for meshgrid
lon, lat = m(xv,yv)
paralles = pylab.arange(-90.0,90.0,2)
meridians = pylab.arange(0.,360.,2)
m.drawcoastlines(color='white')


#m.fillcontinents(color='coral',lake_color='aqua') #for this line, if u want use that, remove the '#' symbol
#m.drawparallels(paralles,labels=[1,0,0,1],dashes = [5,5], fontsize=8)
#m.drawmeridians(meridians,labels=[1,0,0,1],dashes = [5,5],fontsize=8)
```
First we specify our AOI (Area Of Interest), by the way, if in our plotting data is not as large aoi our basemap, it;s okay, as long as the aoi plotting is include in aoi basemap. In 3 last line i'm not use that, because i only want to focus on rgb imagery (feel free if you want use that)

## Open TIF File
Now, we come to important part for introduction plotting. Finally, we open our output tif file with matplotlib.image (im).
```
file = PATH+'rgbtruecolor1.tif'
img = im.imread(file)
```

## Extra Step Before Plotting
Our tif file don't have coordinate variable like lon and lat. So, before perform plotting, we should assign the coordinate in that data.
```
r = m.transform_scalar(np.flipud(img[:,:,0]),lons,lats[::-1],len(lons),len(lats))
g = m.transform_scalar(np.flipud(img[:,:,1]),lons,lats[::-1],len(lons),len(lats))
b = m.transform_scalar(np.flipud(img[:,:,2]),lons,lats[::-1],len(lons),len(lats))
```
Tbh, the line seems really hard to understand, but actually no. This is just ordinary simple line. So we assign back the r, g, and b variable from our tiff data, and add lon and lats in that. Let's take a look one example, r variable lines.
```
r = m.transform_scalar(np.flipud(img[:,:,0]),lons,lats[::-1],len(lons),len(lats))
```
The line we provided appears to be using a function called transform_scalar to transform a scalar data array img[:,:,0] onto a new grid defined by longitude and latitude values.In summary, this line of code takes a scalar data array (img[:,:,0]), flips it vertically, and then transforms it onto a new grid defined by longitude (lons) and latitude (lats) values. The transformed data is stored in the variable r.

## Stack Again and Convert To Array
Now, it's litle bit different, which we stack the r, g, and b variables from our tiff data.
The additional step is we convert our data after get stack to array data (so we can plot that).
```
rgb = np.array(np.dstack((r,g,b)),dtype='uint8')
```

## Plotting
Final step is plotting with imshow from basemap (why don't use imshow from matplotlib. the reason is to get smoother image because the basemap do interpolation in that).
```
m.imshow((rgb),interpolation='nearest')
plt.title('True Color RGB', fontsize=14)
```
## Output
![rgb image](/img/posts/visualize-rgb/true_color.png)

## Built Function For Perform RGB Composite Automatically
This is a extra part from this tutorial. It seems that our script above is too long and not effective if we want use the other file or perform other rgb, example false color. So, i bulit this one (the description is same with above).
```
#programme for plotting rgb composite in himawari
#note : himawari data is from https://www.eorc.jaxa.jp/ptree/index.html

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

    red = b1
    green = b2
    blue = b3

    def norm(band):
        band_min, band_max = np.nanmin(band), np.nanmax(band)
        return ((band-band_min)/(band_max-band_min))
    
    r = norm(red.astype(float))
    g = norm(green.astype(float))
    b = norm(blue.astype(float))
    rgb = np.dstack((r,g,b))
    rgb_image = (rgb*255).astype('uint8')
    imageio.imwrite(PATH+'rgb.tif', rgb_image)
    
    m = Basemap(projection='cyl',llcrnrlat=below,llcrnrlon=left,urcrnrlat=above,urcrnrlon=right,resolution='h',)
    xv, yv = pylab.meshgrid(lons, lats)
    lon, lat = m(xv,yv)
    paralles = pylab.arange(-90.0,90.0,2)
    meridians = pylab.arange(0.,360.,2)
    m.drawcoastlines(color='white')
    #m.fillcontinents(color='coral',lake_color='aqua')
    # #m.drawparallels(paralles,labels=[1,0,0,1],dashes = [5,5], fontsize=8)
    # #m.drawmeridians(meridians,labels=[1,0,0,1],dashes = [5,5],fontsize=8)
    file = PATH+'rgb.tif'
    img = im.imread(file)
    
    r = m.transform_scalar(np.flipud(img[:,:,0]),lons,lats[::-1],len(lons),len(lats))
    g = m.transform_scalar(np.flipud(img[:,:,1]),lons,lats[::-1],len(lons),len(lats))
    b = m.transform_scalar(np.flipud(img[:,:,2]),lons,lats[::-1],len(lons),len(lats))
    rgb = np.array(np.dstack((r,g,b)),dtype='uint8')
    
    plot = m.imshow((rgb),interpolation='nearest')
    plt.title(title, fontsize=14)
    return plot

display = rgbcomposite(band1,band2,band3, below, above, left, right)
```
Thank you, hope that's help.




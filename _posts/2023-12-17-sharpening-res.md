---
layout: post
title: "Enhancing Satellite Himawari's Spatial Resolution"
subtitle: "Improving data resolution from 2 km (or 1 km) to 0.5 km"
background: '/img/posts/enhancing-res/pic2.png'
---

## Background
Himawari satellite data is incredibly useful, but there's a catch—the spatial resolution isn't great, mainly at a 2 km band. This method is here to fix that issue, making the data even more widely useful.


## Reference
This method, known as Additive Template Sharpening (ATS), is a suitable sharpening technique designed for Himawari 8/9, developed by Yamazaki (2021). For detailed information, refer to the paper:
```
https://www.jstage.jst.go.jp/article/sola/17/0/17_2021-039/_article/-char/en
```

Additive Template Sharpening (ATS) boosts image quality by blending finer details from a high-resolution Band 3 into lower-resolution visible or shortwave infrared images of Himawari-8/9. Using this method, the unique pixel layout of Himawari-8/9 adds precise details, improving image clarity. Compared to other methods, ATS enhances Bands 1-6, resulting in clearer images for meteorological study and higher-quality true-color images for public viewing.

## Data
The data used in this tutorial is sourced from JAXA (Japan Aerospace Exploration Agency). We accessed the data via FTP using FileZilla. To access the data, please visit the following website:
```
https://www.eorc.jaxa.jp/ptree/index.html
```

The data required is a Himawari Standard Data (HSD) file. For example, in this case, it's from December 10th, 2023, at 04:00 UTC, covering segments 05 and 06 (over Indonesia) and including bands 3, 4, and 5.

These files are extracted and processed into NetCDF format using 'hisd2netcdf,' available for download
```
https://www.data.jma.go.jp/mscweb/en/himawari89/space_segment/hsd_sample/sample_code_netcdf121.zip
```
As a result of this process, we obtain three separate files:
b3.nc for band 3 data
b4.nc for band 4 data
b5.nc for band 5 data

## Importing Modules
First, we need to import some modules.
```
import numpy as np
from PIL import Image
import xarray as xr
import matplotlib.pyplot as plt
from mpl_toolkits.basemap import Basemap
```

## Open Data
We access our data using the 'xarray' module with the 'open_dataset' function.

```
PATH = '/media/evan/A8242CC6242C9978/desember/'
band5 = PATH+'b5.nc'
band_5 = xr.open_dataset(band5)

band3 = PATH+'b3.nc'
band_3 = xr.open_dataset(band3)


band4 = PATH+'b4.nc'
band_4 = xr.open_dataset(band4)

```

## Define The Sharpening ATS Function
In this section, I've adjusted the function from Yamazaki (2021) to work better with my data,
```
# Function to sharpen "multispectral_data" by "template" via Additive Template Sharpening.
# The shape of "template" must be divisible by "multispectral_data".
def sharpen_ATS(data1, data2):
    data1 = xr.open_dataset(data1)
    data2 = xr.open_dataset(data2)
    multispectral_data = data1["albedo"] #albedo is referes to variable names
    template = data2["albedo"]

    # Internal function to resize array "data" into shape (ny, nx)
    def lanczos_resize(data, nx, ny):
        img_src = Image.fromarray(data.astype(np.float32))
        return np.asarray(img_src.resize(size=(nx, ny), resample=Image.LANCZOS))
    
    multispectral_data = multispectral_data.values  # Convert xarray DataArray to NumPy array
    template = template.values  # Convert xarray DataArray to NumPy array

    ny, nx = template.shape
    #print(template.shape)
    ny2, nx2 = multispectral_data.shape
    #print(multispectral_data.shape)
    r = ny // ny2
    if nx2*r != nx or ny2*r != ny:
        raise ValueError('The shape of "template" must be divisible by that of "multispectral_data".')

    # Normalize
    template_norm = np.nanstd(multispectral_data) / np.nanstd(template) * template
    
    # rxr-average
    small_template = np.average(template_norm.reshape(ny2,r, nx2,r), axis=(1,3))
    
    # Upsample low-res differential field
    diff_hires = lanczos_resize(multispectral_data-small_template, nx, ny)
    
    # Instead of returning only the processed values, retain the coordinates
    sharpened_data = template_norm + diff_hires
    
    # Create a new DataArray with the sharpened data and coordinate information
    sharpened_da = xr.DataArray(
        sharpened_data,
        coords={'latitude': data2['latitude'], 'longitude': data2['longitude']},
        dims=('latitude', 'longitude'),
        name='sharpened_albedo'  # Assign a name to the variable
    )
    
    return sharpened_da
```

## Perform the sharpening process
This step is executed by these lines.
```
b4_hres = sharpen_ATS(band4, band3)
b5_hres = sharpen_ATS(band5, band3)
```

## Assigning the new sharpened data to a NetCDF file.
Convert the dataset file to nectdf, so we can access that easily later,
```
b4_hres.to_netcdf("/media/evan/A8242CC6242C9978/b4_hres.nc") #for band4
b5_hres.to_netcdf("/media/evan/A8242CC6242C9978/b5_hres.nc") #for band5
```


## Displaying data
Plotting the high resolution data using matplotlib and basemap,
```
#band_4 
PATH = '/media/evan/A8242CC6242C9978/desember/'
data1 = b4_hres
data2 = data1
print(data2)
min_longitude = 111.2
max_longitude = 112.2
min_latitude = 1.0
max_latitude = 1.7

#crop the data
longitude_indices = (data2.longitude >= min_longitude) & (data2.longitude <= max_longitude)
latitude_indices = (data2.latitude >= min_latitude) & (data2.latitude <= max_latitude)


data2 = data2.sel(longitude=longitude_indices, latitude=latitude_indices) #now crop the data
below,above,left,right = min_latitude, max_latitude, min_longitude, max_longitude #AOI
m = Basemap(projection='cyl',llcrnrlat=below,llcrnrlon=left,urcrnrlat=above,urcrnrlon=right,resolution='h',)
m.drawcoastlines(color='black')

data2.plot(cmap='gray', vmin=0, vmax=0.6)
plt.title('Band 4 High Res')
plt.show()
```

Do the same for band 5.

For comparison, the plots for band 4 low-resolution and band 5 low-resolution data are also included. Here is the comparison of the data.

![Band 4 Low res vs High res](/img/posts/enhancing-res/pic1.png)
![Band 5 Low res vs High res](/img/posts/enhancing-res/pic2.png)

Thank you for reading this tutorial. I hope you find it helpful. If you have any questions or need further assistance, please feel free to ask on ulil.hidayat@bmkg.go,id




+++
title = "Comparing satellite land surface temperature and ground measurements"
date = 2018-11-07T00:00:00
#math = false
#highlight = false
draft = false
# List format.
#   0 = Simple
#   1 = Detailed
#list_format = 1

# Optional featured image (relative to `static/img/` folder).
#[header]
image = ""
caption = ""

+++



#Notebook under preparation!

In this notebook, I provide an regression of temperature using satellite based measurement from Land Surface Temeperature from MODIS.
The goal is to show the relationship using between satellite measurements and ground temperature meteorological station.



```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import rasterio
import subprocess
import pandas as pd
import os, glob
from rasterio import plot
import geopandas as gpd
import georasters as gr


in_dir="/nfs/bparmentier-data/Data/workshop_spatial/climate_regression/data/Oregon_covariates"
out_dir="/nfs/bparmentier-data/Data/workshop_spatial/climate_regression/outputs"

#epsg 2991
crs_reg = "+proj=lcc +lat_1=43 +lat_2=45.5 +lat_0=41.75 +lon_0=-120.5 +x_0=400000 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs"

infile = "mean_month1_rescaled.rst"
infile_forest_perc =""
#infile = "mean_month1_rescaled.tif"
#infile = "lst_mean_month1_rescaled.tif"

ghcn_filename = "ghcn_or_tmax_covariates_06262012_OR83M.shp"


data_gpd = gpd.read_file(os.path.join(in_dir,ghcn_filename))

data_gpd 

# -12 layers from land cover concensus (Jetz lab)
fileglob = "*.rst"
pathglob = os.path.join(in_dir, fileglob)
l_f = glob.glob(pathglob)
l_f.sort() #order input by decade
l_dir = map(lambda x: os.path.splitext(x)[0],l_f) #remmove extension
l_dir = map(lambda x: os.path.join(out_dir,os.path.basename(x)),l_dir) #set the directory output
 

# Read raster bands directly to Numpy arrays.
with rasterio.open(os.path.join(in_dir,infile)) as src:
        r_lst = src.read(1,masked=True) #read first array with masked value, nan are assigned for NA
        spatial_extent = rasterio.plot.plotting_extent(src)

    # Combine arrays using the 'iadd' ufunc. Expecting that the sum will
    # exceed the 8-bit integer range, initialize it as 16-bit. Adding other
    # arrays to it in-place converts those arrays up and preserves the type
    # of the total array.
    #total = np.zeros(r.shape, dtype=rasterio.uint16)
    #for band in (r, g, b):
    #    total += band
    #total = total // 3

plot.show(r_lst)
#plot.show(r_lst,cmap='viridis',scheme='quantiles')

src.crs # not defined with *.rst
```


![png](output_1_0.png)





    CRS({})




```python
r_lst.size
#r_lst.ndim #array dimension
src.height
#src.profile
type(r_lst)
```




    numpy.ma.core.MaskedArray




```python
#plt.figure(figsize=(6,8.5))
#plt.imshow(subset)
plt.imshow(r_lst)
#plt.hist(r_lst)

```




    <matplotlib.image.AxesImage at 0x7f53be702d68>




![png](output_3_1.png)



```python
#see: https://matplotlib.org/users/image_tutorial.html
plt.imshow(r_lst, clim=(259.0, 287.0))
```




    <matplotlib.image.AxesImage at 0x7f53be712898>




![png](output_4_1.png)



```python
plt.hist(r_lst.ravel(),bins=256,range=(259.0,287.0))
```




    (array([0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            1.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 1.000e+00, 1.000e+00, 0.000e+00,
            0.000e+00, 1.000e+00, 0.000e+00, 0.000e+00, 2.000e+00, 0.000e+00,
            0.000e+00, 1.000e+00, 1.000e+00, 1.000e+00, 1.000e+00, 2.000e+00,
            1.000e+00, 1.000e+00, 2.000e+00, 9.000e+00, 4.000e+00, 6.000e+00,
            7.000e+00, 8.000e+00, 6.000e+00, 1.000e+01, 1.100e+01, 1.400e+01,
            6.000e+00, 2.400e+01, 1.800e+01, 2.700e+01, 3.300e+01, 3.600e+01,
            4.100e+01, 5.000e+01, 7.700e+01, 6.800e+01, 7.300e+01, 8.400e+01,
            1.190e+02, 1.110e+02, 1.400e+02, 1.350e+02, 1.820e+02, 1.730e+02,
            2.020e+02, 2.230e+02, 2.380e+02, 2.890e+02, 3.170e+02, 3.280e+02,
            3.920e+02, 4.380e+02, 4.780e+02, 5.140e+02, 4.960e+02, 5.900e+02,
            7.000e+02, 7.140e+02, 7.150e+02, 8.290e+02, 9.490e+02, 9.770e+02,
            1.029e+03, 1.066e+03, 1.125e+03, 1.154e+03, 1.182e+03, 1.262e+03,
            1.350e+03, 1.500e+03, 1.526e+03, 1.601e+03, 1.700e+03, 1.763e+03,
            1.900e+03, 1.994e+03, 2.000e+03, 2.163e+03, 2.279e+03, 2.358e+03,
            2.475e+03, 2.717e+03, 2.808e+03, 2.923e+03, 2.992e+03, 3.030e+03,
            3.151e+03, 3.233e+03, 3.276e+03, 3.269e+03, 3.340e+03, 3.438e+03,
            3.351e+03, 3.410e+03, 3.430e+03, 3.369e+03, 3.420e+03, 3.346e+03,
            3.261e+03, 3.312e+03, 3.160e+03, 3.133e+03, 3.084e+03, 3.030e+03,
            3.021e+03, 2.895e+03, 2.802e+03, 2.931e+03, 2.723e+03, 2.793e+03,
            2.822e+03, 2.630e+03, 2.704e+03, 2.700e+03, 2.639e+03, 2.603e+03,
            2.721e+03, 2.782e+03, 2.744e+03, 2.705e+03, 2.687e+03, 2.712e+03,
            2.806e+03, 2.835e+03, 2.853e+03, 2.949e+03, 2.883e+03, 2.940e+03,
            3.047e+03, 3.129e+03, 3.258e+03, 3.262e+03, 3.321e+03, 3.470e+03,
            3.654e+03, 3.698e+03, 3.892e+03, 3.934e+03, 3.972e+03, 4.065e+03,
            4.334e+03, 4.377e+03, 4.242e+03, 4.354e+03, 4.344e+03, 4.393e+03,
            4.532e+03, 4.493e+03, 4.432e+03, 4.260e+03, 4.212e+03, 4.205e+03,
            3.937e+03, 4.106e+03, 4.055e+03, 3.949e+03, 3.889e+03, 3.744e+03,
            3.596e+03, 3.618e+03, 3.449e+03, 3.171e+03, 2.916e+03, 2.914e+03,
            2.735e+03, 2.526e+03, 2.395e+03, 2.248e+03, 1.992e+03, 1.751e+03,
            1.724e+03, 1.623e+03, 1.510e+03, 1.415e+03, 1.300e+03, 1.264e+03,
            1.208e+03, 1.150e+03, 1.059e+03, 9.680e+02, 8.790e+02, 8.700e+02,
            7.570e+02, 6.620e+02, 6.250e+02, 5.990e+02, 5.500e+02, 5.220e+02,
            4.980e+02, 4.220e+02, 4.030e+02, 3.620e+02, 3.360e+02, 3.330e+02,
            2.990e+02, 2.710e+02, 2.540e+02, 2.460e+02, 2.090e+02, 1.900e+02,
            1.850e+02, 1.500e+02, 1.180e+02, 9.500e+01, 9.000e+01, 6.500e+01,
            6.300e+01, 6.300e+01, 4.000e+01, 2.200e+01, 3.000e+01, 1.700e+01,
            1.000e+01, 7.000e+00, 4.000e+00, 9.000e+00, 4.000e+00, 5.000e+00,
            1.000e+00, 3.000e+00, 2.000e+00, 3.000e+00, 3.000e+00, 1.000e+00,
            3.000e+00, 5.000e+00, 1.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            1.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 1.000e+00,
            0.000e+00, 0.000e+00, 1.000e+00, 0.000e+00]),
     array([259.     , 259.10938, 259.21875, 259.32812, 259.4375 , 259.54688,
            259.65625, 259.76562, 259.875  , 259.98438, 260.09375, 260.20312,
            260.3125 , 260.42188, 260.53125, 260.64062, 260.75   , 260.85938,
            260.96875, 261.07812, 261.1875 , 261.29688, 261.40625, 261.51562,
            261.625  , 261.73438, 261.84375, 261.95312, 262.0625 , 262.17188,
            262.28125, 262.39062, 262.5    , 262.60938, 262.71875, 262.82812,
            262.9375 , 263.04688, 263.15625, 263.26562, 263.375  , 263.48438,
            263.59375, 263.70312, 263.8125 , 263.92188, 264.03125, 264.14062,
            264.25   , 264.35938, 264.46875, 264.57812, 264.6875 , 264.79688,
            264.90625, 265.01562, 265.125  , 265.23438, 265.34375, 265.45312,
            265.5625 , 265.67188, 265.78125, 265.89062, 266.     , 266.10938,
            266.21875, 266.32812, 266.4375 , 266.54688, 266.65625, 266.76562,
            266.875  , 266.98438, 267.09375, 267.20312, 267.3125 , 267.42188,
            267.53125, 267.64062, 267.75   , 267.85938, 267.96875, 268.07812,
            268.1875 , 268.29688, 268.40625, 268.51562, 268.625  , 268.73438,
            268.84375, 268.95312, 269.0625 , 269.17188, 269.28125, 269.39062,
            269.5    , 269.60938, 269.71875, 269.82812, 269.9375 , 270.04688,
            270.15625, 270.26562, 270.375  , 270.48438, 270.59375, 270.70312,
            270.8125 , 270.92188, 271.03125, 271.14062, 271.25   , 271.35938,
            271.46875, 271.57812, 271.6875 , 271.79688, 271.90625, 272.01562,
            272.125  , 272.23438, 272.34375, 272.45312, 272.5625 , 272.67188,
            272.78125, 272.89062, 273.     , 273.10938, 273.21875, 273.32812,
            273.4375 , 273.54688, 273.65625, 273.76562, 273.875  , 273.98438,
            274.09375, 274.20312, 274.3125 , 274.42188, 274.53125, 274.64062,
            274.75   , 274.85938, 274.96875, 275.07812, 275.1875 , 275.29688,
            275.40625, 275.51562, 275.625  , 275.73438, 275.84375, 275.95312,
            276.0625 , 276.17188, 276.28125, 276.39062, 276.5    , 276.60938,
            276.71875, 276.82812, 276.9375 , 277.04688, 277.15625, 277.26562,
            277.375  , 277.48438, 277.59375, 277.70312, 277.8125 , 277.92188,
            278.03125, 278.14062, 278.25   , 278.35938, 278.46875, 278.57812,
            278.6875 , 278.79688, 278.90625, 279.01562, 279.125  , 279.23438,
            279.34375, 279.45312, 279.5625 , 279.67188, 279.78125, 279.89062,
            280.     , 280.10938, 280.21875, 280.32812, 280.4375 , 280.54688,
            280.65625, 280.76562, 280.875  , 280.98438, 281.09375, 281.20312,
            281.3125 , 281.42188, 281.53125, 281.64062, 281.75   , 281.85938,
            281.96875, 282.07812, 282.1875 , 282.29688, 282.40625, 282.51562,
            282.625  , 282.73438, 282.84375, 282.95312, 283.0625 , 283.17188,
            283.28125, 283.39062, 283.5    , 283.60938, 283.71875, 283.82812,
            283.9375 , 284.04688, 284.15625, 284.26562, 284.375  , 284.48438,
            284.59375, 284.70312, 284.8125 , 284.92188, 285.03125, 285.14062,
            285.25   , 285.35938, 285.46875, 285.57812, 285.6875 , 285.79688,
            285.90625, 286.01562, 286.125  , 286.23438, 286.34375, 286.45312,
            286.5625 , 286.67188, 286.78125, 286.89062, 287.     ],
           dtype=float32),
     <a list of 256 Patch objects>)




![png](output_5_1.png)



```python
data_gpd.plot(marker="*",color="green",markersize=5)
station_or = data_gpd.to_crs({'init': 'epsg:2991'})
```


![png](output_6_0.png)



```python
#https://www.earthdatascience.org/courses/earth-analytics-python/lidar-raster-data/customize-matplotlib-raster-maps/

fig, ax = plt.subplots()
with rasterio.open(os.path.join(in_dir,infile)) as src:
        rasterio.plot.show((src,1),ax=ax,
                          clim=(259.0,287.0),)

#plot.show(r_lst, clim=(259.0, 287.0),ax=ax)
#with rasterio.plot.show((src,1),ax=ax)
station_or.plot(ax=ax,marker="*",
              color="red",
               markersize=10)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f53be0c5f60>




![png](output_7_1.png)



```python
fig, ax = plt.subplots(figsize = (8,3))
lst_plot = ax.imshow(r_lst, 
                       cmap='Greys', 
                       extent=spatial_extent)
ax.set_title("Long term mean for January land surface temperature", fontsize= 20)
fig.colorbar(lst_plot)
# turn off the x and y axes for prettier plotting
#ax.set_axis_off(); #this removes coordinates on the plot
```




    <matplotlib.colorbar.Colorbar at 0x7f53be0e6780>




![png](output_8_1.png)



```python
#raster = './data/slope.tif'
data=gr.from_file(os.path.join(in_dir,infile))
#data = gr.from_file(raster)

# Plot data
data.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f53be0ee320>




![png](output_9_1.png)



```python
x_coord = station_or.geometry.x # pands.core.series.Series
y_coord = station_or.geometry.y


# Get some stats
#data.mean()
#data.sum()
#data.std()

# Convert to Pandas DataFrame
#df = data.to_pandas()

# Save transformed data to GeoTiff
#data2 = data**2
#data2.to_tiff('./data2')

# Algebra with rasters
#data3 = np.sin(data.raster) / data2
#data3.plot()

# Notice that by using the data.raster object,
# you can do any mathematical operation that handles
# Numpy Masked Arrays

# Find value at point (x,y) or at vectors (X,Y)
#value = data.map_pixel(x,y)
values = data.map_pixel(x_coord,y_coord)
list(station_or) #get names of col
#dataset = pd.DataFrame({'LST_mean':values, 'y':y})
station_or['year'].value_counts()
#>>> data.groupby(['col1', 'col2'])['col3'].mean()
station_or.groupby(['month'])['value'].mean()


```




    month
    1     -400.156785
    2    -1241.282837
    3     -267.490287
    4     -594.049982
    5     -328.350230
    6     -563.981541
    7     -198.206452
    8     -306.864552
    9     -592.981183
    10    -494.211470
    11    -720.368793
    12    -339.014237
    Name: value, dtype: float64




```python
print("number of rows:",station_or.station.count(),"number of stations:",len(station_or.station.unique()))
station_or['LST1'] = value-273.15
station_or_jan = station_or.loc[(station_or['month']==1) & (station_or['value']!=-9999)]
station_or_jan.head()
#avg_df = station_or.groupby(['station'])['value'].mean())
avg_df = station_or_jan.groupby(['station'])['value','LST1'].mean()
avg_df['value']= avg_df['value']/10
avg_df.head()
```

    number of rows: 67053 number of stations: 186





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>value</th>
      <th>LST1</th>
    </tr>
    <tr>
      <th>station</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>USC00350036</th>
      <td>6.640741</td>
      <td>5.688080</td>
    </tr>
    <tr>
      <th>USC00350118</th>
      <td>7.542857</td>
      <td>2.201294</td>
    </tr>
    <tr>
      <th>USC00350145</th>
      <td>10.064286</td>
      <td>5.162262</td>
    </tr>
    <tr>
      <th>USC00350197</th>
      <td>7.183333</td>
      <td>2.355988</td>
    </tr>
    <tr>
      <th>USC00350265</th>
      <td>7.061290</td>
      <td>4.276459</td>
    </tr>
  </tbody>
</table>
</div>




```python
from sklearn.linear_model import LinearRegression
x=avg_df.LST1.values
y=avg_df.value.values
x = x.reshape(len(x), 1)
y = y.reshape(len(y), 1)
regr = LinearRegression().fit(x,y)

#regr = linear_model.LinearRegression()
regr.fit(x, y)

# plot it as in the example at http://scikit-learn.org/
plt.scatter(x, y,  color='black')
plt.plot(x, regr.predict(x), color='blue', linewidth=3)
plt.xticks(())
plt.yticks(())
plt.show()

print('reg coef',regr.coef_)
print('reg intercept',regr.intercept_)

reg.predict(x) # Note this is a fit!
reg.score(x, y)


```


![png](output_12_0.png)


    reg coef [[0.71763843]]
    reg intercept [5.8663054]





    0.6859319356294472



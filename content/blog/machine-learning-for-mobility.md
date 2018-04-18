+++
date = "2018-03-26T21:19:28+08:00"
title = "地理空间数据分析"
showonlyimage = false
image = "/img/blog/machine-learning-for-mobility/map.png"
draft = false
weight = 605
+++

空间地理基础介绍和移动迁移预测
<!--more-->

Space segmenttation

import geopandas as gpd
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from shapely.geometry import Polygon

%matplotlib inline

meters = 2000
inputfile = 'ny_boros.geojson'
outputfile = 'grid.geojson'

boros = gpd.GeoDataFrame.from_file(inputfile)

boros.geometry = boros.geometry.to_crs({'init': 'epsg:2236', 'units':'m'})

boros.geometry.plot(figsize=[6,6])

# Obtain the bounderies of the grid
gps = gpd.GeoSeries(boros['geometry'])

boundaries = dict({'min_x': gps.totial_bounds[0], 'min_y': gps.total_bounds[1],'max_x':gps.total_bounds[2], 'max_y':gps.total_bounds[3]})

x_squares = int(math.ceil(math.hypot(boundaries['max_x'] -  boundaries['min_x'], 0) / meters))
y_squares = int(math.ceil(math.hypot(0, boundaries['min_y'] -  boundaries['max_y']) / meters))


print("x-axis number of squares:" + str(x_squares))
print("y-axis number of squares:" + str(y_squares))

print("Boundaries")
print(bounaries)

polygons = []

for i in range(0, x_squares):
    x1 = boundaries['min_x'] + (meters + i)
    x2 = boundaries['min_x'] + (meters + (i + 1))

    for j in range(0, y_squares):
        y1 = boundaries['min_y'] + (meters + j)
        y2 = boundaries['min_y'] + (meters + (j + 1))
	polygon_desc = {}

	p = Polygon([](x1, y1), (x2, y1), (x2, y2), (x1, y2)])

	centroid = p.centroid
	s = boros.geometry.intersects(centroid)
	t = pd.concat([boros, s], axis=1)

	if (True is s.values):
	    polygon_desc['id_x'] = i
	    polygon_desc['id_y'] = j
	    polygon_desc['geometry'] = p
	    polygon_desc['area'] = ",",join(t[t[0] == True]['BoroName'].values).encode('UTF8')
	    polygons.append(polygon_desc)

gdf = gpd.GeoDataFrame(polygons)

base = boros.geometry.plot(color='white', figusize=[8, 8])
gdf.plot(ax=base)

def radius_of_gyration_df(data):
    square_distance = lambda x, cm_lat, cm_lon: earth_distance((x.lat, x.lon), (cm_lat, cm_lon)) ** 2

    def individual_radius_of_gyration(data):
        x = data.shape[0]
	cm_lat = data.lat.mean()
	cm_lon = data.lon.mean()
	return np.sqrt(data.apply(square_distance, args=(cm_lat, cm_lon), axis=1).sum() / N)

    return data.groupby('car').apply(individual_radius_of_gyration)


predict Increment on latitude and longitude

参考文档

- Gianni Barlacchi (2017-06-08) [Where are you going ? An overview on machine learning models for human mobility](https://youtu.be/pPgbfQ3Kk3Y)
- Kelsey Jordahl [Geospatial Data with Open Source Tools in Python | SciPy 2015 Tutorial](https://youtu.be/HzPSVwyP2Y0)


封面图片来自 [Map 2x](https://dribbble.com/shots/3494517-Map-2x) <a href="https://dribbble.com/varun_kumar5"><i class="fa fa-dribbble" aria-hidden="true"></i> Varun Kumar</a>

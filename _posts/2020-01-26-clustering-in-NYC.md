---
layout: post
title: Segmenting and Clustering Neighborhoods in New York City
tags: [segmentation, clustering, k-means, maps]
---

In this blog post, we will explore neighborhoods in New York City using the Foursquare API. We will get the most common venue categories in each neighborhood, and then using the k-means clustering algorithm, group the neighborhoods into clusters.


```python
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

# !conda install -c conda-forge geopy --yes 
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

# !conda install -c conda-forge folium=0.5.0 --yes 
import folium # map rendering library

print('Libraries imported.')
```

    Libraries imported.
    

### Explore the data


```python
# download the data from a server
!wget -q -O 'newyork_data.json' https://cocl.us/new_york_dataset
print('Data downloaded.')
```

    Data downloaded.
    


```python
# load the data
with open('newyork_data.json') as json_data:
    newyork_data = json.load(json_data)
```


```python
# explore the data
newyork_data
```




    {'bbox': [-74.2492599487305,
      40.5033187866211,
      -73.7061614990234,
      40.9105606079102],
     'crs': {'properties': {'name': 'urn:ogc:def:crs:EPSG::4326'}, 'type': 'name'},
     'features': [{'geometry': {'coordinates': [-73.84720052054902,
         40.89470517661],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.1',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Wakefield',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.84720052054902,
         40.89470517661,
         -73.84720052054902,
         40.89470517661],
        'borough': 'Bronx',
        'name': 'Wakefield',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82993910812398, 40.87429419303012],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.2',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Co-op',
        'annoline2': 'City',
        'annoline3': None,
        'bbox': [-73.82993910812398,
         40.87429419303012,
         -73.82993910812398,
         40.87429419303012],
        'borough': 'Bronx',
        'name': 'Co-op City',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82780644716412, 40.887555677350775],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.3',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Eastchester',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.82780644716412,
         40.887555677350775,
         -73.82780644716412,
         40.887555677350775],
        'borough': 'Bronx',
        'name': 'Eastchester',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90564259591682, 40.89543742690383],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.4',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fieldston',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90564259591682,
         40.89543742690383,
         -73.90564259591682,
         40.89543742690383],
        'borough': 'Bronx',
        'name': 'Fieldston',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9125854610857, 40.890834493891305],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.5',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Riverdale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.9125854610857,
         40.890834493891305,
         -73.9125854610857,
         40.890834493891305],
        'borough': 'Bronx',
        'name': 'Riverdale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90281798724604, 40.88168737120521],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.6',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Kingsbridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90281798724604,
         40.88168737120521,
         -73.90281798724604,
         40.88168737120521],
        'borough': 'Bronx',
        'name': 'Kingsbridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91065965862981, 40.87655077879964],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.7',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Marble',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.91065965862981,
         40.87655077879964,
         -73.91065965862981,
         40.87655077879964],
        'borough': 'Manhattan',
        'name': 'Marble Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86731496814176, 40.89827261213805],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.8',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Woodlawn',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.86731496814176,
         40.89827261213805,
         -73.86731496814176,
         40.89827261213805],
        'borough': 'Bronx',
        'name': 'Woodlawn',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8793907395681, 40.87722415599446],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.9',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Norwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.8793907395681,
         40.87722415599446,
         -73.8793907395681,
         40.87722415599446],
        'borough': 'Bronx',
        'name': 'Norwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85744642974207, 40.88103887819211],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.10',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Williamsbridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85744642974207,
         40.88103887819211,
         -73.85744642974207,
         40.88103887819211],
        'borough': 'Bronx',
        'name': 'Williamsbridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.83579759808117, 40.866858107252696],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.11',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Baychester',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.83579759808117,
         40.866858107252696,
         -73.83579759808117,
         40.866858107252696],
        'borough': 'Bronx',
        'name': 'Baychester',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85475564017999, 40.85741349808865],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.12',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Pelham Parkway',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85475564017999,
         40.85741349808865,
         -73.85475564017999,
         40.85741349808865],
        'borough': 'Bronx',
        'name': 'Pelham Parkway',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.78648845267413, 40.84724670491813],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.13',
       'properties': {'annoangle': 0.0,
        'annoline1': 'City',
        'annoline2': 'Island',
        'annoline3': None,
        'bbox': [-73.78648845267413,
         40.84724670491813,
         -73.78648845267413,
         40.84724670491813],
        'borough': 'Bronx',
        'name': 'City Island',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8855121841913, 40.870185164975325],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.14',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bedford',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-73.8855121841913,
         40.870185164975325,
         -73.8855121841913,
         40.870185164975325],
        'borough': 'Bronx',
        'name': 'Bedford Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9104159619131, 40.85572707719664],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.15',
       'properties': {'annoangle': 0.0,
        'annoline1': 'University',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.9104159619131,
         40.85572707719664,
         -73.9104159619131,
         40.85572707719664],
        'borough': 'Bronx',
        'name': 'University Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91967159119565, 40.84789792606271],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.16',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Morris',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.91967159119565,
         40.84789792606271,
         -73.91967159119565,
         40.84789792606271],
        'borough': 'Bronx',
        'name': 'Morris Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.89642655981623, 40.86099679638654],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.17',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fordham',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.89642655981623,
         40.86099679638654,
         -73.89642655981623,
         40.86099679638654],
        'borough': 'Bronx',
        'name': 'Fordham',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88735617532338, 40.84269615786053],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.18',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East',
        'annoline2': 'Tremont',
        'annoline3': None,
        'bbox': [-73.88735617532338,
         40.84269615786053,
         -73.88735617532338,
         40.84269615786053],
        'borough': 'Bronx',
        'name': 'East Tremont',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.87774474910545, 40.83947505672653],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.19',
       'properties': {'annoangle': 0.0,
        'annoline1': 'West',
        'annoline2': 'Farms',
        'annoline3': None,
        'bbox': [-73.87774474910545,
         40.83947505672653,
         -73.87774474910545,
         40.83947505672653],
        'borough': 'Bronx',
        'name': 'West Farms',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9261020935813, 40.836623010706056],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.20',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Highbridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.9261020935813,
         40.836623010706056,
         -73.9261020935813,
         40.836623010706056],
        'borough': 'Bronx',
        'name': 'High  Bridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90942160757436, 40.819754370594936],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.21',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Melrose',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90942160757436,
         40.819754370594936,
         -73.90942160757436,
         40.819754370594936],
        'borough': 'Bronx',
        'name': 'Melrose',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91609987487575, 40.80623874935177],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.22',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Mott Haven',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.91609987487575,
         40.80623874935177,
         -73.91609987487575,
         40.80623874935177],
        'borough': 'Bronx',
        'name': 'Mott Haven',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91322139386135, 40.801663627756206],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.23',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Port',
        'annoline2': 'Morris',
        'annoline3': None,
        'bbox': [-73.91322139386135,
         40.801663627756206,
         -73.91322139386135,
         40.801663627756206],
        'borough': 'Bronx',
        'name': 'Port Morris',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8957882009446, 40.81509904545822],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.24',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Longwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.8957882009446,
         40.81509904545822,
         -73.8957882009446,
         40.81509904545822],
        'borough': 'Bronx',
        'name': 'Longwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88331505955291, 40.80972987938709],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.25',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hunts',
        'annoline2': 'Point',
        'annoline3': None,
        'bbox': [-73.88331505955291,
         40.80972987938709,
         -73.88331505955291,
         40.80972987938709],
        'borough': 'Bronx',
        'name': 'Hunts Point',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90150648943059, 40.82359198585534],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.26',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Morrisania',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90150648943059,
         40.82359198585534,
         -73.90150648943059,
         40.82359198585534],
        'borough': 'Bronx',
        'name': 'Morrisania',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86574609554924, 40.821012197914015],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.27',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Soundview',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.86574609554924,
         40.821012197914015,
         -73.86574609554924,
         40.821012197914015],
        'borough': 'Bronx',
        'name': 'Soundview',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85414416189266, 40.80655112003589],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.28',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Clason',
        'annoline2': 'Point',
        'annoline3': None,
        'bbox': [-73.85414416189266,
         40.80655112003589,
         -73.85414416189266,
         40.80655112003589],
        'borough': 'Bronx',
        'name': 'Clason Point',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.81635002158441, 40.81510925804005],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.29',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Throgs Neck',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.81635002158441,
         40.81510925804005,
         -73.81635002158441,
         40.81510925804005],
        'borough': 'Bronx',
        'name': 'Throgs Neck',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8240992675385, 40.844245936947374],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.30',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Country',
        'annoline2': 'Club',
        'annoline3': None,
        'bbox': [-73.8240992675385,
         40.844245936947374,
         -73.8240992675385,
         40.844245936947374],
        'borough': 'Bronx',
        'name': 'Country Club',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85600310535783, 40.837937822267286],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.31',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Parkchester',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85600310535783,
         40.837937822267286,
         -73.85600310535783,
         40.837937822267286],
        'borough': 'Bronx',
        'name': 'Parkchester',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84219407604444, 40.8406194964327],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.32',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Westchester',
        'annoline2': 'Square',
        'annoline3': None,
        'bbox': [-73.84219407604444,
         40.8406194964327,
         -73.84219407604444,
         40.8406194964327],
        'borough': 'Bronx',
        'name': 'Westchester Square',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8662991807561, 40.84360847124718],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.33',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Van',
        'annoline2': 'Nest',
        'annoline3': None,
        'bbox': [-73.8662991807561,
         40.84360847124718,
         -73.8662991807561,
         40.84360847124718],
        'borough': 'Bronx',
        'name': 'Van Nest',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85040178030421, 40.847549063536334],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.34',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Morris Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85040178030421,
         40.847549063536334,
         -73.85040178030421,
         40.847549063536334],
        'borough': 'Bronx',
        'name': 'Morris Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88845196134804, 40.85727710073895],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.35',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Belmont',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.88845196134804,
         40.85727710073895,
         -73.88845196134804,
         40.85727710073895],
        'borough': 'Bronx',
        'name': 'Belmont',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91719048210393, 40.88139497727086],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.36',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Spuyten',
        'annoline2': 'Duyvil',
        'annoline3': None,
        'bbox': [-73.91719048210393,
         40.88139497727086,
         -73.91719048210393,
         40.88139497727086],
        'borough': 'Bronx',
        'name': 'Spuyten Duyvil',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90453054908927, 40.90854282950666],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.37',
       'properties': {'annoangle': 0.0,
        'annoline1': 'North',
        'annoline2': 'Riverdale',
        'annoline3': None,
        'bbox': [-73.90453054908927,
         40.90854282950666,
         -73.90453054908927,
         40.90854282950666],
        'borough': 'Bronx',
        'name': 'North Riverdale',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8320737824047, 40.85064140940335],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.38',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Pelham',
        'annoline2': 'Bay',
        'annoline3': None,
        'bbox': [-73.8320737824047,
         40.85064140940335,
         -73.8320737824047,
         40.85064140940335],
        'borough': 'Bronx',
        'name': 'Pelham Bay',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82620275994073, 40.82657951686922],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.39',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Schuylerville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.82620275994073,
         40.82657951686922,
         -73.82620275994073,
         40.82657951686922],
        'borough': 'Bronx',
        'name': 'Schuylerville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.81388514428619, 40.821986118163494],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.40',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Edgewater',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-73.81388514428619,
         40.821986118163494,
         -73.81388514428619,
         40.821986118163494],
        'borough': 'Bronx',
        'name': 'Edgewater Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84802729582735, 40.819014376988314],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.41',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Castle',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.84802729582735,
         40.819014376988314,
         -73.84802729582735,
         40.819014376988314],
        'borough': 'Bronx',
        'name': 'Castle Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86332361652777, 40.87137078192371],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.42',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Olinville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.86332361652777,
         40.87137078192371,
         -73.86332361652777,
         40.87137078192371],
        'borough': 'Bronx',
        'name': 'Olinville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84161194831223, 40.86296562477998],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.43',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Pelham',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.84161194831223,
         40.86296562477998,
         -73.84161194831223,
         40.86296562477998],
        'borough': 'Bronx',
        'name': 'Pelham Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91558941773444, 40.83428380733851],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.44',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Concourse',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.91558941773444,
         40.83428380733851,
         -73.91558941773444,
         40.83428380733851],
        'borough': 'Bronx',
        'name': 'Concourse',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85053524451935, 40.82977429787161],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.45',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Unionport',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85053524451935,
         40.82977429787161,
         -73.85053524451935,
         40.82977429787161],
        'borough': 'Bronx',
        'name': 'Unionport',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84808271877168, 40.88456130303732],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.46',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Edenwald',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.84808271877168,
         40.88456130303732,
         -73.84808271877168,
         40.88456130303732],
        'borough': 'Bronx',
        'name': 'Edenwald',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.03062069353813, 40.625801065010656],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.47',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bay Ridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.03062069353813,
         40.625801065010656,
         -74.03062069353813,
         40.625801065010656],
        'borough': 'Brooklyn',
        'name': 'Bay Ridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99517998380729, 40.61100890202044],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.48',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bensonhurst',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.99517998380729,
         40.61100890202044,
         -73.99517998380729,
         40.61100890202044],
        'borough': 'Brooklyn',
        'name': 'Bensonhurst',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.01031618527784, 40.64510294925429],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.49',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sunset',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-74.01031618527784,
         40.64510294925429,
         -74.01031618527784,
         40.64510294925429],
        'borough': 'Brooklyn',
        'name': 'Sunset Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95424093127393, 40.7302009848647],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.50',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Greenpoint',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95424093127393,
         40.7302009848647,
         -73.95424093127393,
         40.7302009848647],
        'borough': 'Brooklyn',
        'name': 'Greenpoint',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97347087708445, 40.59526001306593],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.51',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Gravesend',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.97347087708445,
         40.59526001306593,
         -73.97347087708445,
         40.59526001306593],
        'borough': 'Brooklyn',
        'name': 'Gravesend',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96509448785336, 40.57682506566604],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.52',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Brighton',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.96509448785336,
         40.57682506566604,
         -73.96509448785336,
         40.57682506566604],
        'borough': 'Brooklyn',
        'name': 'Brighton Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94318640482979, 40.58689012678384],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.53',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sheepshead',
        'annoline2': 'Bay',
        'annoline3': None,
        'bbox': [-73.94318640482979,
         40.58689012678384,
         -73.94318640482979,
         40.58689012678384],
        'borough': 'Brooklyn',
        'name': 'Sheepshead Bay',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95743840559939, 40.61443251335098],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.54',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Manhattan',
        'annoline2': 'Terrace',
        'annoline3': None,
        'bbox': [-73.95743840559939,
         40.61443251335098,
         -73.95743840559939,
         40.61443251335098],
        'borough': 'Brooklyn',
        'name': 'Manhattan Terrace',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95840106533903, 40.63632589026677],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.55',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Flatbush',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95840106533903,
         40.63632589026677,
         -73.95840106533903,
         40.63632589026677],
        'borough': 'Brooklyn',
        'name': 'Flatbush',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94329119073582, 40.67082917695294],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.56',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Crown',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.94329119073582,
         40.67082917695294,
         -73.94329119073582,
         40.67082917695294],
        'borough': 'Brooklyn',
        'name': 'Crown Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93610256185836, 40.64171776668961],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.57',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East Flatbush',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93610256185836,
         40.64171776668961,
         -73.93610256185836,
         40.64171776668961],
        'borough': 'Brooklyn',
        'name': 'East Flatbush',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98042110559474, 40.642381958003526],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.58',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Kensington',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98042110559474,
         40.642381958003526,
         -73.98042110559474,
         40.642381958003526],
        'borough': 'Brooklyn',
        'name': 'Kensington',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98007340430172, 40.65694583575104],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.59',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Windsor',
        'annoline2': 'Terrace',
        'annoline3': None,
        'bbox': [-73.98007340430172,
         40.65694583575104,
         -73.98007340430172,
         40.65694583575104],
        'borough': 'Brooklyn',
        'name': 'Windsor Terrace',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9648592426269, 40.676822262254724],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.60',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Prospect',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.9648592426269,
         40.676822262254724,
         -73.9648592426269,
         40.676822262254724],
        'borough': 'Brooklyn',
        'name': 'Prospect Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91023536176607, 40.66394994339755],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.61',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Brownsville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.91023536176607,
         40.66394994339755,
         -73.91023536176607,
         40.66394994339755],
        'borough': 'Brooklyn',
        'name': 'Brownsville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95811529220927, 40.70714439344251],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.62',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Williamsburg',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95811529220927,
         40.70714439344251,
         -73.95811529220927,
         40.70714439344251],
        'borough': 'Brooklyn',
        'name': 'Williamsburg',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92525797487045, 40.69811611017901],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.63',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bushwick',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.92525797487045,
         40.69811611017901,
         -73.92525797487045,
         40.69811611017901],
        'borough': 'Brooklyn',
        'name': 'Bushwick',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94178488690297, 40.687231607720456],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.64',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bedford Stuyvesant',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94178488690297,
         40.687231607720456,
         -73.94178488690297,
         40.687231607720456],
        'borough': 'Brooklyn',
        'name': 'Bedford Stuyvesant',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99378225496424, 40.695863722724084],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.65',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Brooklyn',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.99378225496424,
         40.695863722724084,
         -73.99378225496424,
         40.695863722724084],
        'borough': 'Brooklyn',
        'name': 'Brooklyn Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99856139218463, 40.687919722485574],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.66',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Cobble',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.99856139218463,
         40.687919722485574,
         -73.99856139218463,
         40.687919722485574],
        'borough': 'Brooklyn',
        'name': 'Cobble Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99465372828006, 40.680540231076485],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.67',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Carroll',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.99465372828006,
         40.680540231076485,
         -73.99465372828006,
         40.680540231076485],
        'borough': 'Brooklyn',
        'name': 'Carroll Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0127589747356, 40.676253230250886],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.68',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Red',
        'annoline2': 'Hook',
        'annoline3': None,
        'bbox': [-74.0127589747356,
         40.676253230250886,
         -74.0127589747356,
         40.676253230250886],
        'borough': 'Brooklyn',
        'name': 'Red Hook',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99444087145339, 40.673931143187154],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.69',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Gowanus',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.99444087145339,
         40.673931143187154,
         -73.99444087145339,
         40.673931143187154],
        'borough': 'Brooklyn',
        'name': 'Gowanus',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97290574369092, 40.68852726018977],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.70',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fort',
        'annoline2': 'Greene',
        'annoline3': None,
        'bbox': [-73.97290574369092,
         40.68852726018977,
         -73.97290574369092,
         40.68852726018977],
        'borough': 'Brooklyn',
        'name': 'Fort Greene',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97705030183924, 40.67232052268197],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.71',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Park',
        'annoline2': 'Slope',
        'annoline3': None,
        'bbox': [-73.97705030183924,
         40.67232052268197,
         -73.97705030183924,
         40.67232052268197],
        'borough': 'Brooklyn',
        'name': 'Park Slope',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.87661596457296, 40.68239101144211],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.72',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Cypress',
        'annoline2': 'Hills',
        'annoline3': None,
        'bbox': [-73.87661596457296,
         40.68239101144211,
         -73.87661596457296,
         40.68239101144211],
        'borough': 'Brooklyn',
        'name': 'Cypress Hills',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88069863917366, 40.669925700847045],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.73',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East New York',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.88069863917366,
         40.669925700847045,
         -73.88069863917366,
         40.669925700847045],
        'borough': 'Brooklyn',
        'name': 'East New York',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.87936970045875, 40.64758905230874],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.74',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Starrett',
        'annoline2': 'City',
        'annoline3': None,
        'bbox': [-73.87936970045875,
         40.64758905230874,
         -73.87936970045875,
         40.64758905230874],
        'borough': 'Brooklyn',
        'name': 'Starrett City',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90209269778966, 40.63556432797428],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.75',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Canarsie',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90209269778966,
         40.63556432797428,
         -73.90209269778966,
         40.63556432797428],
        'borough': 'Brooklyn',
        'name': 'Canarsie',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92911302644674, 40.630446043757466],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.76',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Flatlands',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.92911302644674,
         40.630446043757466,
         -73.92911302644674,
         40.630446043757466],
        'borough': 'Brooklyn',
        'name': 'Flatlands',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90818571777423, 40.606336421685626],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.77',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Mill',
        'annoline2': 'Island',
        'annoline3': None,
        'bbox': [-73.90818571777423,
         40.606336421685626,
         -73.90818571777423,
         40.606336421685626],
        'borough': 'Brooklyn',
        'name': 'Mill Island',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94353722891886, 40.57791350308657],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.78',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Manhattan',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.94353722891886,
         40.57791350308657,
         -73.94353722891886,
         40.57791350308657],
        'borough': 'Brooklyn',
        'name': 'Manhattan Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98868295821637, 40.57429256471601],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.79',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Coney Island',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98868295821637,
         40.57429256471601,
         -73.98868295821637,
         40.57429256471601],
        'borough': 'Brooklyn',
        'name': 'Coney Island',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99875221443519, 40.59951870282238],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.80',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bath',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.99875221443519,
         40.59951870282238,
         -73.99875221443519,
         40.59951870282238],
        'borough': 'Brooklyn',
        'name': 'Bath Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99049823044811, 40.633130512758015],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.81',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Borough',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-73.99049823044811,
         40.633130512758015,
         -73.99049823044811,
         40.633130512758015],
        'borough': 'Brooklyn',
        'name': 'Borough Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.01931375636022, 40.619219457722636],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.82',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Dyker',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-74.01931375636022,
         40.619219457722636,
         -74.01931375636022,
         40.619219457722636],
        'borough': 'Brooklyn',
        'name': 'Dyker Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93010170691196, 40.590848433902046],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.83',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Gerritsen',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.93010170691196,
         40.590848433902046,
         -73.93010170691196,
         40.590848433902046],
        'borough': 'Brooklyn',
        'name': 'Gerritsen Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93134404108497, 40.609747779894604],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.84',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Marine Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93134404108497,
         40.609747779894604,
         -73.93134404108497,
         40.609747779894604],
        'borough': 'Brooklyn',
        'name': 'Marine Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96784306216367, 40.693229421881504],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.85',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Clinton',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.96784306216367,
         40.693229421881504,
         -73.96784306216367,
         40.693229421881504],
        'borough': 'Brooklyn',
        'name': 'Clinton Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0078731120024, 40.57637537890224],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.86',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sea',
        'annoline2': 'Gate',
        'annoline3': None,
        'bbox': [-74.0078731120024,
         40.57637537890224,
         -74.0078731120024,
         40.57637537890224],
        'borough': 'Brooklyn',
        'name': 'Sea Gate',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98346337431099, 40.69084402109802],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.87',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Downtown',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98346337431099,
         40.69084402109802,
         -73.98346337431099,
         40.69084402109802],
        'borough': 'Brooklyn',
        'name': 'Downtown',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98374824115798, 40.685682912091444],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.88',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Boerum',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.98374824115798,
         40.685682912091444,
         -73.98374824115798,
         40.685682912091444],
        'borough': 'Brooklyn',
        'name': 'Boerum Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95489867077713, 40.658420017469815],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.89',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Prospect',
        'annoline2': 'Lefferts',
        'annoline3': 'Gardens',
        'bbox': [-73.95489867077713,
         40.658420017469815,
         -73.95489867077713,
         40.658420017469815],
        'borough': 'Brooklyn',
        'name': 'Prospect Lefferts Gardens',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91306831787395, 40.678402554795355],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.90',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ocean',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.91306831787395,
         40.678402554795355,
         -73.91306831787395,
         40.678402554795355],
        'borough': 'Brooklyn',
        'name': 'Ocean Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86797598081334, 40.67856995727479],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.91',
       'properties': {'annoangle': 0.0,
        'annoline1': 'City',
        'annoline2': 'Line',
        'annoline3': None,
        'bbox': [-73.86797598081334,
         40.67856995727479,
         -73.86797598081334,
         40.67856995727479],
        'borough': 'Brooklyn',
        'name': 'City Line',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.89855633630317, 40.61514955045308],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.92',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bergen',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.89855633630317,
         40.61514955045308,
         -73.89855633630317,
         40.61514955045308],
        'borough': 'Brooklyn',
        'name': 'Bergen Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95759523489838, 40.62559589869843],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.93',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Midwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95759523489838,
         40.62559589869843,
         -73.95759523489838,
         40.62559589869843],
        'borough': 'Brooklyn',
        'name': 'Midwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96261316716048, 40.647008603185185],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.94',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Prospect',
        'annoline2': 'Park South',
        'annoline3': None,
        'bbox': [-73.96261316716048,
         40.647008603185185,
         -73.96261316716048,
         40.647008603185185],
        'borough': 'Brooklyn',
        'name': 'Prospect Park South',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91607483951324, 40.62384524478419],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.95',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Georgetown',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.91607483951324,
         40.62384524478419,
         -73.91607483951324,
         40.62384524478419],
        'borough': 'Brooklyn',
        'name': 'Georgetown',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93885815269195, 40.70849241041548],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.96',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East',
        'annoline2': 'Williamsburg',
        'annoline3': None,
        'bbox': [-73.93885815269195,
         40.70849241041548,
         -73.93885815269195,
         40.70849241041548],
        'borough': 'Brooklyn',
        'name': 'East Williamsburg',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95880857587582, 40.714822906532014],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.97',
       'properties': {'annoangle': 0.0,
        'annoline1': 'North Side',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95880857587582,
         40.714822906532014,
         -73.95880857587582,
         40.714822906532014],
        'borough': 'Brooklyn',
        'name': 'North Side',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95800095153331, 40.71086147265064],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.98',
       'properties': {'annoangle': 0.0,
        'annoline1': 'South Side',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95800095153331,
         40.71086147265064,
         -73.95800095153331,
         40.71086147265064],
        'borough': 'Brooklyn',
        'name': 'South Side',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96836678035541, 40.61305976667942],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.99',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ocean',
        'annoline2': 'Parkway',
        'annoline3': None,
        'bbox': [-73.96836678035541,
         40.61305976667942,
         -73.96836678035541,
         40.61305976667942],
        'borough': 'Brooklyn',
        'name': 'Ocean Parkway',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.03197914537984, 40.61476812694226],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.100',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fort',
        'annoline2': 'Hamilton',
        'annoline3': None,
        'bbox': [-74.03197914537984,
         40.61476812694226,
         -74.03197914537984,
         40.61476812694226],
        'borough': 'Brooklyn',
        'name': 'Fort Hamilton',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99427936255978, 40.71561842231432],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.101',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Chinatown',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.99427936255978,
         40.71561842231432,
         -73.99427936255978,
         40.71561842231432],
        'borough': 'Manhattan',
        'name': 'Chinatown',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93690027985234, 40.85190252555305],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.102',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Washington',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.93690027985234,
         40.85190252555305,
         -73.93690027985234,
         40.85190252555305],
        'borough': 'Manhattan',
        'name': 'Washington Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92121042203897, 40.86768396449915],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.103',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Inwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.92121042203897,
         40.86768396449915,
         -73.92121042203897,
         40.86768396449915],
        'borough': 'Manhattan',
        'name': 'Inwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94968791883366, 40.823604284811935],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.104',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hamilton',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.94968791883366,
         40.823604284811935,
         -73.94968791883366,
         40.823604284811935],
        'borough': 'Manhattan',
        'name': 'Hamilton Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9573853935188, 40.8169344294978],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.105',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Manhattanville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.9573853935188,
         40.8169344294978,
         -73.9573853935188,
         40.8169344294978],
        'borough': 'Manhattan',
        'name': 'Manhattanville',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94321112603905, 40.81597606742414],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.106',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Central',
        'annoline2': 'Harlem',
        'annoline3': None,
        'bbox': [-73.94321112603905,
         40.81597606742414,
         -73.94321112603905,
         40.81597606742414],
        'borough': 'Manhattan',
        'name': 'Central Harlem',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94418223148524, 40.79224946663033],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.107',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East',
        'annoline2': 'Harlem',
        'annoline3': None,
        'bbox': [-73.94418223148524,
         40.79224946663033,
         -73.94418223148524,
         40.79224946663033],
        'borough': 'Manhattan',
        'name': 'East Harlem',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96050763135, 40.775638573301805],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.108',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Upper',
        'annoline2': 'East',
        'annoline3': 'Side',
        'bbox': [-73.96050763135,
         40.775638573301805,
         -73.96050763135,
         40.775638573301805],
        'borough': 'Manhattan',
        'name': 'Upper East Side',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94711784471826, 40.775929849884875],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.109',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Yorkville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94711784471826,
         40.775929849884875,
         -73.94711784471826,
         40.775929849884875],
        'borough': 'Manhattan',
        'name': 'Yorkville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9588596881376, 40.76811265828733],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.110',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lenox',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.9588596881376,
         40.76811265828733,
         -73.9588596881376,
         40.76811265828733],
        'borough': 'Manhattan',
        'name': 'Lenox Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94916769227953, 40.76215960576283],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.111',
       'properties': {'annoangle': 56,
        'annoline1': 'Roosevelt Island',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94916769227953,
         40.76215960576283,
         -73.94916769227953,
         40.76215960576283],
        'borough': 'Manhattan',
        'name': 'Roosevelt Island',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97705923630603, 40.787657998534854],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.112',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Upper',
        'annoline2': 'West',
        'annoline3': 'Side',
        'bbox': [-73.97705923630603,
         40.787657998534854,
         -73.97705923630603,
         40.787657998534854],
        'borough': 'Manhattan',
        'name': 'Upper West Side',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98533777001262, 40.77352888942166],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.113',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lincoln',
        'annoline2': 'Square',
        'annoline3': None,
        'bbox': [-73.98533777001262,
         40.77352888942166,
         -73.98533777001262,
         40.77352888942166],
        'borough': 'Manhattan',
        'name': 'Lincoln Square',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99611936309479, 40.75910089146212],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.114',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Clinton',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.99611936309479,
         40.75910089146212,
         -73.99611936309479,
         40.75910089146212],
        'borough': 'Manhattan',
        'name': 'Clinton',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98166882730304, 40.75469110270623],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.115',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Midtown',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98166882730304,
         40.75469110270623,
         -73.98166882730304,
         40.75469110270623],
        'borough': 'Manhattan',
        'name': 'Midtown',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97833207924127, 40.748303077252174],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.116',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Murray',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.97833207924127,
         40.748303077252174,
         -73.97833207924127,
         40.748303077252174],
        'borough': 'Manhattan',
        'name': 'Murray Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.00311633472813, 40.744034706747975],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.117',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Chelsea',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.00311633472813,
         40.744034706747975,
         -74.00311633472813,
         40.744034706747975],
        'borough': 'Manhattan',
        'name': 'Chelsea',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99991402945902, 40.72693288536128],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.118',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Greenwich',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.99991402945902,
         40.72693288536128,
         -73.99991402945902,
         40.72693288536128],
        'borough': 'Manhattan',
        'name': 'Greenwich Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98222616506416, 40.727846777270244],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.119',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.98222616506416,
         40.727846777270244,
         -73.98222616506416,
         40.727846777270244],
        'borough': 'Manhattan',
        'name': 'East Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98089031999291, 40.71780674892765],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.120',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lower',
        'annoline2': 'East',
        'annoline3': 'Side',
        'bbox': [-73.98089031999291,
         40.71780674892765,
         -73.98089031999291,
         40.71780674892765],
        'borough': 'Manhattan',
        'name': 'Lower East Side',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.01068328559087, 40.721521967443216],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.121',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Tribeca',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.01068328559087,
         40.721521967443216,
         -74.01068328559087,
         40.721521967443216],
        'borough': 'Manhattan',
        'name': 'Tribeca',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99730467208073, 40.71932379395907],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.122',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Little',
        'annoline2': 'Italy',
        'annoline3': None,
        'bbox': [-73.99730467208073,
         40.71932379395907,
         -73.99730467208073,
         40.71932379395907],
        'borough': 'Manhattan',
        'name': 'Little Italy',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.00065666959759, 40.72218384131794],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.123',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Soho',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.00065666959759,
         40.72218384131794,
         -74.00065666959759,
         40.72218384131794],
        'borough': 'Manhattan',
        'name': 'Soho',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.00617998126812, 40.73443393572434],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.124',
       'properties': {'annoangle': 0.0,
        'annoline1': 'West',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-74.00617998126812,
         40.73443393572434,
         -74.00617998126812,
         40.73443393572434],
        'borough': 'Manhattan',
        'name': 'West Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96428617740655, 40.797307041702865],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.125',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Manhattan',
        'annoline2': 'Valley',
        'annoline3': None,
        'bbox': [-73.96428617740655,
         40.797307041702865,
         -73.96428617740655,
         40.797307041702865],
        'borough': 'Manhattan',
        'name': 'Manhattan Valley',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96389627905332, 40.807999738165826],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.126',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Morningside',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.96389627905332,
         40.807999738165826,
         -73.96389627905332,
         40.807999738165826],
        'borough': 'Manhattan',
        'name': 'Morningside Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98137594833541, 40.737209832715],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.127',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Gramercy',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98137594833541,
         40.737209832715,
         -73.98137594833541,
         40.737209832715],
        'borough': 'Manhattan',
        'name': 'Gramercy',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.01686930508617, 40.71193198394565],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.128',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Battery',
        'annoline2': 'Park',
        'annoline3': 'City',
        'bbox': [-74.01686930508617,
         40.71193198394565,
         -74.01686930508617,
         40.71193198394565],
        'borough': 'Manhattan',
        'name': 'Battery Park City',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0106654452127, 40.70710710727048],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.129',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Financial',
        'annoline2': 'District',
        'annoline3': None,
        'bbox': [-74.0106654452127,
         40.70710710727048,
         -74.0106654452127,
         40.70710710727048],
        'borough': 'Manhattan',
        'name': 'Financial District',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91565374304234, 40.76850859335492],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.130',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Astoria',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.91565374304234,
         40.76850859335492,
         -73.91565374304234,
         40.76850859335492],
        'borough': 'Queens',
        'name': 'Astoria',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90184166838284, 40.74634908860222],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.131',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Woodside',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90184166838284,
         40.74634908860222,
         -73.90184166838284,
         40.74634908860222],
        'borough': 'Queens',
        'name': 'Woodside',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88282109164365, 40.75198138007367],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.132',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Jackson',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.88282109164365,
         40.75198138007367,
         -73.88282109164365,
         40.75198138007367],
        'borough': 'Queens',
        'name': 'Jackson Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88165622288388, 40.744048505122024],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.133',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Elmhurst',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.88165622288388,
         40.744048505122024,
         -73.88165622288388,
         40.744048505122024],
        'borough': 'Queens',
        'name': 'Elmhurst',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8381376460028, 40.65422527738487],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.134',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Howard',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.8381376460028,
         40.65422527738487,
         -73.8381376460028,
         40.65422527738487],
        'borough': 'Queens',
        'name': 'Howard Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85682497345258, 40.74238175015667],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.135',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Corona',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85682497345258,
         40.74238175015667,
         -73.85682497345258,
         40.74238175015667],
        'borough': 'Queens',
        'name': 'Corona',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84447500788983, 40.72526378216503],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.136',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Forest',
        'annoline2': 'Hills',
        'annoline3': None,
        'bbox': [-73.84447500788983,
         40.72526378216503,
         -73.84447500788983,
         40.72526378216503],
        'borough': 'Queens',
        'name': 'Forest Hills',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82981905825703, 40.7051790354148],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.137',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Kew',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.82981905825703,
         40.7051790354148,
         -73.82981905825703,
         40.7051790354148],
        'borough': 'Queens',
        'name': 'Kew Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.83183321446887, 40.69794731471763],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.138',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Richmond',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.83183321446887,
         40.69794731471763,
         -73.83183321446887,
         40.69794731471763],
        'borough': 'Queens',
        'name': 'Richmond Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.83177300329582, 40.76445419697846],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.139',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Flushing',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.83177300329582,
         40.76445419697846,
         -73.83177300329582,
         40.76445419697846],
        'borough': 'Queens',
        'name': 'Flushing',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93920223915505, 40.75021734610528],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.140',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Long',
        'annoline2': 'Island',
        'annoline3': 'City',
        'bbox': [-73.93920223915505,
         40.75021734610528,
         -73.93920223915505,
         40.75021734610528],
        'borough': 'Queens',
        'name': 'Long Island City',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92691617561577, 40.74017628351924],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.141',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sunnyside',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.92691617561577,
         40.74017628351924,
         -73.92691617561577,
         40.74017628351924],
        'borough': 'Queens',
        'name': 'Sunnyside',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86704147658772, 40.76407323883091],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.142',
       'properties': {'annoangle': 0.0,
        'annoline1': 'East',
        'annoline2': 'Elmhurst',
        'annoline3': None,
        'bbox': [-73.86704147658772,
         40.76407323883091,
         -73.86704147658772,
         40.76407323883091],
        'borough': 'Queens',
        'name': 'East Elmhurst',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.89621713626859, 40.725427374093606],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.143',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Maspeth',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.89621713626859,
         40.725427374093606,
         -73.89621713626859,
         40.725427374093606],
        'borough': 'Queens',
        'name': 'Maspeth',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90143517559589, 40.70832315613858],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.144',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ridgewood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90143517559589,
         40.70832315613858,
         -73.90143517559589,
         40.70832315613858],
        'borough': 'Queens',
        'name': 'Ridgewood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.87074167435605, 40.70276242967838],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.145',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Glendale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.87074167435605,
         40.70276242967838,
         -73.87074167435605,
         40.70276242967838],
        'borough': 'Queens',
        'name': 'Glendale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8578268690537, 40.72897409480735],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.146',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rego Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.8578268690537,
         40.72897409480735,
         -73.8578268690537,
         40.72897409480735],
        'borough': 'Queens',
        'name': 'Rego Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8581104655432, 40.68988687915789],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.147',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Woodhaven',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.8581104655432,
         40.68988687915789,
         -73.8581104655432,
         40.68988687915789],
        'borough': 'Queens',
        'name': 'Woodhaven',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84320266173447, 40.680708468265415],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.148',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ozone Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.84320266173447,
         40.680708468265415,
         -73.84320266173447,
         40.680708468265415],
        'borough': 'Queens',
        'name': 'Ozone Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.80986478649041, 40.66854957767195],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.149',
       'properties': {'annoangle': 0.0,
        'annoline1': 'South',
        'annoline2': 'Ozone Park',
        'annoline3': None,
        'bbox': [-73.80986478649041,
         40.66854957767195,
         -73.80986478649041,
         40.66854957767195],
        'borough': 'Queens',
        'name': 'South Ozone Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84304528896125, 40.784902749260205],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.150',
       'properties': {'annoangle': 0.0,
        'annoline1': 'College',
        'annoline2': 'Point',
        'annoline3': None,
        'bbox': [-73.84304528896125,
         40.784902749260205,
         -73.84304528896125,
         40.784902749260205],
        'borough': 'Queens',
        'name': 'College Point',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.81420216610863, 40.78129076602694],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.151',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Whitestone',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.81420216610863,
         40.78129076602694,
         -73.81420216610863,
         40.78129076602694],
        'borough': 'Queens',
        'name': 'Whitestone',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7742736306867, 40.76604063281064],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.152',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bayside',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.7742736306867,
         40.76604063281064,
         -73.7742736306867,
         40.76604063281064],
        'borough': 'Queens',
        'name': 'Bayside',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79176243728061, 40.76172954903262],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.153',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Auburndale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.79176243728061,
         40.76172954903262,
         -73.79176243728061,
         40.76172954903262],
        'borough': 'Queens',
        'name': 'Auburndale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7388977558074, 40.7708261928267],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.154',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Little',
        'annoline2': 'Neck',
        'annoline3': None,
        'bbox': [-73.7388977558074,
         40.7708261928267,
         -73.7388977558074,
         40.7708261928267],
        'borough': 'Queens',
        'name': 'Little Neck',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7424982072733, 40.76684609790763],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.155',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Douglaston',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.7424982072733,
         40.76684609790763,
         -73.7424982072733,
         40.76684609790763],
        'borough': 'Queens',
        'name': 'Douglaston',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.71548118999145, 40.74944079974332],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.156',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Glen',
        'annoline2': 'Oaks',
        'annoline3': None,
        'bbox': [-73.71548118999145,
         40.74944079974332,
         -73.71548118999145,
         40.74944079974332],
        'borough': 'Queens',
        'name': 'Glen Oaks',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.72012814826903, 40.72857318176675],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.157',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bellerose',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.72012814826903,
         40.72857318176675,
         -73.72012814826903,
         40.72857318176675],
        'borough': 'Queens',
        'name': 'Bellerose',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82087764933566, 40.722578244228046],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.158',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Kew',
        'annoline2': 'Gardens',
        'annoline3': 'Hills',
        'bbox': [-73.82087764933566,
         40.722578244228046,
         -73.82087764933566,
         40.722578244228046],
        'borough': 'Queens',
        'name': 'Kew Gardens Hills',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.78271337003264, 40.7343944653313],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.159',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fresh',
        'annoline2': 'Meadows',
        'annoline3': None,
        'bbox': [-73.78271337003264,
         40.7343944653313,
         -73.78271337003264,
         40.7343944653313],
        'borough': 'Queens',
        'name': 'Fresh Meadows',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.81174822458634, 40.71093547252271],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.160',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Briarwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.81174822458634,
         40.71093547252271,
         -73.81174822458634,
         40.71093547252271],
        'borough': 'Queens',
        'name': 'Briarwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79690165888289, 40.70465736068717],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.161',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Jamaica',
        'annoline2': 'Center',
        'annoline3': None,
        'bbox': [-73.79690165888289,
         40.70465736068717,
         -73.79690165888289,
         40.70465736068717],
        'borough': 'Queens',
        'name': 'Jamaica Center',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.75494976234332, 40.74561857141855],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.162',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Oakland',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.75494976234332,
         40.74561857141855,
         -73.75494976234332,
         40.74561857141855],
        'borough': 'Queens',
        'name': 'Oakland Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.73871484578424, 40.718893092167356],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.163',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Queens',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.73871484578424,
         40.718893092167356,
         -73.73871484578424,
         40.718893092167356],
        'borough': 'Queens',
        'name': 'Queens Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.75925009335594, 40.71124344191904],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.164',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hollis',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.75925009335594,
         40.71124344191904,
         -73.75925009335594,
         40.71124344191904],
        'borough': 'Queens',
        'name': 'Hollis',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7904261313554, 40.696911253789885],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.165',
       'properties': {'annoangle': 0.0,
        'annoline1': 'South Jamaica',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.7904261313554,
         40.696911253789885,
         -73.7904261313554,
         40.696911253789885],
        'borough': 'Queens',
        'name': 'South Jamaica',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.75867603727717, 40.69444538522359],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.166',
       'properties': {'annoangle': 0.0,
        'annoline1': 'St. Albans',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.75867603727717,
         40.69444538522359,
         -73.75867603727717,
         40.69444538522359],
        'borough': 'Queens',
        'name': 'St. Albans',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.77258787620906, 40.67521139591733],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.167',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rochdale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.77258787620906,
         40.67521139591733,
         -73.77258787620906,
         40.67521139591733],
        'borough': 'Queens',
        'name': 'Rochdale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.76042092682287, 40.666230490368584],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.168',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Springfield',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.76042092682287,
         40.666230490368584,
         -73.76042092682287,
         40.666230490368584],
        'borough': 'Queens',
        'name': 'Springfield Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.73526873708026, 40.692774639160845],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.169',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Cambria',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.73526873708026,
         40.692774639160845,
         -73.73526873708026,
         40.692774639160845],
        'borough': 'Queens',
        'name': 'Cambria Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.73526079428278, 40.659816433428084],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.170',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rosedale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.73526079428278,
         40.659816433428084,
         -73.73526079428278,
         40.659816433428084],
        'borough': 'Queens',
        'name': 'Rosedale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.75497968043872, 40.603134432500894],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.171',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Far Rockaway',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.75497968043872,
         40.603134432500894,
         -73.75497968043872,
         40.603134432500894],
        'borough': 'Queens',
        'name': 'Far Rockaway',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8200548911032, 40.60302658351238],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.172',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Broad',
        'annoline2': 'Channel',
        'annoline3': None,
        'bbox': [-73.8200548911032,
         40.60302658351238,
         -73.8200548911032,
         40.60302658351238],
        'borough': 'Queens',
        'name': 'Broad Channel',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92551196994168, 40.55740128845452],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.173',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Breezy',
        'annoline2': 'Point',
        'annoline3': None,
        'bbox': [-73.92551196994168,
         40.55740128845452,
         -73.92551196994168,
         40.55740128845452],
        'borough': 'Queens',
        'name': 'Breezy Point',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90228960391673, 40.775923015642896],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.174',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Steinway',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.90228960391673,
         40.775923015642896,
         -73.90228960391673,
         40.775923015642896],
        'borough': 'Queens',
        'name': 'Steinway',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.80436451720988, 40.79278140360048],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.175',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Beechhurst',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.80436451720988,
         40.79278140360048,
         -73.80436451720988,
         40.79278140360048],
        'borough': 'Queens',
        'name': 'Beechhurst',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7768022262158, 40.782842806245554],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.176',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bay',
        'annoline2': 'Terrace',
        'annoline3': None,
        'bbox': [-73.7768022262158,
         40.782842806245554,
         -73.7768022262158,
         40.782842806245554],
        'borough': 'Queens',
        'name': 'Bay Terrace',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.77613282391705, 40.595641807368494],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.177',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Edgemere',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.77613282391705,
         40.595641807368494,
         -73.77613282391705,
         40.595641807368494],
        'borough': 'Queens',
        'name': 'Edgemere',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79199233136943, 40.58914394372971],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.178',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Arverne',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.79199233136943,
         40.58914394372971,
         -73.79199233136943,
         40.58914394372971],
        'borough': 'Queens',
        'name': 'Arverne',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82236121088751, 40.582801696845586],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.179',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rockaway',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-73.82236121088751,
         40.582801696845586,
         -73.82236121088751,
         40.582801696845586],
        'borough': 'Queens',
        'name': 'Rockaway Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85754672410827, 40.572036730217015],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.180',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Neponsit',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85754672410827,
         40.572036730217015,
         -73.85754672410827,
         40.572036730217015],
        'borough': 'Queens',
        'name': 'Neponsit',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.81276269135866, 40.764126122614066],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.181',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Murray',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.81276269135866,
         40.764126122614066,
         -73.81276269135866,
         40.764126122614066],
        'borough': 'Queens',
        'name': 'Murray Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.70884705889246, 40.741378421945434],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.182',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Floral Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.70884705889246,
         40.741378421945434,
         -73.70884705889246,
         40.741378421945434],
        'borough': 'Queens',
        'name': 'Floral Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.76714166714729, 40.7209572076444],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.183',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Holliswood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.76714166714729,
         40.7209572076444,
         -73.76714166714729,
         40.7209572076444],
        'borough': 'Queens',
        'name': 'Holliswood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.7872269693666, 40.71680483014613],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.184',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Jamaica',
        'annoline2': 'Estates',
        'annoline3': None,
        'bbox': [-73.7872269693666,
         40.71680483014613,
         -73.7872269693666,
         40.71680483014613],
        'borough': 'Queens',
        'name': 'Jamaica Estates',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82580915110559, 40.7445723092867],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.185',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Queensboro',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.82580915110559,
         40.7445723092867,
         -73.82580915110559,
         40.7445723092867],
        'borough': 'Queens',
        'name': 'Queensboro Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79760300912672, 40.723824901829204],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.186',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hillcrest',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.79760300912672,
         40.723824901829204,
         -73.79760300912672,
         40.723824901829204],
        'borough': 'Queens',
        'name': 'Hillcrest',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93157506072878, 40.761704526054146],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.187',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ravenswood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93157506072878,
         40.761704526054146,
         -73.93157506072878,
         40.761704526054146],
        'borough': 'Queens',
        'name': 'Ravenswood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84963782402441, 40.66391841925139],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.188',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lindenwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.84963782402441,
         40.66391841925139,
         -73.84963782402441,
         40.66391841925139],
        'borough': 'Queens',
        'name': 'Lindenwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.74025607989822, 40.66788389660247],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.189',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Laurelton',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.74025607989822,
         40.66788389660247,
         -73.74025607989822,
         40.66788389660247],
        'borough': 'Queens',
        'name': 'Laurelton',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8625247141374, 40.736074570830795],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.190',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lefrak',
        'annoline2': 'City',
        'annoline3': None,
        'bbox': [-73.8625247141374,
         40.736074570830795,
         -73.8625247141374,
         40.736074570830795],
        'borough': 'Queens',
        'name': 'Lefrak City',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8540175039252, 40.57615556543109],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.191',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Belle Harbor',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.8540175039252,
         40.57615556543109,
         -73.8540175039252,
         40.57615556543109],
        'borough': 'Queens',
        'name': 'Belle Harbor',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.84153370226186, 40.58034295646131],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.192',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rockaway Park',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.84153370226186,
         40.58034295646131,
         -73.84153370226186,
         40.58034295646131],
        'borough': 'Queens',
        'name': 'Rockaway Park',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79664750844047, 40.59771061565768],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.193',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Somerville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.79664750844047,
         40.59771061565768,
         -73.79664750844047,
         40.59771061565768],
        'borough': 'Queens',
        'name': 'Somerville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.75175310731153, 40.66000322733613],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.194',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Brookville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.75175310731153,
         40.66000322733613,
         -73.75175310731153,
         40.66000322733613],
        'borough': 'Queens',
        'name': 'Brookville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.73889198912481, 40.73301404027834],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.195',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bellaire',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.73889198912481,
         40.73301404027834,
         -73.73889198912481,
         40.73301404027834],
        'borough': 'Queens',
        'name': 'Bellaire',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85751790676447, 40.7540709990489],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.196',
       'properties': {'annoangle': 0.0,
        'annoline1': 'North',
        'annoline2': 'Corona',
        'annoline3': None,
        'bbox': [-73.85751790676447,
         40.7540709990489,
         -73.85751790676447,
         40.7540709990489],
        'borough': 'Queens',
        'name': 'North Corona',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.8410221123401, 40.7146110815117],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.197',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Forest',
        'annoline2': 'Hills',
        'annoline3': 'Gardens',
        'bbox': [-73.8410221123401,
         40.7146110815117,
         -73.8410221123401,
         40.7146110815117],
        'borough': 'Queens',
        'name': 'Forest Hills Gardens',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.07935312512797, 40.6449815710044],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.198',
       'properties': {'annoangle': 0.0,
        'annoline1': 'St.',
        'annoline2': 'George',
        'annoline3': None,
        'bbox': [-74.07935312512797,
         40.6449815710044,
         -74.07935312512797,
         40.6449815710044],
        'borough': 'Staten Island',
        'name': 'St. George',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08701650516625, 40.64061455913511],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.199',
       'properties': {'annoangle': 0.0,
        'annoline1': 'New',
        'annoline2': 'Brighton',
        'annoline3': None,
        'bbox': [-74.08701650516625,
         40.64061455913511,
         -74.08701650516625,
         40.64061455913511],
        'borough': 'Staten Island',
        'name': 'New Brighton',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.07790192660066, 40.62692762538176],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.200',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Stapleton',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.07790192660066,
         40.62692762538176,
         -74.07790192660066,
         40.62692762538176],
        'borough': 'Staten Island',
        'name': 'Stapleton',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.06980526716141, 40.61530494652761],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.201',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rosebank',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.06980526716141,
         40.61530494652761,
         -74.06980526716141,
         40.61530494652761],
        'borough': 'Staten Island',
        'name': 'Rosebank',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1071817826561, 40.63187892654607],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.202',
       'properties': {'annoangle': 0.0,
        'annoline1': 'West',
        'annoline2': 'Brighton',
        'annoline3': None,
        'bbox': [-74.1071817826561,
         40.63187892654607,
         -74.1071817826561,
         40.63187892654607],
        'borough': 'Staten Island',
        'name': 'West Brighton',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08724819983729, 40.624184791313006],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.203',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Grymes',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-74.08724819983729,
         40.624184791313006,
         -74.08724819983729,
         40.624184791313006],
        'borough': 'Staten Island',
        'name': 'Grymes Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1113288180088, 40.59706851814673],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.204',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Todt',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-74.1113288180088,
         40.59706851814673,
         -74.1113288180088,
         40.59706851814673],
        'borough': 'Staten Island',
        'name': 'Todt Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0795529253982, 40.58024741350956],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.205',
       'properties': {'annoangle': 0.0,
        'annoline1': 'South',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-74.0795529253982,
         40.58024741350956,
         -74.0795529253982,
         40.58024741350956],
        'borough': 'Staten Island',
        'name': 'South Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.12943426797008, 40.63366930554365],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.206',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Port',
        'annoline2': 'Richmond',
        'annoline3': None,
        'bbox': [-74.12943426797008,
         40.63366930554365,
         -74.12943426797008,
         40.63366930554365],
        'borough': 'Staten Island',
        'name': 'Port Richmond',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.15008537046981, 40.632546390481124],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.207',
       'properties': {'annoangle': 0.0,
        'annoline1': "Mariner's",
        'annoline2': 'Harbor',
        'annoline3': None,
        'bbox': [-74.15008537046981,
         40.632546390481124,
         -74.15008537046981,
         40.632546390481124],
        'borough': 'Staten Island',
        'name': "Mariner's Harbor",
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.17464532993542, 40.63968297845542],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.208',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Port',
        'annoline2': 'Ivory',
        'annoline3': None,
        'bbox': [-74.17464532993542,
         40.63968297845542,
         -74.17464532993542,
         40.63968297845542],
        'borough': 'Staten Island',
        'name': 'Port Ivory',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.11918058534842, 40.61333593766742],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.209',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Castleton',
        'annoline2': 'Corners',
        'annoline3': None,
        'bbox': [-74.11918058534842,
         40.61333593766742,
         -74.11918058534842,
         40.61333593766742],
        'borough': 'Staten Island',
        'name': 'Castleton Corners',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.16496031329827, 40.594252379161695],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.210',
       'properties': {'annoangle': 0.0,
        'annoline1': 'New',
        'annoline2': 'Springville',
        'annoline3': None,
        'bbox': [-74.16496031329827,
         40.594252379161695,
         -74.16496031329827,
         40.594252379161695],
        'borough': 'Staten Island',
        'name': 'New Springville',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.19073717538116, 40.58631375103281],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.211',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Travis',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.19073717538116,
         40.58631375103281,
         -74.19073717538116,
         40.58631375103281],
        'borough': 'Staten Island',
        'name': 'Travis',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1164794360638, 40.57257231820632],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.212',
       'properties': {'annoangle': 0.0,
        'annoline1': 'New',
        'annoline2': 'Dorp',
        'annoline3': None,
        'bbox': [-74.1164794360638,
         40.57257231820632,
         -74.1164794360638,
         40.57257231820632],
        'borough': 'Staten Island',
        'name': 'New Dorp',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.12156593771896, 40.5584622432888],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.213',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Oakwood',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.12156593771896,
         40.5584622432888,
         -74.12156593771896,
         40.5584622432888],
        'borough': 'Staten Island',
        'name': 'Oakwood',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.14932381490992, 40.549480228713605],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.214',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Great',
        'annoline2': 'Kills',
        'annoline3': None,
        'bbox': [-74.14932381490992,
         40.549480228713605,
         -74.14932381490992,
         40.549480228713605],
        'borough': 'Staten Island',
        'name': 'Great Kills',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1643308041936, 40.542230747450745],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.215',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Eltingville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.1643308041936,
         40.542230747450745,
         -74.1643308041936,
         40.542230747450745],
        'borough': 'Staten Island',
        'name': 'Eltingville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.17854866165878, 40.53811417474507],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.216',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Annadale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.17854866165878,
         40.53811417474507,
         -74.17854866165878,
         40.53811417474507],
        'borough': 'Staten Island',
        'name': 'Annadale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.20524582480326, 40.541967622888755],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.217',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Woodrow',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.20524582480326,
         40.541967622888755,
         -74.20524582480326,
         40.541967622888755],
        'borough': 'Staten Island',
        'name': 'Woodrow',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.24656934235283, 40.50533376115642],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.218',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Tottenville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.24656934235283,
         40.50533376115642,
         -74.24656934235283,
         40.50533376115642],
        'borough': 'Staten Island',
        'name': 'Tottenville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08055351790115, 40.637316067110326],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.219',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Tompkinsville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.08055351790115,
         40.637316067110326,
         -74.08055351790115,
         40.637316067110326],
        'borough': 'Staten Island',
        'name': 'Tompkinsville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.09629029235458, 40.61919310792676],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.220',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Silver',
        'annoline2': 'Lake',
        'annoline3': None,
        'bbox': [-74.09629029235458,
         40.61919310792676,
         -74.09629029235458,
         40.61919310792676],
        'borough': 'Staten Island',
        'name': 'Silver Lake',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0971255217853, 40.61276015756489],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.221',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sunnyside',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.0971255217853,
         40.61276015756489,
         -74.0971255217853,
         40.61276015756489],
        'borough': 'Staten Island',
        'name': 'Sunnyside',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96101312466779, 40.643675183340974],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.222',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Ditmas',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-73.96101312466779,
         40.643675183340974,
         -73.96101312466779,
         40.643675183340974],
        'borough': 'Brooklyn',
        'name': 'Ditmas Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93718680559314, 40.66094656188111],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.223',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Wingate',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93718680559314,
         40.66094656188111,
         -73.93718680559314,
         40.66094656188111],
        'borough': 'Brooklyn',
        'name': 'Wingate',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.92688212616955, 40.655572313280764],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.224',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rugby',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.92688212616955,
         40.655572313280764,
         -73.92688212616955,
         40.655572313280764],
        'borough': 'Brooklyn',
        'name': 'Rugby',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08015734936296, 40.60919044434558],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.225',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Park',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-74.08015734936296,
         40.60919044434558,
         -74.08015734936296,
         40.60919044434558],
        'borough': 'Staten Island',
        'name': 'Park Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.13304143951704, 40.62109047275409],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.226',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Westerleigh',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.13304143951704,
         40.62109047275409,
         -74.13304143951704,
         40.62109047275409],
        'borough': 'Staten Island',
        'name': 'Westerleigh',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.15315246387762, 40.620171512231884],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.227',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Graniteville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.15315246387762,
         40.620171512231884,
         -74.15315246387762,
         40.620171512231884],
        'borough': 'Staten Island',
        'name': 'Graniteville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.16510420241124, 40.63532509911492],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.228',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Arlington',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.16510420241124,
         40.63532509911492,
         -74.16510420241124,
         40.63532509911492],
        'borough': 'Staten Island',
        'name': 'Arlington',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.06712363225574, 40.596312571276734],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.229',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Arrochar',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.06712363225574,
         40.596312571276734,
         -74.06712363225574,
         40.596312571276734],
        'borough': 'Staten Island',
        'name': 'Arrochar',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.0766743627905, 40.59826835959991],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.230',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Grasmere',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.0766743627905,
         40.59826835959991,
         -74.0766743627905,
         40.59826835959991],
        'borough': 'Staten Island',
        'name': 'Grasmere',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08751118005578, 40.59632891379513],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.231',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Old',
        'annoline2': 'Town',
        'annoline3': None,
        'bbox': [-74.08751118005578,
         40.59632891379513,
         -74.08751118005578,
         40.59632891379513],
        'borough': 'Staten Island',
        'name': 'Old Town',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.09639905312521, 40.588672948199275],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.232',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Dongan',
        'annoline2': 'Hills',
        'annoline3': None,
        'bbox': [-74.09639905312521,
         40.588672948199275,
         -74.09639905312521,
         40.588672948199275],
        'borough': 'Staten Island',
        'name': 'Dongan Hills',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.09348266303591, 40.57352690574283],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.233',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Midland',
        'annoline2': 'Beach',
        'annoline3': None,
        'bbox': [-74.09348266303591,
         40.57352690574283,
         -74.09348266303591,
         40.57352690574283],
        'borough': 'Staten Island',
        'name': 'Midland Beach',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.10585598545434, 40.57621558711788],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.234',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Grant',
        'annoline2': 'City',
        'annoline3': None,
        'bbox': [-74.10585598545434,
         40.57621558711788,
         -74.10585598545434,
         40.57621558711788],
        'borough': 'Staten Island',
        'name': 'Grant City',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.10432707469124, 40.56425549307335],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.235',
       'properties': {'annoangle': 0.0,
        'annoline1': 'New',
        'annoline2': 'Dorp',
        'annoline3': 'Beach',
        'bbox': [-74.10432707469124,
         40.56425549307335,
         -74.10432707469124,
         40.56425549307335],
        'borough': 'Staten Island',
        'name': 'New Dorp Beach',
        'stacked': 3},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.13916622175768, 40.55398800858462],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.236',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bay',
        'annoline2': 'Terrace',
        'annoline3': None,
        'bbox': [-74.13916622175768,
         40.55398800858462,
         -74.13916622175768,
         40.55398800858462],
        'borough': 'Staten Island',
        'name': 'Bay Terrace',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.19174105747814, 40.531911920489605],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.237',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Huguenot',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.19174105747814,
         40.531911920489605,
         -74.19174105747814,
         40.531911920489605],
        'borough': 'Staten Island',
        'name': 'Huguenot',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.21983106616777, 40.524699376118136],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.238',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Pleasant',
        'annoline2': 'Plains',
        'annoline3': None,
        'bbox': [-74.21983106616777,
         40.524699376118136,
         -74.21983106616777,
         40.524699376118136],
        'borough': 'Staten Island',
        'name': 'Pleasant Plains',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.22950350260027, 40.50608165346305],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.239',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Butler',
        'annoline2': 'Manor',
        'annoline3': None,
        'bbox': [-74.22950350260027,
         40.50608165346305,
         -74.22950350260027,
         40.50608165346305],
        'borough': 'Staten Island',
        'name': 'Butler Manor',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.23215775896526, 40.53053148283314],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.240',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Charleston',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.23215775896526,
         40.53053148283314,
         -74.23215775896526,
         40.53053148283314],
        'borough': 'Staten Island',
        'name': 'Charleston',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.21572851113952, 40.54940400650072],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.241',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Rossville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.21572851113952,
         40.54940400650072,
         -74.21572851113952,
         40.54940400650072],
        'borough': 'Staten Island',
        'name': 'Rossville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.18588674583893, 40.54928582278321],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.242',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Arden',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-74.18588674583893,
         40.54928582278321,
         -74.18588674583893,
         40.54928582278321],
        'borough': 'Staten Island',
        'name': 'Arden Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.17079414786092, 40.555295236173194],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.243',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Greenridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.17079414786092,
         40.555295236173194,
         -74.17079414786092,
         40.555295236173194],
        'borough': 'Staten Island',
        'name': 'Greenridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.15902208156601, 40.58913894875281],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.244',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Heartland',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-74.15902208156601,
         40.58913894875281,
         -74.15902208156601,
         40.58913894875281],
        'borough': 'Staten Island',
        'name': 'Heartland Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1895604551969, 40.59472602746295],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.245',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Chelsea',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.1895604551969,
         40.59472602746295,
         -74.1895604551969,
         40.59472602746295],
        'borough': 'Staten Island',
        'name': 'Chelsea',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.18725638381567, 40.60577868452358],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.246',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bloomfield',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.18725638381567,
         40.60577868452358,
         -74.18725638381567,
         40.60577868452358],
        'borough': 'Staten Island',
        'name': 'Bloomfield',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.15940948657122, 40.6095918004203],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.247',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bulls',
        'annoline2': 'Head',
        'annoline3': None,
        'bbox': [-74.15940948657122,
         40.6095918004203,
         -74.15940948657122,
         40.6095918004203],
        'borough': 'Staten Island',
        'name': 'Bulls Head',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95325646837112, 40.7826825671257],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.248',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Carnegie',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.95325646837112,
         40.7826825671257,
         -73.95325646837112,
         40.7826825671257],
        'borough': 'Manhattan',
        'name': 'Carnegie Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98843368023597, 40.72325901885768],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.249',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Noho',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.98843368023597,
         40.72325901885768,
         -73.98843368023597,
         40.72325901885768],
        'borough': 'Manhattan',
        'name': 'Noho',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.00541529873355, 40.71522892046282],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.250',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Civic',
        'annoline2': 'Center',
        'annoline3': None,
        'bbox': [-74.00541529873355,
         40.71522892046282,
         -74.00541529873355,
         40.71522892046282],
        'borough': 'Manhattan',
        'name': 'Civic Center',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98871313285247, 40.7485096643122],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.251',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Midtown',
        'annoline2': 'South',
        'annoline3': None,
        'bbox': [-73.98871313285247,
         40.7485096643122,
         -73.98871313285247,
         40.7485096643122],
        'borough': 'Manhattan',
        'name': 'Midtown South',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1340572986257, 40.56960594275505],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.252',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Richmond',
        'annoline2': 'Town',
        'annoline3': None,
        'bbox': [-74.1340572986257,
         40.56960594275505,
         -74.1340572986257,
         40.56960594275505],
        'borough': 'Staten Island',
        'name': 'Richmond Town',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.06667766061771, 40.60971934079284],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.253',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Shore',
        'annoline2': 'Acres',
        'annoline3': None,
        'bbox': [-74.06667766061771,
         40.60971934079284,
         -74.06667766061771,
         40.60971934079284],
        'borough': 'Staten Island',
        'name': 'Shore Acres',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.072642445484, 40.61917845202843],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.254',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Clifton',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.072642445484,
         40.61917845202843,
         -74.072642445484,
         40.61917845202843],
        'borough': 'Staten Island',
        'name': 'Clifton',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08402364740358, 40.6044731896879],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.255',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Concord',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.08402364740358,
         40.6044731896879,
         -74.08402364740358,
         40.6044731896879],
        'borough': 'Staten Island',
        'name': 'Concord',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.09776206972522, 40.606794394801],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.256',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Emerson',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-74.09776206972522,
         40.606794394801,
         -74.09776206972522,
         40.606794394801],
        'borough': 'Staten Island',
        'name': 'Emerson Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.09805062373887, 40.63563000681151],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.257',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Randall',
        'annoline2': 'Manor',
        'annoline3': None,
        'bbox': [-74.09805062373887,
         40.63563000681151,
         -74.09805062373887,
         40.63563000681151],
        'borough': 'Staten Island',
        'name': 'Randall Manor',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.18622331749823, 40.63843283794795],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.258',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Howland',
        'annoline2': 'Hook',
        'annoline3': None,
        'bbox': [-74.18622331749823,
         40.63843283794795,
         -74.18622331749823,
         40.63843283794795],
        'borough': 'Staten Island',
        'name': 'Howland Hook',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.1418167896889, 40.630146741193826],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.259',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Elm',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-74.1418167896889,
         40.630146741193826,
         -74.1418167896889,
         40.630146741193826],
        'borough': 'Staten Island',
        'name': 'Elm Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91665331978048, 40.652117451793494],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.260',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Remsen',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.91665331978048,
         40.652117451793494,
         -73.91665331978048,
         40.652117451793494],
        'borough': 'Brooklyn',
        'name': 'Remsen Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88511776379292, 40.6627442796966],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.261',
       'properties': {'annoangle': 0.0,
        'annoline1': 'New',
        'annoline2': 'Lots',
        'annoline3': None,
        'bbox': [-73.88511776379292,
         40.6627442796966,
         -73.88511776379292,
         40.6627442796966],
        'borough': 'Brooklyn',
        'name': 'New Lots',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90233474295836, 40.63131755039667],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.262',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Paerdegat',
        'annoline2': 'Basin',
        'annoline3': None,
        'bbox': [-73.90233474295836,
         40.63131755039667,
         -73.90233474295836,
         40.63131755039667],
        'borough': 'Brooklyn',
        'name': 'Paerdegat Basin',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91515391550404, 40.61597423962336],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.263',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Mill',
        'annoline2': 'Basin',
        'annoline3': None,
        'bbox': [-73.91515391550404,
         40.61597423962336,
         -73.91515391550404,
         40.61597423962336],
        'borough': 'Brooklyn',
        'name': 'Mill Basin',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79646462081593, 40.71145964370482],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.264',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Jamaica',
        'annoline2': 'Hills',
        'annoline3': None,
        'bbox': [-73.79646462081593,
         40.71145964370482,
         -73.79646462081593,
         40.71145964370482],
        'borough': 'Queens',
        'name': 'Jamaica Hills',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.79671678028349, 40.73350025429757],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.265',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Utopia',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.79671678028349,
         40.73350025429757,
         -73.79671678028349,
         40.73350025429757],
        'borough': 'Queens',
        'name': 'Utopia',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.80486120040537, 40.73493618075478],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.266',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Pomonok',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.80486120040537,
         40.73493618075478,
         -73.80486120040537,
         40.73493618075478],
        'borough': 'Queens',
        'name': 'Pomonok',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.89467996270574, 40.7703173929982],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.267',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Astoria',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.89467996270574,
         40.7703173929982,
         -73.89467996270574,
         40.7703173929982],
        'borough': 'Queens',
        'name': 'Astoria Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90119903387667, 40.83142834161548],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.268',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Claremont',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.90119903387667,
         40.83142834161548,
         -73.90119903387667,
         40.83142834161548],
        'borough': 'Bronx',
        'name': 'Claremont Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91584652759009, 40.824780490842905],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.269',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Concourse',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.91584652759009,
         40.824780490842905,
         -73.91584652759009,
         40.824780490842905],
        'borough': 'Bronx',
        'name': 'Concourse Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91655551964419, 40.84382617671654],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.270',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Mount',
        'annoline2': 'Eden',
        'annoline3': None,
        'bbox': [-73.91655551964419,
         40.84382617671654,
         -73.91655551964419,
         40.84382617671654],
        'borough': 'Bronx',
        'name': 'Mount Eden',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90829930881988, 40.84884160724665],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.271',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Mount',
        'annoline2': 'Hope',
        'annoline3': None,
        'bbox': [-73.90829930881988,
         40.84884160724665,
         -73.90829930881988,
         40.84884160724665],
        'borough': 'Bronx',
        'name': 'Mount Hope',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96355614094303, 40.76028033131374],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.272',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sutton',
        'annoline2': 'Place',
        'annoline3': None,
        'bbox': [-73.96355614094303,
         40.76028033131374,
         -73.96355614094303,
         40.76028033131374],
        'borough': 'Manhattan',
        'name': 'Sutton Place',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95386782130745, 40.743414090073536],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.273',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hunters',
        'annoline2': 'Point',
        'annoline3': None,
        'bbox': [-73.95386782130745,
         40.743414090073536,
         -73.95386782130745,
         40.743414090073536],
        'borough': 'Queens',
        'name': 'Hunters Point',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.96770824581834, 40.75204236950722],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.274',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Turtle',
        'annoline2': 'Bay',
        'annoline3': None,
        'bbox': [-73.96770824581834,
         40.75204236950722,
         -73.96770824581834,
         40.75204236950722],
        'borough': 'Manhattan',
        'name': 'Turtle Bay',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97121928722265, 40.746917410740195],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.275',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Tudor',
        'annoline2': 'City',
        'annoline3': None,
        'bbox': [-73.97121928722265,
         40.746917410740195,
         -73.97121928722265,
         40.746917410740195],
        'borough': 'Manhattan',
        'name': 'Tudor City',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.97405170469203, 40.73099955477061],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.276',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Stuyvesant',
        'annoline2': 'Town',
        'annoline3': None,
        'bbox': [-73.97405170469203,
         40.73099955477061,
         -73.97405170469203,
         40.73099955477061],
        'borough': 'Manhattan',
        'name': 'Stuyvesant Town',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9909471052826, 40.739673047638426],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.277',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Flatiron',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.9909471052826,
         40.739673047638426,
         -73.9909471052826,
         40.739673047638426],
        'borough': 'Manhattan',
        'name': 'Flatiron',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.91819286431682, 40.74565180608076],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.278',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sunnyside',
        'annoline2': 'Gardens',
        'annoline3': None,
        'bbox': [-73.91819286431682,
         40.74565180608076,
         -73.91819286431682,
         40.74565180608076],
        'borough': 'Queens',
        'name': 'Sunnyside Gardens',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93244235260178, 40.73725071694497],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.279',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Blissville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93244235260178,
         40.73725071694497,
         -73.93244235260178,
         40.73725071694497],
        'borough': 'Queens',
        'name': 'Blissville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.99550751888415, 40.70328109093014],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.280',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fulton',
        'annoline2': 'Ferry',
        'annoline3': None,
        'bbox': [-73.99550751888415,
         40.70328109093014,
         -73.99550751888415,
         40.70328109093014],
        'borough': 'Brooklyn',
        'name': 'Fulton Ferry',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.98111603592393, 40.70332149882874],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.281',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Vinegar',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-73.98111603592393,
         40.70332149882874,
         -73.98111603592393,
         40.70332149882874],
        'borough': 'Brooklyn',
        'name': 'Vinegar Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.93053108817338, 40.67503986503237],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.282',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Weeksville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.93053108817338,
         40.67503986503237,
         -73.93053108817338,
         40.67503986503237],
        'borough': 'Brooklyn',
        'name': 'Weeksville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90331684852599, 40.67786104769531],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.283',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Broadway',
        'annoline2': 'Junction',
        'annoline3': None,
        'bbox': [-73.90331684852599,
         40.67786104769531,
         -73.90331684852599,
         40.67786104769531],
        'borough': 'Brooklyn',
        'name': 'Broadway Junction',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.9887528074504, 40.70317632822692],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.284',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Dumbo',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.9887528074504,
         40.70317632822692,
         -73.9887528074504,
         40.70317632822692],
        'borough': 'Brooklyn',
        'name': 'Dumbo',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.12059399718001, 40.60180957631444],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.285',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Manor',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-74.12059399718001,
         40.60180957631444,
         -74.12059399718001,
         40.60180957631444],
        'borough': 'Staten Island',
        'name': 'Manor Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.13208447484298, 40.60370692627371],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.286',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Willowbrook',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.13208447484298,
         40.60370692627371,
         -74.13208447484298,
         40.60370692627371],
        'borough': 'Staten Island',
        'name': 'Willowbrook',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.21776636068567, 40.541139922091766],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.287',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Sandy',
        'annoline2': 'Ground',
        'annoline3': None,
        'bbox': [-74.21776636068567,
         40.541139922091766,
         -74.21776636068567,
         40.541139922091766],
        'borough': 'Staten Island',
        'name': 'Sandy Ground',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.12727240604946, 40.579118742961214],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.288',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Egbertville',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-74.12727240604946,
         40.579118742961214,
         -74.12727240604946,
         40.579118742961214],
        'borough': 'Staten Island',
        'name': 'Egbertville',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.89213760232822, 40.56737588957032],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.289',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Roxbury',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.89213760232822,
         40.56737588957032,
         -73.89213760232822,
         40.56737588957032],
        'borough': 'Queens',
        'name': 'Roxbury',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.95918459428702, 40.598525095137255],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.290',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Homecrest',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.95918459428702,
         40.598525095137255,
         -73.95918459428702,
         40.598525095137255],
        'borough': 'Brooklyn',
        'name': 'Homecrest',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.88114319200604, 40.716414511158185],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.291',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Middle',
        'annoline2': 'Village',
        'annoline3': None,
        'bbox': [-73.88114319200604,
         40.716414511158185,
         -73.88114319200604,
         40.716414511158185],
        'borough': 'Queens',
        'name': 'Middle Village',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.20152556457658, 40.52626406734812],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.292',
       'properties': {'annoangle': 0.0,
        'annoline1': "Prince's",
        'annoline2': 'Bay',
        'annoline3': None,
        'bbox': [-74.20152556457658,
         40.52626406734812,
         -74.20152556457658,
         40.52626406734812],
        'borough': 'Staten Island',
        'name': "Prince's Bay",
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.13792663771568, 40.57650629379489],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.293',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Lighthouse',
        'annoline2': 'Hill',
        'annoline3': None,
        'bbox': [-74.13792663771568,
         40.57650629379489,
         -74.13792663771568,
         40.57650629379489],
        'borough': 'Staten Island',
        'name': 'Lighthouse Hill',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.22957080626941, 40.51954145748909],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.294',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Richmond',
        'annoline2': 'Valley',
        'annoline3': None,
        'bbox': [-74.22957080626941,
         40.51954145748909,
         -74.22957080626941,
         40.51954145748909],
        'borough': 'Staten Island',
        'name': 'Richmond Valley',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.82667757138641, 40.79060155670148],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.295',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Malba',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.82667757138641,
         40.79060155670148,
         -73.82667757138641,
         40.79060155670148],
        'borough': 'Queens',
        'name': 'Malba',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.890345709872, 40.6819989345173],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.296',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Highland',
        'annoline2': 'Park',
        'annoline3': None,
        'bbox': [-73.890345709872,
         40.6819989345173,
         -73.890345709872,
         40.6819989345173],
        'borough': 'Brooklyn',
        'name': 'Highland Park',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94841515328893, 40.60937770113766],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.297',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Madison',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94841515328893,
         40.60937770113766,
         -73.94841515328893,
         40.60937770113766],
        'borough': 'Brooklyn',
        'name': 'Madison',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.86172577555115, 40.85272297633017],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.298',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bronxdale',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.86172577555115,
         40.85272297633017,
         -73.86172577555115,
         40.85272297633017],
        'borough': 'Bronx',
        'name': 'Bronxdale',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.85931863221647, 40.86578787802982],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.299',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Allerton',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.85931863221647,
         40.86578787802982,
         -73.85931863221647,
         40.86578787802982],
        'borough': 'Bronx',
        'name': 'Allerton',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.90152264513144, 40.8703923914147],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.300',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Kingsbridge',
        'annoline2': 'Heights',
        'annoline3': None,
        'bbox': [-73.90152264513144,
         40.8703923914147,
         -73.90152264513144,
         40.8703923914147],
        'borough': 'Bronx',
        'name': 'Kingsbridge Heights',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94817709920184, 40.64692606658579],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.301',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Erasmus',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94817709920184,
         40.64692606658579,
         -73.94817709920184,
         40.64692606658579],
        'borough': 'Brooklyn',
        'name': 'Erasmus',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.00011136202637, 40.75665808227519],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.302',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hudson',
        'annoline2': 'Yards',
        'annoline3': None,
        'bbox': [-74.00011136202637,
         40.75665808227519,
         -74.00011136202637,
         40.75665808227519],
        'borough': 'Manhattan',
        'name': 'Hudson Yards',
        'stacked': 2},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.80553002968718, 40.58733774018741],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.303',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Hammels',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.80553002968718,
         40.58733774018741,
         -73.80553002968718,
         40.58733774018741],
        'borough': 'Queens',
        'name': 'Hammels',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.76596781445627, 40.611321691283834],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.304',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Bayswater',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.76596781445627,
         40.611321691283834,
         -73.76596781445627,
         40.611321691283834],
        'borough': 'Queens',
        'name': 'Bayswater',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-73.94563070334091, 40.756091297094706],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.305',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Queensbridge',
        'annoline2': None,
        'annoline3': None,
        'bbox': [-73.94563070334091,
         40.756091297094706,
         -73.94563070334091,
         40.756091297094706],
        'borough': 'Queens',
        'name': 'Queensbridge',
        'stacked': 1},
       'type': 'Feature'},
      {'geometry': {'coordinates': [-74.08173992211962, 40.61731079252983],
        'type': 'Point'},
       'geometry_name': 'geom',
       'id': 'nyu_2451_34572.306',
       'properties': {'annoangle': 0.0,
        'annoline1': 'Fox',
        'annoline2': 'Hills',
        'annoline3': None,
        'bbox': [-74.08173992211962,
         40.61731079252983,
         -74.08173992211962,
         40.61731079252983],
        'borough': 'Staten Island',
        'name': 'Fox Hills',
        'stacked': 2},
       'type': 'Feature'}],
     'totalFeatures': 306,
     'type': 'FeatureCollection'}



All the relevant data is in the *features* key, which is basically a list of the neighborhoods.  
Let's define a new variable that includes this data.


```python
neighborhoods_data = newyork_data['features']
```

Let's check the first item in the list.


```python
neighborhoods_data[0]
```




    {'geometry': {'coordinates': [-73.84720052054902, 40.89470517661],
      'type': 'Point'},
     'geometry_name': 'geom',
     'id': 'nyu_2451_34572.1',
     'properties': {'annoangle': 0.0,
      'annoline1': 'Wakefield',
      'annoline2': None,
      'annoline3': None,
      'bbox': [-73.84720052054902,
       40.89470517661,
       -73.84720052054902,
       40.89470517661],
      'borough': 'Bronx',
      'name': 'Wakefield',
      'stacked': 1},
     'type': 'Feature'}



### Transform the data into a pandas dataframe
We will essentially be transforming this data of nested Python dictionaries into a pandas dataframe.  
Let's start by creating an empty dataframe.


```python
# define the dataframe columns
column_names = ['Borough', 'Neighborhood', 'Latitude', 'Longitude']

# instantiate the dataframe
neighborhoods = pd.DataFrame(columns=column_names)
```


```python
# empty dataframe
neighborhoods
```




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
      <th>Borough</th>
      <th>Neighborhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



Let's loop through the data and fill the dataframe one row at a time.


```python
for data in neighborhoods_data:
  borough = neighborhood_name = data['properties']['borough']
  neighborhood_name = data['properties']['name']

  neighborhood_latlon = data['geometry']['coordinates']
  neighborhood_lat = neighborhood_latlon[1]
  neighborhood_lon = neighborhood_latlon[0]

  neighborhoods = neighborhoods.append({'Borough': borough, 
                                        'Neighborhood': neighborhood_name,
                                        'Latitude': neighborhood_lat,
                                        'Longitude': neighborhood_lon}, ignore_index=True)
```


```python
# examine the dataframe
neighborhoods.head()
```




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
      <th>Borough</th>
      <th>Neighborhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bronx</td>
      <td>Wakefield</td>
      <td>40.894705</td>
      <td>-73.847201</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bronx</td>
      <td>Co-op City</td>
      <td>40.874294</td>
      <td>-73.829939</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bronx</td>
      <td>Eastchester</td>
      <td>40.887556</td>
      <td>-73.827806</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bronx</td>
      <td>Fieldston</td>
      <td>40.895437</td>
      <td>-73.905643</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bronx</td>
      <td>Riverdale</td>
      <td>40.890834</td>
      <td>-73.912585</td>
    </tr>
  </tbody>
</table>
</div>




```python
# check whether the dataframe has all 5 boroughs and 306 neighborhoods
print('The dataframe has {} boroughs and {} neighborhoods.'.format(
    len(neighborhoods['Borough'].unique()), neighborhoods.shape[0]))
```

    The dataframe has 5 boroughs and 306 neighborhoods.
    

### Visualize the data

We will use the geopy library to get the latitude and longitude values of New York City.  
In order to define an instance of the geocoder, we need to define a user_agent.


```python
address = 'New York City, NY'

geolocator = Nominatim(user_agent='ny_explorer')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geographical coordinates of New York City are {}, {}.'.format(latitude, longitude))
```

    The geographical coordinates of New York City are 40.7127281, -74.0060152.
    

Let's create a map of New York with the neighborhoods superimposed on top.


```python
# create map of New York using latitude and longitude values
map_newyork = folium.Map(location=[latitude, longitude], zoom_start=11)

# add markers to map
for lat, lng, borough, neighborhood in zip(neighborhoods['Latitude'], neighborhoods['Longitude'], neighborhoods['Borough'], neighborhoods['Neighborhood']):
    label = '{}, {}'.format(neighborhood, borough)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_newyork)  
    
map_newyork
```



<img src="/assets/img/clustering neighborhoods/nyc/map1.JPG">



If you were working in a notebook and using Folium, you could zoom into the above map, and click on each circle mark to reveal the name of the neighborhood and its respective borough.

Let's simplify the above map, and segment and cluster only the neighborhoods in Manhattan. We will slice the original dataframe and create a new dataframe of the Manhattan data.


```python
manhattan_data = neighborhoods[neighborhoods['Borough'] == 'Manhattan'].reset_index(drop=True)
manhattan_data.head()
```




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
      <th>Borough</th>
      <th>Neighborhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Manhattan</td>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.910660</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Manhattan</td>
      <td>Chinatown</td>
      <td>40.715618</td>
      <td>-73.994279</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Manhattan</td>
      <td>Washington Heights</td>
      <td>40.851903</td>
      <td>-73.936900</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Manhattan</td>
      <td>Inwood</td>
      <td>40.867684</td>
      <td>-73.921210</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Manhattan</td>
      <td>Hamilton Heights</td>
      <td>40.823604</td>
      <td>-73.949688</td>
    </tr>
  </tbody>
</table>
</div>



Let's get the geographiical coordinates of Manhattan.


```python
address = 'Manhattan, NY'

geolocator = Nominatim(user_agent='ny_explorer')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geographical coordinates of Manhattan are {}, {}.'.format(latitude, longitude))
```

    The geographical coordinates of Manhattan are 40.7896239, -73.9598939.
    

Let's visualize the neighborhoods in Manhattan.


```python
# create a map of Manhattan using latitude and longitude values
map_manhattan = folium.Map(location=[latitude, longitude], zoom_start=12)

# add markers to map
for lat, lng, label in zip(manhattan_data['Latitude'], manhattan_data['Longitude'], manhattan_data['Neighborhood']):
  label = folium.Popup(label, parse_html=True)
  folium.CircleMarker(
      [lat, lng],
      radius=5,
      popup=label,
      color='blue',
      fill=True,
      fill_color='#3186cc',
      fill_opacity=0.7,
      parse_html=False).add_to(map_manhattan)

map_manhattan
```


<img src = "/assets/img/clustering neighborhoods/nyc/map2.JPG">



### Explore the neighborhoods in Manhattan
We will utilize the Foursquare API to explore the neighborhoods and segment them.


```python
# define Foursquare Credentials
CLIENT_ID = '**********' 
CLIENT_SECRET = '**********'
VERSION = '20191219'
```

#### Explore the first neighborhood in our dataframe.


```python
# get the neighborhood's name
manhattan_data.loc[0, 'Neighborhood']
```




    'Marble Hill'




```python
# get the neighborhood's latitude and longitude values
neighborhood_latitude = manhattan_data.loc[0, 'Latitude'] # neighborhood latitude value
neighborhood_longitude = manhattan_data.loc[0,'Longitude'] #neighborhood longitude value

neighborhood_name = manhattan_data.loc[0, 'Neighborhood'] # neighborhood name

print('The geographical coordinates of {} are {}, {}.'.format(neighborhood_name, neighborhood_latitude, neighborhood_longitude))
```

    The geographical coordinates of Marble Hill are 40.87655077879964, -73.91065965862981.
    

Let's get the top 100 venues that are in Marble Hill within a radius of 500 meters.  
First, create the GET request URL.


```python
# limit the number of venues returned by the foursquare API
LIMIT = 100

# define radius
radius = 500

# Create URL
url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
    CLIENT_ID, CLIENT_SECRET, VERSION, neighborhood_latitude, neighborhood_longitude, radius, LIMIT)

# display URL
url
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=B50RYBOWHJ3ZMRVEFTLHJCXGOYNCXGI13VDT5FMYTUQSUTQC&client_secret=ST2IN4ZGOQ3BXEWJ2HW1LKUML2BBGK1JX1QD2WKEPET31W4W&v=20191219&ll=40.87655077879964,-73.91065965862981&radius=500&limit=100'



Send the GET request and examine the results.


```python
results = requests.get(url).json()
results
```




    {'meta': {'code': 200, 'requestId': '5e68ce37618f43001c2e7a8e'},
     'response': {'groups': [{'items': [{'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b4429abf964a52037f225e3-0',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1ca941735',
             'name': 'Pizza Place',
             'pluralName': 'Pizza Places',
             'primary': True,
             'shortName': 'Pizza'}],
           'delivery': {'id': '72548',
            'provider': {'icon': {'name': '/delivery_provider_seamless_20180129.png',
              'prefix': 'https://fastly.4sqi.net/img/general/cap/',
              'sizes': [40, 50]},
             'name': 'seamless'},
            'url': 'https://www.seamless.com/menu/arturos-pizza-5189-broadway-ave-new-york/72548?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=72548'},
           'id': '4b4429abf964a52037f225e3',
           'location': {'address': '5198 Broadway',
            'cc': 'US',
            'city': 'New York',
            'country': 'United States',
            'crossStreet': 'at 225th St.',
            'distance': 240,
            'formattedAddress': ['5198 Broadway (at 225th St.)',
             'New York, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87441177110231,
              'lng': -73.91027100981574}],
            'lat': 40.87441177110231,
            'lng': -73.91027100981574,
            'postalCode': '10463',
            'state': 'NY'},
           'name': "Arturo's",
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4baf59e8f964a520a6f93be3-1',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/gym_yogastudio_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d102941735',
             'name': 'Yoga Studio',
             'pluralName': 'Yoga Studios',
             'primary': True,
             'shortName': 'Yoga Studio'}],
           'id': '4baf59e8f964a520a6f93be3',
           'location': {'address': '5500 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': '230th Street',
            'distance': 376,
            'formattedAddress': ['5500 Broadway (230th Street)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.876843690797934,
              'lng': -73.90620384419528}],
            'lat': 40.876843690797934,
            'lng': -73.90620384419528,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Bikram Yoga',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b79cc46f964a520c5122fe3-2',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/diner_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d147941735',
             'name': 'Diner',
             'pluralName': 'Diners',
             'primary': True,
             'shortName': 'Diner'}],
           'id': '4b79cc46f964a520c5122fe3',
           'location': {'address': '3033 Tibbett Ave',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': 'btwn 230th & 231st',
            'distance': 452,
            'formattedAddress': ['3033 Tibbett Ave (btwn 230th & 231st)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.8804044222466,
              'lng': -73.90893738006402}],
            'lat': 40.8804044222466,
            'lng': -73.90893738006402,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Tibbett Diner',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b5357adf964a520319827e3-3',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/donuts_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d148941735',
             'name': 'Donut Shop',
             'pluralName': 'Donut Shops',
             'primary': True,
             'shortName': 'Donuts'}],
           'id': '4b5357adf964a520319827e3',
           'location': {'address': '5501 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': 'W 230th St',
            'distance': 342,
            'formattedAddress': ['5501 Broadway (W 230th St)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87713584201589,
              'lng': -73.90666550701411}],
            'lat': 40.87713584201589,
            'lng': -73.90666550701411,
            'postalCode': '10463',
            'state': 'NY'},
           'name': "Dunkin'",
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-55f81cd2498ee903149fcc64-4',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1e0931735',
             'name': 'Coffee Shop',
             'pluralName': 'Coffee Shops',
             'primary': True,
             'shortName': 'Coffee Shop'}],
           'id': '55f81cd2498ee903149fcc64',
           'location': {'address': '171 W 230th St',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': 'Kimberly Pl',
            'distance': 441,
            'formattedAddress': ['171 W 230th St (Kimberly Pl)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87753134921497,
              'lng': -73.90558216359267}],
            'lat': 40.87753134921497,
            'lng': -73.90558216359267,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Starbucks',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4cf6ae55d3a8a1cd71a9d243-5',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/gym_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d176941735',
             'name': 'Gym',
             'pluralName': 'Gyms',
             'primary': True,
             'shortName': 'Gym'}],
           'id': '4cf6ae55d3a8a1cd71a9d243',
           'location': {'address': '5500 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': '230th St',
            'distance': 361,
            'formattedAddress': ['5500 Broadway (230th St)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87670526507199,
              'lng': -73.90637207670373}],
            'lat': 40.87670526507199,
            'lng': -73.90637207670373,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Astral Fitness & Wellness Center',
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '75803748'}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-55f751ca498eacc0307d1cfe-6',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/gym_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d176941735',
             'name': 'Gym',
             'pluralName': 'Gyms',
             'primary': True,
             'shortName': 'Gym'}],
           'id': '55f751ca498eacc0307d1cfe',
           'location': {'address': '5520 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': 'at W 230th St',
            'distance': 433,
            'formattedAddress': ['5520 Broadway (at W 230th St)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.877271495944626,
              'lng': -73.90559491338075}],
            'lat': 40.877271495944626,
            'lng': -73.90559491338075,
            'neighborhood': 'Kingsbridge',
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Blink Fitness',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-546d31ca498e561c698a0320-7',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/departmentstore_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1f6941735',
             'name': 'Department Store',
             'pluralName': 'Department Stores',
             'primary': True,
             'shortName': 'Department Store'}],
           'id': '546d31ca498e561c698a0320',
           'location': {'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 478,
            'formattedAddress': ['Bronx, NY', 'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87723198343352,
              'lng': -73.90504239962168}],
            'lat': 40.87723198343352,
            'lng': -73.90504239962168,
            'state': 'NY'},
           'name': 'T.J. Maxx',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b9c9c6af964a520b27236e3-8',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/seafood_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1ce941735',
             'name': 'Seafood Restaurant',
             'pluralName': 'Seafood Restaurants',
             'primary': True,
             'shortName': 'Seafood'}],
           'delivery': {'id': '277380',
            'provider': {'icon': {'name': '/delivery_provider_seamless_20180129.png',
              'prefix': 'https://fastly.4sqi.net/img/general/cap/',
              'sizes': [40, 50]},
             'name': 'seamless'},
            'url': 'https://www.seamless.com/menu/land--sea-restaurant-5535-broadway-ave-bronx/277380?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=277380'},
           'id': '4b9c9c6af964a520b27236e3',
           'location': {'address': '5535 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': '231st St',
            'distance': 429,
            'formattedAddress': ['5535 Broadway (231st St)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87788463309788,
              'lng': -73.90587282193539}],
            'lat': 40.87788463309788,
            'lng': -73.90587282193539,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Land & Sea Restaurant',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4a725fa1f964a520f6da1fe3-9',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/arts_entertainment/stadium_tennis_',
              'suffix': '.png'},
             'id': '4e39a891bd410d7aed40cbc2',
             'name': 'Tennis Stadium',
             'pluralName': 'Tennis Stadiums',
             'primary': True,
             'shortName': 'Tennis'}],
           'id': '4a725fa1f964a520f6da1fe3',
           'location': {'address': '2600 Netherland Ave',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 402,
            'formattedAddress': ['2600 Netherland Ave',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.8786283,
              'lng': -73.9145678}],
            'lat': 40.8786283,
            'lng': -73.9145678,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'TCR The Club of Riverdale',
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '40358759'}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b88e053f964a5208a1132e3-10',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pharmacy_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d10f951735',
             'name': 'Pharmacy',
             'pluralName': 'Pharmacies',
             'primary': True,
             'shortName': 'Pharmacy'}],
           'id': '4b88e053f964a5208a1132e3',
           'location': {'address': '5237 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': '228th Street',
            'distance': 190,
            'formattedAddress': ['5237 Broadway (228th Street)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.875466574434704,
              'lng': -73.90890629016033}],
            'lat': 40.875466574434704,
            'lng': -73.90890629016033,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Rite Aid',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-57655be738faa66160da7527-11',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1e0931735',
             'name': 'Coffee Shop',
             'pluralName': 'Coffee Shops',
             'primary': True,
             'shortName': 'Coffee Shop'}],
           'id': '57655be738faa66160da7527',
           'location': {'address': '50 W 225th St',
            'cc': 'US',
            'city': 'New York',
            'country': 'United States',
            'distance': 355,
            'formattedAddress': ['50 W 225th St',
             'New York, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.873754554218515,
              'lng': -73.90861305343668}],
            'lat': 40.873754554218515,
            'lng': -73.90861305343668,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Starbucks',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4b9c9c43f964a520ac7236e3-12',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/discountstore_',
              'suffix': '.png'},
             'id': '52dea92d3cf9994f4e043dbb',
             'name': 'Discount Store',
             'pluralName': 'Discount Stores',
             'primary': True,
             'shortName': 'Discount Store'}],
           'id': '4b9c9c43f964a520ac7236e3',
           'location': {'address': '5545 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'crossStreet': '231st St',
            'distance': 492,
            'formattedAddress': ['5545 Broadway (231st St)',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.878270422202085,
              'lng': -73.9052646742604}],
            'lat': 40.878270422202085,
            'lng': -73.9052646742604,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Lot Less Closeouts',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-5631194e498e2de074de661c-13',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/education/lab_',
              'suffix': '.png'},
             'id': '5744ccdfe4b0c0459246b4cd',
             'name': 'Supplement Shop',
             'pluralName': 'Supplement Shops',
             'primary': True,
             'shortName': 'Supplement Shop'}],
           'id': '5631194e498e2de074de661c',
           'location': {'address': '5510 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 428,
            'formattedAddress': ['5510 Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87716,
              'lng': -73.905632}],
            'lat': 40.87716,
            'lng': -73.905632,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Vitamin Shoppe',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4e4e4517bd4101d0d7a67568-14',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/icecream_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1c9941735',
             'name': 'Ice Cream Shop',
             'pluralName': 'Ice Cream Shops',
             'primary': True,
             'shortName': 'Ice Cream'}],
           'id': '4e4e4517bd4101d0d7a67568',
           'location': {'address': '5501 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 343,
            'formattedAddress': ['5501 Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87714929478882,
              'lng': -73.90665810372622}],
            'lat': 40.87714929478882,
            'lng': -73.90665810372622,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Baskin-Robbins',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-56229ff8498e2abb44b6f12b-15',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/default_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1ff941735',
             'name': 'Miscellaneous Shop',
             'pluralName': 'Miscellaneous Shops',
             'primary': True,
             'shortName': 'Shop'}],
           'id': '56229ff8498e2abb44b6f12b',
           'location': {'address': '171 W 230th St',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 487,
            'formattedAddress': ['171 W 230th St',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.8772564,
              'lng': -73.9049384}],
            'lat': 40.8772564,
            'lng': -73.9049384,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Five Below',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-585c205665e7c70a2f1055ea-16',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d14e941735',
             'name': 'American Restaurant',
             'pluralName': 'American Restaurants',
             'primary': True,
             'shortName': 'American'}],
           'delivery': {'id': '1436334',
            'provider': {'icon': {'name': '/delivery_provider_seamless_20180129.png',
              'prefix': 'https://fastly.4sqi.net/img/general/cap/',
              'sizes': [40, 50]},
             'name': 'seamless'},
            'url': 'https://www.seamless.com/menu/boston-market-5520-broadway-bronx/1436334?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=1436334'},
           'id': '585c205665e7c70a2f1055ea',
           'location': {'address': '5520 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 452,
            'formattedAddress': ['5520 Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87743,
              'lng': -73.9054121}],
            'lat': 40.87743,
            'lng': -73.9054121,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Boston Market',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4e4ce4debd413c4cc66d05d0-17',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1c5941735',
             'name': 'Sandwich Place',
             'pluralName': 'Sandwich Places',
             'primary': True,
             'shortName': 'Sandwiches'}],
           'delivery': {'id': '774886',
            'provider': {'icon': {'name': '/delivery_provider_seamless_20180129.png',
              'prefix': 'https://fastly.4sqi.net/img/general/cap/',
              'sizes': [40, 50]},
             'name': 'seamless'},
            'url': 'https://www.seamless.com/menu/subway-5549-broadway-bronx/774886?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=774886'},
           'id': '4e4ce4debd413c4cc66d05d0',
           'location': {'address': '5549 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 482,
            'formattedAddress': ['5549 Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.878464979323944,
              'lng': -73.9055176422437}],
            'lat': 40.878464979323944,
            'lng': -73.9055176422437,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'SUBWAY',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4ec68016cc21b428e1d2060a-18',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/financial_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d10a951735',
             'name': 'Bank',
             'pluralName': 'Banks',
             'primary': True,
             'shortName': 'Bank'}],
           'id': '4ec68016cc21b428e1d2060a',
           'location': {'address': '281 W 230th St',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 347,
            'formattedAddress': ['281 W 230th St',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.8794958,
              'lng': -73.9092856}],
            'lat': 40.8794958,
            'lng': -73.9092856,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'TD Bank',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4dfe40df8877333e195b68fc-19',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/steakhouse_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1cc941735',
             'name': 'Steakhouse',
             'pluralName': 'Steakhouses',
             'primary': True,
             'shortName': 'Steakhouse'}],
           'delivery': {'id': '330981',
            'provider': {'icon': {'name': '/delivery_provider_seamless_20180129.png',
              'prefix': 'https://fastly.4sqi.net/img/general/cap/',
              'sizes': [40, 50]},
             'name': 'seamless'},
            'url': 'https://www.seamless.com/menu/parrilla-latina-5523-broadway-bronx/330981?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=330981'},
           'id': '4dfe40df8877333e195b68fc',
           'location': {'address': '230th St & Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 399,
            'formattedAddress': ['230th St & Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87747294351472,
              'lng': -73.90607346968568}],
            'lat': 40.87747294351472,
            'lng': -73.90607346968568,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Parrilla Latina',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4c852173dc018cfa2bc3e56c-20',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/apparel_kids_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d105951735',
             'name': 'Kids Store',
             'pluralName': 'Kids Stores',
             'primary': True,
             'shortName': 'Kids Store'}],
           'id': '4c852173dc018cfa2bc3e56c',
           'location': {'address': '44 W 225th St',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 383,
            'formattedAddress': ['44 W 225th St',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.873671591133125,
              'lng': -73.90815619608166}],
            'lat': 40.873671591133125,
            'lng': -73.90815619608166,
            'postalCode': '10463',
            'state': 'NY'},
           'name': "The Children's Place",
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-53319bb511d2ef06787f02b4-21',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/mall_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1fd941735',
             'name': 'Shopping Mall',
             'pluralName': 'Shopping Malls',
             'primary': True,
             'shortName': 'Mall'}],
           'id': '53319bb511d2ef06787f02b4',
           'location': {'address': '171 W 231st St',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 456,
            'formattedAddress': ['171 W 231st St',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.87753906779665,
              'lng': -73.90539578168178}],
            'lat': 40.87753906779665,
            'lng': -73.90539578168178,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Broadway Plaza',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4d0a529133d6b60cf4cf9985-22',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d1c5941735',
             'name': 'Sandwich Place',
             'pluralName': 'Sandwich Places',
             'primary': True,
             'shortName': 'Sandwiches'}],
           'id': '4d0a529133d6b60cf4cf9985',
           'location': {'address': '5209 Broadway',
            'cc': 'US',
            'city': 'Bronx',
            'country': 'United States',
            'distance': 463,
            'formattedAddress': ['5209 Broadway',
             'Bronx, NY 10463',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.877720351115315,
              'lng': -73.90537973066263}],
            'lat': 40.877720351115315,
            'lng': -73.90537973066263,
            'postalCode': '10463',
            'state': 'NY'},
           'name': 'Subway',
           'photos': {'count': 0, 'groups': []}}},
         {'reasons': {'count': 0,
           'items': [{'reasonName': 'globalInteractionReason',
             'summary': 'This spot is popular',
             'type': 'general'}]},
          'referralId': 'e-0-4ed7956b8b81b2bf28adc714-23',
          'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
              'suffix': '.png'},
             'id': '4bf58dd8d48988d146941735',
             'name': 'Deli / Bodega',
             'pluralName': 'Delis / Bodegas',
             'primary': True,
             'shortName': 'Deli / Bodega'}],
           'id': '4ed7956b8b81b2bf28adc714',
           'location': {'address': '135 Terrace View Ave.',
            'cc': 'US',
            'city': 'New York',
            'country': 'United States',
            'distance': 218,
            'formattedAddress': ['135 Terrace View Ave.',
             'New York, NY 10034',
             'United States'],
            'labeledLatLngs': [{'label': 'display',
              'lat': 40.875995,
              'lng': -73.913151}],
            'lat': 40.875995,
            'lng': -73.913151,
            'postalCode': '10034',
            'state': 'NY'},
           'name': 'Terrace View Delicatessen',
           'photos': {'count': 0, 'groups': []}}}],
        'name': 'recommended',
        'type': 'Recommended Places'}],
      'headerFullLocation': 'Marble Hill, New York',
      'headerLocation': 'Marble Hill',
      'headerLocationGranularity': 'neighborhood',
      'suggestedBounds': {'ne': {'lat': 40.88105078329964,
        'lng': -73.90471933917806},
       'sw': {'lat': 40.87205077429964, 'lng': -73.91659997808156}},
      'suggestedFilters': {'filters': [{'key': 'openNow', 'name': 'Open now'}],
       'header': 'Tap to show:'},
      'totalResults': 24}}



All the information is in the *items* key.  
Let's borrow the **get_category_type** function from the Foursquare lab.


```python
# function that extracts the category of the venue
def get_category_type(row):
  try:
    categories_list = row['categories']
  except:
    categories_list = row['venue.categories']

  if len(categories_list) == 0:
    return None
  else:
    return categories_list[0]['name']
```

Clean the json and structure it into a pandas dataframe.


```python
venues = results['response']['groups'][0]['items']

nearby_venues = json_normalize(venues) # flatten JSON

# filter columns
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.lat', 'venue.location.lng']
nearby_venues = nearby_venues.loc[:, filtered_columns]

# filter the category for each row
nearby_venues['venue.categories'] = nearby_venues.apply(get_category_type, axis=1)

# clean columns
nearby_venues.columns = [col.split(".")[-1] for col in nearby_venues.columns]

nearby_venues.head()
```




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
      <th>name</th>
      <th>categories</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arturo's</td>
      <td>Pizza Place</td>
      <td>40.874412</td>
      <td>-73.910271</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bikram Yoga</td>
      <td>Yoga Studio</td>
      <td>40.876844</td>
      <td>-73.906204</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tibbett Diner</td>
      <td>Diner</td>
      <td>40.880404</td>
      <td>-73.908937</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dunkin'</td>
      <td>Donut Shop</td>
      <td>40.877136</td>
      <td>-73.906666</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Starbucks</td>
      <td>Coffee Shop</td>
      <td>40.877531</td>
      <td>-73.905582</td>
    </tr>
  </tbody>
</table>
</div>




```python
print('{} venues were returned by Foursquare.'.format(nearby_venues.shape[0]))
```

    24 venues were returned by Foursquare.
    

#### Explore all neighborhoods in Manhattan

Let's create a function to repeat the same process for all the neighborhoods in Manhattan.


```python
def getNearbyVenues(names, latitudes, longitudes, radius=500):
    
    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
        print(name)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            name, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['Neighborhood', 
                  'Neighborhood Latitude', 
                  'Neighborhood Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```

Run the above function on each neighborhood and create a new dataframe.


```python
manhattan_venues = getNearbyVenues(names=manhattan_data['Neighborhood'],
                                   latitudes=manhattan_data['Latitude'], 
                                   longitudes=manhattan_data['Longitude'])
```

    Marble Hill
    Chinatown
    Washington Heights
    Inwood
    Hamilton Heights
    Manhattanville
    Central Harlem
    East Harlem
    Upper East Side
    Yorkville
    Lenox Hill
    Roosevelt Island
    Upper West Side
    Lincoln Square
    Clinton
    Midtown
    Murray Hill
    Chelsea
    Greenwich Village
    East Village
    Lower East Side
    Tribeca
    Little Italy
    Soho
    West Village
    Manhattan Valley
    Morningside Heights
    Gramercy
    Battery Park City
    Financial District
    Carnegie Hill
    Noho
    Civic Center
    Midtown South
    Sutton Place
    Turtle Bay
    Tudor City
    Stuyvesant Town
    Flatiron
    Hudson Yards
    

Let's check the size of the resulting dataframe.


```python
print(manhattan_venues.shape)
manhattan_venues.head()
```

    (3313, 7)
    




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
      <th>Neighborhood</th>
      <th>Neighborhood Latitude</th>
      <th>Neighborhood Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.91066</td>
      <td>Arturo's</td>
      <td>40.874412</td>
      <td>-73.910271</td>
      <td>Pizza Place</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.91066</td>
      <td>Bikram Yoga</td>
      <td>40.876844</td>
      <td>-73.906204</td>
      <td>Yoga Studio</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.91066</td>
      <td>Tibbett Diner</td>
      <td>40.880404</td>
      <td>-73.908937</td>
      <td>Diner</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.91066</td>
      <td>Dunkin'</td>
      <td>40.877136</td>
      <td>-73.906666</td>
      <td>Donut Shop</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.91066</td>
      <td>Starbucks</td>
      <td>40.877531</td>
      <td>-73.905582</td>
      <td>Coffee Shop</td>
    </tr>
  </tbody>
</table>
</div>



Let's check how many venues were returned for each neighborhood.


```python
manhattan_venues.groupby('Neighborhood').count()
```




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
      <th>Neighborhood Latitude</th>
      <th>Neighborhood Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
    <tr>
      <th>Neighborhood</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Battery Park City</th>
      <td>95</td>
      <td>95</td>
      <td>95</td>
      <td>95</td>
      <td>95</td>
      <td>95</td>
    </tr>
    <tr>
      <th>Carnegie Hill</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Central Harlem</th>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
    </tr>
    <tr>
      <th>Chelsea</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Chinatown</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Civic Center</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Clinton</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>East Harlem</th>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
    </tr>
    <tr>
      <th>East Village</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Financial District</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Flatiron</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Gramercy</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Greenwich Village</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Hamilton Heights</th>
      <td>61</td>
      <td>61</td>
      <td>61</td>
      <td>61</td>
      <td>61</td>
      <td>61</td>
    </tr>
    <tr>
      <th>Hudson Yards</th>
      <td>84</td>
      <td>84</td>
      <td>84</td>
      <td>84</td>
      <td>84</td>
      <td>84</td>
    </tr>
    <tr>
      <th>Inwood</th>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
    </tr>
    <tr>
      <th>Lenox Hill</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Lincoln Square</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Little Italy</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Lower East Side</th>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
    </tr>
    <tr>
      <th>Manhattan Valley</th>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
    </tr>
    <tr>
      <th>Manhattanville</th>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
    </tr>
    <tr>
      <th>Marble Hill</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
    </tr>
    <tr>
      <th>Midtown</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Midtown South</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Morningside Heights</th>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
    </tr>
    <tr>
      <th>Murray Hill</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Noho</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Roosevelt Island</th>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
    </tr>
    <tr>
      <th>Soho</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Stuyvesant Town</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>Sutton Place</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Tribeca</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Tudor City</th>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
    </tr>
    <tr>
      <th>Turtle Bay</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Upper East Side</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Upper West Side</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Washington Heights</th>
      <td>90</td>
      <td>90</td>
      <td>90</td>
      <td>90</td>
      <td>90</td>
      <td>90</td>
    </tr>
    <tr>
      <th>West Village</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Yorkville</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
</div>



How many unique categories can be curated from all the returned venues?


```python
print('There are {} unique categories'.format(len(manhattan_venues['Venue Category'].unique())))
```

    There are 343 unique categories
    

### Analyze each neighborhood


```python
# one hot encoding
manhattan_onehot = pd.get_dummies(manhattan_venues[['Venue Category']], prefix='', prefix_sep='')

# add neighborhood column back to the dataframe
manhattan_onehot['Neighborhood'] = manhattan_venues['Neighborhood']

# move neighborhood column to the first column
fixed_columns = [manhattan_onehot.columns[-1]] + list(manhattan_onehot.columns[:-1])
manhattan_onehot = manhattan_onehot[fixed_columns]

manhattan_onehot.head()
```




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
    
    <table border="1">
  <thead>
    <tr>
      <th></th>
      <th>Neighborhood</th>
      <th>Accessories Store</th>
      <th>Adult Boutique</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>Arcade</th>
      <th>Arepa Restaurant</th>
      <th>Argentinian Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Auditorium</th>
      <th>Australian Restaurant</th>
      <th>Austrian Restaurant</th>
      <th>BBQ Joint</th>
      <th>Baby Store</th>
      <th>Bagel Shop</th>
      <th>Bakery</th>
      <th>Bank</th>
      <th>Bar</th>
      <th>Baseball Field</th>
      <th>Basketball Court</th>
      <th>Bed &amp; Breakfast</th>
      <th>Beer Bar</th>
      <th>Beer Garden</th>
      <th>Beer Store</th>
      <th>Big Box Store</th>
      <th>Bike Rental / Bike Share</th>
      <th>Bike Shop</th>
      <th>Bike Trail</th>
      <th>Bistro</th>
      <th>Board Shop</th>
      <th>Boat or Ferry</th>
      <th>Bookstore</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brazilian Restaurant</th>
      <th>Breakfast Spot</th>
      <th>Bridal Shop</th>
      <th>Bridge</th>
      <th>Bubble Tea Shop</th>
      <th>Building</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Line</th>
      <th>Bus Station</th>
      <th>Bus Stop</th>
      <th>Butcher</th>
      <th>Cafeteria</th>
      <th>Café</th>
      <th>Cajun / Creole Restaurant</th>
      <th>Cambodian Restaurant</th>
      <th>Camera Store</th>
      <th>Candy Store</th>
      <th>Caribbean Restaurant</th>
      <th>Caucasian Restaurant</th>
      <th>Cheese Shop</th>
      <th>Chinese Restaurant</th>
      <th>Chocolate Shop</th>
      <th>Circus</th>
      <th>Climbing Gym</th>
      <th>Clothing Store</th>
      <th>Club House</th>
      <th>Cocktail Bar</th>
      <th>Coffee Shop</th>
      <th>College Academic Building</th>
      <th>College Arts Building</th>
      <th>College Bookstore</th>
      <th>College Cafeteria</th>
      <th>College Gym</th>
      <th>College Theater</th>
      <th>Comedy Club</th>
      <th>Comfort Food Restaurant</th>
      <th>Community Center</th>
      <th>Concert Hall</th>
      <th>Convenience Store</th>
      <th>Cooking School</th>
      <th>Cosmetics Shop</th>
      <th>Coworking Space</th>
      <th>Creperie</th>
      <th>Cuban Restaurant</th>
      <th>Cultural Center</th>
      <th>Cupcake Shop</th>
      <th>Cycle Studio</th>
      <th>Czech Restaurant</th>
      <th>Dance Studio</th>
      <th>Daycare</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Design Studio</th>
      <th>Dessert Shop</th>
      <th>Dim Sum Restaurant</th>
      <th>Diner</th>
      <th>Discount Store</th>
      <th>Dive Bar</th>
      <th>Doctor's Office</th>
      <th>Dog Run</th>
      <th>Donut Shop</th>
      <th>Drugstore</th>
      <th>Dry Cleaner</th>
      <th>Dumpling Restaurant</th>
      <th>Duty-free Shop</th>
      <th>Eastern European Restaurant</th>
      <th>Electronics Store</th>
      <th>Empanada Restaurant</th>
      <th>English Restaurant</th>
      <th>Ethiopian Restaurant</th>
      <th>Event Space</th>
      <th>Exhibit</th>
      <th>Falafel Restaurant</th>
      <th>Farmers Market</th>
      <th>Fast Food Restaurant</th>
      <th>Filipino Restaurant</th>
      <th>Fish Market</th>
      <th>Flea Market</th>
      <th>Flower Shop</th>
      <th>Food &amp; Drink Shop</th>
      <th>Food Court</th>
      <th>Food Stand</th>
      <th>Food Truck</th>
      <th>Fountain</th>
      <th>French Restaurant</th>
      <th>Fried Chicken Joint</th>
      <th>Frozen Yogurt Shop</th>
      <th>Furniture / Home Store</th>
      <th>Gaming Cafe</th>
      <th>Garden</th>
      <th>Garden Center</th>
      <th>Gas Station</th>
      <th>Gastropub</th>
      <th>Gay Bar</th>
      <th>General College &amp; University</th>
      <th>General Entertainment</th>
      <th>German Restaurant</th>
      <th>Gift Shop</th>
      <th>Golf Course</th>
      <th>Gourmet Shop</th>
      <th>Greek Restaurant</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gym Pool</th>
      <th>Gymnastics Gym</th>
      <th>Harbor / Marina</th>
      <th>Hardware Store</th>
      <th>Hawaiian Restaurant</th>
      <th>Health &amp; Beauty Service</th>
      <th>Health Food Store</th>
      <th>Heliport</th>
      <th>High School</th>
      <th>Himalayan Restaurant</th>
      <th>Historic Site</th>
      <th>History Museum</th>
      <th>Hobby Shop</th>
      <th>Hookah Bar</th>
      <th>Hostel</th>
      <th>Hot Dog Joint</th>
      <th>Hotel</th>
      <th>Hotel Bar</th>
      <th>Hotpot Restaurant</th>
      <th>Ice Cream Shop</th>
      <th>Indian Restaurant</th>
      <th>Indie Movie Theater</th>
      <th>Indie Theater</th>
      <th>Intersection</th>
      <th>Irish Pub</th>
      <th>Israeli Restaurant</th>
      <th>Italian Restaurant</th>
      <th>Japanese Curry Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Jazz Club</th>
      <th>Jewelry Store</th>
      <th>Jewish Restaurant</th>
      <th>Juice Bar</th>
      <th>Karaoke Bar</th>
      <th>Kebab Restaurant</th>
      <th>Kids Store</th>
      <th>Kitchen Supply Store</th>
      <th>Korean Restaurant</th>
      <th>Kosher Restaurant</th>
      <th>Latin American Restaurant</th>
      <th>Laundry Service</th>
      <th>Leather Goods Store</th>
      <th>Lebanese Restaurant</th>
      <th>Library</th>
      <th>Lingerie Store</th>
      <th>Liquor Store</th>
      <th>Lounge</th>
      <th>Malay Restaurant</th>
      <th>Market</th>
      <th>Martial Arts Dojo</th>
      <th>Massage Studio</th>
      <th>Medical Center</th>
      <th>Mediterranean Restaurant</th>
      <th>Memorial Site</th>
      <th>Men's Store</th>
      <th>Metro Station</th>
      <th>Mexican Restaurant</th>
      <th>Middle Eastern Restaurant</th>
      <th>Mini Golf</th>
      <th>Miscellaneous Shop</th>
      <th>Mobile Phone Shop</th>
      <th>Modern European Restaurant</th>
      <th>Molecular Gastronomy Restaurant</th>
      <th>Monument / Landmark</th>
      <th>Moroccan Restaurant</th>
      <th>Movie Theater</th>
      <th>Museum</th>
      <th>Music School</th>
      <th>Music Venue</th>
      <th>Nail Salon</th>
      <th>New American Restaurant</th>
      <th>Newsstand</th>
      <th>Nightclub</th>
      <th>Non-Profit</th>
      <th>Noodle House</th>
      <th>North Indian Restaurant</th>
      <th>Office</th>
      <th>Opera House</th>
      <th>Optical Shop</th>
      <th>Organic Grocery</th>
      <th>Other Great Outdoors</th>
      <th>Outdoor Sculpture</th>
      <th>Outdoor Supply Store</th>
      <th>Outdoors &amp; Recreation</th>
      <th>Paella Restaurant</th>
      <th>Pakistani Restaurant</th>
      <th>Paper / Office Supplies Store</th>
      <th>Park</th>
      <th>Pastry Shop</th>
      <th>Pedestrian Plaza</th>
      <th>Performing Arts Venue</th>
      <th>Persian Restaurant</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Café</th>
      <th>Pet Service</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Photography Studio</th>
      <th>Physical Therapist</th>
      <th>Piano Bar</th>
      <th>Pie Shop</th>
      <th>Pilates Studio</th>
      <th>Pizza Place</th>
      <th>Playground</th>
      <th>Plaza</th>
      <th>Poke Place</th>
      <th>Pool</th>
      <th>Pop-Up Shop</th>
      <th>Pub</th>
      <th>Public Art</th>
      <th>Ramen Restaurant</th>
      <th>Record Shop</th>
      <th>Recording Studio</th>
      <th>Recreation Center</th>
      <th>Rental Car Location</th>
      <th>Residential Building (Apartment / Condo)</th>
      <th>Resort</th>
      <th>Rest Area</th>
      <th>Restaurant</th>
      <th>Rock Climbing Spot</th>
      <th>Rock Club</th>
      <th>Roof Deck</th>
      <th>Russian Restaurant</th>
      <th>Sake Bar</th>
      <th>Salad Place</th>
      <th>Salon / Barbershop</th>
      <th>Sandwich Place</th>
      <th>Scandinavian Restaurant</th>
      <th>Scenic Lookout</th>
      <th>School</th>
      <th>Sculpture Garden</th>
      <th>Seafood Restaurant</th>
      <th>Shanghai Restaurant</th>
      <th>Shipping Store</th>
      <th>Shoe Store</th>
      <th>Shopping Mall</th>
      <th>Skate Park</th>
      <th>Skating Rink</th>
      <th>Smoke Shop</th>
      <th>Smoothie Shop</th>
      <th>Snack Place</th>
      <th>Soba Restaurant</th>
      <th>Soccer Field</th>
      <th>Social Club</th>
      <th>Soup Place</th>
      <th>South American Restaurant</th>
      <th>South Indian Restaurant</th>
      <th>Southern / Soul Food Restaurant</th>
      <th>Spa</th>
      <th>Spanish Restaurant</th>
      <th>Speakeasy</th>
      <th>Spiritual Center</th>
      <th>Sporting Goods Shop</th>
      <th>Sports Bar</th>
      <th>Sports Club</th>
      <th>Stables</th>
      <th>Steakhouse</th>
      <th>Street Art</th>
      <th>Strip Club</th>
      <th>Supermarket</th>
      <th>Supplement Shop</th>
      <th>Sushi Restaurant</th>
      <th>Swiss Restaurant</th>
      <th>Szechuan Restaurant</th>
      <th>Taco Place</th>
      <th>Tailor Shop</th>
      <th>Taiwanese Restaurant</th>
      <th>Tapas Restaurant</th>
      <th>Tattoo Parlor</th>
      <th>Tea Room</th>
      <th>Tennis Court</th>
      <th>Tennis Stadium</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Park Ride / Attraction</th>
      <th>Thrift / Vintage Store</th>
      <th>Tiki Bar</th>
      <th>Tourist Information Center</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Tree</th>
      <th>Turkish Restaurant</th>
      <th>Udon Restaurant</th>
      <th>Used Bookstore</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Venezuelan Restaurant</th>
      <th>Veterinarian</th>
      <th>Video Game Store</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Volleyball Court</th>
      <th>Waterfront</th>
      <th>Weight Loss Center</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighborhood</th>
      <th>Accessories Store</th>
      <th>Adult Boutique</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>Arcade</th>
      <th>Arepa Restaurant</th>
      <th>Argentinian Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Auditorium</th>
      <th>Australian Restaurant</th>
      <th>Austrian Restaurant</th>
      <th>BBQ Joint</th>
      <th>Baby Store</th>
      <th>Bagel Shop</th>
      <th>Bakery</th>
      <th>Bank</th>
      <th>Bar</th>
      <th>Baseball Field</th>
      <th>Basketball Court</th>
      <th>Bed &amp; Breakfast</th>
      <th>Beer Bar</th>
      <th>Beer Garden</th>
      <th>Beer Store</th>
      <th>Big Box Store</th>
      <th>Bike Rental / Bike Share</th>
      <th>Bike Shop</th>
      <th>Bike Trail</th>
      <th>Bistro</th>
      <th>Board Shop</th>
      <th>Boat or Ferry</th>
      <th>Bookstore</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brazilian Restaurant</th>
      <th>Breakfast Spot</th>
      <th>Bridal Shop</th>
      <th>Bridge</th>
      <th>Bubble Tea Shop</th>
      <th>Building</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Line</th>
      <th>Bus Station</th>
      <th>Bus Stop</th>
      <th>Butcher</th>
      <th>Cafeteria</th>
      <th>Café</th>
      <th>Cajun / Creole Restaurant</th>
      <th>Cambodian Restaurant</th>
      <th>Camera Store</th>
      <th>Candy Store</th>
      <th>Caribbean Restaurant</th>
      <th>Caucasian Restaurant</th>
      <th>Cheese Shop</th>
      <th>Chinese Restaurant</th>
      <th>Chocolate Shop</th>
      <th>Circus</th>
      <th>Climbing Gym</th>
      <th>Clothing Store</th>
      <th>Club House</th>
      <th>Cocktail Bar</th>
      <th>Coffee Shop</th>
      <th>College Academic Building</th>
      <th>College Arts Building</th>
      <th>College Bookstore</th>
      <th>College Cafeteria</th>
      <th>College Gym</th>
      <th>College Theater</th>
      <th>Comedy Club</th>
      <th>Comfort Food Restaurant</th>
      <th>Community Center</th>
      <th>Concert Hall</th>
      <th>Convenience Store</th>
      <th>Cooking School</th>
      <th>Cosmetics Shop</th>
      <th>Coworking Space</th>
      <th>Creperie</th>
      <th>Cuban Restaurant</th>
      <th>Cultural Center</th>
      <th>Cupcake Shop</th>
      <th>Cycle Studio</th>
      <th>Czech Restaurant</th>
      <th>Dance Studio</th>
      <th>Daycare</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Design Studio</th>
      <th>Dessert Shop</th>
      <th>Dim Sum Restaurant</th>
      <th>Diner</th>
      <th>Discount Store</th>
      <th>Dive Bar</th>
      <th>Doctor's Office</th>
      <th>Dog Run</th>
      <th>Donut Shop</th>
      <th>Drugstore</th>
      <th>Dry Cleaner</th>
      <th>Dumpling Restaurant</th>
      <th>Duty-free Shop</th>
      <th>Eastern European Restaurant</th>
      <th>Electronics Store</th>
      <th>Empanada Restaurant</th>
      <th>English Restaurant</th>
      <th>Ethiopian Restaurant</th>
      <th>Event Space</th>
      <th>Exhibit</th>
      <th>Falafel Restaurant</th>
      <th>Farmers Market</th>
      <th>Fast Food Restaurant</th>
      <th>Filipino Restaurant</th>
      <th>Fish Market</th>
      <th>Flea Market</th>
      <th>Flower Shop</th>
      <th>Food &amp; Drink Shop</th>
      <th>Food Court</th>
      <th>Food Stand</th>
      <th>Food Truck</th>
      <th>Fountain</th>
      <th>French Restaurant</th>
      <th>Fried Chicken Joint</th>
      <th>Frozen Yogurt Shop</th>
      <th>Furniture / Home Store</th>
      <th>Gaming Cafe</th>
      <th>Garden</th>
      <th>Garden Center</th>
      <th>Gas Station</th>
      <th>Gastropub</th>
      <th>Gay Bar</th>
      <th>General College &amp; University</th>
      <th>General Entertainment</th>
      <th>German Restaurant</th>
      <th>Gift Shop</th>
      <th>Golf Course</th>
      <th>Gourmet Shop</th>
      <th>Greek Restaurant</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gym Pool</th>
      <th>Gymnastics Gym</th>
      <th>Harbor / Marina</th>
      <th>Hardware Store</th>
      <th>Hawaiian Restaurant</th>
      <th>Health &amp; Beauty Service</th>
      <th>Health Food Store</th>
      <th>Heliport</th>
      <th>High School</th>
      <th>Himalayan Restaurant</th>
      <th>Historic Site</th>
      <th>History Museum</th>
      <th>Hobby Shop</th>
      <th>Hookah Bar</th>
      <th>Hostel</th>
      <th>Hot Dog Joint</th>
      <th>Hotel</th>
      <th>Hotel Bar</th>
      <th>Hotpot Restaurant</th>
      <th>Ice Cream Shop</th>
      <th>Indian Restaurant</th>
      <th>Indie Movie Theater</th>
      <th>Indie Theater</th>
      <th>Intersection</th>
      <th>Irish Pub</th>
      <th>Israeli Restaurant</th>
      <th>Italian Restaurant</th>
      <th>Japanese Curry Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Jazz Club</th>
      <th>Jewelry Store</th>
      <th>Jewish Restaurant</th>
      <th>Juice Bar</th>
      <th>Karaoke Bar</th>
      <th>Kebab Restaurant</th>
      <th>Kids Store</th>
      <th>Kitchen Supply Store</th>
      <th>Korean Restaurant</th>
      <th>Kosher Restaurant</th>
      <th>Latin American Restaurant</th>
      <th>Laundry Service</th>
      <th>Leather Goods Store</th>
      <th>Lebanese Restaurant</th>
      <th>Library</th>
      <th>Lingerie Store</th>
      <th>Liquor Store</th>
      <th>Lounge</th>
      <th>Malay Restaurant</th>
      <th>Market</th>
      <th>Martial Arts Dojo</th>
      <th>Massage Studio</th>
      <th>Medical Center</th>
      <th>Mediterranean Restaurant</th>
      <th>Memorial Site</th>
      <th>Men's Store</th>
      <th>Metro Station</th>
      <th>Mexican Restaurant</th>
      <th>Middle Eastern Restaurant</th>
      <th>Mini Golf</th>
      <th>Miscellaneous Shop</th>
      <th>Mobile Phone Shop</th>
      <th>Modern European Restaurant</th>
      <th>Molecular Gastronomy Restaurant</th>
      <th>Monument / Landmark</th>
      <th>Moroccan Restaurant</th>
      <th>Movie Theater</th>
      <th>Museum</th>
      <th>Music School</th>
      <th>Music Venue</th>
      <th>Nail Salon</th>
      <th>New American Restaurant</th>
      <th>Newsstand</th>
      <th>Nightclub</th>
      <th>Non-Profit</th>
      <th>Noodle House</th>
      <th>North Indian Restaurant</th>
      <th>Office</th>
      <th>Opera House</th>
      <th>Optical Shop</th>
      <th>Organic Grocery</th>
      <th>Other Great Outdoors</th>
      <th>Outdoor Sculpture</th>
      <th>Outdoor Supply Store</th>
      <th>Outdoors &amp; Recreation</th>
      <th>Paella Restaurant</th>
      <th>Pakistani Restaurant</th>
      <th>Paper / Office Supplies Store</th>
      <th>Park</th>
      <th>Pastry Shop</th>
      <th>Pedestrian Plaza</th>
      <th>Performing Arts Venue</th>
      <th>Persian Restaurant</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Café</th>
      <th>Pet Service</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Photography Studio</th>
      <th>Physical Therapist</th>
      <th>Piano Bar</th>
      <th>Pie Shop</th>
      <th>Pilates Studio</th>
      <th>Pizza Place</th>
      <th>Playground</th>
      <th>Plaza</th>
      <th>Poke Place</th>
      <th>Pool</th>
      <th>Pop-Up Shop</th>
      <th>Pub</th>
      <th>Public Art</th>
      <th>Ramen Restaurant</th>
      <th>Record Shop</th>
      <th>Recording Studio</th>
      <th>Recreation Center</th>
      <th>Rental Car Location</th>
      <th>Residential Building (Apartment / Condo)</th>
      <th>Resort</th>
      <th>Rest Area</th>
      <th>Restaurant</th>
      <th>Rock Climbing Spot</th>
      <th>Rock Club</th>
      <th>Roof Deck</th>
      <th>Russian Restaurant</th>
      <th>Sake Bar</th>
      <th>Salad Place</th>
      <th>Salon / Barbershop</th>
      <th>Sandwich Place</th>
      <th>Scandinavian Restaurant</th>
      <th>Scenic Lookout</th>
      <th>School</th>
      <th>Sculpture Garden</th>
      <th>Seafood Restaurant</th>
      <th>Shanghai Restaurant</th>
      <th>Shipping Store</th>
      <th>Shoe Store</th>
      <th>Shopping Mall</th>
      <th>Skate Park</th>
      <th>Skating Rink</th>
      <th>Smoke Shop</th>
      <th>Smoothie Shop</th>
      <th>Snack Place</th>
      <th>Soba Restaurant</th>
      <th>Soccer Field</th>
      <th>Social Club</th>
      <th>Soup Place</th>
      <th>South American Restaurant</th>
      <th>South Indian Restaurant</th>
      <th>Southern / Soul Food Restaurant</th>
      <th>Spa</th>
      <th>Spanish Restaurant</th>
      <th>Speakeasy</th>
      <th>Spiritual Center</th>
      <th>Sporting Goods Shop</th>
      <th>Sports Bar</th>
      <th>Sports Club</th>
      <th>Stables</th>
      <th>Steakhouse</th>
      <th>Street Art</th>
      <th>Strip Club</th>
      <th>Supermarket</th>
      <th>Supplement Shop</th>
      <th>Sushi Restaurant</th>
      <th>Swiss Restaurant</th>
      <th>Szechuan Restaurant</th>
      <th>Taco Place</th>
      <th>Tailor Shop</th>
      <th>Taiwanese Restaurant</th>
      <th>Tapas Restaurant</th>
      <th>Tattoo Parlor</th>
      <th>Tea Room</th>
      <th>Tennis Court</th>
      <th>Tennis Stadium</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Park Ride / Attraction</th>
      <th>Thrift / Vintage Store</th>
      <th>Tiki Bar</th>
      <th>Tourist Information Center</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Tree</th>
      <th>Turkish Restaurant</th>
      <th>Udon Restaurant</th>
      <th>Used Bookstore</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Venezuelan Restaurant</th>
      <th>Veterinarian</th>
      <th>Video Game Store</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Volleyball Court</th>
      <th>Waterfront</th>
      <th>Weight Loss Center</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Marble Hill</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# examine the new dataframe size
manhattan_onehot.shape
```




    (3313, 344)



Let's group rows by neighborhood and by taking the mean of the frequency of occurrence of each category.


```python
manhattan_grouped = manhattan_onehot.groupby('Neighborhood').mean().reset_index()
manhattan_grouped
```




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
      <th>Neighborhood</th>
      <th>Accessories Store</th>
      <th>Adult Boutique</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>Arcade</th>
      <th>Arepa Restaurant</th>
      <th>Argentinian Restaurant</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Auditorium</th>
      <th>Australian Restaurant</th>
      <th>Austrian Restaurant</th>
      <th>BBQ Joint</th>
      <th>Baby Store</th>
      <th>Bagel Shop</th>
      <th>Bakery</th>
      <th>Bank</th>
      <th>Bar</th>
      <th>Baseball Field</th>
      <th>Basketball Court</th>
      <th>Bed &amp; Breakfast</th>
      <th>Beer Bar</th>
      <th>Beer Garden</th>
      <th>Beer Store</th>
      <th>Big Box Store</th>
      <th>Bike Rental / Bike Share</th>
      <th>Bike Shop</th>
      <th>Bike Trail</th>
      <th>Bistro</th>
      <th>Board Shop</th>
      <th>Boat or Ferry</th>
      <th>Bookstore</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brazilian Restaurant</th>
      <th>Breakfast Spot</th>
      <th>Bridal Shop</th>
      <th>Bridge</th>
      <th>Bubble Tea Shop</th>
      <th>Building</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Line</th>
      <th>Bus Station</th>
      <th>Bus Stop</th>
      <th>Butcher</th>
      <th>Cafeteria</th>
      <th>Café</th>
      <th>Cajun / Creole Restaurant</th>
      <th>Cambodian Restaurant</th>
      <th>Camera Store</th>
      <th>Candy Store</th>
      <th>Caribbean Restaurant</th>
      <th>Caucasian Restaurant</th>
      <th>Cheese Shop</th>
      <th>Chinese Restaurant</th>
      <th>Chocolate Shop</th>
      <th>Circus</th>
      <th>Climbing Gym</th>
      <th>Clothing Store</th>
      <th>Club House</th>
      <th>Cocktail Bar</th>
      <th>Coffee Shop</th>
      <th>College Academic Building</th>
      <th>College Arts Building</th>
      <th>College Bookstore</th>
      <th>College Cafeteria</th>
      <th>College Gym</th>
      <th>College Theater</th>
      <th>Comedy Club</th>
      <th>Comfort Food Restaurant</th>
      <th>Community Center</th>
      <th>Concert Hall</th>
      <th>Convenience Store</th>
      <th>Cooking School</th>
      <th>Cosmetics Shop</th>
      <th>Coworking Space</th>
      <th>Creperie</th>
      <th>Cuban Restaurant</th>
      <th>Cultural Center</th>
      <th>Cupcake Shop</th>
      <th>Cycle Studio</th>
      <th>Czech Restaurant</th>
      <th>Dance Studio</th>
      <th>Daycare</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Design Studio</th>
      <th>Dessert Shop</th>
      <th>Dim Sum Restaurant</th>
      <th>Diner</th>
      <th>Discount Store</th>
      <th>Dive Bar</th>
      <th>Doctor's Office</th>
      <th>Dog Run</th>
      <th>Donut Shop</th>
      <th>Drugstore</th>
      <th>Dry Cleaner</th>
      <th>Dumpling Restaurant</th>
      <th>Duty-free Shop</th>
      <th>Eastern European Restaurant</th>
      <th>Electronics Store</th>
      <th>Empanada Restaurant</th>
      <th>English Restaurant</th>
      <th>Ethiopian Restaurant</th>
      <th>Event Space</th>
      <th>Exhibit</th>
      <th>Falafel Restaurant</th>
      <th>Farmers Market</th>
      <th>Fast Food Restaurant</th>
      <th>Filipino Restaurant</th>
      <th>Fish Market</th>
      <th>Flea Market</th>
      <th>Flower Shop</th>
      <th>Food &amp; Drink Shop</th>
      <th>Food Court</th>
      <th>Food Stand</th>
      <th>Food Truck</th>
      <th>Fountain</th>
      <th>French Restaurant</th>
      <th>Fried Chicken Joint</th>
      <th>Frozen Yogurt Shop</th>
      <th>Furniture / Home Store</th>
      <th>Gaming Cafe</th>
      <th>Garden</th>
      <th>Garden Center</th>
      <th>Gas Station</th>
      <th>Gastropub</th>
      <th>Gay Bar</th>
      <th>General College &amp; University</th>
      <th>General Entertainment</th>
      <th>German Restaurant</th>
      <th>Gift Shop</th>
      <th>Golf Course</th>
      <th>Gourmet Shop</th>
      <th>Greek Restaurant</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gym Pool</th>
      <th>Gymnastics Gym</th>
      <th>Harbor / Marina</th>
      <th>Hardware Store</th>
      <th>Hawaiian Restaurant</th>
      <th>Health &amp; Beauty Service</th>
      <th>Health Food Store</th>
      <th>Heliport</th>
      <th>High School</th>
      <th>Himalayan Restaurant</th>
      <th>Historic Site</th>
      <th>History Museum</th>
      <th>Hobby Shop</th>
      <th>Hookah Bar</th>
      <th>Hostel</th>
      <th>Hot Dog Joint</th>
      <th>Hotel</th>
      <th>Hotel Bar</th>
      <th>Hotpot Restaurant</th>
      <th>Ice Cream Shop</th>
      <th>Indian Restaurant</th>
      <th>Indie Movie Theater</th>
      <th>Indie Theater</th>
      <th>Intersection</th>
      <th>Irish Pub</th>
      <th>Israeli Restaurant</th>
      <th>Italian Restaurant</th>
      <th>Japanese Curry Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Jazz Club</th>
      <th>Jewelry Store</th>
      <th>Jewish Restaurant</th>
      <th>Juice Bar</th>
      <th>Karaoke Bar</th>
      <th>Kebab Restaurant</th>
      <th>Kids Store</th>
      <th>Kitchen Supply Store</th>
      <th>Korean Restaurant</th>
      <th>Kosher Restaurant</th>
      <th>Latin American Restaurant</th>
      <th>Laundry Service</th>
      <th>Leather Goods Store</th>
      <th>Lebanese Restaurant</th>
      <th>Library</th>
      <th>Lingerie Store</th>
      <th>Liquor Store</th>
      <th>Lounge</th>
      <th>Malay Restaurant</th>
      <th>Market</th>
      <th>Martial Arts Dojo</th>
      <th>Massage Studio</th>
      <th>Medical Center</th>
      <th>Mediterranean Restaurant</th>
      <th>Memorial Site</th>
      <th>Men's Store</th>
      <th>Metro Station</th>
      <th>Mexican Restaurant</th>
      <th>Middle Eastern Restaurant</th>
      <th>Mini Golf</th>
      <th>Miscellaneous Shop</th>
      <th>Mobile Phone Shop</th>
      <th>Modern European Restaurant</th>
      <th>Molecular Gastronomy Restaurant</th>
      <th>Monument / Landmark</th>
      <th>Moroccan Restaurant</th>
      <th>Movie Theater</th>
      <th>Museum</th>
      <th>Music School</th>
      <th>Music Venue</th>
      <th>Nail Salon</th>
      <th>New American Restaurant</th>
      <th>Newsstand</th>
      <th>Nightclub</th>
      <th>Non-Profit</th>
      <th>Noodle House</th>
      <th>North Indian Restaurant</th>
      <th>Office</th>
      <th>Opera House</th>
      <th>Optical Shop</th>
      <th>Organic Grocery</th>
      <th>Other Great Outdoors</th>
      <th>Outdoor Sculpture</th>
      <th>Outdoor Supply Store</th>
      <th>Outdoors &amp; Recreation</th>
      <th>Paella Restaurant</th>
      <th>Pakistani Restaurant</th>
      <th>Paper / Office Supplies Store</th>
      <th>Park</th>
      <th>Pastry Shop</th>
      <th>Pedestrian Plaza</th>
      <th>Performing Arts Venue</th>
      <th>Persian Restaurant</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Café</th>
      <th>Pet Service</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Photography Studio</th>
      <th>Physical Therapist</th>
      <th>Piano Bar</th>
      <th>Pie Shop</th>
      <th>Pilates Studio</th>
      <th>Pizza Place</th>
      <th>Playground</th>
      <th>Plaza</th>
      <th>Poke Place</th>
      <th>Pool</th>
      <th>Pop-Up Shop</th>
      <th>Pub</th>
      <th>Public Art</th>
      <th>Ramen Restaurant</th>
      <th>Record Shop</th>
      <th>Recording Studio</th>
      <th>Recreation Center</th>
      <th>Rental Car Location</th>
      <th>Residential Building (Apartment / Condo)</th>
      <th>Resort</th>
      <th>Rest Area</th>
      <th>Restaurant</th>
      <th>Rock Climbing Spot</th>
      <th>Rock Club</th>
      <th>Roof Deck</th>
      <th>Russian Restaurant</th>
      <th>Sake Bar</th>
      <th>Salad Place</th>
      <th>Salon / Barbershop</th>
      <th>Sandwich Place</th>
      <th>Scandinavian Restaurant</th>
      <th>Scenic Lookout</th>
      <th>School</th>
      <th>Sculpture Garden</th>
      <th>Seafood Restaurant</th>
      <th>Shanghai Restaurant</th>
      <th>Shipping Store</th>
      <th>Shoe Store</th>
      <th>Shopping Mall</th>
      <th>Skate Park</th>
      <th>Skating Rink</th>
      <th>Smoke Shop</th>
      <th>Smoothie Shop</th>
      <th>Snack Place</th>
      <th>Soba Restaurant</th>
      <th>Soccer Field</th>
      <th>Social Club</th>
      <th>Soup Place</th>
      <th>South American Restaurant</th>
      <th>South Indian Restaurant</th>
      <th>Southern / Soul Food Restaurant</th>
      <th>Spa</th>
      <th>Spanish Restaurant</th>
      <th>Speakeasy</th>
      <th>Spiritual Center</th>
      <th>Sporting Goods Shop</th>
      <th>Sports Bar</th>
      <th>Sports Club</th>
      <th>Stables</th>
      <th>Steakhouse</th>
      <th>Street Art</th>
      <th>Strip Club</th>
      <th>Supermarket</th>
      <th>Supplement Shop</th>
      <th>Sushi Restaurant</th>
      <th>Swiss Restaurant</th>
      <th>Szechuan Restaurant</th>
      <th>Taco Place</th>
      <th>Tailor Shop</th>
      <th>Taiwanese Restaurant</th>
      <th>Tapas Restaurant</th>
      <th>Tattoo Parlor</th>
      <th>Tea Room</th>
      <th>Tennis Court</th>
      <th>Tennis Stadium</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Park Ride / Attraction</th>
      <th>Thrift / Vintage Store</th>
      <th>Tiki Bar</th>
      <th>Tourist Information Center</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Tree</th>
      <th>Turkish Restaurant</th>
      <th>Udon Restaurant</th>
      <th>Used Bookstore</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Venezuelan Restaurant</th>
      <th>Veterinarian</th>
      <th>Video Game Store</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Volleyball Court</th>
      <th>Waterfront</th>
      <th>Weight Loss Center</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Battery Park City</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.010526</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.021053</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.063158</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.021053</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.000000</td>
      <td>0.021053</td>
      <td>0.031579</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.052632</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.031579</td>
      <td>0.021053</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.073684</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021053</td>
      <td>0.010526</td>
      <td>0.021053</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.021053</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.031579</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010526</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.031579</td>
      <td>0.000000</td>
      <td>0.031579</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Carnegie Hill</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.060000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.030000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Central Harlem</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.065217</td>
      <td>0.043478</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.043478</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.043478</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.043478</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.043478</td>
      <td>0.043478</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.043478</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.021739</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.021739</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chelsea</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.050000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chinatown</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.090000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.03</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.03</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Civic Center</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Clinton</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.090000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>East Harlem</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.097561</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02439</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.048780</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.02439</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.073171</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.121951</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.024390</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.024390</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.02439</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.024390</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.073171</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>East Village</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.070000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.03</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Financial District</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.090000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.040000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.01000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Flatiron</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.040000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Gramercy</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.03000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Greenwich Village</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.130000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Hamilton Heights</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.016393</td>
      <td>0.016393</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.065574</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.032787</td>
      <td>0.065574</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.065574</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.016393</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.00000</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.016393</td>
      <td>0.049180</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.098361</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.032787</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.016393</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.032787</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Hudson Yards</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.059524</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011905</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.011905</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011905</td>
      <td>0.011905</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Inwood</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.053571</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.017857</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.053571</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.053571</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Lenox Hill</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.060000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.070000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Lincoln Square</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0100</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.03</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Little Italy</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Lower East Side</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.053571</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.053571</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.053571</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.017857</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.053571</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.017857</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.017857</td>
      <td>0.017857</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Manhattan Valley</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.058824</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.039216</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.039216</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.058824</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.00000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.039216</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.058824</td>
      <td>0.039216</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.039216</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.019608</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.019608</td>
      <td>0.019608</td>
      <td>0.000000</td>
      <td>0.039216</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Manhattanville</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02381</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Marble Hill</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.083333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.083333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.083333</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Midtown</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.070000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.01</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Midtown South</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.050000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.150000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Morningside Heights</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Murray Hill</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00000</td>
      <td>0.040000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Noho</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.050000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.03</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Roosevelt Island</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Soho</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.060000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.100000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Stuyvesant Town</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.125000</td>
      <td>0.0625</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.062500</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.062500</td>
      <td>0.062500</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.062500</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0625</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.06250</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.062500</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.062500</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.062500</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.125000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0625</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.062500</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Sutton Place</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.040000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Tribeca</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.030000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Tudor City</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.025316</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.012658</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.063291</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.037975</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.037975</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.037975</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.025316</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.025316</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.037975</td>
      <td>0.000000</td>
      <td>0.025316</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.00000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.012658</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.063291</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.063291</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.037975</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.025316</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.025316</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.012658</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.025316</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.012658</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.025316</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012658</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Turtle Bay</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.070000</td>
      <td>0.01000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Upper East Side</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.060000</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.06</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.060000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Upper West Side</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.060000</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Washington Heights</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.044444</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.055556</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.044444</td>
      <td>0.022222</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.033333</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.0000</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.011111</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.022222</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.022222</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.022222</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.011111</td>
      <td>0.022222</td>
      <td>0.000000</td>
      <td>0.011111</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>38</th>
      <td>West Village</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.090000</td>
      <td>0.00000</td>
      <td>0.020000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.060000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Yorkville</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.050000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.060000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.070000</td>
      <td>0.00000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# confirm the new size
manhattan_grouped.shape
```




    (40, 344)



Let's print the top 5 most common venues for each neighborhood.


```python
num_top_venues = 5

for hood in manhattan_grouped['Neighborhood']:
  print('----' + hood + '----')
  temp = manhattan_grouped[manhattan_grouped['Neighborhood'] == hood].T.reset_index()
  temp.columns = ['venue', 'freq']
  temp = temp.iloc[1:]
  temp['freq'] = temp['freq'].astype(float)
  temp = temp.round({'freq': 2})
  print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
  print('\n')
```

    ----Battery Park City----
               venue  freq
    0           Park  0.07
    1    Coffee Shop  0.06
    2          Hotel  0.05
    3  Shopping Mall  0.03
    4  Women's Store  0.03
    
    
    ----Carnegie Hill----
                      venue  freq
    0           Coffee Shop  0.06
    1           Pizza Place  0.06
    2                  Café  0.05
    3  Gym / Fitness Center  0.03
    4        Cosmetics Shop  0.03
    
    
    ----Central Harlem----
                    venue  freq
    0  African Restaurant  0.07
    1                 Bar  0.04
    2  Seafood Restaurant  0.04
    3   French Restaurant  0.04
    4      Cosmetics Shop  0.04
    
    
    ----Chelsea----
                    venue  freq
    0         Coffee Shop  0.06
    1  Italian Restaurant  0.05
    2              Bakery  0.05
    3               Hotel  0.03
    4             Theater  0.03
    
    
    ----Chinatown----
                       venue  freq
    0     Chinese Restaurant  0.09
    1           Cocktail Bar  0.05
    2  Vietnamese Restaurant  0.04
    3    American Restaurant  0.04
    4                 Bakery  0.03
    
    
    ----Civic Center----
                      venue  freq
    0           Coffee Shop  0.06
    1  Gym / Fitness Center  0.05
    2                 Hotel  0.04
    3     French Restaurant  0.04
    4           Yoga Studio  0.03
    
    
    ----Clinton----
                      venue  freq
    0               Theater  0.09
    1    Italian Restaurant  0.05
    2  Gym / Fitness Center  0.05
    3   American Restaurant  0.04
    4           Coffee Shop  0.04
    
    
    ----East Harlem----
                           venue  freq
    0         Mexican Restaurant  0.12
    1                     Bakery  0.10
    2            Thai Restaurant  0.07
    3  Latin American Restaurant  0.07
    4              Deli / Bodega  0.05
    
    
    ----East Village----
                    venue  freq
    0                 Bar  0.07
    1      Ice Cream Shop  0.05
    2            Wine Bar  0.05
    3  Mexican Restaurant  0.04
    4        Cocktail Bar  0.04
    
    
    ----Financial District----
                     venue  freq
    0          Coffee Shop  0.09
    1                  Bar  0.05
    2  American Restaurant  0.05
    3          Pizza Place  0.04
    4                  Gym  0.04
    
    
    ----Flatiron----
                      venue  freq
    0           Yoga Studio  0.04
    1                  Café  0.04
    2   Japanese Restaurant  0.04
    3   American Restaurant  0.04
    4  Gym / Fitness Center  0.04
    
    
    ----Gramercy----
                    venue  freq
    0                 Bar  0.05
    1  Italian Restaurant  0.05
    2          Bagel Shop  0.04
    3  Mexican Restaurant  0.04
    4         Pizza Place  0.04
    
    
    ----Greenwich Village----
                    venue  freq
    0  Italian Restaurant  0.13
    1      Clothing Store  0.06
    2                Café  0.04
    3    Sushi Restaurant  0.04
    4   French Restaurant  0.03
    
    
    ----Hamilton Heights----
                    venue  freq
    0         Pizza Place  0.10
    1       Deli / Bodega  0.07
    2         Coffee Shop  0.07
    3                Café  0.07
    4  Mexican Restaurant  0.05
    
    
    ----Hudson Yards----
                      venue  freq
    0   American Restaurant  0.06
    1  Gym / Fitness Center  0.05
    2           Coffee Shop  0.05
    3                 Hotel  0.05
    4    Italian Restaurant  0.05
    
    
    ----Inwood----
                    venue  freq
    0              Lounge  0.07
    1  Mexican Restaurant  0.07
    2         Pizza Place  0.05
    3                Café  0.05
    4          Restaurant  0.05
    
    
    ----Lenox Hill----
                    venue  freq
    0  Italian Restaurant  0.07
    1         Coffee Shop  0.06
    2    Sushi Restaurant  0.05
    3         Pizza Place  0.05
    4        Cocktail Bar  0.04
    
    
    ----Lincoln Square----
                       venue  freq
    0                  Plaza  0.06
    1                   Café  0.05
    2                Theater  0.05
    3     Italian Restaurant  0.05
    4  Performing Arts Venue  0.04
    
    
    ----Little Italy----
                          venue  freq
    0                      Café  0.06
    1                    Bakery  0.05
    2           Bubble Tea Shop  0.04
    3  Mediterranean Restaurant  0.03
    4        Italian Restaurant  0.03
    
    
    ----Lower East Side----
                     venue  freq
    0          Pizza Place  0.05
    1          Coffee Shop  0.05
    2          Art Gallery  0.05
    3                 Café  0.05
    4  Japanese Restaurant  0.04
    
    
    ----Manhattan Valley----
                   venue  freq
    0        Pizza Place  0.06
    1                Bar  0.06
    2  Indian Restaurant  0.06
    3         Playground  0.04
    4    Thai Restaurant  0.04
    
    
    ----Manhattanville----
                    venue  freq
    0         Coffee Shop  0.07
    1                Park  0.05
    2  Mexican Restaurant  0.05
    3       Deli / Bodega  0.05
    4  Seafood Restaurant  0.05
    
    
    ----Marble Hill----
                    venue  freq
    0                 Gym  0.08
    1      Sandwich Place  0.08
    2         Coffee Shop  0.08
    3         Yoga Studio  0.04
    4  Seafood Restaurant  0.04
    
    
    ----Midtown----
                     venue  freq
    0                Hotel  0.07
    1          Coffee Shop  0.04
    2       Clothing Store  0.04
    3         Cocktail Bar  0.03
    4  Sporting Goods Shop  0.03
    
    
    ----Midtown South----
                     venue  freq
    0    Korean Restaurant  0.15
    1  Japanese Restaurant  0.05
    2            Hotel Bar  0.05
    3                Hotel  0.05
    4         Dessert Shop  0.04
    
    
    ----Morningside Heights----
                     venue  freq
    0  American Restaurant  0.07
    1                 Park  0.07
    2            Bookstore  0.07
    3          Coffee Shop  0.07
    4       Sandwich Place  0.05
    
    
    ----Murray Hill----
                     venue  freq
    0          Coffee Shop  0.05
    1       Sandwich Place  0.05
    2  American Restaurant  0.04
    3  Japanese Restaurant  0.04
    4                Hotel  0.04
    
    
    ----Noho----
                    venue  freq
    0   French Restaurant  0.05
    1  Italian Restaurant  0.05
    2        Cocktail Bar  0.04
    3               Hotel  0.04
    4           Rock Club  0.03
    
    
    ----Roosevelt Island----
                                          venue  freq
    0                               Coffee Shop  0.07
    1                            Sandwich Place  0.07
    2                                      Park  0.07
    3  Residential Building (Apartment / Condo)  0.03
    4                                       Gym  0.03
    
    
    ----Soho----
                venue  freq
    0  Clothing Store  0.10
    1        Boutique  0.06
    2   Women's Store  0.04
    3      Shoe Store  0.04
    4     Art Gallery  0.04
    
    
    ----Stuyvesant Town----
                      venue  freq
    0                   Bar  0.12
    1                  Park  0.12
    2        Farmers Market  0.06
    3  Gym / Fitness Center  0.06
    4              Heliport  0.06
    
    
    ----Sutton Place----
                        venue  freq
    0    Gym / Fitness Center  0.06
    1      Italian Restaurant  0.04
    2  Furniture / Home Store  0.04
    3                     Gym  0.04
    4       Indian Restaurant  0.03
    
    
    ----Tribeca----
                     venue  freq
    0                 Park  0.05
    1  American Restaurant  0.05
    2   Italian Restaurant  0.05
    3                 Café  0.04
    4                  Spa  0.04
    
    
    ----Tudor City----
                    venue  freq
    0                Park  0.06
    1                Café  0.06
    2  Mexican Restaurant  0.06
    3         Coffee Shop  0.04
    4               Diner  0.04
    
    
    ----Turtle Bay----
                    venue  freq
    0  Italian Restaurant  0.07
    1    Sushi Restaurant  0.05
    2         Coffee Shop  0.05
    3          Steakhouse  0.05
    4            Wine Bar  0.04
    
    
    ----Upper East Side----
                    venue  freq
    0             Exhibit  0.06
    1         Art Gallery  0.06
    2  Italian Restaurant  0.06
    3         Coffee Shop  0.05
    4              Bakery  0.05
    
    
    ----Upper West Side----
                    venue  freq
    0  Italian Restaurant  0.06
    1         Coffee Shop  0.04
    2                 Bar  0.04
    3            Wine Bar  0.04
    4                Café  0.04
    
    
    ----Washington Heights----
                   venue  freq
    0               Café  0.06
    1      Grocery Store  0.04
    2             Bakery  0.04
    3  Mobile Phone Shop  0.03
    4    Supplement Shop  0.02
    
    
    ----West Village----
                         venue  freq
    0       Italian Restaurant  0.09
    1  New American Restaurant  0.06
    2           Cosmetics Shop  0.05
    3      American Restaurant  0.05
    4                 Wine Bar  0.04
    
    
    ----Yorkville----
                    venue  freq
    0  Italian Restaurant  0.07
    1         Coffee Shop  0.06
    2                 Gym  0.06
    3                 Bar  0.05
    4         Pizza Place  0.04
    
    
    

Let's put that into a pandas dataframe.


```python
# funciton to sort the venues in descending order
def return_most_common_venues(row, num_top_venues):
  row_categories = row.iloc[1:]
  row_categories_sorted = row_categories.sort_values(ascending=False)

  return row_categories_sorted.index.values[0:num_top_venues]
```

Let's create the new dataframe and display the top 10 venues for each neighborhood.


```python
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['Neighborhood']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
neighborhoods_venues_sorted = pd.DataFrame(columns=columns)
neighborhoods_venues_sorted['Neighborhood'] = manhattan_grouped['Neighborhood']

for ind in np.arange(manhattan_grouped.shape[0]):
    neighborhoods_venues_sorted.iloc[ind, 1:] = return_most_common_venues(manhattan_grouped.iloc[ind, :], num_top_venues)

neighborhoods_venues_sorted.head()
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Battery Park City</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Hotel</td>
      <td>Women's Store</td>
      <td>Memorial Site</td>
      <td>Shopping Mall</td>
      <td>Gym</td>
      <td>Wine Shop</td>
      <td>Boat or Ferry</td>
      <td>Food Court</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Carnegie Hill</td>
      <td>Pizza Place</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Bookstore</td>
      <td>Cosmetics Shop</td>
      <td>Grocery Store</td>
      <td>Gym</td>
      <td>Gym / Fitness Center</td>
      <td>Bakery</td>
      <td>Japanese Restaurant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Central Harlem</td>
      <td>African Restaurant</td>
      <td>Fried Chicken Joint</td>
      <td>French Restaurant</td>
      <td>American Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Seafood Restaurant</td>
      <td>Bar</td>
      <td>Metro Station</td>
      <td>Southern / Soul Food Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chelsea</td>
      <td>Coffee Shop</td>
      <td>Italian Restaurant</td>
      <td>Bakery</td>
      <td>Wine Shop</td>
      <td>American Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Theater</td>
      <td>Hotel</td>
      <td>Sushi Restaurant</td>
      <td>Cupcake Shop</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chinatown</td>
      <td>Chinese Restaurant</td>
      <td>Cocktail Bar</td>
      <td>American Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>Hotpot Restaurant</td>
      <td>Optical Shop</td>
      <td>Spa</td>
      <td>Bakery</td>
      <td>Salon / Barbershop</td>
      <td>Bar</td>
    </tr>
  </tbody>
</table>
</div>



### Cluster the neighborhoods

Run the k-means algorithm to cluster the neighborhood into 5 clusters.


```python
# set the number of clusters
kclusters = 5

manhattan_grouped_clustering = manhattan_grouped.drop('Neighborhood', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(manhattan_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10]
```




    array([1, 0, 1, 1, 1, 1, 1, 3, 0, 0], dtype=int32)



Let's create a new dataframe that includes the cluster as well as the top 10 venues for each neighborhood.


```python
# add clustering labels
neighborhoods_venues_sorted.insert(0, 'Cluster Labels', kmeans.labels_)

manhattan_merged = manhattan_data

# merge manhattan_grouped with manhattan_data to add latitude/longitude for each neighborhood
manhattan_merged = manhattan_merged.join(neighborhoods_venues_sorted.set_index('Neighborhood'), on='Neighborhood')

manhattan_merged.head()
```




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
      <th>Borough</th>
      <th>Neighborhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Manhattan</td>
      <td>Marble Hill</td>
      <td>40.876551</td>
      <td>-73.910660</td>
      <td>4</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>Sandwich Place</td>
      <td>Yoga Studio</td>
      <td>Tennis Stadium</td>
      <td>Supplement Shop</td>
      <td>Donut Shop</td>
      <td>Miscellaneous Shop</td>
      <td>Steakhouse</td>
      <td>Discount Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Manhattan</td>
      <td>Chinatown</td>
      <td>40.715618</td>
      <td>-73.994279</td>
      <td>1</td>
      <td>Chinese Restaurant</td>
      <td>Cocktail Bar</td>
      <td>American Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>Hotpot Restaurant</td>
      <td>Optical Shop</td>
      <td>Spa</td>
      <td>Bakery</td>
      <td>Salon / Barbershop</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Manhattan</td>
      <td>Washington Heights</td>
      <td>40.851903</td>
      <td>-73.936900</td>
      <td>3</td>
      <td>Café</td>
      <td>Grocery Store</td>
      <td>Bakery</td>
      <td>Mobile Phone Shop</td>
      <td>Pizza Place</td>
      <td>Chinese Restaurant</td>
      <td>Tapas Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Coffee Shop</td>
      <td>Supplement Shop</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Manhattan</td>
      <td>Inwood</td>
      <td>40.867684</td>
      <td>-73.921210</td>
      <td>3</td>
      <td>Mexican Restaurant</td>
      <td>Lounge</td>
      <td>Pizza Place</td>
      <td>Restaurant</td>
      <td>Café</td>
      <td>Spanish Restaurant</td>
      <td>Bakery</td>
      <td>Park</td>
      <td>Chinese Restaurant</td>
      <td>Frozen Yogurt Shop</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Manhattan</td>
      <td>Hamilton Heights</td>
      <td>40.823604</td>
      <td>-73.949688</td>
      <td>3</td>
      <td>Pizza Place</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Deli / Bodega</td>
      <td>Mexican Restaurant</td>
      <td>Yoga Studio</td>
      <td>Sushi Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>School</td>
    </tr>
  </tbody>
</table>
</div>



Let's visualize the clusters.


```python
# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=11)

# set color scheme for the clusters
x = np.arange(kclusters)
ys = [i + x + (i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(manhattan_merged['Latitude'], manhattan_merged['Longitude'], manhattan_merged['Neighborhood'], manhattan_merged['Cluster Labels']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```

<img src="/assets/img/clustering neighborhoods/nyc/map3.jpg">


### Examine the clusters
You can examine and determine the discriminating venue categories that distinguish each cluster. Based on the defining categories, you can assign a name to each cluster.

**Cluster 1**


```python
manhattan_merged.loc[manhattan_merged['Cluster Labels'] == 0, manhattan_merged.columns[[1] + list(range(5, manhattan_merged.shape[1]))]]
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>Yorkville</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>Bar</td>
      <td>Pizza Place</td>
      <td>Deli / Bodega</td>
      <td>Sushi Restaurant</td>
      <td>Wine Shop</td>
      <td>Mexican Restaurant</td>
      <td>Japanese Restaurant</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Lenox Hill</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Sushi Restaurant</td>
      <td>Pizza Place</td>
      <td>Cocktail Bar</td>
      <td>Burger Joint</td>
      <td>Gym / Fitness Center</td>
      <td>Café</td>
      <td>Gym</td>
      <td>Sporting Goods Shop</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Upper West Side</td>
      <td>Italian Restaurant</td>
      <td>Wine Bar</td>
      <td>Bar</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Mediterranean Restaurant</td>
      <td>Bakery</td>
      <td>Indian Restaurant</td>
      <td>Yoga Studio</td>
      <td>Dessert Shop</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Lincoln Square</td>
      <td>Plaza</td>
      <td>Café</td>
      <td>Theater</td>
      <td>Italian Restaurant</td>
      <td>Concert Hall</td>
      <td>Performing Arts Venue</td>
      <td>Indie Movie Theater</td>
      <td>American Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Park</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Greenwich Village</td>
      <td>Italian Restaurant</td>
      <td>Clothing Store</td>
      <td>Sushi Restaurant</td>
      <td>Café</td>
      <td>Seafood Restaurant</td>
      <td>Indian Restaurant</td>
      <td>French Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Gourmet Shop</td>
      <td>Boutique</td>
    </tr>
    <tr>
      <th>19</th>
      <td>East Village</td>
      <td>Bar</td>
      <td>Wine Bar</td>
      <td>Ice Cream Shop</td>
      <td>Pizza Place</td>
      <td>Mexican Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Ramen Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Japanese Restaurant</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Manhattan Valley</td>
      <td>Pizza Place</td>
      <td>Indian Restaurant</td>
      <td>Bar</td>
      <td>Yoga Studio</td>
      <td>Coffee Shop</td>
      <td>Mexican Restaurant</td>
      <td>Deli / Bodega</td>
      <td>Thai Restaurant</td>
      <td>Playground</td>
      <td>Park</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Financial District</td>
      <td>Coffee Shop</td>
      <td>American Restaurant</td>
      <td>Bar</td>
      <td>Gym</td>
      <td>Pizza Place</td>
      <td>Gym / Fitness Center</td>
      <td>Hotel</td>
      <td>Steakhouse</td>
      <td>Food Truck</td>
      <td>Event Space</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Carnegie Hill</td>
      <td>Pizza Place</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Bookstore</td>
      <td>Cosmetics Shop</td>
      <td>Grocery Store</td>
      <td>Gym</td>
      <td>Gym / Fitness Center</td>
      <td>Bakery</td>
      <td>Japanese Restaurant</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Midtown South</td>
      <td>Korean Restaurant</td>
      <td>Hotel</td>
      <td>Hotel Bar</td>
      <td>Japanese Restaurant</td>
      <td>Dessert Shop</td>
      <td>Gym / Fitness Center</td>
      <td>Coffee Shop</td>
      <td>Cosmetics Shop</td>
      <td>American Restaurant</td>
      <td>Fried Chicken Joint</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Sutton Place</td>
      <td>Gym / Fitness Center</td>
      <td>Furniture / Home Store</td>
      <td>Italian Restaurant</td>
      <td>Gym</td>
      <td>Coffee Shop</td>
      <td>Indian Restaurant</td>
      <td>Yoga Studio</td>
      <td>Chinese Restaurant</td>
      <td>Sushi Restaurant</td>
      <td>Pilates Studio</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Turtle Bay</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Sushi Restaurant</td>
      <td>Steakhouse</td>
      <td>Wine Bar</td>
      <td>Park</td>
      <td>Ramen Restaurant</td>
      <td>Hotel</td>
      <td>Indian Restaurant</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Flatiron</td>
      <td>Yoga Studio</td>
      <td>American Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Café</td>
      <td>Japanese Restaurant</td>
      <td>Spa</td>
      <td>Mediterranean Restaurant</td>
      <td>Cycle Studio</td>
      <td>Cosmetics Shop</td>
      <td>New American Restaurant</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Hudson Yards</td>
      <td>American Restaurant</td>
      <td>Hotel</td>
      <td>Gym / Fitness Center</td>
      <td>Coffee Shop</td>
      <td>Italian Restaurant</td>
      <td>Café</td>
      <td>Thai Restaurant</td>
      <td>Park</td>
      <td>Art Gallery</td>
      <td>Spanish Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



**Cluster 2**


```python
manhattan_merged.loc[manhattan_merged['Cluster Labels'] == 1, manhattan_merged.columns[[1] + list(range(5, manhattan_merged.shape[1]))]]
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Chinatown</td>
      <td>Chinese Restaurant</td>
      <td>Cocktail Bar</td>
      <td>American Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>Hotpot Restaurant</td>
      <td>Optical Shop</td>
      <td>Spa</td>
      <td>Bakery</td>
      <td>Salon / Barbershop</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Central Harlem</td>
      <td>African Restaurant</td>
      <td>Fried Chicken Joint</td>
      <td>French Restaurant</td>
      <td>American Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Seafood Restaurant</td>
      <td>Bar</td>
      <td>Metro Station</td>
      <td>Southern / Soul Food Restaurant</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Upper East Side</td>
      <td>Art Gallery</td>
      <td>Italian Restaurant</td>
      <td>Exhibit</td>
      <td>Coffee Shop</td>
      <td>Bakery</td>
      <td>Juice Bar</td>
      <td>Gym / Fitness Center</td>
      <td>French Restaurant</td>
      <td>Hotel</td>
      <td>Pizza Place</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Clinton</td>
      <td>Theater</td>
      <td>Italian Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>American Restaurant</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Wine Shop</td>
      <td>Spa</td>
      <td>Hotel</td>
      <td>Lounge</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Midtown</td>
      <td>Hotel</td>
      <td>Coffee Shop</td>
      <td>Clothing Store</td>
      <td>Bakery</td>
      <td>Steakhouse</td>
      <td>French Restaurant</td>
      <td>Café</td>
      <td>Theater</td>
      <td>Sporting Goods Shop</td>
      <td>Cocktail Bar</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Murray Hill</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Japanese Restaurant</td>
      <td>American Restaurant</td>
      <td>Hotel</td>
      <td>Gym</td>
      <td>Italian Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Mediterranean Restaurant</td>
      <td>Bagel Shop</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Chelsea</td>
      <td>Coffee Shop</td>
      <td>Italian Restaurant</td>
      <td>Bakery</td>
      <td>Wine Shop</td>
      <td>American Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Theater</td>
      <td>Hotel</td>
      <td>Sushi Restaurant</td>
      <td>Cupcake Shop</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Lower East Side</td>
      <td>Pizza Place</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Art Gallery</td>
      <td>Japanese Restaurant</td>
      <td>Ramen Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Bakery</td>
      <td>Filipino Restaurant</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Tribeca</td>
      <td>Park</td>
      <td>American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Spa</td>
      <td>Café</td>
      <td>Wine Shop</td>
      <td>Wine Bar</td>
      <td>Boutique</td>
      <td>Coffee Shop</td>
      <td>Greek Restaurant</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Little Italy</td>
      <td>Café</td>
      <td>Bakery</td>
      <td>Bubble Tea Shop</td>
      <td>Sandwich Place</td>
      <td>Salon / Barbershop</td>
      <td>Italian Restaurant</td>
      <td>Mediterranean Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Yoga Studio</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Soho</td>
      <td>Clothing Store</td>
      <td>Boutique</td>
      <td>Art Gallery</td>
      <td>Shoe Store</td>
      <td>Women's Store</td>
      <td>Bakery</td>
      <td>Furniture / Home Store</td>
      <td>Sporting Goods Shop</td>
      <td>Mediterranean Restaurant</td>
      <td>Men's Store</td>
    </tr>
    <tr>
      <th>24</th>
      <td>West Village</td>
      <td>Italian Restaurant</td>
      <td>New American Restaurant</td>
      <td>American Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Park</td>
      <td>Wine Bar</td>
      <td>Cocktail Bar</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Bakery</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Gramercy</td>
      <td>Bar</td>
      <td>Italian Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Pizza Place</td>
      <td>Bagel Shop</td>
      <td>Thai Restaurant</td>
      <td>Thrift / Vintage Store</td>
      <td>Coffee Shop</td>
      <td>Diner</td>
      <td>Cocktail Bar</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Battery Park City</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Hotel</td>
      <td>Women's Store</td>
      <td>Memorial Site</td>
      <td>Shopping Mall</td>
      <td>Gym</td>
      <td>Wine Shop</td>
      <td>Boat or Ferry</td>
      <td>Food Court</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Noho</td>
      <td>Italian Restaurant</td>
      <td>French Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Hotel</td>
      <td>Grocery Store</td>
      <td>Art Gallery</td>
      <td>Pizza Place</td>
      <td>Coffee Shop</td>
      <td>Mexican Restaurant</td>
      <td>American Restaurant</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Civic Center</td>
      <td>Coffee Shop</td>
      <td>Gym / Fitness Center</td>
      <td>Hotel</td>
      <td>French Restaurant</td>
      <td>Yoga Studio</td>
      <td>Spa</td>
      <td>Park</td>
      <td>American Restaurant</td>
      <td>Bakery</td>
      <td>Sandwich Place</td>
    </tr>
  </tbody>
</table>
</div>



**Cluster 3**


```python
manhattan_merged.loc[manhattan_merged['Cluster Labels'] == 2, manhattan_merged.columns[[1] + list(range(5, manhattan_merged.shape[1]))]]
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>37</th>
      <td>Stuyvesant Town</td>
      <td>Park</td>
      <td>Bar</td>
      <td>Baseball Field</td>
      <td>Pet Service</td>
      <td>Gas Station</td>
      <td>Boat or Ferry</td>
      <td>Farmers Market</td>
      <td>Gym / Fitness Center</td>
      <td>Cocktail Bar</td>
      <td>Harbor / Marina</td>
    </tr>
  </tbody>
</table>
</div>



**Cluster 4**


```python
manhattan_merged.loc[manhattan_merged['Cluster Labels'] == 3, manhattan_merged.columns[[1] + list(range(5, manhattan_merged.shape[1]))]]
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Washington Heights</td>
      <td>Café</td>
      <td>Grocery Store</td>
      <td>Bakery</td>
      <td>Mobile Phone Shop</td>
      <td>Pizza Place</td>
      <td>Chinese Restaurant</td>
      <td>Tapas Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Coffee Shop</td>
      <td>Supplement Shop</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Inwood</td>
      <td>Mexican Restaurant</td>
      <td>Lounge</td>
      <td>Pizza Place</td>
      <td>Restaurant</td>
      <td>Café</td>
      <td>Spanish Restaurant</td>
      <td>Bakery</td>
      <td>Park</td>
      <td>Chinese Restaurant</td>
      <td>Frozen Yogurt Shop</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hamilton Heights</td>
      <td>Pizza Place</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Deli / Bodega</td>
      <td>Mexican Restaurant</td>
      <td>Yoga Studio</td>
      <td>Sushi Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>School</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Manhattanville</td>
      <td>Coffee Shop</td>
      <td>Deli / Bodega</td>
      <td>Italian Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Park</td>
      <td>Indian Restaurant</td>
      <td>Supermarket</td>
      <td>Boutique</td>
      <td>Spanish Restaurant</td>
    </tr>
    <tr>
      <th>7</th>
      <td>East Harlem</td>
      <td>Mexican Restaurant</td>
      <td>Bakery</td>
      <td>Latin American Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Deli / Bodega</td>
      <td>French Restaurant</td>
      <td>Spa</td>
      <td>Liquor Store</td>
      <td>Taco Place</td>
      <td>Gas Station</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Roosevelt Island</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Kosher Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Gym</td>
      <td>Greek Restaurant</td>
      <td>Dry Cleaner</td>
      <td>Outdoors &amp; Recreation</td>
      <td>Playground</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Morningside Heights</td>
      <td>Bookstore</td>
      <td>Park</td>
      <td>American Restaurant</td>
      <td>Coffee Shop</td>
      <td>Burger Joint</td>
      <td>Sandwich Place</td>
      <td>Deli / Bodega</td>
      <td>Café</td>
      <td>Seafood Restaurant</td>
      <td>Salad Place</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Tudor City</td>
      <td>Park</td>
      <td>Mexican Restaurant</td>
      <td>Café</td>
      <td>Deli / Bodega</td>
      <td>Diner</td>
      <td>Pizza Place</td>
      <td>Coffee Shop</td>
      <td>Greek Restaurant</td>
      <td>Dog Run</td>
      <td>Thai Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



**Cluster 5**


```python
manhattan_merged.loc[manhattan_merged['Cluster Labels'] == 4, manhattan_merged.columns[[1] + list(range(5, manhattan_merged.shape[1]))]]
```




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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marble Hill</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>Sandwich Place</td>
      <td>Yoga Studio</td>
      <td>Tennis Stadium</td>
      <td>Supplement Shop</td>
      <td>Donut Shop</td>
      <td>Miscellaneous Shop</td>
      <td>Steakhouse</td>
      <td>Discount Store</td>
    </tr>
  </tbody>
</table>
</div>



Cluster 2 is the biggest. Cluster 3 and 5 have only one neighborhood each.

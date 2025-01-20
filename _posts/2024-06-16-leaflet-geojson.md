---
title: "Using Leaflet GeoJSON"
date: 2024-06-16
categories: [Leaflet]
tags: [Visualization, Map, Software]
---

# Leaflet with Javascript

The way mapping software enables and transforms our ability to navigate and engage with the world around us has always captivated me. Maps are incredibly helpful in a variety of ways, including information collection, navigating, and discovery. In our increasingly linked world, mapping software has emerged as a vital tool for personal safety, business operations, exploration, and navigation.

**Leaflet** is a fantastic open-source JavaScript library for building interactive maps that are lightweight.Its many features, ease of use, and portability make it immensely popular. [Learn more about it here](https://leafletjs.com/index.html).

One open standard format for expressing geographic data is [GeoJSON](https://geojson.org/). It essentially uses the JSON standard to encode geographic data, such as points, lines, and polygons. Consider it a digital language for location-based data and maps.

Typically, a GeoJSON structure looks like below. The number of coordinates array changes depending on the geometry type.
```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [ 125.6, 10.1 ]
    },
    "properties": {
        "name": "Dinagat Islands"
    }
}
```

We will attempt to build a basic mapping website that uses the **Leaflet** javascript renderer and ingest GeoJSON data as a layer.

## Start with the basics

First, we need to create basic web based files a html, css, and javascript files.

![File structure](/assets/img/posts/2024-06-16-leaflet-geojson/file-structure.png){: width="400"}

For `index.html`, add the elements below. We are using **Leaflet** version _1.9.4_ from a hosted link. This is straightforward html and the `<div id="map"></div>` will contain the **Leaflet** map renderer we defined later.

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div id="map"></div>    
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin="" /></script>
    <script type="text/javascript" src="script.js"></script>
  </body>
</html>
```

Next, add the following to `style.css`. These are simple styling to use the entire browser real estate. Nothing out of the ordinary.

```css
body {
  margin: 0;
}
html, body, #map {
  height: 100%;
}
```

The `script.js` will hold all the logic we are building.

## Add some logic

With **Leaflet**, we need to define the root object. We will use OpenStreetMap as the base map layer. OSM is an [open access, collaborative map data](https://www.openstreetmap.org/about) that can easily be integrate to applications. Think of it as the Wikipedia for map data.

```javascript
var map = L.map('map', {
    'center': [0, 0],
    'zoom': 2,
    'layers': [
        L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            'attribution': 'Map data &copy; OpenStreetMap contributors'
        })
    ]
});
```

We need to create a control layer for our GeoJSON data. One layer is filled and the other displays the border lines between countries.

```javascript
var controlLayers = L.control.layers().addTo(map);
```

Next, we need to ingest some data. Fortunately, there are available GeoJSON data we can gather from our fellow Github user. One in particular is a `world.geo.json` from [git username _johan_](https://github.com/johan/world.geo.json).

```javascript
const urls = [
    // World GeoJSON
    'https://raw.githubusercontent.com/johan/world.geo.json/refs/heads/master/countries.geo.json'
];
```

Then, we need to display the GeoJSON data as a layer on our map. We will add two layers: one color filled in cyan and one with red border lines. 

We will use a `Promise` object to fetch the GeoJSON data since will be adding additional features later.

```javascript
Promise.all(urls.map(url => fetch(url)))
    .then(responses => Promise.all(responses.map(response => response.json())))
    .then(data => {

        const worldGeoJson = data[0];

        // Create the colored layer GeoJSON
        var coloredLayer = L.geoJson(worldGeoJson, {
            style: function (feature) {
                return {
                    'weight': 0,
                    'fillColor': 'cyan',
                    'fillOpacity': 1
                }
            }
        }).addTo(map);

        // Create the border layer GeoJSON
        var borderLayer = L.geoJson(worldGeoJson, {
            style: function (feature) {
                return {
                    'weight': 1,
                    'color': 'red',
                    'fillOpacity': 0
                }
            },
            pane: 'countryBorder'
        }).addTo(map);

        // Add both layers to control
        controlLayers.addOverlay(coloredLayer, 'Colored Layer');
        controlLayers.addOverlay(borderLayer, 'Border Layer');

    })
    .catch(error => {
        // Handle any errors
        console.error(error);
    });
```

The script should pull the GeoJSON layers and display the map like below.

![Full map](/assets/img/posts/2024-06-16-leaflet-geojson/full-map-layer.png){: width="600"}

> **_Note_**: We can toggle the control layer on the top right candidly.
{: .prompt-tip }

## Lets add more logic

Now, lets make some changes so that when we click on a country to get additional information about it.

First, we need to update the `urls` variable to include data from a list of countries with population and life expectancy. Again, we are borrowing data from a fellow GitHub user. In this case, it is a `country-json` from [git username _samayo_](https://github.com/samayo/country-json).

Extend the urls array to include additional data.

```javascript
const urls = [
    // World GeoJSON
    'https://raw.githubusercontent.com/johan/world.geo.json/refs/heads/master/countries.geo.json',

    // Countries by Population
    'https://raw.githubusercontent.com/samayo/country-json/refs/heads/master/src/country-by-population.json',

    // Countries by Life Expectancy
    'https://raw.githubusercontent.com/samayo/country-json/refs/heads/master/src/country-by-life-expectancy.json'
];
```

Inside the Promise object `.then()` clause, set the two additional json data.
```javascript
const countriesByPopulation = data[1];
const countriesByLifeExpentancyJson = data[2];
```

Then, inside the Promise object `.then()` clause again, add a `.bindPopup()` to extend the `borderLayer` object. It should look like below. Essentially, the logic will iterate through both population list and life expentancy list to find a matching country name and use its value; will default to _n/a_ if not found.

```javascript
var borderLayer = L.geoJson(worldGeoJson, {
    style: function (feature) {
        return {
            'weight': 1,
            'color': 'red',
            'fillOpacity': 0
        }
    },
    pane: 'countryBorder'
}).bindPopup(function (layer) {
    var s = `<strong>${layer.feature.properties.name}</strong>`;

    // Add Population
    var population = countriesByPopulation.find(e => e.country == layer.feature.properties.name)?.population ?? "n/a";
    s += `<br />Population: ${population}`;

    // Add Life Expectancy
    var lifeExpectancy = countriesByLifeExpentancyJson.find(e => e.country == layer.feature.properties.name)?.expectancy ?? "n/a";
    s += `<br />Life Expectancy: ${lifeExpectancy}`;

    return s;
}).addTo(map);
```

Back on the map, if we click on any country, we should see both the population and life expectancy information.

![Map country](/assets/img/posts/2024-06-16-leaflet-geojson/map-country-click.png){: width="600"}

Since we are linking `.bindPopup()` to `borderLayer`, lets make sure that layer remains on top by updating its z-index. If you noticed, there is a `pane` defined in the `borderLayer` options. We will update the `map` object to add this same `pane` name and set a z-index value.

```javascript
map.createPane('countryBorder');
map.getPane('countryBorder').style.zIndex = 650;
```

## Host the website

To display this as a full static web page, we can invoke Python's simple http server with a command. This requires having Python installed on your system. See [Python downloads](https://www.python.org/downloads/) if you need to install it.

```python
python -m http.server 8080 
```

Navigate to **_hostname:port_** in your local browser *(e.g. locally by default at localhost:8080)* 


## Full script code

For a full `script.js` code, see below.

```javascript
// Create a map using OpenStreetMap
var map = L.map('map', {
    'center': [0, 0],
    'zoom': 2,
    'layers': [
        L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            'attribution': 'Map data &copy; OpenStreetMap contributors'
        })
    ]
});

// Create a control layer to toggle other layers
var controlLayers = L.control.layers().addTo(map);

// JSON data from git fellows. Thank you!
const urls = [
    // World GeoJSON
    'https://raw.githubusercontent.com/johan/world.geo.json/refs/heads/master/countries.geo.json',

    // Countries by Population
    'https://raw.githubusercontent.com/samayo/country-json/refs/heads/master/src/country-by-population.json',

    // Countries by Life Expectancy
    'https://raw.githubusercontent.com/samayo/country-json/refs/heads/master/src/country-by-life-expectancy.json'
];

// Using Promise.all() to handle multiple fetches
Promise.all(urls.map(url => fetch(url)))
    .then(responses => Promise.all(responses.map(response => response.json())))
    .then(data => {

        // Set readable vars from all the fetches
        const worldGeoJson = data[0];
        const countriesByPopulation = data[1];
        const countriesByLifeExpentancyJson = data[2];

        // Create a colored layer geo json
        var coloredLayer = L.geoJson(worldGeoJson, {
            style: function (feature) {
                return {
                    'weight': 0,
                    'fillColor': 'cyan',
                    'fillOpacity': 1
                }
            }
        }).addTo(map);

        // Create a border layer geo json with pop ups
        var borderLayer = L.geoJson(worldGeoJson, {
            style: function (feature) {
                return {
                    'weight': 1,
                    'color': 'red',
                    'fillOpacity': 0
                }
            },
            pane: 'countryBorder'
        }).bindPopup(function (layer) {
            var s = `<strong>${layer.feature.properties.name}</strong>`;

            // Include population
            var population = countriesByPopulation.find(e => e.country == layer.feature.properties.name)?.population ?? "n/a";
            s += `<br />Population: ${population}`;

            // Include life expectancy
            var lifeExpectancy = countriesByLifeExpentancyJson.find(e => e.country == layer.feature.properties.name)?.expectancy ?? "n/a";
            s += `<br />Life Expectancy: ${lifeExpectancy}`;

            return s;            
        }).addTo(map);

        // Add both layers to control
        controlLayers.addOverlay(coloredLayer, 'Colored Layer');
        controlLayers.addOverlay(borderLayer, 'Border Layer');
    })
    .catch(error => {
        // Handle any errors
        console.error(error);
    });

// Make sure border layer is always on top using a higher z-index number
map.createPane('countryBorder');
map.getPane('countryBorder').style.zIndex = 650;

// Host with
// python -m http.server 8080
```
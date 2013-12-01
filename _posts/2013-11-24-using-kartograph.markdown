---
layout: post
title:  "Kartograph"
date:   2013-11-24 00:16:33
categories: kartograph geomapping  
---


<h1 style="font-size:3em;"> Kartograph - getting started </h1>

Assume you want to display a map in your web page that visualizes some country level data in world map. There are several ways to do this, you could use Google maps, leaflet or polymaps. But all these services depend on a external service provider to render the maps or to get map tile images. But your requirement could be simple, you just want to display a static choropleth or a static map but with some interactive capabilities, then all you need is Kartograph.  

What is Kartograph?
===================
Kartograph is a simple and lightweight framework for building interactive map applications without Google Maps or any other mapping service. It was created with the needs of designers and data journalists in mind.

Actually, Kartograph is two libraries. One generates beautiful & compact SVG maps; the other helps you to create interactive maps that run across all major browsers.

creating maps using Kartograph is pretty simple. Infact, it involves 3 simple steps.

<img src="/img/kartograph.png" height="400" width="640">

Step 1
======

A [shape file](https://en.wikipedia.org/wiki/Shapefile) saves geo-spatial data that could be used by Geographical Information Systems ([GIS](https://en.wikipedia.org/wiki/Geographic_information_system)). Shapefiles basically provide information about the lines, paths and polygons that constitute the map of a given region. Shape files for different countries and locations can be downloaded for free from the various websites. I have listed few of them below.

  + http://www.diva-gis.org
  + http://geocommons.com
  + http://gadm.org
  + http://www.naturalearthdata.com  

Kartograph.py is a python library that takes the *shape file* as input and provides a *svg image* map as output. The beauty of this process is the svg paths can also contain some information, say for e.g if you generate an svg map of the  world, the svg paths could contain the information about which paths correspond to which country. The svg could also contain additional information as layers.


Learning by building an example is always a good way to learn something. Lets do the same here. Lets build a choropleth map of the world. For people who are wondering what a choropleth is? Take a look at the image below. This is a cholorpleth that is made from google geo charting. We will see how to develop a similar cholorpleth map using kartograph.
    
<img src="/img/google_geochart.png" height="400" width="640" />

 First of all, we need a shape file which contains information about the world countries and their boundaries. One such file could be downloaded from the [link](http://geocommons.com/overlays/5603). Now, we will convert this shapefile into svg using Kartograph.py. Now extract the downloaded zip file, after extracting you will find files with 3 different extensions namely .shp, .dbp, .shx. place all of the files in the same folder. 

Step 2
======

Kartograph.py could be used as a command line tool as well as a python library. We will use the command line mode in this example. Kartograph.py needs a configuration file which contains information about the details that should included in the generated svg map. create a configuration file called worldconfig.json and copy the below code in to that.

```javascript
{
    "layers":{
        "world": { 
            "src": "world_countries_boundary_file_world_2002.shp", 
            "simplify": 2,
            "attributes" : {
                    "iso3":"ISO_3_CODE"
                }
            }
    }
}
```

The _src_ key in the json should point to the shape file that we downloaded now. The _simplify_ key is to instruct the Kartograph to perform topology preserving simplifications in the paths in the map. This will help to reduce the size of the generated svg. But be careful, too much simplification will make the map look totally different and make it useless. The _attributes_ key is very special, it instructs the Kartograph to copy the attribute ISO_3_CODE in the shapefile to the svg. By copying the iso3 code in to svg, we can easily identify the paths that are corresponding to a given country.

To generate the svg, run the below commands in the console.

```
$ export KARTOGRAPH_DATA="<path of the directory containing shapefiles>"
$ kartograph worldconfig.json -o world.svg
```
A svg image is nothing but a xml file. SVG is a markup language that contains information about the paths is a drawing. Now inspect the svg file in a text editor to see how it contains the information about iso3 attribute. We will use this attribute in the step 3 to color the required regions of the map.

Step3
=====

The choropleth that is implemented using Kartograph.js is shown below. No need to rely on any external service, we have our pretty geochart that is self contained. The corresponding code with explanatory comments is also presented below.

<div id="map">&nbsp;</div>

<link rel="stylesheet" type="text/css" href="/css/jquery.qtip.css" />

<script type="text/javascript" src="/resources/js/jquery-1.10.2.min.js"></script>
<script type="text/javascript" src="/resources/js/raphael-2.1.0.min.js"></script>
<script type="text/javascript" src="/resources/js/jquery.qtip.min.js"></script>
<script type="text/javascript" src="/resources/js/chroma.min.js"></script>
<script type="text/javascript" src="/resources/js/kartograph.min.js"></script>

<script type="text/javascript">
    $(function() {
        var map,
            colorscale,
            countryPopularity = {};
        // initialize qtip tooltip class
        $.fn.qtip.defaults.style.classes = 'ui-tooltip-bootstrap';
        $.fn.qtip.defaults.style.def = false;
        $.getJSON('/resources/kartograph/popularity.json', function(countryPopularity) {
            $.get('/resources/kartograph/world.svg', function(svg) {
                    var div = $('#map');
                    var map = kartograph.map(div, 640, 420),
                    color = chroma.scale('Greens').domain(countryPopularity,5,'quantiles','popularity');
                    console.log(color.domain());
                    console.log(color(1));
                    map.setMap(svg);
                    map.addLayer('world', {
                        styles: {
                            'stroke-width': 0.7,
                            fill: function(d) { return color(countryPopularity[d.iso3]?countryPopularity[d.iso3].popularity:0); },
                            stroke: function(d) {return color(countryPopularity[d.iso3]?countryPopularity[d.iso3].popularity:0).darker(); }
                        },
                        tooltips: function(d) {
                            return [d.iso3, countryPopularity[d.iso3]?countryPopularity[d.iso3].popularity:0];
                        }
                    });

            });
        });

    });
</script>

```html

<!-- define a div where the map should appear -->
<div id="map">&nbsp;</div>


<!-- include the dependencies and kartograph.js at the last-->
<link rel="stylesheet" type="text/css" href="/css/jquery.qtip.css" />
<script type="text/javascript" src="/resources/js/jquery-1.10.2.min.js"></script>
<script type="text/javascript" src="/resources/js/raphael-2.1.0.min.js"></script>
<script type="text/javascript" src="/resources/js/jquery.qtip.min.js"></script>
<script type="text/javascript" src="/resources/js/chroma.min.js"></script>
<script type="text/javascript" src="/resources/js/kartograph.min.js"></script>

<script type="text/javascript">
    $(function() {
        var map,
            colorscale,
            countryPopularity = {};
        $.fn.qtip.defaults.style.classes = 'ui-tooltip-bootstrap';
        $.fn.qtip.defaults.style.def = false;
        /*
        * Now lets get the data
        */
        $.getJSON('/resources/kartograph/popularity.json', function(countryPopularity) {
            /*
            * once we have data, get the svg
            */
            $.get('/resources/kartograph/world.svg', function(svg) {
                var div = $('#map');
                // declare the dimensions of the map
                var map = kartograph.map(div, 640, 420),
                /*
                * choose the color scale for the choropleth,
                * consult chroma.js api docs for more details
                */
                color = chroma.scale('Greens').
                        domain(countryPopularity,5,'quantiles','popularity');
                map.setMap(svg);
                /* 
                * now add the layer "world" which contains the paths for the country 
                * boundaries, the map will appear empty till this is added
                */
                map.addLayer('world', {
                    styles: {
                        'stroke-width': 0.7,
                        /* 
                        * pass the color scale function to the fill and stroke attribute
                        */
                        fill: function(d) { 
                            return color(countryPopularity[d.iso3]?
                                         countryPopularity[d.iso3].popularity:
                                         0); 
                            },
                        stroke: function(d) {
                            return color(countryPopularity[d.iso3]?
                                         countryPopularity[d.iso3].popularity
                                         :0).darker();
                          }
                    },
                    /* 
                    * set tooltip content
                    */
                    tooltips: function(d) {
                        return [d.iso3, 
                                countryPopularity[d.iso3]?
                                    countryPopularity[d.iso3].popularity:
                                    0
                                ];
                    }
                });
            });
        });

    });
</script>
```

So, this is just a glimpse what is possible with Kartograph. I would highly recommend to go through Kartograph [showcase](http://kartograph.org/showcase/), which showcases the other several possibilites of Kartograph. Happy Kartographing.

References:

+ http://kartograph.org/docs/kartograph.py/
+ http://kartograph.org/docs/kartograph.js/
+ http://kartograph.org/showcase/
+ https://github.com/gka/chroma.js

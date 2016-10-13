## panama-graph-map

This is a very simple demo on how to visualize locations of Addresses from the #PapamaPapers using MapBox.

The code was shamelessly stolen from [legis-graph-spatial](https://github.com/legis-graph/legis-graph-spatial) thanks [Will Lyon](http://twitter.com/lyonwj)!

### Tools used

* Neo4j
* Cypher
* Bolt Javascript Driver
* APOC procedures (geocode)
* MapBox

<img src="https://dl.dropboxusercontent.com/u/14493611/panama_map.jpg" width="400"/>

### Download the Database from ICIJ

Take the #PanamaPapers Neo4j database from the ICIJ's download site: https://offshoreleaks.icij.org/pages/database

Unzip and run the database following the instructions in the readme.

### GeoCode Addresses

First geocode addresses of a country, like Denmark (DNK).

It uses the [apoc procedure library](https://github.com/neo4j-contrib/neo4j-apoc-procedures) with is coming out of the box with the ICIJ database download.

Just run in your Neo4j Browser:

```
MATCH (a:Address) WHERE a.country_codes = "DNK" 
CALL apoc.spatial.geocodeOnce(a.address) YIELD location
SET a += location
```

### Run this Project

Now you can run this project (which is currently pinned to Denmark).

E.g. by using `python -m SimpleHTTPServer` and then open http://localhost:8000

Click on the country and the addresses are queried from Neo4j and rendered on the map with additional information taken from the graph.

### Spatial Query

```
// graph pattern we're looking for
MATCH (a:Address)<--(officer:Officer)-[role]->(entity:Entity)

// in this country with a location
WHERE a.country_codes = {country} AND exists(a.longitude) AND 

// and in this distance from the click point
distance(point({pos}), point(a)) < ({distanceInKm} * 1000)

// return positions and information from the graph pattern
RETURN a.latitude, a.longitude, 
{officer : officer.name, entity: entity.name, role: type(role),  
 address : a.address,   country: entity.country_codes} AS data
```

### Working with Mapbox

~~~ javascript

L.mapbox.accessToken = MB_API_TOKEN;
var map = L.mapbox.map('map', 'mapbox.streets')
  .setView([39.8282, -98.5795], 5);

map.on('click', function(e) {
  clearMap(map);
  getAddresses("DNK",e.latlng, 50)
});

~~~

**Create the map and define a click handler for the map.**

~~~ javascript

function getAddresses(country, pos, distance) {
    
    var addressParams = {
        "country": country,
        "longitude": pos.lng, 
        "latitude": pos.lat,
        "distanceInKm": distance || 50
    }

    session
        .run(addressQuery, addressParams)
        .then(function(result){
            result.records.forEach(function(record) {
                addAddress(record.get("latitude"),record.get("longitude"),
                           record.get("text"));
            });

        })
}


~~~


#### Rendering Addresses On Map 

~~~ javascript

function addAddress(lat,lon, data) {
   L.marker([lat,lon],{title:JSON.stringify(data)}).addTo(map);
}

~~~


---
pagetitle: Leaflet for R - Markers
maps: true
---

## Markers

Use markers to call out points on the map. Marker locations are expressed in latitude/longitude coordinates, and can either appear as icons or as circles. You can use `SpatialPoints`, `SpatialPointsDataFrame`, or a two-column numeric matrix (first column is longitude, second column is latitude) as data objects.

You can also use a data frame with latitude and longitude columns. You can explicitly tell the marker function which columns contain the coorinate data (e.g. `addMarker(lng = ~Longitude, lat = ~Latitude)`), or let the function look for columns named `lat`/`latitude` and `lon`/`lng`/`long`/`longitude` (case insensitive).

### Icon Markers

Icon markers are added using the `addMarker` function. Their default appearance is a dropped pin. As with most layer functions, the `popup` argument can be used to add a message to be displayed on click.

```{r}
data(quakes)

# Show first 20 rows from the `quakes` dataset
leaflet(data = quakes[1:20,]) %>% addTiles() %>%
  addMarkers(~long, ~lat, popup = ~as.character(mag))
```

#### Customizing Marker Icons

You can provide custom markers in one of several ways, depending on the scenario. For each of these ways, the icon can be provided as either a URL or as a file path.

For the simple case of applying a single icon to a set of markers, use `makeIcon()`.

```{r fig.height=1.75}
greenLeafIcon <- makeIcon(
  iconUrl = "http://leafletjs.com/docs/images/leaf-green.png",
  iconWidth = 38, iconHeight = 95,
  iconAnchorX = 22, iconAnchorY = 94,
  shadowUrl = "http://leafletjs.com/docs/images/leaf-shadow.png",
  shadowWidth = 50, shadowHeight = 64,
  shadowAnchorX = 4, shadowAnchorY = 62
)

leaflet(data = quakes[1:4,]) %>% addTiles() %>%
  addMarkers(~long, ~lat, icon = greenLeafIcon)
```

If you have several icons to apply that vary only by a couple of parameters (i.e. they share the same size and anchor points but have different URLs), use the `icons()` function. `icons()` performs similarly to `data.frame()`, in that any arguments that are shorter than the number of markers will be recycled to fit.

```{r fig.height=2}
quakes1 <- quakes[1:10,]

leafIcons <- icons(
  iconUrl = ifelse(quakes1$mag < 4.6,
    "http://leafletjs.com/docs/images/leaf-green.png",
    "http://leafletjs.com/docs/images/leaf-red.png"
  ),
  iconWidth = 38, iconHeight = 95,
  iconAnchorX = 22, iconAnchorY = 94,
  shadowUrl = "http://leafletjs.com/docs/images/leaf-shadow.png",
  shadowWidth = 50, shadowHeight = 64,
  shadowAnchorX = 4, shadowAnchorY = 62
)

leaflet(data = quakes1) %>% addTiles() %>%
  addMarkers(~long, ~lat, icon = leafIcons)
```

Finally, if you have a set of icons that vary in multiple parameters, it may be more convenient to use the `iconList()` function. It lets you create a list of (named or unnamed) `makeIcon()` icons, and select from that list by position or name.

```{r fig.height=1.75}
# Make a list of icons. We'll index into it based on name.
oceanIcons <- iconList(
  ship = makeIcon("ferry-18.png", "ferry-18@2x.png", 18, 18),
  pirate = makeIcon("danger-24.png", "danger-24@2x.png", 24, 24)
)

# Some fake data
df <- sp::SpatialPointsDataFrame(
  cbind(
    (runif(20) - .5) * 10 - 90.620130,  # lng
    (runif(20) - .5) * 3.8 + 25.638077  # lat
  ),
  data.frame(type = factor(
    ifelse(runif(20) > 0.75, "pirate", "ship"),
    c("ship", "pirate")
  ))
)

leaflet(df) %>% addTiles() %>%
  # Select from oceanIcons based on df$type
  addMarkers(icon = ~oceanIcons[type])
```

#### Marker Clusters

When there are a large number of markers on a map, you can cluster them using the [Leaflet.markercluster](https://github.com/Leaflet/Leaflet.markercluster) plug-in. To enable this plug-in, you can provide a list of options to the argument `clusterOptions`, e.g.

```{r fig.height=2.5, message=FALSE}
leaflet(quakes) %>% addTiles() %>% addMarkers(
  clusterOptions = markerClusterOptions()
)
```

### Circle Markers

Circle markers are much like regular circles (see [Lines and Shapes](shapes.html)), except that their radius in onscreen pixels stays constant regardless of zoom level.

You can use their default appearance:

```{r fig.height=1.75}
leaflet(df) %>% addTiles() %>% addCircleMarkers()
```

Or customize their color, radius, stroke, opacity, etc.

```{r fig.height=1.75}
# Create a palette that maps factor levels to colors
pal <- colorFactor(c("navy", "red"), domain = c("ship", "pirate"))

leaflet(df) %>% addTiles() %>%
  addCircleMarkers(
    radius = ~ifelse(type == "ship", 6, 10),
    color = ~pal(type),
    stroke = FALSE, fillOpacity = 0.5
  )
```

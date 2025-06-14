---
date: 2025-06-09T19:27:35-06:00
# description: ""
# image: ""
lastmod: 2025-06-09
showTableOfContents: true
tags: ["thesis", "geospatial data", "api", "python"]
title: "Creating venues database"
type: "post"
---

The goal of this process is to retrieve venues efficiently, prioritizing free
APIs (OSM, Foursquare) and using Google Places only as a fallback.

## Grid creation

* Objective: Create grids over the map to use the APIs over a smaller area to
handle their respective results limit. 
* Tools: `geopandas`, `shapely`,

### Step by step

#### 1. Import and convert to UTM Zone 13N

The imported shape file is using `EPSG:4326` format, which means that the data
is in degrees (latitude/longitude), that can distort distances.

To ensure the grid accuracy, it is necessary to use meters by reproject the data
to [Guadalajara's UTM Zone](https://epsg.io/32613): **UTM Zone 13**
(`EPSG:32613`).

#### 2. Create grids

First, a square covering all of the neighborhoods is created by getting the
bounds (square corners). Then that square is divided in 1km x 1km squares:

![Grid over Zapopan and Guadalajara, Jalisco, Mexico](/images/masters_thesis/grids_zmg.png)

Then, since we're interested on avoiding areas outside the studied neighborhoods,
we only keep grids that are intersecting with the neighborhoods on the shape
file:

![Grid overlaying Zapopan and Guadalajara's neighborhoods](/images/masters_thesis/overlay_grids_zmg.png)

#### 3. Get grids' center (lat/lon)

Since the APIs use latitude and longitude (degrees), the data is turned back to
degrees (`EPSG:4326`) and the center for each grid is retrieved. The associated
neighborhood for each grid is also added to easily filter by it.

The final output includes, the geometry of the 1x1km square, the latitude and
longitude of the center of the grids (for the APIs) and the neighborhood name:

| neighborhood   |   api_lon |   api_lat | geometry                                           |
|:---------------|----------:|----------:|:---------------------------------------------------|
| La primavera   |  -103.571 |   20.717  | POLYGON ((649342.528... 2290987.127)) |
| La primavera   |  -103.571 |   20.7261 | POLYGON ((649342.528... 2291987.127)) |
| Del carril     |  -103.571 |   20.7261 | POLYGON ((649342.528... 2291987.127)) |
| La primavera   |  -103.57  |   20.7351 | POLYGON ((649342.528... 2292987.127)) |
| El calabozo    |  -103.561 |   20.7169 | POLYGON ((650342.528... 2290987.127)) |



### Code

```py
utm = "EPSG:32613"  # UTM Zone 13N for Jalisco
def get_grids(
        gdf:gpd.GeoDataFrame,
        utm:str,
        grid_size:int=1000) -> gpd.GeoDataFrame:
    '''
    Creates a grid map over given neighborhoods (GeoDataFrame) with the
    objective of using those grids for maps APIs later and handle the
    results limit for each API.

    :param gpd.GeoDataFrame gdf: neighborhoods geo data
    :param str utm: UTM Zone for the analyzed geographic area
    :param int grid_size: side size of the square (in meters)
    :return: geodataframe with the grids' center (lat/lon), geometry and
    associated neighborhood
    :rtype: gpd.GeoDataFrame
    '''
    gdf = gdf.to_crs(utm)

    # Get total bounds of all neighborhoods
    minx, miny, maxx, maxy = gdf.total_bounds

    # Create 1km grid cells
    x_coords = np.arange(minx, maxx, grid_size)
    y_coords = np.arange(miny, maxy, grid_size)
    grid_cells = [
        box(x, y, x + grid_size, y + grid_size) 
        for x in x_coords 
        for y in y_coords
    ]

    # Keep overlapping neighborhoods only
    grid = gpd.GeoDataFrame(geometry=grid_cells, crs=utm)
    grid = grid[grid.intersects(gdf.union_all())]

    # Spatial join to assign neighborhood names to grid cells
    grid = gpd.sjoin(
        grid, gdf[["neighborhood", "geometry"]],
        how="left",
        predicate="intersects"
        )

    # Add centroid coordinates (for API calls)
    grid["center"] = grid.geometry.centroid

    # Convert centroids back to WGS84 (EPSG:4326) for APIs
    grid["center_wgs84"] = grid["center"].to_crs("EPSG:4326")
    grid["api_lon"] = grid["center_wgs84"].x
    grid["api_lat"] = grid["center_wgs84"].y

    return grid[["neighborhood", "api_lon", "api_lat", "geometry"]]
```

<!--delete from here #######################################################-->
<iframe
 width="100%" height="500px"
src='/graphs/folium_map_test.html'></iframe>
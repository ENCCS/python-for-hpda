# Visualising Large Datasets

:::{objectives}

- Understand why traditional plotting methods struggle with large datasets.
- Create static visualisations of tens of millions of points using Datashader.
- Visualise geographic point data from the NYC Taxi dataset.
- Build interactive visualisations using HoloViews.
- Learn how aggregation-based visualisation differs from point-based rendering.
:::

:::{questions}

- Why do plotting libraries become slow for large datasets?
- How can we visualise millions of GPS locations efficiently?
- What is the difference between plotting points and plotting aggregates?
- How can interactive visualisation remain responsive with large datasets?
:::

## Motivation

The NYC Taxi dataset contains records of taxi trips in New York City and is a
classic benchmark dataset for data analysis, since a single year can contain
hundreds of millions of trips. Suppose we want to visualise every taxi pickup
location: naively, we would create a marker for each pickup/dropoff. However,
this approach becomes very expensive for large datasets.

In this lesson we will learn a different approach:

1. Aggregate data into pixels.
2. Render aggregate statistics rather than individual points.
3. Create interactive visualisations that remain responsive even for very large
datasets.

---

## Traditional plotting

Imagine a dataset containing 100 million taxi pickups (we are using a subset in this case). A scatter plot in Matplotlib would attempt to render 100M markers. While rendering each marker is quick, the sheer scale of the dataset makes this operation expensive at

Furthermore:

- many points overlap
- dense regions become saturated
- important structures disappear

The problem is not only performance.

It is also visual clarity.

:::{figure} ../img/placeholder-overplotting.png
:alt: Overplotting concept

Overplotting occurs when so many points occupy the same screen area that structures become difficult to interpret.
:::

### Datashader's approach

Datashader does not attempt to draw every point.

Instead, it computes values for screen pixels.

Conceptually:

```
taxi pickups
      ↓
aggregate into pixels
      ↓
compute count per pixel
      ↓
render image
```

This means computational cost depends primarily on image size rather than the number of markers displayed.

:::{keypoints}

- Datashader visualises aggregates instead of graphical objects.
- Performance depends largely on output resolution.
- Dense datasets often become easier to interpret.
:::

---

### Loading taxi data

We start with a Polars dataframe containing pickup coordinates.

```python
import polars as pl

rides = pl.read_parquet("yellow_tripdata_2023.parquet")

rides.select(
    "pickup_longitude",
    "pickup_latitude"
).head()
```

Before plotting, inspect the shape:

```python
rides.shape
```

Typical datasets may contain millions of rows.

#### Cleaning coordinates

Real-world datasets often contain:

- missing values
- invalid coordinates
- outliers

Let's filter obvious errors.

```python
rides_clean = rides.filter(
    pl.col("pickup_longitude").is_not_null()
    & pl.col("pickup_latitude").is_not_null()
    & pl.col("pickup_longitude").is_between(-75, -72)
    & pl.col("pickup_latitude").is_between(40, 42)
)
```

---

### Exercise: Inspect the data

::::{exercise}
Determine:

1. Number of rides before filtering.
2. Number of rides after filtering.
3. Percentage of rows removed.

Use Polars expressions where possible.
::::

::::{solution}

```python
before = rides.height
after = rides_clean.height

removed = 100 * (before - after) / before

print(f"Before: {before}")
print(f"After : {after}")
print(f"Removed: {removed:.2f}%")
```

::::

---

## Static visualisation with Datashader

### Creating a canvas

Datashader operates through a canvas.

The canvas specifies:

- image width
- image height
- coordinate ranges

```python
import datashader as ds

canvas = ds.Canvas(
    plot_width=800,
    plot_height=600
)
```

---

### Converting Polars to Pandas

Datashader currently operates most naturally with Pandas dataframes.

```python
df = rides_clean.select(
    "pickup_longitude",
    "pickup_latitude"
).to_pandas()
```

For interactive workflows this conversion cost is often acceptable since the visualisation stage dominates.

---

### Rasterising points

Now aggregate pickup locations.

```python
agg = canvas.points(
    df,
    "pickup_longitude",
    "pickup_latitude"
)
```

The result is not yet an image.

It is a two-dimensional array containing counts per pixel.

---

### Shading

Transform counts into colours.

```python
from datashader import transfer_functions as tf

img = tf.shade(
    agg,
    cmap=["lightblue", "darkblue"]
)

img
```

You should now see the spatial distribution of taxi pickups.

---

### What can we observe?

Dense clusters usually appear in:

- Manhattan
- JFK Airport
- LaGuardia Airport

Connections between dense clusters frequently reveal:

- major roads
- bridges
- tunnels

Interestingly, these structures emerge purely from point density.

No street network data was required.

---

### Exercise: Compare resolutions

::::{exercise}
Generate two visualisations:

1. 400 × 300 pixels
2. 1600 × 1200 pixels

Compare:

- rendering speed
- visual detail

What changes and what stays the same?
::::

::::{solution}

```python
small = ds.Canvas(
    plot_width=400,
    plot_height=300
)

large = ds.Canvas(
    plot_width=1600,
    plot_height=1200
)

agg_small = small.points(
    df,
    "pickup_longitude",
    "pickup_latitude"
)

agg_large = large.points(
    df,
    "pickup_longitude",
    "pickup_latitude"
)
```

The larger canvas contains more pixels and therefore more detail. The underlying dataset remains unchanged.
::::

---

### Visualising another variable

Instead of counting rides, we can aggregate other quantities.

For example, average trip distance.

```python
agg_distance = canvas.points(
    df,
    "pickup_longitude",
    "pickup_latitude",
    agg=ds.mean("trip_distance")
)
```

Render:

```python
img = tf.shade(agg_distance)
```

This shows how aggregated statistics can be mapped spatially.

---

### Discussion

Think about the following question:

::::{discussion}
Why might a density map be more informative than plotting all taxi locations directly?
::::

Possible answers:

- less visual clutter
- improved performance
- easier identification of hotspots
- visibility of spatial structures

---

## Interactive visualisation with HoloViews

Static images are useful, but exploration often requires:

- zooming
- panning
- filtering
- comparing subsets

HoloViews provides a high-level interface for this.

---

### Initial setup

```python
import holoviews as hv

hv.extension("bokeh")
```

---

### Creating a point dataset

```python
points = hv.Points(
    df,
    kdims=[
        "pickup_longitude",
        "pickup_latitude"
    ]
)
```

We could display this directly:

```python
points
```

For small datasets this works well.

For large datasets it becomes slow.

---

### Datashading interactively

Instead of rendering all points:

```python
from holoviews.operation.datashader import datashade

interactive_map = datashade(points)

interactive_map
```

Now:

- zooming triggers re-aggregation
- plots remain responsive
- full-resolution data remain available

This is one of the key advantages of combining HoloViews and Datashader.

---

### Dynamic exploration

We can create subsets.

For example:

```python
airport_rides = df.query(
    "pickup_longitude < -73.7"
)
```

Create another view:

```python
airport_points = hv.Points(
    airport_rides,
    kdims=[
        "pickup_longitude",
        "pickup_latitude"
    ]
)

datashade(airport_points)
```

---

### Exercise: Compare day and night

::::{exercise}
Create separate visualisations for:

- daytime rides
- nighttime rides

Compare spatial patterns.

Questions:

- Which areas are active throughout the day?
- Which areas become more prominent at night?
::::

Hints:

```python
pickup_datetime
```

can be converted into an hour value and used for filtering.

::::{solution}

```python
rides_day = rides.filter(
    pl.col("hour").is_between(6, 18)
)

rides_night = rides.filter(
    ~pl.col("hour").is_between(6, 18)
)
```

Build separate HoloViews objects and compare them side by side.
::::

---

## Building an interactive dashboard

A natural next step is to use Panel.

```python
import panel as pn
```

Widgets can control:

- time of day
- passenger count
- payment type
- trip distance

A typical workflow becomes

```text
Polars
   ↓
filter
   ↓
HoloViews
   ↓
Datashader
   ↓
Panel dashboard
```

This architecture scales surprisingly well because rendering remains aggregation-based.

---

### Exercise: Design a dashboard

::::{exercise}
Work in pairs.

Design an interactive taxi exploration dashboard.

Decide:

- which filters are useful
- which variables should control colour
- what questions the dashboard should answer

Sketch the layout on paper before implementing anything.
::::

---

## Summary

Datashader changes the visualisation problem.

Instead of rendering graphical objects:

```text
data → points → screen
```

it performs:

```text
data → aggregation → image
```

This allows visualisation of datasets that would overwhelm traditional plotting tools.

HoloViews then adds an interactive layer, making it possible to explore the complete dataset rather than a sample.

:::{keypoints}

- Large datasets often require different visualisation techniques rather than faster hardware.
- Datashader aggregates data into pixels instead of rendering markers.
- HoloViews provides interactive exploration on top of Datashader.
- Polars and Datashader work well together for large-scale analysis workflows.
- Geographic datasets such as NYC Taxi data are particularly well suited for density-based visualisation.
:::

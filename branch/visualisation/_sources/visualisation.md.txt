# Visualising large datasets with Datashader and HoloViews

:::{objectives}

- Understand why traditional plotting approaches struggle with large datasets.
- Learn the basic ideas behind Datashader's aggregation-based rendering model.
- Create static visualisations of the NYC Taxi dataset using Datashader.
- Use HoloViews and hvPlot to build interactive visualisations that remain
responsive when exploring millions of records.
- Overlay aggregated data on geographic map tiles and use interactive
exploration to discover patterns in the data.
:::

:::{questions}

- Why do scatter plots become both slow and difficult to interpret for large datasets?
- What does Datashader do differently from traditional plotting libraries?
- How can we visualise millions of taxi trips without reducing the dataset to a small sample?
- How can interactive visualisation remain responsive even when the underlying dataset is very large?
:::

## Motivation

The NYC Taxi dataset from the previous episode has become a classic example in
data science and visualisation. A single year of taxi trips contains millions
of records, each associated with a pickup location, a dropoff location,
timestamps, fares, distances, and other attributes.

Suppose we want to answer a simple question:

> Where are taxis picked up in New York City?

The most direct approach is to create a scatter plot of all pickup locations:

```python
df.plot.scatter(
    x="pickup_longitude",
    y="pickup_latitude"
)
```

This sounds reasonable, but for a sufficiently large dataset two problems
quickly appear: first, plotting becomes slow because the plotting library
attempts to draw enormous numbers of graphical objects. Second, the resulting
figure is often difficult to interpret. Dense regions become saturated with
points and important structures disappear beneath a cloud of overlapping
markers.

In this episode we will explore an alternative approach. Rather than plotting
every individual observation, we will aggregate observations into pixels and
visualise those aggregates. This gives us both better performance and, somewhat
surprisingly, more informative visualisations.

## The challenge of overplotting

Before introducing any new tools, it is worth reflecting on why a traditional
scatter plot struggles.

Imagine plotting ten million taxi pickup locations. At first sight it seems
natural to draw ten million points. However, the display only contains a finite
number of pixels. Large numbers of observations will inevitably fall into the
same screen regions.

This leads to a number of problems:

- rendering becomes expensive,
- dense areas become visually saturated,
- regions with low density become difficult to distinguish,
- many individual markers are hidden behind other markers.

The figure may faithfully represent the data, yet reveal surprisingly little
about its structure. Thus, we are led to ask whether it's actually *desirable*
to plot every single point.

## Datashader's approach

[Datashader](https://datashader.org/) answers that question by changing how
visualisation is performed.

Rather than creating one graphical object for every observation, Datashader
divides the plotting area into pixels and computes statistics for each pixel.
The statistic might be:

- the number of observations,
- the mean value of a variable,
- the sum of a variable,
- some other aggregation.

Conceptually, the workflow looks like this: raw observations -> assign
observations to pixels -> aggregate values per pixel -> render image.

Because the number of pixels is fixed by the size of the display, visualisation
can remain practical even when the dataset contains millions or tens of
millions of rows.

An important consequence is that we no longer think primarily in terms of
plotting points. Instead, we think in terms of visualising aggregates.

## Loading and preparing the data

Throughout this lesson we assume that the taxi data have already been loaded
into a Polars DataFrame.

```python
import polars as pl

rides = pl.read_parquet(
    "nyc_yellow_taxi_2025-01.parquet"
)
```

For visualisation we will focus on pickup locations.

```python
rides.select(
    "pickup_longitude",
    "pickup_latitude"
).head()
```

Before plotting geographic data it is often worth performing a small amount of cleaning.

```python
rides_clean = rides.filter(
    pl.col("pickup_longitude").is_not_null()
    & pl.col("pickup_latitude").is_not_null()
    & pl.col("pickup_longitude").is_between(-75, -72)
    & pl.col("pickup_latitude").is_between(40, 42)
)
```

The exact filtering criteria are not particularly important. The goal is simply
to remove clearly invalid coordinates.

### Exercise: Inspect the dataset

::::{exercise}
Determine:

1. The number of rows before filtering.
2. The number of rows after filtering.
3. The percentage of rows removed.

How large is the dataset that you are working with?
::::

::::{solution}

```python
before = rides.height
after = rides_clean.height

removed = 100 * (before - after) / before

print(f"Rows before filtering: {before}")
print(f"Rows after filtering : {after}")
print(f"Removed: {removed:.2f}%")
```

::::

## First attempt: a scatter plot

Before introducing Datashader, let us try the obvious solution.

```python
import hvplot.polars

rides_clean.hvplot.scatter(
    x="pickup_longitude",
    y="pickup_latitude",
    alpha=0.1,
    width=700,
    height=500
)
```

Depending on the dataset size, this may still work reasonably well.

However, zoom out and consider what information the figure provides. Dense
areas become dark blobs. Individual points are no longer meaningful. The
overall structure of the city is difficult to see.

This is a useful teaching moment because it demonstrates that the challenge is
not only computational. Even if plotting were instantaneous, the visual
representation is not necessarily the most informative.

### A note on Polars support

If you are using hvPlot, you can often work directly with Polars DataFrames as
in the example above. For many workflows this is the most convenient approach.

Internally, hvPlot currently performs conversions when working with Polars
data, since Polars is not yet a native HoloViews data interface. Most users do
not need to worry about these details, but it explains why examples in
documentation and tutorials sometimes convert data explicitly to Pandas before
visualisation.

For this lesson we will mostly use the direct Polars interface when possible
and discuss lower-level details only when they help explain how the system
works.

::::{danger}

Polars has a (experimental) `DataFrame.plot` interface which used to be a
wrapper for `hvplot`. In recent versions (>=1.6.0), the default plotting
package is instead [Altair](https://altair-viz.github.io/). To restore the old
`hvplot`-based behaviour, the user has to explicitly import `hvplot.polars` and
call `DataFrame.hvplot.scatter()`.

::::

:::{discussion}

- What information is visible in the scatter plot?

- What information is hidden?

- Would subsampling the dataset solve all of these problems?

:::

## Static visualisation with Datashader

Datashader provides a different visualisation model based on aggregation.

We begin by creating a canvas that defines the output image resolution.

```python
import datashader as ds

canvas = ds.Canvas(
    plot_width=800,
    plot_height=600
)
```

We then aggregate pickup locations onto that canvas.

```python
import pandas as pd

coords = (
    rides_clean
    .select(
        "pickup_longitude",
        "pickup_latitude"
    )
    .to_pandas()
)

agg = canvas.points(
    coords,
    "pickup_longitude",
    "pickup_latitude"
)
```

Notice that the result is not yet an image. It is an aggregated data structure
containing counts for each pixel. To transform it into a viewable image we
apply a colour mapping:

```python
from datashader import transfer_functions as tf

img = tf.shade(
    agg,
    cmap=["lightblue", "darkblue"]
)

img
```

At this point many structures that were invisible in the scatter plot begin to
emerge naturally.

Dense concentrations of pickups reveal activity centres, while transportation
corridors often become visible even though no road network data have been
supplied.

### Exercise: Comparing resolutions

::::{exercise}
Create Datashader visualisations at two different resolutions:

- 400 × 300 pixels
- 1600 × 1200 pixels

Compare the resulting images.

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
    coords,
    "pickup_longitude",
    "pickup_latitude"
)

agg_large = large.points(
    coords,
    "pickup_longitude",
    "pickup_latitude"
)
```

::::

### Beyond simple counts

So far each pixel represents the number of rides associated with that location, so our first example is equivalent to the following:

```python
canvas.points(
    df,
    "pickup_longitude",
    "pickup_latitude",
    agg=ds.count()
)
```

Datashader can aggregate other quantities as well. For example, suppose we want
to examine average trip distance spatially.

```python
agg_distance = canvas.points(
    rides_clean.to_pandas(),
    "pickup_longitude",
    "pickup_latitude",
    agg=ds.mean("trip_distance")
)
```

Rendering this aggregation may reveal different spatial patterns than a density map.

### Other options and settings

#### Colouring

As we saw earlier, the mapping between a colour scale and the aggregated values is performed using the `tf.shade()` function. A colourmap can be prescribed with the `cmap` keyword argument; different colourmaps can help emphasise different structures. Common choices for density plots include `fire`, `viridis` and `inferno`, e.g.:

```python
tf.shade(agg, cmap='fire')
```

#### Linear and logarithmic scaling

Taxi pickups are not distributed uniformly across New York. For example,
Manhattan may have hundreds of times more pickups than other residential
neighbourhoods. Thus, with linear scaling (which is the default), the denser
areas dominate the image. If we use logarithmic scaling instead, we can inspect
in more detail the low-density regions:

```python
tf.shade(agg, how='log')
```

This is usually a useful first step to have a better grasp on a overly
concentrated density plot.

#### Histogram equalisation

Another useful option is [histogram
equalisation](https://en.wikipedia.org/wiki/Histogram_equalization), with which
colours are redistributed so as to have a more equalised distribution across
the image:

```python
tf.shade(agg, how='eq_hist')
```

#### Spreading

Sometimes, when zooming, sparse structures can be difficult to see. This is due
to the very nature of datashader plotting: each pixel represents aggregate
values, so sparse points are reduced to single pixels. To counteract this
phenomenon, datashader provides functions as `spread()` or `dynspread()` which
make help *spreading* smaller datapoints.

::::{exercise}

Create different types of visualisations using the techniques we introduced:
linear, logarithmic, histogram equalisation and (dynamic) spreading). Try to
find the most optimal (for your case) visualisation.

:::{solution}

```python
from datashader import transfer_functions as tf

linear = tf.shade(
    agg,
    cmap="fire",
    how="linear",
)

log = tf.shade(
    agg,
    cmap="fire",
    how="log",
)

eq_hist = tf.shade(
    agg,
    cmap="fire",
    how="eq_hist",
)

eq_hist_dyn = tf.dynspread(
    tf.shade(
        agg,
        cmap="fire",
        how="eq_hist",
    )
)

linear
```

```python
log
```

```python
eq_hist
```

```python
eq_hist_dyn
```

:::

::::

#### Further options

Datashader supports many other options, such as transparency control and
category-based aggregation. Further information can be found in the
[documentation](https://datashader.org/user_guide/index.html).

## Interactive visualisation with HoloViews

Static figures are useful, but exploratory analysis often involves repeatedly zooming, panning, and filtering.

The HoloViz ecosystem combines particularly well with Datashader because aggregation can be performed dynamically as the user explores the data.

Start by enabling the HoloViews backend.

```python
import holoviews as hv

hv.extension("bokeh")
```

Next construct a set of points.

```python
points = hv.Points(
    coords,
    kdims=[
        "pickup_longitude",
        "pickup_latitude"
    ]
)
```

For small datasets we could display these points directly:

```python
points
```

For large datasets it is more effective to datashade them.

```python
from holoviews.operation.datashader import datashade

datashade(points)
```

Try zooming into different parts of the figure.

One of the key ideas behind this workflow is that the visualisation is recomputed when the viewport changes. Rather than plotting all observations at all scales, the visualisation is adapted to the current view.

---

### Looking at the city through data

The pickup locations alone already contain a surprising amount of information.

As you zoom through the city, try to identify features that become visible through taxi activity patterns.

::::{exercise}
Working in pairs, identify:

- JFK Airport,
- LaGuardia Airport,
- Central Park,
- at least one major bridge crossing,
- at least one tunnel crossing.

Discuss what visual clues helped you identify each feature.
::::

Many participants find this exercise surprisingly engaging because the city effectively emerges from the data.

---

## Adding geographic context

Longitude and latitude become much easier to interpret when shown together with a background map.

One of the most striking visualisations in the HoloViz ecosystem combines Datashader aggregation with web map tiles.

```python
import hvplot.polars

rides_clean.hvplot.points(
    x="pickup_longitude",
    y="pickup_latitude",
    rasterize=True,
    tiles=True,
    cmap="fire",
    alpha=0.7,
    frame_width=800,
    frame_height=600,
)
```

The visualisation now combines two pieces of information:

- geographic context from the map,
- taxi activity from the datashaded aggregation.

At city scale we immediately see where activity is concentrated. As we zoom further in, neighbourhood-level and street-level structures begin to appear.

This is often the first point in the lesson where participants experience the full benefit of the Datashader approach. Millions of observations can be explored interactively without first reducing the dataset to a small sample.

---

### Exercise: Day versus night

Taxi activity changes dramatically throughout the day.

Create an additional column containing the pickup hour.

```python
rides = rides.with_columns(
    pl.col("pickup_datetime").dt.hour()
    .alias("hour")
)
```

::::{exercise}
Create separate visualisations for daytime and nighttime rides.

Questions to investigate:

- Which areas are busy throughout the day?
- Which areas become more important at night?
- Do airports appear differently during different periods of the day?

Discuss your observations with a neighbour before comparing them with the rest of the group.
::::

::::{solution}

```python
day = rides.filter(
    pl.col("hour").is_between(6, 18)
)

night = rides.filter(
    ~pl.col("hour").is_between(6, 18)
)
```

Create separate datashaded visualisations and compare them side by side.
::::

---

## Mini-project: Explore your own question

At this stage you have all of the building blocks needed to perform exploratory visualisation on a large dataset.

::::{exercise}
Work in groups of two or three.

Formulate a question that can be investigated using the taxi dataset.

Examples include:

- How do pickup locations vary with time of day?
- Are different payment types associated with different parts of the city?
- Which locations generate the longest average trips?
- How does weekend activity differ from weekday activity?

Create one visualisation that helps answer your question.
::::

Be prepared to present both:

1. your question,
2. the visualisation you created,
3. one interesting observation.

---

## Summary

A common reaction when large visualisations become slow is to look for faster hardware or more efficient plotting libraries. Datashader takes a different approach. Instead of attempting to draw every observation, it focuses on visualising meaningful aggregates.

For geographic datasets such as NYC Taxi trips, this often leads not only to better performance but also to more informative figures. Patterns that are difficult to see in traditional scatter plots become immediately visible once observations are aggregated into a density map.

Combined with HoloViews and hvPlot, Datashader makes it possible to explore datasets containing millions of records interactively, without reducing them to a tiny sample.

:::{keypoints}

- Large datasets often require different visualisation strategies rather than simply faster plotting.
- Overplotting is both a performance problem and a visualisation problem.
- Datashader aggregates observations into pixels before rendering.
- HoloViews and hvPlot provide an interactive interface on top of Datashader.
- Geographic map tiles provide useful context when exploring spatial data.
- Interactive aggregation allows exploration of datasets that would otherwise be difficult to visualise directly.
:::

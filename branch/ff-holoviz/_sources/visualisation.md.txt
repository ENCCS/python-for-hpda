# Visualisations and dashboards

:::{questions}

- What syntax is used to make a lesson?
- How do you structure a lesson effectively for teaching?
- `questions` are at the top of a lesson and provide a starting
  point for what you might learn.  It is usually a bulleted list.
:::

:::{objectives}

- Show a complete lesson page with all of the most common
  structures.
- ...
:::

## The plotting & visualisation ecosystem

## Holoviz for (geographic) data plotting

Let us now use the NYC yellow taxi dataset extended with pickup and dropoff
locations as we built it in the previous episode. It is also available in this
repo for convenience. We can use the [HoloViz](https://holoviz.org/) ecosystem
to plot large scale data. Holoviz comprises several libraries:

| Library | What it does | Why it matters for the taxi data |
|---------|--------------|----------------------------------|
| **Holoviews** | Declarative, high‑level plotting API. You describe *what* you want to see (e.g., points, histograms) and Holoviews chooses the best renderer. | Lets us write concise code (`hv.Points`, `hv.Histogram`) without worrying about Matplotlib/Bokeh boilerplate. |
| **Datashader** | Rasterises millions (or billions) of points on the CPU/GPU before sending a bitmap to the browser. Keeps interactions smooth. | The NYC taxi dataset contains tens of millions of rows; Datashader prevents the browser from freezing. |
| **Panel** | Turns any Bokeh/Matplotlib/Plotly/Holoviews object into a full‑featured web dashboard with layout, widgets, and live updates. | Gives us a single page where the map and the time‑series chart sit side‑by‑side, ready to be embedded in Sphinx‑Lesson. |
| **Bokeh** (backend) | Interactive JavaScript library that renders the final graphics in the browser. | Provides hover tooltips, pan/zoom, and responsive UI. |

Together they let us **load**, **process**, **visualise**, and **share** a massive dataset with only a few lines of code.

After loading the dataset using Polars:

```python

import polars as pl
df = pl.read_parquet("yellow_tripdata_2025-01.parquet")
```

we can load the `holoviews` package and plot a sample of 1000 of the whole data:

```python
import numpy as np
import hvplot.polars # noqa
import holoviews as hv
from holoviews import opts
from holoviews.streams import PlotSize


plot_width  = int(750)
plot_height = int(plot_width//1.2)
x_range, y_range =(-8242000,-8210000), (4965000,4990000)
PlotSize.scale=2.0

opts.defaults(
    opts.Points(width=plot_width, height=plot_height, size=5, color='blue'),
    opts.Overlay(width=plot_width, height=plot_height, xaxis=None, yaxis=None),
    opts.RGB(width=plot_width, height=plot_height),
    opts.Histogram(responsive=True, min_height=250))

samples = df.sample(frac=1e-4)
samples.hvplot.points('dropoff_x', 'dropoff_y', tiles='EsriStreet')

```

Basic hvplots can easily handle several hundred thousand datapoints, even
though it is easy to run into overplotting:

```python
df.sample(frac=1e-2).hvplot.points('dropoff_x', 'dropoff_y', tiles='EsriStreet', alpha=0.1, size=1)
```

In this example, we made the points a bit more transparent to combat this phenomenon.

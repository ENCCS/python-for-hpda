# Tabular data (aka Dataframes)

:::{questions}

- What are series and dataframes?
- What do we mean by tidy and untidy data?
- What packages are available in Python to handle dataframes?

:::

:::{objectives}

- Learn how to manipulate dataframes in Pandas
- Lazy and eager dataframes in Polars
- Learn how to abstract the underlying libraries away with Narwhals

:::

:::{highlight} python
:::

This episode will give an introduction to the concepts of *Series* and *DataFrame*
and how they can be manipulated using different Python packages.

## Series and dataframes

A collection of observations (e.g. a time series or simply a set of observations
of a feature of a phenomenon) can be represented by a homogeneous vector, i.e.
an array where all the elements are of the same type. This is known as a *Series*
in many frameworks. Several series (of different types) can be used as columns of
a tabular structure called a *Dataframe*, as depicted in the figure below.

![Structure of dataframe](01_table_dataframe.svg)

### Tidy vs untidy dataframes

Let us look at the following two dataframes:

::::{tabs}
:::{group-tab} Untidy format

```{csv-table}
:delim: ;
# ; Runner ; 400 ; 800 ; 1200 ; 1500
0 ; Runner 1 ; 64 ; 128 ; 192 ; 240
1 ; Runner 2 ; 80 ; 160 ; 240 ; 300
2 ; Runner 3 ; 96 ; 192 ; 288 ; 360
```

:::

:::{group-tab} Tidy format

```{csv-table}
:delim: ,

#, Runner, distance, time
0, Runner 1, 400, 64
1, Runner 2, 400, 80
2, Runner 3, 400, 96
3, Runner 1, 800, 128
4, Runner 2, 800, 160
5, Runner 3, 800, 192
6, Runner 1, 1200, 192
7, Runner 2, 1200, 240 
8, Runner 3, 1200, 288 
9, Runner 1, 1500, 240 
10, Runner 2, 1500, 300 
11, Runner 3, 1500, 360
```

:::

::::

Most tabular data is either in a tidy format or a untidy format (some people
refer them as the long format or the wide format). The main differences are
summarised below:

- In untidy (wide) format, each row represents an observation consisting of
multiple variables and each variable has its own column. This is intuitive and
easy for us to understand and make comparisons across different variables,
calculate statistics, etc.
- In tidy (long) format , i.e. column-oriented format, each row represents only
one variable of the observation, and can be considered “computer readable”.
When it comes to data analysis using Pandas, the tidy format is recommended:
- Each column can be stored as a vector and this not only saves memory but also
allows for vectorized calculations which are much faster.
- It’s easier to filter, group, join and aggregate the data.

## Pandas & Polars

Historically, [Pandas](https://pandas.pydata.org/) has been the go-to package
to handle dataframes in Python. It is based on NumPy (each column is a Numpy
vector) and has been the traditional workhorse for tabular data, with a stable
API and a large ecosystem built around it, including the [Seaborn](https://seaborn.pydata.org/)
statistical plotting framework.
More recently, [Polars](https://docs.pola.rs/) was introduced as a more modern
and faster alternative to handle dataframes. It is written in Rust and supports
out of the box out of core evaluation (i.e. does not need loading the whole
dataset in memory), lazy evaluation of queries and automatically uses multiple
threads. Moreover, experimental GPU support is available through
[cuDF](https://docs.rapids.ai/api/cudf/stable/). In the remainder of this
episode, the [NYC taxi](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
will be used to showcase how datasets can be accessed, summarised and manipulated
in both Pandas and Polars. The dataset can be download in [Parquet](https://parquet.apache.org/)
format from the link above (the file for the month of January was used in this
case). The dataset contains information about taxi trips performed in New York,
such as the ID of the vendor, the total fare, pickup and drop-off time and
location (expressed as an ID), type of payment, whether additional fees were
charged and more.

### Opening a dataset

Assuming the file is called `yellow_tripdata_2025-01.parquet`, the dataset can be
opened as:

::::{tabs}

:::{group-tab} Pandas

```python
import pandas as pd 
df = pd.read_parquet("yellow_tripdata_2025-01.parquet")
```

:::

:::{group-tab} Polars

```python
import polars as pl 
df = pl.read_parquet("yellow_tripdata_2025-01.parquet")
```

:::
::::

### Description and summarisation

We can get a first understanding of the contents of a dataframe by printing
the first few lines, the "schema" (i.e. the number and type of each column)
and summary statistics as follows:

:::::{tabs}

::::{group-tab} Pandas

```python
df.head()
```

:::{exercise} Output
:class: dropdown

```
   VendorID tpep_pickup_datetime tpep_dropoff_datetime  ...  congestion_surcharge  Airport_fee  cbd_congestion_fee
0         1  2025-01-01 00:18:38   2025-01-01 00:26:59  ...                   2.5          0.0                 0.0
1         1  2025-01-01 00:32:40   2025-01-01 00:35:13  ...                   2.5          0.0                 0.0
2         1  2025-01-01 00:44:04   2025-01-01 00:46:01  ...                   2.5          0.0                 0.0
3         2  2025-01-01 00:14:27   2025-01-01 00:20:01  ...                   0.0          0.0                 0.0
4         2  2025-01-01 00:21:34   2025-01-01 00:25:06  ...                   0.0          0.0                 0.0
```

:::

```python
df.info()
```

:::{exercise} Output
:class: dropdown

```
RangeIndex: 3475226 entries, 0 to 3475225
Data columns (total 20 columns):
 #   Column                 Dtype
---  ------                 -----
 0   VendorID               int32
 1   tpep_pickup_datetime   datetime64[us]
 2   tpep_dropoff_datetime  datetime64[us]
 3   passenger_count        float64
 4   trip_distance          float64
 5   RatecodeID             float64
 6   store_and_fwd_flag     object
 7   PULocationID           int32
 8   DOLocationID           int32
 9   payment_type           int64
 10  fare_amount            float64
 11  extra                  float64
 12  mta_tax                float64
 13  tip_amount             float64
 14  tolls_amount           float64
 15  improvement_surcharge  float64
 16  total_amount           float64
 17  congestion_surcharge   float64
 18  Airport_fee            float64
 19  cbd_congestion_fee     float64
dtypes: datetime64[us](2), float64(13), int32(3), int64(1), object(1)
memory usage: 490.5+ MB
```

:::

```python
df.describe()
```

:::{exercise} Output
:class: dropdown

```
           VendorID        tpep_pickup_datetime       tpep_dropoff_datetime  ...  congestion_surcharge   Airport_fee  cbd_congestion_fee
count  3.475226e+06                     3475226                     3475226  ...          2.935077e+06  2.935077e+06        3.475226e+06
mean   1.785428e+00  2025-01-17 11:02:55.910964  2025-01-17 11:17:56.997901  ...          2.225237e+00  1.239111e-01        4.834093e-01
min    1.000000e+00         2024-12-31 20:47:55         2024-12-18 07:52:40  ...         -2.500000e+00 -1.750000e+00       -7.500000e-01
25%    2.000000e+00         2025-01-10 07:59:01  2025-01-10 08:15:29.500000  ...          2.500000e+00  0.000000e+00        0.000000e+00
50%    2.000000e+00         2025-01-17 15:41:33         2025-01-17 15:59:34  ...          2.500000e+00  0.000000e+00        7.500000e-01
75%    2.000000e+00         2025-01-24 19:34:06         2025-01-24 19:48:31  ...          2.500000e+00  0.000000e+00        7.500000e-01
max    7.000000e+00         2025-02-01 00:00:44         2025-02-01 23:44:11  ...          2.500000e+00  6.750000e+00        7.500000e-01
std    4.263282e-01                         NaN                         NaN  ...          9.039932e-01  4.725090e-01        3.619307e-01

[8 rows x 19 columns]
```

:::
::::

::::{group-tab} Polars

```python
df.head()
```

:::{exercise} Output
:class: dropdown

```python
shape: (5, 20)
┌──────────┬──────────────────┬──────────────────┬─────────────────┬───┬──────────────┬──────────────────┬─────────────┬─────────────────┐
│ VendorID ┆ tpep_pickup_date ┆ tpep_dropoff_dat ┆ passenger_count ┆ … ┆ total_amount ┆ congestion_surch ┆ Airport_fee ┆ cbd_congestion_ │
│ ---      ┆ time             ┆ etime            ┆ ---             ┆   ┆ ---          ┆ arge             ┆ ---         ┆ fee             │
│ i32      ┆ ---              ┆ ---              ┆ i64             ┆   ┆ f64          ┆ ---              ┆ f64         ┆ ---             │
│          ┆ datetime[μs]     ┆ datetime[μs]     ┆                 ┆   ┆              ┆ f64              ┆             ┆ f64             │
╞══════════╪══════════════════╪══════════════════╪═════════════════╪═══╪══════════════╪══════════════════╪═════════════╪═════════════════╡
│ 1        ┆ 2025-01-01       ┆ 2025-01-01       ┆ 1               ┆ … ┆ 18.0         ┆ 2.5              ┆ 0.0         ┆ 0.0             │
│          ┆ 00:18:38         ┆ 00:26:59         ┆                 ┆   ┆              ┆                  ┆             ┆                 │
│ 1        ┆ 2025-01-01       ┆ 2025-01-01       ┆ 1               ┆ … ┆ 12.12        ┆ 2.5              ┆ 0.0         ┆ 0.0             │
│          ┆ 00:32:40         ┆ 00:35:13         ┆                 ┆   ┆              ┆                  ┆             ┆                 │
│ 1        ┆ 2025-01-01       ┆ 2025-01-01       ┆ 1               ┆ … ┆ 12.1         ┆ 2.5              ┆ 0.0         ┆ 0.0             │
│          ┆ 00:44:04         ┆ 00:46:01         ┆                 ┆   ┆              ┆                  ┆             ┆                 │
│ 2        ┆ 2025-01-01       ┆ 2025-01-01       ┆ 3               ┆ … ┆ 9.7          ┆ 0.0              ┆ 0.0         ┆ 0.0             │
│          ┆ 00:14:27         ┆ 00:20:01         ┆                 ┆   ┆              ┆                  ┆             ┆                 │
│ 2        ┆ 2025-01-01       ┆ 2025-01-01       ┆ 3               ┆ … ┆ 8.3          ┆ 0.0              ┆ 0.0         ┆ 0.0             │
│          ┆ 00:21:34         ┆ 00:25:06         ┆                 ┆   ┆              ┆                  ┆             ┆                 │
└──────────┴──────────────────┴──────────────────┴─────────────────┴───┴──────────────┴──────────────────┴─────────────┴─────────────────┘
```

:::

```python
df.describe()
```

:::{exercise} Output
:class: dropdown

```
shape: (9, 21)
┌────────────┬────────────┬──────────────────┬─────────────────┬───┬──────────────┬─────────────────┬─────────────┬─────────────────┐
│ statistic  ┆ VendorID   ┆ tpep_pickup_date ┆ tpep_dropoff_da ┆ … ┆ total_amount ┆ congestion_surc ┆ Airport_fee ┆ cbd_congestion_ │
│ ---        ┆ ---        ┆ time             ┆ tetime          ┆   ┆ ---          ┆ harge           ┆ ---         ┆ fee             │
│ str        ┆ f64        ┆ ---              ┆ ---             ┆   ┆ f64          ┆ ---             ┆ f64         ┆ ---             │
│            ┆            ┆ str              ┆ str             ┆   ┆              ┆ f64             ┆             ┆ f64             │
╞════════════╪════════════╪══════════════════╪═════════════════╪═══╪══════════════╪═════════════════╪═════════════╪═════════════════╡
│ count      ┆ 3.475226e6 ┆ 3475226          ┆ 3475226         ┆ … ┆ 3.475226e6   ┆ 2.935077e6      ┆ 2.935077e6  ┆ 3.475226e6      │
│ null_count ┆ 0.0        ┆ 0                ┆ 0               ┆ … ┆ 0.0          ┆ 540149.0        ┆ 540149.0    ┆ 0.0             │
│ mean       ┆ 1.785428   ┆ 2025-01-17       ┆ 2025-01-17      ┆ … ┆ 25.611292    ┆ 2.225237        ┆ 0.123911    ┆ 0.483409        │
│            ┆            ┆ 11:02:55.910964  ┆ 11:17:56.997901 ┆   ┆              ┆                 ┆             ┆                 │
│ std        ┆ 0.426328   ┆ null             ┆ null            ┆ … ┆ 463.658478   ┆ 0.903993        ┆ 0.472509    ┆ 0.361931        │
│ min        ┆ 1.0        ┆ 2024-12-31       ┆ 2024-12-18      ┆ … ┆ -901.0       ┆ -2.5            ┆ -1.75       ┆ -0.75           │
│            ┆            ┆ 20:47:55         ┆ 07:52:40        ┆   ┆              ┆                 ┆             ┆                 │
│ 25%        ┆ 2.0        ┆ 2025-01-10       ┆ 2025-01-10      ┆ … ┆ 15.2         ┆ 2.5             ┆ 0.0         ┆ 0.0             │
│            ┆            ┆ 07:59:01         ┆ 08:15:29        ┆   ┆              ┆                 ┆             ┆                 │
│ 50%        ┆ 2.0        ┆ 2025-01-17       ┆ 2025-01-17      ┆ … ┆ 19.95        ┆ 2.5             ┆ 0.0         ┆ 0.75            │
│            ┆            ┆ 15:41:34         ┆ 15:59:34        ┆   ┆              ┆                 ┆             ┆                 │
│ 75%        ┆ 2.0        ┆ 2025-01-24       ┆ 2025-01-24      ┆ … ┆ 27.78        ┆ 2.5             ┆ 0.0         ┆ 0.75            │
│            ┆            ┆ 19:34:06         ┆ 19:48:31        ┆   ┆              ┆                 ┆             ┆                 │
│ max        ┆ 7.0        ┆ 2025-02-01       ┆ 2025-02-01      ┆ … ┆ 863380.37    ┆ 2.5             ┆ 6.75        ┆ 0.75            │
│            ┆            ┆ 00:00:44         ┆ 23:44:11        ┆   ┆              ┆                 ┆             ┆                 │
└────────────┴────────────┴──────────────────┴─────────────────┴───┴──────────────┴─────────────────┴─────────────┴─────────────────┘
```

:::

::::

:::::

### Indexing

We can index data in the dataframe as follows:

:::::{tabs}

::::{group-tab} Pandas

```python
# With this we can select a column
df['VendorID'] # Could also be df.VendorID 
```

:::{exercise} Output
:class: dropdown

```
0          1
1          1
2          1
3          2
4          2
          ..
3475221    2
3475222    2
3475223    2
3475224    2
3475225    2
```

:::

```python
# Get a row 
df.iloc[1000,:]
```

:::{exercise} Output
:class: dropdown

```
VendorID                                   2
tpep_pickup_datetime     2025-01-01 00:08:06
tpep_dropoff_datetime    2025-01-01 00:16:20
passenger_count                          4.0
trip_distance                           1.53
RatecodeID                               1.0
store_and_fwd_flag                         N
PULocationID                             114
DOLocationID                              90
payment_type                               1
fare_amount                             10.0
extra                                    1.0
mta_tax                                  0.5
tip_amount                              2.25
tolls_amount                             0.0
improvement_surcharge                    1.0
total_amount                           17.25
congestion_surcharge                     2.5
Airport_fee                              0.0
cbd_congestion_fee                       0.0
Name: 1000, dtype: object
>>> df.iloc[1000,:]
VendorID                                   2
tpep_pickup_datetime     2025-01-01 00:08:06
tpep_dropoff_datetime    2025-01-01 00:16:20
passenger_count                          4.0
trip_distance                           1.53
RatecodeID                               1.0
store_and_fwd_flag                         N
PULocationID                             114
DOLocationID                              90
payment_type                               1
fare_amount                             10.0
extra                                    1.0
mta_tax                                  0.5
tip_amount                              2.25
tolls_amount                             0.0
improvement_surcharge                    1.0
total_amount                           17.25
congestion_surcharge                     2.5
Airport_fee                              0.0
cbd_congestion_fee                       0.0
```

:::

::::

::::{group-tab} Polars

```python
df["VendorID"] # A more Polars-y idiom is to use df.select(["VendorID"])
```

:::{exercise} Output

:class: dropdown

```
shape: (3_475_226,)
Series: 'VendorID' [i32]
[
 1
 1
 1
 2
 2
 …
 2
 2
 2
 2
 2
]
```

:::

```python
df[1000][:]
```

:::{exercise} Output
:class: dropdown

```
┌──────────┬─────────────┬─────────────┬────────────┬───┬────────────┬────────────┬────────────┬────────────┐
│ VendorID ┆ tpep_pickup ┆ tpep_dropof ┆ passenger_ ┆ … ┆ total_amou ┆ congestion ┆ Airport_fe ┆ cbd_conges │
│ ---      ┆ _datetime   ┆ f_datetime  ┆ count      ┆   ┆ nt         ┆ _surcharge ┆ e          ┆ tion_fee   │
│ i32      ┆ ---         ┆ ---         ┆ ---        ┆   ┆ ---        ┆ ---        ┆ ---        ┆ ---        │
│          ┆ datetime[μs ┆ datetime[μs ┆ i64        ┆   ┆ f64        ┆ f64        ┆ f64        ┆ f64        │
│          ┆ ]           ┆ ]           ┆            ┆   ┆            ┆            ┆            ┆            │
╞══════════╪═════════════╪═════════════╪════════════╪═══╪════════════╪════════════╪════════════╪════════════╡
│ 2        ┆ 2025-01-01  ┆ 2025-01-01  ┆ 4          ┆ … ┆ 17.25      ┆ 2.5        ┆ 0.0        ┆ 0.0        │
│          ┆ 00:08:06    ┆ 00:16:20    ┆            ┆   ┆            ┆            ┆            ┆            │
└──────────┴─────────────┴─────────────┴────────────┴───┴────────────┴────────────┴────────────┴────────────┘
```

:::

::::
:::::

In both cases, a similar syntax can be used to do in-place modification (e.g. `df[row][column]=...`).
Please note that this kind of replacement carries a big performance penalty,
which is designed to do column-wide operations with minimal overhead. This is
commonly achieved through the [expression API](https://docs.pola.rs/user-guide/concepts/expressions-and-contexts/), as detailed in the next section.

### Common workflows

It is quite common to compute stratified statistics of different groups to
produce descriptive statistics. This is commonly achieved through a `group-by`
workflow, where the following happens:

- Splitting: data is partitioned into different groups based on some criterion
- Applying: applying a function/performing a calculation to each group
- Combining: assembling a dataframe (of potentially any size) with the results.

This is type of workflow is represented below.
![split-apply-combine](groupby.png)

As an example, let us try to to compute the total fare for each hour, split
by payment type.

:::::{tabs}

::::{group-tab} Pandas

```python
#First let us extract the hour from the tpep_pickup_datetime column 
df["hour"] = df['tpep_pickup_datetime'].dt.hour 

hourly_fare = (
  df.groupby(['hour', 'payment_type'], observed=False)['fare_amount']
  .sum()
  .reset_index()
  .sort_values(['hour', 'payment_type'])
)
```

:::{exercise} Output
:class: dropdown

```
     hour  payment_type  fare_amount
0       0             0    352227.86
1       0             1   1088201.12
2       0             2    156546.07
3       0             3      3537.91
4       0             4      3941.24
..    ...           ...          ...
116    23             0    534063.76
117    23             1   1618143.37
118    23             2    219991.22
119    23             3      4765.54
120    23             4      3293.61
```

:::

The `groupby` statement is used to stratify the `fare_amount` column by
hour and payment type. Then the amounts per hour and type get summed and
sorted according to time and payment type.

::::

::::{group-tab} Polars

```python
#First, let us extract the hour from the tpep_pickup_datetime column 
df = df.with_columns(pl.col('tpep_pickup_datetime').dt.hour().alias('hour'))

hourly_fare = (
    df.group_by(['hour', 'payment_type'])
    .agg(pl.sum('fare_amount').alias('total_fare'))
    .sort(['hour', 'payment_type'])
)
```

:::{exercise} Output
:class: dropdown

```
┌──────┬──────────────┬────────────┐
│ hour ┆ payment_type ┆ total_fare │
│ ---  ┆ ---          ┆ ---        │
│ i8   ┆ i64          ┆ f64        │
╞══════╪══════════════╪════════════╡
│ 0    ┆ 0            ┆ 352227.86  │
│ 0    ┆ 1            ┆ 1.0882e6   │
│ 0    ┆ 2            ┆ 156546.07  │
│ 0    ┆ 3            ┆ 3537.91    │
│ 0    ┆ 4            ┆ 3941.24    │
│ …    ┆ …            ┆ …          │
│ 23   ┆ 0            ┆ 534063.76  │
│ 23   ┆ 1            ┆ 1.6181e6   │
│ 23   ┆ 2            ┆ 219991.22  │
│ 23   ┆ 3            ┆ 4765.54    │
│ 23   ┆ 4            ┆ 3293.61    │
└──────┴──────────────┴────────────┘
```

:::

The `group_by` statement is used to stratify by hour and payment type,
followed by an aggregated sum of the `fare_amount` column and a sort.
Notice how the syntax has more of a functional programming flavour to it
(`pl.col`, `pl.sum` as pure functions). This will be clarified further in
the next section. Also note that Polars by default spreads the workload
over multiple threads.

::::

:::::

## Idiomatic Polars

Polars introduces a few variations to dataset operations compared to the
traditional Pandas approach. In particular, a domain-specific language
(DSL) was developed, where *expressions* are written to represent dataset
operations and *context*s provide the environment where they produce a
result.

### Expressions

Let's say that we created a `trip_duration_sec` column in our NYC cab database
and, given the `trip_distance` column, we want to compute the average speed.
In Polars, this can be achieved with:

```python
pl.col('trip_distance') / pl.col(`trip_duration_sec`)
```

This is a lazy representation of an operation we want to perform, which can
be further manipulated or just printed. For it to actually produce data, a
*context* is needed.

### Contexts

The same Polars expression can produce different results depending on the
context where it is used. Four common contexts include:

- `select`
- `with_columns`
- `filter`
- `group_by`

Both `select` and `with_columns` can produce new columns, which may be
aggregations, combinations of other columns, or literals. The difference
between the two is that `select` only includes the columns contained in its
input expression, whereas `with_columns` returns a new dataframe which
contains all the columns from the original dataframe and the new ones created
by the expression. To exemplify, using our earlier example of computing the
average speed during a trip, using `select` would yield a single column,
whereas `with_columns` would return the original dataframe with an additional
column called `trip distance`:

```python
df.select(pl.col('trip_distance')/pl.col('trip_duration_sec')*3600)
shape: (3_475_226, 1)
┌───────────────┐
│ trip_distance │
│ ---           │
│ f64           │
╞═══════════════╡
│ 11.497006     │
│ 11.764706     │
│ 18.461538     │
│ 5.60479       │
│ 11.207547     │
│ …             │
│ 13.68899      │
│ 19.42398      │
│ 9.879418      │
│ 9.339901      │
│ 12.781395     │
└───────────────┘
```

```python
df.with_columns((pl.col('trip_distance')/pl.col('trip_duration_sec')*3600).alias("avg_sp
eed_mph"))
shape: (3_475_226, 22)
┌──────────┬──────────┬──────────┬──────────┬───┬──────────┬──────────┬──────────┬──────────┐
│ VendorID ┆ tpep_pic ┆ tpep_dro ┆ passenge ┆ … ┆ Airport_ ┆ cbd_cong ┆ trip_dur ┆ avg_spee │
│ ---      ┆ kup_date ┆ poff_dat ┆ r_count  ┆   ┆ fee      ┆ estion_f ┆ ation_se ┆ d_mph    │
│ i32      ┆ time     ┆ etime    ┆ ---      ┆   ┆ ---      ┆ ee       ┆ c        ┆ ---      │
│          ┆ ---      ┆ ---      ┆ i64      ┆   ┆ f64      ┆ ---      ┆ ---      ┆ f64      │
│          ┆ datetime ┆ datetime ┆          ┆   ┆          ┆ f64      ┆ i64      ┆          │
│          ┆ [μs]     ┆ [μs]     ┆          ┆   ┆          ┆          ┆          ┆          │
╞══════════╪══════════╪══════════╪══════════╪═══╪══════════╪══════════╪══════════╪══════════╡
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ 1        ┆ … ┆ 0.0      ┆ 0.0      ┆ 501      ┆ 11.49700 │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆ 6        │
│          ┆ 00:18:38 ┆ 00:26:59 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ 1        ┆ … ┆ 0.0      ┆ 0.0      ┆ 153      ┆ 11.76470 │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆ 6        │
│          ┆ 00:32:40 ┆ 00:35:13 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ 1        ┆ … ┆ 0.0      ┆ 0.0      ┆ 117      ┆ 18.46153 │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆ 8        │
│          ┆ 00:44:04 ┆ 00:46:01 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ 3        ┆ … ┆ 0.0      ┆ 0.0      ┆ 334      ┆ 5.60479  │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:14:27 ┆ 00:20:01 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ 3        ┆ … ┆ 0.0      ┆ 0.0      ┆ 212      ┆ 11.20754 │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆ 7        │
│          ┆ 00:21:34 ┆ 00:25:06 ┆          ┆   ┆          ┆          ┆          ┆          │
│ …        ┆ …        ┆ …        ┆ …        ┆ … ┆ …        ┆ …        ┆ …        ┆ …        │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 881      ┆ 13.68899 │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:01:48 ┆ 23:16:29 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-02- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 1618     ┆ 19.42398 │
│          ┆ 31       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:50:29 ┆ 00:17:27 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 962      ┆ 9.879418 │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:26:59 ┆ 23:43:01 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 1218     ┆ 9.339901 │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:14:34 ┆ 23:34:52 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-02- ┆ null     ┆ … ┆ null     ┆ 0.0      ┆ 645      ┆ 12.78139 │
│          ┆ 31       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆ 5        │
│          ┆ 23:56:42 ┆ 00:07:27 ┆          ┆   ┆          ┆          ┆          ┆          │
└──────────┴──────────┴──────────┴──────────┴───┴──────────┴──────────┴──────────┴──────────┘
```

The `filter` context filters the rows of a dataframe based on one (or more)
expressions which evaluate to a Boolean, e.g.

```python
df.filter(pl.col('avg_speed_mph') < 1)
shape: (104_410, 22)
┌──────────┬──────────┬──────────┬──────────┬───┬──────────┬──────────┬──────────┬──────────┐
│ VendorID ┆ tpep_pic ┆ tpep_dro ┆ passenge ┆ … ┆ Airport_ ┆ cbd_cong ┆ trip_dur ┆ avg_spee │
│ ---      ┆ kup_date ┆ poff_dat ┆ r_count  ┆   ┆ fee      ┆ estion_f ┆ ation_se ┆ d_mph    │
│ i32      ┆ time     ┆ etime    ┆ ---      ┆   ┆ ---      ┆ ee       ┆ c        ┆ ---      │
│          ┆ ---      ┆ ---      ┆ i64      ┆   ┆ f64      ┆ ---      ┆ ---      ┆ f64      │
│          ┆ datetime ┆ datetime ┆          ┆   ┆          ┆ f64      ┆ i64      ┆          │
│          ┆ [μs]     ┆ [μs]     ┆          ┆   ┆          ┆          ┆          ┆          │
╞══════════╪══════════╪══════════╪══════════╪═══╪══════════╪══════════╪══════════╪══════════╡
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ 1        ┆ … ┆ 0.0      ┆ 0.0      ┆ 10       ┆ 0.0      │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:37:43 ┆ 00:37:53 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ 3        ┆ … ┆ 0.0      ┆ 0.0      ┆ 8        ┆ 0.0      │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:57:08 ┆ 00:57:16 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ 1        ┆ … ┆ 0.0      ┆ 0.0      ┆ 1910     ┆ 0.0      │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:27:40 ┆ 00:59:30 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ 4        ┆ … ┆ 0.0      ┆ 0.0      ┆ 5        ┆ 0.0      │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:56:49 ┆ 00:56:54 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ 0        ┆ … ┆ 0.0      ┆ 0.0      ┆ 2        ┆ 0.0      │
│          ┆ 01       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 00:42:42 ┆ 00:42:44 ┆          ┆   ┆          ┆          ┆          ┆          │
│ …        ┆ …        ┆ …        ┆ …        ┆ … ┆ …        ┆ …        ┆ …        ┆ …        │
│ 1        ┆ 2025-01- ┆ 2025-02- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 266      ┆ 0.0      │
│          ┆ 31       ┆ 01       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:59:17 ┆ 00:03:43 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 1100     ┆ 0.0      │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:17:38 ┆ 23:35:58 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 2        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.0      ┆ 161      ┆ 0.0      │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:39:25 ┆ 23:42:06 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 24       ┆ 0.0      │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:30:42 ┆ 23:31:06 ┆          ┆   ┆          ┆          ┆          ┆          │
│ 1        ┆ 2025-01- ┆ 2025-01- ┆ null     ┆ … ┆ null     ┆ 0.75     ┆ 1556     ┆ 0.0      │
│          ┆ 31       ┆ 31       ┆          ┆   ┆          ┆          ┆          ┆          │
│          ┆ 23:10:25 ┆ 23:36:21 ┆          ┆   ┆          ┆          ┆          ┆          │
└──────────┴──────────┴──────────┴──────────┴───┴──────────┴──────────┴──────────┴──────────┘
```

The `group_by` context behaves like its Pandas counterpart.

### Transformations

A `join` operation combines columns from one or more dataframes into a new
dataframe. There are different joining strategies, which influence how columns
are combined and what rows are included in the final set. A common type is the
*equi* join, where rows are matched by a key expression. Let us clarify this
with an example. The `df` dataframe does not include specific coordinates for
each pickup and drop-off, rather only a `PULocationID` and a `DOLocationID`.
There is a `taxy_zones_xy.csv` file that contains, for each `LocationID`, the
latitude (X) and longitude (Y) of each location, as well as the name of zone
and borough:

```python

lookup_df = pl.read_csv('taxy_zones_xy.csv', has_header=True)
lookup_df.head()
┌────────────┬────────────┬───────────┬─────────────────────────┬───────────────┐
│ LocationID ┆ X          ┆ Y         ┆ zone                    ┆ borough       │
│ ---        ┆ ---        ┆ ---       ┆ ---                     ┆ ---           │
│ i64        ┆ f64        ┆ f64       ┆ str                     ┆ str           │
╞════════════╪════════════╪═══════════╪═════════════════════════╪═══════════════╡
│ 1          ┆ -74.176786 ┆ 40.689516 ┆ Newark Airport          ┆ EWR           │
│ 2          ┆ -73.826126 ┆ 40.625724 ┆ Jamaica Bay             ┆ Queens        │
│ 3          ┆ -73.849479 ┆ 40.865888 ┆ Allerton/Pelham Gardens ┆ Bronx         │
│ 4          ┆ -73.977023 ┆ 40.724152 ┆ Alphabet City           ┆ Manhattan     │
│ 5          ┆ -74.18993  ┆ 40.55034  ┆ Arden Heights           ┆ Staten Island │
└────────────┴────────────┴───────────┴─────────────────────────┴───────────────┘
```

This can be used to append these columns to the original df to have some form
of geographical data as follows (e.g. for the `PULocationID`):

```python
df = df.join(lookup_df, left_on='PULocationID', right_on='LocationID', how='left'
, suffix='_pickup')
```

In the line above, `left_on` is used to indicate the *key* in the original
dataframe, `right_on` is used to specify the *key* in the `lookup_df` dataframe,
`how=left` means that the columns from the second dataframe will be added to
the first (and not the other way around) and `suffix` is what will be added to
the names of the joined columns (i.e., df will contain columns called `X_pickup`,
`Y_pickup`, `zone_pickup` and `borough_pickup`). More information on join
operations can be found [here](https://docs.pola.rs/user-guide/transformations/joins/).

## Exercises

::::{exercise} Joining geographical data
We have already seen how to add actual latitude and longitude for the pickups.
Now do the same for the drop-offs!

:::{solution}

```python
df = df.join(lookup_df, left_on='DOLocationID', right_on='LocationID', how='left'
, suffix='_dropoff')
```

:::

::::

::::{exercise} Feature engineering: enriching the dataset
We want to understand a bit more of the traffic in the city by creating
new features (i.e. columns), in particular:

- Split the pickup datetime into hour, minute, day of the week and month
to indentify daily, weekly and monthly trends
- Compute the average speed as an indicator of congestion (low speed ->
traffic jam)
- Stratify the trip distance and fare by zone to identify how expensive
different zones are.
Below is a skeleton of the code, where some lines have been blanked out
for you to fill (marked with `TODO:...`)

```python
import polars as pl
raw_df = pl.read_parquet('yellow_tripdata_2025-01.parquet')
df = raw_df.with_columns([
    pl.col("tpep_pickup_datetime").dt.hour().alias("pickup_hour"),
    #TODO: do this for the minute
    pl.col("tpep_pickup_datetime").dt.day_of_week().alias("pickup_dow"),   # Mon=0 … Sun=6
    pl.col("tpep_pickup_datetime").dt.month().alias("pickup_month"),
    # Trip duration in seconds
    (pl.col("tpep_dropoff_datetime") - pl.col("tpep_pickup_datetime"))
        .dt.total_seconds()
        .alias("trip_duration_sec"),
])

df = df.with_column(
    #TODO: add expression for average velocity here
    .replace_nan(None)                        # protect against div‑by‑zero
    .alias("avg_speed_mph")
)

# Compute per‑pickup‑zone statistics once
zone_stats = (
    df.groupby("PULocationID")
      .agg([
          pl.mean("fare_amount").alias("zone_avg_fare"),
          #TODO: do the same for the trip distance here
          pl.count().alias("zone_trip_cnt"),
      ])
      .rename({"PULocationID": "pickup_zone_id"})   # avoid name clash later
)

# Join those stats back onto the original rows
df = df.join(zone_stats, left_on="PULocationID", right_on="pickup_zone_id", how="left")
```

While we haven't covered the `join` instruction earlier, its main role
is to "spread" the `zone_stats` over all the rides in the original dataframe
(i.e. write the `zone_avg_fare` on each ride in `df`). `join` has its roots
in relational databases, where different tables can be merged based on a
common column.

:::{solution}

```python
import polars as pl
raw_df = pl.read_parquet('yellow_tripdata_2025-01.parquet')
df = raw_df.with_columns([
    pl.col("tpep_pickup_datetime").dt.hour().alias("pickup_hour"),
    pl.col("tpep_pickup_datetime").dt.minute().alias("pickup_minute"),
    pl.col("tpep_pickup_datetime").dt.day_of_week().alias("pickup_dow"),   # Mon=0 … Sun=6
    pl.col("tpep_pickup_datetime").dt.month().alias("pickup_month"),
    # Trip duration in seconds
    (pl.col("tpep_dropoff_datetime") - pl.col("tpep_pickup_datetime"))
        .dt.seconds()
        .alias("trip_duration_sec"),
])

df = df.with_column(
    (
        pl.col("trip_distance") /
        (pl.col("trip_duration_sec") / 3600)   # seconds → hours
    )
    .replace_nan(None)                        # protect against div‑by‑zero
    .alias("avg_speed_mph")
)

# Compute per‑pickup‑zone statistics once
zone_stats = (
    df.groupby("PULocationID")
      .agg([
          pl.mean("fare_amount").alias("zone_avg_fare"),
          pl.mean("trip_distance").alias("zone_avg_dist"),
          pl.count().alias("zone_trip_cnt"),
      ])
      .rename({"PULocationID": "pickup_zone_id"})   # avoid name clash later
)

# Join those stats back onto the original rows
df = df.join(zone_stats, left_on="PULocationID", right_on="pickup_zone_id", how="left")
```

:::
::::

::::{exercise} More feature engineering!
Similarly to the exercise above, define the following features in the data:

- `pickup_hour` extracted from `tpep_pickup_time`
- `is_weekend`, a Boolean value for each trip
- `avg_speed_mph`, exactly as before
- `tip_to_fare_ratio`, dividing the tip amount by the total fare. Be careful
with division by 0
- `fare_per_mile`, dividing the total fare by the distance
- `dist_per_passenger`, the average distance travelled by each passenger
(sum of all trip distances divided by number of trips)
- `speed_per_pickup_area`, the average velocity stratified by pickup location
- `dropoff_trip_count`, count of trips stratified per dropoff location

:::{solution}

```python
import polars as pl
raw_df = pl.read_parquet("yellow_tripdata_2025-01.parquet")
df = raw_df.with_columns([
    # 1. pickup_hour
    pl.col("tpep_pickup_datetime").dt.hour().alias("pickup_hour"),

    # 2. is_weekend (Sat=5, Sun=6)
    pl.col("tpep_pickup_datetime")
      .dt.day_of_week()
      .is_in([5, 6])
      .alias("is_weekend"),

    # 3. trip_duration_sec
    (pl.col("tpep_dropoff_datetime") - pl.col("tpep_pickup_datetime"))
        .dt.seconds()
        .alias("trip_duration_sec"),

    # 4. avg_speed_mph
    (
        pl.col("trip_distance") /
        (pl.col("trip_duration_sec") / 3600)
    )
    .replace_nan(None)                       # protect against div‑by‑zero
    .alias("avg_speed_mph"),

    # 5. tip_to_fare_ratio
    (pl.col("tip_amount") / pl.col("fare_amount"))
        .replace_inf(None)
        .replace_nan(None)
        .alias("tip_to_fare_ratio"),

    # 6. fare_per_mile
    (pl.col("fare_amount") / pl.col("trip_distance"))
        .replace_inf(None)
        .replace_nan(None)
        .alias("fare_per_mile"),

    # 7. dist_per_passenger
    (pl.col("trip_distance") / pl.col("passenger_count"))
        .replace_inf(None)
        .replace_nan(None)
        .alias("dist_per_passenger"),
])

dropoff_stats = (
    df.groupby("DOLocationID")
      .agg([
          pl.mean("avg_speed_mph").alias("dropoff_avg_speed"),
          pl.count().alias("dropoff_trip_cnt"),
      ])
      .rename({"DOLocationID": "dropoff_zone_id"})   # avoid name clash later
)

# Join the per‑zone stats back onto every row
df = df.join(dropoff_stats, left_on="DOLocationID", right_on="dropoff_zone_id", how="left")

```

:::
::::

## Summary

We have seen how to deal with common workflows in both Pandas and Polars,
starting from basic tasks like opening a dataset and inspecting it to performing
split-apply-combine pipelines. We have seen how to use Polars to manipulate
datasets and perform some basic feature engineering.

:::{keypoints}

- Dataframes are combinations of series
- Both Pandas and Polars can be used to manipulate them
- The expression API in Polars allows to perform advanced operations with a
simple DSL.

:::

## See also

There is a lot more to Polars than what we covered in this short introduction.
For example, queries like the ones we introduced can be performed lazily, i.e.
just declared and then run all together, giving the backend a chance to
optimise them. This can dramatically improve performance in the case of complex
queries. For this and a lot more, we refer you to the official
[documentation](https://docs.pola.rs/).

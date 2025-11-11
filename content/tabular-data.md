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

## Pandas

Pandas is a Python package that provides high-performance and easy to use data structures and data analysis tools. Built on NumPy arrays, Pandas is particularly well suited to analyze tabular and time series data. Although NumPy could in principle deal with structured arrays (arrays with mixed data types), it is not efficient.

The core data structures of Pandas are Series and Dataframes.

- A Pandas series is a one-dimensional NumPy array with an index which we could use to access the data

- A dataframe consist of a table of values with labels for each row and column. A dataframe can combine multiple data types, such as numbers and text, but the data in each column is of the same type.
- Each column of a dataframe is a series object - a dataframe is thus a collection of series.

### Tidy vs untidy dataframes

- Most tabular data is either in a tidy format or a untidy format (some people refer them as the long format or the wide format).
- In untidy (wide) format, each row represents an observation consisting of multiple variables and each variable has its own column. This is intuitive and easy for us to understand and make comparisons across different variables, calculate statistics, etc.
- In tidy (long) format , i.e. column-oriented format, each row represents only one variable of the observation, and can be considered “computer readable”.
When it comes to data analysis using Pandas, the tidy format is recommended:

    Each column can be stored as a vector and this not only saves memory but also allows for vectorized calculations which are much faster.

    It’s easier to filter, group, join and aggregate the data.

The name “tidy data” comes from Wickham’s paper (2014) which describes the ideas in great detail.

:::{discussion}
Discuss the following.

- A discussion section
- Another discussion topic
:::

## Section

```
print("hello world")
# This uses the default highlighting language
```

```python
print("hello world)
```

## Exercises: description

:::{exercise} Exercise Topic-1: imperative description of exercise
Exercise text here.
:::

:::{solution}
Solution text here
:::

## Summary

A Summary of what you learned and why it might be useful.  Maybe a
hint of what comes next.

## See also

- Other relevant links
- Other link

:::{keypoints}

- What the learner should take away
- point 2
- ...

This is another holdover from the carpentries style.  This perhaps
is better done in a "summary" section.
:::

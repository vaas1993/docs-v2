
When [querying data from InfluxDB](/influxdb/version/query-data/get-started/query-influxdb),
you often need to transform that data in some way.
Common examples are aggregating data, downsampling data, etc.

This guide demonstrates using [Flux functions](/flux/v0/stdlib/) to transform your data.
It walks through creating a Flux script that partitions data into windows of time,
averages the `_value`s in each window, and outputs the averages as a new table.

{{% note %}}
If you're not familiar with how Flux structures and operates on data, see
[Flux data model](/flux/v0/get-started/data-model/).
{{% /note %}}

## Query data
Use the query built in the previous [Query data from InfluxDB](/influxdb/version/query-data/get-started/query-influxdb)
guide, but update the range to pull data from the last hour:

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
```

## Flux functions
Flux provides a number of functions that perform specific operations, transformations, and tasks.
You can also [create custom functions](/influxdb/version/query-data/flux/custom-functions) in your Flux queries.
_Functions are covered in detail in the [Flux standard library](/flux/v0/stdlib/) documentation._

A common type of function used when transforming data queried from InfluxDB is an aggregate function.
Aggregate functions take a set of `_value`s in a table, aggregate them, and transform
them into a new value.

This example uses the [`mean()` function](/flux/v0/stdlib/universe/mean)
to average values within each time window.

{{% note %}}
The following example walks through the steps required to window and aggregate data,
but there is a [`aggregateWindow()` helper function](#helper-functions) that does it for you.
It's just good to understand the steps in the process.
{{% /note %}}

## Window your data
Flux's [`window()` function](/flux/v0/stdlib/universe/window) partitions records based on a time value.
Use the `every` parameter to define a duration of each window.

{{% note %}}
#### Calendar months and years
`every` supports all [valid duration units](/flux/v0/spec/types/#duration-types),
including **calendar months (`1mo`)** and **years (`1y`)**.
{{% /note %}}

For this example, window data in five minute intervals (`5m`).

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
    |> window(every: 5m)
```

As data is gathered into windows of time, each window is output as its own table.
When visualized, each table is assigned a unique color.

![Windowed data tables](/img/flux/windowed-data.png)

## Aggregate windowed data
Flux aggregate functions take the `_value`s in each table and aggregate them in some way.
Use the [`mean()` function](/flux/v0/stdlib/universe/mean) to average the `_value`s of each table.

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
    |> window(every: 5m)
    |> mean()
```

As rows in each window are aggregated, their output table contains only a single row with the aggregate value.
Windowed tables are all still separate and, when visualized, will appear as single, unconnected points.

![Windowed aggregate data](/img/flux/windowed-aggregates.png)

## Add times to your aggregates
As values are aggregated, the resulting tables do not have a `_time` column because
the records used for the aggregation all have different timestamps.
Aggregate functions don't infer what time should be used for the aggregate value.
Therefore the `_time` column is dropped.

A `_time` column is required in the [next operation](#unwindow-aggregate-tables).
To add one, use the [`duplicate()` function](/flux/v0/stdlib/universe/duplicate)
to duplicate the `_stop` column as the `_time` column for each windowed table.

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
    |> window(every: 5m)
    |> mean()
    |> duplicate(column: "_stop", as: "_time")
```

## Unwindow aggregate tables

Use the `window()` function with the `every: inf` parameter to gather all points
into a single, infinite window.

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
    |> window(every: 5m)
    |> mean()
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)
```

Once ungrouped and combined into a single table, the aggregate data points will appear connected in your visualization.

![Unwindowed aggregate data](/img/flux/windowed-aggregates-ungrouped.png)

## Helper functions
This may seem like a lot of coding just to build a query that aggregates data, however going through the
process helps to understand how data changes "shape" as it is passed through each function.

Flux provides (and allows you to create) "helper" functions that abstract many of these steps.
The same operation performed in this guide can be accomplished using
[`aggregateWindow()`](/flux/v0/stdlib/universe/aggregatewindow).

```js
from(bucket: "example-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total")
    |> aggregateWindow(every: 5m, fn: mean)
```

## Congratulations!
You have now constructed a Flux query that uses Flux functions to transform your data.
There are many more ways to manipulate your data using both Flux's primitive functions
and your own custom functions, but this is a good introduction into the basic syntax and query structure.

---

_For a deeper dive into windowing and aggregating data with example data output for each transformation,
view the [Window and aggregate data](/influxdb/version/query-data/flux/window-aggregate) guide._

---

{{< page-nav prev="/influxdb/version/query-data/get-started/query-influxdb/" >}}

---
title: InfluxDB line protocol tutorial
description: Tutorial for using InfluxDB line protocol.
aliases:
    - /enterprise_influxdb/v1/write_protocols/line/
menu:
  enterprise_influxdb_v1:
    weight: 20
    parent: Write protocols
---

The InfluxDB line protocol is a text-based format for writing points to the
database.
Points must be in line protocol format for InfluxDB to successfully parse and
write points (unless you're using a [service plugin](/enterprise_influxdb/v1/supported_protocols/)).

Using fictional temperature data, this page introduces InfluxDB line protocol.
It covers:

- [Syntax](#syntax)
- [Data types](#data-types)
- [Quoting](#quoting)
- [Special characters and keywords](#special-characters-and-keywords)
- [Writing data to InfluxDB](#writing-data-to-influxdb)

The final section, [Writing data to InfluxDB](#writing-data-to-influxdb),
describes how to get data into InfluxDB and how InfluxDB handles Line
Protocol duplicates.

## Syntax

A single line of text in line protocol format represents one data point in InfluxDB.
It informs InfluxDB of the point's measurement, tag set, field set, and
timestamp.

{{< influxdb/line-protocol >}}

Moving across each element in the diagram:

### Measurement

The name of the [measurement](/enterprise_influxdb/v1/concepts/glossary/#measurement)
that you want to write your data to.
The measurement is required in line protocol.

In the example, the measurement name is `weather`.

### Tag set

The [tag(s)](/enterprise_influxdb/v1/concepts/glossary/#tag) that you want to include
with your data point.
Tags are optional in line protocol.

{{% note %}}
Avoid using the reserved keys `_field`, `_measurement`, and `time`.
If reserved keys are included as a tag or field key, the associated point is discarded.
{{% /note %}}

Notice that the measurement and tag set are separated by a comma and no spaces.

Separate tag key-value pairs with an equals sign `=` and no spaces:

```
<tag_key>=<tag_value>
```

Separate multiple tag-value pairs with a comma and no spaces:

```
<tag_key>=<tag_value>,<tag_key>=<tag_value>
```

In the example, the tag set consists of one tag: `location=us-midwest`.
Adding another tag (`season=summer`) to the example looks like this:

```
weather,location=us-midwest,season=summer temperature=82 1465839830100400200
```

For best performance you should sort tags by key before sending them to the
database.
The sort should match the results from the
[Go bytes.Compare function](http://golang.org/pkg/bytes/#Compare).

### First whitespace

Separate the measurement and the field set or, if you're including a tag set
with your data point, separate the tag set and the field set with a whitespace.
The whitespace is required in line protocol.

Valid line protocol with no tag set:

```
weather temperature=82 1465839830100400200
```

### Field set

The [field(s)](/enterprise_influxdb/v1/concepts/glossary/#field) for your data point.
Every data point requires at least one field in line protocol.

Separate field key-value pairs with an equals sign `=` and no spaces:

```
<field_key>=<field_value>
```

Separate multiple field-value pairs with a comma and no spaces:

```
<field_key>=<field_value>,<field_key>=<field_value>
```

In the example, the field set consists of one field: `temperature=82`.
Adding another field (`humidity=71`) to the example looks like this:

```
weather,location=us-midwest temperature=82,humidity=71 1465839830100400200
```

### Second whitespace

Separate the field set and the optional timestamp with a whitespace.
The whitespace is required in line protocol if you're including a timestamp.

### Timestamp

The [timestamp](/enterprise_influxdb/v1/concepts/glossary/#timestamp) for your data
point in nanosecond-precision Unix time.
The timestamp is optional in line protocol.
If you do not specify a timestamp for your data point InfluxDB uses the server's
local nanosecond timestamp in UTC.

In the example, the timestamp is `1465839830100400200` (that's
`2016-06-13T17:43:50.1004002Z` in RFC3339 format).
The line protocol below is the same data point but without the timestamp.
When InfluxDB writes it to the database it uses your server's
local timestamp instead of `2016-06-13T17:43:50.1004002Z`.

```
weather,location=us-midwest temperature=82
```

Use the InfluxDB API to specify timestamps with a precision other than nanoseconds,
such as microseconds, milliseconds, or seconds.
We recommend using the coarsest precision possible as this can result in
significant improvements in compression.
See the [API Reference](/enterprise_influxdb/v1/tools/api/#write-http-endpoint) for more information.

{{% note %}}
#### Use NTP to synchronize time between hosts

Use the Network Time Protocol (NTP) to synchronize time between hosts.
InfluxDB uses a host's local time in UTC to assign timestamps to data; if
hosts' clocks aren't synchronized with NTP, the timestamps on the data written
to InfluxDB can be inaccurate.
{{% /note %}}

## Data types

This section covers the data types of line protocol's major components:
[measurements](/enterprise_influxdb/v1/concepts/glossary/#measurement),
[tag keys](/enterprise_influxdb/v1/concepts/glossary/#tag-key),
[tag values](/enterprise_influxdb/v1/concepts/glossary/#tag-value),
[field keys](/enterprise_influxdb/v1/concepts/glossary/#field-key),
[field values](/enterprise_influxdb/v1/concepts/glossary/#field-value), and
[timestamps](/enterprise_influxdb/v1/concepts/glossary/#timestamp).


Measurements, tag keys, tag values, and field keys are always strings.

{{% note %}}
Because InfluxDB stores tag values as strings, InfluxDB cannot perform math on
tag values.
In addition, InfluxQL [functions](/enterprise_influxdb/v1/query_language/functions/)
do not accept a tag value as a primary argument.
It's a good idea to take into account that information when designing your
[schema](/enterprise_influxdb/v1/concepts/glossary/#schema).
{{% /note %}}

Timestamps are Unix timestamps.
The minimum valid timestamp is `-9223372036854775806` or `1677-09-21T00:12:43.145224194Z`.
The maximum valid timestamp is `9223372036854775806` or `2262-04-11T23:47:16.854775806Z`.
As mentioned above, by default, InfluxDB assumes that timestamps have
nanosecond precision.
See the [API Reference](/enterprise_influxdb/v1/tools/api/#write-http-endpoint) for how to specify
alternative precisions.

Field values can be floats, integers, strings, or Booleans:

- **Floats**: by default, InfluxDB assumes all numeric field values are floats.

  Store the field value `82` as a float:

  ```
  weather,location=us-midwest temperature=82 1465839830100400200
  ```

- **Integers**: append an `i` to the field value to tell InfluxDB to store the
  number as an integer.

  Store the field value `82` as an integer:

  ```
  weather,location=us-midwest temperature=82i 1465839830100400200
  ```

- **Strings**: double quote string field values (more on quoting in line
  protocol [below](#quoting)).

  Store the field value `too warm` as a string:

  ```
  weather,location=us-midwest temperature="too warm" 1465839830100400200
  ```

- **Booleans**: specify TRUE with `t`, `T`, `true`, `True`, or `TRUE`. Specify
FALSE with `f`, `F`, `false`, `False`, or `FALSE`.

  Store the field value `true` as a Boolean:

  ```
  weather,location=us-midwest too_hot=true 1465839830100400200
  ```

  {{% note %}}
  Acceptable Boolean syntax differs for data writes and data
  queries. See [Frequently Asked Questions](/enterprise_influxdb/v1/troubleshooting/frequently-asked-questions/#why-can-t-i-query-boolean-field-values)
  for more information.
  {{% /note %}}

Within a measurement, a field's type cannot differ within a
[shard](/enterprise_influxdb/v1/concepts/glossary/#shard), but it can differ across
shards. For example, writing an integer to a field that previously accepted
floats fails if InfluxDB attempts to store the integer in the same shard as the
floats:

```sql
> INSERT weather,location=us-midwest temperature=82 1465839830100400200
> INSERT weather,location=us-midwest temperature=81i 1465839830100400300
ERR: {"error":"field type conflict: input field \"temperature\" on measurement \"weather\" is type int64, already exists as type float"}
```

But, writing an integer to a field that previously accepted floats succeeds if
InfluxDB stores the integer in a new shard:

```sql
> INSERT weather,location=us-midwest temperature=82 1465839830100400200
> INSERT weather,location=us-midwest temperature=81i 1467154750000000000
>
```

See
[Frequently Asked Questions](/enterprise_influxdb/v1/troubleshooting/frequently-asked-questions/#how-does-influxdb-handle-field-type-discrepancies-across-shards)
for how field value type discrepancies can affect `SELECT *` queries.

## Quoting

This section covers when not to and when to double (`"`) or single (`'`)
quote in line protocol.
Moving from never quote to please do quote:

- Never double or single quote the timestamp--for example:

  ```sql
  > INSERT weather,location=us-midwest temperature=82 "1465839830100400200"
  ERR: {"error":"unable to parse 'weather,location=us-midwest temperature=82 \"1465839830100400200\"': bad timestamp"}
  ```

- Never single quote field values, even if they're strings--for example:

  ```sql
  > INSERT weather,location=us-midwest temperature='too warm'
  ERR: {"error":"unable to parse 'weather,location=us-midwest temperature='too warm'': invalid boolean"}
  ``` 

- Do not double or single quote measurement names, tag keys, tag values, and field
  keys unless the quotes are part of the name--for example:

  ```sql
  > INSERT weather,location=us-midwest temperature=82 1465839830100400200
  > INSERT "weather",location=us-midwest temperature=87 1465839830100400200
  ```
  
  ```
  > SHOW MEASUREMENTS
  name: measurements
  ------------------
  name
  "weather"
  weather
  ```

  To query data in `"weather"` you need to double quote the measurement name and
  escape the measurement's double quotes:

  ```sql
  > SELECT * FROM "\"weather\""
    
  name: "weather"
  ---------------
  time				            location	 temperature
  2016-06-13T17:43:50.1004002Z	us-midwest	 87
  ```

- Do not double quote field values that are floats, integers, or Booleans.
  InfluxDB parses quoted field values as strings--for example:

  ```sql
  > INSERT weather,location=us-midwest temperature="82"
  > SELECT * FROM weather WHERE temperature >= 70
  >
  ```

- Do double quote field values that are strings--for example:

  ```sql
  > INSERT weather,location=us-midwest temperature="too warm"
  > SELECT * FROM weather
  name: weather
  -------------
  time				            location	 temperature
  2016-06-13T19:10:09.995766248Z	us-midwest	 too warm
  ```

## Special characters and keywords

### Special characters

For tag keys, tag values, and field keys always use a backslash character `\`
to escape:

- commas (`,`)

  ```
  weather,location=us\,midwest temperature=82 1465839830100400200
  ```
- equal signs (`=`)

  ```
  weather,location=us-midwest temp\=rature=82 1465839830100400200
  ```
- spaces

  ```
  weather,location\ place=us-midwest temperature=82 1465839830100400200
  ```

For measurements always use a backslash character `\` to escape:

- commas (`,`)

  ```
  wea\,ther,location=us-midwest temperature=82 1465839830100400200
  ```

- spaces

  ```
  wea\ ther,location=us-midwest temperature=82 1465839830100400200
  ```

For string field values use a backslash character `\` to escape:

- double quotes (`"`)

  ```
  weather,location=us-midwest temperature="too\"hot\"" 1465839830100400200
  ```

Line protocol does not require you to escape the backslash character `\` but
you can if you want to--for example:

```
weather,location=us-midwest temperature_str="too hot/cold" 1465839830100400201
weather,location=us-midwest temperature_str="too hot\cold" 1465839830100400202
weather,location=us-midwest temperature_str="too hot\\cold" 1465839830100400203
weather,location=us-midwest temperature_str="too hot\\\cold" 1465839830100400204
weather,location=us-midwest temperature_str="too hot\\\\cold" 1465839830100400205
weather,location=us-midwest temperature_str="too hot\\\\\cold" 1465839830100400206
```

Is interpreted as:

```sql
time                location   temperature_str
----                --------   ---------------
1465839830100400201 us-midwest too hot/cold
1465839830100400202 us-midwest too hot\cold
1465839830100400203 us-midwest too hot\cold
1465839830100400204 us-midwest too hot\\cold
1465839830100400205 us-midwest too hot\\cold
1465839830100400206 us-midwest too hot\\\cold
```

{{% note %}}
Notice that a single and double backslash produce the same record.
{{% /note %}}

All other special characters also do not require escaping.
For example, line protocol handles emojis with no problem:

```sql
> INSERT we⛅️ther,location=us-midwest temper🔥ture=82 1465839830100400200
> SELECT * FROM "we⛅️ther"
name: we⛅️ther
------------------
time			              location	   temper🔥ture
1465839830100400200	 us-midwest	 82
```

### Keywords

Line protocol accepts
[InfluxQL keywords](/enterprise_influxdb/v1/query_language/spec/#keywords)
as [identifier](/enterprise_influxdb/v1/concepts/glossary/#identifier) names.
In general, we recommend avoiding using InfluxQL keywords in your schema as
it can cause
[confusion](/enterprise_influxdb/v1/troubleshooting/errors/#error-parsing-query-found-expected-identifier-at-line-char) when querying the data.

The keyword `time` is a special case.
`time` can be a
[continuous query](/enterprise_influxdb/v1/concepts/glossary/#continuous-query-cq) name,
database name,
[measurement](/enterprise_influxdb/v1/concepts/glossary/#measurement) name,
[retention policy](/enterprise_influxdb/v1/concepts/glossary/#retention-policy-rp) name,
[subscription](/enterprise_influxdb/v1/concepts/glossary/#subscription) name, and
[user](/enterprise_influxdb/v1/concepts/glossary/#user) name.
In those cases, `time` does not require double quotes in queries.
`time` cannot be a [field key](/enterprise_influxdb/v1/concepts/glossary/#field-key) or
[tag key](/enterprise_influxdb/v1/concepts/glossary/#tag-key);
InfluxDB rejects writes with `time` as a field key or tag key and returns an error.
See [Frequently Asked Questions](/enterprise_influxdb/v1/troubleshooting/frequently-asked-questions/#time) for more information.

## Writing data to InfluxDB

### Getting data in the database

Now that you know all about the InfluxDB line protocol, how do you actually get the
line protocol to InfluxDB?
Here, we'll give two quick examples and then point you to the
[Tools](/enterprise_influxdb/v1/tools/) sections for further
information.

#### InfluxDB API

Write data to InfluxDB using the InfluxDB API.
Send a `POST` request to the `/write` endpoint and provide your line protocol in
the request body:

```bash
curl -i -XPOST "http://localhost:8086/write?db=science_is_cool" --data-binary 'weather,location=us-midwest temperature=82 1465839830100400200'
```

For in-depth descriptions of query string parameters, status codes, responses,
and more examples, see the [API Reference](/enterprise_influxdb/v1/tools/api/#write-http-endpoint).

#### CLI

Write data to InfluxDB using the InfluxDB command line interface (CLI).
[Launch](/enterprise_influxdb/v1/tools/influx-cli/use-influx/#launch-influx) the CLI, use the relevant
database, and put `INSERT` in
front of your line protocol:

```sql
INSERT weather,location=us-midwest temperature=82 1465839830100400200
```

You can also use the CLI to
[import](/enterprise_influxdb/v1/tools/influx-cli/use-influx/#import-data-from-a-file-with-import) Line
Protocol from a file.

There are several ways to write data to InfluxDB.
See the [Tools](/enterprise_influxdb/v1/tools/) section for more
on the [InfluxDB API](/enterprise_influxdb/v1/tools/api/#write-http-endpoint), the
[CLI](/enterprise_influxdb/v1/tools/influx-cli/use-influx/), and the available Service Plugins (
[UDP](/enterprise_influxdb/v1/tools/udp/),
[Graphite](/enterprise_influxdb/v1/tools/graphite/),
[CollectD](/enterprise_influxdb/v1/tools/collectd/), and
[OpenTSDB](/enterprise_influxdb/v1/tools/opentsdb/)).

### Duplicate points

A point is uniquely identified by the measurement name, tag set, and timestamp.
If you submit line protocol with the same measurement, tag set, and timestamp,
but with a different field set, the field set becomes the union of the old
field set and the new field set, where any conflicts favor the new field set.

For a complete example of this behavior and how to avoid it, see
[How does InfluxDB handle duplicate point?](/enterprise_influxdb/v1/troubleshooting/frequently-asked-questions/#how-does-influxdb-handle-duplicate-points)

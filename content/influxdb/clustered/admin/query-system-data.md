--- 
title: Query system data
description: >
  Query system tables in your InfluxDB cluster to see data related
  to queries, tables, partitions, and compaction in your cluster.
menu:
  influxdb_clustered:
    parent: Administer InfluxDB Cloud
    name: Query system data
weight: 105
related:
  - /influxdb/clustered/reference/cli/influxctl/query/
--- 

{{< product-name >}} stores data related to queries, tables, partitions, and
compaction in _system tables_ within your cluster.
You can query the cluster system tables for information about your cluster.

- [Query system tables](#query-system-tables)
  - [Optimize queries to reduce impact to your cluster](#optimize-queries-to-reduce-impact-to-your-cluster)
- [System tables](#system-tables)
  - [system.queries](#systemqueries)
  - [system.tables](#systemtables)
  - [system.partitions](#systempartitions)
  - [system.compactor](#systemcompactor)
- [System query examples](#system-query-examples)
  - [Query logs](#query-logs)
  - [Partitions](#partitions)
  - [Storage usage](#storage-usage)
  - [Compaction](#compaction)

{{% warn %}}
#### May impact cluster performance

Querying InfluxDB v3 system tables may impact write and query
performance of your {{< product-name omit=" Clustered" >}} cluster.
Use filters to [optimize queries to reduce impact to your cluster](#optimize-queries-to-reduce-impact-to-your-cluster).

<!--------------- UPDATE THE DATE BELOW AS EXAMPLES ARE UPDATED --------------->

#### System tables are subject to change

System tables are not part of InfluxDB's stable API and may change with new releases.
The provided schema information and query examples are valid as of **September 18, 2024**.
If you detect a schema change or a non-functioning query example, please
[submit an issue](https://github.com/influxdata/docs-v2/issues/new/choose).

<!--------------- UPDATE THE DATE ABOVE AS EXAMPLES ARE UPDATED --------------->
{{% /warn %}}

## Query system tables


{{% note %}}
Querying system tables with `influxctl` requires **`influxctl` v2.8.0 or newer**.
{{% /note %}}

Use the [`influxctl query` command](/influxdb/clustered/reference/cli/influxctl/query/)
and SQL to query system tables.
Provide the following:

- **Enable system tables** with the `--enable-system-tables` command flag.
- **Database token**: A [database token](/influxdb/clustered/admin/tokens/#database-tokens)
  with read permissions on the specified database. Uses the `token` setting from
  the [`influxctl` connection profile](/influxdb/clustered/reference/cli/influxctl/#configure-connection-profiles)
  or the `--token` command flag.
- **Database name**: The name of the database to query information about.
  Uses the `database` setting from the
  [`influxctl` connection profile](/influxdb/clustered/reference/cli/influxctl/#configure-connection-profiles)
  or the `--database` command flag.
- **SQL query**: The SQL query to execute.

  Pass the query in one of the following ways:

  - a string on the command line
  - a path to a file that contains the query
  - a single dash (`-`) to read the query from stdin

{{% code-placeholders "DATABASE_(TOKEN|NAME)|SQL_QUERY" %}}

{{< code-tabs-wrapper >}}
{{% code-tabs %}}
[string](#)
[file](#)
[stdin](#)
{{% /code-tabs %}}
{{% code-tab-content %}}
```sh
influxctl query \
  --enable-system-tables \
  --database DATABASE_NAME \
  --token DATABASE_TOKEN \
  "SQL_QUERY"
```
{{% /code-tab-content %}}
{{% code-tab-content %}}
```sh
influxctl query \
  --enable-system-tables \
  --database DATABASE_NAME \
  --token DATABASE_TOKEN \
  /path/to/query.sql
```
{{% /code-tab-content %}}
{{% code-tab-content %}}
```sh
cat ./query.sql | influxctl query \
  --enable-system-tables \
  --database DATABASE_NAME \
  --token DATABASE_TOKEN \
  - 
```
{{% /code-tab-content %}}
{{< /code-tabs-wrapper >}}

{{% /code-placeholders %}}

Replace the following:

- {{% code-placeholder-key %}}`DATABASE_TOKEN`{{% /code-placeholder-key %}}:
  A database token with read access to the specified database
- {{% code-placeholder-key %}}`DATABASE_NAME`{{% /code-placeholder-key %}}:
  The name of the database to query information about.
- {{% code-placeholder-key %}}`SQL_QUERY`{{% /code-placeholder-key %}}:
  The SQL query to execute. For examples, see
  [System query examples](#system-query-examples).

When prompted, enter `y` to acknowledge the potential impact querying system
tables may have on your cluster.

### Optimize queries to reduce impact to your cluster

Querying InfluxDB v3 system tables may impact the performance of your
{{< product-name omit=" Clustered" >}} cluster.
As you write data to a cluster, the number of partitions and Parquet files
can increase to a point that impacts system table performance.
Queries that took milliseconds with fewer files and partitions might take 10
seconds or longer as files and partitions increase.

Use the following filters to optimize your system table queries and reduce the impact on your
cluster's performance.

##### Filter by table name

Filter by `table_name` when querying `system.tables`, `system.partitions`, or `system.compactor`. 

```sql
WHERE table_name = 'TABLE_NAME' (string)
```

##### Filter by partition key 

Filter by `partition_key` when querying `system.partitions` or `system.compactor`.

```sql
WHERE partition_key = 'PARTITION_KEY (string)
```

To further improve performance, use `AND` to pair `partition_key` with `table_name`--for example: 
   
```sql
SELECT * 
FROM system.partitions 
WHERE table_name = 'TABLE_NAME' 
 AND partition_key = 'PARTITION_KEY';
```

- `PARTITION_KEY`: a [partition key]((/influxdb/clustered/admin/custom-partitions/#partition-keys))
   derived from the table's partition template.

###### Partition key format

The default format for `partition_key` is `%Y-%m-%d` (for example, `2024-01-01`).
If the table uses a custom partition template, the `partition_key` format will differ.

##### Filter by partition ID 

Filter by `partition_id` when querying `system.partitions` or `system.compactor`.

```sql
WHERE partition_id = PARTITION_ID (int64)
```

For the most optimized approach, use `AND` to pair `partition_id` with `table_name`--for example:

```sql
SELECT * 
FROM system.partitions 
WHERE table_name = 'TABLE_NAME' 
 AND partition_id = PARTITION_ID;
```

- `PARTITION_ID`: a [partition ID](#retrieve-a-partition-id)

Although you don't need to pair `partition_id` with `table_name` (because a partition ID is unique within a cluster),
it's the most optimized approach, _especially when you have many tables in a database_.

###### Retrieve a partition ID

To retrieve a partition ID, query `system.partitions` for a `table_name` and `partition_key` pair--for example:

```sql
SELECT * 
FROM system.partitions
WHERE table_name = 'TABLE_NAME'
  AND partition_key = 'PARTITION_KEY';
```

The result contains the `partition_id`:

| partition_id | table_name | partition_key     | last_new_file_created_at | num_files | total_size_mb |
| -----------: | :--------- | :---------------- | -----------------------: | --------: | ------------: |
|         1362 | weather    | 43 \| 2020-05-27  |      1683747418763813713 |         1 |             0 |

##### Combine filters for performance improvement

Use the `AND`, `OR`, or `IN` keywords to combine filters in your query.

- **Use `OR` or `IN` conditions when filtering for different values in the same column**--for example:
  
  ```sql
  WHERE partition_id = 1 OR partition_id = 2
  ```

  Use `IN` to make multiple `OR` conditions more readable--for example:
  
  ```sql
  WHERE table_name IN ('foo', 'bar', 'baz')
  ```

- **Avoid mixing different columns in `OR` conditions**, as this won't improve performance--for example:
  
  ```sql
  WHERE table_name = 'foo' OR partition_id = 2  -- This will not improve performance
  ```

## System tables

{{% warn %}}
_System tables are [subject to change](#system-tables-are-subject-to-change)._
{{% /warn %}}

- [system.queries](#systemqueries)
- [system.tables](#systemtables)
- [system.partitions](#systempartitions)
- [system.compactor](#systemcompactor)

### system.queries

The `system.queries` table contains an unpersisted log of queries run against
the current [InfluxDB Querier](/influxdb/clustered/reference/internals/storage-engine/#querier)
to which your query is routed.
The query log is specific to the current Querier and is not shared across Queriers
in your cluster.
Logs are scoped to the specified database.

{{< expand-wrapper >}}
{{% expand "View `system.queries` schema" %}}

The `system.queries` table contains the following columns:

- id
- phase
- issue_time
- query_type
- query_text  
- partitions
- parquet_files
- plan_duration
- permit_duration
- execute_duration
- end2end_duration
- compute_duration
- max_memory
- success
- running
- cancelled
- trace_id

{{% /expand %}}
{{< /expand-wrapper >}}

### system.tables

The `system.tables` table contains information about tables in the specified database.

{{< expand-wrapper >}}
{{% expand "View `system.tables` schema" %}}

The `system.tables` table contains the following columns:

- table_name
- partition_template

{{% /expand %}}
{{< /expand-wrapper >}}

### system.partitions

The `system.partitions` table contains information about partitions associated
with the specified database.

{{< expand-wrapper >}}
{{% expand "View `system.partitions` schema" %}}

The `system.partitions` table contains the following columns:

- partition_id
- table_name
- partition_key
- last_new_file_created_at
- num_files
- total_size_mb

{{% /expand %}}
{{< /expand-wrapper >}}

### system.compactor

The `system.compaction` table contains information about compacted partition Parquet
files associated with the specified database.

{{< expand-wrapper >}}
{{% expand "View `system.compactor` schema" %}}

The `system.compactor` table contains the following columns:

- partition_id
- table_name 
- partition_key
- total_l0_files
- total_l1_files
- total_l2_files
- total_l0_bytes
- total_l1_bytes
- total_l2_bytes
- skipped_reason

{{% /expand %}}
{{< /expand-wrapper >}} 

## System query examples

{{% warn %}}
#### May impact cluster performance

Querying InfluxDB v3 system tables may impact write and query
performance of your {{< product-name omit=" Clustered" >}} cluster.

The examples in this section include `WHERE` filters to [optimize queries and reduce impact to your cluster](#optimize-queries-to-reduce-impact-to-your-cluster).
{{% /warn %}}

- [Query logs](#query-logs)
  - [View all stored query logs](#view-all-stored-query-logs)
  - [View query logs for queries with end-to-end durations above a threshold](#view-query-logs-for-queries-with-end-to-end-durations-above-a-threshold)
- [Partitions](#partitions)
  - [View partition templates of all tables](#view-partition-templates-of-all-tables)
  - [View the partition template of a specific table](#view-the-partition-template-of-a-specific-table)
  - [View all partitions for a table](#view-all-partitions-for-a-table)
  - [View the number of partitions per table](#view-the-number-of-partitions-per-table)
  - [View the number of partitions for a specific table](#view-the-number-of-partitions-for-a-specific-table)
- [Storage usage](#storage-usage)
  - [View the size of tables in megabytes](#view-the-size-of-tables-in-megabytes)
  - [View the size of a specific table in megabytes](#view-the-size-of-a-specific-table-in-megabytes)
  - [View the total size of all compacted partitions per table in bytes](#view-the-total-size-of-all-compacted-partitions-per-table-in-bytes)
  - [View the total size of all compacted partitions in bytes](#view-the-total-size-of-all-compacted-partitions-in-bytes)
- [Compaction](#compaction)
  - [View overall compaction totals for each table](#view-overall-compaction-totals-for-each-table)
  - [View overall compaction totals for a specific table](#view-overall-compaction-totals-for-a-specific-table)

In the examples below, replace {{% code-placeholder-key %}}`TABLE_NAME`{{% /code-placeholder-key %}}
with the name of the table you want to query information about.

--- 

{{% code-placeholders "TABLE_NAME" %}}

### Query logs

#### View all stored query logs

```sql
SELECT * FROM system.queries
```

#### View query logs for queries with end-to-end durations above a threshold

The following returns query logs for queries with an end-to-end duration greater
than 50 milliseconds.

```sql
SELECT *
FROM
  system.queries
WHERE
  end2end_duration::BIGINT > (50 * 1000000)
```

--- 

### Partitions

#### View the partition template of a specific table

```sql
SELECT *
FROM
  system.tables
WHERE
  table_name = 'TABLE_NAME'
```

#### View all partitions for a table

```sql
SELECT *
FROM
  system.partitions
WHERE
  table_name = 'TABLE_NAME'
```

#### View the number of partitions per table

```sql
SELECT
  table_name,
  COUNT(*) AS partition_count
FROM
  system.partitions
WHERE
  table_name IN ('foo', 'bar', 'baz')
GROUP BY
  table_name
```

#### View the number of partitions for a specific table

```sql
SELECT
  COUNT(*) AS partition_count
FROM
  system.partitions
WHERE
  table_name = 'TABLE_NAME'
```

--- 

### Storage usage

#### View the size of tables in megabytes

```sql
SELECT
  table_name,
  SUM(total_size_mb) AS total_size_mb
FROM
  system.partitions
WHERE
  table_name IN ('foo', 'bar', 'baz')
GROUP BY
  table_name
```

#### View the size of a specific table in megabytes

```sql
SELECT
  SUM(total_size_mb) AS total_size_mb
FROM
  system.partitions
WHERE
  table_name = 'TABLE_NAME'
```

#### View the total size of all compacted partitions per table in bytes

```sql
SELECT
  table_name,
  SUM(total_l0_bytes) + SUM(total_l1_bytes) + SUM(total_l2_bytes) AS total_bytes
FROM
  system.compactor
WHERE
  table_name IN ('foo', 'bar', 'baz')
GROUP BY
  table_name
```

#### View the total size of all compacted partitions in bytes

```sql
SELECT
  SUM(total_l0_bytes) + SUM(total_l1_bytes) + SUM(total_l2_bytes) AS total_bytes
FROM
  system.compactor
WHERE
  table_name = 'TABLE_NAME'
```

--- 

### Compaction

#### View overall compaction totals for each table

```sql
SELECT
  table_name,
  SUM(total_l0_files) AS total_l0_files,
  SUM(total_l1_files) AS total_l1_files,
  SUM(total_l2_files) AS total_l2_files,
  SUM(total_l0_bytes) AS total_l0_bytes,
  SUM(total_l1_bytes) AS total_l1_bytes,
  SUM(total_l2_bytes) AS total_l2_bytes
FROM
  system.compactor
WHERE
  table_name IN ('foo', 'bar', 'baz')
GROUP BY
  table_name
```

#### View overall compaction totals for a specific table

```sql
SELECT
  SUM(total_l0_files) AS total_l0_files,
  SUM(total_l1_files) AS total_l1_files,
  SUM(total_l2_files) AS total_l2_files,
  SUM(total_l0_bytes) AS total_l0_bytes,
  SUM(total_l1_bytes) AS total_l1_bytes,
  SUM(total_l2_bytes) AS total_l2_bytes
FROM
  system.compactor
WHERE
  table_name = 'TABLE_NAME'
```

{{% /code-placeholders %}}
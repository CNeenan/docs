---
title: EXPLAIN ANALYZE
summary: The EXPLAIN ANALYZE statement executes a query and generates a physical statement plan with execution statistics.
toc: true
---

The `EXPLAIN ANALYZE` [statement](sql-statements.html) **executes a SQL query** and generates a statement plan with execution statistics. The `(DEBUG)` option generates a URL to download a bundle with more details about the statement plan for advanced debugging. Statement plans provide information around SQL execution, which can be used to troubleshoot slow queries by figuring out where time is being spent, how long a processor (i.e., a component that takes streams of input rows and processes them according to a specification) is not doing work, etc. For more information about distributed SQL queries, see the [DistSQL section of our SQL Layer Architecture docs](architecture/sql-layer.html#distsql).

{{site.data.alerts.callout_info}}
{% include {{ page.version.version }}/sql/physical-plan-url.md %}
{{site.data.alerts.end}}

## Aliases

In CockroachDB, the following are aliases for `EXPLAIN ANALYZE`:

- `EXPLAIN ANALYSE`

## Synopsis

<section>{% include {{ page.version.version }}/sql/generated/diagrams/explain_analyze.html %}</section>

## Parameters

Parameter          | Description
-------------------|-----------
`PLAN`             | <span class="version-tag">New in v21.1:</span> <br /> _(Default)_ Executes the statement and returns CockroachDB's statement plan with planning and execution time for an [explainable statement](sql-grammar.html#preparable_stmt). For more information, see [Default option](#default-option).
`DISTSQL`          | Return the statement plan and performance statistics as well as a generated link to a graphical distributed SQL physical statement plan tree.
`DEBUG`            |  Generate a ZIP file containing files with detailed information about the query and the database objects referenced in the query. For more information, see [`DEBUG` option](#debug-option).
`preparable_stmt`  | The [statement](sql-grammar.html#preparable_stmt) you want to execute and analyze. All preparable statements are explainable.

## Required privileges

The user requires the appropriate [privileges](authorization.html#assign-privileges) for the statement being explained.

## Success responses

Successful `EXPLAIN ANALYZE` statements return tables with the following details in the `info` column:

 Detail | Description
--------|------------
[Global properties](#global-properties) | The properties and statistics that apply to the entire statement plan.
[Statement plan tree](#statement-plan-tree-properties) | A tree representation of the hierarchy of the statement plan.
Node details | The properties, columns, and ordering details for the current statement plan node in the tree.
Time | The time details for the statement. The total time is the planning and execution time of the statement. The execution time is the time it took for the final statement plan to complete. The network time is the amount of time it took to distribute the statement across the relevant nodes in the cluster. Some statements do not need to be distributed, so the network time is 0ms.

If you use the `DISTSQL` option, the statement will also return a URL generated for a physical statement plan that provides high level information about how a statement will be executed. For details about reading the physical statement plan, see [DistSQL Plan Viewer](#distsql-plan-viewer).<br><br>{{site.data.alerts.callout_info}}{% include {{ page.version.version }}/sql/physical-plan-url.md %} {{site.data.alerts.end}}

If you use the [`DEBUG` option](#debug-option), the statement will return a single `text` column with a URL and instructions to download the `DEBUG` bundle, which includes the physical statement plan.

### Global properties

Global property | Description
----------------|------------
planning time | The total time the planner took to create a statement plan.
execution time | The time it took for the final statement plan to complete.
distribution | Shows whether the statement was distributed or local. If `distribution` is `full`, execution of the statement is performed by multiple nodes in parallel, then the final results are returned by the gateway node. If `local`, the execution plan is performed only on the gateway node. Even if the execution plan is `local`, row data may be fetched from remote nodes, but the processing of the data is performed by the local node.
vectorized | Indicates whether the [vectorized execution engine](vectorized-execution.html) was used in this statement.
rows read from KV | The number of rows read from the [Storage layer](architecture/storage-layer.html).
cumulative time spent in KV | The total amount of time spent in the [Storage layer](architecture/storage-layer.html).
cumulative time spent due to contention | The total amount of time this statement was in contention with another transaction during execution.
maximum memory usage | The maximum amount of memory used by this statement anytime during its execution.
network usage | The amount of data transferred over the network while the statement was executed. If this value is 0 B, the statement was executed on a single node and didn't use the network.

### Statement plan tree properties

The statement plan tree shows

Statement plan tree properties | Description
-------------------------------|------------
processor | Each processor in the statement plan hierarchy has a node with details about that phase of the statement. For example, a statement with a `GROUP BY` clause has a `group` processor with details about the cluster nodes, rows, and operations related to the `GROUP BY` operation.
cluster nodes | The names of the CockroachDB cluster nodes affected by this phase of the statement.
actual row count | The actual number of rows affected by this processor during execution.
estimated row count | The estimated number of rows affected by this processor according to the statement planner.
KV rows read | During scans, the number of rows in the [Storage layer](architecture/storage-layer.html) read by this phase of the statement.
KV bytes read | During scans, the amount of data read from the [Storage layer](architecture/storage-layer.html) during this phase of the statement.
table | The table and index used in a scan operation in a statement, in the form `{table name}@{index name}`.
spans | The interval of the key space read by the processor. If `spans` is `FULL SCAN` the table is scanned on all key ranges of the index. If `spans` is `[/1 - /1]` only the key with value `1` is read by the processor.

## Default option

<span class="version-tag">New in v21.1:</span> By default, `EXPLAIN ANALYZE` uses the `PLAN` option. `EXPLAIN ANALYZE` and `EXPLAIN ANALYZE (PLAN)` produce the same output.

## `DISTSQL` option

`EXPLAIN ANALYZE (DISTSQL)` generates a physical statement plan diagram in the [DistSQL Plan Viewer](#distsql-plan-viewer). The DistSQL Plan Viewer displays the physical statement plan, as well as execution statistics. The statistics listed depend on the query type and the [execution engine used](vectorized-execution.html). There will be multiple diagrams if the query contains subqueries or post-queries.

{{site.data.alerts.callout_info}}
<span class="version-tag">New in v21.1:</span> `EXPLAIN ANALYZE (DISTSQL)` can only be used as the top-level statement in a query.
{{site.data.alerts.end}}

## `DEBUG` option

 `EXPLAIN ANALYZE (DEBUG)` executes a query and generates a link to a ZIP file that contains the [physical statement plan](#distsql-plan-viewer), execution statistics, statement tracing, and other information about the query.

        File        | Description
--------------------+-------------------
`stats-<table>.sql` | Contains [statistics](create-statistics.html) for a table in the query.
`schema.sql`        | Contains [`CREATE`](create-table.html) statements for objects in the query.
`env.sql`           | Contains information about the CockroachDB environment.
`trace.txt`         | Contains [statement traces](show-trace.html) in plaintext format.
`trace.json`        | Contains [statement traces](show-trace.html) in JSON format.
`trace-jaeger.json` | Contains [statement traces](show-trace.html) in JSON format that can be [imported to Jaeger](query-behavior-troubleshooting.html#visualize-statement-traces-in-jaeger).
`distsql.html`      | The query's [physical statement plan](#distsql-plan-viewer). This diagram is identical to the one generated by [`EXPLAIN(DISTSQL)`](explain.html#distsql-option)
`plan.txt`          | The query execution plan. This is identical to the output of [`EXPLAIN (VERBOSE)`](explain.html#verbose-option).
`opt.txt`           | The statement plan tree generated by the [cost-based optimizer](cost-based-optimizer.html). This is identical to the output of [`EXPLAIN (OPT)`](explain.html#opt-option).
`opt-v.txt`         | The statement plan tree generated by the [cost-based optimizer](cost-based-optimizer.html), with cost details. This is identical to the output of [`EXPLAIN (OPT, VERBOSE)`](explain.html#opt-option).
`opt-vv.txt`        | The statement plan tree generated by the [cost-based optimizer](cost-based-optimizer.html), with cost details and input column data types. This is identical to the output of [`EXPLAIN (OPT, TYPES)`](explain.html#opt-option).
`vec.txt`           | The statement plan tree generated by the [vectorized execution](vectorized-execution.html) engine. This is identical to the output of [`EXPLAIN (VEC)`](explain.html#vec-option).
`vec-v.txt`         | The statement plan tree generated by the [vectorized execution](vectorized-execution.html) engine. This is identical to the output of [`EXPLAIN (VEC, VERBOSE)`](explain.html#vec-option).

`statement.txt`     | The SQL statement for the query.

You can obtain this ZIP file by following the link provided in the `EXPLAIN ANALYZE (DEBUG)` output, or by activating [statement diagnostics](ui-statements-page.html#diagnostics) in the DB Console.

{% include {{ page.version.version }}/sql/statement-bundle-warning.md %}

## DistSQL plan viewer

The graphical diagram when using the `DISTSQL` option displays the processors and operations that make up the statement plan. While the text output from `PLAN` shows the statement plan across the cluster, `DISTSQL` shows details on each node involved in the query.

Field | Description | Execution engine
------+-------------+----------------
&lt;Processor&gt;/&lt;id&gt; | The processor and processor ID used to read data into the SQL execution engine.<br><br>A processor is a component that takes streams of input rows, processes them according to a specification, and outputs one stream of rows. For example, a "TableReader" processor reads in data, and an "Aggregator" aggregates input rows. | Both
&lt;table&gt;@&lt;index&gt; | The index used by the processor. | Both
Spans | The interval of the key space read by the processor. For example, `[/1 - /1]` indicates that only the key with value `1` is read by the processor. | Both
Out | The output columns. | Both
cluster nodes | The names of the CockroachDB cluster nodes involved in the execution of this processor. | Both
batches output | The number of batches of columnar data output. | Vectorized engine only
rows output | The number of rows output. | Vectorized engine only
IO time | How long the TableReader processor spent reading data from disk. | Vectorized engine only
stall time | How long the processor spent not doing work. This is aggregated into the stall time numbers as the query progresses down the tree (i.e., stall time is added up and overlaps with previous time). | Row-oriented engine only
bytes read | The size of the data read by the processor. | Both
rows read | The number of rows read by the processor. | Both
KV time | The total time this phase of the query was in the [Storage layer](architecture/storage-layer.html). | Both
KV contention time | The time the [Storage layer](architecture/storage-layer.html) was in contention during this phase of the query. | Both
KV rows read | During scans, the number of rows in the [Storage layer](architecture/storage-layer.html) read by this phase of the query. | Both
KV bytes read | During scans, the amount of data read from the [Storage layer](architecture/storage-layer.html) during this phase of the query. | Both
@&lt;n&gt; | The index of the column relative to the input. | Both
max memory used | How much memory (if any) is used to buffer rows. | Row-oriented engine only
max disk used | How much disk (if any) is used to buffer data. Routers and processors will spill to disk buffering if there is not enough memory to buffer the data. | Row-oriented engine only
execution time | How long the engine spent executing the processor. | Vectorized engine only
max vectorized memory allocated | How much memory is allocated to the processor to buffer batches of columnar data. | Vectorized engine only
max vectorized disk used | How much disk (if any) is used to buffer columnar data. Processors will spill to disk buffering if there is not enough memory to buffer the data. | Vectorized engine only
left(@&lt;n&gt;)=right(@&lt;n&gt;) | The equality columns used in the join. | Both
stored side | The smaller table that was stored as an in-memory hash table. | Both
rows routed | How many rows were sent by routers, which can be used to understand network usage. | Row-oriented engine only
network latency | The latency time in nanoseconds between nodes in a stream. | Vectorized engine only
bytes sent | The number of actual bytes sent (i.e., encoding of the rows). This is only relevant when doing network communication. | Both
Render | The stage that renders the output. | Both
by hash | _(Orange box)_ The router, which is a component that takes one stream of input rows and sends them to a node according to a routing algorithm.<br><br>For example, a hash router hashes columns of a row and sends the results to the node that is aggregating the result rows. | Both
unordered / ordered | _(Blue box)_ A synchronizer that takes one or more output streams and merges them to be consumable by a processor. An ordered synchronizer is used to merge ordered streams and keeps the rows in sorted order. | Both
&lt;data type&gt; |  If [`EXPLAIN (DISTSQL, TYPES)`](explain.html#distsql-option) is specified, lists the data types of the input columns. | Both
Response | The response back to the client. | Both

## Examples

To run the examples, initialize a demo cluster with the MovR workload.

{% include {{ page.version.version }}/demo_movr.md %}

### `EXPLAIN ANALYZE`

Use `EXPLAIN ANALYZE` without an option, or equivalently with the `PLAN` option, to execute a query and display the physical statement plan with execution statistics.

For example, the following `EXPLAIN ANALYZE` statement executes a simple query against the [MovR database](movr.html) and then displays the physical statement plan with execution statistics:

{% include copy-clipboard.html %}
~~~ sql
> EXPLAIN ANALYZE SELECT city, AVG(revenue) FROM rides GROUP BY city;
~~~

~~~
                                          info
-----------------------------------------------------------------------------------------
  planning time: 475µs
  execution time: 53ms
  distribution: full
  vectorized: true
  rows read from KV: 125,000 (21 MiB)
  cumulative time spent in KV: 132ms
  maximum memory usage: 1.1 MiB
  network usage: 2.9 KiB (25 messages)

  • group
  │ cluster nodes: n1, n2, n3
  │ actual row count: 9
  │ estimated row count: 9
  │ group by: city
  │ ordered: +city
  │
  └── • scan
        cluster nodes: n1, n2, n3
        actual row count: 125,000
        KV rows read: 125,000
        KV bytes read: 21 MiB
        estimated row count: 125,000 (100% of the table; stats collected 2 minutes ago)
        table: rides@primary
        spans: FULL SCAN
(24 rows)

Time: 54ms total (execution 54ms / network 0ms)
~~~

### `EXPLAIN ANALYZE (DISTSQL)`

Use `EXPLAIN ANALYZE (DISTSQL)` to execute a query, display the physical statement plan with execution statistics, and generate a link to a graphical DistSQL statement plan.

~~~ sql
EXPLAIN ANALYZE (DISTSQL) SELECT city, AVG(revenue) FROM rides GROUP BY city;
~~~

~~~
                          info
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  planning time: 510µs
  execution time: 58ms
  distribution: full
  vectorized: true
  rows read from KV: 125,000 (21 MiB)
  cumulative time spent in KV: 146ms
  maximum memory usage: 1.1 MiB
  network usage: 2.9 KiB (25 messages)

  • group
  │ cluster nodes: n1, n2, n3
  │ actual row count: 9
  │ estimated row count: 9
  │ group by: city
  │ ordered: +city
  │
  └── • scan
        cluster nodes: n1, n2, n3
        actual row count: 125,000
        KV rows read: 125,000
        KV bytes read: 21 MiB
        estimated row count: 125,000 (100% of the table; stats collected 15 minutes ago)
        table: rides@primary
        spans: FULL SCAN

  Diagram: https://cockroachdb.github.io/distsqlplan/decode.html#eJy8V81u4zYQvvcpCJ6yqDYWKVmSdfLuIm2DTexFfgosqiBgpKkjRJa8JJ3YDfJYvfTYh-jzFJTsja0fWnbsvQgiRzOa-Wa-T9QzFt8S7OPLk7OTT1cojOXcQOxxdMThEdIpvEO_XAzPEY8jEOjXi-H1F_Txa_4YNnCaRTBgYxDY_wMTbGCKDWzhGwNPeBaCEBlXpuf8wdNohn3TwHE6mUq1fWPgMOOA_WcsY5kA9vEVu0vgAlgEvGNiA0cgWZzk4fMM-hMejxlX776csFT46H0nwGwsJPCIjQPcCXAQzEI3CGakdxwEM9ZV1z97_SCYmWYQzDxzeVe-kGflagYYG3g4lT7qU6NPVBqff0cyHoOPiD0WxTrMUgmpjLN0YTL__Xth4tmTQBxY5CNiGZ5Hi-27uYTlPj220Hn8ERv4jsnwHgTKpnKi3klsbOA8wvedIsbNi4GLrQV2QrIRYJ-8GO3x_TAacRgxmfGOtQ5vX7VvyCPgEPkoX30YfL0dDK9uB9dnZ0d98k6Bfn1-1Kfq7tPwenC1uIcZhNMVKEgOUqWySmGlml7TvJujeybuKxnevLzWTRvrfo2TFfWU4_xcBNKA47wNnMvr89tTBY-lVheQRsCVn4H6tNO3aiBzSDE-G0GzGgfBagHINK2DpBaNQfY-m3R660BUOl2fdWWGe41Z22tZk_byQFrJwwHUQSlOkgnE0hEkIJZRGVMXFTqC__5RocLfWkQ9KcwYsTRCFGXyHrhoFiCXbidATs-wbbsqQIQ26I_jlXu3CNFOfzY0cIVi9uH0p9tOf7w96g_Zr_64P1p_CO3WU5mWUeu2FCDansq0LZX3TztF5hSe0DzjD8uQJPw2VFwHmZxuDmU6_QV7ScHeZvLa1nbktYnhuDXkdY9JA3vtypAvYrRj74aerQxo93DstdqxtzyHb2Ev3S97vR_NXstteXooH7nanB5qar0AMclSAaVTRH1kU6EF0QgKdEU25SF84VmYv6ZYDnO_fCMCIQsrKRanaWFSCa46k7IzWXWma84kz0bWn2GKz8WYzdAYxhmfI5YkWcik6hYx0eecacosQq4ARlEsHlYfMlEdF80y-KYCf5sanMYaUpBPGX9ACZOQhnMfdR1ajMDS8sRiuSzQyQuMQACPWRL_xVaqp-a630J-Qogf8_pXTEsJWtosWhS-tI9BCDZaf6T1UG6DS28LXNzebrhYWlhMDSwbQaE7DgvVssVaB6XsbOmpZuq5Zmu9u3rn7o5ErTbTdhqb6VBNM4mn7SbVDbmznyGn1X7qcWkmf_XfUSNg1u4CVv_J3aaGLYjqWt6OvSW7CxihhxIwZxuu7jrzrqfBxdELe1eDi9N1NuFit_5VWMfF1eLi6YXE25uQdJuHjZAF4A3fBaqF1dKNm2fvZdysKqx6ZLY4R7hm8_fSpTpg3sLDwx0k9MA061NZYxelH15ja2ro7U1LNGOv1RLL3rm31N2fxt68_PR_AAAA___8Kx2h
(26 rows)

Time: 62ms total (execution 61ms / network 0ms)
~~~

To view the [DistSQL Plan Viewer](#distsql-plan-viewer), point your browser to the URL provided.

### `EXPLAIN ANALYZE (DEBUG)`

Use the [`DEBUG`](#debug-option) option to generate a ZIP file containing files with information about the query and the database objects referenced in the query. For example:

{% include copy-clipboard.html %}
~~~ sql
> EXPLAIN ANALYZE (DEBUG) SELECT city, AVG(revenue) FROM rides GROUP BY city;
~~~

~~~
                                      info
--------------------------------------------------------------------------------
  Statement diagnostics bundle generated. Download from the Admin UI (Advanced
  Debug -> Statement Diagnostics History), via the direct link below, or using
  the command line.
  Admin UI: http://127.0.0.1:8080
  Direct link: http://127.0.0.1:8080/_admin/v1/stmtbundle/645706706591449089
  Command line: cockroach statement-diag list / download
(6 rows)

Time: 150ms total (execution 150ms / network 0ms)
~~~

Navigating to the URL will automatically download the ZIP file. You can also obtain the bundle by activating [statement diagnostics](ui-statements-page.html#diagnostics) in the DB Console.

## See also

- [`ALTER TABLE`](alter-table.html)
- [`ALTER SEQUENCE`](alter-sequence.html)
- [`BACKUP`](backup.html)
- [`CANCEL JOB`](cancel-job.html)
- [`CREATE DATABASE`](create-database.html)
- [`DROP DATABASE`](drop-database.html)
- [`EXPLAIN`](explain.html)
- [`EXECUTE`](sql-grammar.html#execute_stmt)
- [`IMPORT`](import.html)
- [Indexes](indexes.html)
- [`INSERT`](insert.html)
- [`PAUSE JOB`](pause-job.html)
- [`RESET`](reset-vars.html)
- [`RESTORE`](restore.html)
- [`RESUME JOB`](resume-job.html)
- [`SELECT`](select-clause.html)
- [Selection Queries](selection-queries.html)
- [`SET`](set-vars.html)
- [`SET CLUSTER SETTING`](set-cluster-setting.html)
- [`SHOW COLUMNS`](show-columns.html)
- [`UPDATE`](update.html)
- [`UPSERT`](upsert.html)

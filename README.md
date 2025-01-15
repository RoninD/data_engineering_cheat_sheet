# 1. Data Engineering Cheat Sheet

- [1. Data Engineering Cheat Sheet](#1-data-engineering-cheat-sheet)
- [2. Documentation Links](#2-documentation-links)
- [3. Tutorials](#3-tutorials)
- [4. Cheat Sheets](#4-cheat-sheets)
  - [Pyspark Structured Streaming Triggers](#pyspark-structured-streaming-triggers)
  - [4.1. Databricks Python API vs SQL](#41-databricks-python-api-vs-sql)
  - [4.2. Pyspark Dataframe API vs SQL](#42-pyspark-dataframe-api-vs-sql)
- [5. Snippets](#5-snippets)
  - [5.1. Multihop streaming from Kafka](#51-multihop-streaming-from-kafka)


# 2. Documentation Links

- Kafka
  - [Official Docs](https://kafka.apache.org/documentation/)
- Pyspark
  - [Python API](https://spark.apache.org/docs/3.5.1/api/python/index.html)
  - SQL
    - [Built-in Spark SQL Functions](https://spark.apache.org/docs/latest/api/sql/index.html)
  - Structured Streaming
    - [Python API](https://spark.apache.org/docs/3.5.1/api/python/reference/pyspark.ss/index.html)
    - [Programming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
    - [Kafka Integration](https://spark.apache.org/docs/3.5.1/structured-streaming-kafka-integration.html)
- Delta Lake
    - [Official Documentation](https://docs.delta.io/latest/index.html)
    - [Spark Python API Docs](https://docs.delta.io/latest/api/python/spark/index.html)
    - [Delta-spark API ](https://docs.delta.io/latest/delta-spark.html)
      - [Streaming](https://docs.delta.io/latest/delta-streaming.html)
- Databricks
  - [Official Docs](https://docs.databricks.com/en/index.html)
    - [Delta Live Tables - Python Ref](https://docs.databricks.com/en/delta-live-tables/python-ref.html)

# 3. Tutorials

- [Writing a Kafka Stream to Delta Lake with Spark Structured Streaming](https://delta.io/blog/write-kafka-stream-to-delta-lake/)
- [Mock Kafka Stream](https://rmoff.net/2018/05/10/quick-n-easy-population-of-realistic-test-data-into-kafka/)

# 4. Cheat Sheets

- [Github pyspark-cheatsheet project](https://github.com/cartershanklin/pyspark-cheatsheet)

## [Pyspark Structured Streaming Triggers](https://spark.apache.org/docs/latest/api/python/reference/pyspark.ss/api/pyspark.sql.streaming.DataStreamWriter.trigger.html)

https://medium.com/@kiranvutukuri/trigger-modes-in-apache-spark-structured-streaming-part-6-91107a69de39



Minimal test code

```python
df = spark.readStream.format("kafka")\
    .option("kafka.bootstrap.servers", "localhost:9092")\
    .option("subscribe", "espresso-machine-events")\
    .option("startingOffsets", "earliest")\
    .load()
    .option("maxOffsetsPerTrigger", 5)\
df = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)", "timestamp")

# Define the streaming query and output to the console
query = df.writeStream\
    .outputMode("append")\
    .format("console")\
    .trigger(continuous="1 second")\
    .trigger(once=True)\
    .trigger(availableNow=True)\
    .trigger(processingTime="2 seconds")\
    .option("checkpointLocation", str(data_path / "tmp/checkpoints/processingTime"))\
    .start()
```

**Messages created with 1 sample/s**

| **Trigger Mode**                        | **Checkpoint**                                | **maxOffsetsPerTrigger** | **Output**                                                                                                                                                         |
| --------------------------------------- | --------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `trigger(once=True)`                    | No                                            | Ignored                  | Entire topic content in Batch 0                                                                                                                                    |
| `trigger(once=True)`                    | Yes                                           | Ignored                  | Batch 0: Entire topic at runtime.<br>Batch 1: created after Batch 0.                                                                                               |
| ``trigger(availableNow=True)``          | N/A                                           | 5                        | 5 elements per batch. Terminates with the last available offset at the time of starting the query                                                                  |
| ``trigger(processingTime="5 seconds")`` | N/A                                           | 2                        | Triggers a batch every 5 seconds. Each batch contains only 2 elements.                                                                                             |
| ``trigger(processingTime="5 seconds")`` | N/A                                           | 10                       | Triggers a batch every 5 seconds. Each batch all new available elements, since 10 maxOffsetsPerTrigger accomodate all new messages generated since the last batch. |
| ``trigger(continuous="1 second")``      | Created automatically in /tmp if not provided | Ignored                  | Running continously to process each message with low latency. Decides on its own how many messages to put into each batch                                          |


## 4.1. Databricks Python API vs SQL

| **Data Engineering Task**     | **Python API Example**                                                       | **SQL Example**                                                                    |
| ----------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Load Data**                 | `df = spark.read.csv("data.csv", header=True)`                               | `CREATE OR REPLACE TEMP VIEW data AS SELECT * FROM csv. OPTIONS ('header' 'true')` |
| **Data Cataloging**           | `spark.catalog.listTables("my_database")`                                    | `SHOW TABLES IN my_database`                                                       |
| **Data Lineage Tracking**     | `spark.sql("DESCRIBE HISTORY my_table")`                                     | `DESCRIBE HISTORY my_table`                                                        |
| **Streaming Data Processing** | `df = spark.readStream.format("kafka").option("subscribe", "topic1").load()` | `CREATE STREAMING TABLE data_stream AS SELECT * FROM kafka.\`topic1\``             |


## 4.2. Pyspark Dataframe API vs SQL

To run SQL

```Python
    df.createOrReplaceTempView("data_view")
    result = spark.sql("SELECT col1 FROM data_view")
```

| **Use Case**          | **Python API**                                      | **SQL**                                                                                        |
| --------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Load Data**         | `df = spark.read.csv("data.csv", header=True)`      | `CREATE OR REPLACE TEMP VIEW data AS SELECT * FROM csv.\`data.csv\` OPTIONS ('header' 'true')` |
| **Select Columns**    | `df.select("col1", "col2")`                         | `SELECT col1, col2 FROM data`                                                                  |
| **Filter Rows**       | `df.filter(df["col1"] > 10)`                        | `SELECT * FROM data WHERE col1 > 10`                                                           |
| **Add Column**        | `df.withColumn("new_col", df["col1"] * 2)`          | `SELECT *, col1 * 2 AS new_col FROM data`                                                      |
| **Group & Aggregate** | `df.groupBy("col1").agg({"col2": "sum"})`           | `SELECT col1, SUM(col2) AS total FROM data GROUP BY col1`                                      |
| **Join Tables**       | `df1.join(df2, df1["key"] == df2["key"], "inner")`  | `SELECT * FROM df1 INNER JOIN df2 ON df1.key = df2.key`                                        |
| **Sort Data**         | `df.orderBy(df["col1"].desc())`                     | `SELECT * FROM data ORDER BY col1 DESC`                                                        |
| **Write Data**        | `df.write.format("parquet").save("output.parquet")` | `CREATE TABLE parquet.\`output.parquet\` AS SELECT * FROM data`                                |

Approaches can be mixed via [df.selectExpr()](https://spark.apache.org/docs/3.5.1/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.selectExpr.html)

```df.selectExpr("col1", "col2 AS renamed_col").show()```


# 5. Snippets

## 5.1. Multihop streaming from Kafka

```python
raw_stream = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", topic) \
    .option("startingOffsets", "earliest") \
    .load()


bronze_writer_query = raw_stream.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", checkpointLocation ) \
    .option("path", bronze_path) \
    .trigger(availableNow=True) \
    .start()
```
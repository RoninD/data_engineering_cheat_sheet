# 1. Data Engineering Cheat Sheet

- [1. Data Engineering Cheat Sheet](#1-data-engineering-cheat-sheet)
- [2. Documentation Links](#2-documentation-links)
- [3. Tutorials](#3-tutorials)
- [4. Cheat Sheets](#4-cheat-sheets)
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

# 4. Cheat Sheets

- [Github pyspark-cheatsheet project](https://github.com/cartershanklin/pyspark-cheatsheet)

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
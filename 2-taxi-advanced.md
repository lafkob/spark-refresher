# 🔥 Next-Level PySpark Project: Advanced Analysis of NYC Taxi Data

Goal: Use Spark to explore, transform, and analyze real-world data while focusing on individual core Spark concepts.

---

## ✅ Environment Recap

You're still using:

- Docker container: jupyter/pyspark-notebook:spark-3.5.0

- Dataset: yellow_tripdata_2014-08.parquet

- Notebook: You can continue using the previous notebook or create a new one for this level

If you've torn down your previous container, just run:

```bash
docker-compose up
```

- Open the Jupyter link in from terminal (usually http://127.0.0.1:8888, includes a token) and navigate to work/.

## 🧠 SECTION 1: SparkSession and Lazy Execution

### 🔍 Objective:

Understand how Spark initializes and lazily evaluates transformations.

### 🧪 Code:

```python
from pyspark.sql import SparkSession

# SparkSession is the entry point to Spark
spark = SparkSession.builder.appName("Advanced NYC Taxi Analysis").getOrCreate()

# Load dataset
df = spark.read.parquet("yellow_tripdata_2014-08.parquet")

# Lazy evaluation demo
df_filtered = df.filter(df.passenger_count > 1)
print("This has NOT triggered Spark to do anything yet!")  # Lazy
```

### 🧠 Concept:

Transformations like filter are lazy—they don't execute until an action like `.show()` or `.count()` is called.

## 🔁 SECTION 2: Transformations vs Actions

### 🔍 Objective:

See the difference between transformations (lazy) and actions (eager).

### 🧪 Code:

```python
# Transformation
df_long_trips = df.filter(df.trip_distance > 10)

# Action
df_long_trips.show(5)
print(f"Total long trips: {df_long_trips.count()}")
```

### 💡 Note:

- `.filter()` and `.select()` = transformations

- `.show()`, `.count()`, `.collect()` = actions

## 🧱 SECTION 3: DataFrames vs RDDs

### 🔍 Objective:

Compare ease-of-use and performance of DataFrames vs RDDs.

### 🧪 Code:

```python
# Convert to RDD
rdd = df.rdd
print(rdd.take(2))

# Map and reduce example
total_fare = rdd.map(lambda row: row.total_amount).filter(lambda x: x is not None).sum()
print(f"Total fare collected: {total_fare}")
```

### 💡 Note:

- Use DataFrames for structured data and performance.

- Use RDDs for lower-level control or unstructured data.

## 📊 SECTION 4: Spark SQL

### 🔍 Objective:

Use SQL to do expressive, familiar queries.

### 🧪 Code:

```python
import pyspark.sql.functions as f
print("Using DataFrame API:")
df.where(df.trip_distance > 1) \
.groupBy(df.passenger_count) \
.agg(f.round(f.avg("trip_distance"), 2).alias("avg_distance")) \
.sort(f.asc("passenger_count")) \
.show()

print("Using SparkSQL:")
df.createOrReplaceTempView("taxi")
spark.sql("""
SELECT passenger_count, ROUND(AVG(trip_distance), 2) as avg_distance
FROM taxi
WHERE trip_distance > 1
GROUP BY passenger_count
ORDER BY passenger_count
""").show()
```

### 💡 Notes:

✅ The DataFrame API:

- aka Domain-Specific Language or Functional API
- Type-safe (to some extent — less prone to SQL typos).
- More programmatic and dynamic (e.g., easier to construct pipelines).
- Often easier to debug with IDE autocompletion.

✅ The Spark SQL API (or Spark SQL interface):

- Leverages the Catalyst query optimizer
- More familiar to users with SQL backgrounds.
- Great for ad-hoc querying and notebooks.
- Useful when interacting with BI tools like Tableau or when running on Databricks.

## 🧠 SECTION 5: Caching and Persistence

### 🔍 Objective:

Improve performance when reusing DataFrames.

### 🧪 Code:

```python
df.cache()  # Keeps data in memory across actions
df.count()  # Triggers computation and caching

# Try running multiple actions
df.select(f.avg("trip_distance")).show()
df.select(f.max("trip_distance")).show()
```

### 💡 Note:

Use `cache()` or `persist()` when you'll reuse a DataFrame multiple times.

## 🧮 SECTION 6: Partitioning and Repartitioning

### 🔍 Objective:

Understand how data is distributed and how to control it.

### 🧪 Code:

```python
print(f"Initial partitions: {df.rdd.getNumPartitions()}")

# Repartition
df_repart = df.repartition(10)
print(f"After repartition: {df_repart.rdd.getNumPartitions()}")

# Coalesce to reduce partitions
df_small = df_repart.coalesce(2)
print(f"After coalesce: {df_small.rdd.getNumPartitions()}")
```

### 💡 Note:

- `repartition(n)`: full shuffle, useful for balancing

- `coalesce(n)`: no shuffle, best for downsizing before writing

## 📈 SECTION 7: Aggregations and Grouping

### 🔍 Objective:

Perform meaningful aggregations across dimensions.

### 🧪 Code:

```python
from pyspark.sql.functions import avg, max, min

# Group and aggregate
df.groupBy("passenger_count").agg(
    avg("trip_distance").alias("avg_distance"),
    max("trip_distance").alias("max_distance"),
    min("trip_distance").alias("min_distance")
).orderBy("passenger_count").show()
```

### 💡 Note:

Use `groupBy().agg()` for powerful aggregations on grouped data.

## 🧹 SECTION 8: Cleanup and Shutdown

```bash
docker-compose down -v
```

## 🎯 Bonus Challenges (Optional)

Try enhancing this notebook by:

- Writing results to Parquet or JSON

- Creating new calculated columns (e.g., `withColumn("fare_per_mile", col("total_amount") / col("trip_distance"))`)

- Filtering and aggregating based on `payment_type`

- Plotting aggregated results using `matplotlib` or `plotly`

## 🎁 Next: 3-taxi-moar-advanced

In the next level you'll see:

- Structured Streaming (e.g., live Kafka or file streaming)

- MLlib (machine learning with Spark)

- Delta Lake / Lakehouse style processing

- ETL pipelines with partitioned output on S3 or local FS

---

Happy Spark Hacking! 🔥

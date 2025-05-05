# 🚀 Spark Basics Starter Project with PySpark

A beginner-friendly, containerized Apache Spark project using **Python (PySpark)** to analyze NYC Taxi data. Covers core concepts: SparkSession, DataFrames, Transformations, Actions, Spark SQL, Caching, and Partitioning.

---

## 🧱 Environment Setup (macOS + Docker)

### 1. 🐳 Install Docker

This container we use includes Spark, Python, Jupyter, and useful packages pre-installed.

### 2. 📂 Create working directory

```bash
mkdir work
```

### 3. 🚀 Launch the container

```bash
docker-compose up
```

- Open the Jupyter link in from terminal (usually http://127.0.0.1:8888, includes a token) and navigate to work/.

## 📊 Dataset

Download samples of NYC Yellow Taxi data:

```bash
cd work
curl -O https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2014-08.parquet
curl -O https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2014-09.parquet
```

Source: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page (Year: 2014)
Note: This used to be available in CSV but NYC up-converted everything into parquet

## 📘 Spark Notebook: Core Concepts

Inside the Jupyter UI, create a new notebook and paste the following code:

```python
from pyspark.sql import SparkSession

# Initialize Spark
spark = SparkSession.builder \
    .appName("NYC Taxi Analysis") \
    .getOrCreate()

# Load parquet as DataFrame
df = spark.read.parquet("yellow_tripdata_2014-08.parquet")

# Show schema
df.printSchema()

# Show sample records
df.show(5)

# Basic transformations
df_filtered = df.filter(df.passenger_count > 1)
df_grouped = df_filtered.groupBy("passenger_count").count()

# Show results
df_grouped.show()

# Calculate some distributions for trip distance
from pyspark.sql.functions import when, lit, col

df_ranges = df.withColumn("trip_range",
                         when((col("trip_distance") <= 5), lit("<5")).otherwise(
                         when((col("trip_distance") > 5) & (col("trip_distance") <= 25), lit("5-25")).otherwise(
                         when((col("trip_distance") > 25) & (col("trip_distance") <= 50), lit("26-50")).otherwise(
                         when((col("trip_distance") > 50) & (col("trip_distance") <= 100), lit("51-100")).otherwise(
                         when((col("trip_distance") > 100) & (col("trip_distance") <= 250), lit("101-250")).otherwise(
                         when((col("trip_distance") > 250) & (col("trip_distance") <= 500), lit("251-500")).otherwise(
                         when((col("trip_distance") > 500) & (col("trip_distance") <= 1000), lit("501-1000")).otherwise(
                         when((col("trip_distance") > 1000) & (col("trip_distance") <= 5000), lit("1001-5000")).otherwise(
                         when((col("trip_distance") > 5000) & (col("trip_distance") <= 10000), lit("5001-10000"))
                         .otherwise(lit("other")))))))))))
df_ranges.groupBy("trip_range").count().show()

# Use SQL on the DataFrame
df.createOrReplaceTempView("trips")
spark.sql("""
SELECT passenger_count, AVG(trip_distance) as avg_distance
FROM trips
GROUP BY passenger_count
""").show()

# Cache the DataFrame to memory
df.cache()
df.count()  # Action to trigger cache

# Check partitions
print(f"Number of partitions: {df.rdd.getNumPartitions()}")

# Repartitioning
df_repartitioned = df.repartition(10)
print(f"Repartitioned to: {df_repartitioned.rdd.getNumPartitions()}")
```

## ✅ Concepts Covered

| Concept         | Description                                 |
| --------------- | ------------------------------------------- |
| SparkSession    | Entry point to Spark                        |
| DataFrames      | High-level abstraction over structured data |
| Transformations | filter(), groupBy() are lazy                |
| Actions         | show(), count() trigger execution           |
| Spark SQL       | Query with familiar SQL syntax              |
| Caching         | Reuse data efficiently in memory            |
| Partitioning    | Control parallelism and data distribution   |

## 🧹 Tear Down

Stop and remove all Docker containers and volumes:

```bash
docker-compose down -v
```

## 🧭 Next Steps

Try expanding this notebook by:

- Adding calculated columns (withColumn)

- Handling missing data

- Writing results to Parquet or JSON

- Creating a dashboard in Jupyter using matplotlib or plotly

---

Happy Spark Hacking! 🔥

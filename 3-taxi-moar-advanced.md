# 🚀 What's Next: Advanced Spark Project with PySpark

This project helps you explore production-level capabilities of Apache Spark, including:

- Real-time **Structured Streaming**
- Machine learning with **MLlib**
- Using **Delta Lake** for ACID-compliant data lakes
- Creating modular **ETL pipelines**

Your environment remains containerized via Docker and runs locally using Jupyter Notebooks.

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

## 🔁 SECTION 1: Structured Streaming with PySpark

NOTE: WIP

### 🎯 Objective:

Ingest and process data in real time from a directory using Structured Streaming.

### ✅ Setup

In your terminal:

```bash
mkdir -p work/streaming_input
```

Drop new Parquet into `work/streaming_input/` as if they're arriving in real time.

Source: Yellow taxis on https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page (already using August 2014)

(Optional) Curl them right into place, one at a time:

```bash
cd work/streaming_input
curl -O https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2014-09.parquet
```

### 🧠 Code

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("StructuredStreamingDemo") \
    .getOrCreate()

# TODO: needs update for parquet
# Watch a directory for new CSV files
schema = "vendor_id INT, passenger_count INT, trip_distance DOUBLE, total_amount DOUBLE"

streaming_df = spark.readStream \
    .schema(schema) \
    .option("maxFilesPerTrigger", 1) \
    .csv("streaming_input")

# Simple transformation
agg = streaming_df.groupBy("passenger_count").count()

# Output to console
query = agg.writeStream \
    .format("console") \
    .outputMode("complete") \
    .start()

query.awaitTermination()
```

✅ Explanation: Structured Streaming treats a directory as an unbounded stream source and uses continuous execution under the hood. You can simulate streaming by dropping files into `streaming_input/`.

## 🤖 SECTION 2: MLlib — Basic Classification

### 🎯 Objective:

Train a Logistic Regression model to classify trips as short or long based on features.

### 🧠 Code

```python
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.classification import LogisticRegression
from pyspark.sql.functions import when

df = spark.read.parquet("yellow_tripdata_2014-08.parquet")

# Label: 1 if trip_distance > 5, else 0
df = df.withColumn("label", when(df.trip_distance > 5, 1).otherwise(0))

# Feature vector
assembler = VectorAssembler(
    inputCols=["passenger_count", "trip_distance", "total_amount"],
    outputCol="features"
)

data = assembler.transform(df).select("label", "features")

# Split and train
train, test = data.randomSplit([0.7, 0.3])
lr = LogisticRegression()
model = lr.fit(train)

# Evaluate
predictions = model.transform(test)
predictions.select("label", "prediction", "probability").show(5)
```

✅ Explanation: MLlib provides scalable ML pipelines. Here, you use `VectorAssembler` to prepare features and apply logistic regression using Spark's DataFrame-based API.

## 💾 SECTION 3: Delta Lake for ACID Tables

### 🎯 Objective:

Use Delta Lake to store data with ACID guarantees, schema enforcement, and time travel.

```python
pip install delta-spark
```

> ⚠️ You should restart the python kernel for your notebook at this point. Continue with the following in a new cell(s).

```python
from delta import configure_spark_with_delta_pip
from pyspark.sql import SparkSession

builder = SparkSession.builder \
    .appName("DeltaLakeExample") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")

spark = configure_spark_with_delta_pip(builder).getOrCreate()

df = spark.read.parquet("yellow_tripdata_2014-08.parquet")

# Write to a delta table
df.write.format("delta").mode("overwrite").save("delta/trips")

# Read it back
delta_df = spark.read.format("delta").load("delta/trips")
delta_df.printSchema()

# Overwrite the table with the following month's data
df = spark.read.parquet("yellow_tripdata_2014-09.parquet")
# overwriteSchema avoids typing issues (08 vs 09 has int vs double column types)
df.write.format("delta").mode("overwrite").option("overwriteSchema", "true").save("delta/trips")

# Query version history (time travel), notice the sample date
spark.read.format("delta").option("versionAsOf", 0).load("delta/trips").select("tpep_pickup_datetime").show(1)
spark.read.format("delta").option("versionAsOf", 1).load("delta/trips").select("tpep_pickup_datetime").show(1)
```

✅ Explanation: Delta Lake adds powerful transactional and schema evolution capabilities to Spark tables, critical for production-grade pipelines.

## 🔧 SECTION 4: Modular ETL Pipeline Design

### 🎯 Objective:

Break your logic into functions and stages for maintainable, testable pipelines.

```python
def extract(path):
    return df = spark.read.parquet(path)

def transform(df):
    return df.withColumn("fare_per_mile", df.total_amount / df.trip_distance)

def load(df, path="output/trips_cleaned"):
    df.write.mode("overwrite").parquet(path)

# Orchestrate pipeline
df_raw = extract("yellow_tripdata_2014-08.parquet")
df_transformed = transform(df_raw)
load(df_transformed)
```

✅ Explanation: Modular ETL design helps isolate responsibilities (Extract, Transform, Load). You can test these functions individually and reuse logic across pipelines.

## 🔬 Bonus: Combine Streaming + ML

Train a model using batch data, then apply it to streaming input:

```python
# After training model...
streaming_features = assembler.transform(streaming_df).select("features")

# Predict live!
preds = model.transform(streaming_features)

query = preds.writeStream \
    .format("console") \
    .outputMode("append") \
    .start()

query.awaitTermination()
```

## ✅ Final Cleanup

When you're done:

```bash
docker-compose down -v
```

## 🧭 Next Steps

TODO: create next level for the following

Consider exploring:

- 💡 Feature engineering with OneHotEncoder, StringIndexer

- ⏱ Windowed aggregations in streaming

- 🔄 Writing to Delta + versioning

- 🔐 Adding unit tests to your ETL functions

---

Happy Spark Hacking! 🔥

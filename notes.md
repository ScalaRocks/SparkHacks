---
title:  "CRT020 Certification Feedback & Tips!"
excerpt: "In this post I'm sharing my feedback and some preparation tips on the CRT020 - Databricks Certified Associate Developer for Apache Spark 2.4 certification exam I took recently."
classes: wide
categories: [certifications]
tags: [crt020, databricks, spark]
---

In this post I'm sharing my feedback and some preparation tips on the *[CRT020](https://academy.databricks.com/exam/crt020-scala) - Databricks Certified Associate Developer for Apache Spark 2.4 with Scala 2.11* certification exam I took recently.

I will not leak any particular question since I'm not allowed to (and because I don't remember as well :)), but I hope to provide you some important insights that will help you to pass on the exam.

# Exam

The exam has a duration of approximately 180 minutes and mine had 20 multiple-choice questions and 19 coding challenges. Since the exam is done within the Databricks platform, it's important that you become familiar with their environment.

You will have access to the [Databricks](https://docs.databricks.com/index.html), [Spark](https://spark.apache.org/docs/2.4.0/) and [Scala](https://docs.scala-lang.org/) (or Python) documentations, but remember that you only have 180 minutes, thus knowing beforehand where each *class / function* resides it's crucial to finish the assessment on time.

The assessment is composed of 4 main notebooks:
- **Getting started** - contains some basic examination information
- **Tutorial** - briefly shows how to use the Databricks platform (how to attach a cluster, run a cell on a notebook, run all the notebook cells, etc.)
- **Multiple-choice questions** - page with 20 links pointing to each of the multiple-choice question. Each question is a separate notebook.
- **Coding challenges** - same as previous

 Multiple-choice questions range from very easy (e.g which of the following is a *narrow transformation*) to quite hard (e.g about Catalyst optimizer or Tungsten row format), but I would say the most part of questions are medium level ones. 

Some questions may cover:
- identify _actions_ and _transformations_
- differentiate between _narrow_ and _wide transformations_
- know how _caching_ works and Spark's LRU partition eviction
- know how to increase or decrease the partition number, without or without shuffling
- reason about the pros and cons, GC, execution time and OOM errors when there is data skew in multiple setups, e.g it is better to have a setup with 1 executor with 100GB of RAM and 20 cores or 10 executors with 10GB of RAM and 2 cores each, etc.
- know how shuffle works
- know what is a *DAG* and Spark's units of execution: _Job_, _Stage_ and _Task_
- know the relation between the RDD partition count and the number of spawned _Tasks_ 


All the coding challenges notebooks were composed by 4 or 5 cells in total:
- **Description**
- **Environment setup cell** - may execute some bash script that prepares the environment
- **Data format cell** - just a cell with `display(df)` so that you can check the structure of the DataFrame to be transformed (present in some exercises)
- **Coding cell** - skeleton of the to-be-implemented function that usually receives a DataFrame and may return one or multiple results, e.g DataFrame or a tuple with a DataFrame and its StructType schema.
- **Result cell** - surprisingly you can check if the previously implemented function passes the basic tests, resulting in a table with _PASSED_ or _FAILED_ column value. This cell has a description with something like "Run this to check if you are on track", however there are no guarantees that the same tests will be used for the final evaluation.

Since the *Environment setup cell* may take some time to execute, I would recommend to run this cell before reading the _Description_.

Be careful with the imports, since none of them are included! Just memorize all the important imports like `import org.apache.spark.sql.functions._` or `import spark.implicits._` over the SparkSession.

Unfortunately the Databricks platform was getting slower with each notebook I opened and I was afraid of refreshing or closing the browser with the risk of somehow ending the assessment session. It got to the point of writing the whole `import org.apache.spark.sql.functions._` and waiting some additional 5 seconds until the written text appeared. Almost after implementing all the coding challenges I finally refreshed the tab, cause it was just impossible to continue, and the platform became normal again.

Also the copy paste was not working. I was unable to paste anything from the documentation tabs.

I waited approximately 3 business days until I received the grading result. Expect at least 72 hours considering business days.

# Cheat Sheet 

This section contains all the topics included in the [certification page](https://academy.databricks.com/exam/crt020-scala). As you can see I've already completed some code blocks and others are marked with `// TODO`. You can use this section as a studying guide / cheat sheet. Feel free to contribute to the remaining topics :)

The dataset is available [here](https://github.com/databricks/Spark-The-Definitive-Guide/tree/master/data). 

Good luck on your preparation!!!

## Spark Architecture Components

Candidates are expected to be familiar with the following architectural components and their relationship to each other:
- Driver 
- Executor
- Cores/Slots/Threads
- Partitions

## Spark Execution

Candidates are expected to be familiar with Spark’s execution model and the breakdown between the different elements:
- Jobs
- Stages 
- Tasks

## Spark Concepts

Candidates are expected to be familiar with the following concepts:
- Caching
- Shuffling
- Partitioning
- Wide vs. Narrow Transformations
- DataFrame Transformations vs. Actions vs. Operations
- High-level Cluster Configuration

## DataFrames API

### SparkContext
Candidates are expected to know **how to use the SparkContext** to control basic configuration settings such as `spark.sql.shuffle.partitions`.

```scala
// TODO
```

### SparkSession
Candidates are expected to know how to:
- Create a DataFrame/Dataset from a collection (e.g. list or set)

```scala
import org.apache.spark.sql._ 
import org.apache.spark.sql.types._

val schema = StructType(
    StructField("id", StringType) ::
    StructField("value", IntegerType) ::
    Nil
)

val data = Seq(
    Row("id1", 1),
    Row("id2", 2),
    Row("id3", 3)
)

val df = spark.createDataFrame(
     spark.sparkContext.parallelize(data),
     schema
)

// Or using .toDF(...)
import spark.implicits._

val df2 = Seq(
    ("id1", 1),
    ("id2", 2),
    ("id3", 3)
).toDF("id", "value")

// Or using Datasets
case class Record(id: String, value: Int)

val ds = Seq(
    Record("id1", 1),
    Record("id2", 2),
    Record("id3", 3)
).toDS()
```

- Create a DataFrame for a range of numbers

```scala
val ds = spark.range(100).toDF("number")
```

- Access the DataFrameReaders

```scala
// See https://spark.apache.org/docs/2.4.0/api/scala/index.html#org.apache.spark.sql.DataFrameReader

spark.read.csv(...)
spark.read.jdbc(...)
spark.read.json(...)
spark.read.orc(...)
spark.read.parquet(...)
spark.read.table(...)
spark.read.text(...)
spark.read.textFile(...)
```

- Register User Defined Functions - UDFs

```scala
// See https://docs.databricks.com/spark/latest/spark-sql/udf-scala.html

// Use in Spark SQL
val squared = (s: Long) => s * s

spark.udf.register("square", squared)

spark.range(1, 20).registerTempTable("test")

val df = spark.sql("select id, square(id) as id_squared from test")
df.show()

// Use with DataFrames
import org.apache.spark.sql.functions.{col, udf}

val squared = udf((s: Long) => s * s)

val df = spark.range(1, 20).select(col("id"), squared(col("id")) as "id_squared")
df.show()
```

### DataFrameReader
Candidates are expected to know how to:

- **Read data for the "core" data formats** (CSV, JSON, JDBC, ORC, Parquet, text and tables)

```scala
// See
// https://spark.apache.org/docs/2.4.0/api/scala/index.html#org.apache.spark.sql.DataFrameReader
// https://spark.apache.org/docs/2.4.0/sql-data-sources-load-save-functions.html

// CSV
val csvDF = spark.read
    .option("header", true)
    .option("inferSchema", true)
    .csv("data/flight-data/csv/2015-summary.csv")
csvDF.show()

// CSV alternative way
val csvDS = spark.createDataset(Seq(
    """DEST_COUNTRY_NAME,ORIGIN_COUNTRY_NAME,count""",
    """United States,Romania,15""",
    """United States,Croatia,1"""
))
val csvDF = spark.read
    .option("header", true)
    .csv(csvDS)
csvDF.show()

// JSON
val jsonDF = spark.read.json("data/flight-data/json/2015-summary.json")
jsonDF.show()

// JSON alternative way
val jsonDS = spark.createDataset(Seq(
    """{"ORIGIN_COUNTRY_NAME":"Romania","DEST_COUNTRY_NAME":"United States","count":15}""",
    """{"ORIGIN_COUNTRY_NAME":"Croatia","DEST_COUNTRY_NAME":"United States","count":1}"""
))
val jsonDF = spark.read.json(jsonDS)
jsonDF.show()

// JSON handle malformed records
val jsonDS = spark.createDataset(Seq(
    """{"id":1, "name":"bob"}""",
    """{"id":2, "name":"tom"}""",
    """{"id":3, "name":broken}""",
    """{"id":4, "name":"derrick"}"""
))
val jsonDF = spark.read
    .schema("id INT, name STRING, _corrupt_record STRING")
    .json(jsonDS)
jsonDF.show()

/*
Default read mode "Sets all fields to null when it encounters a corrupted record and places 
all corrupted records in a string column called _corrupt_record"

+----+-------+--------------------+
|  id|   name|     _corrupt_record|
+----+-------+--------------------+
|   1|    bob|                null|
|   2|    tom|                null|
|null|   null|{"id":3, "name":b...|
|   4|derrick|                null|
+----+-------+--------------------+

Other read modes:
    dropMalformed  => Drops the row that contains malformed records
    failFast  =>  Fails immediately upon encountering malformed records
*/

val jsonDF = spark.read
    .schema("id INT, name STRING, _corrupt_record STRING")
    .option("mode", "dropMalformed")
    .json(jsonDS)
jsonDF.show()


// JDBC
val jdbcDF = spark.read
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .jdbc()


// JDBC alternative way
val connectionProperties = new Properties()
connectionProperties.put("user", "username")
connectionProperties.put("password", "password")
connectionProperties.put("customSchema", "id DECIMAL(38, 0), name STRING")

val jdbcDF = spark.read
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties) 

// Parquet
val parquetDF = spark.read.parquet("data/flight-data/parquet/2010-summary.parquet")
parquetDF.show()

// Text
val textDF = spark.read.text("data/flight-data/csv/2015-summary.csv") // Returns a DataFrame
textDF.show()

val textDS = spark.read.textFile("data/flight-data/csv/2015-summary.csv")    // Returns a Dataset
textDS.show()

// Table
val tableDF = spark.read.table("some_table")
```
- How to **configure options** for specific formats

```scala
// See above
```

- How to **read data from non-core formats** using format() and load()

```scala
val csvDF = spark.read.format("csv")
    .option("header", true)
    .option("inferSchema", true)
    .load("data/2015-summary.csv")
csvDF.show()
```

- How to **specify a DDL-formatted schema**

```scala
spark.read.schema("a INT, b STRING, c DOUBLE").csv("...")
```

- How to **construct and specify a schema** using the StructType classes

```scala
val schema = StructType(
    StructField("id", StringType) ::
    StructField("value", IntegerType) ::
    Nil
)
```

### DataFrameWriter
Candidates are expected to know how to:

- **Write data to the "core" data formats** (csv, json, jdbc, orc, parquet, text and tables)

```scala
// See
// https://spark.apache.org/docs/2.4.0/sql-data-sources-load-save-functions.html

// JDBC
jdbcDF.write
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .jdbc()

// JDBC alternative way
val connectionProperties = new Properties()
connectionProperties.put("user", "username")
connectionProperties.put("password", "password")

jdbcDF2.write
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)


// TODO write to other data formats
```
- **Overwriting** existing files

```scala
val textDF = spark.read.text("data/flight-data/csv/2015-summary.csv")
textDF.write.mode("overwrite").save("/tmp/deleteme")

// Other options are:
//    overwrite: overwrite the existing data.
//    append: append the data.
//    ignore: ignore the operation (i.e. no-op).
//    error or errorifexists: default option, throw an exception at runtime.
```

- How to **configure options** for specific formats

```scala
// See above
```
- How to **write a data source to 1 single file or N separate files**

```scala
val df = spark.read.parquet("data/flight-data/parquet/2010-summary.parquet")

// Creates a single /tmp/output/part-00000... file
// Should only be used if the data fits in a single partition 
df.repartition(1).write.save("/tmp/output")
```

- How to **write partitioned data**

```scala
val df = spark.read.parquet("data/flight-data/parquet/2010-summary.parquet")

df.write.partitionBy("ORIGIN_COUNTRY_NAME").save("/tmp/output")
```

- How to **bucket data** by a given set of columns

```scala
// See https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-bucketing.html

val df = spark.read.parquet("data/flight-data/parquet/2010-summary.parquet")

df.write
    .bucketBy(6, "ORIGIN_COUNTRY_NAME")
    .sortBy("DEST_COUNTRY_NAME")
    .saveAsTable("test")
```

### DataFrame
- Have a **working understanding of every action** such as take(), collect(), and foreach()

```scala
// Actions
def collect(): Array[T]
def collectAsList(): List[T] 
def count(): Long 
def first(): T 
def foreach(f: (T) ⇒ Unit): Unit 
def foreachPartition(f: (Iterator[T]) ⇒ Unit): Unit 
def head(): T 
def head(n: Int): Array[T] 
def reduce(func: (T, T) ⇒ T): T 
def take(n: Int): Array[T] 
def takeAsList(n: Int): List[T] 
def toLocalIterator(): Iterator[T] 
```

- Have a **working understanding of the various transformations** and how they work such as producing a distinct set, filtering data, repartitioning and coalescing, performing joins and unions as well as producing aggregates.

```scala

// Typed transformations
def alias(alias: String): Dataset[T]
def as(alias: Symbol): Dataset[T] 
def coalesce(numPartitions: Int): Dataset[T] 
def distinct(): Dataset[T]
def dropDuplicates(): Dataset[T] 
def dropDuplicates(col1: String, cols: String*): Dataset[T] 
def except(other: Dataset[T]): Dataset[T] 
def exceptAll(other: Dataset[T]): Dataset[T]
def filter(func: (T) ⇒ Boolean): Dataset[T]
def filter(conditionExpr: String): Dataset[T]
def filter(condition: Column): Dataset[T]
def flatMap[U](func: (T) ⇒ TraversableOnce[U])(implicit arg0: Encoder[U]): Dataset[U]
def groupByKey[K](func: (T) ⇒ K)(implicit arg0: Encoder[K]): KeyValueGroupedDataset[K, T] 
def intersect(other: Dataset[T]): Dataset[T]
def intersectAll(other: Dataset[T]): Dataset[T]
def joinWith[U](other: Dataset[U], condition: Column): Dataset[(T, U)]
def joinWith[U](other: Dataset[U], condition: Column, joinType: String): Dataset[(T, U)]
def limit(n: Int): Dataset[T]
def map[U](func: (T) ⇒ U)(implicit arg0: Encoder[U]): Dataset[U]
def mapPartitions[U](func: (Iterator[T]) ⇒ Iterator[U])(implicit arg0: Encoder[U]): Dataset[U]
def orderBy(sortExprs: Column*): Dataset[T]
def orderBy(sortCol: String, sortCols: String*): Dataset[T]
def randomSplit(weights: Array[Double]): Array[Dataset[T]]
def randomSplit(weights: Array[Double], seed: Long): Array[Dataset[T]]
def randomSplitAsList(weights: Array[Double], seed: Long): List[Dataset[T]]
def repartition(partitionExprs: Column*): Dataset[T]
def repartition(numPartitions: Int, partitionExprs: Column*): Dataset[T]
def repartition(numPartitions: Int): Dataset[T]
def repartitionByRange(partitionExprs: Column*): Dataset[T]
def repartitionByRange(numPartitions: Int, partitionExprs: Column*): Dataset[T] 

def sort(sortExprs: Column*): Dataset[T]
def sort(sortCol: String, sortCols: String*): Dataset[T]
def sortWithinPartitions(sortExprs: Column*): Dataset[T]
def sortWithinPartitions(sortCol: String, sortCols: String*): Dataset[T] 
def transform[U](t: (Dataset[T]) ⇒ Dataset[U]): Dataset[U] 
def union(other: Dataset[T]): Dataset[T]
def unionByName(other: Dataset[T]): Dataset[T]
def where(conditionExpr: String): Dataset[T]
def where(condition: Column): Dataset[T]
```

- Know **how to cache data**, specifically to disk, memory or both

```scala
spark.catalog.cacheTable("tableName") 

// Persist this Dataset/DataFrame with the default storage level (MEMORY_AND_DISK)
ds.cache()
ds.count()  // Because cache are lazily evaluated

import org.apache.spark.storage.StorageLevel

ds.persist(StorageLevel.MEMORY_AND_DISK_SER)
ds.count()  // Because cache are lazily evaluated

/*
Possible Storage Formats:
    NONE
    DISK_ONLY
    DISK_ONLY_2
    MEMORY_ONLY
    MEMORY_ONLY_2
    MEMORY_ONLY_SER
    MEMORY_ONLY_SER_2
    MEMORY_AND_DISK
    MEMORY_AND_DISK_2
    MEMORY_AND_DISK_SER
    MEMORY_AND_DISK_SER_2
    OFF_HEAP
*/
```
- Know **how to uncache** previously cached data

```scala
spark.catalog.uncacheTable("tableName")

ds.unpersist()
```

- Converting a **DataFrame to a global or temp view**

```scala
// Global temporary view is cross-session. 
// Its lifetime is the lifetime of the Spark application, 
// i.e. it will be automatically dropped when the application terminates
df1.createGlobalTempView("temp1")

// Local temporary view is session-scoped.
// Its lifetime is the lifetime of the session that created it, 
// i.e. it will be automatically dropped when the session terminates
df2.createTempView("temp2")
```

- Applying **hints**

```scala
import org.apache.spark.sql.functions.broadcast

val df1 = spark.table("table1")
val df2 = spark.table("table2")

// Broadcast Hash Join
df1.join(broadcast(df2), "key")

// Or
df1.join(df2.hint("broadcast"))
```

## Row & Column
- Candidates are expected to know **how to work with row and columns** to successfully extract data from a DataFrame

## Spark SQL Functions
When instructed what to do, candidates are expected to be able to employ the multitude of Spark SQL functions. Examples include, but are not limited to:
- **Aggregate functions**: getting the first or last item from an array or computing the min and max values of a column.

```scala
// TODO
```

- **Collection functions**: testing if an array contains a value, exploding or flattening data.

```scala
// TODO
```

- **Date time functions**: parsing strings into timestamps or formatting timestamps into strings

```scala
// TODO
```

- **Math functions**: computing the cosign, floor or log of a number

```scala
// See:
// https://spark.apache.org/docs/2.4.0/api/scala/index.html#org.apache.spark.sql.functions$
// https://spark.apache.org/docs/2.4.0/api/sql/

// See rounding modes: https://docs.oracle.com/javase/7/docs/api/java/math/RoundingMode.html
def bround(e: Column, scale: Int): Column // Uses HALF_EVEN, which rounds up if the number is odd and down if even
def round(e: Column, scale: Int): Column // Uses HALF_UP
def ceil(columnName: String): Column
def floor(columnName: String): Column 
```

- **Misc functions**: converting a value to crc32, md5, sha1 or sha2

```scala
// TODO
```

- **Non-aggregate functions**: creating an array, testing if a column is 
null, not-null, nan, etc

```scala
// TODO
```

- **Sorting functions**: sorting data in descending order, ascending order, and sorting with proper null handling

```scala
import spark.implicits._
import org.apache.spark.sql.functions._

val df = Seq[(Int, Integer)](
    (1, 100),
    (2, 200),
    (3, null),
    (4, 50)
).toDF("row", "value")

df.sort($"value").show()    // alias for orderBy
df.orderBy($"value").show()
df.orderBy($"value".asc_nulls_first).show()
df.orderBy($"value".desc_nulls_first).show()
df.orderBy($"value".asc_nulls_last).show()
df.orderBy($"value".desc_nulls_last).show()

df.sortWithinPartitions($"value".desc_nulls_last).show()
```

- **String functions**: applying a provided regular expression, trimming string and extracting substrings.

```scala
// TODO
```

- **UDF functions**: employing a UDF function.

```scala
// See:
// https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-udfs.html
// https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-udfs-blackbox.html

// It is always better to perform the logic using the existing functions on Columns
val dateDiff: (Column, Column) => Column = (x, y) => { datediff(to_date(y), to_date(x)) }
df = df.withColumn("date_diff", dateDiff(col("start_date"), col("end_date")))

val df = Seq(
    (0, "hello"), 
    (1, "world")
).toDF("id", "text")

// Define a UDF that wraps the upper Scala function
import org.apache.spark.sql.functions.udf
val upperUDF = udf((x: String) => if(x != null) x.toUpperCase else null)

// Apply the UDF to change the source dataset
df.withColumn("upper", upperUDF('text)).show()
+---+-----+-----+
| id| text|upper|
+---+-----+-----+
|  0|hello|HELLO|
|  1|world|WORLD|
+---+-----+-----+

// Alternativaly register de UDFs to use in SQL-based query expressions
spark.udf.register("myUpper", (x: String) => if(x != null) x.toUpperCase else null)

// Check all the standard and user-defined functions with listFunctions over the Catalog
spark.catalog.listFunctions.filter('name like "%upper%").show(false)
+-----+--------+-----------+-----------------------------------------------+-----------+
|name |database|description|className                                      |isTemporary|
+-----+--------+-----------+-----------------------------------------------+-----------+
|upper|null    |null       |org.apache.spark.sql.catalyst.expressions.Upper|true       |
+-----+--------+-----------+-----------------------------------------------+-----------+

// Register the DF to be used in plain SQL
df.createTempView("tempTable")

// Use the registered UDF
spark.sql("SELECT id, text, myUpper(text) as upper FROM tempTable").show()
```

- **Window functions**: computing the rank or dense rank.

```scala
// TODO
```

### Other

- **Working with missing data in DataFrames types**: by using the [DataFrameNaFunctions](https://spark.apache.org/docs/2.4.0/api/scala/index.html#org.apache.spark.sql.DataFrameNaFunctions) class

```scala
import spark.implicits._

val df = Seq(
    (1, null), 
    (2, "lol")
).toDF("row","value")

df.show()
+---+-----+
|row|value|
+---+-----+
|  1| null|
|  2|  lol|
+---+-----+

// Drop rows with null values
df.na.drop(Seq("value")).show()
+---+-----+
|row|value|
+---+-----+
|  2|  lol|
+---+-----+

// Replace null values from all string type columns
df.na.fill("lolada").show()
+---+------+
|row| value|
+---+------+
|  1|lolada|
|  2|   lol|
+---+------+

// Replace null values in specified columns
df.na.fill("asd", Seq("value")).show()
+---+-----+
|row|value|
+---+-----+
|  1|  asd|
|  2|  lol|
+---+-----+

// Replace null values in specified columns
df.na.fill(Map("value" -> "lolada")).show()
+---+------+
|row| value|
+---+------+
|  1|lolada|
|  2|   lol|
+---+------+

// Replace values in specified columns
df.na.replace(Seq("value"), Map("lol" -> "lolada")).show()
+---+------+
|row| value|
+---+------+
|  1|  null|
|  2|lolada|
+---+------+
```

- File system utility **dbutils**: see [Databricks Utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html)

```scala
dbutils.fs.rm("/tmp/dataframe_sample.csv", true)
dbutils.fs.put("/tmp/dataframe_sample.csv", """
id|end_date|start_date|location
1|2015-10-14 00:00:00|2015-09-14 00:00:00|CA-SF
2|2015-10-15 01:00:20|2015-08-14 00:00:00|CA-SD
3|2015-10-16 02:30:00|2015-01-14 00:00:00|NY-NY
4|2015-10-17 03:00:20|2015-02-14 00:00:00|NY-NY
5|2015-10-18 04:30:00|2014-04-14 00:00:00|CA-LA
""", true)

// List all Databricks datasets
display(dbutils.fs.ls("/databricks-datasets"))
```
- **Convert DataFrame to JSON** using the Dataset's `toJSON` method

```scala
val df = Seq(
    (1, null, 1.23, 66), 
    (2, "lol", 3.45, 77),
    (3, "strings", 5.6789, 88),
    (4, "another", 0.12345, 99)
).toDF("row","str_alue", "dbl_Value", "int_value")

df.toJSON.show(false)
+-----------------------------------------------------------------+
|value                                                            |
+-----------------------------------------------------------------+
|{"row":1,"dbl_Value":1.23,"int_value":66}                        |
|{"row":2,"str_alue":"lol","dbl_Value":3.45,"int_value":77}       |
|{"row":3,"str_alue":"strings","dbl_Value":5.6789,"int_value":88} |
|{"row":4,"str_alue":"another","dbl_Value":0.12345,"int_value":99}|
+-----------------------------------------------------------------+
```

 - **Manage Databases and Tables**

 ```scala
// See https://docs.databricks.com/user-guide/tables.html

 // Update the table metadata
refresh table <table-name>
 ``` 

 - **Create a DataFrame from an external file**

 ```scala
%sh 

// The file will actually be saved to file:/databricks/driver/flight_data.parquet
wget -O flight_data.csv https://raw.githubusercontent.com/databricks/Spark-The-Definitive-Guide/master/data/flight-data/csv/2010-summary.csv
 ```

# SPARK JDBC Google BigQuery Connector

The connector support using jdbc connection for Google BigQuery.
If want to use jdbc connection for Google BigQuery we need to write function for register dialect, but pyspark doesn't have this function.

You can build jar file for BigQuery yourself, or download a ready-made [MyDialect.jar](https://github.com/Fox-sv/spark-bigquery/raw/master/MyDialect.jar)

# Version

|  MyDialect | Spark  | Scala |
|:----------:|--------|:------|
| 0.1        | 3.3.0  | 2.12  |


### Python

```python
from pyspark.sql import SparkSession
from py4j.java_gateway import java_import

user_email = "EMAIL"
project_id = "PROJECT_ID"
creds = "PATH_TO_FILE"

jdbc_conn = f"jdbc:bigquery://https://www.googleapis.com/bigquery/v2:443;OAuthServiceAcctEmail={user_email};ProjectId={project_id};OAuthPvtKeyPath={creds};"

spark = SparkSession.builder.getOrCreate()

jvm = spark.sparkContext._gateway.jvm
java_import(jvm, "MyDialect")
jvm.org.apache.spark.sql.jdbc.JdbcDialects.registerDialect(jvm.MyDialect().change_dialect())

df = spark.read.jdbc(url=jdbc_conn,table='(SELECT * FROM babynames.names_2014) AS table')

```

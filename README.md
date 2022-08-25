# SPARK JDBC Google BigQuery Connector

The connector support using jdbc connection for Google BigQuery.
If want to use jdbc connection for Google BigQuery we need to write function for register dialect, but pyspark doesn't have this function.

You can build jar file for BigQuery yourself, or download a ready-made [MyDialect.jar](https://github.com/Fox-sv/spark-bigquery/raw/master/MyDialect.jar)
And put the file in $SPARK_HONE\jars\

# Version

|  MyDialect | Spark  | Scala |
|:----------:|--------|:------|
| 0.1        | 3.3.0  | 2.12  |

# Contains
jar file contains java code, which build on Maven
### Java
```java
import org.apache.spark.sql.jdbc.JdbcDialect;

public class MyDialect {
    public static void main(String[] args) {
        System.out.println("MyDialect");
    }

    public static JdbcDialect change_dialect() {

        JdbcDialect myDialect = new JdbcDialect() {

            @Override
            public boolean canHandle(String url) {
                return url.startsWith("jdbc:bigquery") || url.contains("bigquery");
            }

            @Override
            public String quoteIdentifier(String colName) { return colName; }
        };

        return myDialect;

    }
}

```
### pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>java_spark</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>

        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.3.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>3.3.0</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.4</version>
        </dependency>
        
    </dependencies>
</project>
```
# Usage
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

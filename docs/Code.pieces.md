## pyodbc
``` python
import pyodbc
'''
Use password to connect
'''
conn = pyodbc.connect('Driver={SQL Server Native Client 11.0};'
                    'Server=11.22.33.44;'
                    'Database=AdventureWorks2017;'
                    'UID=devuser;'
                    'PWD=fakepassword'
)
'''
Use Integrated login (Windows)
MARS_Connections = Yes would allow opening multiple cursors 
'''
conn = pyodbc.connect('Driver={SQL Server Native Client 11.0};'
                    'Server=fakeServerP;'
                    'Database=fakeDB;'
                    'MARS_Connection=Yes;'
                    'Trusted_Connection=yes;'
)

cursor = conn.cursor()
cursor.execute('select FirstName, LastName from Person.Person')
for row in cursor:
    print (row)
```

## cx_Oracle

``` python
import cx_Oracle
connection = cx_Oracle.connect('devuser','fakepassword','11.22.33.44/xepdb1')

cursor = connection.cursor()
queryString = "select A from a"

cursor.execute(queryString)
for abc in cursor:
    print (abc)
```

## tqdm
``` python
''' pbar.update(1) progresses 1 unit
'''
import time
from tqdm import tqdm

with tqdm(total=500) as pbar:
    for row in range (500):
        time.sleep(0.01)
        pbar.update(1)
        pass
```

## Spark

### Set environment
``` python
import findspark
import pyspark
from pyspark.sql import SparkSession
findspark.init()
spark = SparkSession.builder.getOrCreate()

df = spark.sql ("select 'spark' as hello")
df.show()
```

### Read csv
``` python
df = spark.read.csv("a.csv",header=True, inferSchema=True)
df.show()
df.count()
```

### Read from JDBC
```python
empDF = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:oracle:thin:@//hostname.com:1521/service_name") \
    .option("dbtable", "TEST_TABLE") \
    .option("user", "username") \
    .option("password", "fakepassword") \
    .option("driver", "oracle.jdbc.driver.OracleDriver") \
    .load()
empDF.count()
```
### Not tested but seemed good structure
```python
from pyspark.sql import SparkSession

def init_spark():
  spark = SparkSession.builder.appName("HelloWorld").getOrCreate()
  sc = spark.sparkContext
  return spark,sc

def main():
  spark,sc = init_spark()
  nums = sc.parallelize([1,2,3,4])
  print(nums.map(lambda x: x*x).collect())


if __name__ == '__main__':
  main()
  ```
### Not tested SQL Context
```python
from pyspark import SparkContext
from pyspark.sql import SQLContext

sc=SparkContext('local','example')
sql_sc = SQLContext(sc)
rdd = sc.textFile('edx.py')
rdd.collect()
```
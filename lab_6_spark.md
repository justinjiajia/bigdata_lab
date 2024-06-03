
# Data Preparation


```shell
wget -O transactions.txt  https://raw.githubusercontent.com/justinjiajia/datafiles/main/browsing.csv
```

# Run `PySpark` Locally

## Start the PySpark shell

```shell
$ pyspark --master local[*]
```

## Code to run sequentially

```pyspark
transactions = sc.textFile("file:///home/hadoop/transactions.txt")  # use the absolute path of the input file

transactions.cache()

transactions.take(10)

from itertools import combinations

pairs = transactions.map(lambda x: x.strip().split(" ")).flatMap(lambda x: combinations(x, 2)).map(lambda x: (x[0], x[1]) if x[0]<x[1] else (x[1], x[0]))

pairs_count = pairs.map(lambda x: (x, 1)).reduceByKey(lambda x, y: x+y)

reversed_pairs = pairs_count.map(lambda x: ((x[0][1], x[0][0]), x[1]))

all_suggested_pairs = pairs_count.union(reversed_pairs).map(lambda x: (x[0][0], [(x[0][1], x[1])]))

suggested_pairs_desc_ordered = all_suggested_pairs.reduceByKey(lambda x, y: sorted(x+y, key=lambda val: val[1], reverse=True)[:5])

suggested_pairs_desc_ordered.saveAsTextFile("file:///home/hadoop/output")

quit()
```



> Any values specified as flags or in the properties file will be passed on to the application and merged with those specified through [`SparkConf`](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.SparkConf.html). Properties set directly on the `SparkConf` take highest precedence, then flags passed to `spark-submit` or `spark-shell`, then options in the *spark-defaults.conf* file.

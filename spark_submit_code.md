
### How to modify a file that is owned by `root`?


Add `hadoop` to the `root` group:

```
$ sudo usermod -a -G root hadoop
$ groups hadoop
hadoop : hadoop root hdfsadmingroup
```

Change the owner of the file to `hadoop`

```shell
$ sudo chown hadoop:root /path/to/file
```

Once you're done, remember to set it back:

```
$ sudo chown root:root /path/to/file
```


### *spark-submit* in */usr/lib/spark/bin*

```shell
#!/usr/bin/env bash

...

if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0

exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```


- `if [ -z "${SPARK_HOME}" ];`: if `${SPARK_HOME}` is of length 0, 

- `source "$(dirname "$0")"/find-spark-home`: read and execute the code in *find-spark-home* under the same directory as *spark-submit*

- [$(dirname "$0")"/find-spark-home](https://stackoverflow.com/questions/54228196/bash-script-trying-to-get-path-of-script) 



### *find-spark-home* in */usr/lib/spark/bin*

```shell
#!/usr/bin/env bash
...
FIND_SPARK_HOME_PYTHON_SCRIPT="$(cd "$(dirname "$0")"; pwd)/find_spark_home.py"

# Short circuit if the user already has this set.
if [ ! -z "${SPARK_HOME}" ]; then
   exit 0
elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ]; then
  # If we are not in the same directory as find_spark_home.py we are not pip installed so we don't
  # need to search the different Python directories for a Spark installation.
  # Note only that, if the user has pip installed PySpark but is directly calling pyspark-shell or
  # spark-submit in another directory we want to use that version of PySpark rather than the
  # pip installed version of PySpark.
  export SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"
else
  # We are pip installed, use the Python script to resolve a reasonable SPARK_HOME
  # Default to standard python3 interpreter unless told otherwise
  if [[ -z "$PYSPARK_DRIVER_PYTHON" ]]; then
     PYSPARK_DRIVER_PYTHON="${PYSPARK_PYTHON:-"python3"}"
  fi
  export SPARK_HOME=$($PYSPARK_DRIVER_PYTHON "$FIND_SPARK_HOME_PYTHON_SCRIPT")
fi
```

- `elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ];`: if *find_spark_home.py* doesn't exist in the same directory as *find-spark-home*, 

- `export SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"`: assign the absolute path of the parent directory to `SPARK_HOME` and mark the variable to be passed to child processes 



### *spark-class* in */usr/lib/spark/bin*

```shell
#!/usr/bin/env bash

...

if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

. "${SPARK_HOME}"/bin/load-spark-env.sh
. "${SPARK_HOME}"/bin/load-emr-env.sh 2>/dev/null

...

# Find Spark jars.
if [ -d "${SPARK_HOME}/jars" ]; then
  SPARK_JARS_DIR="${SPARK_HOME}/jars"
else
  SPARK_JARS_DIR="${SPARK_HOME}/assembly/target/scala-$SPARK_SCALA_VERSION/jars"
fi
```

- `.` means `source` here: https://ss64.com/bash/source.html

  

### *load-spark-env.sh* in */usr/lib/spark/bin*  


```shell
#!/usr/bin/env bash
...
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

SPARK_ENV_SH="spark-env.sh"
if [ -z "$SPARK_ENV_LOADED" ]; then
  export SPARK_ENV_LOADED=1

  export SPARK_CONF_DIR="${SPARK_CONF_DIR:-"${SPARK_HOME}"/conf}"

  SPARK_ENV_SH="${SPARK_CONF_DIR}/${SPARK_ENV_SH}"
  if [[ -f "${SPARK_ENV_SH}" ]]; then
    # Promote all variable declarations to environment (exported) variables
    set -a
    . ${SPARK_ENV_SH}
    set +a
  fi
fi
...
```

- Experimented by adding one line of `echo $SPARK_ENV_SH` after `SPARK_ENV_SH="${SPARK_CONF_DIR}/${SPARK_ENV_SH}"`. It printed `/usr/lib/spark/conf/spark-env.sh`, indicating `SPARK_CONF_DIR` is assigned `/usr/lib/spark/conf`

- read and execute the code in *spark-env.sh* under `${SPARK_HOME}"/conf`
  
- `set -a`: mark variables which are modified or created for export

- 

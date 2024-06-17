 

### [*spark-submit*](https://github.com/apache/spark/blob/master/bin/spark-submit) in */usr/lib/spark/bin*

<br>

 

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

- `${SPARK_HOME}` is initially empty. Verified by additing one line of `echo ${SPARK_HOME}` before the if test.

- `if [ -z "${SPARK_HOME}" ];`: check if the value of variable `SPARK_HOME` is of length 0. Check out [conditional expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html) for different options Bash supports.

- `source "$(dirname "$0")"/find-spark-home`:
  
   - Placing a list of commands between parentheses causes a subshell environment to be created to execute them.
     
   - `dirname "$0"` returns the directory that contains the current script. [`$0`](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#index-0) represents the name of the script.

   - `"$(dirname "$0")"` is an instance of [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html). It executes command in the list marked by parentheses in a subshell environment and replace the whole thing with the output.
     
   - [`source`](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source) is a builtin command of the [Bash shell](https://www.gnu.org/software/bash/manual/html_node/index.html). It reads and executes the code from *find-spark-home* under the same directory in the current shell context.

- `export PYTHONHASHSEED=0`: [`export`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-export) is a builtin command of the Bash shell. It marks a name to be passed to child processes in the environment.

  
- `exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"`

   -  [`exec`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-exec) is a builtin command of the Bash shell. It allows us to execute a command that completely replaces the current process.
   
   -  ["$@"](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#index-_0040) represents all the arguments passed to *spark-submit*.

   - `org.apache.spark.deploy.SparkSubmit` is defined in [scala/org/apache/spark/deploy/SparkSubmit.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)



<br>

### [*find-spark-home*](https://github.com/apache/spark/blob/master/bin/find-spark-home) in */usr/lib/spark/bin*


<br>


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

- `elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ];`: test if file *find_spark_home.py* doesn't exist in the same directory as *find-spark-home*.

- `export SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"`: assign the absolute path of the parent directory (i.e., */usr/lib/spark/*) to `SPARK_HOME` and mark the name to be passed to child processes in the environment.

  - It includes a nested command substitution.  


<br>
  
### [*spark-class*](https://github.com/apache/spark/blob/master/bin/spark-class) in */usr/lib/spark/bin* 


<br>



```shell
#!/usr/bin/env bash

...

if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

. "${SPARK_HOME}"/bin/load-spark-env.sh
. "${SPARK_HOME}"/bin/load-emr-env.sh 2>/dev/null

...
# Find the java binary
if [ -n "${JAVA_HOME}" ]; then
  RUNNER="${JAVA_HOME}/bin/java"
else
  if [ "$(command -v java)" ]; then
    RUNNER="java"
  else
    echo "JAVA_HOME is not set" >&2
    exit 1
  fi
fi
# Find Spark jars.
if [ -d "${SPARK_HOME}/jars" ]; then
  SPARK_JARS_DIR="${SPARK_HOME}/jars"
else
  SPARK_JARS_DIR="${SPARK_HOME}/assembly/target/scala-$SPARK_SCALA_VERSION/jars"
fi

if [ ! -d "$SPARK_JARS_DIR" ] && [ -z "$SPARK_TESTING$SPARK_SQL_TESTING" ]; then
  echo "Failed to find Spark jars directory ($SPARK_JARS_DIR)." 1>&2
  echo "You need to build Spark with the target \"package\" before running this program." 1>&2
  exit 1
else
  LAUNCH_CLASSPATH="$SPARK_JARS_DIR/*"
fi

build_command() {
  "$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

```

- `. "${SPARK_HOME}"/bin/load-spark-env.sh`: `.` means `source`. See https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source

- `if [ -d "${SPARK_HOME}/jars" ];`: check if directory `${SPARK_HOME}/jars` exists or not;

  -  Variable `SPARK_HOME` has been assigned a value of `/usr/lib/spark/` when executing *find-spark-home*

-  `"$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"`

    - The flag `-Xmx` specifies the maximum memory allocation pool for a Java Virtual Machine (JVM)
      
    -  Variable `LAUNCH_CLASSPATH` holds a value of `/usr/lib/spark/jars/*`, and variable `SPARK_LAUNCHER_OPTS` holds an empty value. Verified by adding `echo` commands before `build_command()`
  
    - `-cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main`
      
      - `-cp` is used to specify classpath.
        
      - `org.apache.spark.launcher.Main` is located in `/usr/lib/spark/jars/spark-launcher*.jar`.  
        ```shell
        [hadoop@ip-xxxx ~]$ jar tf /usr/lib/spark/jars/spark-launcher*.jar | grep Main
        org/apache/spark/launcher/Main$MainClassOptionParser.class
        org/apache/spark/launcher/Main$1.class
        org/apache/spark/launcher/Main.class
        ```
        
   - The `$@`in `build_command()` includes `org.apache.spark.deploy.SparkSubmit` and all the arguments passed to *spark-submit*.
   
   - Effectively, this executes `java -Xmx128m  -cp /usr/lib/spark/jars org.apache.spark.launcher.Main "$@"`

<br>

### [*load-spark-env.sh*](https://github.com/apache/spark/blob/master/bin/load-spark-env.sh) in */usr/lib/spark/bin*  

<br>


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

- [`set -a`](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html#index-set): mark variables which are modified or created for export to the environment of subsequent commands.

- `. ${SPARK_ENV_SH}`: read and execute the code in *spark-env.sh* under `${SPARK_HOME}"/conf`
  
- Varaible `SPARK_ENV_SH` holds a value of `/usr/lib/spark/conf/spark-env.sh`. Verified by adding one line of `echo $SPARK_ENV_SH` after `SPARK_ENV_SH="${SPARK_CONF_DIR}/${SPARK_ENV_SH}"`. It also indicates `SPARK_CONF_DIR` is assigned `/usr/lib/spark/conf`



<br>

### [*pyspark*](https://github.com/apache/spark/blob/master/bin/pyspark) in */usr/lib/spark/bin*  

<br>
  

```shell
...
# Default to standard python3 interpreter unless told otherwise
if [[ -z "$PYSPARK_PYTHON" ]]; then
  PYSPARK_PYTHON=python3
fi
if [[ -z "$PYSPARK_DRIVER_PYTHON" ]]; then
  PYSPARK_DRIVER_PYTHON=$PYSPARK_PYTHON
fi
export PYSPARK_PYTHON
export PYSPARK_DRIVER_PYTHON
export PYSPARK_DRIVER_PYTHON_OPT

...
# Add the PySpark classes to the Python path:
export PYTHONPATH="${SPARK_HOME}/python/:$PYTHONPATH"
export PYTHONPATH="${SPARK_HOME}/python/lib/py4j-0.10.9.7-src.zip:$PYTHONPATH"

# Load the PySpark shell.py script when ./pyspark is used interactively:
export OLD_PYTHONSTARTUP="$PYTHONSTARTUP"
export PYTHONSTARTUP="${SPARK_HOME}/python/pyspark/shell.py"

...
exec "${SPARK_HOME}"/bin/spark-submit pyspark-shell-main --name "PySparkShell" "$@"
```


- Environment variable [`PYTHONSTARTUP`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONSTARTUP) is set to `"${SPARK_HOME}/python/pyspark/shell.py"`, which will be executed automatically when we start a Python intepreter.

- Effectively, this executes `
java -Xmx128m -cp /usr/lib/spark/jars org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit pyspark-shell-main --name "PySparkShell" "$@"`



<br>


### How to modify a file owned by `root`?

<br>

```shell
$ groups hadoop
hadoop : hadoop hdfsadmingroup
```

Add `hadoop` to the `root` group:

```shell
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



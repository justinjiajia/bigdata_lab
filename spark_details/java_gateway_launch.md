
`Popen(command, **popen_kwargs)` 


`env` contains the environment variables exported from the starter scripts

```python
{'HOSTNAME': 'ip-172-31-62-159.ec2.internal',
 'SPARK_LOG_DIR': '/var/log/spark',
 'JAVA17_HOME': '/usr/lib/jvm/jre-17',
 'PYTHONHASHSEED': '0',
 'PYSPARK_DRIVER_PYTHON': '/usr/bin/python3', 'JAVA_HOME': '/usr/lib/jvm/jre-17', 'SPARK_WORKER_PORT': '7078',
 'SPARK_MASTER_WEBUI_PORT': '8080',
 'AWS_DEFAULT_REGION': 'us-east-1',
 'HUDI_CONF_DIR': '/etc/hudi/conf',
 'SPARK_DAEMON_JAVA_OPTS': ' -XX:+ExitOnOutOfMemoryError -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0',
 'SPARK_SUBMIT_OPTS': ' -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0',
 'EMR_CLUSTER_ID': 'j-VC0KIOO5V5LS', 'SPARK_WORKER_WEBUI_PORT': '8081',
 'SPARK_MASTER_IP': 'ip-172-31-62-159.ec2.internal',
 'PYTHONSTARTUP': '/usr/lib/spark/python/pyspark/shell.py',
 'SPARK_MASTER_PORT': '7077', 'HIVE_CONF_DIR': '/etc/hive/conf', 'PYSPARK_PYTHON': '/usr/bin/python3', 'PYTHONPATH': '/usr/lib/spark/python/lib/py4j-0.10.9.7-src.zip:/usr/lib/spark/python/:',
 'HADOOP_CONF_DIR': '/etc/hadoop/conf',
 'HADOOP_HOME': '/usr/lib/hadoop',  'EMR_RELEASE_LABEL': 'emr-7.1.0', 'SPARK_PUBLIC_DNS': 'ip-172-31-62-159.ec2.internal',
 'HIVE_SERVER2_THRIFT_PORT': '10001', 'OLD_PYTHONSTARTUP': '',
 'HIVE_SERVER2_THRIFT_BIND_HOST': '0.0.0.0',
 'AWS_ACCOUNT_ID': '154048744197', 'SPARK_HOME': '/usr/lib/spark', 'STANDALONE_SPARK_MASTER_HOST': 'ip-172-31-62-159.ec2.internal',
 'SPARK_CONF_DIR': '/usr/lib/spark/conf', 'AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME': 'EMR', 
 '_SPARK_CMD_USAGE': 'Usage: ./bin/pyspark [options]', 'SPARK_WORKER_DIR': '/var/run/spark/work',
 'SPARK_ENV_LOADED': '1',
 'SPARK_SCALA_VERSION': '2.12'}
```

also 2 entries created during the execution of *Main.java*:

```python
{'PYSPARK_SUBMIT_ARGS': '"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"',
'LD_LIBRARY_PATH': '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server'}
```

and one added in *java_gateway.py*:

```python
{'_PYSPARK_DRIVER_CONN_INFO_PATH': '/tmp/tmpzcoujk06/tmpkbdpvov9'}
```


`command` equals

```python
 ['/usr/lib/spark/./bin/spark-submit', '--master', 'yarn', '--conf', 'spark.driver.memory=2g', '--name', 'PySparkShell', '--executor-memory', '2g', 'pyspark-shell']` after the assignment.
```


<br> 

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

  
- `exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"`

   - `org.apache.spark.deploy.SparkSubmit` is defined in [scala/org/apache/spark/deploy/SparkSubmit.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)






<br>
  
### [*spark-class*](https://github.com/apache/spark/blob/master/bin/spark-class) in */usr/lib/spark/bin* 


<br>


```shell
#!/usr/bin/env bash
...

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

# Add the launcher build dir to the classpath if requested.
if [ -n "$SPARK_PREPEND_CLASSES" ]; then
  LAUNCH_CLASSPATH="${SPARK_HOME}/launcher/target/scala-$SPARK_SCALA_VERSION/classes:$LAUNCH_CLASSPATH"
fi

...

build_command() {
  "$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

# Turn off posix mode since it does not allow process substitution
set +o posix
CMD=()
DELIM=$'\n'
CMD_START_FLAG="false"
while IFS= read -d "$DELIM" -r _ARG; do
  ARG=${_ARG//$'\r'}
  if [ "$CMD_START_FLAG" == "true" ]; then
    CMD+=("$ARG")
  else
    if [ "$ARG" == $'\0' ]; then
      # After NULL character is consumed, change the delimiter and consume command string.
      DELIM=''
      CMD_START_FLAG="true"
    elif [ "$ARG" != "" ]; then
      echo "$ARG"
    fi
  fi
done < <(build_command "$@")

COUNT=${#CMD[@]}
LAST=$((COUNT - 1))
LAUNCHER_EXIT_CODE=${CMD[$LAST]}

# Certain JVM failures result in errors being printed to stdout (instead of stderr), which causes
# the code that parses the output of the launcher to get confused. In those cases, check if the
# exit code is an integer, and if it's not, handle it as a special error case.
if ! [[ $LAUNCHER_EXIT_CODE =~ ^[0-9]+$ ]]; then
  echo "${CMD[@]}" | head -n-1 1>&2
  exit 1
fi

if [ $LAUNCHER_EXIT_CODE != 0 ]; then
  exit $LAUNCHER_EXIT_CODE
fi

CMD=("${CMD[@]:0:$LAST}")
exec "${CMD[@]}"
```

- `. "${SPARK_HOME}"/bin/load-spark-env.sh`: *load-spark-env.sh* contains commands that load variables from *spark-env.sh*.

   - On an EMR instance, `JAVA_HOME` is reset by *spark-env.sh* as follows:
     ```shell
     JAVA17_HOME=/usr/lib/jvm/jre-17
     export JAVA_HOME=$JAVA17_HOME
     ```

- `. "${SPARK_HOME}"/bin/load-emr-env.sh 2>/dev/null`: only available in *spark-class* on an EMR instance; load EMR specific environment variables  

- `if [ -d "${SPARK_HOME}/jars" ];`: check if directory `${SPARK_HOME}/jars` exists or not;

  -  Variable `SPARK_HOME` has been assigned a value of `/usr/lib/spark/` when executing *find-spark-home*

- Define a function named `build_command()`
  
  -  `"$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"`
  
      - The flag `-Xmx` specifies the maximum memory allocation pool for a Java Virtual Machine (JVM)
        
      -  Verified that variable `LAUNCH_CLASSPATH` holds a value of `/usr/lib/spark/jars/*`, and both variables `$SPARK_PREPEND_CLASSES` and `SPARK_LAUNCHER_OPTS` hold an empty value. 
    
      - `-cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main`
        
        - `-cp` is used to specify the class search path. So, `-cp "$LAUNCH_CLASSPATH" expands to all directories and zip/jar files under */usr/lib/spark/jars*.
          
        - `org.apache.spark.launcher.Main` is located in `/usr/lib/spark/jars/spark-launcher*.jar`.  
          ```shell
          [hadoop@ip-xxxx ~]$ jar tf /usr/lib/spark/jars/spark-launcher*.jar | grep Main
          org/apache/spark/launcher/Main$MainClassOptionParser.class
          org/apache/spark/launcher/Main$1.class
          org/apache/spark/launcher/Main.class
          ```
          
     - `$@` inside the definition of `build_command()` refers to all the arguments passed to the build_command function at the point of call. 
   
     - Effectively, this executes `/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit pyspark-shell-main --name "PySparkShell" "$@"`
  
  - `printf "%d\0" $?`       
  
     - [`$?`](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#index-_003f) expands to the exit status of the most recently executed foreground pipeline.
   
     - [The `printf` statement](https://www.gnu.org/software/gawk/manual/html_node/Basic-Printf.html) does not automatically append a newline to its output. 

- `set +o posix`: disables POSIX mode, allowing for more flexible shell features, such as process substitution.

- `CMD=()`: initialize `CMD` to an array

- The `while` loop reads parts of the output of `build_command "$@"` delineated by a custom delimiter `DELIM`

   - `while ...; do ...; done`: this constructs a loop that continues reading lines into `_ARG` and executing the commands within the loop body until the input is exhausted.
     
   - `< <(build_command "$@")` is made up of two parts:
 
       - `<(build_command "$@")` is an instance of [process substitution](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html).  The process `build_command "$@"` is run asynchronously, and its output appears as a filename (something like */dev/fd/11*). This filename is passed as an argument to the first `<`.
         ```shell
         % echo <(true)  
         /dev/fd/11
         ```
         - `$@` in `build_command "$@"` refers to all the arguments passed to *spark-class*.
    
       - The 1st `<` is an instance of [redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html). It directs the content of the resulting file to the standard input.
    
   - [`read`](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-read) is a builtin command of the Bash shell. It reads `-d delim`-delimited parts from the standard input one at a time.
 
      - If the `-r` option is given, backslashes in the input do not act as an escape character. 

      - It uses `$IFS` to split the input into words. The special shell variable [`IFS`](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html) determines how Bash recognizes word boundaries while splitting a sequence of character strings. Its default value is a three-character string comprising a space, tab, and newline. The following code shows the default value of `IFS`:
        ```shell
        [hadoop@ip-xxxx ~]$ echo "$IFS" | cat -et
         ^I$
        $
        ```
        [`cat -et`](https://www.gnu.org/software/coreutils/manual/html_node/cat-invocation.html) displays a newline as `$` before starting a new line, and displays a tab character as `^I`. Note `echo` appends a newline to its output, which leads to the appearance of the 2nd `$`.

     - `IFS= ` sets `IFS` to an empty value. It ensures that no word splitting occurs and leading/trailing whitespace is preserved.
    
   - `ARG=${_ARG//$'\r'}`: [`${parameter//pattern`](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) removes any carriage return (`\r`) characters from the argument.

   - If `CMD_START_FLAG` is `true`, [`CMD+=("$ARG")`](https://www.gnu.org/software/bash/manual/bash.html#Arrays) add `"$ARG"` to the `CMD` array.
 
       - It is important to quote `$ARG`; otherwise, it will be added as multiple elements due to the presence of spaces.
         
   - If not, check for the null character (`$'\0'`); when encountered, switch the delimiter to an empty string (`DELIM=''`) and set `CMD_START_FLAG` to `true`.

        - Once `DELIM` is set to `''`, `-d ''` indicates that each input should be delimited by a null string or `$'\0'`. 
 
        - Note the 1st line of the output of `build_command "$@"` is `\0\n`, which is generated by `System.out.println('\0');` in [*Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java#L96)

   - Verified that the output of `build_command "$@"` is a NULL-separated list of arguments: 
     ```
     envPYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server/usr/bin/python30
     ```


-  Verified the value of `"${CMD[@]}"` with the following code:
  
   ```shell
   for str_tmp in "${CMD[@]}"; do
     echo $str_tmp
   done
   ```
   prints
   ```
   env
   PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"
   LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server
   /usr/bin/python3
   ```
   - Remember to quote the variable. Using the unquoted version `${CMD[@]}` may result in breaking apart a single element into many due to the presence of whitespaces.
      
- `${#CMD[@]}` expands to the length of `${CMD[@]}`. So `COUNT` is assgined the number of elements (i.e., `5`) in the `CMD` array.

- `LAST` is assigned the index of the last element in `CMD`.

- `${CMD[$LAST]}` returns the last element in the `CMD` array, which should be the exit code.

- Validate if `$LAUNCHER_EXIT_CODE` is an integer. 

- `CMD=("${CMD[@]:0:$LAST}")` remove the last element (exit code) from the `CMD` array.

- `exec "${CMD[@]}"` replaces the current shell with the command stored in `CMD`. It executes *python/pyspark/shell.py* and starts a Python intepreter.

    - `echo "${CMD[@]}"` prints:
      ```shell
      env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
      ```



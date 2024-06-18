
List all environment variables on an EMR instance:

```shell
[hadoop@ip-xxxx ~]$ env
SHELL=/bin/bash
HISTCONTROL=ignoredups
SYSTEMD_COLORS=false
HISTSIZE=1000
HOSTNAME=ip-172-31-58-216.ec2.internal
JAVA_HOME=/etc/alternatives/jre
AWS_DEFAULT_REGION=us-east-1
PWD=/home/hadoop
LOGNAME=hadoop
XDG_SESSION_TYPE=tty
MANPATH=:/opt/puppetlabs/puppet/share/man
MOTD_SHOWN=pam
HOME=/home/hadoop
LANG=C.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36:
SSH_CONNECTION=1.64.142.155 62314 172.31.58.216 22
GEM_HOME=/home/hadoop/.local/share/gem/ruby
XDG_SESSION_CLASS=user
SELINUX_ROLE_REQUESTED=
TERM=xterm-256color
LESSOPEN=||/usr/bin/lesspipe.sh %s
USER=hadoop
SELINUX_USE_CURRENT_RANGE=
SHLVL=1
XDG_SESSION_ID=11
XDG_RUNTIME_DIR=/run/user/992
S_COLORS=auto
SSH_CLIENT=1.64.142.155 62314 22
which_declare=declare -f
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/aws/puppet/bin/:/opt/puppetlabs/bin
SELINUX_LEVEL_REQUESTED=
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/992/bus
MAIL=/var/spool/mail/hadoop
SSH_TTY=/dev/pts/0
BASH_FUNC_which%%=() {  ( alias;
 eval ${which_declare} ) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot "$@"
}
_=/usr/bin/env

[hadoop@ip-xxxx ~]$ command -v java
/usr/bin/java
```



<br>

### [*pyspark*](https://github.com/apache/spark/blob/master/bin/pyspark) in */usr/lib/spark/bin*  

<br>
  

```shell
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

source "${SPARK_HOME}"/bin/load-spark-env.sh
export _SPARK_CMD_USAGE="Usage: ./bin/pyspark [options]"

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

- The [`$` character](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) introduces parameter expansion, command substitution, or arithmetic expansion.
  
- Verified that `${SPARK_HOME}` is initially empty

- `if [ -z "${SPARK_HOME}" ];`: check if the value of variable `SPARK_HOME` is of length 0. Check out [conditional expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html) for different options Bash supports.

- `source "$(dirname "$0")"/find-spark-home`:
     
   - `dirname "$0"` returns the directory that contains the current script. [`$0`](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#index-0) expands to the name of the script.

   - `"$(dirname "$0")"` is an instance of [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html). It executes the command enclosed by parentheses in a subshell environment and replace the whole thing with the output.
     
   - [`source`](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source) is a builtin command of the [Bash shell](https://www.gnu.org/software/bash/manual/html_node/index.html). It reads and executes the code from *find-spark-home* under the same directory in the current shell context.

- [`export`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-export) is a builtin command of the Bash shell. It marks a name to be passed to child processes in the environment.

- `source "${SPARK_HOME}"/bin/load-spark-env.sh`
         
- Because */usr/lib/spark/conf/spark-env.sh* on an EMR instance contains `export PYSPARK_PYTHON=/usr/bin/python3`, both `PYSPARK_PYTHON` and `PYSPARK_DRIVER_PYTHON` are set to `/usr/bin/python3`

- Environment variable [`PYTHONSTARTUP`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONSTARTUP) is set to `"${SPARK_HOME}/python/pyspark/shell.py"`, which will be executed automatically when starting a Python intepreter.

- `exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"`

   -  [`exec`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-exec) is a builtin command of the Bash shell. It allows us to execute a command that completely replaces the current process.
   
   -  ["$@"](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#index-_0040) expands to all the arguments passed to *pyspark*.


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

- `elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ];`: test if no *find_spark_home.py* exists in the same directory as *find-spark-home*.

- `export SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"`: the nested command substitution part evaluates the absolute path of the parent directory (i.e., */usr/lib/spark/*) . It is assigned to `SPARK_HOME`, which is marked to be passed to child processes in the environment.

- [`:-`](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) in `${PYSPARK_PYTHON:-"python3"}` means if `PYSPARK_PYTHON` is unset or null, the whole thing expands to `"python3"`.

<br>


### [*load-spark-env.sh*](https://github.com/apache/spark/blob/master/bin/load-spark-env.sh) in */usr/lib/spark/bin*  

<br>


```shell
#!/usr/bin/env bash
...
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

- `. ${SPARK_ENV_SH}`: [`.`](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source) means `source`; read and execute the code in *spark-env.sh* under `${SPARK_HOME}"/conf`
  
- Verified that varaible `SPARK_ENV_SH` holds a value of `/usr/lib/spark/conf/spark-env.sh`. It also indicates that `SPARK_CONF_DIR` is assigned `/usr/lib/spark/conf`.



  
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

- `CMD = ()`: initialize `CMD` to an array

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

   - If `CMD_START_FLAG` is `true`, [`CMD+=("$ARG")`](https://www.gnu.org/software/bash/manual/bash.html#Arrays) add `$ARG` to the `CMD` array.
 
       - If `"$ARG"` contains spaces, it will be treated as a space-separated list of elements, and thereby add multiple elements to the `CMD` array.

   - If not, check for the null character (`$'\0'`); when encountered, switch the delimiter to an empty string (`DELIM=''`) and set `CMD_START_FLAG` to `true`.

        - Once `DELIM` is set to `''`, `-d ''` indicates that each input should be delimited by a null string or `$'\0'`. 
 
        - Note the first line of the output of `build_command "$@"` is `\0\n`, which is generated by `System.out.println('\0');` in [*Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java#L96)

   - Verified that the output of `build_command "$@"` is: 
     ```
     envPYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server/usr/bin/python30
     ```


-  `"${CMD[@]}"` expands each element of `CMD` to a separate word. Adding the code snippet below to the shell script
    ```shell
    for tp_str in ${CMD[@]}; do
      echo $tp_str
    done
    ```
    prints
    ```
    env
    PYSPARK_SUBMIT_ARGS="--master"
    "yarn"
    "--conf"
    "spark.driver.memory=2g"
    "--name"
    "PySparkShell"
    "--executor-memory"
    "2g"
    "pyspark-shell"
    LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server
    /usr/bin/python3
    0
    ```

      
- `${#CMD[@]}` expands to the length of `${CMD[@]}`. So `COUNT` is assgined the number of elements in the `CMD` array.

- `LAST` is assigned the index of the last element in `CMD`.

- `${CMD[$LAST]}` returns the last element in the `CMD` array, which should be the exit code.

- Validate if `$LAUNCHER_EXIT_CODE` is an integer. 

- `CMD=("${CMD[@]:0:$LAST}")` remove the last element (exit code) from the `CMD` array.

- `exec "${CMD[@]}"` replaces the current shell with the command stored in `CMD`. It executes *python/pyspark/shell.py* and starts a Python intepreter.

    - `${CMD[@]}` expands to the following:
      ```shell
      env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
      ```



<br>




### Environment variables

<br>


Order of execution: *pyspark* -> *find-spark-home* -> *load-spark-env.sh* 

-> *spark-env.sh* -> *spark-submit* ->  *spark-class* -> *load-emr-env.sh*




| Command | Value |Source |
|---|-----|--------|
| `SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"` | `/usr/lib/spark` | set by *find-spark-home*  |
|`SPARK_ENV_LOADED=1 `     |  1 | set by *load-spark-env.sh* |
|`SPARK_CONF_DIR="${SPARK_CONF_DIR:-"${SPARK_HOME}"/conf}"`|  `/usr/lib/spark/conf` | set by *load-spark-env.sh* |
| `JAVA17_HOME=/usr/lib/jvm/jre-17` | `/usr/lib/jvm/jre-17`| set by *spark-env.sh* |
|  `JAVA_HOME=$JAVA17_HOME` | `/usr/lib/jvm/jre-17` | overridden by *spark-env.sh* |
| `SPARK_LOG_DIR=${SPARK_LOG_DIR:-/var/log/spark}` | |  set by *spark-env.sh* |
| `HADOOP_HOME=${HADOOP_HOME:-/usr/lib/hadoop}` |`/usr/lib/hadoop` | set by *spark-env.sh* |
| `HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-/etc/hadoop/conf}` |`/etc/hadoop/conf` | set by *spark-env.sh* |
| `HIVE_CONF_DIR=${HIVE_CONF_DIR:-/etc/hive/conf}` |  `/etc/hive/conf`  | set by *spark-env.sh* |
| `HUDI_CONF_DIR=${HUDI_CONF_DIR:-/etc/hudi/conf}` | `/etc/hudi/conf` | set by *spark-env.sh* |
|`STANDALONE_SPARK_MASTER_HOST=ip-xxxx.ec2.internal`|  `ip-xxxx.ec2.internal` | set by *spark-env.sh* |
| `SPARK_MASTER_PORT=7077`| `7077` | set by *spark-env.sh* |
|`SPARK_MASTER_IP=$STANDALONE_SPARK_MASTER_HOST` | `ip-xxxx.ec2.internal` | set by *spark-env.sh* |
|`SPARK_MASTER_WEBUI_PORT=8080` | `8080` | set by *spark-env.sh* |
|`SPARK_WORKER_DIR=${SPARK_WORKER_DIR:-/var/run/spark/work}` | `/var/run/spark/work`| set by *spark-env.sh* |
|`SPARK_WORKER_PORT=7078`| `7078` | set by *spark-env.sh* |
|`SPARK_WORKER_WEBUI_PORT=8081`| `8081` | set by *spark-env.sh* |
|`HIVE_SERVER2_THRIFT_BIND_HOST=0.0.0.0`| `0.0.0.0` | set by *spark-env.sh* |
|`HIVE_SERVER2_THRIFT_PORT=10001| `10001` | set by *spark-env.sh* |
|`AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME=EMR`| `EMR` | set by *spark-env.sh* |
|`SPARK_DAEMON_JAVA_OPTS="$SPARK_DAEMON_JAVA_OPTS -XX:+ExitOnOutOfMemoryError"` <br>  `SPARK_DAEMON_JAVA_OPTS+=$EXTRA_OPTS` | `-XX:+ExitOnOutOfMemoryError -DAWS_ACCOUNT_ID=xxxx -DEMR_CLUSTER_ID=j-xxxx -DEMR_RELEASE_LABEL=emr-7.1.0`  |set by *spark-env.sh* <br> later appended by  *load-emr-env.sh* (EMR specific) |
|`SPARK_PUBLIC_DNS=ip-xxxx.ec2.internal` | `ip-xxxx.ec2.internal` |set by *spark-env.sh* |
|`PYSPARK_PYTHON=/usr/bin/python3`| `/usr/bin/python3` |set by *spark-env.sh* |
|`SPARK_SCALA_VERSION=2.13` | `2.13` |set by *load-spark-env.sh* |
|`_SPARK_CMD_USAGE="Usage: ./bin/pyspark [options]"` |`Usage: ./bin/pyspark [options]`  | set by *pyspark*  |
| `PYSPARK_DRIVER_PYTHON=$PYSPARK_PYTHON` |`/usr/bin/python3`| set by *pyspark*  |
|`PYTHONPATH="${SPARK_HOME}/python/:$PYTHONPATH"`<br> `PYTHONPATH="${SPARK_HOME}/python/lib/py4j-0.10.9.7-src.zip:$PYTHONPATH"`| `/usr/lib/spark/python/lib/py4j-0.10.9.7-src.zip:/usr/lib/spark/python/:`| set by *pyspark*  |
|`OLD_PYTHONSTARTUP="$PYTHONSTARTUP"`|   | set by *pyspark*  |
| `PYTHONSTARTUP="${SPARK_HOME}/python/pyspark/shell.py"` |  `/usr/lib/spark/python/pyspark/shell.py` | set by *pyspark*  |
|`PYTHONHASHSEED=0`| `0` |  set by *spark-submit*  |
|`AWS_ACCOUNT_ID=xxxx` | `xxxx` |  set by  *load-emr-env.sh* (EMR specific)  |
|`EMR_CLUSTER_ID=j-xxxx` | `j-xxxx` |  set by  *load-emr-env.sh* (EMR specific)  |
|`EMR_RELEASE_LABEL=emr-7.1.0` | `emr-7.1.0` |  set by  *load-emr-env.sh* (EMR specific)  |
|`EMR_RELEASE_LABEL=emr-7.1.0` | `emr-7.1.0` |  set by  *load-emr-env.sh* (EMR specific)  |
|`SPARK_SUBMIT_OPTS+=$EXTRA_OPTS` | `-DAWS_ACCOUNT_ID=xxxx -DEMR_CLUSTER_ID=j-xxxx -DEMR_RELEASE_LABEL=emr-7.1.0` |  set by  *load-emr-env.sh* (EMR specific)  |



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



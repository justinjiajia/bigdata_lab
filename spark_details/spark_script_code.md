
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

# Add the launcher build dir to the classpath if requested.
if [ -n "$SPARK_PREPEND_CLASSES" ]; then
  LAUNCH_CLASSPATH="${SPARK_HOME}/launcher/target/scala-$SPARK_SCALA_VERSION/classes:$LAUNCH_CLASSPATH"
fi

...

build_command() {
  "$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

```

- `. "${SPARK_HOME}"/bin/load-spark-env.sh`: `.` means `source`. See https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source

   - *load-spark-env.sh* contains commands that load variables from *spark-env.sh*.

   - On an EMR instance, `JAVA_HOME` is reset by *spark-env.sh* as follows:
     ```shell
     JAVA17_HOME=/usr/lib/jvm/jre-17
     export JAVA_HOME=$JAVA17_HOME
     ``` 

- `if [ -d "${SPARK_HOME}/jars" ];`: check if directory `${SPARK_HOME}/jars` exists or not;

  -  Variable `SPARK_HOME` has been assigned a value of `/usr/lib/spark/` when executing *find-spark-home*

-  `"$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"`

    - The flag `-Xmx` specifies the maximum memory allocation pool for a Java Virtual Machine (JVM)
      
    -  Verified that variable `LAUNCH_CLASSPATH` holds a value of `/usr/lib/spark/jars/*`, and both variables `$SPARK_PREPEND_CLASSES` and `SPARK_LAUNCHER_OPTS` hold an empty value. 
  
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
   
   - Effectively, this executes `/usr/lib/jvm/jre-17/bin/java -Xmx128m  -cp <all files under /usr/lib/spark/jars> org.apache.spark.launcher.Main "$@"`

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


- Environment variable [`PYTHONSTARTUP`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONSTARTUP) is set to `"${SPARK_HOME}/python/pyspark/shell.py"`, which will be executed automatically when starting a Python intepreter.


- Effectively, this executes `/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit pyspark-shell-main --name "PySparkShell" "$@"`


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



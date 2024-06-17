 

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

- `if [ -z "${SPARK_HOME}" ];`: check if the value of variable `SPARK_HOME` is of length 0.

- `source "$(dirname "$0")"/find-spark-home`:
  
   - Placing a list of commands between parentheses causes a subshell environment to be created to execute them.
     
   - `dirname "$0"` returns the directory that contains the current script. `$0` represents the name of the script.

   - `"$(dirname "$0")"` is an instance of [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html). It executes command in the list marked by parentheses in a subshell environment and replace the whole thing with the output.
     
   - [`source`](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-source) is a builtin command of the [Bash shell](https://www.gnu.org/software/bash/manual/html_node/index.html). It reads and executes the code from *find-spark-home* under the same directory in the current shell context.

- `export PYTHONHASHSEED=0`: [`export`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-export) is a builtin command of the Bash shell. It marks a name to be passed to child processes in the environment.

  
- `exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"`

   -  [`exec`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-exec) is a builtin command of the Bash shell. It allows us to execute a command that completely replaces the current process.
   
   -  "$@" represents all the arguments passed to spark-submit.

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

- `elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ];`: check if file *find_spark_home.py* doesn't exist in the same directory as *find-spark-home*.

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
   
   - Effectively, this executes
     ```shell
     java -Xmx128m  -cp /usr/lib/spark/jars org.apache.spark.launcher.Main "$@"
    ```



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

- Effectively, this executes
```shell
java -Xmx128m  -cp /usr/lib/spark/jars org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit \
pyspark-shell-main --name "PySparkShell" "$@"
```

### class Main in `/usr/lib/spark/jars/spark-launcher*.jar`

Source code can be found at https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java

```java
package org.apache.spark.launcher;

...

import static org.apache.spark.launcher.CommandBuilderUtils.*;

...

  public static void main(String[] argsArray) throws Exception {
    checkArgument(argsArray.length > 0, "Not enough arguments: missing class name.");

    List<String> args = new ArrayList<>(Arrays.asList(argsArray));
    String className = args.remove(0);

    boolean printLaunchCommand = !isEmpty(System.getenv("SPARK_PRINT_LAUNCH_COMMAND"));
    Map<String, String> env = new HashMap<>();
    List<String> cmd;
    if (className.equals("org.apache.spark.deploy.SparkSubmit")) {
      try {
        AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(args);
        cmd = buildCommand(builder, env, printLaunchCommand);
      } catch (IllegalArgumentException e) {
        printLaunchCommand = false;
        System.err.println("Error: " + e.getMessage());
        System.err.println();

        MainClassOptionParser parser = new MainClassOptionParser();
        try {
          parser.parse(args);
        } catch (Exception ignored) {
          // Ignore parsing exceptions.
        }

        List<String> help = new ArrayList<>();
        if (parser.className != null) {
          help.add(parser.CLASS);
          help.add(parser.className);
        }
        help.add(parser.USAGE_ERROR);
        AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(help);
        cmd = buildCommand(builder, env, printLaunchCommand);
      }
    } else {
      AbstractCommandBuilder builder = new SparkClassCommandBuilder(className, args);
      cmd = buildCommand(builder, env, printLaunchCommand);
    }

    // test for shell environments, to enable non-Windows treatment of command line prep
    boolean shellflag = !isEmpty(System.getenv("SHELL"));
    if (isWindows() && !shellflag) {
      System.out.println(prepareWindowsCommand(cmd, env));
    } else {
      // A sequence of NULL character and newline separates command-strings and others.
      System.out.println('\0');

      // In bash, use NULL as the arg separator since it cannot be used in an argument.
      List<String> bashCmd = prepareBashCommand(cmd, env);
      for (String c : bashCmd) {
        System.out.print(c.replaceFirst("\r$",""));
        System.out.print('\0');
      }
    }
  }
```



https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java#L62


https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java


```java
...
import static org.apache.spark.launcher.CommandBuilderUtils.*;

/**
 * Abstract Spark command builder that defines common functionality.
 */
abstract class AbstractCommandBuilder {

  ...

  AbstractCommandBuilder() {
    this.appArgs = new ArrayList<>();
    this.childEnv = new HashMap<>();
    this.conf = new HashMap<>();
    this.files = new ArrayList<>();
    this.jars = new ArrayList<>();
    this.pyFiles = new ArrayList<>();
  }

 ...
  Map<String, String> getEffectiveConfig() throws IOException {
    if (effectiveConfig == null) {
      effectiveConfig = new HashMap<>(conf);
      Properties p = loadPropertiesFile();
      p.stringPropertyNames().forEach(key ->
        effectiveConfig.computeIfAbsent(key, p::getProperty));
      effectiveConfig.putIfAbsent(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH,
        SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH_VALUE);
    }
    return effectiveConfig;
  }

  /**
   * Loads the configuration file for the application, if it exists. This is either the
   * user-specified properties file, or the spark-defaults.conf file under the Spark configuration
   * directory.
   */
  private Properties loadPropertiesFile() throws IOException {
    Properties props = new Properties();
    File propsFile;
    if (propertiesFile != null) {
      propsFile = new File(propertiesFile);
      checkArgument(propsFile.isFile(), "Invalid properties file '%s'.", propertiesFile);
    } else {
      propsFile = new File(getConfDir(), DEFAULT_PROPERTIES_FILE);
    }

    if (propsFile.isFile()) {
      try (InputStreamReader isr = new InputStreamReader(
          new FileInputStream(propsFile), StandardCharsets.UTF_8)) {
        props.load(isr);
        for (Map.Entry<Object, Object> e : props.entrySet()) {
          e.setValue(e.getValue().toString().trim());
        }
      }
    }
    return props;
  }

  private String getConfDir() {
    String confDir = getenv("SPARK_CONF_DIR");
    return confDir != null ? confDir : join(File.separator, getSparkHome(), "conf");
  }
  ...
}
```

It seems that the property file is `/usr/lib/spark/confspark-defaults.conf`

https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java

https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L28C1-L32C53

```java
...
class CommandBuilderUtils {

  static final String DEFAULT_MEM = "1g";
  static final String DEFAULT_PROPERTIES_FILE = "spark-defaults.conf";
  static final String ENV_SPARK_HOME = "SPARK_HOME";

  ...

}
```

https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java

```java
/**
 * Special command builder for handling a CLI invocation of SparkSubmit.
 * <p>
 * This builder adds command line parsing compatible with SparkSubmit. It handles setting
 * driver-side options and special parsing behavior needed for the special-casing certain internal
 * Spark applications.
 * <p>
 * This class has also some special features to aid launching shells (pyspark and sparkR) and also
 * examples.
 */
class SparkSubmitCommandBuilder extends AbstractCommandBuilder {

  /**
   * Name of the app resource used to identify the PySpark shell. The command line parser expects
   * the resource name to be the very first argument to spark-submit in this case.
   *
   * NOTE: this cannot be "pyspark-shell" since that identifies the PySpark shell to SparkSubmit
   * (see java_gateway.py), and can cause this code to enter into an infinite loop.
   */
  static final String PYSPARK_SHELL = "pyspark-shell-main";

  /**
   * This is the actual resource name that identifies the PySpark shell to SparkSubmit.
   */
  static final String PYSPARK_SHELL_RESOURCE = "pyspark-shell";

  /**
   * Name of the app resource used to identify the SparkR shell. The command line parser expects
   * the resource name to be the very first argument to spark-submit in this case.
   *
   * NOTE: this cannot be "sparkr-shell" since that identifies the SparkR shell to SparkSubmit
   * (see sparkR.R), and can cause this code to enter into an infinite loop.
   */
  static final String SPARKR_SHELL = "sparkr-shell-main";

  /**
   * This is the actual resource name that identifies the SparkR shell to SparkSubmit.
   */
  static final String SPARKR_SHELL_RESOURCE = "sparkr-shell";

  /**
   * Name of app resource used to identify examples. When running examples, args[0] should be
   * this name. The app resource will identify the example class to run.
   */
  static final String RUN_EXAMPLE = "run-example";

  /**
   * Prefix for example class names.
   */
  static final String EXAMPLE_CLASS_PREFIX = "org.apache.spark.examples.";

  ...

    /**
   * This constructor is used when invoking spark-submit; it parses and validates arguments
   * provided by the user on the command line.
   */
  SparkSubmitCommandBuilder(List<String> args) {
    this.allowsMixedArguments = false;
    this.parsedArgs = new ArrayList<>();
    boolean isExample = false;
    List<String> submitArgs = args;
    this.userArgs = Collections.emptyList();

    if (args.size() > 0) {
      switch (args.get(0)) {
        case PYSPARK_SHELL:
          this.allowsMixedArguments = true;
          appResource = PYSPARK_SHELL;
          submitArgs = args.subList(1, args.size());
          break;

        case SPARKR_SHELL:
          this.allowsMixedArguments = true;
          appResource = SPARKR_SHELL;
          submitArgs = args.subList(1, args.size());
          break;

        case RUN_EXAMPLE:
          isExample = true;
          appResource = findExamplesAppJar();
          submitArgs = args.subList(1, args.size());
      }

      this.isExample = isExample;
      OptionParser parser = new OptionParser(true);
      parser.parse(submitArgs);
      this.isSpecialCommand = parser.isSpecialCommand;
    } else {
      this.isExample = isExample;
      this.isSpecialCommand = true;
    }
  }
```



### How to modify a file owned by `root`?


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



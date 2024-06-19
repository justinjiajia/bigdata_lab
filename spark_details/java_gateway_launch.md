
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
 'AWS_ACCOUNT_ID': '154048744197',
 'SPARK_HOME': '/usr/lib/spark', 'STANDALONE_SPARK_MASTER_HOST': 'ip-172-31-62-159.ec2.internal',
 'SPARK_CONF_DIR': '/usr/lib/spark/conf',
 'AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME': 'EMR', 
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
 ['/usr/lib/spark/./bin/spark-submit', '--master', 'yarn', '--conf', 'spark.driver.memory=2g', '--name', 'PySparkShell', '--executor-memory', '2g', 'pyspark-shell']
```
after the assignment.

<br> 

### [*spark-submit*](https://github.com/apache/spark/blob/master/bin/spark-submit) in */usr/lib/spark/bin*

<br>

 
```shell
#!/usr/bin/env bash
...
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

- `org.apache.spark.deploy.SparkSubmit` is defined in [scala/org/apache/spark/deploy/SparkSubmit.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)

- `"$@"` expands to `--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell` 





<br>
  
### [*spark-class*](https://github.com/apache/spark/blob/master/bin/spark-class) in */usr/lib/spark/bin* 


<br>


```shell
#!/usr/bin/env bash
...

build_command() {
  "$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

...

while IFS= read -d "$DELIM" -r _ARG; do
  ...
done < <(build_command "$@")

...
exec "${CMD[@]}"
```


- Define a function named `build_command()`
  
  -  `"$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"`
          

   
     - Effectively, this executes `/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit --master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell`
  
 


-  Verified the value of `"${CMD[@]}"` with the following code:
  
 

- `exec "${CMD[@]}"` replaces the current shell with the command stored in `CMD`. It executes *python/pyspark/shell.py* and starts a Python intepreter.

    - `echo "${CMD[@]}"` prints:
      ```shell
      env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
      ```


<br>
 
### [*java/org/apache/spark/launcher/Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java)

<br>
 
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
        ...
      }
    } else {
      ...
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

  /**
   * Prepare spark commands with the appropriate command builder.
   * If printLaunchCommand is set then the commands will be printed to the stderr.
   */
  private static List<String> buildCommand(
      AbstractCommandBuilder builder,
      Map<String, String> env,
      boolean printLaunchCommand) throws IOException, IllegalArgumentException {
    List<String> cmd = builder.buildCommand(env);
    ...
    return cmd;
  }
  ...

  /**
   * Prepare the command for execution from a bash script. The final command will have commands to
   * set up any needed environment variables needed by the child process.
   */
  private static List<String> prepareBashCommand(List<String> cmd, Map<String, String> childEnv) {
    if (childEnv.isEmpty()) {
      return cmd;
    }

    List<String> newCmd = new ArrayList<>();
    newCmd.add("env");

    for (Map.Entry<String, String> e : childEnv.entrySet()) {
      newCmd.add(String.format("%s=%s", e.getKey(), e.getValue()));
    }
    newCmd.addAll(cmd);
    return newCmd;
  }
  ...
}
```

- `List<String> args = new ArrayList<>(Arrays.asList(argsArray));`

-  `printLaunchCommand` is `false`. Verified by adding `echo $(env | grep SPARK_PRINT_LAUNCH_COMMAND)` to proper places in  *spark-class*.
  
- `Map<String, String> env = new HashMap<>();`: used to maintain the user environment 

- Remove the 1st command line option and check if it equals `"org.apache.spark.deploy.SparkSubmit"`

- If so, `AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(args);`, which creates a `SparkSubmitCommandBuilder` instance with the remaining command line options (i.e., `--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell`).


- `cmd = buildCommand(builder, env, printLaunchCommand);`: calling `buildCommand()` further invokes `builder.buildCommand(env);` defined in [*SparkSubmitCommandBuilder.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L159C3-L169C4)

    - If `appResource` equals `PYSPARK_SHELL`,  execute `return buildPySparkShellCommand(env);`. What `buildPySparkShellCommand(env)` returns is is a list of strings containing python-related configurations.

- `shellflag` is `true`, because environment variable `SHELL` holds a value of `/bin/bash`. Verified by adding `echo $(env | grep SHELL)` to proper places in  *spark-class*.

- `List<String> bashCmd = prepareBashCommand(cmd, env);`

    - `bashCmd` equals `["env", "LD_LIBRARY_PATH=hive-jackson/*", "PYSPARK_SUBMIT_ARGS='--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-driver 2g pyspark-shell'", "/usr/bin/python3"]`

    - The value of environment variable `PYSPARK_SUBMIT_ARGS` is verified (adding a printing step to a proper place in *java-gateway.py*; the output was `"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"`)
 
    - The output printed for environment variable `LD_LIBRARY_PATH` was `/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server`. why??

    - Note that Java HashMap class doesn’t guarantee the insertion order.

- Lastly, print each string to the standard output, followed by a null character (`'\0'`). It executes [*/usr/lib/spark/python/pyspark/shell.py*](https://github.com/apache/spark/blob/master/python/pyspark/shell.py) and starts the Python interpreter (because we have export `PYTHONSTARTUP="${SPARK_HOME}/python/pyspark/shell.py"` in [*pyspark*](https://github.com/apache/spark/blob/master/bin/pyspark#L84 )

   
<br>

### [*java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java)

<br>

- Class `SparkSubmitCommandBuilder` extends class [`AbstractCommandBuilder`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java)

- Define several static constants for pattern matching , e.g., `static final String PYSPARK_SHELL = "pyspark-shell-main";`


- Create a `Map<String, String>` called `specialClasses` and initialize it to have several entries:

  ```java
  private static final Map<String, String> specialClasses = new HashMap<>();
  static {
    specialClasses.put("org.apache.spark.repl.Main", "spark-shell");
    specialClasses.put("org.apache.spark.sql.hive.thriftserver.SparkSQLCLIDriver",
      SparkLauncher.NO_RESOURCE);
    specialClasses.put("org.apache.spark.sql.hive.thriftserver.HiveThriftServer2",
      SparkLauncher.NO_RESOURCE);
    specialClasses.put("org.apache.spark.sql.connect.service.SparkConnectServer",
      SparkLauncher.NO_RESOURCE);
  }
  ```
  
- The constructor `SparkSubmitCommandBuilder(List<String> args)`

  - First execute the initializer defined in its parent class [`AbstractCommandBuilder`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L43C2-L70C4):
    ```java
    abstract class AbstractCommandBuilder {
    
      boolean verbose;
      String appName;
      String appResource;
      String deployMode;
      String javaHome;
      String mainClass;
      String master;
      String remote;
      protected String propertiesFile;
      final List<String> appArgs;
      final List<String> jars;
      final List<String> files;
      final List<String> pyFiles;
      final Map<String, String> childEnv;
      final Map<String, String> conf;
    
      // The merged configuration for the application. Cached to avoid having to read / parse
      // properties files multiple times.
      private Map<String, String> effectiveConfig;
    
      AbstractCommandBuilder() {
        this.appArgs = new ArrayList<>();
        this.childEnv = new HashMap<>();
        this.conf = new HashMap<>();
        this.files = new ArrayList<>();
        this.jars = new ArrayList<>();
        this.pyFiles = new ArrayList<>();
      }
    ...
    ```

  - Class `SparkSubmitCommandBuilder`'s instance initializer includes
    ```java
    final List<String> userArgs;
    private final List<String> parsedArgs;
    // Special command means no appResource and no mainClass required
    private final boolean isSpecialCommand;
    private final boolean isExample;

    ...
    private boolean allowsMixedArguments;
    
    ...
    SparkSubmitCommandBuilder(List<String> args) {
      this.allowsMixedArguments = false;
      this.parsedArgs = new ArrayList<>();
      boolean isExample = false;
      List<String> submitArgs = args;
      this.userArgs = Collections.emptyList();
      ...
    ```
    -  Note `List<String> submitArgs = args;`
      
  - Get the 1st argument for case maching. But there is no matching case.

  - `OptionParser parser = new OptionParser(true)`: [class `OptionParser`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L488C3-L577C4) extends class `SparkSubmitOptionParser` defined in [*SparkSubmitOptionParser.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java)
 
  - `parser.parse(submitArgs);`: `parse()` defined in [*SparkSubmitOptionParser.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L137C3-L193C4)
 
    - If an option name exists in a two-level list named [`opts`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L92), call [`handle()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L497C5-L544C6) to assigns its value to the corresponding field of the `SparkCommandBuilder` instance (e.g., fields `master`, `remote `, `deployMode`, `propertiesFile`, etc.) or add an entry to the `HashMap` named `conf` (several driver-related properties such as `"spark.driver.memory"`, ` "spark.driver.defaultExtraClassPath"`, etc. as defined in  [*SparkLauncher.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkLauncher.java), and the options specified via `--conf` or `-c`), depending on which case it matches: 
      ```java
      switch (opt) {
        case MASTER -> master = value;
        case REMOTE -> remote = value;
        case DEPLOY_MODE -> deployMode = value;
        case PROPERTIES_FILE -> propertiesFile = value;
        case DRIVER_MEMORY -> conf.put(SparkLauncher.DRIVER_MEMORY, value);
        case DRIVER_JAVA_OPTIONS -> conf.put(SparkLauncher.DRIVER_EXTRA_JAVA_OPTIONS, value);
        case DRIVER_LIBRARY_PATH -> conf.put(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH, value);
        case DRIVER_DEFAULT_CLASS_PATH ->
          conf.put(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH, value);
        case DRIVER_CLASS_PATH -> conf.put(SparkLauncher.DRIVER_EXTRA_CLASSPATH, value);
        case CONF -> {
          checkArgument(value != null, "Missing argument to %s", CONF);
          String[] setConf = value.split("=", 2);
          checkArgument(setConf.length == 2, "Invalid argument to %s: %s", CONF, value);
          conf.put(setConf[0], setConf[1]);
        }
      ...
        default -> {
          parsedArgs.add(opt);
          if (value != null) {
            parsedArgs.add(value);
          }
        }
      ```
         - `.put()` of a HashMap instance:  If an existing key is passed, the previous value gets replaced by the new value.
           
     - If there is no matching case, add an entry to `ArrayList<>` `parsedArgs` (e.g., options `--name`, `--packages`, etc.)


  - The resulting `SparkCommandBuilder` instance is assigned to `builder` in *Main.java*
     
- `buildCommand(builder, env, printLaunchCommand)` in [*Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java) -> `builder.buildCommand(env)` 

  ```java
  @Override
  public List<String> buildCommand(Map<String, String> env)
      throws IOException, IllegalArgumentException {
    if (PYSPARK_SHELL.equals(appResource) && !isSpecialCommand) {
      ...
    } 
    ...
    else {
      return buildSparkSubmitCommand(env);
    }
  }
  ```

- [`private List<String> buildSparkSubmitCommand(Map<String, String> env)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L262C3-L318C4)
  ```java
  private List<String> buildSparkSubmitCommand(Map<String, String> env)
      throws IOException, IllegalArgumentException {
    // Load the properties file and check whether spark-submit will be running the app's driver
    // or just launching a cluster app. When running the driver, the JVM's argument will be
    // modified to cover the driver's configuration.
    Map<String, String> config = getEffectiveConfig();
    boolean isClientMode = isClientMode(config);
    String extraClassPath = isClientMode ? config.get(SparkLauncher.DRIVER_EXTRA_CLASSPATH) : null;
    String defaultExtraClassPath = config.get(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH);
    if (extraClassPath == null || extraClassPath.trim().isEmpty()) {
      extraClassPath = defaultExtraClassPath;
    } else {
      extraClassPath += File.pathSeparator + defaultExtraClassPath;
    }
  
    List<String> cmd = buildJavaCommand(extraClassPath);
    // Take Thrift/Connect Server as daemon
    if (isThriftServer(mainClass) || isConnectServer(mainClass)) {
      addOptionString(cmd, System.getenv("SPARK_DAEMON_JAVA_OPTS"));
    }
    addOptionString(cmd, System.getenv("SPARK_SUBMIT_OPTS"));
  
    // We don't want the client to specify Xmx. These have to be set by their corresponding
    // memory flag --driver-memory or configuration entry spark.driver.memory
    String driverDefaultJavaOptions = config.get(SparkLauncher.DRIVER_DEFAULT_JAVA_OPTIONS);
    checkJavaOptions(driverDefaultJavaOptions);
    String driverExtraJavaOptions = config.get(SparkLauncher.DRIVER_EXTRA_JAVA_OPTIONS);
    checkJavaOptions(driverExtraJavaOptions);
  
    if (isClientMode) {
      // Figuring out where the memory value come from is a little tricky due to precedence.
      // Precedence is observed in the following order:
      // - explicit configuration (setConf()), which also covers --driver-memory cli argument.
      // - properties file.
      // - SPARK_DRIVER_MEMORY env variable
      // - SPARK_MEM env variable
      // - default value (1g)
      // Take Thrift/Connect Server as daemon
      String tsMemory =
        isThriftServer(mainClass) || isConnectServer(mainClass) ?
          System.getenv("SPARK_DAEMON_MEMORY") : null;
      String memory = firstNonEmpty(tsMemory, config.get(SparkLauncher.DRIVER_MEMORY),
        System.getenv("SPARK_DRIVER_MEMORY"), System.getenv("SPARK_MEM"), DEFAULT_MEM);
      cmd.add("-Xmx" + memory);
      addOptionString(cmd, driverDefaultJavaOptions);
      addOptionString(cmd, driverExtraJavaOptions);
      mergeEnvPathList(env, getLibPathEnvName(),
        config.get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));
    }
  
    // SPARK-36796: Always add some JVM runtime default options to submit command
    addOptionString(cmd, JavaModuleOptions.defaultModuleOptions());
    addOptionString(cmd, "-Dderby.connection.requireAuthentication=false");
    cmd.add("org.apache.spark.deploy.SparkSubmit");
    cmd.addAll(buildSparkSubmitArgs());
    return cmd;
  }
  ```

  - `Map<String, String> config = getEffectiveConfig();`
 
     -  [`getEffectiveConfig()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L274C3-L284C4)
        -  `effectiveConfig = new HashMap<>(conf);`, i.e., `effectiveConfig` is initialized to have some configurations specified via several driver-related command line options and the flags `--conf` and `-c`.
          
        - [loadPropertiesFile()](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L286C3-L311C4): load configurations from a file specified via the command line option `--properties-file` or the *spark-defaults.conf* file under */usr/lib/spark/conf*. [`DEFAULT_PROPERTIES_FILE`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L31) is a constant with the value of `"spark-defaults.conf"`.
     
         - Add to `effectiveConfig` all configurations specified by the properties file and an additional one with the key `"spark.driver.defaultExtraClassPath"` and the value `"hive-jackson/*"` if no such an entry is present in the properties file.
     
            - No entry with the name `"spark.driver.defaultExtraClassPath"` in */usr/lib/spark/conf/spark-defaults.conf* on an EMR instance.
          
  - if (isClientMode): `isClientMode` evaluates to `true` because `userDeployMode == null` is `true`.
    ```java
    boolean isClientMode(Map<String, String> userProps) {
      String userMaster = firstNonEmpty(master, userProps.get(SparkLauncher.SPARK_MASTER));
      String userDeployMode = firstNonEmpty(deployMode, userProps.get(SparkLauncher.DEPLOY_MODE));
      // Default master is "local[*]", so assume client mode in that case
      return userMaster == null || userDeployMode == null || "client".equals(userDeployMode);
    }
    ```
  - Set `extraClassPath` to the value of property `"spark.driver.extraClassPath"`. This property is set in */usr/lib/spark/conf/spark-defaults.conf* on an EMR instance.

  - Set `defaultExtraClassPath` to `"hive-jackson/*"`
 
  - `extraClassPath += File.pathSeparator + defaultExtraClassPath;` appends `"hive-jackson/*"` to `extraClassPath`

  - `List<String> cmd = buildJavaCommand(extraClassPath);`
 
      - `List<String> buildJavaCommand(String extraClassPath)` is defined in [*AbstractCommandBuilder.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L83C3-L120C4)  

      - Return an `ArrayList<>` that contains `'/usr/lib/jvm/jre-17/bin/java'`, `"-cp"`, and a list of class files formed from `extraClassPath`

  - `driverDefaultJavaOptions` is set to the value configured for `"spark.driver.defaultJavaOptions"` in *spark-defaults.conf* .
 
  - `String memory = firstNonEmpty(tsMemory, config.get(SparkLauncher.DRIVER_MEMORY), `System.getenv("SPARK_DRIVER_MEMORY"), System.getenv("SPARK_MEM"), DEFAULT_MEM);`
 
      - Since `config.get(SparkLauncher.DRIVER_MEMORY)` is non-empty and equals `"2g"`, `memory` is set to `"2g"`
    
  - `cmd.add("-Xmx" + memory);`
 
  -  `addOptionString(cmd, driverDefaultJavaOptions);`
 
  - `mergeEnvPathList(env, getLibPathEnvName(), getEffectiveConfig().get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));`
      
       -  [`getLibPathEnvName()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L90C3-L102C4): return `"LD_LIBRARY_PATH"` because `System.getProperty("os.name")` returns `Linux` on an EMR instance.  
     
       - [`mergeEnvPathList(Map<String, String> userEnv, String envKey, String pathList)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L110C3-L119C4): append the value of property "spark.driver.extraLibraryPath" to the first nom-empty value between the entry `"LD_LIBRARY_PATH"` in the user environment `env` and the same-name environment variable, and write the prolonged path to the user environment `env`.

          - Now, `env` contains the 1st entry with the key `LD_LIBRARY_PATH` and the value of property `"spark.driver.extraLibraryPath"`, which is set in *spark-defaults.conf*.

   - `addOptionString(cmd, JavaModuleOptions.defaultModuleOptions());`
   - `addOptionString(cmd, "-Dderby.connection.requireAuthentication=false");`
   - `cmd.add("org.apache.spark.deploy.SparkSubmit");`
   - `cmd.addAll(buildSparkSubmitArgs());`
     
   - `buildSparkSubmitArgs()`: add a restricted set of options in a particular order (e.g., `--master`, `--remote`, `--deploy-mode`, etc.) to an `ArrayList<>`; then add all configurations `conf` contains to the same list as pairs of `"--conf"` and `"<key string>=<value>"`. So driver-related properties set via options such as `--driver-memory` get translated to pairs of `"--conf"` and `"spark.driver.memory=<value>"`; then add all configurations maintained by `parsedArgs`.
     
     ```java
     List<String> buildSparkSubmitArgs() {
       List<String> args = new ArrayList<>();
       OptionParser parser = new OptionParser(false);
       final boolean isSpecialCommand;
   
       ...
   
       if (master != null) {
         args.add(parser.MASTER);
         args.add(master);
       }
   
       ...
   
       for (Map.Entry<String, String> e : conf.entrySet()) {
         args.add(parser.CONF);
         args.add(String.format("%s=%s", e.getKey(), e.getValue()));
       }
   
       if (propertiesFile != null) {
         args.add(parser.PROPERTIES_FILE);
         args.add(propertiesFile);
       }
     
       ..
       args.addAll(parsedArgs);
     
       ...
       return args;
     }
     ```




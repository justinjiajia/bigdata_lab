Effectively

```shell
/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> \
org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit \
--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell
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


- If so, `AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(args);`, which creates a `SparkSubmitCommandBuilder` instance with the remaining command line options.

  - There's no matching case this time. As a result, several fields are set as follows:
    ```java
    this.allowsMixedArguments = false;
    boolean isExample = false;
    List<String> submitArgs = args;
    ```
    and `appResource` is initialized to `null` 
    

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
    
  - Get the 1st argument for case maching. There's no matching case in this run.

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
           
     - If there is no case match, add an entry to `ArrayList<>` `parsedArgs` (e.g., options `--name`, `--packages`, etc.)


  - The resulting `SparkCommandBuilder` instance is assigned to `builder` in *Main.java*
     
- `buildCommand(builder, env, printLaunchCommand)` in [*Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java) -> `builder.buildCommand(env)` 

  ```java
  @Override
  public List<String> buildCommand(Map<String, String> env)
      throws IOException, IllegalArgumentException {
    if (PYSPARK_SHELL.equals(appResource) && !isSpecialCommand) {
      ...
    } else if (SPARKR_SHELL.equals(appResource) && !isSpecialCommand) {
      ...
    } else {
      return buildSparkSubmitCommand(env);
    }
  }
  ```

- [`private List<String> buildSparkSubmitCommand(Map<String, String> env)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L262C3-L318C4)
  ```java
  Map<String, String> config = getEffectiveConfig();
  boolean isClientMode = isClientMode(config);
  String extraClassPath = isClientMode ? config.get(SparkLauncher.DRIVER_EXTRA_CLASSPATH) : null;
  String defaultExtraClassPath = config.get(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH);
  if (extraClassPath == null || extraClassPath.trim().isEmpty()) {
    ...
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
  ...

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
    ...
    mergeEnvPathList(env, getLibPathEnvName(),
      config.get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));
  }

  // SPARK-36796: Always add some JVM runtime default options to submit command
  addOptionString(cmd, JavaModuleOptions.defaultModuleOptions());
  addOptionString(cmd, "-Dderby.connection.requireAuthentication=false");
  cmd.add("org.apache.spark.deploy.SparkSubmit");
  cmd.addAll(buildSparkSubmitArgs());
  return cmd;
  ```

  - `Map<String, String> config = getEffectiveConfig();`
         
       -  [`getEffectiveConfig()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L274C3-L284C4) merges several driver-related command line options and options specified via the flags `--conf` or `-c` with configurations specified in the properties file
  
     
  - `String extraClassPath = isClientMode ? config.get(SparkLauncher.DRIVER_EXTRA_CLASSPATH) : null;`: load the value of `"spark.driver.extraClassPath"` specified in *spark-defaults.conf* available on an EMR instance.
        
  - `config.get(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH)` in `String defaultExtraClassPath = config.get(SparkLauncher.DRIVER_DEFAULT_EXTRA_CLASS_PATH);` retrieves the value of `"spark.driver.defaultExtraClassPath"` specified in [*AbstractCommandBuilder.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L280C7-L281C62) and [*SparkLauncher.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkLauncher.java#L60).
 
  - `extraClassPath += File.pathSeparator + defaultExtraClassPath;`
    
  - `List<String> cmd = buildJavaCommand(extraClassPath);`:
 
     - [`buildJavaCommand(extraClassPath)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L83C3-L120C4)
       ```java
       List<String> cmd = new ArrayList<>();
   
       String firstJavaHome = firstNonEmpty(javaHome,
         childEnv.get("JAVA_HOME"),
         System.getenv("JAVA_HOME"),
         System.getProperty("java.home"));
   
       if (firstJavaHome != null) {
         cmd.add(join(File.separator, firstJavaHome, "bin", "java"));
       }
       
       ...
       cmd.add("-cp");
       cmd.add(join(File.pathSeparator, buildClassPath(extraClassPath)));
       return cmd;
       ```
       - There is no file *java-opts* in */usr/lib/spark/conf* on an EMR instance.
       - [`List<String> buildClassPath(String appClassPath)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L128C3-L209C3)
         ```java
         ...
         Set<String> cp = new LinkedHashSet<>();
         addToClassPath(cp, appClassPath);

         addToClassPath(cp, getConfDir());
         ...
         String jarsDir = findJarsDir(getSparkHome(), getScalaVersion(), !isTesting && !isTestingSql);
         if (jarsDir != null) {
           // Place slf4j-api-* jar first to be robust
           for (File f: new File(jarsDir).listFiles()) {
             if (f.getName().startsWith("slf4j-api-")) {
               addToClassPath(cp, f.toString());
             }
           }
           addToClassPath(cp, join(File.separator, jarsDir, "*"));
         }
         addToClassPath(cp, getenv("HADOOP_CONF_DIR"));
         ...
         ```
         - The returned list contains the value of the `spark.driver.extraClassPath` property specified in the *spark-defaults.conf* file, `/usr/lib/spark/conf/`, `/usr/lib/spark/jars/*`, and `/etc/hadoop/conf/`.
         - [`static String findJarsDir(String sparkHome, String scalaVersion, boolean failIfNotFound)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L310C3-L327C4)
         
       - The returned list `cmd` contains `"/usr/lib/jvm/jre-17/bin/java"`, `"-cp"`, and what `buildClassPath(extraClassPath)` returns.

   - The environment variable `SPARK_SUBMIT_OPTS` was set by *load-emr-env.sh*. `addOptionString(cmd, System.getenv("SPARK_SUBMIT_OPTS"));` adds the following items into the `cmd` list:
     ```shell
     -DAWS_ACCOUNT_ID=688430810480
     -DEMR_CLUSTER_ID=j-3JZ8WOC269WHI
     -DEMR_RELEASE_LABEL=emr-7.1.0
     -DAWS_ACCOUNT_ID=688430810480
     -DEMR_CLUSTER_ID=j-3JZ8WOC269WHI
     -DEMR_RELEASE_LABEL=emr-7.1.0
     ```
   - `cmd.add("-Xmx" + memory);` inserts `-Xmx2g` into the `cmd` list.
   - `String driverDefaultJavaOptions = config.get(SparkLauncher.DRIVER_DEFAULT_JAVA_OPTIONS);`
      - `SparkLauncher.DRIVER_DEFAULT_JAVA_OPTIONS` is an alias of `"spark.driver.defaultJavaOptions"`, whose value is set to `-XX:OnOutOfMemoryError='kill -9 %p'` in *spark-defaults.conf*. 
   - `addOptionString(cmd, driverDefaultJavaOptions);` inserts `-XX:OnOutOfMemoryError='kill -9 %p'` into the `cmd` list.
  
   - `mergeEnvPathList(env, getLibPathEnvName(), config.get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));` 

     - `getLibPathEnvName()` returns `"LD_LIBRARY_PATH"` because `System.getProperty("os.name")` returns Linux on an EMR instance.
    
     - [`mergeEnvPathList(Map<String, String> userEnv, String envKey, String pathList)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L110C3-L119C4) appends the value of property `"spark.driver.extraLibraryPath"` to the first non-empty value between the entry `"LD_LIBRARY_PATH"` in the user environment `env` and the same-name environment variable, and write the prolonged path to the user environment `env`
        - *spark-defaults.conf* has set the value of the property `"spark.driver.extraLibraryPath"` to `/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server`
        - *java_gateway.py* also passed an environment variable named `"LD_LIBRARY_PATH"`. Its value is `'/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server'`.
        - So the value of `"LD_LIBRARY_PATH"` contains 2 identical copies of the paths `/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server`
        - but found 3 additional paths present in the beginning: `/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib:/usr/lib/jvm/java-17-amazon-corretto.x86_64/../lib`. Why??
        - Now, the user environment `env` contains the 1st entry with the key `LD_LIBRARY_PATH`.
   

   -  `addOptionString(cmd, JavaModuleOptions.defaultModuleOptions());` adds the following items into the `cmd` list:
      ```shell
      --add-opens=java.base/java.lang=ALL-UNNAMED
      --add-opens=java.base/java.lang.invoke=ALL-UNNAMED
      --add-opens=java.base/java.lang.reflect=ALL-UNNAMED
      --add-opens=java.base/java.io=ALL-UNNAMED
      --add-opens=java.base/java.net=ALL-UNNAMED
      --add-opens=java.base/java.nio=ALL-UNNAMED
      --add-opens=java.base/java.util=ALL-UNNAMED
      --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
      --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
      --add-opens=java.base/sun.nio.ch=ALL-UNNAMED
      --add-opens=java.base/sun.nio.cs=ALL-UNNAMED
      --add-opens=java.base/sun.security.action=ALL-UNNAMED
      --add-opens=java.base/sun.util.calendar=ALL-UNNAMED
      --add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED
      -Djdk.reflect.useDirectMethodHandle=false
      ```
      - [`JavaModuleOptions.defaultModuleOptions()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/JavaModuleOptions.java#L52C5-L54C6)
           
  - `cmd.add("org.apache.spark.deploy.SparkSubmit");` adds `"org.apache.spark.deploy.SparkSubmit"` into the `cmd` list.
  - `cmd.add(join(File.pathSeparator, buildClassPath(extraClassPath)));` adds the following items into the `cmd` list:
    ```shell
    --master
    yarn
    --conf
    spark.driver.memory=2g
    --name
    PySparkShell
    --executor-memory
    2g
    pyspark-shell
    ```
 
 

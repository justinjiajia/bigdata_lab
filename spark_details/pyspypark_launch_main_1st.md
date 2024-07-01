```shell
pyspark --master yarn --driver-memory=2g --executor-memory=2g
```

Effectively:

```shell
/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> \
org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit \
pyspark-shell-main --name "PySparkShell" --master yarn --driver-memory=2g --executor-memory=2g
```

<br>
 
### [*java/org/apache/spark/launcher/Main.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/Main.java)

<br>
 
```java
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
  
- `Map<String, String> env = new HashMap<>();`
    
    -  Used to maintain the user environment 

- Remove the 1st command line option and check if it equals `"org.apache.spark.deploy.SparkSubmit"`


- If so, `AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(args);`, which creates a `SparkSubmitCommandBuilder` instance with the remaining command line options (i.e., `pyspark-shell-main --name "PySparkShell" "$@"`).

  - When the 1st element in `args` equals `pyspark-shell-main`, the [case matching code](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L131C9-L147C8) assigns the following values to the two fields:
    ```java
    appResource = PYSPARK_SHELL;
    submitArgs = args.subList(1, args.size());
    ```
    where `PYSPARK_SHELL` is a static constant equal to `"pyspark-shell-main"`.

- `cmd = buildCommand(builder, env, printLaunchCommand);`: calling `buildCommand()` further invokes `builder.buildCommand(env);` defined in [*SparkSubmitCommandBuilder.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L159C3-L169C4)

    - If `appResource` equals `PYSPARK_SHELL`,  execute `return buildPySparkShellCommand(env);`. What `buildPySparkShellCommand(env)` returns is a list of strings containing python-related configurations.

- `shellflag` is `true`, because environment variable `SHELL` holds a value of `/bin/bash`. Verified by adding `echo $(env | grep SHELL)` to proper places in  *spark-class*.

- `List<String> bashCmd = prepareBashCommand(cmd, env);`

    - `bashCmd` equals `["env", "PYSPARK_SUBMIT_ARGS=<some value>", "LD_LIBRARY_PATH=<some value>", "/usr/bin/python3"]`. Note that Java HashMap class doesn't guarantee the insertion order. 
 

    - The value of environment variable `PYSPARK_SUBMIT_ARGS` is verified (adding a printing step to a proper place in [*java-gateway.py*](https://github.com/apache/spark/blob/master/python/pyspark/java_gateway.py); the output was `"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"`).
 
    - The value of environment variable `LD_LIBRARY_PATH` is also verified to be the same as property `"spark.driver.extraLibraryPath"` set in *spark-defaults.conf*.
 

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
    
  - Get the 1st argument for pattern maching: 
    ```java
    if (args.size() > 0) {
      switch (args.get(0)) {
        case PYSPARK_SHELL:
          this.allowsMixedArguments = true;
          appResource = PYSPARK_SHELL;
          submitArgs = args.subList(1, args.size());
          break;
        ...
      }
    ```

    - Because the 1st element equals `"pyspark-shell-main"`, the `appResource` field is assigned the value `"pyspark-shell-main"`, the `submitArgs` field is assigned a list packing the rest of the arguments (i.e., `--name "PySparkShell" "$@"`).

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
      return buildPySparkShellCommand(env);
    } 
    ...
  }
  ```

- [`private List<String> buildPySparkShellCommand(Map<String, String> env)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L333C3-L385C4)

  - `appResource = PYSPARK_SHELL_RESOURCE;`. Note `static final String PYSPARK_SHELL_RESOURCE = "pyspark-shell";`
    
  - `constructEnvVarArgs(env, "PYSPARK_SUBMIT_ARGS");`
  
    ```java
    private void constructEnvVarArgs(
        Map<String, String> env,
        String submitArgsEnvVariable) throws IOException {
      mergeEnvPathList(env, getLibPathEnvName(),
        getEffectiveConfig().get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));
  
      StringBuilder submitArgs = new StringBuilder();
      for (String arg : buildSparkSubmitArgs()) {
        if (submitArgs.length() > 0) {
          submitArgs.append(" ");
        }
        submitArgs.append(quoteForCommandString(arg));
      }
      env.put(submitArgsEnvVariable, submitArgs.toString());
    }
    ```
    - `mergeEnvPathList(env, getLibPathEnvName(), getEffectiveConfig().get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH));`
      
       -  [`getLibPathEnvName()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L90C3-L102C4): return `"LD_LIBRARY_PATH"` because `System.getProperty("os.name")` returns `Linux` on an EMR instance.  
 
       -  [`getEffectiveConfig()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L274C3-L284C4)
           -  `effectiveConfig = new HashMap<>(conf);`, i.e., `effectiveConfig` is initialized to have some configurations specified via several driver-related command line options and the flags `--conf` and `-c`.
            
           - [loadPropertiesFile()](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L286C3-L311C4): load configurations from a file specified via the command line option `--properties-file` or the *spark-defaults.conf* file under the Spark configuration directory. [`DEFAULT_PROPERTIES_FILE`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L31) is a constant with the value of `"spark-defaults.conf"`.
       
           - Add to `effectiveConfig` all configurations specified by the properties file and an additional one with the key `"spark.driver.defaultExtraClassPath"` and the value `"hive-jackson/*"` if no such an entry is present in the properties file.
       
              - No entry with the name `"spark.driver.defaultExtraClassPath"` in */usr/lib/spark/conf/spark-defaults.conf* on an EMR instance.
 
           - `getEffectiveConfig().get(SparkLauncher.DRIVER_EXTRA_LIBRARY_PATH)` gets the value of the property `"spark.driver.extraLibraryPath"`, which is set in *spark-default.confs*.
     
     - [`mergeEnvPathList(Map<String, String> userEnv, String envKey, String pathList)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L110C3-L119C4): append the value of property `"spark.driver.extraLibraryPath"` to the first nom-empty value between the entry `"LD_LIBRARY_PATH"` in the user environment `env` and the same-name environment variable, and write the prolonged path to the user environment `env`
   
        - Now, `env` contains the 1st entry with the key `LD_LIBRARY_PATH` and the value of property `"spark.driver.extraLibraryPath"`, which is set in *spark-defaults.conf*.


    - `buildSparkSubmitArgs()`: add a restricted set of options in a particular order (e.g., `--master`, `--remote`, `--deploy-mode`, etc.) to an `ArrayList<>`; then add all configurations `conf` contains to the same list as pairs of `"--conf"` and `"<key string>=<value>"`. So driver-related command line options such as `--driver-memory` get translated to `--conf spark.driver.memory=<value>`; then add all configurations maintained by `parsedArgs`; lastly, add `"pyspark-shell"`, the value of `appResource`.
      
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
    
        if (remote != null) {
          args.add(parser.REMOTE);
          args.add(remote);
        }
    
        if (deployMode != null) {
          args.add(parser.DEPLOY_MODE);
          args.add(deployMode);
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
    
        if (appResource != null) {
          args.add(appResource);
        }
    
        args.addAll(appArgs);
    
        return args;
      }
      ```

     - Construct a string from the `ArrayList<>` returned from `buildSparkSubmitArgs()`, e.g.,  `'--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-driver 2g pyspark-shell'`, and associate it with the key `"PYSPARK_SUBMIT_ARGS"`, and write it into `env`.

         - Now, `env` contains the 2nd entry with the key `PYSPARK_SUBMIT_ARGS` and the value `'--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-driver 2g pyspark-shell'`
           
  -  `List<String> pyargs = new ArrayList<>();`
 
  -  Pick up the binary executable in the following order: `--conf spark.pyspark.driver.python` > `--conf spark.pyspark.python` > environment variable `PYSPARK_DRIVER_PYTHON` > environment variable `PYSPARK_PYTHON` > `python3`, and add it to  `pyargs`. Note that environment variables `PYSPARK_DRIVER_PYTHON` and `PYSPARK_PYTHON` were set to `/usr/bin/python3` in script [*pyspark*](https://github.com/apache/spark/blob/master/bin/pyspark#L41C1-L46C3)

  - return `pyargs`



<br>



### [*java/org/apache/spark/launcher/SparkSubmitOptionParser.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java)

<br>

Include the [definitions](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L39C3-L80C44) of instance-level constant fields [Define constants] used for matching (e.g., `CONF`, `PROPERTIES_FILE`, `EXECUTOR_MEMORY`, etc.) 

  

###  [*scala/org/apache/spark/deploy/SparkSubmit.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)

<br>

- `val appArgs = parseArguments(args)` -> `new SparkSubmitArguments(args.toImmutableArraySeq)`

- `val sparkConf = appArgs.toSparkConf()`: `toSparkConf()` defined for class `SparkSubmitArguments` in *SparkSubmitArguments.scala*
  ```scala
  private[deploy] def toSparkConf(sparkConf: Option[SparkConf] = None): SparkConf = {
    // either use an existing config or create a new empty one
    sparkProperties.foldLeft(sparkConf.getOrElse(new SparkConf())) {
      case (conf, (k, v)) => conf.set(k, v)
    }
  }
  ```
  It puts all properties contained in `sparkProperties` into a `SparkConf` instance.
  
```scala
/**
 * Main gateway of launching a Spark application.
 *
 * This program handles setting up the classpath with relevant Spark dependencies and provides
 * a layer over the different cluster managers and deploy modes that Spark supports.
 */
private[spark] class SparkSubmit extends Logging {

  override protected def logName: String = classOf[SparkSubmit].getName

  import DependencyUtils._
  import SparkSubmit._

  def doSubmit(args: Array[String]): Unit = {
    val appArgs = parseArguments(args)
    val sparkConf = appArgs.toSparkConf()
    ... 
  }

  protected def parseArguments(args: Array[String]): SparkSubmitArguments = {
    new SparkSubmitArguments(args.toImmutableArraySeq)
  }

  ...

```

<br>

### [*scala/org/apache/spark/deploy/SparkSubmitArguments.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala)

<br>

`SparkSubmitArguments` is derived from class [`SparkSubmitArgumentsParser`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/launcher/SparkSubmitArgumentsParser.scala), which makes the Java class [`SparkSubmitOptionParser`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java) visible for Spark code

- `parse(args.asJava)`: [`parse()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L137C1-L193C4) defined for the parent class `SparkSubmitOptionParser` parses and handles different type of command line options. 

  - If an option name exists in a two-level list named [`opts`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L92), 
  [`handle()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L349C1-L473C4) 
  assigns its value to the corresponding field declared at the [beginning](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L44C1-L86C50) of the definition of class `SparkSubmitArguments`.

    ```scala
    override protected def handle(opt: String, value: String): Boolean = {
      opt match {
        ...
        case EXECUTOR_CORES =>
          executorCores = value
  
        case EXECUTOR_MEMORY =>
          executorMemory = value
    
        ...
        case PROPERTIES_FILE =>
          propertiesFile = value
        ...
    ```
 
   - For options defined via `--conf` or `-c`, e.g., `--conf spark.eventLog.enabled=false
      --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps"`, the processing defined in `handle()` is a bit different:
    
      ```scala
          case CONF =>
            val (confName, confValue) = SparkSubmitUtils.parseSparkConfProperty(value)
            sparkProperties(confName) = confValue
      ```
      where `sparkProperties` is an empty `HashMap[String, String]` [initialized by this constructor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L77).
  
   - After `parse()` completes, `sparkProperties` is already filled with properties specified through `--conf`

   - The constants used for matching (e.g., `CONF`, `PROPERTIES_FILE`, `EXECUTOR_MEMORY`, etc.) are defined in [*SparkSubmitOptionParser.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L39C3-L80C44)

- [`mergeDefaultSparkProperties()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L129C1-L144C4) 


  - `loadPropertiesFromFile(propertiesFile)`: When `--properties-file` was used to specify a properties file (so `propertiesFile` is not `null`), merge values from that file with those specified through `--conf` in `sparkProperties`.
  
  
  - `loadPropertiesFromFile(Utils.getDefaultPropertiesFile(env))`: When no input properties file is specified via `--properties-file` or when `--load-spark-defaults` flag is set, load properties from `spark-defaults.conf`. Note: `env: Map[String, String] = sys.env` is in [the signature of the primary constructor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L42) of class `SparkSubmitArguments`
  
  
  - [`loadPropertiesFromFile(filePath: String)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L109) only adds new entries to `sparkProperties`
      ```scala
      ...
      val properties = Utils.getPropertiesFromFile(filePath)
      properties.foreach { case (k, v) =>
            if (!sparkProperties.contains(k)) {
              sparkProperties(k) = v
            }
      }
      ...
      ```
  
  - In summary, the precedence of property setting is as follows: `--conf` > properties in a file specified via  `--properties-file` > properties in file `spark-defaults.conf`



- [`loadEnvironmentArguments()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L158C1-L239C4): Load arguments from environment variables, Spark properties etc.

  - E.g,, if `executorMemory` is still `null` (e.g., hasn't be set via the `--executor-memory` flag), try to load the value associated with the key `"spark.executor.memory"` from `sparkProperties` first; if there's no such a key in `sparkProperties`, try to load the value from a relevant environment variable.
    ```scala
    ...
    executorMemory = Option(executorMemory)
      .orElse(sparkProperties.get(config.EXECUTOR_MEMORY.key))
      .orElse(env.get("SPARK_EXECUTOR_MEMORY"))
      .orNull
    executorCores = Option(executorCores)
      .orElse(sparkProperties.get(config.EXECUTOR_CORES.key))
      .orElse(env.get("SPARK_EXECUTOR_CORES"))
      .orNull
    ```
  - Objects like `config.EXECUTOR_MEMORY` and `config.DRIVER_CORES` are `ConfigEntry` instances defined in [*package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala). Their `.key` fields are strings like `"spark.executor.memory"` and `"spark.driver.memory"`.
 
  - This method does not change the content of `sparkProperties`.
    
- [validateArguments()](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L241C2-L249C4): validate all fields. Note: some fields can hold a `null` value. E.g., `executorMemory` could be `null` if it is not configured by `--executor-memory`, `--conf spark.executor.memory`,  the properties files, and the relevant environment variable.

  - This method does not change the content of `sparkProperties`. 


```scala
...
/**
 * Parses and encapsulates arguments from the spark-submit script.
 * The env argument is used for testing.
 */
private[deploy] class SparkSubmitArguments(args: Seq[String], env: Map[String, String] = sys.env)
  extends SparkSubmitArgumentsParser with Logging {
  var maybeMaster: Option[String] = None
  // Global defaults. These should be keep to minimum to avoid confusing behavior.
  def master: String = maybeMaster.getOrElse("local[*]")
  var maybeRemote: Option[String] = None
  var deployMode: String = null
  var executorMemory: String = null
  var executorCores: String = null
  var totalExecutorCores: String = null
  var propertiesFile: String = null
  private var loadSparkDefaults: Boolean = false
  ...
  val sparkProperties: HashMap[String, String] = new HashMap[String, String]()
  ...

  // Set parameters from command line arguments
  parse(args.asJava)

  // Populate `sparkProperties` map from properties file
  mergeDefaultSparkProperties()
  // Remove keys that don't start with "spark." from `sparkProperties`.
  ignoreNonSparkProperties()
  // Use `sparkProperties` map along with env vars to fill in any missing parameters
  loadEnvironmentArguments()

  useRest = sparkProperties.getOrElse("spark.master.rest.enabled", "false").toBoolean

  validateArguments()

  ...
}

```


<br>

### [*scala/org/apache/spark/deploy/yarn/Client.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala)

<br>

```scala
...
import org.apache.spark.deploy.yarn.config._
...
import org.apache.spark.internal.config._
...
  ...
  private val amMemoryOverhead = {
    val amMemoryOverheadEntry = if (isClusterMode) DRIVER_MEMORY_OVERHEAD else AM_MEMORY_OVERHEAD
    sparkConf.get(amMemoryOverheadEntry).getOrElse(
      math.max((amMemoryOverheadFactor * amMemory).toLong,
        driverMinimumMemoryOverhead)).toInt
  }

  private val amCores = if (isClusterMode) {
    sparkConf.get(DRIVER_CORES)
  } else {
    sparkConf.get(AM_CORES)
  }

  // Executor related configurations
  private val executorMemory = sparkConf.get(EXECUTOR_MEMORY)
  // Executor offHeap memory in MiB.
  protected val executorOffHeapMemory = Utils.executorOffHeapMemorySizeAsMb(sparkConf)

  private val executorMemoryOvereadFactor = sparkConf.get(EXECUTOR_MEMORY_OVERHEAD_FACTOR)
  private val minMemoryOverhead = sparkConf.get(EXECUTOR_MIN_MEMORY_OVERHEAD)
  ...

```

-  Some capitalized names, such as `EXECUTOR_MEMORY`, `DRIVER_MEMORY_OVERHEAD`, `DRIVER_CORES`, etc., are `ConfigEntry` instances created in [*package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala), while others, such as `AM_MEMORY_OVERHEAD`, `AM_CORES`, etc., are `ConfigEntry` instances created in [*config.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/config.scala)
  
- `sparkConf` is present in [the signature of the primary constructor](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L66C1-L70C20) of class `Client` 

- `sparkConf.get(entry)`

    -  `get[T](entry: ConfigEntry[T])` is defined in [*SparkConf.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkConf.scala#L255C3-L264C4) as
      ```scala
      private[spark] def get[T](entry: ConfigEntry[T]): T = {
        entry.readFrom(reader)
      }
      ```
    - `reader` is defined in [*SparkConf.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkConf.scala#L65C3-L69C4) as follows:
      ```scala
      @transient private lazy val reader: ConfigReader = {
          val _reader = new ConfigReader(new SparkConfigProvider(settings))
          _reader.bindEnv((key: String) => Option(getenv(key)))
          _reader
      }
      ```

    - class `ConfigReader` is defined in [*ConfigReader.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigReader.scala)
      ```scala
      private[spark] class ConfigReader(conf: ConfigProvider) {
  
        def this(conf: JMap[String, String]) = this(new MapProvider(conf))
        ...
      ```

<br>


### [*scala/org/apache/spark/internal/config/ConfigEntry.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigEntry.scala)



<br>

There are multiple `ConfigEntry` classes.
  
- If `entry` is of type `ConfigEntryWithDefault`, [`readFrom(reader)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigEntry.scala#L140C3-L142C4) is defined as
  ```scala
  def readFrom(reader: ConfigReader): T = {
    readString(reader).map(valueConverter).getOrElse(_defaultValue)
  }
  ``` 


- If `entry` is of type `OptionalConfigEntry`: [`readFrom(reader)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigEntry.scala#L238C3-L240C4) is defined as:
  ```scala
  override def readFrom(reader: ConfigReader): Option[T] = {
    readString(reader).map(rawValueConverter)
  }
  ```
  
- Both `rawValueConverter` and `_defaultValue` are fields of a `ConfigEntry` instance.

- `readString(reader)`:  [readString(reader: ConfigReader)](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigEntry.scala#L91C3-L101C4) defined in *ConfigEntry.scala*
  ```scala
  protected def readString(reader: ConfigReader): Option[String] = {
    val values = Seq(
      prependedKey.flatMap(reader.get(_)),
      alternatives.foldLeft(reader.get(key))((res, nextKey) => res.orElse(reader.get(nextKey)))
    ).flatten
    if (values.nonEmpty) {
      Some(values.mkString(prependSeparator))
    } else {
      None
    }
  }

- `key: String, prependedKey`: Option[String], ..., alternatives: List[String]` are present in the signature of each ConfigEntry class

- The `alternatives` field is a list of strings that can be expanded with [`withAlternative(key: String)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigBuilder.scala#L245C3-L248C4)

- The `prependedKey` field can be assigned an `Option[String]` with [`withPrepended(key: String, separator: String = " ")`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigBuilder.scala#L239C3-L243C4) 

- `reader.get(key)` inside `foldLeft(reader.get(key))`

  - [`get(key: String)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigReader.scala#L79C3-L79C71) defined in *ConfigReader.scala* : read a configuration key from the default provider, and apply variable substitution
    ```scala
    def get(key: String): Option[String] = conf.get(key).map(substitute)
    ```

- If `values.nonEmpty` is `true`, return `Some(values.mkString(prependSeparator))` ; otherwise, return `None`

<br>


### [*scala/org/apache/spark/deploy/yarn/Client.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala)

<br>
 


    




<br>

 
### [*scala/org/apache/spark/internal/config/package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala)

<br>

Create application-related configuration entries

```scala
  ...
  private[spark] val DRIVER_CORES = ConfigBuilder("spark.driver.cores")
    .doc("Number of cores to use for the driver process, only in cluster mode.")
    .version("1.3.0")
    .intConf
    .createWithDefault(1)

  private[spark] val DRIVER_MEMORY = ConfigBuilder(SparkLauncher.DRIVER_MEMORY)
    .doc("Amount of memory to use for the driver process, in MiB unless otherwise specified.")
    .version("1.1.1")
    .bytesConf(ByteUnit.MiB)
    .createWithDefaultString("1g")

  private[spark] val DRIVER_MEMORY_OVERHEAD = ConfigBuilder("spark.driver.memoryOverhead")
    .doc("The amount of non-heap memory to be allocated per driver in cluster mode, " +
      "in MiB unless otherwise specified.")
    .version("2.3.0")
    .bytesConf(ByteUnit.MiB)
    .createOptional
    ...
```

> Constants defined in [java/org/apache/spark/launcher/SparkLauncher.java](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L39C3-L80C44) hold corresponding string values, e.g., `public static final String DRIVER_MEMORY = "spark.driver.memory";`.

|Entry Name| Key String | Type | Default |
|--|--|--|--|
|`SUBMIT_DEPLOY_MODE`| `"spark.submit.deployMode"`| `ConfigEntryWithDefault`| `"client"` |
|`DRIVER_CORES`| `"spark.driver.cores"`| `ConfigEntryWithDefault`| `1` |
|`DRIVER_MEMORY`| `"spark.driver.memory"`| `ConfigEntryWithDefault`| `"1g"` |
|`DRIVER_MEMORY_OVERHEAD`| `"spark.driver.memoryOverhead"`| `OptionalConfigEntry`|  | 
|`DRIVER_MIN_MEMORY_OVERHEAD`|`"spark.driver.minMemoryOverhead"`| `ConfigEntryWithDefault`| `"384m"` | 
|`DRIVER_MEMORY_OVERHEAD_FACTOR`| `"spark.driver.memoryOverheadFactor"`| `ConfigEntryWithDefault`| `0.1` | 
|`EXECUTOR_CORES` |`"spark.executor.cores"`| `ConfigEntryWithDefault`| `1` | 
|`EXECUTOR_MEMORY` |`"spark.executor.memory"`| `ConfigEntryWithDefault`| `"1g"` | 
|`EXECUTOR_MEMORY_OVERHEAD`| `"spark.executor.memoryOverhead"`| `OptionalConfigEntry`|  | 
|`EXECUTOR_MIN_MEMORY_OVERHEAD`|`"spark.executor.minMemoryOverhead"`| `ConfigEntryWithDefault`| `"384m"` | 
|`EXECUTOR_MEMORY_OVERHEAD_FACTOR`| `"spark.executor.memoryOverheadFactor"`| `ConfigEntryWithDefault`| `0.1` | 
|`MEMORY_OFFHEAP_ENABLED`| `"spark.memory.offHeap.enabled"`| `ConfigEntryWithDefault`| `false` | 
|`MEMORY_OFFHEAP_SIZE`| `"spark.memory.offHeap.size"`| `ConfigEntryWithDefault`| `0` | 
|`MEMORY_STORAGE_FRACTION`| `"spark.memory.storageFraction"`| `ConfigEntryWithDefault`| `0.5` | 
|`MEMORY_FRACTION`| `"spark.memory.fraction"`| `ConfigEntryWithDefault`| `0.6` | 
|`DYN_ALLOCATION_ENABLED`| `"spark.dynamicAllocation.enabled"`| `ConfigEntryWithDefault`| `false` | 



<br>

### [*scala/org/apache/spark/internal/config/Python.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/Python.scala)

|Entry Name| Key String | Type | Default |
|--|--|--|--|
|`PYSPARK_EXECUTOR_MEMORY`| `"spark.executor.pyspark.memory"`| `OptionalConfigEntry`|  | 


<br>


### [*scala/org/apache/spark/deploy/yarn/config.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/config.scala)

<br>

Create configuration entries specific to Spark on YARN.



|Entry Name| Key String | Type | Default |
|--|--|--|--|
|`AM_CORES`| `"spark.yarn.am.cores"`| `ConfigEntryWithDefault`| `1` |
|`AM_MEMORY_OVERHEAD`| `"spark.yarn.am.memoryOverhead"`| `OptionalConfigEntry`|  | 
|`AM_MEMORY`| `"spark.yarn.am.memory"`|  `ConfigEntryWithDefault`| `"512m"` |








<br>

### [*scala/org/apache/spark/internal/config/ConfigBuilder.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigBuilder.scala)

```scala

private object ConfigHelpers {

  def toNumber[T](s: String, converter: String => T, key: String, configType: String): T = {
    try {
      converter(s.trim)
    } catch {
      case _: NumberFormatException =>
        throw new IllegalArgumentException(s"$key should be $configType, but was $s")
    }
  }

  def toBoolean(s: String, key: String): Boolean = {
    try {
      s.trim.toBoolean
    } catch {
      case _: IllegalArgumentException =>
        throw new IllegalArgumentException(s"$key should be boolean, but was $s")
    }
  }

  def stringToSeq[T](str: String, converter: String => T): Seq[T] = {
    Utils.stringToSeq(str).map(converter)
  }

  def seqToString[T](v: Seq[T], stringConverter: T => String): String = {
    v.map(stringConverter).mkString(",")
  }

  def timeFromString(str: String, unit: TimeUnit): Long = JavaUtils.timeStringAs(str, unit)

  def timeToString(v: Long, unit: TimeUnit): String = s"${TimeUnit.MILLISECONDS.convert(v, unit)}ms"

  def byteFromString(str: String, unit: ByteUnit): Long = {
    val (input, multiplier) =
      if (str.length() > 0 && str.charAt(0) == '-') {
        (str.substring(1), -1)
      } else {
        (str, 1)
      }
    multiplier * JavaUtils.byteStringAs(input, unit)
  }

  def byteToString(v: Long, unit: ByteUnit): String = s"${unit.convertTo(v, ByteUnit.BYTE)}b"

  def regexFromString(str: String, key: String): Regex = {
    try str.r catch {
      case e: PatternSyntaxException =>
        throw new IllegalArgumentException(s"$key should be a regex, but was $str", e)
    }
  }

}

/**
 * A type-safe config builder. Provides methods for transforming the input data (which can be
 * used, e.g., for validation) and creating the final config entry.
 *
 * One of the methods that return a [[ConfigEntry]] must be called to create a config entry that
 * can be used with [[SparkConf]].
 */
private[spark] class TypedConfigBuilder[T](
  ...
  /** Creates a [[ConfigEntry]] that does not have a default value. */
  def createOptional: OptionalConfigEntry[T] = {
    val entry = new OptionalConfigEntry[T](parent.key, parent._prependedKey,
      parent._prependSeparator, parent._alternatives, converter, stringConverter, parent._doc,
      parent._public, parent._version)
    parent._onCreate.foreach(_(entry))
    entry
  }

  /** Creates a [[ConfigEntry]] that has a default value. */
  def createWithDefault(default: T): ConfigEntry[T] = {
    assert(default != null, "Use createOptional.")
    // Treat "String" as a special case, so that both createWithDefault and createWithDefaultString
    // behave the same w.r.t. variable expansion of default values.
    default match {
      case str: String => createWithDefaultString(str)
      case _ =>
        val transformedDefault = converter(stringConverter(default))
        val entry = new ConfigEntryWithDefault[T](parent.key, parent._prependedKey,
          parent._prependSeparator, parent._alternatives, transformedDefault, converter,
          stringConverter, parent._doc, parent._public, parent._version)
        parent._onCreate.foreach(_ (entry))
        entry
    }
  }
  ...
}


/**
 * Basic builder for Spark configurations. Provides methods for creating type-specific builders.
 *
 * @see TypedConfigBuilder
 */
private[spark] case class ConfigBuilder(key: String) {

  import ConfigHelpers._

  private[config] var _prependedKey: Option[String] = None
  private[config] var _prependSeparator: String = ""
  private[config] var _public = true
  private[config] var _doc = ""
  private[config] var _version = ""
  private[config] var _onCreate: Option[ConfigEntry[_] => Unit] = None
  private[config] var _alternatives = List.empty[String]

  def internal(): ConfigBuilder = {
    _public = false
    this
  }

  def doc(s: String): ConfigBuilder = {
    _doc = s
    this
  }

  def version(v: String): ConfigBuilder = {
    _version = v
    this
  }

  /**
   * Registers a callback for when the config entry is finally instantiated. Currently used by
   * SQLConf to keep track of SQL configuration entries.
   */
  def onCreate(callback: ConfigEntry[_] => Unit): ConfigBuilder = {
    _onCreate = Option(callback)
    this
  }

  def withPrepended(key: String, separator: String = " "): ConfigBuilder = {
    _prependedKey = Option(key)
    _prependSeparator = separator
    this
  }

  def withAlternative(key: String): ConfigBuilder = {
    _alternatives = _alternatives :+ key
    this
  }

  def intConf: TypedConfigBuilder[Int] = {
    checkPrependConfig
    new TypedConfigBuilder(this, toNumber(_, _.toInt, key, "int"))
  }

  def longConf: TypedConfigBuilder[Long] = {
    checkPrependConfig
    new TypedConfigBuilder(this, toNumber(_, _.toLong, key, "long"))
  }

  def doubleConf: TypedConfigBuilder[Double] = {
    checkPrependConfig
    new TypedConfigBuilder(this, toNumber(_, _.toDouble, key, "double"))
  }

  def booleanConf: TypedConfigBuilder[Boolean] = {
    checkPrependConfig
    new TypedConfigBuilder(this, toBoolean(_, key))
  }

  def stringConf: TypedConfigBuilder[String] = {
    new TypedConfigBuilder(this, v => v)
  }

  def timeConf(unit: TimeUnit): TypedConfigBuilder[Long] = {
    checkPrependConfig
    new TypedConfigBuilder(this, timeFromString(_, unit), timeToString(_, unit))
  }

  def bytesConf(unit: ByteUnit): TypedConfigBuilder[Long] = {
    checkPrependConfig
    new TypedConfigBuilder(this, byteFromString(_, unit), byteToString(_, unit))
  }

  def fallbackConf[T](fallback: ConfigEntry[T]): ConfigEntry[T] = {
    val entry = new FallbackConfigEntry(key, _prependedKey, _prependSeparator, _alternatives, _doc,
      _public, _version, fallback)
    _onCreate.foreach(_(entry))
    entry
  }

  def regexConf: TypedConfigBuilder[Regex] = {
    checkPrependConfig
    new TypedConfigBuilder(this, regexFromString(_, this.key), _.toString)
  }

  private def checkPrependConfig = {
    if (_prependedKey.isDefined) {
      throw new IllegalArgumentException(s"$key type must be string if prepend used")
    }
  }
}
```

<br>

### [*scala/org/apache/spark/SparkConf.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkConf.scala)

```scala

class SparkConf(loadDefaults: Boolean) extends Cloneable with Logging with Serializable {

  import SparkConf._

  /** Create a SparkConf that loads defaults from system properties and the classpath */
  def this() = this(true)

  private val settings = new ConcurrentHashMap[String, String]()

  @transient private lazy val reader: ConfigReader = {
    val _reader = new ConfigReader(new SparkConfigProvider(settings))
    _reader.bindEnv((key: String) => Option(getenv(key)))
    _reader
  }

  if (loadDefaults) {
    loadFromSystemProperties(false)
  }

  private[spark] def loadFromSystemProperties(silent: Boolean): SparkConf = {
    // Load any spark.* system properties
    for ((key, value) <- Utils.getSystemProperties if key.startsWith("spark.")) {
      set(key, value, silent)
    }
    this
  }

  ...

  /**
   * Retrieves the value of a pre-defined configuration entry.
   *
   * - This is an internal Spark API.
   * - The return type if defined by the configuration entry.
   * - This will throw an exception is the config is not optional and the value is not set.
   */
  private[spark] def get[T](entry: ConfigEntry[T]): T = {
    entry.readFrom(reader)
  }
  ...

  /**
   * By using this instead of System.getenv(), environment variables can be mocked
   * in unit tests.
   */
  private[spark] def getenv(name: String): String = System.getenv(name)

  ...

  /** Does the configuration contain a given parameter? */
  def contains(key: String): Boolean = {
    settings.containsKey(key) ||
      configsWithAlternatives.get(key).toSeq.flatten.exists { alt => contains(alt.key) }
  }

  private[spark] def contains(entry: ConfigEntry[_]): Boolean = contains(entry.key)

}
```

<br>

### [*scala/org/apache/spark/internal/config/ConfigEntry.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigEntry.scala)


```scala
private[spark] abstract class ConfigEntry[T] (
    val key: String,
    val prependedKey: Option[String],
    val prependSeparator: String,
    val alternatives: List[String],
    val valueConverter: String => T,
    val stringConverter: T => String,
    val doc: String,
    val isPublic: Boolean,
    val version: String) {

  import ConfigEntry._

  registerEntry(this)

  def defaultValueString: String

  protected def readString(reader: ConfigReader): Option[String] = {
    val values = Seq(
      prependedKey.flatMap(reader.get(_)),
      alternatives.foldLeft(reader.get(key))((res, nextKey) => res.orElse(reader.get(nextKey)))
    ).flatten
    if (values.nonEmpty) {
      Some(values.mkString(prependSeparator))
    } else {
      None
    }
  }

  def readFrom(reader: ConfigReader): T

  def defaultValue: Option[T] = None

  ...

}

private class ConfigEntryWithDefault[T] (
    key: String,
    prependedKey: Option[String],
    prependSeparator: String,
    alternatives: List[String],
    _defaultValue: T,
    valueConverter: String => T,
    stringConverter: T => String,
    doc: String,
    isPublic: Boolean,
    version: String)
  extends ConfigEntry(
    key,
    prependedKey,
    prependSeparator,
    alternatives,
    valueConverter,
    stringConverter,
    doc,
    isPublic,
    version
  ) {

  override def defaultValue: Option[T] = Some(_defaultValue)

  override def defaultValueString: String = stringConverter(_defaultValue)

  def readFrom(reader: ConfigReader): T = {
    readString(reader).map(valueConverter).getOrElse(_defaultValue)
  }
}

...
/**
 * A config entry that does not have a default value.
 */
private[spark] class OptionalConfigEntry[T](
    key: String,
    prependedKey: Option[String],
    prependSeparator: String,
    alternatives: List[String],
    val rawValueConverter: String => T,
    val rawStringConverter: T => String,
    doc: String,
    isPublic: Boolean,
    version: String)
  extends ConfigEntry[Option[T]](
    key,
    prependedKey,
    prependSeparator,
    alternatives,
    s => Some(rawValueConverter(s)),
    v => v.map(rawStringConverter).orNull,
    doc,
    isPublic,
    version
  ) {

  override def defaultValueString: String = ConfigEntry.UNDEFINED

  override def readFrom(reader: ConfigReader): Option[T] = {
    readString(reader).map(rawValueConverter)
  }
}
```

<br>

### [*scala/org/apache/spark/internal/config/ConfigReader.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigReader.scala)

```scala
/**
 * A helper class for reading config entries and performing variable substitution.
 *
 * If a config value contains variable references of the form "${prefix:variableName}", the
 * reference will be replaced with the value of the variable depending on the prefix. By default,
 * the following prefixes are handled:
 *
 * - no prefix: use the default config provider
 * - system: looks for the value in the system properties
 * - env: looks for the value in the environment
 *
 * Different prefixes can be bound to a `ConfigProvider`, which is used to read configuration
 * values from the data source for the prefix, and both the system and env providers can be
 * overridden.
 *
 * If the reference cannot be resolved, the original string will be retained.
 *
 * @param conf The config provider for the default namespace (no prefix).
 */
private[spark] class ConfigReader(conf: ConfigProvider) {

  def this(conf: JMap[String, String]) = this(new MapProvider(conf))
  ...
  /**
   * Reads a configuration key from the default provider, and apply variable substitution.
   */
  def get(key: String): Option[String] = conf.get(key).map(substitute)

  /**
   * Perform variable substitution on the given input string.
   */
  def substitute(input: String): String = substitute(input, Set())
  ...
}
```

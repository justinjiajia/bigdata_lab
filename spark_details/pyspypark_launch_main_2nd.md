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


- If so, `AbstractCommandBuilder builder = new SparkSubmitCommandBuilder(args);`, which creates a `SparkSubmitCommandBuilder` instance with the remaining command line options (i.e., `pyspark-shell-main --name "PySparkShell" "$@"`).

  - The instance constructor contains [case matching code](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitCommandBuilder.java#L131C9-L147C8) that assigns values to two fields:
    ```java
    appResource = PYSPARK_SHELL;
    submitArgs = args.subList(1, args.size());
    ```
    when the 1st element in `args` equals `pyspark-shell-main`. Note `PYSPARK_SHELL` is a static constant equal to `"pyspark-shell-main"`.

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
    
  - Get the 1st argument for case maching: 
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
      
       -  [`getLibPathEnvName()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L90C3-L102C4): return `"LD_LIBRARY_PATH"` because `System.getProperty("os.name")` returns `Linux` on an EMR instance.  the the name of the env variable that holds the native library path.
         
       -  [`getEffectiveConfig()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L274C3-L284C4)
         - [loadPropertiesFile()](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/AbstractCommandBuilder.java#L286C3-L311C4): load configurations from a file specified via the command line option `--properties-file` or the *spark-defaults.conf* file under the Spark configuration directory. [`DEFAULT_PROPERTIES_FILE`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L31) is a constant with the value of `"spark-defaults.conf"`.
     
         - Return a `HashMap<>` with all configurations loaded from the properties file and an entry with the key `"spark.driver.defaultExtraClassPath"` and the value `"hive-jackson/*"` if no such an entry is specified in the properties file.
     
            - No entry with the name `"spark.driver.defaultExtraClassPath"` in */usr/lib/spark/conf/spark-defaults.conf* on an EMR instance.
     
     - [`mergeEnvPathList(Map<String, String> userEnv, String envKey, String pathList)`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/CommandBuilderUtils.java#L110C3-L119C4): append `"hive-jackson/*"` to the first nom-empty value between the entry named `"LD_LIBRARY_PATH"` in `HashMap` `env` and the same-name environment variable, and write the prolonged path to the user environment `env`
   
        - Now, `env` contains the 1st entry with the key `LD_LIBRARY_PATH` and the value `"hive-jackson/*"`


    - `buildSparkSubmitArgs()`: add a restricted set of options in a particular order (e.g., `--master`, `--remote`, `--deploy-mode`, etc.) to an `ArrayList<>`; then add all configurations `conf` contains to the same list as pairs of `"--conf"` and `"<key string>=<value>"`. So driver-related properties set via options such as `--driver-memory` get translated to pairs of `"--conf" and "spark.driver.memory=<value>"; then add all configurations maintained by `parsedArgs`; lastly, add `"pyspark-shell"`
      
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

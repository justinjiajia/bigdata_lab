

https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala

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
  var driverMemory: String = null
  var driverExtraClassPath: String = null
  var driverExtraLibraryPath: String = null
  var driverExtraJavaOptions: String = null
  var queue: String = null
  var numExecutors: String = null
  var files: String = null
  var archives: String = null
  var mainClass: String = null
  var primaryResource: String = null
  var name: String = null
  var childArgs: ArrayBuffer[String] = new ArrayBuffer[String]()
  var jars: String = null
  var packages: String = null
  var repositories: String = null
  var ivyRepoPath: String = null
  var ivySettingsPath: Option[String] = None
  var packagesExclusions: String = null
  var verbose: Boolean = false
  var isPython: Boolean = false
  var pyFiles: String = null
  var isR: Boolean = false
  var action: SparkSubmitAction = null
  val sparkProperties: HashMap[String, String] = new HashMap[String, String]()
  var proxyUser: String = null
  var principal: String = null
  var keytab: String = null
  private var dynamicAllocationEnabled: Boolean = false
  // Standalone cluster mode only
  var supervise: Boolean = false
  var driverCores: String = null
  var submissionToKill: String = null
  var submissionToRequestStatusFor: String = null
  var useRest: Boolean = false // used internally

  override protected def logName: String = classOf[SparkSubmitArguments].getName

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

  /**
   * Load properties from the file with the given path into `sparkProperties`.
   * No-op if the file path is null
   */
  private def loadPropertiesFromFile(filePath: String): Unit = {
    if (filePath != null) {
      if (verbose) {
        logInfo(log"Using properties file: ${MDC(PATH, filePath)}")
      }
      val properties = Utils.getPropertiesFromFile(filePath)
      properties.foreach { case (k, v) =>
        if (!sparkProperties.contains(k)) {
          sparkProperties(k) = v
        }
      }
      // Property files may contain sensitive information, so redact before printing
      if (verbose) {
        Utils.redact(properties).foreach { case (k, v) =>
          logInfo(log"Adding default property: ${MDC(KEY, k)}=${MDC(VALUE, v)}")
        }
      }
    }
  }

  /**
   * Merge values from the default properties file with those specified through --conf.
   * When this is called, `sparkProperties` is already filled with configs from the latter.
   */
  private def mergeDefaultSparkProperties(): Unit = {
    // Honor --conf before the specified properties file and defaults file
    loadPropertiesFromFile(propertiesFile)

    // Also load properties from `spark-defaults.conf` if they do not exist in the properties file
    // and --conf list when:
    //   - no input properties file is specified
    //   - input properties file is specified, but `--load-spark-defaults` flag is set
    if (propertiesFile == null || loadSparkDefaults) {
      loadPropertiesFromFile(Utils.getDefaultPropertiesFile(env))
    }
  }

  /**
   * Remove keys that don't start with "spark." from `sparkProperties`.
   */
  private def ignoreNonSparkProperties(): Unit = {
    sparkProperties.keys.foreach { k =>
      if (!k.startsWith("spark.")) {
        sparkProperties -= k
        logWarning(log"Ignoring non-Spark config property: ${MDC(CONFIG, k)}")
      }
    }
  }

  /**
   * Load arguments from environment variables, Spark properties etc.
   */
  private def loadEnvironmentArguments(): Unit = {
    maybeMaster = maybeMaster
      .orElse(sparkProperties.get("spark.master"))
      .orElse(env.get("MASTER"))
    maybeRemote = maybeRemote
      .orElse(sparkProperties.get("spark.remote"))
      .orElse(env.get("SPARK_REMOTE"))

    driverExtraClassPath = Option(driverExtraClassPath)
      .orElse(sparkProperties.get(config.DRIVER_CLASS_PATH.key))
      .orNull
    driverExtraJavaOptions = Option(driverExtraJavaOptions)
      .orElse(sparkProperties.get(config.DRIVER_JAVA_OPTIONS.key))
      .orNull
    driverExtraLibraryPath = Option(driverExtraLibraryPath)
      .orElse(sparkProperties.get(config.DRIVER_LIBRARY_PATH.key))
      .orNull
    driverMemory = Option(driverMemory)
      .orElse(sparkProperties.get(config.DRIVER_MEMORY.key))
      .orElse(env.get("SPARK_DRIVER_MEMORY"))
      .orNull
    driverCores = Option(driverCores)
      .orElse(sparkProperties.get(config.DRIVER_CORES.key))
      .orNull
    executorMemory = Option(executorMemory)
      .orElse(sparkProperties.get(config.EXECUTOR_MEMORY.key))
      .orElse(env.get("SPARK_EXECUTOR_MEMORY"))
      .orNull
    executorCores = Option(executorCores)
      .orElse(sparkProperties.get(config.EXECUTOR_CORES.key))
      .orElse(env.get("SPARK_EXECUTOR_CORES"))
      .orNull
    totalExecutorCores = Option(totalExecutorCores)
      .orElse(sparkProperties.get(config.CORES_MAX.key))
      .orNull
    name = Option(name).orElse(sparkProperties.get("spark.app.name")).orNull
    jars = Option(jars).orElse(sparkProperties.get(config.JARS.key)).orNull
    files = Option(files).orElse(sparkProperties.get(config.FILES.key)).orNull
    archives = Option(archives).orElse(sparkProperties.get(config.ARCHIVES.key)).orNull
    pyFiles = Option(pyFiles).orElse(sparkProperties.get(config.SUBMIT_PYTHON_FILES.key)).orNull
    ivyRepoPath = sparkProperties.get(config.JAR_IVY_REPO_PATH.key).orNull
    ivySettingsPath = sparkProperties.get(config.JAR_IVY_SETTING_PATH.key)
    packages = Option(packages).orElse(sparkProperties.get(config.JAR_PACKAGES.key)).orNull
    packagesExclusions = Option(packagesExclusions)
      .orElse(sparkProperties.get(config.JAR_PACKAGES_EXCLUSIONS.key)).orNull
    repositories = Option(repositories)
      .orElse(sparkProperties.get(config.JAR_REPOSITORIES.key)).orNull
    deployMode = Option(deployMode)
      .orElse(sparkProperties.get(config.SUBMIT_DEPLOY_MODE.key))
      .orElse(env.get("DEPLOY_MODE"))
      .orNull
    numExecutors = Option(numExecutors)
      .getOrElse(sparkProperties.get(config.EXECUTOR_INSTANCES.key).orNull)
    queue = Option(queue).orElse(sparkProperties.get("spark.yarn.queue")).orNull
    keytab = Option(keytab)
      .orElse(sparkProperties.get(config.KEYTAB.key))
      .orElse(sparkProperties.get("spark.yarn.keytab"))
      .orNull
    principal = Option(principal)
      .orElse(sparkProperties.get(config.PRINCIPAL.key))
      .orElse(sparkProperties.get("spark.yarn.principal"))
      .orNull
    dynamicAllocationEnabled =
      sparkProperties.get(DYN_ALLOCATION_ENABLED.key).exists("true".equalsIgnoreCase)

    // In YARN mode, app name can be set via SPARK_YARN_APP_NAME (see SPARK-5222)
    if (master.startsWith("yarn")) {
      name = Option(name).orElse(env.get("SPARK_YARN_APP_NAME")).orNull
    }

    // Set name from main class if not given
    name = Option(name).orElse(Option(mainClass)).orNull
    if (name == null && primaryResource != null) {
      name = new File(primaryResource).getName()
    }

    // Action should be SUBMIT unless otherwise specified
    action = Option(action).getOrElse(SUBMIT)
  }

  ...
  /** Fill in values by parsing user options. */
  override protected def handle(opt: String, value: String): Boolean = {
    opt match {
      case NAME =>
        name = value

      case MASTER =>
        maybeMaster = Option(value)

      case REMOTE =>
        maybeRemote = Option(value)

      case CLASS =>
        mainClass = value

      case DEPLOY_MODE =>
        if (value != "client" && value != "cluster") {
          error("--deploy-mode must be either \"client\" or \"cluster\"")
        }
        deployMode = value

      case NUM_EXECUTORS =>
        numExecutors = value

      case TOTAL_EXECUTOR_CORES =>
        totalExecutorCores = value

      case EXECUTOR_CORES =>
        executorCores = value

      case EXECUTOR_MEMORY =>
        executorMemory = value

      case DRIVER_MEMORY =>
        driverMemory = value

      case DRIVER_CORES =>
        driverCores = value

      case DRIVER_CLASS_PATH =>
        driverExtraClassPath = value

      case DRIVER_JAVA_OPTIONS =>
        driverExtraJavaOptions = value

      case DRIVER_LIBRARY_PATH =>
        driverExtraLibraryPath = value

      case PROPERTIES_FILE =>
        propertiesFile = value

      case LOAD_SPARK_DEFAULTS =>
        loadSparkDefaults = true

      case KILL_SUBMISSION =>
        submissionToKill = value
        if (action != null) {
          error(s"Action cannot be both $action and $KILL.")
        }
        action = KILL

      case STATUS =>
        submissionToRequestStatusFor = value
        if (action != null) {
          error(s"Action cannot be both $action and $REQUEST_STATUS.")
        }
        action = REQUEST_STATUS

      case SUPERVISE =>
        supervise = true

      case QUEUE =>
        queue = value

      case FILES =>
        files = Utils.resolveURIs(value)

      case PY_FILES =>
        pyFiles = Utils.resolveURIs(value)

      case ARCHIVES =>
        archives = Utils.resolveURIs(value)

      case JARS =>
        jars = Utils.resolveURIs(value)

      case PACKAGES =>
        packages = value

      case PACKAGES_EXCLUDE =>
        packagesExclusions = value

      case REPOSITORIES =>
        repositories = value

      case CONF =>
        val (confName, confValue) = SparkSubmitUtils.parseSparkConfProperty(value)
        sparkProperties(confName) = confValue

      case PROXY_USER =>
        proxyUser = value

      case PRINCIPAL =>
        principal = value

      case KEYTAB =>
        keytab = value

      case HELP =>
        printUsageAndExit(0)

      case VERBOSE =>
        verbose = true

      case VERSION =>
        action = SparkSubmitAction.PRINT_VERSION

      case USAGE_ERROR =>
        printUsageAndExit(1)

      case _ =>
        error(s"Unexpected argument '$opt'.")
    }
    action != SparkSubmitAction.PRINT_VERSION
  }

```


`parse()` defined in [SparkSubmitOptionParser.java](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L137C1-L193C4)

  - If the option name is in a predefined two-level list (`opts`), 
  `handle()` defined in [SparkSubmitOptionParser.java](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L349C1-L473C4)
  assigns the option value to the corrsponding variable declared in the beginning

  ```
      case EXECUTOR_CORES =>
        executorCores = value

      case EXECUTOR_MEMORY =>
        executorMemory = value
  ```
 
 - For options defined via `--conf` or `-c`, e.g., `--conf spark.eventLog.enabled=false
    --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps"`, the processing defined in `handle()` is a bit different:
  
  ```
        case CONF =>
          val (confName, confValue) = SparkSubmitUtils.parseSparkConfProperty(value)
          sparkProperties(confName) = confValue
  ```
  where `sparkProperties` is an empty `HashMap[String, String]` initially.

 - After `parse()`, `sparkProperties` is already filled with properties specified through `--conf`


`mergeDefaultSparkProperties()` defined in [SparkSubmitArguments.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L129C1-L144C4)


- When `--properties-file` was used to specify a properties file (so `propertiesFile` is not `null`), merge values from that file with those specified through `--conf` in `sparkProperties`.

    - `loadPropertiesFromFile(propertiesFile)`

- When no input properties file is specified via `--properties-file` or when `--load-spark-defaults` flag is set, load properties from `spark-defaults.conf`

    - `loadPropertiesFromFile(Utils.getDefaultPropertiesFile(env))` 

- `loadPropertiesFromFile(filePath: String)` only addes a new entry to `sparkProperties`
  ```scala
  val properties = Utils.getPropertiesFromFile(filePath)
  properties.foreach { case (k, v) =>
        if (!sparkProperties.contains(k)) {
          sparkProperties(k) = v
        }
  }
  ```

- so the precedence is as follows: `--conf` > properties in a file specified via  `--properties-file` > properties in file `spark-defaults.conf`


`loadEnvironmentArguments()`: * Load arguments from environment variables, Spark properties etc.

E.g,, if `executorMemory` is still `null` (e.g., hasn't be set via the `--executor-memory` flag), try to load the value associated with the key `"spark.executor.memory"` from `sparkProperties` first; if there's no such a key in `sparkProperties`, 

`private[deploy] class SparkSubmitArguments(args: Seq[String], env: Map[String, String] = sys.env)`

```
    executorMemory = Option(executorMemory)
      .orElse(sparkProperties.get(config.EXECUTOR_MEMORY.key))
      .orElse(env.get("SPARK_EXECUTOR_MEMORY"))
      .orNull
    executorCores = Option(executorCores)
      .orElse(sparkProperties.get(config.EXECUTOR_CORES.key))
      .orElse(env.get("SPARK_EXECUTOR_CORES"))
      .orNull
```


https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/launcher/SparkSubmitArgumentsParser.scala

```java
package org.apache.spark.launcher

/**
 * This class makes SparkSubmitOptionParser visible for Spark code outside of the `launcher`
 * package, since Java doesn't have a feature similar to `private[spark]`, and we don't want
 * that class to be public.
 */
private[spark] abstract class SparkSubmitArgumentsParser extends SparkSubmitOptionParser
```

https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java

```java
/**
 * Parser for spark-submit command line options.
 * <p>
 * This class encapsulates the parsing code for spark-submit command line options, so that there
 * is a single list of options that needs to be maintained (well, sort of, but it makes it harder
 * to break things).
 */
class SparkSubmitOptionParser {

  // The following constants define the "main" name for the available options. They're defined
  // to avoid copy & paste of the raw strings where they're needed.
  //
  // The fields are not static so that they're exposed to Scala code that uses this class. See
  // SparkSubmitArguments.scala. That is also why this class is not abstract - to allow code to
  // easily use these constants without having to create dummy implementations of this class.
  protected final String CLASS = "--class";
  protected final String CONF = "--conf";
  protected final String DEPLOY_MODE = "--deploy-mode";
  protected final String DRIVER_CLASS_PATH = "--driver-class-path";
  protected final String DRIVER_DEFAULT_CLASS_PATH = "--driver-default-class-path";
  protected final String DRIVER_CORES = "--driver-cores";
  protected final String DRIVER_JAVA_OPTIONS =  "--driver-java-options";
  protected final String DRIVER_LIBRARY_PATH = "--driver-library-path";
  protected final String DRIVER_MEMORY = "--driver-memory";
  protected final String EXECUTOR_MEMORY = "--executor-memory";
  protected final String FILES = "--files";
  protected final String JARS = "--jars";
  protected final String KILL_SUBMISSION = "--kill";
  protected final String MASTER = "--master";
  protected final String REMOTE = "--remote";
  protected final String NAME = "--name";
  protected final String PACKAGES = "--packages";
  protected final String PACKAGES_EXCLUDE = "--exclude-packages";
  protected final String PROPERTIES_FILE = "--properties-file";
  protected final String LOAD_SPARK_DEFAULTS = "--load-spark-defaults";
  protected final String PROXY_USER = "--proxy-user";
  protected final String PY_FILES = "--py-files";
  protected final String REPOSITORIES = "--repositories";
  protected final String STATUS = "--status";
  protected final String TOTAL_EXECUTOR_CORES = "--total-executor-cores";

  // Options that do not take arguments.
  protected final String HELP = "--help";
  protected final String SUPERVISE = "--supervise";
  protected final String USAGE_ERROR = "--usage-error";
  protected final String VERBOSE = "--verbose";
  protected final String VERSION = "--version";

  // Standalone-only options.

  // YARN-only options.
  protected final String ARCHIVES = "--archives";
  protected final String EXECUTOR_CORES = "--executor-cores";
  protected final String KEYTAB = "--keytab";
  protected final String NUM_EXECUTORS = "--num-executors";
  protected final String PRINCIPAL = "--principal";
  protected final String QUEUE = "--queue";

  ...

  /**
   * Parse a list of spark-submit command line options.
   * <p>
   * See SparkSubmitArguments.scala for a more formal description of available options.
   *
   * @throws IllegalArgumentException If an error is found during parsing.
   */
  protected final void parse(List<String> args) {
    Pattern eqSeparatedOpt = Pattern.compile("(--[^=]+)=(.+)");

    int idx = 0;
    for (idx = 0; idx < args.size(); idx++) {
      String arg = args.get(idx);
      String value = null;

      Matcher m = eqSeparatedOpt.matcher(arg);
      if (m.matches()) {
        arg = m.group(1);
        value = m.group(2);
      }

      // Look for options with a value.
      String name = findCliOption(arg, opts);
      if (name != null) {
        if (value == null) {
          if (idx == args.size() - 1) {
            throw new IllegalArgumentException(
                String.format("Missing argument for option '%s'.", arg));
          }
          idx++;
          value = args.get(idx);
        }
        if (!handle(name, value)) {
          break;
        }
        continue;
      }

      // Look for a switch.
      name = findCliOption(arg, switches);
      if (name != null) {
        if (!handle(name, null)) {
          break;
        }
        continue;
      }

      if (!handleUnknown(arg)) {
        break;
      }
    }

    if (idx < args.size()) {
      idx++;
    }
    handleExtraArgs(args.subList(idx, args.size()));
  }



```

`sparkConf.get(entry)` in *Client.scala* -> `entry.readFrom(reader)` in *SparkConf.scala*

- if `entry` is of type `OptionalConfigEntry`: `entry.readFrom(reader)` -> `readString(reader).map(rawValueConverter)`
  
- If `entry` is of type `ConfigEntryWithDefault`: `entry.readFrom(reader)` -> `readString(reader).map(valueConverter).getOrElse(_defaultValue)`

- Note both `rawValueConverter` and `_defaultValue` are names used in the signatures of the respective classes 


`readString(reader)` ->  `reader.get(key)` -> `conf.get(key).map(substitute)`

-  If no such a `key` (of type `String`), return `None`; otherwise, return `Some(values.mkString(prependSeparator))`
    




<br>

 
### [*scala/org/apache/spark/internal/config/package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala)

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

> Variables defined in [java/org/apache/spark/launcher/SparkLauncher.java](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkLauncher.java), e.g., `DRIVER_MEMORY`, hold corresponding string values.

|Entry Name| Key String | Type | Default |
|--|--|--|--|
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


### [scala/org/apache/spark/deploy/yarn/config.scala](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/config.scala)

Create configuration entries specific to Spark on YARN.



|Entry Name| Key String | Type | Default |
|--|--|--|--|
|`AM_CORES`| `"spark.yarn.am.cores"`| `ConfigEntryWithDefault`| `1` |
|`AM_MEMORY_OVERHEAD`| `"spark.yarn.am.memoryOverhead"`| `OptionalConfigEntry`|  | 
|`AM_MEMORY`| `"spark.yarn.am.memory"`|  `ConfigEntryWithDefault`| `"512m"` |






<br>

### [*scala/org/apache/spark/deploy/yarn/Client.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala)

```scala
  ...
  private val amMemoryOverhead = {
    val amMemoryOverheadEntry = if (isClusterMode) DRIVER_MEMORY_OVERHEAD else AM_MEMORY_OVERHEAD
    sparkConf.get(amMemoryOverheadEntry).getOrElse(
      math.max((amMemoryOverheadFactor * amMemory).toLong,
        driverMinimumMemoryOverhead)).toInt
  }
```

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

### [scala/org/apache/spark/internal/config/ConfigReader.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/ConfigReader.scala)

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

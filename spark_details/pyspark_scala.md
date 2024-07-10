```shell
env
LD_LIBRARY_PATH=/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib:/usr/lib/jvm/java-17-amazon-corretto.x86_64/../lib:/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server
/usr/lib/jvm/jre-17/bin/java
-cp
/usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/usr/lib/spark/conf/:/usr/lib/spark/jars/*:/etc/hadoop/conf/
-DAWS_ACCOUNT_ID=688430810480
-DEMR_CLUSTER_ID=j-3JZ8WOC269WHI
-DEMR_RELEASE_LABEL=emr-7.1.0
-DAWS_ACCOUNT_ID=688430810480
-DEMR_CLUSTER_ID=j-3JZ8WOC269WHI
-DEMR_RELEASE_LABEL=emr-7.1.0
-Xmx2g
-XX:OnOutOfMemoryError=kill -9 %p
-XX:+IgnoreUnrecognizedVMOptions
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
org.apache.spark.deploy.SparkSubmit
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

<br>


###  [*scala/org/apache/spark/deploy/SparkSubmit.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)

<br>


- In Scala, `main` must be a method in an object.  It is the entry point to the application. The command-line arguments for the application are passed to main in an array of strings, e.g., `args: Array[String]`.
  ```scala
  override def main(args: Array[String]): Unit = {
    Option(System.getenv("SPARK_PREFER_IPV6"))
      .foreach(System.setProperty("java.net.preferIPv6Addresses", _))
    val submit = new SparkSubmit() {
      self =>

      override protected def parseArguments(args: Array[String]): SparkSubmitArguments = {
        new SparkSubmitArguments(args.toImmutableArraySeq) {
          override protected def logInfo(msg: => String): Unit = self.logInfo(msg)

          override protected def logInfo(entry: LogEntry): Unit = self.logInfo(entry)

          override protected def logWarning(msg: => String): Unit = self.logWarning(msg)

          override protected def logWarning(entry: LogEntry): Unit = self.logWarning(entry)

          override protected def logError(msg: => String): Unit = self.logError(msg)

          override protected def logError(entry: LogEntry): Unit = self.logError(entry)
        }
      }

      override protected def logInfo(msg: => String): Unit = printMessage(msg)

      override protected def logInfo(entry: LogEntry): Unit = printMessage(entry.message)

      override protected def logWarning(msg: => String): Unit = printMessage(s"Warning: $msg")

      override protected def logWarning(entry: LogEntry): Unit =
        printMessage(s"Warning: ${entry.message}")

      override protected def logError(msg: => String): Unit = printMessage(s"Error: $msg")

      override protected def logError(entry: LogEntry): Unit =
        printMessage(s"Error: ${entry.message}")

      override def doSubmit(args: Array[String]): Unit = {
        try {
          super.doSubmit(args)
        } catch {
          case e: SparkUserAppException =>
            exitFn(e.exitCode)
        }
      }

    }

    submit.doSubmit(args)
  }
  ```
  -  `val submit = new SparkSubmit() { ... }` instantiates an anonymous class subclassed from the `SparkSubmit` class
  - `self =>` is used to alias `this`.
  - `submit.doSubmit(args)`
    ```scala
    override def doSubmit(args: Array[String]): Unit = {
        try {
          super.doSubmit(args)
        } catch {
          ...
        }
      }
    ```

    - `super.doSubmit(args)` invokes `doSubmit()` defined in the [body of the `SparkSubmit` class](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala#L71C3-L101C4)
   
      ```scala
      def doSubmit(args: Array[String]): Unit = {
        val appArgs = parseArguments(args)
        val sparkConf = appArgs.toSparkConf()
    
        // For interpreters, structured logging is disabled by default to avoid generating mixed
        // plain text and structured logs on the same console.
        if (isShell(appArgs.primaryResource) || isSqlShell(appArgs.mainClass)) {
          Logging.disableStructuredLogging()
        } else {
          ...
        }
        // Initialize logging if it hasn't been done yet. Keep track of whether logging needs to
        // be reset before the application starts.
        val uninitLog = initializeLogIfNecessary(true, silent = true)
    
        if (appArgs.verbose) {
          logInfo(appArgs.toString)
        }
        appArgs.action match {
          case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog, sparkConf)
          case SparkSubmitAction.KILL => kill(appArgs, sparkConf)
          case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs, sparkConf)
          case SparkSubmitAction.PRINT_VERSION => printVersion()
        }
      }
      ```
      - `parseArguments(args)` invokes `parseArguments()` overridden in the [anonymous class subclassed from the `SparkSubmit` class](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala#L1097C7-L1112C1)
        ```scala
        new SparkSubmitArguments(args.toImmutableArraySeq) {
           override protected def logInfo(msg: => String): Unit = self.logInfo(msg)
 
           override protected def logInfo(entry: LogEntry): Unit = self.logInfo(entry)
 
           override protected def logWarning(msg: => String): Unit = self.logWarning(msg)
 
           override protected def logWarning(entry: LogEntry): Unit = self.logWarning(entry)
 
           override protected def logError(msg: => String): Unit = self.logError(msg)
 
           override protected def logError(entry: LogEntry): Unit = self.logError(entry)
        }

  



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



`sparkConf.get(entry)` -> `entry.readFrom(reader)`

- if `entry` is of type `OptionalConfigEntry`: `entry.readFrom(reader)` -> `readString(reader).map(rawValueConverter)`
  
- If `entry` is of type `ConfigEntryWithDefault`: `entry.readFrom(reader)` -> `readString(reader).map(valueConverter).getOrElse(_defaultValue)`

- Note both `rawValueConverter` and `_defaultValue` are names used in the signatures of the respective classes 


`readString(reader)` ->  `reader.get(key)` -> `conf.get(key).map(substitute)`

-  if no such a `key`, return `None`; otherwise, return `Some(values.mkString(prependSeparator))`
    




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

Variables defined in [java/org/apache/spark/launcher/SparkLauncher.java](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkLauncher.java), e.g., `DRIVER_MEMORY`, hold corresponding string values.

|Entry Name| Key String | Type | Default |
|--|--|--|--|
|`DRIVER_CORES`| `"spark.driver.cores"`| `ConfigEntryWithDefault`| `1` |
|`DRIVER_MEMORY`| `"spark.driver.memory"`| `ConfigEntryWithDefault`| `"1g"` |
|`DRIVER_MEMORY_OVERHEAD`| `"spark.driver.memoryOverhead"`| `OptionalConfigEntry`|  | 
|`DRIVER_MIN_MEMORY_OVERHEAD`|`"spark.driver.minMemoryOverhead"`| `ConfigEntryWithDefault`| `"384m"` | 
|`DRIVER_MEMORY_OVERHEAD_FACTOR`| `"spark.driver.memoryOverheadFactor"`| `ConfigEntryWithDefault`| `0.1` | 
|`EXECUTOR_CORES` |`"spark.executor.cores"`| `ConfigEntryWithDefault`| `1` | 
|`EXECUTOR_MEMORY_OVERHEAD`| `"spark.executor.memoryOverhead"`| `OptionalConfigEntry`|  | 


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
```

<br>

### [*scala/org/apache/spark/SparkConf.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkConf.scala)

```scala

class SparkConf(loadDefaults: Boolean) extends Cloneable with Logging with Serializable {

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

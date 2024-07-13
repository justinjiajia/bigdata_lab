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

Adding `echo "${CMD[@]}"` to the second-to-last line in *spark-class* prints:

```shell
env LD_LIBRARY_PATH=/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib:/usr/lib/jvm/java-17-amazon-corretto.x86_64/../lib:/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/lib/jvm/jre-17/bin/java -cp /usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/usr/lib/spark/conf/:/usr/lib/spark/jars/*:/etc/hadoop/conf/ -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 -Xmx2g -XX:OnOutOfMemoryError=kill -9 %p -XX:+IgnoreUnrecognizedVMOptions --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/sun.nio.cs=ALL-UNNAMED --add-opens=java.base/sun.security.action=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED --add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED -Djdk.reflect.useDirectMethodHandle=false org.apache.spark.deploy.SparkSubmit --master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell
```

Adding `echo $(env)` prints
```shell
SHELL=/bin/bash HISTCONTROL=ignoredups SYSTEMD_COLORS=false HISTSIZE=1000 HOSTNAME=ip-172-31-55-179.ec2.internal SPARK_LOG_DIR=/var/log/spark JAVA17_HOME=/usr/lib/jvm/jre-17 PYTHONHASHSEED=0 PYSPARK_DRIVER_PYTHON=/usr/bin/python3 JAVA_HOME=/usr/lib/jvm/jre-17 SPARK_WORKER_PORT=7078 SPARK_MASTER_WEBUI_PORT=8080 AWS_DEFAULT_REGION=us-east-1 HUDI_CONF_DIR=/etc/hudi/conf _PYSPARK_DRIVER_CONN_INFO_PATH=/tmp/tmp_2k43lsv/tmpsn06obq1 SPARK_DAEMON_JAVA_OPTS= -XX:+ExitOnOutOfMemoryError -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 PWD=/home/hadoop LOGNAME=hadoop SPARK_SUBMIT_OPTS= -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 -DAWS_ACCOUNT_ID=688430810480 -DEMR_CLUSTER_ID=j-VQKB9MH4KRD3 -DEMR_RELEASE_LABEL=emr-7.1.0 XDG_SESSION_TYPE=tty MANPATH=:/opt/puppetlabs/puppet/share/man EMR_CLUSTER_ID=j-VQKB9MH4KRD3 PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" SPARK_WORKER_WEBUI_PORT=8081 MOTD_SHOWN=pam SPARK_MASTER_IP=ip-172-31-55-179.ec2.internal HOME=/home/hadoop LANG=C.UTF-8 LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36: PYTHONSTARTUP=/usr/lib/spark/python/pyspark/shell.py SPARK_MASTER_PORT=7077 SSH_CONNECTION=182.136.218.241 53401 172.31.55.179 22 HIVE_CONF_DIR=/etc/hive/conf PYSPARK_PYTHON=/usr/bin/python3 GEM_HOME=/home/hadoop/.local/share/gem/ruby XDG_SESSION_CLASS=user SELINUX_ROLE_REQUESTED= PYTHONPATH=/usr/lib/spark/python/lib/py4j-0.10.9.7-src.zip:/usr/lib/spark/python/: TERM=xterm-256color HADOOP_CONF_DIR=/etc/hadoop/conf HADOOP_HOME=/usr/lib/hadoop LESSOPEN=||/usr/bin/lesspipe.sh %s USER=hadoop EMR_RELEASE_LABEL=emr-7.1.0 SPARK_PUBLIC_DNS=ip-172-31-55-179.ec2.internal HIVE_SERVER2_THRIFT_PORT=10001 OLD_PYTHONSTARTUP= HIVE_SERVER2_THRIFT_BIND_HOST=0.0.0.0 SELINUX_USE_CURRENT_RANGE= AWS_ACCOUNT_ID=688430810480 SHLVL=1 SPARK_HOME=/usr/lib/spark STANDALONE_SPARK_MASTER_HOST=ip-172-31-55-179.ec2.internal XDG_SESSION_ID=2 LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server SPARK_CONF_DIR=/usr/lib/spark/conf XDG_RUNTIME_DIR=/run/user/992 S_COLORS=auto AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME=EMR SSH_CLIENT=182.136.218.241 53401 22 _SPARK_CMD_USAGE=Usage: ./bin/pyspark [options] which_declare=declare -f SPARK_WORKER_DIR=/var/run/spark/work SPARK_ENV_LOADED=1 PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/aws/puppet/bin/:/opt/puppetlabs/bin SELINUX_LEVEL_REQUESTED= DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/992/bus SPARK_SCALA_VERSION=2.12 MAIL=/var/spool/mail/hadoop SSH_TTY=/dev/pts/0 BASH_FUNC_which%%=() { ( alias; eval ${which_declare} ) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot "$@" } _=/usr/bin/env
```
<br>


###  [*scala/org/apache/spark/deploy/SparkSubmit.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)

<br>

- The fields and methods responsible for recognizing the primary resource:
  ```scala
  private val PYSPARK_SHELL = "pyspark-shell"

  /**
   * Return whether the given primary resource represents a shell.
   */
  private[deploy] def isShell(res: String): Boolean = {
    (res == SPARK_SHELL || res == PYSPARK_SHELL || res == SPARKR_SHELL)
  }
  
  /**
   * Return whether the given primary resource requires running python.
   */
  private[deploy] def isPython(res: String): Boolean = {
    res != null && res.endsWith(".py") || res == PYSPARK_SHELL
  }
  ```

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
  -  `val submit = new SparkSubmit() { ... }` instantiates an anonymous class subclassed from the `SparkSubmit` class and assigns it to `submit`.
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
      - `val appArgs = parseArguments(args)` invokes `parseArguments()` overridden in the [anonymous class subclassed from the `SparkSubmit` class](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala#L1097C7-L1112C1) and assigns what returns to `appArgs`.
        ```scala
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
        ```
        - `new SparkSubmitArguments(args.toImmutableArraySeq){...}` invokes the primmary constructor of the anonymous class. In Scala, the primary constructor is the class body (the entire class body is inside the outermost curly braces (`{...}`)).
    

      - `val sparkConf = appArgs.toSparkConf()`: `toSparkConf()` defined for class `SparkSubmitArguments` in *SparkSubmitArguments.scala*
        ```scala
        private[deploy] def toSparkConf(sparkConf: Option[SparkConf] = None): SparkConf = {
          // either use an existing config or create a new empty one
          sparkProperties.foldLeft(sparkConf.getOrElse(new SparkConf())) {
            case (conf, (k, v)) => conf.set(k, v)
          }
        }
        ```
        It puts all properties contained in `sparkProperties` into a `SparkConf` instance, the class for which is defined in [*SparkConf.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkConf.scala).

     - `case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog, sparkConf)`
       ```scala
       private def submit(args: SparkSubmitArguments, uninitLog: Boolean, sparkConf: SparkConf): Unit = {
      
         def doRunMain(): Unit = {
            if (args.proxyUser != null) {
              ...
            } else {
              runMain(args, uninitLog)
            }
          }
      
          // In standalone cluster mode, there are two submission gateways:
          //   (1) The traditional RPC gateway using o.a.s.deploy.Client as a wrapper
          //   (2) The new REST-based gateway introduced in Spark 1.3
          // The latter is the default behavior as of Spark 1.3, but Spark submit will fail over
          // to use the legacy gateway if the master endpoint turns out to be not a REST server.
          if (args.isStandaloneCluster && args.useRest) {
            ...
            }
          // In all other modes, just run the main class as prepared
          } else {
            doRunMain()
          }
        }
        ```
    - `runMain(args, uninitLog)`
      ```scala
      private def runMain(args: SparkSubmitArguments, uninitLog: Boolean): Unit = {
        val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)
        // Let the main class re-initialize the logging system once it starts.
        if (uninitLog) {
          Logging.uninitialize()
        }
    
        if (args.verbose) {
          logInfo(log"Main class:\n${MDC(LogKeys.CLASS_NAME, childMainClass)}")
          logInfo(log"Arguments:\n${MDC(LogKeys.ARGS, childArgs.mkString("\n"))}")
          // sysProps may contain sensitive information, so redact before printing
          logInfo(log"Spark config:\n" +
          log"${MDC(LogKeys.CONFIG, Utils.redact(sparkConf.getAll.toMap).sorted.mkString("\n"))}")
          logInfo(log"Classpath elements:\n${MDC(LogKeys.CLASS_PATHS, childClasspath.mkString("\n"))}")
          logInfo("\n")
        }
        assert(!(args.deployMode == "cluster" && args.proxyUser != null && childClasspath.nonEmpty) ||
          sparkConf.get(ALLOW_CUSTOM_CLASSPATH_BY_PROXY_USER_IN_CLUSTER_MODE),
          s"Classpath of spark-submit should not change in cluster mode if proxy user is specified " +
            s"when ${ALLOW_CUSTOM_CLASSPATH_BY_PROXY_USER_IN_CLUSTER_MODE.key} is disabled")
        val loader = getSubmitClassLoader(sparkConf)
        for (jar <- childClasspath) {
          addJarToClasspath(jar, loader)
        }
    
        var mainClass: Class[_] = null
    
        try {
          mainClass = Utils.classForName(childMainClass)
        } catch {
          case e: ClassNotFoundException =>
            logError(log"Failed to load class ${MDC(LogKeys.CLASS_NAME, childMainClass)}.")
            if (childMainClass.contains("thriftserver")) {
              logInfo(log"Failed to load main class ${MDC(LogKeys.CLASS_NAME, childMainClass)}.")
              logInfo("You need to build Spark with -Phive and -Phive-thriftserver.")
            } else if (childMainClass.contains("org.apache.spark.sql.connect")) {
              logInfo(log"Failed to load main class ${MDC(LogKeys.CLASS_NAME, childMainClass)}.")
              // TODO(SPARK-42375): Should point out the user-facing page here instead.
              logInfo("You need to specify Spark Connect jars with --jars or --packages.")
            }
            throw new SparkUserAppException(CLASS_NOT_FOUND_EXIT_STATUS)
          case e: NoClassDefFoundError =>
            logError(log"Failed to load ${MDC(LogKeys.CLASS_NAME, childMainClass)}", e)
            if (e.getMessage.contains("org/apache/hadoop/hive")) {
              logInfo("Failed to load hive class.")
              logInfo("You need to build Spark with -Phive and -Phive-thriftserver.")
            }
            throw new SparkUserAppException(CLASS_NOT_FOUND_EXIT_STATUS)
        }
    
        val app: SparkApplication = if (classOf[SparkApplication].isAssignableFrom(mainClass)) {
          mainClass.getConstructor().newInstance().asInstanceOf[SparkApplication]
        } else {
          new JavaMainApplication(mainClass)
        }
    
        @tailrec
        def findCause(t: Throwable): Throwable = t match {
          case e: UndeclaredThrowableException =>
            if (e.getCause() != null) findCause(e.getCause()) else e
          case e: InvocationTargetException =>
            if (e.getCause() != null) findCause(e.getCause()) else e
          case e: Throwable =>
            e
        }
    
        try {
          app.start(childArgs.toArray, sparkConf)
        } catch {
          case t: Throwable =>
            throw findCause(t)
        } finally {
          if (args.master.startsWith("k8s") && !isShell(args.primaryResource) &&
              !isSqlShell(args.mainClass) && !isThriftServer(args.mainClass) &&
              !isConnectServer(args.mainClass)) {
            try {
              SparkContext.getActive.foreach(_.stop())
            } catch {
              case e: Throwable => logError("Failed to close SparkContext", e)
            }
          }
        }
      }
      ```
      - `val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)`: [prepareSubmitEnvironment(args: SparkSubmitArguments, conf: Option[HadoopConfiguration] = None)
          : (Seq[String], Seq[String], SparkConf, String)]()
         - `val sparkConf = args.toSparkConf()`
         - Set the cluster manager. `clusterManager` is set to `YARN` because `args.maybeMaster` matches `Some(v)` and `v` matches `"yarn".
           ```scala
           val clusterManager: Int = args.maybeMaster match {
             case Some(v) =>
               assert(args.maybeRemote.isEmpty || sparkConf.contains("spark.local.connect"))
               v match {
                 case "yarn" => YARN
                 ...
                 case _ =>
                   error("Master must either be yarn or start with spark, k8s, or local")
                   -1
               }
             case None => LOCAL // default master or remote mode.
           }
           ```
        - Set the deploy mode.  `deployMode` is set `CLIENT` because `args.deployMode` matches `null`.
          ```scala
          val deployMode: Int = args.deployMode match {
            case "client" | null => CLIENT
            case "cluster" => CLUSTER
            case _ =>
              error("Deploy mode must be either client or cluster")
              -1
          }
          ```
        - Raise errors for the following ineligible settings:
          ```scala
          (clusterManager, deployMode) match {
            case (STANDALONE, CLUSTER) if args.isPython =>
              error("Cluster deploy mode is currently not supported for python " +
                "applications on standalone clusters.")
            case (STANDALONE, CLUSTER) if args.isR =>
              error("Cluster deploy mode is currently not supported for R " +
                "applications on standalone clusters.")
            case (LOCAL, CLUSTER) =>
              error("Cluster deploy mode is not compatible with master \"local\"")
            case (_, CLUSTER) if isShell(args.primaryResource) =>
              error("Cluster deploy mode is not applicable to Spark shells.")
            case (_, CLUSTER) if isSqlShell(args.mainClass) =>
              error("Cluster deploy mode is not applicable to Spark SQL shell.")
            case (_, CLUSTER) if isThriftServer(args.mainClass) =>
              error("Cluster deploy mode is not applicable to Spark Thrift server.")
            case (_, CLUSTER) if isConnectServer(args.mainClass) =>
              error("Cluster deploy mode is not applicable to Spark Connect server.")
            case _ =>
          }
          ```
          
      ```scala
      private[deploy] def prepareSubmitEnvironment(
          args: SparkSubmitArguments,
          conf: Option[HadoopConfiguration] = None)
          : (Seq[String], Seq[String], SparkConf, String) = {
        // Return values
        val childArgs = new ArrayBuffer[String]()
        val childClasspath = new ArrayBuffer[String]()
        val sparkConf = args.toSparkConf()
        if (sparkConf.contains("spark.local.connect")) sparkConf.remove("spark.remote")
        var childMainClass = ""
    
        
    
        
    
        if (clusterManager == YARN) {
          // Make sure YARN is included in our build if we're trying to use it
          if (!Utils.classIsLoadable(YARN_CLUSTER_SUBMIT_CLASS) && !Utils.isTesting) {
            error(
              "Could not load YARN classes. " +
              "This copy of Spark may not have been compiled with YARN support.")
          }
        }
    
        ...
    
        // Fail fast,
        ...
    
        // Update args.deployMode if it is null. It will be passed down as a Spark property later.
        (args.deployMode, deployMode) match {
          case (null, CLIENT) => args.deployMode = "client"
          case (null, CLUSTER) => args.deployMode = "cluster"
          case _ =>
        }
        val isYarnCluster = clusterManager == YARN && deployMode == CLUSTER
        val isStandAloneCluster = clusterManager == STANDALONE && deployMode == CLUSTER
        val isKubernetesCluster = clusterManager == KUBERNETES && deployMode == CLUSTER
        val isKubernetesClient = clusterManager == KUBERNETES && deployMode == CLIENT
        val isKubernetesClusterModeDriver = isKubernetesClient &&
          sparkConf.getBoolean("spark.kubernetes.submitInDriver", false)
        val isCustomClasspathInClusterModeDisallowed =
          !sparkConf.get(ALLOW_CUSTOM_CLASSPATH_BY_PROXY_USER_IN_CLUSTER_MODE) &&
          args.proxyUser != null &&
          (isYarnCluster || isStandAloneCluster || isKubernetesCluster)
    
        if (!isStandAloneCluster) {
          // Resolve maven dependencies if there are any and add classpath to jars. Add them to py-files
          // too for packages that include Python code
          val resolvedMavenCoordinates = DependencyUtils.resolveMavenDependencies(
            packagesTransitive = true, args.packagesExclusions, args.packages,
            args.repositories, args.ivyRepoPath, args.ivySettingsPath)
    
          if (resolvedMavenCoordinates.nonEmpty) {
            if (isKubernetesCluster) {
              ...
            } else {
              ...
              args.jars = mergeFileLists(args.jars, mergeFileLists(resolvedMavenCoordinates: _*))
              if (args.isPython || isInternal(args.primaryResource)) {
                args.pyFiles = mergeFileLists(args.pyFiles,
                  mergeFileLists(resolvedMavenCoordinates: _*))
              }
            }
          }
    
          ...
        }
    
        // update spark config from args
        args.toSparkConf(Option(sparkConf))
        val hadoopConf = conf.getOrElse(SparkHadoopUtil.newConfiguration(sparkConf))
        val targetDir = Utils.createTempDir()
    
        // Kerberos is not supported in standalone mode
        if (clusterManager != STANDALONE
            && args.principal != null
            && args.keytab != null) {
          // If client mode, make sure the keytab is just a local path.
          if (deployMode == CLIENT && Utils.isLocalUri(args.keytab)) {
            args.keytab = new URI(args.keytab).getPath()
          }
    
          if (!Utils.isLocalUri(args.keytab)) {
            require(new File(args.keytab).exists(), s"Keytab file: ${args.keytab} does not exist")
            UserGroupInformation.loginUserFromKeytab(args.principal, args.keytab)
          }
        }
    
        // Resolve glob path for different resources.
        args.jars = Option(args.jars).map(resolveGlobPaths(_, hadoopConf)).orNull
        args.files = Option(args.files).map(resolveGlobPaths(_, hadoopConf)).orNull
        args.pyFiles = Option(args.pyFiles).map(resolveGlobPaths(_, hadoopConf)).orNull
        args.archives = Option(args.archives).map(resolveGlobPaths(_, hadoopConf)).orNull
    
    
        // In client mode, download remote files.
        var localPrimaryResource: String = null
        var localJars: String = null
        var localPyFiles: String = null
        if (deployMode == CLIENT) {
          localPrimaryResource = Option(args.primaryResource).map {
            downloadFile(_, targetDir, sparkConf, hadoopConf)
          }.orNull
          localJars = Option(args.jars).map {
            downloadFileList(_, targetDir, sparkConf, hadoopConf)
          }.orNull
          localPyFiles = Option(args.pyFiles).map {
            downloadFileList(_, targetDir, sparkConf, hadoopConf)
          }.orNull
    
          if (isKubernetesClusterModeDriver) {
            ...
          }
        }
    
        // When running in YARN, for some remote resources with scheme:
        //   1. Hadoop FileSystem doesn't support them.
        //   2. We explicitly bypass Hadoop FileSystem with "spark.yarn.dist.forceDownloadSchemes".
        // We will download them to local disk prior to add to YARN's distributed cache.
        // For yarn client mode, since we already download them with above code, so we only need to
        // figure out the local path and replace the remote one.
        if (clusterManager == YARN) {
          val forceDownloadSchemes = sparkConf.get(FORCE_DOWNLOAD_SCHEMES)
    
          def shouldDownload(scheme: String): Boolean = {
            forceDownloadSchemes.contains("*") || forceDownloadSchemes.contains(scheme) ||
              Try { FileSystem.getFileSystemClass(scheme, hadoopConf) }.isFailure
          }
    
          def downloadResource(resource: String): String = {
            val uri = Utils.resolveURI(resource)
            uri.getScheme match {
              case "local" | "file" => resource
              case e if shouldDownload(e) =>
                val file = new File(targetDir, new Path(uri).getName)
                if (file.exists()) {
                  file.toURI.toString
                } else {
                  downloadFile(resource, targetDir, sparkConf, hadoopConf)
                }
              case _ => uri.toString
            }
          }
    
          args.primaryResource = Option(args.primaryResource).map { downloadResource }.orNull
          args.files = Option(args.files).map { files =>
            Utils.stringToSeq(files).map(downloadResource).mkString(",")
          }.orNull
          args.pyFiles = Option(args.pyFiles).map { pyFiles =>
            Utils.stringToSeq(pyFiles).map(downloadResource).mkString(",")
          }.orNull
          args.jars = Option(args.jars).map { jars =>
            Utils.stringToSeq(jars).map(downloadResource).mkString(",")
          }.orNull
          args.archives = Option(args.archives).map { archives =>
            Utils.stringToSeq(archives).map(downloadResource).mkString(",")
          }.orNull
        }
    
        // At this point, we have attempted to download all remote resources.
        // Now we try to resolve the main class if our primary resource is a JAR.
        if (args.mainClass == null && !args.isPython && !args.isR) {
          try {
            val uri = new URI(
              Option(localPrimaryResource).getOrElse(args.primaryResource)
            )
            val fs = FileSystem.get(uri, hadoopConf)
    
            Utils.tryWithResource(new JarInputStream(fs.open(new Path(uri)))) { jar =>
              args.mainClass = jar.getManifest.getMainAttributes.getValue("Main-Class")
            }
          } catch {
            case e: Throwable =>
              error(
                s"Failed to get main class in JAR with error '${e.getMessage}'. " +
                " Please specify one with --class."
              )
          }
    
          if (args.mainClass == null) {
            // If we still can't figure out the main class at this point, blow up.
            error("No main class set in JAR; please specify one with --class.")
          }
        }
    
        // If we're running a python app, set the main class to our specific python runner
        if (args.isPython && deployMode == CLIENT) {
          if (args.primaryResource == PYSPARK_SHELL) {
            args.mainClass = "org.apache.spark.api.python.PythonGatewayServer"
          } else {
            ...
          }
        }
    
        // Non-PySpark applications can need Python dependencies.
        if (deployMode == CLIENT && clusterManager != YARN) {
          ...
        }
    
        if (localPyFiles != null) {
          sparkConf.set(SUBMIT_PYTHON_FILES, localPyFiles.split(",").toImmutableArraySeq)
        }
    
        // In YARN mode for an R app, add the SparkR package archive and the R package
        // archive containing all of the built R libraries to archives so that they can
        // be distributed with the job
        if (args.isR && clusterManager == YARN) {
          ...
        }
    
        // TODO: Support distributing R packages with standalone cluster
        if (args.isR && clusterManager == STANDALONE && !RUtils.rPackages.isEmpty) {
          error("Distributing R packages with standalone cluster is not supported.")
        }
    
        // If we're running an R app, set the main class to our specific R runner
        if (args.isR && deployMode == CLIENT) {
          ...
        }
    
        if (isYarnCluster && args.isR) {
          ...
        }
    
        // Special flag to avoid deprecation warnings at the client
        sys.props("SPARK_SUBMIT") = "true"
    
        // A list of rules to map each argument to system properties or command-line options in
        // each deploy mode; we iterate through these below
        val options = List[OptionAssigner](
    
          // All cluster managers
          OptionAssigner(
            // If remote is not set, sets the master,
            // In local remote mode, starts the default master to to start the server.
            if (args.maybeRemote.isEmpty || sparkConf.contains("spark.local.connect")) args.master
            else args.maybeMaster.orNull,
            ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES, confKey = "spark.master"),
          OptionAssigner(
            // In local remote mode, do not set remote.
            if (sparkConf.contains("spark.local.connect")) null
            else args.maybeRemote.orNull, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES, confKey = "spark.remote"),
          OptionAssigner(args.deployMode, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = SUBMIT_DEPLOY_MODE.key),
          OptionAssigner(args.name, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES, confKey = "spark.app.name"),
          OptionAssigner(args.ivyRepoPath, ALL_CLUSTER_MGRS, CLIENT,
            confKey = JAR_IVY_REPO_PATH.key),
          OptionAssigner(args.driverMemory, ALL_CLUSTER_MGRS, CLIENT,
            confKey = DRIVER_MEMORY.key),
          OptionAssigner(args.driverExtraClassPath, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = DRIVER_CLASS_PATH.key),
          OptionAssigner(args.driverExtraJavaOptions, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = DRIVER_JAVA_OPTIONS.key),
          OptionAssigner(args.driverExtraLibraryPath, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = DRIVER_LIBRARY_PATH.key),
          OptionAssigner(args.principal, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = PRINCIPAL.key),
          OptionAssigner(args.keytab, ALL_CLUSTER_MGRS, ALL_DEPLOY_MODES,
            confKey = KEYTAB.key),
          OptionAssigner(args.pyFiles, ALL_CLUSTER_MGRS, CLUSTER, confKey = SUBMIT_PYTHON_FILES.key),
    
          // Propagate attributes for dependency resolution at the driver side
          OptionAssigner(args.packages, STANDALONE | KUBERNETES,
            CLUSTER, confKey = JAR_PACKAGES.key),
          OptionAssigner(args.repositories, STANDALONE | KUBERNETES,
            CLUSTER, confKey = JAR_REPOSITORIES.key),
          OptionAssigner(args.ivyRepoPath, STANDALONE | KUBERNETES,
            CLUSTER, confKey = JAR_IVY_REPO_PATH.key),
          OptionAssigner(args.packagesExclusions, STANDALONE | KUBERNETES,
            CLUSTER, confKey = JAR_PACKAGES_EXCLUSIONS.key),
    
          // Yarn only
          OptionAssigner(args.queue, YARN, ALL_DEPLOY_MODES, confKey = "spark.yarn.queue"),
          OptionAssigner(args.pyFiles, YARN, ALL_DEPLOY_MODES, confKey = "spark.yarn.dist.pyFiles",
            mergeFn = Some(mergeFileLists(_, _))),
          OptionAssigner(args.jars, YARN, ALL_DEPLOY_MODES, confKey = "spark.yarn.dist.jars",
            mergeFn = Some(mergeFileLists(_, _))),
          OptionAssigner(args.files, YARN, ALL_DEPLOY_MODES, confKey = "spark.yarn.dist.files",
            mergeFn = Some(mergeFileLists(_, _))),
          OptionAssigner(args.archives, YARN, ALL_DEPLOY_MODES, confKey = "spark.yarn.dist.archives",
            mergeFn = Some(mergeFileLists(_, _))),
    
          // Other options
          OptionAssigner(args.numExecutors, YARN | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = EXECUTOR_INSTANCES.key),
          OptionAssigner(args.executorCores, STANDALONE | YARN | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = EXECUTOR_CORES.key),
          OptionAssigner(args.executorMemory, STANDALONE | YARN | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = EXECUTOR_MEMORY.key),
          OptionAssigner(args.totalExecutorCores, STANDALONE, ALL_DEPLOY_MODES,
            confKey = CORES_MAX.key),
          OptionAssigner(args.files, LOCAL | STANDALONE | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = FILES.key),
          OptionAssigner(args.archives, LOCAL | STANDALONE | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = ARCHIVES.key),
          OptionAssigner(args.jars, LOCAL, CLIENT, confKey = JARS.key),
          OptionAssigner(args.jars, STANDALONE | KUBERNETES, ALL_DEPLOY_MODES,
            confKey = JARS.key),
          OptionAssigner(args.driverMemory, STANDALONE | YARN | KUBERNETES, CLUSTER,
            confKey = DRIVER_MEMORY.key),
          OptionAssigner(args.driverCores, STANDALONE | YARN | KUBERNETES, CLUSTER,
            confKey = DRIVER_CORES.key),
          OptionAssigner(args.supervise.toString, STANDALONE, CLUSTER,
            confKey = DRIVER_SUPERVISE.key),
          OptionAssigner(args.ivyRepoPath, STANDALONE, CLUSTER, confKey = JAR_IVY_REPO_PATH.key),
    
          // An internal option used only for spark-shell to add user jars to repl's classloader,
          // previously it uses "spark.jars" or "spark.yarn.dist.jars" which now may be pointed to
          // remote jars, so adding a new option to only specify local jars for spark-shell internally.
          OptionAssigner(localJars, ALL_CLUSTER_MGRS, CLIENT, confKey = "spark.repl.local.jars")
        )
    
        // In client mode, launch the application main class directly
        // In addition, add the main application jar and any added jars (if any) to the classpath
        if (deployMode == CLIENT) {
          childMainClass = args.mainClass
          if (localPrimaryResource != null && isUserJar(localPrimaryResource)) {
            childClasspath += localPrimaryResource
          }
          if (localJars != null) { childClasspath ++= localJars.split(",") }
        }
        // Add the main application jar and any added jars to classpath in case YARN client
        // requires these jars.
        // This assumes both primaryResource and user jars are local jars, or already downloaded
        // to local by configuring "spark.yarn.dist.forceDownloadSchemes", otherwise it will not be
        // added to the classpath of YARN client.
        if (isYarnCluster) {
          if (isUserJar(args.primaryResource)) {
            childClasspath += args.primaryResource
          }
          if (args.jars != null) { childClasspath ++= args.jars.split(",") }
        }
    
        if (deployMode == CLIENT) {
          if (args.childArgs != null) { childArgs ++= args.childArgs }
        }
    
        // Map all arguments to command-line options or system properties for our chosen mode
        for (opt <- options) {
          if (opt.value != null &&
              (deployMode & opt.deployMode) != 0 &&
              (clusterManager & opt.clusterManager) != 0) {
            if (opt.clOption != null) { childArgs += opt.clOption += opt.value }
            if (opt.confKey != null) {
              if (opt.mergeFn.isDefined && sparkConf.contains(opt.confKey)) {
                sparkConf.set(opt.confKey, opt.mergeFn.get.apply(sparkConf.get(opt.confKey), opt.value))
              } else {
                sparkConf.set(opt.confKey, opt.value)
              }
            }
          }
        }
    
        // In case of shells, spark.ui.showConsoleProgress can be true by default or by user. Except,
        // when Spark Connect is in local mode, because Spark Connect support its own progress
        // reporting.
        if (isShell(args.primaryResource) && !sparkConf.contains(UI_SHOW_CONSOLE_PROGRESS) &&
            !sparkConf.contains("spark.local.connect")) {
          sparkConf.set(UI_SHOW_CONSOLE_PROGRESS, true)
        }
    
        // Add the application jar automatically so the user doesn't have to call sc.addJar
        // For isKubernetesClusterModeDriver, the jar is already added in the previous spark-submit
        // For YARN cluster mode, the jar is already distributed on each node as "app.jar"
        // For python and R files, the primary resource is already distributed as a regular file
        if (!isKubernetesClusterModeDriver && !isYarnCluster && !args.isPython && !args.isR) {
          var jars = sparkConf.get(JARS)
          if (isUserJar(args.primaryResource)) {
            jars = jars ++ Seq(args.primaryResource)
          }
          sparkConf.set(JARS, jars)
        }
    
        // In standalone cluster mode, use the REST client to submit the application (Spark 1.3+).
        // All Spark parameters are expected to be passed to the client through system properties.
        if (args.isStandaloneCluster) {
          ...
        }
    
        // Let YARN know it's a pyspark app, so it distributes needed libraries.
        if (clusterManager == YARN) {
          if (args.isPython) {
            sparkConf.set("spark.yarn.isPython", "true")
          }
        }
    
        if (clusterManager == KUBERNETES && UserGroupInformation.isSecurityEnabled) {
          ...
        }
    
        // In yarn-cluster mode, use yarn.Client as a wrapper around the user class
        if (isYarnCluster) {
          ...
        }
    
        if (isKubernetesCluster) {
          ...
        }
    
        // Load any properties specified through --conf and the default properties file
        for ((k, v) <- args.sparkProperties) {
          sparkConf.setIfMissing(k, v)
        }
    
        // Ignore invalid spark.driver.host in cluster modes.
        if (deployMode == CLUSTER) {
          sparkConf.remove(DRIVER_HOST_ADDRESS)
        }
    
        // Resolve paths in certain spark properties
        val pathConfigs = Seq(
          JARS.key,
          FILES.key,
          ARCHIVES.key,
          "spark.yarn.dist.files",
          "spark.yarn.dist.archives",
          "spark.yarn.dist.jars")
        pathConfigs.foreach { config =>
          // Replace old URIs with resolved URIs, if they exist
          sparkConf.getOption(config).foreach { oldValue =>
            sparkConf.set(config, Utils.resolveURIs(oldValue))
          }
        }
    
        // Resolve and format python file paths properly before adding them to the PYTHONPATH.
        // The resolving part is redundant in the case of --py-files, but necessary if the user
        // explicitly sets `spark.submit.pyFiles` in his/her default properties file.
        val pyFiles = sparkConf.get(SUBMIT_PYTHON_FILES)
        val resolvedPyFiles = Utils.resolveURIs(pyFiles.mkString(","))
        val formattedPyFiles = if (deployMode != CLUSTER) {
          PythonRunner.formatPaths(resolvedPyFiles).mkString(",")
        } else {
          // Ignoring formatting python path in yarn cluster mode, these two modes
          // support dealing with remote python files, they could distribute and add python files
          // locally.
          resolvedPyFiles
        }
        sparkConf.set(SUBMIT_PYTHON_FILES, formattedPyFiles.split(",").toImmutableArraySeq)
    
        if (args.verbose && isSqlShell(childMainClass)) {
          childArgs ++= Seq("--verbose")
        }
    
        val setSubmitTimeInClusterModeDriver =
          sparkConf.getBoolean("spark.kubernetes.setSubmitTimeInDriver", true)
        if (!sparkConf.contains("spark.app.submitTime")
          || isKubernetesClusterModeDriver && setSubmitTimeInClusterModeDriver) {
          sparkConf.set("spark.app.submitTime", System.currentTimeMillis().toString)
        }
    
        if (childClasspath.nonEmpty && isCustomClasspathInClusterModeDisallowed) {
          childClasspath.clear()
          logWarning(log"Ignore classpath " +
            log"${MDC(LogKeys.CLASS_PATH, childClasspath.mkString(", "))} " +
            log"with proxy user specified in Cluster mode when " +
            log"${MDC(LogKeys.CONFIG, ALLOW_CUSTOM_CLASSPATH_BY_PROXY_USER_IN_CLUSTER_MODE.key)} is " +
            log"disabled")
        }
    
        (childArgs.toSeq, childClasspath.toSeq, sparkConf, childMainClass)
      }   
      ```

       
  


<br>

### [*scala/org/apache/spark/deploy/SparkSubmitArguments.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala)

<br>

`SparkSubmitArguments` extends class [`SparkSubmitArgumentsParser`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/launcher/SparkSubmitArgumentsParser.scala), which makes the Java class [`SparkSubmitOptionParser`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java) visible for Spark code.

```scala
...
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


- `parse(args.asJava)`: [`parse()`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L137C1-L193C4) defined for the parent class `SparkSubmitOptionParser` parses and handles different type of command line options. 

  - If a command-line option exists in [`opts`](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L92), 
  [`handle()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L349C1-L473C4) 
  assigns its value to the corresponding field declared at the [beginning](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L44C1-L86C50) of the definition of class `SparkSubmitArguments`.

    ```scala
    override protected def handle(opt: String, value: String): Boolean = {
      opt match {
        case NAME =>
          name = value

        case MASTER =>
          maybeMaster = Option(value)
    
        ...
        case CLASS =>
          mainClass = value
  
        ...
  
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
  
        ...

        case CONF =>
          val (confName, confValue) = SparkSubmitUtils.parseSparkConfProperty(value)
          sparkProperties(confName) = confValue
        ...
        case VERSION =>
          action = SparkSubmitAction.PRINT_VERSION
        ...
      }
      action != SparkSubmitAction.PRINT_VERSION
    }
    ```
    - The constants used for matching (e.g., `CONF`, `PROPERTIES_FILE`, `EXECUTOR_MEMORY`, etc.) are defined in [*SparkSubmitOptionParser.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkSubmitOptionParser.java#L39C3-L80C44)
   
    - Insert the options defined via `--conf` or `-c` into `sparkProperties`, which is a `HashMap[String, String]` [initialized by this constructor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L77).
      
    - `action` equals `null` unless `opt` matches `VERSION`. As a result, an invocation of `handle()` returns `true` unless `opt` matches `VERSION`.

  - [`handleUnknown()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L475C3-L495C4) handles unrecognized options (those do not exist in both `opts` and `switches`). The first unrecognized option is treated as the "primary resource". Everything else is treated as application arguments.
    ```scala
    override protected def handleUnknown(opt: String): Boolean = {
      if (opt.startsWith("-")) {
        error(s"Unrecognized option '$opt'.")
      }
  
      primaryResource =
        if (!SparkSubmit.isShell(opt) && !SparkSubmit.isInternal(opt)) {
          ...
        } else {
          opt
        }
      isPython = SparkSubmit.isPython(opt)
      isR = SparkSubmit.isR(opt)
      false
    }
    ```
    - This handles the last command-line option `pyspark-shell`. `primaryResource` is set to `"pyspark-shell"`, while `isPython` is set to `true`.

- [`mergeDefaultSparkProperties()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L129C1-L144C4) merges values from the default properties file with those specified through `--conf`. When this is called, `sparkProperties` is already filled with configurations from the latter.


  - `loadPropertiesFromFile(propertiesFile)`: When `--properties-file` was used to specify a properties file (so `propertiesFile` is not `null`), merge values from that file with those specified through `--conf` in `sparkProperties`.
  
  
  - `loadPropertiesFromFile(Utils.getDefaultPropertiesFile(env))`: When no input properties file is specified via `--properties-file` or when `--load-spark-defaults` flag is set, load properties from `spark-defaults.conf`. Note: `env: Map[String, String] = sys.env` is in [the signature of the primary constructor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L42) of class `SparkSubmitArguments`
  
  
  - [`loadPropertiesFromFile(filePath: String)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L109) adds only new entries to `sparkProperties` when `filePath != null` is `true`.
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
  - [`Utils.getDefaultPropertiesFile(env)`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/util/Utils.scala#L1984C3-L1992C4) returns the path of the default Spark properties file.
    ```scala  
    def getDefaultPropertiesFile(env: Map[String, String] = sys.env): String = {
      env.get("SPARK_CONF_DIR")
        .orElse(env.get("SPARK_HOME").map { t => s"$t${File.separator}conf" })
        .map { t => new File(s"$t${File.separator}spark-defaults.conf")}
        .filter(_.isFile)
        .map(_.getAbsolutePath)
        .orNull
    }
    ```
    - Variable `SPARK_CONF_DIR` is one of the environment variables exported by the Bash process that runs *spark-class*, and its value is `/usr/lib/spark/conf`.
      
  - In summary, the precedence of property setting is as follows: `--conf` > properties in a file specified via  `--properties-file` > properties in file `spark-defaults.conf`



- [`loadEnvironmentArguments()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L158C1-L239C4) loads arguments from environment variables, Spark properties etc.
  ```scala
  private def loadEnvironmentArguments(): Unit = {
    maybeMaster = maybeMaster
      .orElse(sparkProperties.get("spark.master"))
      .orElse(env.get("MASTER"))
    ...
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
    ...
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
    ...
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
  ```

  - E.g., `executorCores` is not assigned a value by `handle()` because there's no `--executor-cores` flag among the command-line options. As a result, the function tries to load the value associated with the key `"spark.executor.cores"` from `sparkProperties` first; if there's no such a key in `sparkProperties`, try to load the value from a relevant environment variable. Eventually, `executorCores` is set to `4` because the property `"spark.executor.cores"` is associated with a value of `4` in *spark-defaults.conf*.
  - `driverExtraClassPath` is set to the value associated with the property `"spark.driver.extraClassPath"` in *spark-defaults.conf*.
  - `driverExtraLibraryPath` is set to the value associated with the property `"spark.driver.extraLibraryPath"` in *spark-defaults.conf*.
  - `driverMemory` is set to `2g` due to `.orElse(sparkProperties.get(config.DRIVER_MEMORY.key))`.
  - `name` is set to `PySparkShell`.
  - `action` is set to `SUBMIT`.
  - Objects like `config.EXECUTOR_MEMORY` and `config.DRIVER_CORES` are `ConfigEntry` instances defined in [*package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala). Their `.key` fields are strings like `"spark.executor.memory"` and `"spark.driver.memory"`. The aliases of the keys are defined in [*java/org/apache/spark/launcher/SparkLauncher.java*](https://github.com/apache/spark/blob/master/launcher/src/main/java/org/apache/spark/launcher/SparkLauncher.java)
 
  - This method does not change the content of `sparkProperties`.
    
- [validateArguments()](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L241C2-L249C4) calls `validateSubmitArguments()` to validate all fields as `action` was set to `SUBMIT`.




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

<br>

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

  /** Set a configuration variable. */
  def set(key: String, value: String): SparkConf = {
    set(key, value, false)
  }

  private[spark] def set(key: String, value: String, silent: Boolean): SparkConf = {
    if (key == null) {
      throw new NullPointerException("null key")
    }
    if (value == null) {
      throw new NullPointerException("null value for " + key)
    }
    if (!silent) {
      logDeprecationWarning(key)
    }
    settings.put(key, value)
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

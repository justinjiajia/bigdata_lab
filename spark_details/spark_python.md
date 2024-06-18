

```shell
env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
```

- [`env`](https://www.gnu.org/software/coreutils/manual/html_node/env-invocation.html) runs a command with a modified environment. Changes introduced by `env` only apply to the command that is executed by `env` and do not persist in the current shell environment. 

- `LD_LIBRARY_PATH` tells the dynamic link loader (a.k.a. dynamic linker; find and load the shared libraries needed by a program, prepare the program to run, and then run it; typically named `ld.so` on Linux) where to search for the dynamic shared libraries an application was linked against.

    - Many standard library modules and third party libraries in Python are implemented as C-based Modules. The dynamic linker is responsible for loading shared libraries (files with extension *.so*) corresponding to these modules.
 
    - The dynamic linker scans the list of shared library names embedded in the executable (e.g., `python3`), and load 

      - [`readelf -d`](https://man7.org/linux/man-pages/man1/readelf.1.html) shows the dynamic section of the ELF file, listing the direct dependencies (shared libraries) that the executable was linked against.
        ```shell
        [hadoop@ip-xxxx ~]$ readelf -d /usr/bin/python3
        Dynamic section at offset 0x2db0 contains 28 entries:
         Tag        Type                         Name/Value
        0x0000000000000001 (NEEDED)             Shared library: [libpython3.9.so.1.0]
        0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
        ...
        ```

      - There is the [`ldd` command](https://man7.org/linux/man-pages/man1/ldd.1.html) that lists both direct and indirect dependencies (libraries required by other libraries) at runtime and the paths resolved by the dynamic linker:
      
          ```shell
          [hadoop@ip-xxxx ~]$ which python3
          /usr/bin/python3
          [hadoop@ip-xxxx ~]$ ldd $(which python3)
          linux-vdso.so.1 (0x00007ffd981a1000)
          libpython3.9.so.1.0 => /lib64/libpython3.9.so.1.0 (0x00007fb61f800000)
          libc.so.6 => /lib64/libc.so.6 (0x00007fb61f400000)
          libm.so.6 => /lib64/libm.so.6 (0x00007fb61f725000)
          /lib64/ld-linux-x86-64.so.2 (0x00007fb61fbe2000)
    
          [hadoop@ip-xxxx ~]$ ldd /lib64/libpython3.9.so.1.0
          linux-vdso.so.1 (0x00007ffcc2551000)
          libm.so.6 => /lib64/libm.so.6 (0x00007fcab7224000)
          libc.so.6 => /lib64/libc.so.6 (0x00007fcab6a00000)
          /lib64/ld-linux-x86-64.so.2 (0x00007fcab7308000)
          ```
      
          - The last digit in a name above is a library version number

    - To locate the libraries, the dynamic linker searches multiple resources (environment variable `LD_LIBRARY_PATH` is one of them) with the order explained [here](https://man7.org/linux/man-pages/man8/ld.so.8.html).
 
      
    https://cseweb.ucsd.edu/~gbournou/CSE131/the_inside_story_on_shared_libraries_and_dynamic_loading.pdf

   https://python.plainenglish.io/extending-python-with-c-extension-modules-7beb68a0249b

<br>    

### [*python/pyspark/shell.py*](https://github.com/apache/spark/blob/master/python/pyspark/shell.py)


<br>


```python

...

if is_remote():
    ...
else:
    if os.environ.get("SPARK_EXECUTOR_URI"):
        SparkContext.setSystemProperty("spark.executor.uri", os.environ["SPARK_EXECUTOR_URI"])

    SparkContext._ensure_initialized()

...
```

<br>

### [*python/pyspark/core/context.py*](https://github.com/apache/spark/blob/master/python/pyspark/core/context.py)


<br>

```ptyhon
class SparkContext:

    """
    Main entry point for Spark functionality. A SparkContext represents the
    connection to a Spark cluster, and can be used to create :class:`RDD` and
    broadcast variables on that cluster.

    When you create a new SparkContext, at least the master and app name should
    be set, either through the named parameters here or through `conf`.

    Parameters
    ----------
    master : str, optional
        Cluster URL to connect to (e.g. spark://host:port, local[4]).
    appName : str, optional
        A name for your job, to display on the cluster web UI.
    sparkHome : str, optional
        Location where Spark is installed on cluster nodes.
    pyFiles : list, optional
        Collection of .zip or .py files to send to the cluster
        and add to PYTHONPATH.  These can be paths on the local file
        system or HDFS, HTTP, HTTPS, or FTP URLs.
    environment : dict, optional
        A dictionary of environment variables to set on
        worker nodes.
    batchSize : int, optional, default 0
        The number of Python objects represented as a single
        Java object. Set 1 to disable batching, 0 to automatically choose
        the batch size based on object sizes, or -1 to use an unlimited
        batch size
    serializer : :class:`Serializer`, optional, default :class:`CPickleSerializer`
        The serializer for RDDs.
    conf : :class:`SparkConf`, optional
        An object setting Spark properties.
    gateway : class:`py4j.java_gateway.JavaGateway`,  optional
        Use an existing gateway and JVM, otherwise a new JVM
        will be instantiated. This is only used internally.
    jsc : class:`py4j.java_gateway.JavaObject`, optional
        The JavaSparkContext instance. This is only used internally.
    profiler_cls : type, optional, default :class:`BasicProfiler`
        A class of custom Profiler used to do profiling
    udf_profiler_cls : type, optional, default :class:`UDFBasicProfiler`
        A class of custom Profiler used to do udf profiling

    Notes
    -----
    Only one :class:`SparkContext` should be active per JVM. You must `stop()`
    the active :class:`SparkContext` before creating a new one.

    :class:`SparkContext` instance is not supported to share across multiple
    processes out of the box, and PySpark does not guarantee multi-processing execution.
    Use threads instead for concurrent processing purpose.

    Examples
    --------
    >>> from pyspark.core.context import SparkContext
    >>> sc = SparkContext('local', 'test')
    >>> sc2 = SparkContext('local', 'test2') # doctest: +IGNORE_EXCEPTION_DETAIL
    Traceback (most recent call last):
        ...
    ValueError: ...
    """

    _gateway: ClassVar[Optional[JavaGateway]] = None
    _jvm: ClassVar[Optional[JVMView]] = None
    _next_accum_id = 0
    _active_spark_context: ClassVar[Optional["SparkContext"]] = None
    _lock = RLock()
    _python_includes: Optional[
        List[str]
    ] = None  # zip and egg files that need to be added to PYTHONPATH
    serializer: Serializer
    profiler_collector: ProfilerCollector

    PACKAGE_EXTENSIONS: Iterable[str] = (".zip", ".egg", ".jar")

    ...


    @classmethod
    def _ensure_initialized(
        cls,
        instance: Optional["SparkContext"] = None,
        gateway: Optional[JavaGateway] = None,
        conf: Optional[SparkConf] = None,
    ) -> None:
        """
        Checks whether a SparkContext is initialized or not.
        Throws error if a SparkContext is already running.
        """
        with SparkContext._lock:
            if not SparkContext._gateway:
                SparkContext._gateway = gateway or launch_gateway(conf)
                SparkContext._jvm = SparkContext._gateway.jvm

            if instance:
                if (
                    SparkContext._active_spark_context
                    and SparkContext._active_spark_context != instance
                ):
                    currentMaster = SparkContext._active_spark_context.master
                    currentAppName = SparkContext._active_spark_context.appName
                    callsite = SparkContext._active_spark_context._callsite

                    # Raise error if there is already a running Spark context
                    raise ValueError(
                        "Cannot run multiple SparkContexts at once; "
                        "existing SparkContext(app=%s, master=%s)"
                        " created by %s at %s:%s "
                        % (
                            currentAppName,
                            currentMaster,
                            callsite.function,
                            callsite.file,
                            callsite.linenum,
                        )
                    )
                else:
                    SparkContext._active_spark_context = instance
    

```


- `SparkContext._gateway = gateway or launch_gateway(conf)`


<br>

### [*python/pyspark/java_gateway.py*](https://github.com/apache/spark/blob/master/python/pyspark/java_gateway.py)


<br>

```python
def launch_gateway(conf=None, popen_kwargs=None):
    """
    launch jvm gateway

    Parameters
    ----------
    conf : :py:class:`pyspark.SparkConf`
        spark configuration passed to spark-submit
    popen_kwargs : dict
        Dictionary of kwargs to pass to Popen when spawning
        the py4j JVM. This is a developer feature intended for use in
        customizing how pyspark interacts with the py4j JVM (e.g., capturing
        stdout/stderr).

    Returns
    -------
    ClientServer or JavaGateway
    """
    if "PYSPARK_GATEWAY_PORT" in os.environ:
        gateway_port = int(os.environ["PYSPARK_GATEWAY_PORT"])
        gateway_secret = os.environ["PYSPARK_GATEWAY_SECRET"]
        # Process already exists
        proc = None
    else:
        SPARK_HOME = _find_spark_home()
        # Launch the Py4j gateway using Spark's run command so that we pick up the
        # proper classpath and settings from spark-env.sh
        on_windows = platform.system() == "Windows"
        script = "./bin/spark-submit.cmd" if on_windows else "./bin/spark-submit"
        command = [os.path.join(SPARK_HOME, script)]
        if conf:
            for k, v in conf.getAll():
                command += ["--conf", "%s=%s" % (k, v)]
        submit_args = os.environ.get("PYSPARK_SUBMIT_ARGS", "pyspark-shell")
        if os.environ.get("SPARK_TESTING"):
            submit_args = " ".join(["--conf spark.ui.enabled=false", submit_args])
        command = command + shlex.split(submit_args)

        # Create a temporary directory where the gateway server should write the connection
        # information.
        conn_info_dir = tempfile.mkdtemp()
        try:
            fd, conn_info_file = tempfile.mkstemp(dir=conn_info_dir)
            os.close(fd)
            os.unlink(conn_info_file)

            env = dict(os.environ)
            env["_PYSPARK_DRIVER_CONN_INFO_PATH"] = conn_info_file

            # Launch the Java gateway.
            popen_kwargs = {} if popen_kwargs is None else popen_kwargs
            # We open a pipe to stdin so that the Java gateway can die when the pipe is broken
            popen_kwargs["stdin"] = PIPE
            # We always set the necessary environment variables.
            popen_kwargs["env"] = env
            if not on_windows:
                # Don't send ctrl-c / SIGINT to the Java gateway:
                def preexec_func():
                    signal.signal(signal.SIGINT, signal.SIG_IGN)

                popen_kwargs["preexec_fn"] = preexec_func
                proc = Popen(command, **popen_kwargs)
            else:
                # preexec_fn not supported on Windows
                proc = Popen(command, **popen_kwargs)

            # Wait for the file to appear, or for the process to exit, whichever happens first.
            while not proc.poll() and not os.path.isfile(conn_info_file):
                time.sleep(0.1)

            if not os.path.isfile(conn_info_file):
                raise PySparkRuntimeError(
                    error_class="JAVA_GATEWAY_EXITED",
                    message_parameters={},
                )

            with open(conn_info_file, "rb") as info:
                gateway_port = read_int(info)
                gateway_secret = UTF8Deserializer().loads(info)
        finally:
            shutil.rmtree(conn_info_dir)

        # In Windows, ensure the Java child processes do not linger after Python has exited.
        # In UNIX-based systems, the child process can kill itself on broken pipe (i.e. when
        # the parent process' stdin sends an EOF). In Windows, however, this is not possible
        # because java.lang.Process reads directly from the parent process' stdin, contending
        # with any opportunity to read an EOF from the parent. Note that this is only best
        # effort and will not take effect if the python process is violently terminated.
        if on_windows:
            # In Windows, the child process here is "spark-submit.cmd", not the JVM itself
            # (because the UNIX "exec" command is not available). This means we cannot simply
            # call proc.kill(), which kills only the "spark-submit.cmd" process but not the
            # JVMs. Instead, we use "taskkill" with the tree-kill option "/t" to terminate all
            # child processes in the tree (http://technet.microsoft.com/en-us/library/bb491009.aspx)
            def killChild():
                Popen(["cmd", "/c", "taskkill", "/f", "/t", "/pid", str(proc.pid)])

            atexit.register(killChild)

    # Connect to the gateway (or client server to pin the thread between JVM and Python)
    if os.environ.get("PYSPARK_PIN_THREAD", "true").lower() == "true":
        gateway = ClientServer(
            java_parameters=JavaParameters(
                port=gateway_port, auth_token=gateway_secret, auto_convert=True
            ),
            python_parameters=PythonParameters(port=0, eager_load=False),
        )
    else:
        gateway = JavaGateway(
            gateway_parameters=GatewayParameters(
                port=gateway_port, auth_token=gateway_secret, auto_convert=True
            )
        )

    # Store a reference to the Popen object for use by the caller (e.g., in reading stdout/stderr)
    gateway.proc = proc

    # Import the classes used by PySpark
    java_import(gateway.jvm, "org.apache.spark.SparkConf")
    java_import(gateway.jvm, "org.apache.spark.api.java.*")
    java_import(gateway.jvm, "org.apache.spark.api.python.*")
    java_import(gateway.jvm, "org.apache.spark.ml.python.*")
    java_import(gateway.jvm, "org.apache.spark.mllib.api.python.*")
    java_import(gateway.jvm, "org.apache.spark.resource.*")
    # TODO(davies): move into sql
    java_import(gateway.jvm, "org.apache.spark.sql.*")
    java_import(gateway.jvm, "org.apache.spark.sql.api.python.*")
    java_import(gateway.jvm, "org.apache.spark.sql.hive.*")
    java_import(gateway.jvm, "scala.Tuple2")

    return gateway
```





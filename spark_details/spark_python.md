
`echo "${CMD[@]}"` prints:

```shell
env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
```

- [`env`](https://www.gnu.org/software/coreutils/manual/html_node/env-invocation.html) runs a command with a modified environment (also with exported environment variables). Changes introduced by `env` only apply to the command that is executed by `env` and do not persist in the current shell environment. 

- `LD_LIBRARY_PATH` tells the dynamic link loader (a.k.a. dynamic linker; find and load the shared libraries needed by a program, prepare the program to run, and then run it; typically named `ld.so` on Linux) where to search for the dynamic shared libraries an application was linked against.

    - Many standard library modules and third party libraries in Python are implemented as C-based Modules. The dynamic linker is responsible for loading shared libraries (files with extension *.so*) corresponding to these modules.
 
    - The dynamic linker scans the list of shared library names embedded in the executable (e.g., `python3`).

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
...

class SparkContext:

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
...
import shlex
...

def launch_gateway(conf=None, popen_kwargs=None):

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
                ...

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

        ...

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

- `script = "./bin/spark-submit.cmd" if on_windows else "./bin/spark-submit"` where the conditional expression evaluates to `"./bin/spark-submit"`
  
- `submit_args = os.environ.get("PYSPARK_SUBMIT_ARGS", "pyspark-shell")` gets the value associated with the key `"PYSPARK_SUBMIT_ARGS"`. `print(submit_args)` outputs `"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"`.

- `command = command + shlex.split(submit_args)`
  
    - `shlex.split(submit_args)` splits the string `submit_args` using shell-like syntax.
    
    - `command` equals `['/usr/lib/spark/./bin/spark-submit', '--master', 'yarn', '--conf', 'spark.driver.memory=2g', '--name', 'PySparkShell', '--executor-memory', '2g', 'pyspark-shell']` after the assignment.
 
       - `SPARK_HOME` was assigned `"/usr/lib/spark/"`.

- `conn_info_dir = tempfile.mkdtemp()`

    - [`tempfile.mkdtemp()`](https://docs.python.org/3/library/tempfile.html#tempfile.mkdtemp) creates a temporary directory and returns the absolute pathname of the new directory.

- `fd, conn_info_file = tempfile.mkstemp(dir=conn_info_dir)`

    - [`tempfile.mkstemp(dir=conn_info_dir)`](https://docs.python.org/3/library/tempfile.html#tempfile.mkstemp) creates a temporary file in the given directory, and then opens the file in binary mode. It returns both a file descriptor and the absolute pathname of that file.


- `os.unlink(conn_info_file)` removes the temporary file, and let the subsequent child process to recreate it later.
  
- `Popen(command, **popen_kwargs)` launches a child process to run the program with the arguments specified by `command`. `preexec_func()` will be called before the child process to set a signal handler that [ignores](https://docs.python.org/3/library/signal.html#signal.SIG_IGN) [the interrupt from keyboard](https://docs.python.org/3/library/signal.html#signal.SIGINT) (CTRL + C). [`Popen()`](https://docs.python.org/3/library/subprocess.html#using-the-subprocess-module) is a non-blocking call. It'll run the child process in parallel. Check out [this tutorial](https://realpython.com/python-subprocess/#the-popen-class) for more details

    - `popen_kwargs` contains an entry with the key `env` and the value that is set as follows:
      ```python
      env = dict(os.environ)
      env["_PYSPARK_DRIVER_CONN_INFO_PATH"] = conn_info_file
      ```
      
        - The `env` argument defines the environment variables for the new process; these are used instead of the default behavior of inheriting the current process' environment.
     
        - In addition to the variables exported by the script files, `env` contains 3 more entries: `{'PYSPARK_SUBMIT_ARGS': '"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"', 'LD_LIBRARY_PATH': '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server', '_PYSPARK_DRIVER_CONN_INFO_PATH': '/tmp/tmp8d5mne_q/tmp3dk7xihm'}`



- The `while` loop makes the program to wait for the file to appear, or for the child process to exit, whichever happens first.

    - `proc.poll()` returns `None` when the child process is ongoing.
 
- `gateway_port = read_int(info)`

    - `read_int()` is defined in [*python/pyspark/serializers.py*](https://github.com/apache/spark/blob/master/python/pyspark/serializers.py) as follows:
      ```python
      def read_int(stream):
        length = stream.read(4)
        if not length:
            raise EOFError
        return struct.unpack("!i", length)[0]
      ```

      

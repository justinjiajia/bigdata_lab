
`echo "${CMD[@]}"` prints:

```shell
env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
```

- [`env`](https://www.gnu.org/software/coreutils/manual/html_node/env-invocation.html) runs a command with a modified environment. Changes introduced by `env` only apply to the command that is executed by `env` and do not persist in the current shell environment. 

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
...
import shlex
...

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

- `script = "./bin/spark-submit.cmd" if on_windows else "./bin/spark-submit"` evaluates to `"./bin/spark-submit"`
  
- `submit_args = os.environ.get("PYSPARK_SUBMIT_ARGS", "pyspark-shell")` gets the value associated with the key `"PYSPARK_SUBMIT_ARGS"`. `print(submit_args)` outputs `"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"`.

- `command = command + shlex.split(submit_args)`
  
    - `shlex.split(submit_args)` splits the string `submit_args` using shell-like syntax.
    
    - `command` equals `['/usr/lib/spark/./bin/spark-submit', '--master', 'yarn', '--conf', 'spark.driver.memory=2g', '--name', 'PySparkShell', '--executor-memory', '2g', 'pyspark-shell']` after the assignment.
 
       - `SPARK_HOME` was assigned `"/usr/lib/spark/"`.

- `conn_info_dir = tempfile.mkdtemp()`

    - [`tempfile.mkdtemp()`](https://docs.python.org/3/library/tempfile.html#tempfile.mkdtemp) creates a temporary directory and returns the absolute pathname of the new directory.

- `fd, conn_info_file = tempfile.mkstemp(dir=conn_info_dir)`

    - [`tempfile.mkstemp(dir=conn_info_dir)`](https://docs.python.org/3/library/tempfile.html#tempfile.mkstemp) creates a temporary file in the given directory, and then opens the file in binary mode. It returns both a file descriptor and the absolute pathname of that file.


- `Popen(command, **popen_kwargs)` launches a child process to run the program with the arguments specified by `command`. `preexec_func()` will be called before the child process to set a signal handler that [ignores](https://docs.python.org/3/library/signal.html#signal.SIG_IGN) [the interrupt from keyboard](https://docs.python.org/3/library/signal.html#signal.SIGINT) (CTRL + C). [`Popen()`](https://docs.python.org/3/library/subprocess.html#using-the-subprocess-module) is a non-blocking call. It'll run the child process in parallel. Check out [this tutorial](https://realpython.com/python-subprocess/#the-popen-class) for more details

    - `popen_kwargs` contains an entry with the key `env` and the value that is set as follows:
      ```python
      env = dict(os.environ)
      env["_PYSPARK_DRIVER_CONN_INFO_PATH"] = conn_info_file
      ```
      
        - The `env` argument defines the environment variables for the new process; these are used instead of the default behavior of inheriting the current process' environment.
     


```
{'SHELL': '/bin/bash', 'HISTCONTROL': 'ignoredups', 'SYSTEMD_COLORS': 'false', 'HISTSIZE': '1000', 'HOSTNAME': 'ip-172-31-62-159.ec2.internal', 'SPARK_LOG_DIR': '/var/log/spark', 'JAVA17_HOME': '/usr/lib/jvm/jre-17', 'PYTHONHASHSEED': '0', 'PYSPARK_DRIVER_PYTHON': '/usr/bin/python3', 'JAVA_HOME': '/usr/lib/jvm/jre-17', 'SPARK_WORKER_PORT': '7078', 'SPARK_MASTER_WEBUI_PORT': '8080', 'AWS_DEFAULT_REGION': 'us-east-1', 'HUDI_CONF_DIR': '/etc/hudi/conf', 'SPARK_DAEMON_JAVA_OPTS': ' -XX:+ExitOnOutOfMemoryError -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0', 'PWD': '/home/hadoop', 'LOGNAME': 'hadoop', 'SPARK_SUBMIT_OPTS': ' -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0', 'XDG_SESSION_TYPE': 'tty', 'MANPATH': ':/opt/puppetlabs/puppet/share/man', 'EMR_CLUSTER_ID': 'j-VC0KIOO5V5LS', 'SPARK_WORKER_WEBUI_PORT': '8081', 'MOTD_SHOWN': 'pam', 'SPARK_MASTER_IP': 'ip-172-31-62-159.ec2.internal', 'HOME': '/home/hadoop', 'LANG': 'C.UTF-8', 'LS_COLORS': 'rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36:', 'PYTHONSTARTUP': '/usr/lib/spark/python/pyspark/shell.py', 'SPARK_MASTER_PORT': '7077', 'SSH_CONNECTION': '1.64.143.12 59381 172.31.62.159 22', 'HIVE_CONF_DIR': '/etc/hive/conf', 'PYSPARK_PYTHON': '/usr/bin/python3', 'GEM_HOME': '/home/hadoop/.local/share/gem/ruby', 'XDG_SESSION_CLASS': 'user', 'SELINUX_ROLE_REQUESTED': '', 'PYTHONPATH': '/usr/lib/spark/python/lib/py4j-0.10.9.7-src.zip:/usr/lib/spark/python/:', 'TERM': 'xterm-256color', 'HADOOP_CONF_DIR': '/etc/hadoop/conf', 'HADOOP_HOME': '/usr/lib/hadoop', 'LESSOPEN': '||/usr/bin/lesspipe.sh %s', 'USER': 'hadoop', 'EMR_RELEASE_LABEL': 'emr-7.1.0', 'SPARK_PUBLIC_DNS': 'ip-172-31-62-159.ec2.internal', 'HIVE_SERVER2_THRIFT_PORT': '10001', 'OLD_PYTHONSTARTUP': '', 'HIVE_SERVER2_THRIFT_BIND_HOST': '0.0.0.0', 'SELINUX_USE_CURRENT_RANGE': '', 'AWS_ACCOUNT_ID': '154048744197', 'SHLVL': '1', 'SPARK_HOME': '/usr/lib/spark', 'STANDALONE_SPARK_MASTER_HOST': 'ip-172-31-62-159.ec2.internal', 'XDG_SESSION_ID': '4', 'SPARK_CONF_DIR': '/usr/lib/spark/conf', 'XDG_RUNTIME_DIR': '/run/user/992', 'S_COLORS': 'auto', 'AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME': 'EMR', 'SSH_CLIENT': '1.64.143.12 59381 22', '_SPARK_CMD_USAGE': 'Usage: ./bin/pyspark [options]', 'which_declare': 'declare -f', 'SPARK_WORKER_DIR': '/var/run/spark/work', 'SPARK_ENV_LOADED': '1', 'PATH': '/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/aws/puppet/bin/:/opt/puppetlabs/bin', 'SELINUX_LEVEL_REQUESTED': '', 'DBUS_SESSION_BUS_ADDRESS': 'unix:path=/run/user/992/bus', 'SPARK_SCALA_VERSION': '2.12', 'MAIL': '/var/spool/mail/hadoop', 'SSH_TTY': '/dev/pts/0', 'BASH_FUNC_which%%': '() {  ( alias;\n eval ${which_declare} ) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot "$@"\n}', 'PYSPARK_SUBMIT_ARGS': '"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"', 'LD_LIBRARY_PATH': '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server', '_PYSPARK_DRIVER_CONN_INFO_PATH': '/tmp/tmp8d5mne_q/tmp3dk7xihm'}
```


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

      

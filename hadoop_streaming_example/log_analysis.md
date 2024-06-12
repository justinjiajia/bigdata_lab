
# Log analysis


Key information:

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000001
user.name: yarn
...
Executing with tokens: [Kind: YARN_AM_RM_TOKEN, Service: , Ident: (appAttemptId { application_id { id: 1 cluster_timestamp: 1717955085543 } attemptId: 1 } keyId: -249482518)]
...
Default file system [hdfs://ip-172-31-53-255.ec2.internal:8020]
...
Adding job token for job_1717955085543_0001 to jobTokenSecretManager
Not uberizing job_1717955085543_0001 because: not enabled; too many maps; too many reduces; too much input;
Input size for job job_1717955085543_0001 = 44791189. Number of splits = 16
Number of reduces for job job_1717955085543_0001 = 2
job_1717955085543_0001Job Transitioned from NEW to INITED
MRAppMaster launching normal, non-uberized, multi-container job job_1717955085543_0001.
...
Instantiated MRClientService at ip-172-31-52-162.ec2.internal/172.31.52.162:43637
...
JOB_CREATE job_1717955085543_0001
...
Connecting to ResourceManager at ip-172-31-53-255.ec2.internal/172.31.53.255:8030
maxContainerCapability: <memory:6144, vCores:4>
...
org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Upper limit on the thread pool size is 500
org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: The thread pool initial size is 10
job_1717955085543_0001Job Transitioned from INITED to SETUP
...
org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
```

<br>

## Task Configarations and Schedulding


[A]: [AsyncDispatcher event handler] 

[e]: [eventHandlingThread]

```
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: job_1717955085543_0001Job Transitioned from SETUP to RUNNING
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: Resource capability of task type MAP is set to <memory:1536, max memory:9223372036854775807, vCores:1, max vCores:2147483647>
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_m_000000 Task Transitioned from NEW to SCHEDULED
...
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_m_000015 Task Transitioned from NEW to SCHEDULED
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: Resource capability of task type REDUCE is set to <memory:3072, max memory:9223372036854775807, vCores:1, max vCores:2147483647>
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000000 Task Transitioned from NEW to SCHEDULED
[e] Event Writer setup for JobId: job_1717955085543_0001, File: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1.jhist
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000001 Task Transitioned from NEW to SCHEDULED
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000000_0 TaskAttempt Transitioned from NEW to UNASSIGNED
...
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000015_0 TaskAttempt Transitioned from NEW to UNASSIGNED
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000000_0 TaskAttempt Transitioned from NEW to UNASSIGNED
[A] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000001_0 TaskAttempt Transitioned from NEW to UNASSIGNED
[Thread-88] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: mapResourceRequest:<memory:1536, max memory:9223372036854775807, vCores:1, max vCores:2147483647>
[Thread-88] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: reduceResourceRequest:<memory:3072, max memory:9223372036854775807, vCores:1, max vCores:2147483647>
```

Create and schedule all tasks and then create 1 attempt per task


<br>

## Request Containers from ResourceManager

```shell
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:2 ScheduledMaps:16 ScheduledReds:0 AssignedMaps:0 [RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: AssignedReds:0 CompletedMaps:0 CompletedReds:0 ContAlloc:0 ContRel:0 HostLocal:0 RackLocal:0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: getResources() for application_1717955085543_0001: ask=6 release= 0 newContainers=0 finishedContainers=0 resourcelimit=<memory:21504, vCores:15> knownNMs=4
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Recalculating schedule, headroom=<memory:21504, vCores:15>
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
```

- **RMCommunicator Allocator**: A component in MRAppMaster; Responsible for resource allocation and communication with the ResourceManager. Note that `RMContainerAllocator` is a specific implementation of `RMCommunicator`. 
- `PendingReds:2`: There are 2 reduce tasks pending.
- `ScheduledMaps:16`: There are 16 map tasks scheduled.
- `ScheduledReds:0`: There are no reduce tasks scheduled yet.
- `AssignedMaps:0`: No map tasks have been assigned to containers yet.
- `AssignedReds:0`: No reduce tasks have been assigned to containers yet.
- `CompletedMaps:0`: No map tasks have been completed yet.
- `CompletedReds:0`: No reduce tasks have been completed yet.
- `ContAlloc:0`: No containers have been allocated yet.
- ContRel: 0: No containers have been released yet.
- HostLocal: 0: No tasks have been assigned to containers on the same node as the data.
- RackLocal: 0: No tasks have been assigned to containers on the same rack as the data.
- `getResources() for application_1717955085543_0001`: This log entry is related to the resource requests and allocations for the specific application.
- `ask=6`: The application is requesting 6 containers.
- `release=0`: No containers are being released back to the ResourceManager.
- `newContainers=0`: No new containers have been allocated in this scheduling iteration.
- finishedContainers=0: No containers have finished their tasks in this iteration.
- resourcelimit=<memory:21504, vCores:15>: The total resources available for allocation are 21,504 MB of memory and 15 vCores.
- `knownNMs=4`: There are 4 NodeManagers known to the ResourceManager.
- `Recalculating schedule`: initiated by the RMCommunicator Allocator. The ResourceManager is recalculating the resource allocation and scheduling based on the current state and resource requests.
- `headroom=<memory:21504, vCores:15>`: There are 21,504 MB of memory and 15 vCores available for allocation to applications.
- `Reduce slow start threshold not met`: Indicates that the required fraction of completed map tasks (as specified by `mapreduce.job.reduce.slowstart.completedmaps`) has not been reached, so reduce tasks are not yet being scheduled.
- `mapreduce.job.reduce.slowstart.completedmaps` is a configuration parameter that controls when the reduce tasks are allowed to start executing relative to the progress of the map tasks. The default value is 0.05. So, 0.05*16=0.8, and it rounds up to 1.



#### Code for the MRAppMaster's interaction with the ResourceManager? (org/apache/hadoop/mapreduce/v2/app/rm/)

https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-app/src/main/java/org/apache/hadoop/mapreduce/v2/app/rm/RMCommunicator.java


```
import org.apache.hadoop.yarn.api.ApplicationMasterProtocol;
...
/**
 * Registers/unregisters to RM and sends heartbeats to RM.
 */
public abstract class RMCommunicator extends AbstractService
    implements RMHeartbeatHandler {
  ...
  protected ApplicationMasterProtocol scheduler;
  ...


```
https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-app/src/main/java/org/apache/hadoop/mapreduce/v2/app/rm/RMContainerRequestor.java

```java
...
import org.apache.hadoop.yarn.api.protocolrecords.AllocateResponse;
...
import org.apache.hadoop.yarn.api.records.ResourceRequest.ResourceRequestComparator;
...
/**
 * Keeps the data structures to send container requests to RM.
 */
public abstract class RMContainerRequestor extends RMCommunicator {
  ...
  private static final ResourceRequestComparator RESOURCE_REQUEST_COMPARATOR =
      new ResourceRequestComparator();
  ...
  // use custom comparator to make sure ResourceRequest objects differing only in 
  // numContainers dont end up as duplicates
  private final Set<ResourceRequest> ask = new TreeSet<ResourceRequest>(
      RESOURCE_REQUEST_COMPARATOR);
  private final Set<ContainerId> release = new TreeSet<ContainerId>();
  // pendingRelease holds history or release requests.request is removed only if
  // RM sends completedContainer.
  // How it different from release? --> release is for per allocate() request.
  protected Set<ContainerId> pendingRelease = new TreeSet<ContainerId>();

  ...
  protected AllocateResponse makeRemoteRequest() throws YarnException,
      IOException {
    applyRequestLimits();
    ResourceBlacklistRequest blacklistRequest =
        ResourceBlacklistRequest.newInstance(new ArrayList<String>(blacklistAdditions),
            new ArrayList<String>(blacklistRemovals));
    AllocateRequest allocateRequest =
        AllocateRequest.newInstance(lastResponseID,
          super.getApplicationProgress(), new ArrayList<ResourceRequest>(ask),
          new ArrayList<ContainerId>(release), blacklistRequest);
    AllocateResponse allocateResponse = scheduler.allocate(allocateRequest);
    lastResponseID = allocateResponse.getResponseId();
    availableResources = allocateResponse.getAvailableResources();
    lastClusterNmCount = clusterNmCount;
    clusterNmCount = allocateResponse.getNumClusterNodes();
    int numCompletedContainers =
        allocateResponse.getCompletedContainersStatuses().size();

    if (ask.size() > 0 || release.size() > 0) {
      LOG.info("applicationId={}: ask={} release={} newContainers={} finishedContainers={}"
              + " resourceLimit={} knownNMs={}", applicationId, ask.size(), release.size(),
          allocateResponse.getAllocatedContainers().size(), numCompletedContainers,
          availableResources, clusterNmCount);
    }

    ask.clear();
    release.clear();

    if (numCompletedContainers > 0) {
      // re-send limited requests when a container completes to trigger asking
      // for more containers
      requestLimitsToUpdate.addAll(requestLimits.keySet());
    }

    if (blacklistAdditions.size() > 0 || blacklistRemovals.size() > 0) {
      LOG.info("Update the blacklist for " + applicationId +
          ": blacklistAdditions=" + blacklistAdditions.size() +
          " blacklistRemovals=" +  blacklistRemovals.size());
    }
    blacklistAdditions.clear();
    blacklistRemovals.clear();
    return allocateResponse;
  }

  ...
}
```

https://github.com/apache/hadoop/blob/trunk/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-api/src/main/java/org/apache/hadoop/yarn/api/records/ResourceRequest.java


https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-app/src/main/java/org/apache/hadoop/mapreduce/v2/app/rm/RMContainerAllocator.java

```java
/**
 * Allocates the container from the ResourceManager scheduler.
 */
public class RMContainerAllocator extends RMContainerRequestor
    implements ContainerAllocator {
   ...
}
```

https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-app/src/main/java/org/apache/hadoop/mapreduce/v2/app/MRAppMaster.java

the MRAppMaster creates a RMContainerAllocator for the non-uber mode. 

```java
    protected void serviceStart() throws Exception {
      if (job.isUber()) {
        MRApps.setupDistributedCacheLocal(getConfig());
        this.containerAllocator = new LocalContainerAllocator(
            this.clientService, this.context, nmHost, nmPort, nmHttpPort
            , containerID);
      } else {
        this.containerAllocator = new RMContainerAllocator(
            this.clientService, this.context, preemptionPolicy);
      }
      ((Service)this.containerAllocator).init(getConfig());
      ((Service)this.containerAllocator).start();
      super.serviceStart();
    }
```

## Assign Containers to Task Attempts


Before: `headroom=<memory:21504, vCores:15>`

After: `headroom=<memory:0, vCores:1>`

21504 / (1536 per map container) = 14 containers


Entity: [RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator

```shell
Got allocated containers 14
Assigned container container_1717955085543_0001_01_000002 to attempt_1717955085543_0001_m_000000_0
Assigned container container_1717955085543_0001_01_000003 to attempt_1717955085543_0001_m_000013_0
Assigned container container_1717955085543_0001_01_000004 to attempt_1717955085543_0001_m_000014_0
Assigned container container_1717955085543_0001_01_000005 to attempt_1717955085543_0001_m_000015_0
Assigned container container_1717955085543_0001_01_000006 to attempt_1717955085543_0001_m_000001_0
Assigned container container_1717955085543_0001_01_000007 to attempt_1717955085543_0001_m_000002_0
Assigned container container_1717955085543_0001_01_000008 to attempt_1717955085543_0001_m_000003_0
Assigned container container_1717955085543_0001_01_000009 to attempt_1717955085543_0001_m_000004_0
Assigned container container_1717955085543_0001_01_000010 to attempt_1717955085543_0001_m_000007_0
Assigned container container_1717955085543_0001_01_000011 to attempt_1717955085543_0001_m_000008_0
Assigned container container_1717955085543_0001_01_000012 to attempt_1717955085543_0001_m_000009_0
Assigned container container_1717955085543_0001_01_000013 to attempt_1717955085543_0001_m_000010_0
Assigned container container_1717955085543_0001_01_000014 to attempt_1717955085543_0001_m_000005_0
Assigned container container_1717955085543_0001_01_000015 to attempt_1717955085543_0001_m_000006_0
Recalculating schedule, headroom=<memory:0, vCores:1>
Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
After Scheduling: PendingReds:2 ScheduledMaps:2 ScheduledReds:0 AssignedMaps:14 AssignedReds:0 CompletedMaps:0 CompletedReds:0 ContAlloc:14 ContRel:0 HostLocal:14 RackLocal:0
```


#### Summary of Assignments

```
container 2 <---> m_00
container 3 <---> m_13
container 4 <---> m_14
container 5 <---> m_15
container 6 <---> m_01
container 7 <---> map task 2
container 8 <---> map task 3
container 9 <---> map task 4
container 10 <---> map task 7
container 11 <---> map task 8
container 12 <---> map task 9
container 13 <---> map task 10
container 14 <---> map task 5
container 15 <---> map task 6
```
 
Entity: [AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl

```shell
The job-jar file on the remote FS is hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job.jar
The job-conf file on the remote FS is /tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job.xml
Adding #0 tokens and #1 secret keys for NM use for launching container
Size of containertokens_dob is 1
Putting shuffle token in serviceData
attempt_1717955085543_0001_m_000000_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000013_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000014_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000015_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000001_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000002_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000003_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000004_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000007_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000008_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000009_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000010_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000005_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
attempt_1717955085543_0001_m_000006_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
```


## Launch Containers 

The assignments of containers to tasks has been determined previously

There are 10 container launcher threads in the thread pool (of MRAppMaster?). The 14 contaners are launched as follows:

| container | Launcher ID | task | core instance|
|-----|-----|------|----|
| container 2 | 0| m_00 | ip-xxxx-63-90.xxxx  |
| container 3 | 1| m_13  | ip-xxxx-63-90.xxxx  |
| container 4 | 2 | m_14 | ip-xxxx-63-90.xxxx |
| container 5 | 3| m_15 | ip-xxxx-63-90.xxxx  |
| container 6 | 4| m_01  | ip-xxxx-57-35.xxxx  |
| container 7 | 5| m_02 | ip-xxxx-57-35.xxxx  |
| container 8 | 6| m_03 | ip-xxxx-57-35.xxxx  |
| container 9 | 7 | m_04 | ip-xxxx-57-35.xxxx |
| container 10| 8  | m_07  | ip-xxxx-52-142.xxxx |
| container 11| 9 | m_08  | ip-xxxx-52-142.xxxx  |
| container 12 | 0| m_09  | ip-xxxx-52-142.xxxx  |
| container 13 | 1|  m_10  | ip-xxxx-52-142.xxxx  |
| container 14 | 2 |  m_05 | ip-xxxx-52-162.xxxx |
| container 15| 3 |  m_06 | ip-xxxx-52-162.xxxx  |

> Every core instance have 4 vCores.  Each container gets allocated 1 vCore. MRAppMster runs in container 1 on ip-xxxx-52-162.xxxx

Lauching a container prints the following information:

```
[ContainerLauncher #0] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_REMOTE_LAUNCH for container container_1717955085543_0001_01_000002 taskAttempt attempt_1717955085543_0001_m_000000_0
[ContainerLauncher #0] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Launching attempt_1717955085543_0001_m_000000_0
[ContainerLauncher #0] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Shuffle port returned by ContainerManager for attempt_1717955085543_0001_m_000000_0 : 13562
```

Then container launcher #0 is allocated to launch a different container.

Then the designated attempt gets launched within the container (Note the difference between TaskAttempt and Task):

```shell
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: TaskAttempt: [attempt_1717955085543_0001_m_000000_0] using containerId: [container_1717955085543_0001_01_000002 on NM: [ip-172-31-63-90.ec2.internal:8041]
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000000_0 TaskAttempt Transitioned from ASSIGNED to RUNNING
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.speculate.DefaultSpeculator: ATTEMPT_START task_1717955085543_0001_m_000000
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_m_000000 Task Transitioned from SCHEDULED to RUNNING
```

Attempt launch seems to occur asynchronously among containers.


<br>

## Request Additional Containers


Entity: [RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor

```shell
getResources() for application_1717955085543_0001: ask=6 release= 0 newContainers=0 finishedContainers=0 resourcelimit=<memory:0, vCores:1> knownNMs=4
Recalculating schedule, headroom=<memory:0, vCores:1>
Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
Recalculating schedule, headroom=<memory:0, vCores:1>
Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
...
```

repeatedly recalculate schedule



in between, also see:

```
[Socket Reader #1 for port 0] SecurityLogger.org.apache.hadoop.ipc.Server: Auth successful for job_1717955085543_0001 (auth:SIMPLE) from ip-172-31-52-162.ec2.internal:38426 / 172.31.52.162:38426
[IPC Server handler 2 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: JVM with ID : jvm_1717955085543_0001_m_000015 asked for a task
[IPC Server handler 2 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: JVM with ID: jvm_1717955085543_0001_m_000015 given task: attempt_1717955085543_0001_m_000006_0
```

jvm_1717955085543_0001_m_000015 (container 15; ip-xxxx-52-162.xxxx) given task: attempt_1717955085543_0001_m_000006_0




```
...
[IPC Server handler 37 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000006_0 is : 0.0
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Recalculating schedule, headroom=<memory:0, vCores:1>
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
[IPC Server handler 3 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000005_0 is : 0.0
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Recalculating schedule, headroom=<memory:0, vCores:1>
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Reduce slow start threshold not met. completedMapsForReduceSlowstart 1
[IPC Server handler 2 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000006_0 is : 1.0
[IPC Server handler 1 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000006_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000006_0 TaskAttempt Transitioned from RUNNING to SUCCESS_FINISHING_CONTAINER
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Task succeeded with attempt attempt_1717955085543_0001_m_000006_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_m_000006 Task Transitioned from RUNNING to SUCCEEDED
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 1
```

detect that the progress becomes 1.0 -> receive acknowledgement -> attempt status change: RUNNING to SUCCESS_FINISHING_CONTAINER 
-> task succeeded -> task status change: RUNNING to SUCCEEDED -> tally the number of completed tasks

```
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:2 ScheduledMaps:2 ScheduledReds:0 AssignedMaps:14 AssignedReds:0 CompletedMaps:1 CompletedReds:0 ContAlloc:14 ContRel:0 HostLocal:14 RackLocal:0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Recalculating schedule, headroom=<memory:0, vCores:1>
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Reduce slow start threshold reached. Scheduling reduces.
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: completedMapPercent 0.0625 totalResourceLimit:<memory:21504, vCores:15> finalMapResourceLimit:<memory:20160, vCores:15> finalReduceResourceLimit:<memory:1344, vCores:0> netScheduledMapResource:<memory:24576, vCores:16> netScheduledReduceResource:<memory:0, vCores:0>
[IPC Server handler 0 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000005_0 is : 1.0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:2 ScheduledMaps:2 ScheduledReds:0 AssignedMaps:14 AssignedReds:0 CompletedMaps:2 CompletedReds:0 ContAlloc:14 ContRel:0 HostLocal:14 RackLocal:0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000015
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: Diagnostics report from attempt_1717955085543_0001_m_000006_0: 
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000006_0 TaskAttempt Transitioned from SUCCESS_FINISHING_CONTAINER to SUCCEEDED
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000014
[ContainerLauncher #7] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000015 taskAttempt attempt_1717955085543_0001_m_000006_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: Diagnostics report from attempt_1717955085543_0001_m_000005_0: 
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000005_0 TaskAttempt Transitioned from SUCCESS_FINISHING_CONTAINER to SUCCEEDED
[ContainerLauncher #4] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000014 taskAttempt attempt_1717955085543_0001_m_000005_0
```

for each completed task:

```
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000015
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: Diagnostics report from attempt_1717955085543_0001_m_000006_0: 
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000006_0 TaskAttempt Transitioned from SUCCESS_FINISHING_CONTAINER to SUCCEEDED
[ContainerLauncher #7] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000015 taskAttempt attempt_1717955085543_0001_m_000006_0
```



###

| container | Launcher ID | task | core instance |  
|-----------|-------------|------|---------------| 
| container 16 | 5 | m_011 | ip-xxxx-52-162.xxxx |  
| container 17 | 6 | m_012 | ip-xxxx-52-162.xxxx | 

 > MRAppMster runs in container 1 on ip-xxxx-52-162.xxxx


```
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Got allocated containers 2
```





the order of events is a bit different from those for the previous tasks

```
*[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000011_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
[ContainerLauncher #5] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_REMOTE_LAUNCH for container container_1717955085543_0001_01_000016 taskAttempt attempt_1717955085543_0001_m_000011_0
[ContainerLauncher #5] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Launching attempt_1717955085543_0001_m_000011_0
*[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned container container_1717955085543_0001_01_000016 to attempt_1717955085543_0001_m_000011_0
```

```
*[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000012_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
[ContainerLauncher #6] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_REMOTE_LAUNCH for container container_1717955085543_0001_01_000017 taskAttempt attempt_1717955085543_0001_m_000012_0
[ContainerLauncher #6] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Launching attempt_1717955085543_0001_m_000012_0
*[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned container container_1717955085543_0001_01_000017 to attempt_1717955085543_0001_m_000012_0
```

#####

```
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=5 release= 0 newContainers=0 finishedContainers=0 resourcelimit=<memory:0, vCores:1> knownNMs=4
...
[IPC Server handler 8 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000000_0 is : 0.0
[DefaultSpeculator background processing] org.apache.hadoop.mapreduce.v2.app.speculate.DefaultSpeculator: DefaultSpeculator.addSpeculativeAttempt -- we are speculating task_1717955085543_0001_m_000000
[DefaultSpeculator background processing] org.apache.hadoop.mapreduce.v2.app.speculate.DefaultSpeculator: We launched 1 speculations.  Sleeping 15000 milliseconds.
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Scheduling a redundant attempt for task task_1717955085543_0001_m_000000
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000000_1 TaskAttempt Transitioned from NEW to UNASSIGNED
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:1 ScheduledReds:2 AssignedMaps:14 AssignedReds:0 CompletedMaps:2 CompletedReds:0 ContAlloc:16 ContRel:0 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=4 release= 0 newContainers=0 finishedContainers=0 resourcelimit=<memory:0, vCores:1> knownNMs=4
...
[IPC Server handler 11 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000013_0 is : 1.0
[IPC Server handler 5 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000012_0 is : 0.0
[IPC Server handler 6 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000014_0 is : 0.667
[IPC Server handler 7 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000011_0 is : 0.0
[IPC Server handler 12 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000013_0 is : 1.0
[IPC Server handler 9 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000013_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 3
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:1 ScheduledReds:2 AssignedMaps:14 AssignedReds:0 CompletedMaps:3 CompletedReds:0 ContAlloc:16 ContRel:0 HostLocal:14 RackLocal:2
[IPC Server handler 10 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000007_0 is : 0.0
[IPC Server handler 13 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000015_0 is : 1.0
[IPC Server handler 14 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000015_0 is : 1.0
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000015_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 4
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000000_0 is : 1.0
[IPC Server handler 3 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000014_0 is : 1.0
[IPC Server handler 4 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000014_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 5
[IPC Server handler 11 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000008_0 is : 0.0
[IPC Server handler 5 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000000_0 is : 1.0
[IPC Server handler 6 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000000_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000000_0 TaskAttempt Transitioned from RUNNING to SUCCESS_FINISHING_CONTAINER
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Task succeeded with attempt attempt_1717955085543_0001_m_000000_0
*[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Issuing kill to other attempt attempt_1717955085543_0001_m_000000_1
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_m_000000 Task Transitioned from RUNNING to SUCCEEDED
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 6
*[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_m_000000_1 TaskAttempt Transitioned from UNASSIGNED to KILLED
*[Thread-88] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Processing the event EventType: CONTAINER_DEALLOCATE
...
[IPC Server handler 7 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000009_0 is : 0.0
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:2 AssignedMaps:14 AssignedReds:0 CompletedMaps:6 CompletedReds:0 ContAlloc:16 ContRel:0 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=4 release= 0 newContainers=1 finishedContainers=3 resourcelimit=<memory:1536, vCores:3> knownNMs=4
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000003
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000005
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000004
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Got allocated containers 1
... 
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned to reduce
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned container container_1717955085543_0001_01_000018 to attempt_1717955085543_0001_r_000000_0
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:1 AssignedMaps:11 AssignedReds:1 CompletedMaps:6 CompletedReds:0 ContAlloc:17 ContRel:0 HostLocal:14 RackLocal:2
[ContainerLauncher #2] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000003 taskAttempt attempt_1717955085543_0001_m_000013_0
[ContainerLauncher #3] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000005 taskAttempt attempt_1717955085543_0001_m_000015_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000000_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000004 taskAttempt attempt_1717955085543_0001_m_000014_0
[ContainerLauncher #8] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_REMOTE_LAUNCH for container container_1717955085543_0001_01_000018 taskAttempt attempt_1717955085543_0001_r_000000_0
[ContainerLauncher #8] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Launching attempt_1717955085543_0001_r_000000_0
[ContainerLauncher #8] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Shuffle port returned by ContainerManager for attempt_1717955085543_0001_r_000000_0 : 13562
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: TaskAttempt: [attempt_1717955085543_0001_r_000000_0] using containerId: [container_1717955085543_0001_01_000018 on NM: [ip-172-31-63-90.ec2.internal:8041]
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000000 Task Transitioned from SCHEDULED to RUNNING
[IPC Server handler 7 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000010_0 is : 0.0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=1 release= 0 newContainers=1 finishedContainers=1 resourcelimit=<memory:0, vCores:3> knownNMs=4
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000002
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Got allocated containers 1
... 
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned to reduce
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Assigned container container_1717955085543_0001_01_000019 to attempt_1717955085543_0001_r_000001_0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:10 AssignedReds:2 CompletedMaps:6 CompletedReds:0 ContAlloc:18 ContRel:0 HostLocal:14 RackLocal:2
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000001_0 TaskAttempt Transitioned from UNASSIGNED to ASSIGNED
[ContainerLauncher #9] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000002 taskAttempt attempt_1717955085543_0001_m_000000_0
[ContainerLauncher #1] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_REMOTE_LAUNCH for container container_1717955085543_0001_01_000019 taskAttempt attempt_1717955085543_0001_r_000001_0
[ContainerLauncher #1] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Launching attempt_1717955085543_0001_r_000001_0
[ContainerLauncher #1] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Shuffle port returned by ContainerManager for attempt_1717955085543_0001_r_000001_0 : 13562
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: TaskAttempt: [attempt_1717955085543_0001_r_000001_0] using containerId: [container_1717955085543_0001_01_000019 on NM: [ip-172-31-63-90.ec2.internal:8041]
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000001 Task Transitioned from SCHEDULED to RUNNING
[IPC Server handler 13 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000007_0 is : 0.667
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000012_0 is : 1.0
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000012_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 7
[IPC Server handler 17 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000011_0 is : 1.0
[IPC Server handler 19 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000011_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 8
[IPC Server handler 2 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000002_0 is : 0.0
[IPC Server handler 0 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000001_0 is : 0.0
[IPC Server handler 12 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000009_0 is : 0.667
[IPC Server handler 12 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000004_0 is : 0.0
[IPC Server handler 9 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000003_0 is : 0.0
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:10 AssignedReds:2 CompletedMaps:8 CompletedReds:0 ContAlloc:18 ContRel:0 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=1 release= 0 newContainers=1 finishedContainers=2 resourcelimit=<memory:0, vCores:4> knownNMs=4
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000017
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000016
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Got allocated containers 1
*[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Cannot assign container Container: [ContainerId: container_1717955085543_0001_01_000020, AllocationRequestId: -1, Version: 0, NodeId: ip-172-31-52-162.ec2.internal:8041, NodeHttpAddress: ip-172-31-52-162.ec2.internal:8042, Resource: <memory:3072, max memory:6144, vCores:1, max vCores:4>, Priority: 10, Token: Token { kind: ContainerToken, service: 172.31.52.162:8041 }, ExecutionType: GUARANTEED, ] for a reduce as either  container memory less than required <memory:3072, max memory:9223372036854775807, vCores:1, max vCores:2147483647> or no pending reduce tasks.
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:8 AssignedReds:2 CompletedMaps:8 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #7] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000017 taskAttempt attempt_1717955085543_0001_m_000012_0
[ContainerLauncher #7] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000016 taskAttempt attempt_1717955085543_0001_m_000011_0
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000008_0 is : 0.667
[IPC Server handler 5 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000010_0 is : 0.667
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerRequestor: getResources() for application_1717955085543_0001: ask=0 release= 1 newContainers=0 finishedContainers=1 resourcelimit=<memory:3072, vCores:5> knownNMs=4
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000020
*ERROR [RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Container complete event for unknown container container_1717955085543_0001_01_000020
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000007_0 is : 1.0
[IPC Server handler 17 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000007_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 9
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:8 AssignedReds:2 CompletedMaps:9 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000010
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:7 AssignedReds:2 CompletedMaps:9 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #5] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000010 taskAttempt attempt_1717955085543_0001_m_000007_0
[IPC Server handler 19 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000009_0 is : 1.0
[IPC Server handler 20 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000009_0
...
2024-06-09 18:17:15,488 INFO [AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 10
...
[IPC Server handler 6 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000010_0 is : 1.0
[IPC Server handler 7 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000010_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 11
[IPC Server handler 8 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000008_0 is : 1.0
[IPC Server handler 12 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000008_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 12
[IPC Server handler 10 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000004_0 is : 0.667
[IPC Server handler 13 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000002_0 is : 0.667
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:7 AssignedReds:2 CompletedMaps:12 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000012
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000013
... 
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:5 AssignedReds:2 CompletedMaps:12 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #6] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000012 taskAttempt attempt_1717955085543_0001_m_000009_0
[ContainerLauncher #2] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000013 taskAttempt attempt_1717955085543_0001_m_000010_0
[IPC Server handler 14 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000003_0 is : 0.667
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000001_0 is : 0.667
[IPC Server handler 11 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000002_0 is : 1.0
[IPC Server handler 5 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000002_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 13
[IPC Server handler 9 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000001_0 is : 1.0
[IPC Server handler 10 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000001_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 14
[IPC Server handler 13 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000004_0 is : 1.0
[IPC Server handler 14 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000004_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 15
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_m_000003_0 is : 1.0
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_m_000003_0
...
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 16
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:5 AssignedReds:2 CompletedMaps:16 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000011
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:4 AssignedReds:2 CompletedMaps:16 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #3] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000011 taskAttempt attempt_1717955085543_0001_m_000008_0
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000007
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000006
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000009
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000008
...
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:0 AssignedReds:2 CompletedMaps:16 CompletedReds:0 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #0] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000007 taskAttempt attempt_1717955085543_0001_m_000002_0
[ContainerLauncher #8] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000006 taskAttempt attempt_1717955085543_0001_m_000001_0
[ContainerLauncher #0] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000009 taskAttempt attempt_1717955085543_0001_m_000004_0
[ContainerLauncher #8] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000008 taskAttempt attempt_1717955085543_0001_m_000003_0
[IPC Server handler 9 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: MapCompletionEvents request from attempt_1717955085543_0001_r_000000_0. startIndex 0 maxEvents 10000
[IPC Server handler 0 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000000_0 is : 0.0
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: MapCompletionEvents request from attempt_1717955085543_0001_r_000001_0. startIndex 0 maxEvents 10000
[IPC Server handler 18 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000001_0 is : 0.0
[IPC Server handler 22 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000000_0 is : 0.7813089
[IPC Server handler 13 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000001_0 is : 0.96449625
[IPC Server handler 15 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Commit-pending state update from attempt_1717955085543_0001_r_000001_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000001_0 TaskAttempt Transitioned from RUNNING to COMMIT_PENDING
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: attempt_1717955085543_0001_r_000001_0 given a go for committing the task output.
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Commit go/no-go request from attempt_1717955085543_0001_r_000001_0
[IPC Server handler 16 on default port 34655] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Result of canCommit for attempt_1717955085543_0001_r_000001_0:true
[IPC Server handler 17 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000001_0 is : 1.0
[IPC Server handler 19 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_r_000001_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000001_0 TaskAttempt Transitioned from COMMIT_PENDING to SUCCESS_FINISHING_CONTAINER
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Task succeeded with attempt attempt_1717955085543_0001_r_000001_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000001 Task Transitioned from RUNNING to SUCCEEDED
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 17
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Before Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:0 AssignedReds:2 CompletedMaps:16 CompletedReds:1 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Received completed container container_1717955085543_0001_01_000019
[RMCommunicator Allocator] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: After Scheduling: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:0 AssignedReds:1 CompletedMaps:16 CompletedReds:1 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
...
[ContainerLauncher #7] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: Processing the event EventType: CONTAINER_COMPLETED for container container_1717955085543_0001_01_000019 taskAttempt attempt_1717955085543_0001_r_000001_0
[IPC Server handler 20 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Commit-pending state update from attempt_1717955085543_0001_r_000000_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000000_0 TaskAttempt Transitioned from RUNNING to COMMIT_PENDING
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: attempt_1717955085543_0001_r_000000_0 given a go for committing the task output.
[IPC Server handler 23 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Commit go/no-go request from attempt_1717955085543_0001_r_000000_0
[IPC Server handler 23 on default port 34655] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Result of canCommit for attempt_1717955085543_0001_r_000000_0:true
[IPC Server handler 18 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Progress of TaskAttempt attempt_1717955085543_0001_r_000000_0 is : 1.0
[IPC Server handler 22 on default port 34655] org.apache.hadoop.mapred.TaskAttemptListenerImpl: Done acknowledgment from attempt_1717955085543_0001_r_000000_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000000_0 TaskAttempt Transitioned from COMMIT_PENDING to SUCCESS_FINISHING_CONTAINER
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: Task succeeded with attempt attempt_1717955085543_0001_r_000000_0
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskImpl: task_1717955085543_0001_r_000000 Task Transitioned from RUNNING to SUCCEEDED
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Num completed Tasks: 18
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: job_1717955085543_0001Job Transitioned from RUNNING to COMMITTING
[CommitterEvent Processor #1] org.apache.hadoop.mapreduce.v2.app.commit.CommitterEventHandler: Processing the event EventType: JOB_COMMIT
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: Calling handler for JobFinishedEvent 
[AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.JobImpl: job_1717955085543_0001Job Transitioned from COMMITTING to SUCCEEDED
```




[Thread-122] org.apache.hadoop.mapreduce.v2.app.MRAppMaster: Job finished cleanly, recording last MRAppMaster retry
2024-06-09 18:17:36,562 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.MRAppMaster: Notify RMCommunicator isAMLastRetry: true
2024-06-09 18:17:36,562 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.rm.RMCommunicator: RMCommunicator notified that shouldUnregistered is: true
2024-06-09 18:17:36,562 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.MRAppMaster: Notify JHEH isAMLastRetry: true
2024-06-09 18:17:36,562 INFO [Thread-122] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: JobHistoryEventHandler notified that forceJobCompletion is true
2024-06-09 18:17:36,562 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.MRAppMaster: Calling stop for all the services
2024-06-09 18:17:36,563 INFO [Thread-122] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Stopping JobHistoryEventHandler. Size of the outstanding queue size is 0
2024-06-09 18:17:37,013 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copying hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1.jhist to file:/var/log/hadoop-mapreduce/history/hadoop/2024/06/09/000000/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist
2024-06-09 18:17:37,104 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copied to backup location: file:/var/log/hadoop-mapreduce/history/hadoop/2024/06/09/000000/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist
2024-06-09 18:17:37,106 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copying hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1.jhist to hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist_tmp
2024-06-09 18:17:37,143 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copied from: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1.jhist to done location: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist_tmp
2024-06-09 18:17:37,146 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Set historyUrl to http://ip-172-31-53-255.ec2.internal:19888/jobhistory/job/job_1717955085543_0001
2024-06-09 18:17:37,147 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copying hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1_conf.xml to file:/var/log/hadoop-mapreduce/history/hadoop/2024/06/09/000000/job_1717955085543_0001_conf.xml
2024-06-09 18:17:37,158 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copied to backup location: file:/var/log/hadoop-mapreduce/history/hadoop/2024/06/09/000000/job_1717955085543_0001_conf.xml
2024-06-09 18:17:37,159 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copying hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1_conf.xml to hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001_conf.xml_tmp
2024-06-09 18:17:37,203 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Copied from: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001/job_1717955085543_0001_1_conf.xml to done location: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001_conf.xml_tmp
2024-06-09 18:17:37,207 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Moved tmp to done: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001.summary_tmp to hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001.summary
[eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Moved tmp to done: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001_conf.xml_tmp to hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001_conf.xml
2024-06-09 18:17:37,212 INFO [eventHandlingThread] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Moved tmp to done: hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist_tmp to hdfs://ip-172-31-53-255.ec2.internal:8020/tmp/hadoop-yarn/staging/history/done_intermediate/hadoop/job_1717955085543_0001-1717956984817-hadoop-streamjob12540940317434631248.jar-1717957056550-16-2-SUCCEEDED-default-1717956995862.jhist
2024-06-09 18:17:37,213 INFO [Thread-122] org.apache.hadoop.mapreduce.jobhistory.JobHistoryEventHandler: Stopped JobHistoryEventHandler. super.stop()
2024-06-09 18:17:37,213 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl: KILLING attempt_1717955085543_0001_r_000000_0
2024-06-09 18:17:37,237 INFO [AsyncDispatcher event handler] org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl: attempt_1717955085543_0001_r_000000_0 TaskAttempt Transitioned from SUCCESS_FINISHING_CONTAINER to SUCCEEDED
2024-06-09 18:17:37,240 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.rm.RMCommunicator: Setting job diagnostics to 
2024-06-09 18:17:37,240 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.rm.RMCommunicator: History url is http://ip-172-31-53-255.ec2.internal:19888/jobhistory/job/job_1717955085543_0001
2024-06-09 18:17:37,252 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.rm.RMCommunicator: Waiting for application to be successfully unregistered.
2024-06-09 18:17:38,253 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.rm.RMContainerAllocator: Final Stats: PendingReds:0 ScheduledMaps:0 ScheduledReds:0 AssignedMaps:0 AssignedReds:1 CompletedMaps:16 CompletedReds:1 ContAlloc:19 ContRel:1 HostLocal:14 RackLocal:2
2024-06-09 18:17:38,254 INFO [Thread-122] org.apache.hadoop.mapreduce.v2.app.MRAppMaster: Deleting staging directory hdfs://ip-172-31-53-255.ec2.internal:8020 /tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001
2024-06-09 18:17:38,257 INFO [Thread-122] org.apache.hadoop.ipc.Server: Stopping server on 34655
2024-06-09 18:17:38,266 INFO [IPC Server listener on 0] org.apache.hadoop.ipc.Server: Stopping IPC Server listener on 0
2024-06-09 18:17:38,266 INFO [IPC Server Responder] org.apache.hadoop.ipc.Server: Stopping IPC Server Responder
2024-06-09 18:17:38,267 INFO [TaskHeartbeatHandler PingChecker] org.apache.hadoop.mapreduce.v2.app.TaskHeartbeatHandler: TaskHeartbeatHandler thread interrupted
2024-06-09 18:17:38,268 INFO [Ping Checker for TaskAttemptFinishingMonitor] org.apache.hadoop.yarn.util.AbstractLivelinessMonitor: TaskAttemptFinishingMonitor thread interrupted
2024-06-09 18:17:43,268 INFO [Thread-122] org.apache.hadoop.ipc.Server: Stopping server on 43637
2024-06-09 18:17:43,269 INFO [IPC Server listener on 0] org.apache.hadoop.ipc.Server: Stopping IPC Server listener on 0
2024-06-09 18:17:43,269 INFO [IPC Server Responder] org.apache.hadoop.ipc.Server: Stopping IPC Server Responder
2024-06-09 18:17:43,272 INFO [Thread-122] org.eclipse.jetty.server.handler.ContextHandler: Stopped o.e.j.w.WebAppContext@f42336c{mapreduce,/,null,STOPPED}{jar:file:/usr/lib/hadoop-yarn/hadoop-yarn-common-3.3.6-amzn-3.jar!/webapps/mapreduce}
2024-06-09 18:17:43,275 INFO [Thread-122] org.eclipse.jetty.server.AbstractConnector: Stopped ServerConnector@2eb60c71{HTTP/1.1, (http/1.1)}{0.0.0.0:0}
[Thread-122] org.eclipse.jetty.server.session: node0 Stopped scavenging
[Thread-122] org.eclipse.jetty.server.handler.ContextHandler: Stopped o.e.j.s.ServletContextHandler@4792f119{static,/static,jar:file:/usr/lib/hadoop-yarn/hadoop-yarn-common-3.3.6-amzn-3.jar!/webapps/static,STOPPED}

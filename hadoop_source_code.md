


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

  protected Resource getAvailableResources() {
    return availableResources == null ? Resources.none() : availableResources;
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

  @Override
  protected synchronized void heartbeat() throws Exception {
    scheduleStats.updateAndLogIfChanged("Before Scheduling: ");
    List<Container> allocatedContainers = getResources();
    if (allocatedContainers != null && allocatedContainers.size() > 0) {
      scheduledRequests.assign(allocatedContainers);
    }

    int completedMaps = getJob().getCompletedMaps();
    int completedTasks = completedMaps + getJob().getCompletedReduces();
    if ((lastCompletedTasks != completedTasks) ||
          (scheduledRequests.maps.size() > 0)) {
      lastCompletedTasks = completedTasks;
      recalculateReduceSchedule = true;
    }

    if (recalculateReduceSchedule) {
      boolean reducerPreempted = preemptReducesIfNeeded();

      if (!reducerPreempted) {
        // Only schedule new reducers if no reducer preemption happens for
        // this heartbeat
        scheduleReduces(getJob().getTotalMaps(), completedMaps,
            scheduledRequests.maps.size(), scheduledRequests.reduces.size(),
            assignedRequests.maps.size(), assignedRequests.reduces.size(),
            mapResourceRequest, reduceResourceRequest, pendingReduces.size(),
            maxReduceRampupLimit, reduceSlowStart);
      }

      recalculateReduceSchedule = false;
    }

    scheduleStats.updateAndLogIfChanged("After Scheduling: ");
  }

  ...
  
  @SuppressWarnings("unchecked")
  private List<Container> getResources() throws Exception {
    applyConcurrentTaskLimits();

    // will be null the first time
    Resource headRoom = Resources.clone(getAvailableResources());
    AllocateResponse response;
    /*
     * If contact with RM is lost, the AM will wait MR_AM_TO_RM_WAIT_INTERVAL_MS
     * milliseconds before aborting. During this interval, AM will still try
     * to contact the RM.
     */
    try {
      response = makeRemoteRequest();
      // Reset retry count if no exception occurred.
      retrystartTime = System.currentTimeMillis();
    } catch (ApplicationAttemptNotFoundException e ) {
      // This can happen if the RM has been restarted. If it is in that state,
      // this application must clean itself up.
      eventHandler.handle(new JobEvent(this.getJob().getID(),
        JobEventType.JOB_AM_REBOOT));
      throw new RMContainerAllocationException(
        "Resource Manager doesn't recognize AttemptId: "
            + this.getContext().getApplicationAttemptId(), e);
    } catch (ApplicationMasterNotRegisteredException e) {
      LOG.info("ApplicationMaster is out of sync with ResourceManager,"
          + " hence resync and send outstanding requests.");
      // RM may have restarted, re-register with RM.
      lastResponseID = 0;
      register();
      addOutstandingRequestOnResync();
      return null;
    } catch (InvalidLabelResourceRequestException e) {
      // If Invalid label exception is received means the requested label doesnt
      // have access so killing job in this case.
      String diagMsg = "Requested node-label-expression is invalid: "
          + StringUtils.stringifyException(e);
      LOG.info(diagMsg);
      JobId jobId = this.getJob().getID();
      eventHandler.handle(new JobDiagnosticsUpdateEvent(jobId, diagMsg));
      eventHandler.handle(new JobEvent(jobId, JobEventType.JOB_KILL));
      throw e;
    } catch (Exception e) {
      // This can happen when the connection to the RM has gone down. Keep
      // re-trying until the retryInterval has expired.
      if (System.currentTimeMillis() - retrystartTime >= retryInterval) {
        LOG.error("Could not contact RM after " + retryInterval + " milliseconds.");
        eventHandler.handle(new JobEvent(this.getJob().getID(),
                                         JobEventType.JOB_AM_REBOOT));
        throw new RMContainerAllocationException("Could not contact RM after " +
                                retryInterval + " milliseconds.");
      }
      // Throw this up to the caller, which may decide to ignore it and
      // continue to attempt to contact the RM.
      throw e;
    }
    Resource newHeadRoom = getAvailableResources();
    List<Container> newContainers = response.getAllocatedContainers();
    // Setting NMTokens
    if (response.getNMTokens() != null) {
      for (NMToken nmToken : response.getNMTokens()) {
        NMTokenCache.setNMToken(nmToken.getNodeId().toString(),
            nmToken.getToken());
      }
    }

    // Setting AMRMToken
    if (response.getAMRMToken() != null) {
      updateAMRMToken(response.getAMRMToken());
    }

    List<ContainerStatus> finishedContainers =
        response.getCompletedContainersStatuses();

    // propagate preemption requests
    final PreemptionMessage preemptReq = response.getPreemptionMessage();
    if (preemptReq != null) {
      preemptionPolicy.preempt(
          new PreemptionContext(assignedRequests), preemptReq);
    }

    if (newContainers.size() + finishedContainers.size() > 0
        || !headRoom.equals(newHeadRoom)) {
      //something changed
      recalculateReduceSchedule = true;
      if (LOG.isDebugEnabled() && !headRoom.equals(newHeadRoom)) {
        LOG.debug("headroom=" + newHeadRoom);
      }
    }

    if (LOG.isDebugEnabled()) {
      for (Container cont : newContainers) {
        LOG.debug("Received new Container :" + cont);
      }
    }

    //Called on each allocation. Will know about newly blacklisted/added hosts.
    computeIgnoreBlacklisting();

    handleUpdatedNodes(response);
    handleJobPriorityChange(response);
    // Handle receiving the timeline collector address and token for this app.
    MRAppMaster.RunningAppContext appContext =
        (MRAppMaster.RunningAppContext)this.getContext();
    if (appContext.getTimelineV2Client() != null) {
      appContext.getTimelineV2Client().
          setTimelineCollectorInfo(response.getCollectorInfo());
    }
    for (ContainerStatus cont : finishedContainers) {
      processFinishedContainer(cont);
    }
    return newContainers;
  }

  ...

  class ScheduledRequests {
    
    private final LinkedList<TaskAttemptId> earlierFailedMaps = 
      new LinkedList<TaskAttemptId>();
    
    /** Maps from a host to a list of Map tasks with data on the host */
    private final Map<String, LinkedList<TaskAttemptId>> mapsHostMapping = 
      new HashMap<String, LinkedList<TaskAttemptId>>();
    private final Map<String, LinkedList<TaskAttemptId>> mapsRackMapping = 
      new HashMap<String, LinkedList<TaskAttemptId>>();
    @VisibleForTesting
    final Map<TaskAttemptId, ContainerRequest> maps =
      new LinkedHashMap<TaskAttemptId, ContainerRequest>();
    int mapsMod100 = 0;
    int numOpportunisticMapsPercent = 0;

    ...
        // this method will change the list of allocatedContainers.
    private void assign(List<Container> allocatedContainers) {
      Iterator<Container> it = allocatedContainers.iterator();
      LOG.info("Got allocated containers " + allocatedContainers.size());
      containersAllocated += allocatedContainers.size();
      int reducePending = reduces.size();
      while (it.hasNext()) {
        Container allocated = it.next();
        if (LOG.isDebugEnabled()) {
          LOG.debug("Assigning container " + allocated.getId()
              + " with priority " + allocated.getPriority() + " to NM "
              + allocated.getNodeId());
        }
        
        // check if allocated container meets memory requirements 
        // and whether we have any scheduled tasks that need 
        // a container to be assigned
        boolean isAssignable = true;
        Priority priority = allocated.getPriority();
        Resource allocatedResource = allocated.getResource();
        if (PRIORITY_FAST_FAIL_MAP.equals(priority) 
            || PRIORITY_MAP.equals(priority)
            || PRIORITY_OPPORTUNISTIC_MAP.equals(priority)) {
          if (ResourceCalculatorUtils.computeAvailableContainers(allocatedResource,
              mapResourceRequest, getSchedulerResourceTypes()) <= 0
              || maps.isEmpty()) {
            LOG.info("Cannot assign container " + allocated 
                + " for a map as either "
                + " container memory less than required " + mapResourceRequest
                + " or no pending map tasks - maps.isEmpty=" 
                + maps.isEmpty()); 
            isAssignable = false; 
          }
        } 
        else if (PRIORITY_REDUCE.equals(priority)) {
          if (ResourceCalculatorUtils.computeAvailableContainers(allocatedResource,
              reduceResourceRequest, getSchedulerResourceTypes()) <= 0
              || (reducePending <= 0)) {
            LOG.info("Cannot assign container " + allocated
                + " for a reduce as either "
                + " container memory less than required " + reduceResourceRequest
                + " or no pending reduce tasks.");
            isAssignable = false;
          } else {
            reducePending--;
          }
        } else {
          LOG.warn("Container allocated at unwanted priority: " + priority + 
              ". Returning to RM...");
          isAssignable = false;
        }
        
        if(!isAssignable) {
          // release container if we could not assign it 
          containerNotAssigned(allocated);
          it.remove();
          continue;
        }
        
        // do not assign if allocated container is on a  
        // blacklisted host
        String allocatedHost = allocated.getNodeId().getHost();
        if (isNodeBlacklisted(allocatedHost)) {
          // we need to request for a new container 
          // and release the current one
          LOG.info("Got allocated container on a blacklisted "
              + " host "+allocatedHost
              +". Releasing container " + allocated);

          // find the request matching this allocated container 
          // and replace it with a new one 
          ContainerRequest toBeReplacedReq = 
              getContainerReqToReplace(allocated);
          if (toBeReplacedReq != null) {
            LOG.info("Placing a new container request for task attempt " 
                + toBeReplacedReq.attemptID);
            ContainerRequest newReq = 
                getFilteredContainerRequest(toBeReplacedReq);
            decContainerReq(toBeReplacedReq);
            if (toBeReplacedReq.attemptID.getTaskId().getTaskType() ==
                TaskType.MAP) {
              maps.put(newReq.attemptID, newReq);
            }
            else {
              reduces.put(newReq.attemptID, newReq);
            }
            addContainerReq(newReq);
          }
          else {
            LOG.info("Could not map allocated container to a valid request."
                + " Releasing allocated container " + allocated);
          }
          
          // release container if we could not assign it 
          containerNotAssigned(allocated);
          it.remove();
          continue;
        }
      }

      assignContainers(allocatedContainers);
       
      // release container if we could not assign it 
      it = allocatedContainers.iterator();
      while (it.hasNext()) {
        Container allocated = it.next();
        LOG.info("Releasing unassigned container " + allocated);
        containerNotAssigned(allocated);
      }
    }
    ...
    }

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
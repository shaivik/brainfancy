### About Sidekiq Workers

- Worker can be CRON, Delayed, Recurring or Or-Demand. 
  - CRON: Scheduled worker running at specific time of day. 
  - Delayed: Worker scheduled to run after a specific time. 
  - Recurring: Worker running at specific interval. 
  - Or-Demand: Worker running in background asynchronously. 

- Workers can be Computer Intensive and/or Memory Intensive. 
  - Compute Intensive: Workers needing more CPU cycles. These kind of workers generally involves CPU intensive operations like loops, arithmetic operations, logical operations (if-else) and input/output operations where RAM read/writes are involved. 
    - Example: CatalogSeedDataPusher, CatalogDataPusher
      - Kibana: [Link](http://kibana.box8.co.in/app/kibana#/discover?_g=(refreshInterval:(pause:!t,value:0),time:(from:'2022-10-30T01:04:23.239Z',to:'2022-10-30T01:10:23.127Z'))&_a=(columns:!(log_event),filters:!(('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'3a5009f0-414b-11ed-82e1-15016f44671c',key:source_class,negate:!t,params:(query:Catalog_DataStore),type:phrase,value:Catalog_DataStore),query:(match:(source_class:(query:Catalog_DataStore,type:phrase))))),index:'3a5009f0-414b-11ed-82e1-15016f44671c',interval:auto,query:(language:lucene,query:%22non-critical%22),sort:!('@timestamp',desc)))
      - Grafana: [Link](https://grafana.box8.co.in/d/nt8WSwuWm/eks-deployment-new?orgId=1&var-namespace=box8-sidekiq&var-Deployment=box8-sidekiq-non-critical-prod&var-Node=All&var-server=172.0.16.182:9100&from=1667091814055&to=1667092060658)
  - Memory Intesive: Workers that need considerably high footprint of memory in RAM to run to complete its execution.
    - Example: ZomatoLocationUpdateWorker
      - Kibana: [Link](http://kibana.box8.co.in/app/kibana#/discover?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:'2022-10-30T12:18:00.000Z',to:'2022-10-30T12:19:00.000Z'))&_a=(columns:!(log_event),filters:!(('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'3a5009f0-414b-11ed-82e1-15016f44671c',key:method,negate:!t,params:(query:'NoDelayScheduler::Worker._start'),type:phrase,value:'NoDelayScheduler::Worker._start'),query:(match:(method:(query:'NoDelayScheduler::Worker._start',type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'3a5009f0-414b-11ed-82e1-15016f44671c',key:log_event,negate:!t,params:(query:location_updates_for_active_orders),type:phrase,value:location_updates_for_active_orders),query:(match:(log_event:(query:location_updates_for_active_orders,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'3a5009f0-414b-11ed-82e1-15016f44671c',key:vidyoot_params.type,negate:!t,params:(query:MessageTrigger),type:phrase,value:MessageTrigger),query:(match:(vidyoot_params.type:(query:MessageTrigger,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'3a5009f0-414b-11ed-82e1-15016f44671c',key:mailer,negate:!t,params:(query:OrderStatusMailer),type:phrase,value:OrderStatusMailer),query:(match:(mailer:(query:OrderStatusMailer,type:phrase))))),index:'3a5009f0-414b-11ed-82e1-15016f44671c',interval:auto,query:(language:lucene,query:%22non-critical%22),sort:!('@timestamp',desc)))
      - Grafana: [Link](https://grafana.box8.co.in/d/nt8WSwuWm/eks-deployment-new?orgId=1&from=1667129706825&to=1667136364035&var-namespace=box8-sidekiq&var-Deployment=box8-sidekiq-non-critical-prod&var-Node=All&var-server=172.0.16.182:9100)
  - CPU intensive and memory intensive jobs need not be mutually exclusive.

- Workers can have third-party calls involved. Third party calls basically refers to any external/internal services which are being called inside the worker. Please note that internal services do not include **databases, elasticsearch, redis, AWS resources calls**. For the sake of simplicity, we consider these services to be reliable enough to handle any scale.
  - Example: ZomatoLocationUpdateWorker, RouterWorker, OrderTracker, VidyootWorker


### High Level Diagram 

![Copy_of_Sidekiq_Architecture.drawio](https://drive.google.com/file/d/1X3P2JlvVAkyS7TD5bpRNyKHjhvwfboFn/view?usp=share_link)


- The workers need to lie in the above four patterns in the diagram. The 4 patterns are: 
  - **Pattern 1:** Background processing job concerned with only computation job on the data set supplied to it. This has no dependency on external services. It doesn't involve database/redis/elasticsearch/AWS resources read writes as well. Objective of this pattern is to split workers into multiple smaller units of jobs in case such situations arise. 
  - **Pattern 2:** Job concerned with basic operations and making http calls to external services asynchronously(*preferred). Async call is preferred here because it gives us additional flexibility of handling failure of external service by pausing queues and thus preventing starvation in sidekiq. 
  - **Pattern 3:** Job concerned with read/write operations from reliable sources of data with some basic operation on data set. Please ensure this pattern does not promote CPU/Memory intensive operations inside single job, if such cases arise we need to use multiple patterns to ensure don't run into any such issue.
  - **Pattern 4:** Job concerned with bulk operations requiring high (*constraints discussed later*) CPU or memory footprints. This pattern suggests to spawn new pods to ensure job is executed outside the scope of current cluster in which original sidekiq pods are running. This is done to ensure, 
  - **Pattern X:** This is a variant of the Pattern 1, 2, 3, 4, wherein the Pattern 1, 2, 3, 4 is followed by scheduling another worker of any Pattern 1, 2, 3, 4. This becomes useful in case we want to split worker and make constraint bound workers. 


### Constraints 

#### Memory Constraints 

By the use of memory profiler, we can get to know the allocated and retained memory for a worker run. Setting the parameters for worst case, we can get to know the memory footprint in the worst. For every worker, before its deployment in production we need to make sure that worker's allocated memory should not exceed **100 MB**, and retained memory should not exceed **10 MB**. 

**Why these numbers?**

Memory issue arises because when a high memory footprint jobs run in bulk this leads to sidekiq pods scaling, and these pods don't get downscaled because subsequently less memory footprint jobs run and this does not lead to garbage collection. Our sidekiq pods run at 8 GB memory limit which with defined constraint allows 80 jobs in the worst case on a single pod. 


#### CPU/Response Time Constraints 

The number of CPU cycles utilised by sidekiq worker. This is calculated on the basis of average response time x average throughput for a worker. This number cannot be constraint because throughput cannot be controlled and it depends on the scale at which our system is working. Also, the nature in which sidekiq works ensures high throughput and low latency jobs are appropriately handled. 

So, we introduce here response time constraint for sidekiq worker.

- For high throughput jobs **(tp >= 60 rpm)**, response time should not under no circumstances exceed **1s**. This number ensures if we have set concurrency set to 40, all the 60 jobs will get executed in 1.5s if only single pod for sidekiq is operational. 
- For low throughput jobs **(tp < 60 rpm)**, response time should not exceed **5s**. 
- If the job takes more than 5s, we have to be sure that it is bulk in nature and it never runs during operational hours (11 AM to 3 PM).


### Other Considerations 

#### 1. Collapsible Jobs 

Jobs which are of nature such that it is recursively enqueued and only the most recent job needs to be performed, we can mark such jobs are collapsible. 
In our current system we have added a middleware of StaleJobClient and StaleJobStopper which get executed before any job execution. 
To implement StaleJobStopper in the system, please refer to GoogleOrderStatusWorker, NumberMaskingWorker.  


#### 2. Unique Worker 

This strategy can be used in worker when we want to limit concurrent worker execution with same parameters. This ensures a lock is acquired on set of condition and no other worker with that same condition gets enqueued till the lock is released. RouterWorker works on this principle. 

#### 3. Async vs Sync API calls 

In this strategy, we need to consider pulling out the logic of making thirdparty API calls in other worker. This gives us more granular control over the API calls and handling the fallbacks. 

Consider using webhook based architecture in case response is not required synchronously. This can be used in our interservice communication where we sync data and then update successful sync flags at source upon successful response. Example: ProcurementOrderStatusUpdateWorker. For external thirdparty services we would need to confirm if webhook architecture is possible at their end. 

![webhook.drawio__1_](https://drive.google.com/file/d/1hFycS7moLFpPad9ExaHsTI2MR4SbP7Se/view?usp=share_link)


But before doing this we need to consider the amount of data which we will need to store in redis to perform this action. Bulk operations API calls in other worker should be avoided as it might lead to redis limits. 


#### 4. Batching 

We can allow batching for bulk operations, only if batches perform within the constraints discussed above. We would need to reduce batch size and schedule another worker if constraints are not satisfied. 

#### Retries Specification

Sidekiq defaults to 25 retries with back-off between each retry. 25 retries means that the last retry would happen around three weeks after the first attempt (assuming all 24 prior retries failed).

- For non-idempotent worker having no API calls, retry count to 0. 
- For idempotent worker having no API calls, we will be going ahead with 1 retry. 
- For non-idempotent worker having API calls, retries to be handled by faraday only and fallback with count as 2 and uniqueness mechanism need to be implemented in faraday failures and worker uniqueness. 
- For idempotent worker having API calls, retries to handled by faraday only with count as 2.

*Please note idempotent worker means whose repeated execution would not lead to inconsistency in system's state.*

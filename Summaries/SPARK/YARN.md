[Apache Spark Resource Management and YARN App Models - Cloudera Engineering Blog (wpengine.com)](https://clouderatemp.wpengine.com/blog/2014/05/apache-spark-resource-management-and-yarn-app-models/)
- what is the lifetime of worker processes, in which tasks are executed for spark and MR?
	- executors stick around for the lifetime of spark application, even when no jobs are running
		- in spark, many tasks can run concurrently in a single process (executor)
	- while in MapReduce, each task runs in its own process. when a task completes, the process goes away
- what are the pros and cons of such lifetimes in spark and MR? 
	- one downside of lifetime of executor is for the time of the application every executor takes up fixed resources even if the executor is not running a job
		- advantage is that any tasks can immediately run without having to wait for fresh resources to be allocated
	- in MR, for each task to run a process must be spawned - con
	- there will not be any process in MR that will be lingering without having a purpose

- what is the lifetime of driver and what are their uses? 
	- spark relies on ***driver*** for task scheduling and job flow
		- driver process is the same as the client process used to initiate the job
		- in YARN mode, the driver can run on the cluster
		- driver is alive for the time of existence of the application
	- in MR, client process can go away and the job can continue running
- what is resource management?
	- allocation and management of computational resources(including nw bw)
	- resource management involves:
		- ***allocation***: determining how much of each resource should be assigned to each application or job
		- ***scheduling***: deciding the order and priority in which jobs should be executed
		- ***monitoring***: continuously tracking resource usage to ensure that apps do not exceed their allocated resources
		- ***balancing***: distributing workloads evenly across the cluster to prevent any single node from becoming a bottleneck
		- ***isolation***: ensuring that different jobs or applications do not interfere with each other, maintaining performance and security
- what is a [cluster manager]([Cluster manager - IBM Documentation](https://www.ibm.com/docs/en/powerha-aix/7.2?topic=software-cluster-manager))?
	- daemon that runs in each node of a cluster
	- main task of the CM is to respond to unplanned events, such as recovering from sw and hw failures, or user-initiated events
	- changes in the state of the cluster are referred to as cluster events
	- on each node, the cluster manager monitors local hw and sw subsystems for such events
	- in response to such events, CM runs one or more event scripts, such as restart application script
	- cluster managers on all nodes exchange messages to coordinate any actions required in response to an event
- why spark needs a CM and what are the supported CMs?
	- spark supports pluggable cluster management
	- cluster manager is responsible for starting the executor processes
	- spark supports YARN, Mesos and its own standalone cluster manager
- what is YARN and what is the need for YARN? 
	- Yet another resource negotiator
	- used to manage resources in a typical cluster and also schedule jobs
	- YARN enables multiple applications to run simultaneously on the same shared cluster
	- allows applications to negotiate resources based on need
	- earlier in hadoop there was JobTracker which was responsible for resource management among other things
		- this was part of map reduce
	- YARN allows diff processing engines like graph, interactive and stream processing as well to run and process data stored in HDFS
- what are the components of the supported CMs?
	- two components in each of the CMs
	- central master service(YARN ResourceManager), mesos master, Spark master
		- these decide which applications get to run executor processes
		- also where and when they get to run
	- a slave service running on every node (YARN NodeManager, Mesos slave or spark standalone slave) 
		- actually starts the processes required by applications
		- these may also monitor their liveliness and resource consumption
- what are the components in YARN?
	- split up the functionalities of resource management and job scheduling/monitoring into separate daemons is the idea behind YARN
	- global ResourceManager
	- per-application ApplicationMaster(AM) - an application is a job or a group of jobs
	- node manager



[YARN (Hadoop): A primer. Whats YARN? | by Abhinav Vinci | Jul, 2024 | Medium](https://medium.com/@vinciabhinav7/yarn-hadoop-a-primer-a381378768ae)

- what is YARN ResourceManager?
	- responsible for the management of resources including RAMs, CPUs and other resources throughout the cluster
	- obviously a daemon
	- makes resource allocation decisions for various running applications
	- in the medium primer he has mentioned that it schedules applications
- what is YARN NodeManager?
	- per node daemon
	- manages resources on individual nodes
	- also monitors resource usage of containers(CPU, Memory)
	- reports monitored metrics to RM
- what is application master?
	- framework specific library
	- tasked with negotiating resources from the resource manager
	- works with node managers to execute and monitor the tasks
- what is a container?
	- encapsulation of all the resources(CPU, mem, etc.) necessary for running a task 

- what are the benefits of using YARN?
	- multiple diff applications can run at the same time in a cluster without having to worry about resource management
		- possible due to the decoupling of resource management from the processing models
	- support
		- supports multiple data processing fws
	- improved utilization
		- minimizing idle resources
	- enhanced scheduling
		- different policies supported: FIFO, capacity, and fair scheduling
		- this enables fine grained control over how resources are allocated and ensures that critical jobs get the necessary resources when needed
	- Data Locality awareness
		- designed to take advantage of data locality
		- tries to schedule tasks on nodes where the data resides
		- reduces data movement and improves processing speeds
	- isolation
		- multiple users and apps can share the same cluster without interfering with each other
	- fault tolerance
		- distributing resource management and app monitoring tasks
		- making it easier to recover from node failures



- what are the two components in resource manager?
	- resource manager has two main components
		- scheduler: allocates resources to various running applications based on the configured policies
		- Applications manager: manages application lifecycle, including starting the application master and monitoring the resource usage of jobs
- what is application master?
	- responsible for application scheduling throughout the life cycle


- what is YARN Scheduler?
	- ![[Pasted image 20240731183902.png]]
	- 
- how YARN works?
	- job submission: the client submits a job to the resource manager, which assigns it to an appropriate application master
	- application master initialization: the resource manager allocates ***a container*** for the application master on ***one of the nodes***. The AM then starts and initializes
	- ***resource request***: ***app master*** requests additional resources(containers) from the ***resource manager*** for executing tasks
	- resource allocation: resource manager allocates containers on different nodes based on availability and resource policies
	- ***task execution***: the application master communicates with the ***node manager*** to launch tasks in the allocated containers. Each node manager manages the lifecycle of these tasks
	- ***monitoring and completion***: the application master monitors the progress of tasks and resource usage
- 
[How to design a Read Heavy system ? Some strategies and best practices | by Abhinav Vinci | Medium](https://medium.com/@vinciabhinav7/how-to-design-a-read-heavy-system-some-strategies-and-best-practices-20e416a77cfd)
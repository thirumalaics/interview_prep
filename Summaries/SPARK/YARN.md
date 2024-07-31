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

- what is YARN and what is the need for YARN? 
	- Yet another resource negotiator
	- used to manage resources in a typical cluster and also schedule jobs
	- YARN enables multiple applications to run simultaneously on the same shared cluster
	- allows applications to negotiate resources based on need
	- resource allocation/management is central to YARN
	- earlier in hadoop there was JobTracker which was responsible for resource management among other things
		- this was part of map reduce
	- 
- https://learning.oreilly.com/library/view/yarn-essentials/9781784391737/ch01.html#ch01lvl1sec08
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
- what are the components of the CMs which are supported?
	- two components in each of the CMs
	- central master service(YARN ResourceManager), mesos master, Spark master
		- these decide which applications get to run executor processes
		- also where and when they get to run
	- a slave service running on every node (YARN NodeManager, Mesos slave or spark standalone slave) 
		- actually starts the processes required by applications
		- these may also monitor their liveliness and resource consumption



- what is YARN ResourceManager?
	- responsible for the management of resources including RAMs, CPUs and other resources throughout the cluster
	- obviously a daemon
	- makes allocation decisions
	- resource manager has two main components
		- scheduler: 
		- applications manager
- what is application master?
	- responsible for application scheduling throughout the life cycle
- what is YARN NodeManager?
	- responsible for the supply and isolation of resources on this node

- what is YARN Scheduler?
	- ![[Pasted image 20240731183902.png]]
	- 
- Why run on YARN?
	- allows us to dynamically share and centrally configure the same pool of cluster resources between all fws that run on YARN
		- we can throw our entire cluster to run a MR job, then use some of it on an impala query and the rest of spark application without any changes in config
	- we can take advantage of YARN schedulers for categorizing, isolating, prioritizing workloads
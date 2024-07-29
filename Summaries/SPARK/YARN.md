- executors stick around for the lifetime of spark application, even when no jobs are running
	- one downside of this is for the time of the application every executor takes up fixed resources even if the executor is not running a job
	- advantage is that any tasks can immediately running without having to wait for fresh resources to be allocated
		- while in mapreduce, each task runs in its own process. when a task completes, the process goes away
		- in spark, many tasks can run concurrently in a single process (executor)
- spark relies on driver for task scheduling and job flow
	- driver process is the same as the client process used to initiate the job
	- in YARN mode, the driver can run on the cluster
	- driver is alive for the time of existence of the application
- in MR, client process can go away and the job can continue running


- what is a cluster manager?
	- daemon that runs in each nod
- spark supports pluggable cluster management
- cluster manager is responsible for starting the executor processes
- spark supports YARN, Mesos and its own standalone cluster manager
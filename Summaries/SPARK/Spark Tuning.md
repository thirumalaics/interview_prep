[Part I](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-1/)
- concurrent tasks execute the same operations
- each stage contains a sequence of transformations that can be completed without shuffling the full data
- [coalesce and repartition](https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce)
	- coalesce is an optimized version of repartition
	- allows avoiding data movement for decreasing number of RDD partitions
	- coalesce avoids a full shuffle
		- as per the given value to coalesce, spark keeps the data on these many number of partitions
		- moves the data off the extra nodes on to the nodes where data did not move
		- repartition internally calls same method as coalesce but with shuffle argument set to True
		- both are narrow transformation
		- repartition algo creates new partitions with data distributed evenly
		- whereas coalesce may result in partitions with different amounts of data
			- and remember unequal sized partitions are generally slower to work with than equal sized partitions
- at each stage boundary, data is written to disks by tasks in the parent stages and then fetched over the nw by tasks in the child stage
	- involves heavy disk and nw IO
- avoid GBK when performing an associative reductive operation
	- sum, count
	- gbk transfers the entire dataset across the nw, rbk will compute local sums for each key in each partition and shuffles
- when shuffles are better
	- when the data arrives in large unsplittable files
		- repartitioning can will allow operations that come after it to leverage more of the cluster's CPU

[PART II](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/)
#### Tuning resource allocation
- every executor in an application has the same fixed number of cores and heap size
- --executor-cores or `spark.executor.cores`
	- determines max num of concurrent tasks an executor can run
- `spark.executor.memory`
	- impacts the amount of data Spark can cache, as well as the maximum sizes of the shuffle data structures
- `spark.executor.instances`
	- num of executors requested
	- we can avoid setting this by turning dynamic allocation `spark.dynamicAllocation.enabled`
	- dynamic allocations enables a Spark application to request executors when there is a backlog of pending tasks and frees up executors when idle
- YARN has configurations that constrain the amount of resources that spark can use
	- `yarn.nodemanager.resource.memory-mb` controls the max sum of memory used by ***containers(plural)*** on each node
	- `yarn.nodemanager.resource.cup-vcords` controls the maximum sum of cores used by the ***containers(plural)*** on each node
- full mem requested to YARN for each executor `spark.yarn.executor.memoryOverhead` + `spark.executor.memory`
- yarn may also increment the requested memory up a little
- running executors with too much mem often results in excessive garbage collection delays, good upper limit is 65 gb
	- total mem available is an important factor affecting GC performance
	- throughput is [inversely proportional](https://docs.oracle.com/en/java/javase/17/gctuning/factors-affecting-garbage-collection-performance.html) to the amount of mem available
- shuffle partitions
- cluster with 6 nodes, 16 cores and 64 GB mem/ node
	- `yarn.nodemanager.resource.memory-mb` = 63GB * 1024
	- `yarn.nodemanager.resource.cpu-vcores` = 15
	- we avoid allocating 100% resources to the containers themselves, because the node needs some resources to run the OS and Hadoop daemons
		- so a gig and a core for these system processes on every machine
- the instinctive approach would be:
	- --num-executors 6 --executor-cores 15 --executor-memory 63G
		- this means
			- 1 executor per available node
			- all cores available to each executor
			- all available memory allocated to the executor alone
		- what this configuration does not consider
			- ![[Executor memory#^dd1ddf]]
				- if all mem available to container is allocated just to the executor, it leaves out: memoryOverhead, pyspark memory, offheap size(if enabled)
			- if we assign one executor per core with 15 cores, concurrency problem comes up
			- we do not account for the driver, where will it be located if executors consumed all the available resource
- will the number of executors be equally distributed across the available nodes?
- better config:
	- --num-executors 17 --executor-cores 5 --executor-memory 19G
		- 17 / 6(nodes) = 3 executors per node except one 
		- why this is a better option:
		    - starting with mem, 19G per executor
			    - since there with be max 3 executors per node, total executor mem for each node is (3\*19) = 57 this leaves out 3.99 specifically for memory overhead, we might also have to allocate mem for pyspark(ignored here)
			    - so consideration for memory other than the executor mem is there
			- driver is accounted for, as the node with 2 executors will be hosting the driver
			- each executor has a good 5 cores 
- memory available to each task is a factor of the executor cores
	- (`spark.executor.memory` * `spark.shuffle.memoryFraction` * `spark.shuffle.safetyFraction`)/`spark.executor.cores`
	- these shuffle configs are not seen in the spark3+ doc
- much of the blog was filled with components that are not applicable to dataframes
- many configs mentioned are also not found in doc


[Spark 3 tuning]([How does Apache Spark 3.0 increase the performance of your SQL workloads - Cloudera Blog](https://blog.cloudera.com/how-does-apache-spark-3-0-increase-the-performance-of-your-sql-workloads/))
- the following are the problems AQE solves
### flaw in the initial catalyst design
![[Pasted image 20240706101615.png]]
- first stage has an optimum number of partitions for the first stage
- for the second stage since shuffling is needed, it uses the default 200
- bad for three reasons
	- if the processing continues after the second stage we could miss potential opportunities for more opportunities
	- if we write the output of that second stage to disk, we may end up with 200 small files
- what we can do is to manually set the shuffle partitions
	- setting this b4 each query is tedious
	- these values will become outdated as the data evolves
	- this setting will be applied to all shuffles in our query
- spark does not know what will be the size of the output of stage 1 before hand in order to determine optimum num of partitions
	- before AQE the plan that was generated was considered final, there were no refinements
### AQE Design principle
- AQE principle is not to make the execution plan final
	- allow for reviews at each stage boundary
- catalyst now stops at each stage boundary to try and apply additional optimizations given the information available on the intermediate data
- drawbacks
	- execution stops at each stage boundary for spark to review its plan
	- Spark UI is more difficult to read because spark creates more jobs for a given app and those jobs do not pick up the job group and description we set
		- what is job group and description










- in yarn running mode, the default for the following are:
	- [reference](https://spark.apache.org/docs/latest/running-on-yarn.html) for the following
		- spark.executor.instances = 2
		- spark.driver.cores has a default of 1
	- [reference](https://spark.apache.org/docs/latest/configuration.html) for the following
		- spark.executor.cores has a default of 1 in YARN mode
		- spark.driver.memory has a default of 1GB
		- spark.executor.memory has a default of 1GB
- [Spark Task Memory allocation - Stack Overflow](https://stackoverflow.com/questions/45553492/spark-task-memory-allocation)
[Distribution of Executors, Cores and Memory for a Spark Application running in Yarn: | spark-notes (spoddutur.github.io)](https://spoddutur.github.io/spark-notes/distribution_of_executors_cores_and_memory_for_spark_application.html)
[Advanced Apache Spark Training - Sameer Farooqui (Databricks) (youtube.com)](https://www.youtube.com/watch?v=7ooZ4S7Ay6Y)
[apache spark - What is the relationship between a Node, Worker, Executor, Task and Partition - Stack Overflow](https://stackoverflow.com/questions/68560515/what-is-the-relationship-between-a-node-worker-executor-task-and-partition)
https://stackoverflow.com/questions/71922559/in-spark-is-it-better-to-have-many-small-workers-or-few-bigger-workers

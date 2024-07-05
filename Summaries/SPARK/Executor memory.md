## What is a VM?
- is no different than any physical computer
- it has cpu, mem, disks and nw-ing capability
- VMs are sw defined computers within physical servers, existing only as code
- sw based version of a computer with resources borrowed from a physical host computer
- VM is a computer file, typically called an image that behaves like an actual computer
	- specs of the VM goes in the image
- partitioned from the host system - isolated
## What is JVM?
- virtual machine that enables a computer to run
	- java programs
	- programs written in other languages that are also compiled to Java bytecode
- does not understand java source code
- understands \*.class
	- class files are compiled \*.java files
	- class files contains byte codes that JVM understands
- JVM is the entity that makes java to be a portable language
- there are specific implementations of the JVM for different systems(windows, Linux, mac)
## JVM Heap
- now what the hell is heap?
- it is an area in memory where java objects reside
- heap is created when the JVM starts up
- the size of this area can increase or decrease in size while the app runs
- when the heap becomes full, garbage is collected
- JVM uses more memory than just the heap
## Executor
- executor: JVM process launched on a worker node
	- important to understand JVM mem management
	- categorized into two types
		- On-heap memory management(in-heap memory): Objects are allocated on the JVM heap and bound by GC
		- off-heap memory management: objects are created in memory outside JVM by serialization, managed by the OS
			- not bound by GC
		- read write speed comparison:
			- on-heap > off-heap > disk
		- is off-heap within the memory available to JVM?
			- uses [direct-memory portion]([Configuring Off-Heap Store (softwareag.com)](https://documentation.softwareag.com/terracotta/terracotta_440/webhelp/bigmemory-go-webhelp/index.html#page/bmg-webhelp/co-tiers_configuring_offheap_store.html)) of the JVM
			- there is confusion, because above, off-heap memory is described as mem outside JVM
## Mem Management
- divided into two types:
	- Static memory manager
		- deprecated due to lack of flexibility
	- Unified mem manager(default from spark 1.6.0)
- in both memory managers, portion of Java Heap is allocated for processing spark apps
- rest is used for Java class references and md usage

## SMM
- traditional model
- divides mem into two fixed partitions statically
	- size of storage, execution and other memory is fixed during application processing
	- configurable before app start
## UMM
- provides dynamic memory allocation
- allocates region of memory as a unified memory container 
	- shared by execution and storage
- borrowing of memory is possible bw storage and execution
	- storage can borrow unused execution memory
	- execution can borrow storage memory irrespective of whether it is in use or not
- additional to heap and off-heap, there is external process memory
	- this kind of memory is mainly used for PySpark and SparkR applications
	- these processes reside outside the JVM
#### Heap memory
- by default spark uses on-heap mem only
	- this size is configured by `spark.executor.memory` parameter when the spark application starts
	- can be set while initiating spark session
- the concurrent tasks running inside executor share the JVM's on-heap memory
- spark supports three mem regions within an executor
	- reserved memory
		- used to store spark internal objects
		- executor mem should be at least 1.5 times reserved memory, other wise the app will fail stating that the exec mem is too low
		- still this value is hardcoded
		- https://stackoverflow.com/questions/78692374/spark-executor-memory-overhead

	- user memory
		- set by (exec mem - reserved)\*(1 - `spark.memory.fraction`)
		- spark internal md, including 
		- used for user defined objects like UDfs, hashmap
			- stores RDD dependency and transformations info
			- data in here cannot spill into disk
			- throws OOM error if objects exceed the given space
			- user defined data structures: custom classes, collections or any other structures required for processing logics
			- md and data structures related to RDD partitions may reside in user mem
			- any external lib used, their objects and data structures will be allocated from user mem
			- md related to the broadcast variables reside in user mem
	- spark memory
		- memory pool managed by Spark
		- used to store intermediate states while doing task execution
		- cached persistent data stored here in storage mem
		- set by (executor mem - reserved)\*`spark.memory.fraction`
		- spark mem = storage mem + execution mem
			- storage mem = `spark.memory.storageFraction` \* spark mem
				- LRU to clear out old objects
			- execution mem = 1- storage mem
				- used to store objects required during the execution of spark tasks
					- ex: stores hash map for hash agg step
				- blocks from this pool cannot be forcefully evicted by other threads
				- evicted immideately after each operation
				- used for shuffles, joins, sorts and aggs

#### off-Heap memory
- allocate memory objects(serialized to byte array) to memory outside the heap of JVM
	- directly managed by the OS and not the Virtual machine
	- garbage collector does not have access to this
- slower read writes to this space
- user has to deal with managing the allocated memory
- by default off-heap memory is disabled
- off heap mem utilization can be seen in UI
- execution memory and storage mem once the off heap mem is abled
	- exec mem and storage mem =  inside heap + outside heap
- offheap mem usage can improve performance as it is safe from GC
- when an executor is killed, all cached data for that executor would be gone but with off-heap mem, the data would persist
- The maximum memory size of container to running executor is determined by the sum of `spark.executor.memoryOverhead`, `spark.executor.memory`, `spark.memory.offHeap.size` and `spark.executor.pyspark.memory`. ^dd1ddf
	- [ref](The maximum memory size of container to running executor is determined by the sum of `spark.executor.memoryOverhead`, `spark.executor.memory`, `spark.memory.offHeap.size` and `spark.executor.pyspark.memory`.)


- there is YARN memory overhead
	- 
	- this causes OOM errors
	- off-heap mem allocated to executor
		- this is a wrong statement with proofs that I have now
			- memory over head is not part of off-heap
			- https://stackoverflow.com/questions/63561233/spark-memory-overhead
	- stores spark internal objects, language specific objects
		- thread stacks - maintains state of individual threads in a multithread app
		- [Java NIO direct buffers]([Java NIO (oracle.com)](https://docs.oracle.com/en/java/javase//21/core/java-nio.html)) - 
			- buffers: containers for data, and other structures such as charsets, channels and selectable channels
			- charsets are mapping bw bytes and unicode characters
			- channels represent connections to entities capable of performing IO operations
		- [interned strings](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/#:~:text=for%20example%20for%20interned%20Strings%20and%20direct%20byte%20buffers.)
	- by default 10% of executor mem or 384 mb, whichever is higher
	- [Resolve the error "Container killed by YARN for exceeding memory limits" in Spark on Amazon EMR | AWS re:Post (repost.aws)](https://repost.aws/knowledge-center/emr-spark-yarn-memory-limit)
		- reducing the num of cores reduces the max number of tasks
			- which reduces the amount of memory required
- `spark.executor.pyspark.memory`
	- amount of mem allocated to pyspark in each executor
	- in MB unless specified otherwise
	- if set, pyspark mem will be limited to this number
	- if not set, spark will not limit python's mem use
		- it is app's responsibility to avoid exceeding the overhead memory space shared with other non-JVM processes
		- when pyspark is run in YARN or Kubernetes, this memory is added to executor resource requests
[Spark Memory Management - Cloudera Community - 317794](https://community.cloudera.com/t5/Community-Articles/Spark-Memory-Management/ta-p/317794#toc-hId-1674349369)
[pyspark - What is user memory in spark? - Stack Overflow](https://stackoverflow.com/questions/74586108/what-is-user-memory-in-spark)
[(6) Apache Spark Memory Management: Deep Dive | LinkedIn](https://www.linkedin.com/pulse/apache-spark-memory-management-deep-dive-deepak-rajak/)
https://stackoverflow.com/questions/77812120/what-does-data-is-stored-in-deserialized-format-in-spark-mean
[Apache Spark: Tackling Out-of-Memory Errors &Memory Management | LinkedIn](https://www.linkedin.com/pulse/apache-spark-tackling-out-of-memory-errors-memory-management-kumar/)
https://michaelheil.medium.com/understanding-common-performance-issues-in-apache-spark-deep-dive-data-spill-7cdba81e697e
- spark.yarn.executor.memoryOverhead is deprecated!!!!
	-  changed to `spark.executor.memoryOverhead`, which is common for YARN and Kubernetes
## What is a VM?
- VMs are ***sw defined computers*** within physical servers, existing only as code
	- is no different than any physical computer
	- it has cpu, mem, disks and nw-ing capability
- resources borrowed from a physical host computer
- VM is a computer file, typically called an image that behaves like an actual computer
	- specs of the VM goes in the image
- partitioned from the host system - isolated
## What is JVM?
- virtual machine that enables a computer to run
	- class files that contain java bytecode
	- programs written in other languages that are also compiled to Java bytecode
- does not understand java source code
- understands \*.class
	- class files are compiled \*.java files
	- class files contains byte codes that JVM understands
- JVM is the entity that makes java to be a portable language
- there are specific implementations of the JVM for different systems(windows, Linux, mac)
## What is JVM Heap?
- it is an area in memory where java objects reside
- heap is created when the JVM starts up
- the size of this area can increase or decrease in size while the app runs
- when the heap becomes full, garbage is collected
- JVM uses more memory than just the heap
	- not too sure atm by this statement
## What is JVM off-heap?
- directly managed by the OS and not the JVirtual machine
- garbage collector does not have access to this
- user has to deal with managing the allocated memory
## What is interning of Strings?
## What is off-heap space used for in Spark?
- in spark, off-heap memory used for certain use-cases(interning of strings and JVM overheads)
- as per caching level, off-heap memory can be used to store ***serialized dataframes***
- if we are running out of memory, we can try to use [off-heap memory more](https://medium.com/@sathamn.n/spark-on-heap-and-off-heap-memory-in-pyspark-7016b48f8512#:~:text=On%2DHeap%20memory%20refers%20to,managed%20by%20the%20operating%20system.%E2%80%9D)

## How does JVM manages it's memory?
- two ways:
	- On-heap memory management(in-heap memory)
	- off-heap memory management: objects are created in memory outside JVM by serialization, managed by the OS
		- not bound by GC
	- read write speed comparison:
		- on-heap > off-heap > disk
	- is off-heap within the memory available to JVM?
		- uses [direct-memory portion]([Configuring Off-Heap Store (softwareag.com)](https://documentation.softwareag.com/terracotta/terracotta_440/webhelp/bigmemory-go-webhelp/index.html#page/bmg-webhelp/co-tiers_configuring_offheap_store.html)) of the JVM
		- there is confusion, because above, off-heap memory is described as mem outside JVM
- ![[Pasted image 20240723192659.png]]
## How is executor memory managed?
- executor: JVM process launched on a worker node
	- important to understand JVM mem management
	
## What are the two types of executor mem managers?
- Static memory manager
	- deprecated due to lack of flexibility
	- can be activated using a config
- Unified mem manager(default from spark 1.6.0)
- in both memory managers, portion of Java Heap is allocated for processing spark apps
- rest is used for Java class references and md usage


![[Pasted image 20240723192944.png]]
##  What is SMM?
- static memory manager
- traditional model
- divides mem into two fixed partitions statically - storage and execution
	- size of storage, execution and other memory is fixed during application processing
		- even in the article, he mentioned [other memory]([Spark Memory Management - Cloudera Community - 317794](https://community.cloudera.com/t5/Community-Articles/Spark-Memory-Management/ta-p/317794#toc-hId-1674349369)) - what is other memory?
	- configurable before app start

## What is UMM?
- unified memory manager
- provides dynamic memory allocation
- allocates region of memory as a unified memory container 
	- shared by execution and storage
	- unified because this memory is shared by execution and storage
- borrowing of memory is possible bw storage and execution
	- storage can borrow unused execution memory
	- execution can borrow storage memory irrespective of whether it is in use or not
- additional to heap and off-heap, there is external process memory
	- this kind of memory is mainly used for PySpark and SparkR applications
	- these processes reside outside the JVM

## what is pyspark memory?
## How is heap memory used in Spark?
- by default spark uses on-heap mem only
	- this size is configured by `spark.executor.memory` parameter when the spark application starts
	- can be set while initiating spark session
- the concurrent tasks running inside executor share the JVM's on-heap memory

## How is the total container memory split up in Spark?
- ![[Pasted image 20240723195226.png]]

- The maximum memory size of container to running executor is determined by the sum of `spark.executor.memoryOverhead`, `spark.executor.memory`, `spark.memory.offHeap.size` and `spark.executor.pyspark.memory`. ^dd1ddf
	- [ref](The maximum memory size of container to running executor is determined by the sum of `spark.executor.memoryOverhead`, `spark.executor.memory`, `spark.memory.offHeap.size` and `spark.executor.pyspark.memory`.)

## How is the heap memory segregated by Spark?
- spark supports three mem regions within an executor' heap memory
	- ***reserved memory***
	- ***user memory***
		- set by (exec mem - reserved)\*(1 - `spark.memory.fraction`)
	- ***spark memory***
		- memory pool managed by Spark
		- set by (executor mem - reserved)\*`spark.memory.fraction`
		- spark mem = storage mem + execution mem
			- storage mem = `spark.memory.storageFraction` \* spark mem
				- LRU to clear out old objects
			- execution mem = 1- storage mem
				- blocks from this pool cannot be forcefully evicted by other threads
				- evicted immediately after each operation
				
## What is reserved memory used for?
- used to store spark internal objects
- this value is hardcoded
- https://stackoverflow.com/questions/78692374/spark-executor-memory-overhead

## What is user memory used for?
- spark internal md
- used for user defined objects like UDfs, hashmap
- stores RDD dependency and transformations info
- user defined data structures: custom classes, collections or any other structures required for processing logics
- md and data structures related to RDD partitions may reside in user mem
- any external lib used, their objects and data structures will be allocated from user mem
- md related to the broadcast variables
- data in here cannot spill into disk
- throws OOM error if objects exceed the given space
## What is the constraint that relates reserved memory to the total executor memory?
- remember that executor memory comprises reserved memory
- executor mem should be at least 1.5 times ***reserved memory***, other wise the app will fail stating that the exec mem is too low
## What is the use of Spark mem and it's components?
- used to store intermediate states while doing task execution
- cached persistent data stored here in storage mem
- execution memory
	- used to store objects required during the execution of spark tasks
		- ex: stores hash map for hash agg step
	- used for shuffles, joins, sorts and aggs

## What are the characteristics of off-Heap memory in spark?
- by default off-heap memory is disabled
- off heap mem utilization can be seen in UI
- execution memory and storage mem once the off heap mem is abled
	- exec mem and storage mem =  inside heap + outside heap
	- sourced from cloudera blog
- off-heap mem usage can improve performance as it is safe from GC
- when an executor is killed, all cached data for that executor would be gone but with off-heap mem, the data would persist

## How is total off-hep memory split up?
- until spark 2.x(inclusive), total off-heap memory = spark.executor.memoryOverhead(spark.offHeap.size included within)
	- what this means is over head should account for [offHeap.size](https://medium.com/walmartglobaltech/decoding-memory-in-spark-parameters-that-are-often-confused-c11be7488a24#:~:text=The%20amount%20of%20off%2Dheap%20memory%20used%20by%20Spark%20to%20store%20actual%20data%20frames%20is%20governed%20by%20spark.memory.offHeap.size) as well
		- this is used to store dataframes and/or broad cast vars exclusively
- from spark 3.x, total off heap memory = spark.executor.memoryOverhead + spark.offHeap.size
- in conclusion, in the most recent version(3.x), total off heap mem is sum of memory overhead and off heap size(if enabled)

## What is memory overhead and what is it used for?
- set by spark.executor.memoryOverhead
	- this causes OOM errors - not sure about that
	- this memory is used
	- off-heap mem allocated to container
		- [Difference between "spark.yarn.executor.memoryOverhead" and "spark.memory.offHeap.size" - Stack Overflow](https://stackoverflow.com/questions/58666517/difference-between-spark-yarn-executor-memoryoverhead-and-spark-memory-offhea/61723456#61723456)
		- https://stackoverflow.com/questions/63561233/spark-memory-overhead
	- accounts for [VM overheads](https://stackoverflow.com/questions/49988475/why-increase-spark-yarn-executor-memoryoverhead)
		- with whatever I have searched, I understand that VM overhead is the memory required by the VM to start
	- stores spark internal objects, language specific objects
		- thread stacks - maintains state of individual threads in a multithread app
		- [Java NIO direct buffers]([Java NIO (oracle.com)](https://docs.oracle.com/en/java/javase//21/core/java-nio.html)) - 
			- buffers: containers for data, and other structures such as charsets, channels and selectable channels
			- charsets are mapping bw bytes and unicode characters
			- channels represent connections to entities capable of performing IO operations
		- [interned strings](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/#:~:text=for%20example%20for%20interned%20Strings%20and%20direct%20byte%20buffers.)
			- this means storing only one copy of each distinct string which must be immutable
			- the distinct value will then be referenced as required
	- by default 10% of executor mem or 384 mb, whichever is higher
	- [Resolve the error "Container killed by YARN for exceeding memory limits" in Spark on Amazon EMR | AWS re:Post (repost.aws)](https://repost.aws/knowledge-center/emr-spark-yarn-memory-limit)
		- reducing the num of cores reduces the max number of tasks
			- which reduces the amount of memory required

## What is pyspark memory and what is it used for?
- `spark.executor.pyspark.memory`
	- amount of mem allocated to pyspark in each executor
	- in MB unless specified otherwise
	- if set, pyspark mem will be limited to this number
	- if not set, spark will not limit python's mem use - from official doc
		- it is app's responsibility to avoid exceeding the overhead memory space shared with other non-JVM processes
		- when pyspark is run in YARN or Kubernetes, this memory is added to executor resource requests
- [In spark what is the meaning of spark.executor.pyspark.memory configuration option? - Stack Overflow](https://stackoverflow.com/questions/68249294/in-spark-what-is-the-meaning-of-spark-executor-pyspark-memory-configuration-opti)
	- two big parts of spark mem management(SMM)
		- Memory inside JVM
			- divided into 4 different parts
			- storage mem - spar cached data, broad cast variables
			- execution memory - this memory is for storing data required during execution of spark tasks
			- User memory - custom data structures, UDfs
			- reserved memory: mem for spark's internal purposes
		- Memory outside JVM
			- off heap mem: outside JVM but for JVM purposes, and can be used for project tungsten
			- external process mem: used by processes that reside out of JVM, like Python or R processes
	- --executor-memory = mem allocated inside Java heap
	- spark.executor.pyspark.memory is part of external process memory
		- responsible for how much mem py daemon will be able to use
		- one use of py daemon is for executing UDfs in python
	- spark.python.worker.memory
		- py4j bridge exposes objects between JVM and python
		- JVM process and python process communicate to each other with py4j
		- the config above determines how much mem can be occupied by py4j for creating objects b4 spilling them in to disk
[Spark Memory Management - Cloudera Community - 317794](https://community.cloudera.com/t5/Community-Articles/Spark-Memory-Management/ta-p/317794#toc-hId-1674349369)
[pyspark - What is user memory in spark? - Stack Overflow](https://stackoverflow.com/questions/74586108/what-is-user-memory-in-spark)
[(6) Apache Spark Memory Management: Deep Dive | LinkedIn](https://www.linkedin.com/pulse/apache-spark-memory-management-deep-dive-deepak-rajak/)
https://stackoverflow.com/questions/77812120/what-does-data-is-stored-in-deserialized-format-in-spark-mean
[Apache Spark: Tackling Out-of-Memory Errors &Memory Management | LinkedIn](https://www.linkedin.com/pulse/apache-spark-tackling-out-of-memory-errors-memory-management-kumar/)
https://michaelheil.medium.com/understanding-common-performance-issues-in-apache-spark-deep-dive-data-spill-7cdba81e697e


[Decoding Memory in Spark — Parameters that are often confused | by Sohom Majumdar | Walmart Global Tech Blog | Medium](https://medium.com/walmartglobaltech/decoding-memory-in-spark-parameters-that-are-often-confused-c11be7488a24) - explore

https://stackoverflow.com/questions/32621990/what-are-workers-executors-cores-in-spark-standalone-cluster

https://stackoverflow.com/questions/63561233/spark-memory-overhead
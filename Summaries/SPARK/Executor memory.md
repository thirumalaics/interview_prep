## What is a VM?
- is no different than any physical computer
- it has cpu, mem, disks and nwing capability
- VMs are sw defined computers within physical servers, existing only as code
- sw based version of a computer with resources borrowed from a physical host computer
- VM is a computer file, typically called an image that behaves like an actual computer
- partitioned from the host system - isolated
## What is JVM?
- virtual machine that enables a computer to run
	- java programs
	- programs written in other languages that are also compiled to Java bytecode
- does not understand java source code
- understands \*.class
	- class files are compiled \*.java files
	- class files contains byte codes that JVM understands
- this is the entity that makes java to be a portable language
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
- allocates region of memory as a unified memory container that is shared by execution and storage
- borrowing of memory is possible bw storage and execution
	- storage can borrow unused execution memory
	- execution can borrow storage memory irrespective of whether it is in use or not
- additional to heap and off-heap, there is external process memory
	- kind of memory is mainly used for PySpark and SparkR applications
	- these processes reside outside the JVM
#### Heap memory

[Spark Memory Management - Cloudera Community - 317794](https://community.cloudera.com/t5/Community-Articles/Spark-Memory-Management/ta-p/317794#toc-hId-1674349369)

[(6) Apache Spark Memory Management: Deep Dive | LinkedIn](https://www.linkedin.com/pulse/apache-spark-memory-management-deep-dive-deepak-rajak/)
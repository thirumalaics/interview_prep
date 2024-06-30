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
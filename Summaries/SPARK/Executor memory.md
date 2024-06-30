## What is JVM?

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
			
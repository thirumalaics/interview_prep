![[Pasted image 20240816093625.png]]

[Project Tungsten and Catalyst SQL optimizer | by saurabh goyal | Medium](https://medium.com/@goyalsaurabh66/project-tungsten-and-catalyst-sql-optimizer-9d3c83806b63#bypass)
- goal of tungsten is to improve the memory and CPU efficiency of the spark apps
	- why specifically CPU and mem?
		- because there two are the bottle necks rather than IO and nw communication
		- hws have improved, faster nws, high IO disks such as SSD
		- spark's IO has been optimized as well
		- many workloads now avoid significant disk IO by pruning input data that is not needed in the given job
		- new shuffle and nw layer implementations
		- data formats have improved, densely storing data
			- parquet, binary data formats
			- this puts pressure on CPU to deserialize and decompress
		- serialization and hashing are CPU bound bottlenecks
			- workloads spend more time here
			- hashing for sorting, comparing and partitioning
- how tungsten improves CPU and mem efficiency
	- manual mem management and binary processing
		- leverage application semantics to manage memory explicitly
		- eliminate overhead of JVM object model and garbage collection
		- instead of representing objects using the JVM object model we use our knowledge of schema to directly layout objects in mem
		- how this benefits us, fitting more data into mem and avoiding some of the GC overheads
	- cache aware optimizations
		- when data fits in mem, the mem access patterns still matter
		- the difference between doing a random mem access and a sequential scan could be quite significant
		- algos and ds to exploit mem hierarchy
		- when we control the mem layout, we can make it in a way that it is friendly for CPU cache efficiency
	- code gen: 
		- exploit modern compilers and CPUs, allow efficient operation directly on binary data
- a string 'abcd' would take 4 bytes to store using UTF-8 encoding
	- in java, 48 bytes required for storing the above
		- java object header tracking md about the object
		- space for the hash code
		- array of chars to represent each char of the string
			- this char array itself has hash code, header and other overheads
			- in the array each char is stored with 2 bytes
- using the knowledge of data schema to directly layout the memory ourselves
	- gets rid of GC
	- helps serialize data in less memory
- there are encoders available for primitive types and product types(case classes) are supported by importing sqlContext.implicits._ for serializing data
- aggregation and sorting can be don over serialized data


[Deep Dive into Project Tungsten Bringing Spark Closer to Bare Metal -Josh Rosen (Databricks) (youtube.com)](https://www.youtube.com/watch?v=5ajs8EIPWGI)
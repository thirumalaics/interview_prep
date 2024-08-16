![[Pasted image 20240816093625.png]]

[Project Tungsten and Catalyst SQL optimizer | by saurabh goyal | Medium](https://medium.com/@goyalsaurabh66/project-tungsten-and-catalyst-sql-optimizer-9d3c83806b63#bypass)
- goal of tungsten is to improve the memory and CPU efficiency of the spark apps
	- why specifically CPU and mem?
		- because there two are the bottle necks rather than IO and nw communication
- a string 'abcd' would take 4 bytes to store using UTF-8 encoding
- JVM's native string implementation stores this differently to facilitate more common workloads
	- encodes each char using 2 bytes UTF-16 encoding and each string object also contains 12 byte header and 8 byte hash code
- using the knowledge of data schema to directly layout the memory ourselves
	- gets rid of GC
	- helps serialize data in less memory
- there are encoders available for primitive types and product types(case classes) are supported by importing sqlContext.implicits._ for serializing data
- aggregation and sorting can be don over serialized data
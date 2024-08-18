https://stackoverflow.com/questions/37871194/how-to-tune-spark-executor-number-cores-and-executor-memory
https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-1/
https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/
https://sparkbyexamples.com/spark/spark-driver-maxresultsize/

- how to do ascending nulls first?
- does a running spark application detect newly arrived data files?
	- data that arrives after few operations have completed?


https://stackoverflow.com/questions/24622108/apache-spark-the-number-of-cores-vs-the-number-of-executors
- the most intuitive thing would be to think that less num of executors with high cores mean that less shuffling
- number of data nodes: 3 
	- each comes with
	- cores: 8
	- RAM: 32 GB
- i should allow cores for driver
- i should also make room for pyspark mem, yarn mem overhead and off heap memory(if enabled)
- yarn mem overhead: 10% of 32 gb = 3.2GB per node will go in YMO
- per executor we leave out 1GB of mem and one core for OS use and hadoop daemons
- 32-3.2-1 = 27.8 GB


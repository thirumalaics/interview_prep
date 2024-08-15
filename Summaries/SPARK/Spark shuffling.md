- primary goal of shuffling in spark is data ***re distribution***
- why do we need to redistribute, what is the need?
1. increase or decrease number of partitions:
	- existing number of partitions are not sufficient to maximize usage of available resources
	- where existing partitions are too heavy to be computed without memory overruns
	- where existing partitions are too high in number
		- task scheduling overhead becomes the bottleneck in the processing time
	- when there is a need to change the num of partitions, shuffling is done
		- on the distributed data collection via commonly available ***repartition API*** among RDDs, Datasets and Df
2. perform aggregation/join on a data collection
	- all records belonging to an agg key or join key need to reside in a single partition
	- if the existing partitioning scheme does not satisfy this condition, then re-distribution becomes mandatory

- partitioner and number of shuffle partitions:
	- two important aspects of shuffling
	- number of shuffle partitions specifies the number of output partitions after the shuffle is executed on a data collection
	- partitioner decides which record goes to which partition(out of all the available output partitions)
	- Spark APIs which trigger shuffling provide both of the above details implicitly or explicitly
	- ![[Pasted image 20240706185706.png]]
	- Spark provides two implementations of partitioners: Hash and Range partitioners
	- hash partitioner - output partition decided based on the hash code generated for the given key(s)
		- most of the Spark APIs requiring shuffling implicitly provision the Hash partitioner
	- Range partitioner - partition for a record decided based on comparison of key value against the range of key values estimated for each of the shuffled partition
		- very limited ops use this
		- he linked for the [ops]([Overview (Spark 3.5.1 JavaDoc) (apache.org)](https://spark.apache.org/docs/latest/api/java/index.html?org%2Fapache%2Fspark%2Fsql%2FDataset.html=))
	- one can define their own partitioner and use it for shuffling in limited RDD APIs
		- not possible for Df and DS
	- provision of number of shuffle partitions varies bw RDD and Dataset/df APIs
		- in case of RDD, num are either implicitly assumed or given explicitly as argument
		- in DS API, `spark.sql.shuffle.partitions` decides the number of shuffle partitions for most of the APIs requiring shuffling
			- default is 200
		- in few DS API requiring shuffling, we can give it explicitly
- what is shuffle write?
	- writing of serialized data on all executors before transmitting
- what is shuffle read?
	- reading serialized data on all executors at the beginning of a stage
- what is shuffle block?
	- uniquely identifies a block of data which belongs to a single shuffled partition and is produced from executing shuffle write operation(by ShuffleMap task) on a single input partition
	- a shuffle block comes from a single input partition's shuffle write operation
	- a shuffle block belongs to a single shuffled partition
	- there is a one-to-one mapping that goes both ways
		- i/p partition <- > block <-> shuffled partition
		- remember this block belongs to either of these partitions
	- unique identifier for the shuffle block: (shuffleId, MapID, ReduceId)
	- shuffle id uniquely identifies each shuffle write/read stage in a spark application
	- map id uniquely identifies each of the input partition
	- reduceId uniquely identifies each of the shuffled partition
	- ![[Pasted image 20240814200412.png]]
- 
 [Revealing Apache Spark Shuffling Magic | by Ajay Gupta | The Startup | Medium](https://medium.com/swlh/revealing-apache-spark-shuffling-magic-b2c304306142)
 https://www.slideshare.net/slideshow/spark-shuffle-introduction/43046270
For Shuffle read and write
https://stackoverflow.com/questions/27276884/what-is-shuffle-read-shuffle-write-in-apache-spark#:~:text=Shuffling%20means%20the%20reallocation%20of,the%20beginning%20of%20a%20stage.
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
	- a shuffle block is hosted in a disk file on cluster nodes, and is either serviced by the block manager of an executor, or via external shuffle service
	- all shuffle blocks of a shuffle stage are tracked by mapoutputtracker hosted in the driver
		- If the status of a Shuffle block is absent against a shuffle stage tracked by MapOutPutTracker, then it leads to ‘MetadataFetchFailedException’ in the reducer task corresponding to ReduceId in Shuffle block. Also, failure in fetching the shuffle block from the designated Block manager leads to ‘FetchFailedException’ in the corresponding reducer task.
- shuffle read/write
	- a shuffle operation introduces a pair of stage in a spark application
	- shuffle write happens in one of the stage while shuffle read happens in subsequent stage
	- shuffle write executed independently for each of the input partition which needs to be shuffled
	- shuffle read executed independently for each of the shuffled partition
	- shuffle write operation is executed mostly using either sortshufflewriter(for rdds) or unsafeshufflewriter(for DF, DS)
		- sortshuffle writer there is only one consolidated shuffle data file sorted by reduce partitions
	- both shuffle writers produce an index file and a data file
		- corresponding to each ip partition to be shuffled
	- index file contains locations inside the data file for each of the shuffled partition
	- data file contains actual shuffled data records ordered by shuffled partitions
	- shuffle read operation is executed using BlockStoreShuffleReader which first queries for all relevant shuffle blocks and their locations
	- this is then followed by pulling or fetching of the blocks from respective locations using block manager module
	- shuffle operation generally involves remote fetches of shuffle blocks over network
- shuffle spill
	- before writing to a final index and data file, a buffer is used to store the data records(while iterating over the input partition) in order to sort the records on the basis of targeted shuffled partitions
	- if the memory limits of this buffer is breached, the contents of the buffer are first sorted and then spilled to disk in a temporary shuffle file
		- this process is called as shuffle spilling
		- if the breach happens multiple times, multiple spill files could be created during the iteration process
		- after iteration, these spilled files are read again and merged to produce final shuffle index and data file
	- on the shuffle read side as well there is a buffer
		- data records in shuffle blocks being fetched are required to be sorted on the basis of key values in records
		- if the buffer size is breached, sort and spill records to disk
		- after all shuffle blocks are fetched, all spilled files are again read and merged to generate final iterator of the data records for further use
	- these spills incur disk write costs and ser/deser cycles(in case where data records are java objects)
	- shuffle spill metric is also available

https://0x0fff.com/spark-architecture-shuffle/
- spark.shuffle.manager parameter determines the shuffle implementation
	- there are many implementations
	- hash, sort, tungsten-sort
	- sort operation is default from spark 1.2.0
	- hash shuffle was default b4
		- creates many files
		- each mapper task creates a separate file for each separate reducer
		- so total files on the cluster = M\*R
			- num of mappers * num of reducers
			- number of reducers here means number of partitions on the reduce side
		- later some optimizations were included
		- i have skipped much of the stuff in hash shuffle implementation because it is deprecated
		- pros:
			- fast, no sorting required
![[Pasted image 20240815093134.png]]

 [Revealing Apache Spark Shuffling Magic | by Ajay Gupta | The Startup | Medium](https://medium.com/swlh/revealing-apache-spark-shuffling-magic-b2c304306142)
 https://www.slideshare.net/slideshow/spark-shuffle-introduction/43046270
For Shuffle read and write
https://stackoverflow.com/questions/27276884/what-is-shuffle-read-shuffle-write-in-apache-spark#:~:text=Shuffling%20means%20the%20reallocation%20of,the%20beginning%20of%20a%20stage.
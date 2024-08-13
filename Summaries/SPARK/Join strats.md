## Broadcast Hash join
- happens in two phases
	- small dataset broadcasted to all executors
	- small dataset is hashed in all executors and joined with the partitioned big dataset
- does not require sort operation
- hash join phase
	- small dataset which was broadcasted to the executors will be hashed by key into buckets 
	- once the small dataset is hashed and bucketed, keys from the big dataset will be attempted to match only with the respective buckets
	- now within this bucket only we search for records that match the join condition
		- we cannot definitely say that all records in this bucket will be part of the result
		- because depending on the number of buckets, there might be collisions
		- the benefit of using hash is that it reduces the amount of records to be scanned for each row from the other table
- broadcast join only works for equi joins
- works for all join except full outer joins
- if the smaller dataset itself is large, chances are we might face issues while
	- collecting the dataset in driver as a prerequisite of broadcast
	- or when generating hashmap in the executor


## Shuffle Hash join
- shuffle both datasets so the same keys from both sides end up in the same partition or task
- after shuffling, smallest of the two will be hashed into buckets and a hash join is performed within the partition
- does not support non equi join
- does not require sortable join key

## Sort merge join

- does not support non equi join


[Using join hints in Spark SQL - AWS Prescriptive Guidance (amazon.com)](https://docs.aws.amazon.com/prescriptive-guidance/latest/spark-tuning-glue-emr/using-join-hints-in-spark-sql.html#:~:text=The%20Shuffle%20Hash%20join%2C%20as,is%20performed%20within%20the%20partition.)
[How nested loop, hash, and merge joins work. (youtube.com)](https://www.youtube.com/watch?v=-htbah3eCYg)
[Crack Interview Problems in an Animated Way (youtube.com)](https://www.youtube.com/watch?v=pJWCwfv983Q&t=23s)
[Different Types of Spark Join Strategies | by ONGCJ | Medium](https://medium.com/@ongchengjie/different-types-of-spark-join-strategies-997671fbf6b0)
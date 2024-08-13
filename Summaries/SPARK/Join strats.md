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
- 


[How nested loop, hash, and merge joins work. (youtube.com)](https://www.youtube.com/watch?v=-htbah3eCYg)
[Crack Interview Problems in an Animated Way (youtube.com)](https://www.youtube.com/watch?v=pJWCwfv983Q&t=23s)
[Different Types of Spark Join Strategies | by ONGCJ | Medium](https://medium.com/@ongchengjie/different-types-of-spark-join-strategies-997671fbf6b0)
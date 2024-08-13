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

## Shuffle Sort merge join
- when both datasets are large
- [Spark Sort Merge Join: Efficient Data Joining : Spark SQL interview questions (youtube.com)](https://www.youtube.com/watch?v=5d9JuBX7kVA&t=21s)
- 3 phases
	- shuffle - both are shuffled
	- sort - records sorted by the join key on both sides
	- merge - both sides of the join condition are iterated based on join key
- records with identical keys are grouped
	- so when the iteration takes place and the key is matched, a cartesian product is performed with the records in this group
	- check data savvy video above
- does not support non equi join
- require sortable join key
- default join

## shuffle-and-replicated nested loop join
- similar to BHJ
- entire partition of the dataset is replicated to all the partitions for a nested loop join
- works for both equi and non-equi
- works only for inner like joins

## broadcast nested loop join (BNLJ)
- broadcasting one of the entire datasets and performing a nested loop to join the data
- every record of dataset 1 is attempted to join with every record from dataset 2
- works for both equi and non-equi
- works for all join types

![[Pasted image 20240813093700.png]]
![[Pasted image 20240813093911.png]]
[Using join hints in Spark SQL - AWS Prescriptive Guidance (amazon.com)](https://docs.aws.amazon.com/prescriptive-guidance/latest/spark-tuning-glue-emr/using-join-hints-in-spark-sql.html#:~:text=The%20Shuffle%20Hash%20join%2C%20as,is%20performed%20within%20the%20partition.)
[How nested loop, hash, and merge joins work. (youtube.com)](https://www.youtube.com/watch?v=-htbah3eCYg)
[Crack Interview Problems in an Animated Way (youtube.com)](https://www.youtube.com/watch?v=pJWCwfv983Q&t=23s)
[Different Types of Spark Join Strategies | by ONGCJ | Medium](https://medium.com/@ongchengjie/different-types-of-spark-join-strategies-997671fbf6b0)
https://towardsdatascience.com/strategies-of-spark-join-c0e7b4572bcf
https://stackoverflow.com/questions/73939433/how-does-cartesian-product-join-transfer-data-internally-for-join-in-spark
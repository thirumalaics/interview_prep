## Broadcast Hash join
- happens in two phases
	- small dataset broadcasted to all executors
	- small dataset is hashed in all executors and joined with the partitioned big dataset
- does not require sort operation
- hash join phase
	- small dataset which was broadcasted to the executors will be hashed by key into buckets 
	- once the small dataset is hashed and bucketed, keys from the big dataset will be attempted to match only with the respective buckets
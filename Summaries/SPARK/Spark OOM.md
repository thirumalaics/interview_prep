
[Spark OOM Error — Closeup. Does the following look familiar when… | by Amit Singh Rathore | The Startup | Medium](https://medium.com/swlh/spark-oom-error-closeup-462c7a01709d)

## User mem
- throws OOM if this limit is exceeded, as spilling is not allowed
- what user mem contains?
	- md and additional data structures related to broadcast variables
	- user-defined data structures and objects created during the course of a spark app
		- custom classes, collections required for our processing logic
		- third-party libraries
	- md and data structures related to RDD partitions
	- spark sql structures, query plans, expression trees and md for SQL or df queries

- Execution mem
	- used for shuffle, join, sort
	- spill to disk in case of allocated mem limit is breached
	- short lived
- storage mem
	- cached objects, broadcast variables
	- upon storage limit breached, spilling happens
- off-heap mem 
	- is used for string internings
	- part of this mem can be used to store serialized dataframes
		- only if off heap mem is enabled
![[Pasted image 20240818083217.png]]
## executor side memory errors
- if memory overhead is exceeded, we may get OOM
	- used for spark internal objects, language specific objects, thread stacks and nio buffers
- high concurrency
	- disproportionate number of cores for executors, we will be processing too many partitions
	- each partition will have its md and memory overhead
- big partitions
	- if an executor is trying to process a big partition it will throw an OOM error
	- big partition may have resulted from decompression or md overhead of file format(exploding parquet file)
- fetch failed exception, increase exec mem or shuffle partitions

## Driver side mem errors
- collect operation
	- limit the result size by using spark.driver.maxResultSize
- broadcast
	- before a relation is broadcasted to executors it is materialized at the the driver node
	- even with single relation, if the size is bigger than the driver's mem it will throw an OOM error
- [rdd - How spark read a large file (petabyte) when file can not be fit in spark's main memory - Stack Overflow](https://stackoverflow.com/questions/46638901/how-spark-read-a-large-file-petabyte-when-file-can-not-be-fit-in-sparks-main)
	- all concurrently loaded partitions must fit into memory, or we will get OOM
	- assuming several stages are there for the current job, spark runs transformations from the first stage on the loaded partitions only
	- once this is done, it stores the output as shuffle-data and then reads in more partitions
	- transformation is applied on these partitions as well
	- this process continues till all data is read and transformed
	- even if we just apply count on the df, spark reads in partitions but in this case it will not write any file
	- on
https://michaelheil.medium.com/understanding-common-performance-issues-in-apache-spark-deep-dive-data-spill-7cdba81e697e
https://medium.com/road-to-data-engineering/spark-performance-optimization-series-2-spill-685126e9d21f

https://stackoverflow.com/questions/78216245/spark-how-is-it-even-possible-to-get-an-oom#:~:text=Here%2C%20the%20OOM%20occurs%20when,it%20throws%20an%20OOM%20error.

[Spark working internals, and why should you care? (anhcodes.dev)](https://anhcodes.dev/blog/tune-spark/)


https://blog.devgenius.io/spark-spill-7e027085ca4c
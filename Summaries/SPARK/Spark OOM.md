
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
	- 

https://michaelheil.medium.com/understanding-common-performance-issues-in-apache-spark-deep-dive-data-spill-7cdba81e697e
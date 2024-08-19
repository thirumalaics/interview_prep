- reserve one node for OS and cluster manager
- let's say 15 cores available per node
	- (num_executors * num_of_cores)
	- ![[Pasted image 20240813204341.png]]
- 1 executor, 15 cores
	- fat executors need a large memory pool to support
	- garbage collection delays will slow down the job
- 15 executors, 1 core
	- they do not take advantage of parallelism that multiple cores within an executor enable
	- finding the optimal amount of overhead mem for single core executors can be difficult
	- spark default overhead memory value will be really small which will cause problems with our jobs
	- fixed overhead for all executors will result in overhead mem being too large and leaving less room for executors
- 5 executors, 3 cores or 3 executors, 5 cores
	- most spark guides say 5 cores
	- 5 cores mean, less overhead mem consuming node mem
	- also better parallelism than 3 executor cores
- memory per node
	- let's say we have 112GB of physical mem available for our 3 executors
	- 10% if overhead: 11.2
	- remaining: 100.8
		- per executor 33.6, round down to 33
- num executors
	- ideal number that accounts for one driver (3x - 1)
- driver mem
	- recommended to be same as driver mem
	- in some cases we may need driver mem to be more than executor mem
	- then use 3x-2 to determine the number of executors for the job
- driver core
	-  default one
	- try same as executors

[Part 3: Cost Efficient Executor Configuration for Apache Spark | by Brad Caffey | Expedia Group Technology | Medium](https://medium.com/expedia-group-tech/part-3-efficient-executor-configuration-for-apache-spark-b4602929262)


[How to tune spark executor number, cores and executor memory? - Stack Overflow](https://stackoverflow.com/questions/37871194/how-to-tune-spark-executor-number-cores-and-executor-memory)
- more than one executor in a node is possible
- when using dynamic allocation, we can give, initial executors, max executors, min executors
	- also interval - if there are pending tasks waiting for more than this time, request. num of executors requested in each round increases exponentially from the previous round
	- another interval to determine which can executor be killed - idle timeout
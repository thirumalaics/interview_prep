![[Pasted image 20240814083612.png]]

- parallel computing means two or more processes are crunching through different parts of the same dataset at the same time
- distributed computing is similar but the processes live on different computers
- our pc can slow down massively if some program is using to much mem
	- when we start a program, OS will start assigning it memory
	- when our program is trying process more data than it has got memory, it will start to spend a lot of time saving and reading data from the disk instead
	- the processor might start accepting other jobs because our program is busy dumping files into disk
		- puts more strain on the available resources
- inter process communication
	- it is not allowed for a process to access another process's assigned memory
		- if I run a python program to create a dictionary I cannot sharing this piece of memory directly to another process
		- that is why we need to write stuff down
			- this writing can be into a file
		- in py, two diff processes can save data from one another as a bunch of bytes on the hard disk(python calls it pickling)
	- IPC is slow
- Java has its own special way of translating code to instructions for the computer - JVM
- pyspark launches two processes
	- python process and JVM process
	- pyspark allows us to use py process to send commands to a JVM process named spark
	- this is needed as Spark is written in Java and scala, but not in python
![[Pasted image 20240814085022.png]]
https://medium.com/analytics-vidhya/how-does-pyspark-work-step-by-step-with-pictures-c011402ccd57

- spark.sparkContext
	- entry point for spark functionality
	- represents connection to a Spark cluster
	- can be used to create RDD and broadcast variables on that cluster
	- but in python, SparkContext is way to communicate with SparkContext object in JVM
		- python spark context te
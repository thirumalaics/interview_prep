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


- spark.sparkContext
	- entry point for spark functionality
	- represents connection to a Spark cluster
	- can be used to create RDD and broadcast variables on that cluster
	- but in python, SparkContext is way to communicate with SparkContext object in JVM
		- python spark context tells the java spark context what we want done
		- python object is a mouth piece through which we talk to Java object
- python is able to make stuff happen inside a JVM process thanks to py4j
- py4j allows py program to
	- open up a port to listen on(25334)
	- start up a JVM
	- make the JVM programme listen on a different nw port(25333)
	- send commands to the Java process and listen for responses
![[Pasted image 20240814085834.png]]

- using pyspark makes our Spark based apps slower
	- as we are performing inter process comms
	- again python process and JVM process cannot have access to the same data in memory
	- so the two process can either write messages in to a file or they can talk to each other over a nw socket
		- both require serialization and deserialization

```
some_string = spark.sparkContext.parallelize("hello hello hello")
some_string.take(5)
```

- what happened here is 
	- python serialized the string and wrote it into a temp file
	- via py4j, communicates to JVM to pick the file and create a Java RDD
	- creates a py variable to store info about the Java RDD
	- JVM reads the file into a collection
	- creates a java rdd object from this collection(partitions and distributes the data in memory)
	- tell py where to find the object in the JVM
	- ask jvm for first five records from the data stored in spark mem
	- returns results via py4j
	- py process unpickles data to display
- what will happen if I use map function of pyspark to run an upper method on the strings and create a new dataframe
	- spark does not map all the functions in the python language to the equivalent functions in JAva/scala
	- [spark creates python worker processes to execute python functions](https://github.com/apache/spark/blob/7dff3b125de23a4d6ce834217ee08973b259414c/core/src/main/scala/org/apache/spark/SparkEnv.scala#L75)
	- spark shares the serialized data and serialized python functions to execute
	- python process unpickles the data and code received from spark, performs the function and serializes and shares the results

https://ankiweb.net/shared/info/2004145278
https://norvig.com/21-days.html#answers

https://norvig.com/



https://medium.com/analytics-vidhya/how-does-pyspark-work-step-by-step-with-pictures-c011402ccd57
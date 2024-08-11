- 10 executors 2 core each or 2 executors 10 core each
- what is the difference between row_number, rank and dense_rank?
	- rank function
		- assigns the same rank for ties
		- next ranking is skipped
	- dense_rank
		- ties are given the same rank
		- ranks are consecutive
- fastest method to find duplicate records
	- distinct will cause results to be ordered
	- [group by](https://www.google.com/search?q=why+group+by+performace+is+better+than+windowing&rlz=1C1RXQR_enIN964IN964&oq=why+group+by+performace+is+better+than+windowing&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIJCAEQIRgKGKABMgkIAhAhGAoYoAEyBggDECEYCtIBCTE1ODEzajBqN6gCALACAA&sourceid=chrome&ie=UTF-8) all our columns
	- https://stackoverflow.com/questions/2129717/how-to-verify-if-two-tables-have-exactly-the-same-data
	- https://stackoverflow.com/questions/5341276/sql-distinct-keyword-bogs-down-performance
- diff between coalesce and repartition

- row_number performance
	- window function return a row for each input row
	- where as group by focuses on data reduction
	- use group by when we want to combine both aggregated and non-aggregated columns in a single query
	- ![[Pasted image 20240123094410.png]]
- how to find if three lines form a triangle
	- the sum of two side lengths of a triangle is always greater than the third side
![[Pasted image 20240811080943.png]]


- how to apply row number function using data frame api
```
from pyspark.sql import Window
from pyspark.sql.functions import row_number
partition = Window.partitionBy("").orderBy("")
df.select(row_number().over(window),"*")
```

- how to identify new, unchanged and updated records between source and target?
- how are nulls handled in bq order by and group by?
	- in bq order by nulls are considered the least possible values, so in ascending they appear first and in descending they appear last
	- what about in spark?
- following is one of the best explanations
	- https://stackoverflow.com/questions/13997177/why-no-windowed-functions-in-where-clauses

https://medium.com/expedia-group-tech/part-3-efficient-executor-configuration-for-apache-spark-b4602929262
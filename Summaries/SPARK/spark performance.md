- use dataset/dataframe over RDD
	- df and ds includes several optimization modules to improve the performance of the spark workloads
- why RDD is slow?
	- spark does not know how to apply the optimization techniques
	- RDDs are opaque
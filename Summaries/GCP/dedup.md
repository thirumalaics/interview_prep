- the table used below had: 3,582,714 records
- what is elapsed time:
- what is slot time consumed
- what is bytes shuffled
- what is bytes spilled to disk?
- 
- when I used UNION DISTINCT to dedup:
![[Pasted image 20240710184944.png]]
![[Pasted image 20240710185244.png]]
- when I used group by to dedup:(had to use all columns in group by other wise I could not have displayed all, did not think of join because it will be one more step bound to increase processing time)
	- ![[Pasted image 20240710185320.png]]![[Pasted image 20240710185344.png]]
- when I used row_number as part of qualify(used only pk in partition by)
	- ![[Pasted image 20240710190238.png]]
	- ![[Pasted image 20240710190323.png]]
- surprisingly row number was the fastest
## how does bq query execution happens
- declarative sql statement converted into graph of execution **query stages**
- query stages are composed of **sets of execution steps**
- heavily distributed parallel architecture to execute query
- stages denote the units of work that many workers may execute in parallel
	- stages communicate with one another by using fast distributed shuffle arch
- [slot](https://cloud.google.com/bigquery/docs/slots#:~:text=A%20BigQuery%20slot%20is%20a,a%20capacity%2Dbased%20pricing%20model.) is a virtual CPU used by BQ to execute sql queries
	- during execution, bq decides how many slots a query requires, depending on the query size and complexity
	- [diff definition](https://cloud.google.com/bigquery/docs/query-plan-explanation?_gl=1*cm6jj3*_ga*Mzg0MzczOTM3LjE3MjAwNzc2Mzk.*_ga_WH2QY8WWF5*MTcyMDYxNjQwNC4xOC4xLjE3MjA2MTgxNDkuMzEuMC4w&_ga=2.111928070.-384373937.1720077639#:~:text=which%20is%20an%20abstracted%20representation%20of%20multiple%20facets%20of%20query%20execution%2C%20including%20compute%2C%20memory%2C%20and%20I/O%20resources) - slot abstracts multiple facets of query execution(compute, mem and IO resources)
- within query plan, mention of work units and workers are used to inform us about parallelism
- top level job statistics provide the estimate of individual query cost using totalSlotMs
- query plan can be modified which the query is running
	- adaptive/dynamic
	- stages can be introduced while a query is running
	- these are usually introduced to improve data distribution throughout query workers
	- in query plans where this occurs, these stages are called repartition stages
- query jobs expose timeline of execution
	- units of work completed, pending and active with query workers
- we can get the query plan info as an api response
	- contains more infor than the UI
- these fields are available for each stage
![[Pasted image 20240710200116.png]]

https://cloud.google.com/bigquery/docs/query-plan-explanation?_gl=1*cm6jj3*_ga*Mzg0MzczOTM3LjE3MjAwNzc2Mzk.*_ga_WH2QY8WWF5*MTcyMDYxNjQwNC4xOC4xLjE3MjA2MTgxNDkuMzEuMC4w&_ga=2.111928070.-384373937.1720077639
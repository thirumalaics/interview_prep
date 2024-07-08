- if a query stage writes more than other stages, filter early in the query
https://cloud.google.com/bigquery/docs/best-practices-performance-compute#avoid-oversharding-tables
https://cloud.google.com/bigquery/docs/multi-statement-queries#temporary_tables
https://cloud.google.com/bigquery/docs/query-insights
https://anywhere.epam.com/en/blog/google-cloud-interview-questions
https://www.naukri.com/code360/library/google-cloud-platform-interview-questions
https://towardsdatascience.com/14-ways-to-optimize-bigquery-sql-for-ferrari-speed-at-honda-cost-632ec705979
https://datacouch.medium.com/optimizations-in-bigquery-bb396b6ecab9
## Reduce data processed
- using the below options we can reduce the data processed
- avoid select \*
	- query only what is needed
	- querying excess columns can incur additional wasted IO and materialization
	- if we need a preview of data, we have a separate section called Preview
	- select specific columns
		- using select with limit does no good
		- billed for reading all bytes in the entire table
		- `SELECT * EXCEPT (salary, id) from thiru.creating1` 
	- use partitioned tables
		- this can read only rows that are needed and in turn reducing IO
		- there is also a mention of the following and I did not understand
			- materializing results in a destination table and querying that table instead
- avoid excessive wildcard tables
	- when querying wildcard tables, we must use the most granular prefix
	- wildcard tables are union of tables that match the wildcard expression
		- so obviously schemas should match
	- wildcard tables are useful if a dataset contains
		- multiple similarly named tables with compatible schemas
		- sharded tables
			- dividing large datasets into separate tables and adding a suffix to each table name
		- the more granular the prefix is the less number of tables read
- avoid date sharded tables
	- aka date named table
	- partitioned tables perform better than date-named tables
	- bq must maintain a copy of schema and md for each date-named table
		- bq needs to verify permission for each and every table when a wildcard table is used
		- adds query overhead
- over-sharding tables
	- bq storage is low cost
	- creating large number of table shards has performance impacts that outweight any cost benefits
	- each table in bq requires bq to maintain schema, md and permissions related to it
- prune partitioned queries
	- filter based on the following columns:
		- for ingestion time partitioned col: `_PARTITIONTIME`
			- ![[Pasted image 20240707185908.png]]
		- time-unit, column-based and integer-range, use the partitioning column
	- group by is also an intensive operation, so use it when it benefits
- reduce data before using a join
	- perform aggs earlier in the query
- use the where clause
	- to limit the amount of data a query returns
	- when possible use filters on BOOL, INT, float or DATE columns
		- typically faster in these columns than STRING


## Optimize Query operations
- avoid repeatedly transforming data
	- if we are using SQL to trim strings or extract data by using regular expressions, it is more performant to materialize results into a table and use it
	- regex requires extra processing
- avoid multiple evaluations of the same CTEs
	- use instead
		- variables
		- procedural language
		- temp tables
		- automatically expiring tables
	- these persist the calculation
	- with clauses are used for query readability, not performance
		- it does not mean bq materialize the CTEs as temp intermediate tables and reuse them
		- WITH clause might be evaluated multiple times within a query, depending on query optimizer decisions
		- query optimizer attempts to detect parts of the query that can be executed once and be reused but it might not always be possible
	- results can be stored in a var depending on the data returned
	- CTEs might not help decrease complexity or resource consumptions
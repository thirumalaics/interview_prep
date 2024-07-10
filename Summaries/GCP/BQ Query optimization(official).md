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
		- the ***more granular*** the prefix is the ***less number of tables read***
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
- avoid repeated joins and subqueries
	- avoid repeatedly joining the same tables and using the same subqueries
	- use nested repeated data to represent the relationships
	- nesting allows us to represent foreign entities in line
		- nested data is represented as a STRUCT type in Google sql
		- `CREATE TABLE thiru.orders (id INTEGER,placed_by STRUCT<cust_id int, name STRING>);
		- ![[Pasted image 20240708093943.png]]
		- `insert into thiru.orders VALUES(1,(1,'thiru'));`
	- repeated data allows us to store data with one-to-many relationships
		- `CREATE TABLE thiru.books_owned (cust_id INTEGER, books ARRAY<STRING>`
		- ![[Pasted image 20240708094150.png]]
		- `insert into thiru.books_owned (1, ['book1', 'book2']);`
			- when inserting into a col with repeated mode, we have to ensure the square brackets
			- when I used round brackets, it gave an error as () are interpreted as struct entries
			- `SELECT books[1], books[Offset(1)], books[ORDINAL(2)] from thiru.books_owned;`
			- ORDINAL is one based indexing, all the above columns access the second element of the array cols books
	- nested repeated fields
		- array of structs
		- `CREATE TABLE thiru.books_owned_ext (cust_id INTEGER,books ARRAY<STRUCT<book_id INTEGER, book_name STRING>>);`
		- `INSERT INTO thiru.books_owned_ext VALUES(1,[(1, 'Harry Potter'),(2, 'fooled by randomness')]);`
		- ![[Pasted image 20240708094646.png]]
		- ![[Pasted image 20240708095240.png]]
		- `SELECT (SELECT book_name from UNNEST(books) limit 1) as x from thiru.books_owned_ext;`
		- displays only the first element
	- nested repeated data saves us the performance impact of the comms bw to represent the relationship
	- saves us the IO costs that we incur by repeatedly reading and writing the same data
	- using the same subqueries multiple time leads to reprocessing
		- materialize
		- small cost of storing the materialized data outweighs the performance impact of repeated IO and query processing
		- materializing our subquery results decreases the overall amount data read and written
- optimize join patterns
	- when joining data from multiple tables, optimize by starting with the largest table
	- join clause should have the biggest table on the left, followed by tables in decreasing order of size in terms of rows
	- when large on the left, small on the right - a broadcast join is created
		- sends all the data in the smaller table to each slot that processes the table

- optimize the order by clause
	- use order by in the outer most query or within a window fn
	- push complex operations to the end of the query
		- regex and mathematical fns
		- reduces the data to be processed before the complex operations are performed
	- use limit clause when we do not need many rows in the output
		- when trying to order a very large result set we may face Resources exceeded error
		- this can be solved if we use `limit`
	- limit the data passed to the window function, if the amount of data passed to the window fn is high, performance will suffer
		- ![[Pasted image 20240708183808.png]]
		- this took 2 seconds

## Split complex queries into smaller ones
- leverage multi-statement query capabilities and stored procedures
	- to split complex queries into smaller components
	- regex, layered subqueries or joins can be slow and intensive
	- trying to fit all computations in one huge select query is some times an anti pattern
	- splitting up can help us materialize intermediate results as variables and temp tables
	- especially when one result is needed in many places
- use int64 data types in joins
	- instead of string to reduce cost and improve comparison performance
	- wider the join column, longer it takes

## Reduce query outputs
- materialize large result sets to a destination table
	- bq limits cached results to approx 10 GB
	- queries that return large results frequently result in `response too large`
		- use filters to limit result set
		- use a limit clause to reduce the result set especially if we are using order by
		- write to a destination table
			- writing has query performance impacts(i/o)

## Avoid anti-sql patterns
- avoid self join
	- use window analytic fn or the pivot operator
	- squares the number of output rows
	- increase in output data can cause poor performance
	- self joins usually used to compute row-dependent relationships
- avoid cross join
	- avoid joins that create more outputs than inputs
	- when a cross join or any join that can give high outputs is required, agg the data
	- window fns are often more efficient than cross joins
	- some times cross joins do not even complete
- avoid dmls that update or insert single rows
	- batch our updates and inserts
- temp tables
	- let's us save intermediate results to a table
	- managed by bq, so we dont have to save or maintain them in a dataset
	- charged for storage of the temp table
	- default expiry time - 24 hrs
	- or we can delete manually in our multi-statement query
	- ![[Pasted image 20240708195255.png]]
	- when we create a temp table, we should not use project or dataset qualifier
		- created in a special dataset
	- we can refer temp tables by name for the duration of the current multi-statement query
		- this includes temp tables created by a procedure within the multi-statement query
		- stored in a special dataset `_script%` with randomly generated names
		- `DROP TABLE table_name;`
	- `_SESSION.temptablename` - explicitly mention that we are referring a temp table
	- bq interprets any request with multiple statements as multi-statement query
- what is qualify
```SELECT

 table_name, ddl

FROM

 `adept-box-428408-s6`.thiru.INFORMATION_SCHEMA.TABLES where table_name = 'creating1';
```

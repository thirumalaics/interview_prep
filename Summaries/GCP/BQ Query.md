- limit is a booby trap
	- speeds up performance by does not reduce cost
	- limit puts a cap on the output rows, we need to move around less data on BQ nw
- Select few columns
- use exists instead of count
	- exits processing cycle as soon as the first matching row is found
- use approx agg fns
	- count distinct will need more mem to keep count of unique ids
		- spilling happens if mem not available
	- error % 1-2
- replace self join with window fn
	- self join squares the number of rows
- order by or join on int 64 columns
	- bytes scanned is less in int
	- ints have adv over strings because of collation
	- [collation](https://stackoverflow.com/questions/4538732/what-does-collation-mean)
		-  tells our db how to sort and compare strings
		- all rules in collation adds to the complexity of string comparison
- optimize anti joins
	- instead of NOT IN, use NOT EXISTS operator to write anti joins because it triggers a more resource friendly query execution plan
	- [NOT IN](https://www.sqlshack.com/t-sql-commands-performance-comparison-not-vs-not-exists-vs-left-join-vs-except/) Triggers a few heavy operators that run nested looping and counting operations
- filter data early and often
- WHERE sequence matters
	- bq assumes that user has provided best order of expressions in the where clause and does not attempt to reorder expressions
	- expressions in where clauses should be ordered with most selective expression first
	- ex: order = operator b4 like operator
- utilize partitions and clusters
- push order by to the end of query
	- 
- delay resource intensive operations
	- LOWER(), TRIM(), CAST, regex and, mathematical
- use SEARCH
	- whether a BQ table or other search data contains a set of search terms
	- returns true if all search terms appear in the data, based on the rules for search query
	- `SEARCH(data_to_search, search_query)`
	- no regex
- take adv of caching

https://freedium.cfd/https://towardsdatascience.com/14-ways-to-optimize-bigquery-sql-for-ferrari-speed-at-honda-cost-632ec705979


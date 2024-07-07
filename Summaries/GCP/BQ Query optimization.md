- if a query stage writes more than other stages, filter early in the query

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
- unenforced key constraints and how they may benefit queries in BQ
	- both PK and fk
- users can define constraints on a table when creating a table using CREATE TABLE statement
	- ALTER TABLE ADD PRIMARY KEY or ALTER TABLE ADD CONSTRAINT
	- use DROP instead of ADD to drop a constraint
```
CREATE TABLE thiru.inventory (

 inv_date_sk INT64 REFERENCES thiru.date_dim(d_date_sk) NOT ENFORCED,

 inv_item_sk INT64 REFERENCES thiru.item(i_item_sk) NOT ENFORCED,

 inv_warehouse_sk INT64 REFERENCES thiru.warehouse(w_warehouse_sk) NOT ENFORCED,

 inv_quantity_on_hand INT64,

 PRIMARY KEY(inv_date_sk, inv_item_sk, inv_warehouse_sk) NOT ENFORCED

);
```
```
ALTER TABLE thiru.xxx
ADD PRIMARY KEY(x,y,z) NOT ENFORCED,
ADD FOREIGN KEY(asd) REFERENCES item(ads) NOT ENFORCED
```
- three query optimizations that leverage key constraints:
	- inner join elimination
	- outer join elimination
	- join reordering
### Inner join elimination
- in the following query, ss_customer_sk is a fk that references PK of customer table's c_customer_sk
- also note that the select statement only has columns from the left hand side table
	- it is guaranteed that the ss_customer_sk's value will either be null(no match with rhs table), or some value(for which there will be only one match on the right)
	- the info stated in () can be inferred only if there is a constraint
![[Pasted image 20240711184255.png]]

## Outer JOIN elimination
- left outer joins are eliminated when the join keys on the rhs are unique and only columns from the left table are selected
- right outer joins are eliminated when the join keys on the lhs are unique and only cols from the right are selected
- when the join key from the other side is unique, we will get utmost one match

- shouldnt the user take care of above queries themselves?
	- in dwh, there are views which usually join fact and dim tables
	- instead of joining over and over, devs query the view
	- each query that accesses the view selects a diff set of columns
	- in that case the above optimizations work

## Join reordering
- when joins can't be eliminated, the query optimizer uses the table constraints to infer information about join cardinalities
- the query optimizer may then use the information when performing join re
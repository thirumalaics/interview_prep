- unenforced key constraints and how they may benefit queries in BQ
	- both PK and fk
- users can define constraints on a table when creating a table using CREATE TABLE statement
	- ALTER TABLE ADD PRIMARY KEY or ALTER TABLE ADD CONSTRAINT
	- use DROP instead of ADD to drop a constraint
```CREATE TABLE thiru.inventory (

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
ADD FOREIGN KEY(asd) REFERENCES 
```
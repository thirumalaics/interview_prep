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
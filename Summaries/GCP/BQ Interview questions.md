https://www.projectpro.io/article/bigquery-interview-questions-and-answers/731
- explain architecture of bq
	- dremel is used as the query processing engine, which compiles our queries into execution trees
	- Colossus is a cluster based storage system that stores data as column based files
	- There is jupiter which is the Google pb network which helps decoupling of compute and storage
	- borg is the job scheduler and resource manager. spins up resources as needed
- difference between legacy and std sql in bq
- try dedup in bq
- https://medium.com/@mrayandutta/enhancing-data-integrity-in-bigquery-mastering-deduplication-with-qualify-and-beyond-2cb48c815ba0#:~:text=1.-,Deduplication%20with%20QUALIFY,particularly%20useful%20for%20large%20datasets.
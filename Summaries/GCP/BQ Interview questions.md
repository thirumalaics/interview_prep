https://www.projectpro.io/article/bigquery-interview-questions-and-answers/731
- explain architecture of bq
	- dremel is used as the query processing engine, which compiles our queries into execution trees
	- Colossus is a cluster based storage system that stores data as column based files
	- There is jupiter which is the Google pb network which helps decoupling of compute and storage
	- borg is the job scheduler and resource manager. spins up resources as needed
- difference between legacy and std sql in bq
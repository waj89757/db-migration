# Data Migration Project
this project is a distributed migration service for mysql.It includes full data migration, incremental data migration and real-time data check.The service can process big-data migration efficiently,exactly.Our companyt use it in expansion and shrink of distributed database,disaster recovery and business service updating.

##Flow Chart

![flow-chart]()


##System Frame
![frame]()


##Full data migration
####1.Two way for Extract
Way 1: extracting from master directly and control IO and Loading of master by config the threshold.
Way 2: copying a slave from master and extract from slave.
####2.Index
We rule only the table upon 10 thousand or 100 mb must have index.generally We extract by primary key.
####3.performance
We divide a db into a lot of table tasks and every node process some of these.A master node control thr program of other slave node.
For one table data, We divide it into some chunks by index and run some threads to process every chunk.

##Incrmental data migration
####1.log the binlog position
incre service start by a position which logged by full data task.
####2.catch incre data
we enhanced canal which a mysql binlog server.incre service can subscribe binlog position and catch up data from canal by RPC.
####3.performance
We choose one thread for one db task in order to the data sequence.We batch pull data and process data to increase performance.

##Full data check
####1.When to start?
incre data service will notify check service starting when it keep migrationg and master in sync.
####2.How to check full data?
first logging the binlog position.
Second extracting and chunking data by index.
Finally compareing and repairing data.
####3.How to compare data?
signaturing row data by MD5 and compare target and master.

##Incre data check
####1.When to start?
it start from binlog position when full data check finished.
####2.How to compare data?
subscribing target and master,Comparing binlog data in a certian range.
Incre data service log target and master binlog position and notify the incre data check to consume these.

##Distrubtion and recovery
Migration Services use master-slave frame.The master node allocate schedules and transfer the fail task.
slave node will report progress to master and master save progrees.If slave failed,master will choose another node to continue task from the point of interruption.


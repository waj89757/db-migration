# Data Migration Project
This project is a distributed migration service for mysql.It includes full data migration, incremental data migration and real-time data check.The service can process big-data migration efficiently,exactly.Our company use it in expansion and shrink of distributed database,disaster recovery and business service updating.

##Flow Chart

![flow-chart](https://github.com/waj89757/db-migration/blob/master/readme/架构图.jpg)


##System Frame
![frame](https://github.com/waj89757/db-migration/blob/master/readme/迁移流程.jpg)


##Full data migration
####1.Two way for Extract
Way 1: Extracting from master directly and controlling IO and Loading of master by configing the threshold.</br>
Way 2: Copying a slave from master and extracting from slave.
####2.Index
We rule the table upon 10 thousand rows or 100 mb must has index.Generally we extract data by primary key.
####3.performance
We divide a schema into a lot of table tasks and every node process some of these.A master node controls the program of all slave nodes.</br>
For one table data, We divide it into some chunks by index and run some threads to process every chunk.

##Incrmental data migration
####1.log the binlog position
Incre service starts by a position which logged by full data task.
####2.catch incre data
We enhanced canal which a mysql binlog server.Incre service can subscribes binlog position and catches up data from canal by RPC.
####3.performance
We choose one thread for one db task in order to the data sequence.We batch pull data and process data to increase performance.

##Full data check
####1.When to start?
Incre data service will notify check service starting when it keep migrationg and master in sync.
####2.How to check full data?
First logging the binlog position.</br>
Second extracting and chunking data by index.</br>
Finally compareing and repairing data.
####3.How to compare data?
Signaturing row data by MD5 and compare target and master.

##Incre data check
####1.When to start?
It start from binlog position when full data check finished.
####2.How to compare data?
Subscribing target and master,Comparing binlog data in a certian range.</br>
Incre data service log target and master binlog position and notify the incre data check to consume these.

##Distrubtion and recovery
Migration Services use master-slave frame.The master node allocate schedules and transfer the fail task.</br>
Slave node will report progress to master and master save progress.If slave fail,master will choose another node to continue task from the point of interruption.



# 数据同步工具
##介绍
mysql数据同步工具，具有全量、增量迁移、数据校验、可定制等功能。可做到大数据的高效、实时迁移，平滑切换、保证数据无误。我公司内部用于分布式数据库的数据扩容缩容，机房灾备以及业务升级。

##迁移流程
![flow-chart](https://github.com/waj89757/db-migration/blob/master/readme/架构图.jpg)

##系统架构
![frame](https://github.com/waj89757/db-migration/blob/master/readme/迁移流程.jpg)

##全量迁移
####1.Extract的两种方案
方案一 可直接在主库上执行extract，可配置迁移阈值以控制主库上的IO和Load。</br>
方案二，从postion点复制一个slave用于全量同步。
####2.大数据索引
全量服务，小表（低于10万，容量小于500mb）可选择无索引的复制方式。但大表必须配置索引， Extract默认根据主键批量获取数据。     
####3.性能
多节点共同完成迁移工作，以表为最小粒度，由主节点分配。</br>
根据索引进行分块，多块并行迁移。 

##增量迁移
####1.记录position
全量迁移开始需记录binlog的position，增量服务从该节点进行迁移
####2.增量数据获取
扩展了阿里巴巴的canal服务，通过订阅binlog拉去增量数据。
####3.性能
需要保持数据顺序行，所以采用单库单线程方式，批量拉数据，批量更新方式。
          
##全量数据校验
####1.何时触发
增量迁移服务发现已经追上主库时，通知校验服务开启全量校验。
####2.全量数据
记录position点，分块，进行merge。
####3.数据merge
通过查询源库和目标库，对数据进行签名，比较，进行修正。
##增量数据校验
####1.何时触发
全量进行完后，根据pos点进行增量校验
####2.如何merge
订阅源库和目标库binlog，再一定区间内进行比对和校验.</br>
增量迁移会记录两个库binlog位点，校验服务通过notify消费。
##分布式与容灾
迁移工具采用主从集群架构。主节点用于分配进度和失效转移。</br>
迁移节点会汇报进度，若中途某个节点异常，可选其他节点进行断点续传。

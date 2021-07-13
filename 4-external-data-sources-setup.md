## External data sources and their setup ##
This page will provide an overview of how to setup external data sources for MySQL, Kafka and Hive (Hadoop).

### MySQL ###
1. Install MySQL (version 8.0.21) from https://www.mysql.com. Select community (GP) version and download **MySQL Community Server** for your OS.

1. Take note during installation, use the same password which you have configured in `etc/catalog/mysql.properties` for the field `connection-password`.

1. For testing, you can load a local data file into MySQL. First you will need to use MySQL CLI and create the right table 1st. Use the MySQL CLI to load the data into the right table.
A copy of the data set can be found [here](./data/MySQL_Dataset.csv).

1. Accessing the data in MySQL from Presto CLI:  
-- Start the Presto server;  
-- Run the Presto CLI and once launched with `presto>` prompt, enter `show catalogs`;  
-- Run `show schemas from mysql` and you should see the new schema created using the MySQL CLI earlier on;  
-- Run `show tables from <schema_name>` to show the table(s) created using MySQL CLI earlier on;  
-- Perform your regular SQL DML once you know the table name.

### Kafka ###
1. Download tar file Kafka from https://kafka.apache.org and unzip the tar file to an installation folder of your choice.

1. Create 2 subfolders `myData/zookeeper` and `myData/kafka` to store the meta data and kafka logs respectively.

1. Change in `config/zookeeper.properties` the value for **dataDir** to the full path to `myData/zookeeper`.

1. Chang ein `config/server.properties` the values for kafka log dir **log.dirs** to the full path to `myData/kafka` and the **listeners** to the same host:port value you have specified in Presto's `etc/catalog/kafka.properties` for **kafka.nodes**.

1. Start zookeeper and start Kafka on the command line.

1. Create new Kakfa topic on the command line:
```
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic testTopic --create --partitions 1 --replication-factor 1
```
The Kafka topic created should match the one specified in Presto's `etc/catalog/kafka.properties` for **kafka.table-names**.

7. Submit messages to **testTopic** using the command line for Kafka producer.

1. Use the console consumer from the command line to check if the messages are submitted to the topic.

1. Start the Presto server and CLI client and execute:
```
presto> show catalogs;

presto> show schemas from kafka;

presto> show tables from kafka.default;
-- You should see the testTopic which was created

presto> select * from kafka.default.testTopic;
-- You should see the messages submitted to the Kafka topic
```

### More Kafka-Presto stuff ###
1. Start zookeeper & Kafka from the command line.
-- Always start zookeeper before starting kafka

1. Create a new topic called **ppeAvailability**

1. Use the console producer from the command line and load data using a pre-populated json data set found [here](./data/Kafka_Dataset.json).

1. Verify the data was loaded using the console consumer command line.

1. Add the new topic to the `/etc/catalog/kafka.properties` file.

1. Start Presto server and CLI client and execute:
```
presto> show catalogs;

presto> show schemas from kafka;

presto> show tables from kafka.default;
-- You should see the ppeAvailability which was created

presto> select * from kafka.default.ppeAvailability;
-- You should see the messages submitted to the Kafka topic
```

7. Use select statements, even nested selects with the appropriate where clauses and apply Presto predefined SQL function in the select statements: `json_extract_message(_message, '$<some_column_name>')`

### Hadoop ###
1. Version in this notes: 2.10.0.

1. Three modes of installation:
- Stand-alone [pre-requisite for ***pseudo-distributed*** mode]
- Pseudo-distributed (used in this notes)
- Fully-distributed (for Production use)

3. Both ***stand-alone*** and ***pseudo-distributed*** can be installed on a single computer and for testing;
***Pseudo-distributed*** requires HDFS to the storage layer and YARN as the resource negotiation.

1. Download Hadoop binary (tar file) from https://hadoop.apache.org and unzip into a location of choice.

1. Under the Hadoop installation:
- `etc` folder: contains various config files;
- `bin` folder: contains utility programs such as the hadoop command;
- `sbin` folder: contains the scripts for starting HDFS and YARN etc.

6. Setup password-less SSH; we need this for pseudo-distributed mode:
- Run `ssh-keygen -t rsa` then check folder `~/.ssh/` and you should see 2 files: `id_rsa` and `id_rsa.pub`;
- Add the public key to the list of authorized keys using `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`;
- Try `ssh localhost` and you should NOT be prompted for password and be in a SSH session.

7. In `etc/hadoop/hadoop-env.sh`:
- Add Java installation path to the bottom of the file, i.e. **export JAVA_HOME=/Library...**;
- Add Hadoop installation path to the bottom of the file, i.e. **export HADOOP_PREFIX=[your installation path]**

8. In `etc/hadoop/core-site.xml`, configure HDFS as the default file system under the **< configuration />** tag.

1. In `etc/hadoop/hdfs-site.xml`, configure the replication factor value, the data directory for the namenode and datanode, under the **< configuration />** tag.

1. In `etc/hadoop/mapred-site.xml`, configure **yarn** as the map reduce framework's negotiator, under the **< configuration />** tag.

1.  In `etc/hadoop/yarn-site.xml`, configure yarn nodemanager to be shuffler of the map reduce jobs.

1. Make tmp directories for the namenode and datanode use, e.g. create `hadoop_tmp/hdfs/namenode` and `hadoop_tmp/hdfs/datanode`.

1. Formant the name node (you only need to do this once) by running `bin/hdfs namenode -format`.

1. Start Hadoop by running `sbin/start-dfs.sh`. This will kick start the NameNode, DataNode and the SecondaryNameNode.

1. Start YARN by running `sbin/start-yarn.sh`. This will kick start the NodeManager and ResourceManager.

1. Run `jps` to make sure all the above processes have been started.

1. To shutdown completely:
- Run `sbin/stop-yarn.sh`;
- Run `sbin/stop-dfs.sh`;
- Run `jps` again to make sure no services are running now.
---

### Hive ###
1. Version in this notes: 2.3.7.

1. Download Hive from https://hive.apache.org by navigating to the FTP folder following the links and unzip into a location of choice. 

1. Navigate under the installation folder, `conf` subfolder and make a copy of `hive-default.xml.template` to `hive-site.xml`.

1. Edit `hive-site.xml`:
- Put the same host:port you have specified in Presto's catalog file for `hive.properties`, under tag `hive.metastore.uris`;
- Inspect and note the database specified under tag `javax.jdo.option.ConnectionURL`; default DB is derby; use proper RDMS for Production;
- Inspect tag `hive.metastore.warehouse.dir` and note the path which points to data location in HDFS;

5. Start Hadoop (see above) by running both `start-dfs.sh` and `start-yarn.sh`

1. Make tmp folder for temp files storage when using Hive, by running `hadoop fs -mkdir /tmp`.

1. Make these user folders, by running `hadoop fs -mkdir /user` & `/user/hive`, `/user/hive/warehouse`. Hive will store everything we create using the above ***warehouse*** directory.

1. Run `hadoop fs -chmod g+w` for all the above user directories.

1. Initiatilise the Derby database by running `schematool -initSchema -dbType derby`.  
NOTE: Hive meta data are stored in the database and accessed via the hive metastore service.

1. Start hive metastore as a service by running `hive --service metastore`.

1. Start Hive by running `hive` and if all goes well, you should see the `hive>` prompt.

1. Start executing HiveQL commands such as:
- `show tables`, `show databases` etc;
- create new table(s) and load data, e.g. ppe availability table;
- do `select` on the table(s) created.

### Presto --> Hive ###
1. Start Hadoop as per above instructions.

1. Start Hive metastore service as per above instructions.

1. Start Presto & Presto CLI as per previous instructions.

1. In `presto>` prompt, run:
```
presto> show catalogs; // you should see hive listed

presto> show schemas from hive;

presto> show tables from hive.default // you should see the new tables created from my previous steps

presto> select * from < new table > ... // execute your queries and you should see the same data that has been stored in Hive
```
---

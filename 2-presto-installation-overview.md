## Installation of Presto ##
This page list an overview of the dependencies and to-dos for installing Presto.
We are not installing Presto as a distributed model here (like in PROD); all installations happen on 1 node (e.g. using our laptop)

### Installing JDK ###
1. Use JDK11.0.11 as most Preso open source libs have been tested with JDK11. Any vendor ones will do.
Note: Amazon has one too - `Amazon Corretto JDK`
- https://trino.io/docs/current/installation/deployment.html;
- Recommended to use `Azul Zulu JDK`;
- Docker image at: https://hub.docker.com/r/trinodb/trino (uses Azul Zulu JDK) 

2. Install Presto (release 331):
- Download from prestosql.io (now known as trino.io | release 359 as of Jul 2021); read why the name change here: https://trino.io/blog/2020/12/27/announcing-trino.html;
- Note there is also PrestoDB (prestodb.io) which has been contributed by FB to the Linux Foundation;
- You will to have **python** installed too as the launch script for Presto uses a `launcher.py` file (why???).

**Under `etc` folder - configuration files**  

3. Create a `etc` folder under the root insallation folder. 

4. Configure `etc/node.properties` file, so that we can setup the `coordinator` and `workers` nodes required for Presto to run:  
-- `node.environment` = [Use same name for all coordinator and workers nodes for the same cluster];  
-- `node.id` = [Unique id per node];  
-- `node.data-dir` = [path where you want Presto store its logs].  

5. Configure `etc/jvm.config` file to configure the JVM properties:  
-- The usual JVM start up parameters such as `-Xmx16G` (max memory), `-server` to run as server mode for the JVM, `-XX:+UseG1GC` to set the garbage collector to use etc.

6. Configure `etc/config.properties` file to configure the Presto server:  
-- `coordinator=true`- means this node will act as the coordinator;  
-- `node-scheduler.include-coordinator=true` - will allow the **coordinator** and **worker** to run on the same node;  
-- `http-server.http.port=8080` - the port to talk to Presto server;  
-- `query.max-memory=5GB` - the amount of max distributed memory usage for entire Presto;  
-- `query.max-memory-per-node=1GB` - the max amount of distributed memory that a query may use;  
-- `query.max-total-memory-per-node=2GB` - the max amount of user memory, that a query may use on any one machine;  
-- `discovery-server.enabled=true` - discovery service on the `coordinator` node, which every Presto instance registers itself with on start up and sends heartbeats to keep its registration alive;  
-- `discovery.uri=[same hostname:port as the coordinator node`] - discovery service shares the same HTTP server with the same Presto instance and hence replace this value to match the hostname and port of the `coordinator`.

7. Configure `etc/log.properties` file to set the logging level.

**Under `etc/catalog` folder - catalog of data sources access files**  
1. Presto comes with binary for connectors to Hive, MySQL, Kafka etc. The `catalog` subfolder we create and the different `.properties` files will ask Presto to create instance of the connectors for the respective data sources.

1. You can create different properties (for different connector instances) to connect to 2 different MySQL backend for example: `mysql1.properties` & `mysql2.properties`.

1. Create a `catalog` folder under `etc`.

1. Create a `hive.properties` file to configure access to a Hive server:  
-- `connector.nme=hive-hadoop2` - Hadoop hive connector type;  
-- `hive.metastore.url=thrift://localhost:9083` - the host and port of your Hiv metastore;  
-- `hive.allow-drop-table=true` - whether the Hive connector is allowed to drop tables;  
-- `hive.allow-rename-table=true` - whether the Hive connector is allowd to rename tables;  
-- `hive.time-zone=UTC` - specify the timezone of the connector.

1. Create a `mysql.properties` file to configure access to a MySQL database:  
-- `connector.name=mysql` - MySQL DB connector type;  
-- `connection-url` - the **jdbc:** connection string;  
-- `connection-user` & `connection-password` - the user credentials to login to the database.  

1. Create a `kafka.properties` file to configure access to a kafka topic:  
-- `connector.name=kafka` - Kafka connector type;  
-- `kafka-nodes=localhost:9092` - the host and port of your kafka installation;  
-- `kafka.table-names` - contains the name of the Kafka topic name you plan to get the data from; put multiple topics here, separated by commas;    
-- `kafka.hide-internal-columns=false` - whether the table schema served through Presto, includes internal columns.

**Starting, stopping & see status of Presto server**
1. Use `launcher.py` with **start**, **stop**, **status** to start, stop and check status of the Presto server.
---
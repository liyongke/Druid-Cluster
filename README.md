# Druid + HDFS + Mysql Cluster Setup

## 1. Download

* [Druid](http://druid.io/downloads.html)

* [Hadoop](https://hadoop.apache.org/releases.html)
 
* [Zookeeper](https://www.apache.org/dyn/closer.cgi/zookeeper/)

* [Mysql](https://www.mysql.com/downloads/)

* [Mysql Connector jar(Optional)](https://dev.mysql.com/downloads/connector/j/)

## 2. Requirements 

* Java, release 1.6 or greater (JDK 6 or greater). 
* ssh trusted access is configured

## 3. Ground Knowledge

* [Architecture about Druid](http://druid.io/docs/latest/design/index.html#architecture)

* [Architecture about Hadoop(HDFS)](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)

## 4. Let's start!

### 4.1 Mysql Setup

1. Create a druid database and user

   * Connect to MySQL from the machine where it is installed.

    ```
     > mysql -u root
  
     Paste the following snippet into the mysql prompt:
  
     -- create a druid database, make sure to use utf8mb4 as encoding
     CREATE DATABASE druid DEFAULT CHARACTER SET utf8mb4;

     -- create a druid user, and grant it all permission on the database we just created
     GRANT ALL ON druid.* TO 'druid'@'localhost' IDENTIFIED BY 'diurd';
  
     Other configurations, please refer to /etc/my.cnf.d/server.cnf.

    ```
    For more info, please refer to https://dev.mysql.com/doc

### 4.2 Hadoop Cluster Setup

1. Configuring Environment of Hadoop Daemons

2. Configuring the Hadoop Daemons
 
   * (Must) etc/hadoop/core-site.xml -- Configurations for master node

    ```
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://ctrl-secondary:9000</value>
     </property>
    ```
    
   * (Must) etc/hadoop/hdfs-site.xml  --  Configurations for NameNode and Datanode
   
    ```
    <property>
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
    <property>
    
    <property>
      <name>dfs.data.dir</name>
      <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
    </property>
    ```
   
   * (Must) etc/hadoop/slaves -- Configurations for cluster enviroment(master and slave nodes)
    ```
    ctrl-secondary ------ master node
    ctrl-primary ------ slave node
    ```

   * Based on requirements, following configuration could be optional and take use of default settings
  
    ```
    > etc/hadoop/yarn-site.xml  --  Configurations for ResourceManager and NodeManager
    > etc/hadoop/mapred-site.xml  --  Configurations for MapReduce 
    ```
     <A href="http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html">
         For details on the above configurations, please refer to official doc here.</A>

3. Copy the whole hadoop folder to slave servers

     * Hadoop will automatically search for the slave nodes based on the hosts configured in slave file above.

4. Hadoop Startup

 ```
 >[hdfs]$ $HADOOP_PREFIX/bin/hdfs namenode -format
 
 >[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemon.sh hdfs start namenode
 
 >[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemons.sh hdfs start datanode
 
 If etc/hadoop/slaves and ssh trusted access is configured (see Single Node Setup), all of the YARN processes can be stopped with a utility script. As yarn:
 >[hdfs]$ $HADOOP_PREFIX/sbin/start-dfs.sh
 
 >[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemon.sh start resourcemanager
 
 >[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemons.sh start nodemanager
 
 If etc/hadoop/slaves and ssh trusted access is configured (see Single Node Setup), all of the YARN processes can be stopped with a utility script. As yarn:
 >[yarn]$ $HADOOP_PREFIX/sbin/start-yarn.sh
 ```
 
5. Hadoop Console

 > Check on the cluster status from the following console websites.
   
 >>   * HDFS Explorer Console : http://<hostname>:50070/dfshealth.html#tab-overview
 
        [HDFS Explorer Console sample](http://10.75.44.107:50070/dfshealth.html#tab-overview)
   
 >>   * Resource Manager Console : http://<hostname>:8088/cluster
 
        [Resource Manager Console sample](http://10.75.44.107:8088/cluster)
 
6. Hadoop Shutdown

 ``` 
 >[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemon.sh hdfs stop namenode
 
 >[hdfs]$ $HADOOP_PREFIX/sbin/hadoop-daemons.sh hdfs stop datanode

 If etc/hadoop/slaves and ssh trusted access is configured (see Single Node Setup), all of the YARN processes can be stopped with a utility script. As yarn:
 >[hdfs]$ $HADOOP_PREFIX/sbin/stop-dfs.sh
 
 >[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemon.sh stop resourcemanager
 
 >[yarn]$ $HADOOP_YARN_HOME/sbin/yarn-daemons.sh stop nodemanager
 
 If etc/hadoop/slaves and ssh trusted access is configured (see Single Node Setup), all of the YARN processes can be stopped with a utility script. As yarn:
 >[yarn]$ $HADOOP_PREFIX/sbin/stop-yarn.sh
 ```

### 4.3 Zookeeper Setup

1. Unzip download tar file

2. In conf/zoo.cfg,
    ```
    tickTime=2000
    dataDir=/var/lib/zookeeper/
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=zoo1:2888:3888
    server.2=zoo2:2888:3888
    server.3=zoo3:2888:3888
    ```
     For more details on [Configuration Parameters](https://zookeeper.apache.org/doc/r3.4.6/zookeeperAdmin.html#sc_configuration)

3. Start a ZooKeeper server

    ```
    bin/zkServer.sh start
 
    or
 
    java -cp zookeeper.jar:lib/slf4j-api-1.6.1.jar:lib/slf4j-log4j12-1.6.1.jar:lib/log4j-1.2.15.jar:conf \ org.apache.zookeeper.server.quorum.QuorumPeerMain zoo.cfg
    ```

### 4.4 Druid Cluster Setup

1. Unzip download tar file

   ```
    |- bin/* - scripts related to the single-machine quickstart
    |- conf/* - template configurations for a clustered setup
        |- _common - shared config file used by the following components
        |- broker - details on http://druid.io/docs/latest/design/broker.html
        |- coordinator - details on http://druid.io/docs/latest/design/coordinator.html
        |- historical - details on http://druid.io/docs/latest/design/historical.html
        |- middleManager - details on http://druid.io/docs/latest/design/middleManager.html
        |- overlord - details on http://druid.io/docs/latest/design/overlord.html
    |- extensions/* - core Druid extensions
    |- hadoop-dependencies/* - Druid Hadoop dependencies
    |- lib/* - libraries and dependencies for core Druid
    |- quickstart/* - files related to the single-machine quickstart
   ```
 
2. In conf/druid/_common/common.runtime.properties,

   2.1 Set-up HDFS for Deep storage
   
     1. Set druid.extensions.loadList=["druid-hdfs-storage"].

     2. Comment out the configurations for local storage under "Deep Storage" and "Indexing service logs".

     3. Uncomment and configure appropriate values in the "For HDFS" sections of "Deep Storage" and "Indexing service logs".
    
         ```
         druid.extensions.loadList=["druid-hdfs-storage"]

         #druid.storage.type=local
         #druid.storage.storageDirectory=var/druid/segments

         druid.storage.type=hdfs
         druid.storage.storageDirectory=druid.storage.storageDirectory=hdfs://ctrl-secondary:9000/druid/segments

         druid.indexer.logs.type=file
         druid.indexer.logs.directory=var/druid/indexing-logs

         druid.indexer.logs.type=hdfs
         druid.indexer.logs.directory=hdfs://ctrl-secondary:9000/druid/indexing-logs
         ```
       
     4. Place your Hadoop configuration XMLs (core-site.xml, hdfs-site.xml, yarn-site.xml, mapred-site.xml) on the classpath of your Druid nodes. You can do this by copying them into conf/druid/_common/.

   2.2 Set-up zookeeper dependency 
   
     1. Replace "zk.service.host" with the address of the machine that runs your ZK instance:
		
          ```
          druid.zk.service.host=localhost
          ```
        
   2.3 Set-up Mysql for Metadata storage

     1. Configure your Druid metadata storage extension:

          ```
          druid.extensions.loadList=["mysql-metadata-storage"]
          druid.metadata.storage.type=mysql
          druid.metadata.storage.connector.connectURI=jdbc:mysql://localhost/druid
          druid.metadata.storage.connector.user=druid
          druid.metadata.storage.connector.password=druid
          ```

   2.4 Sample

	<details>
	<summary><mark><font color=darkred>Click to view</font></mark></summary>

			#
			# Licensed to the Apache Software Foundation (ASF) under one
			# or more contributor license agreements.  See the NOTICE file
			# distributed with this work for additional information
			# regarding copyright ownership.  The ASF licenses this file
			# to you under the Apache License, Version 2.0 (the
			# "License"); you may not use this file except in compliance
			# with the License.  You may obtain a copy of the License at
			#
			#   http://www.apache.org/licenses/LICENSE-2.0
			#
			# Unless required by applicable law or agreed to in writing,
			# software distributed under the License is distributed on an
			# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
			# KIND, either express or implied.  See the License for the
			# specific language governing permissions and limitations
			# under the License.
			#

			#
			# Extensions
			#

			# This is not the full list of Druid extensions, but common ones that people often use. You may need to change this list
			# based on your particular setup.
			druid.extensions.loadList=["druid-histogram", "druid-datasketches", "druid-lookups-cached-global"]
			druid.extensions.localRepository=extensions-repo

			# If you have a different version of Hadoop, place your Hadoop client jar files in your hadoop-dependencies directory
			# and uncomment the line below to point to your directory.
			#druid.extensions.hadoopDependenciesDir=/my/dir/hadoop-dependencies

			#
			# Logging
			#

			# Log all runtime properties on startup. Disable to avoid logging properties on startup:
			druid.startup.logging.logProperties=true

			#
			# Zookeeper
			#

			# newly added for cluster -- Li Yong Ke
			# druid.zk.service.host=zk.host.ip
			druid.zk.service.host=localhost
			druid.zk.paths.base=/druid

			#
			# Metadata storage
			#

			# For Derby server on your Druid Coordinator (only viable in a cluster with a single Coordinator, no fail-over):
			# druid.metadata.storage.type=derby
			# druid.metadata.storage.connector.connectURI=jdbc:derby://metadata.store.ip:1527/var/druid/metadata.db;create=true
			# druid.metadata.storage.connector.host=metadata.store.ip
			# druid.metadata.storage.connector.port=1527

			# For MySQL (make sure to include the MySQL JDBC driver on the classpath):
			 druid.extensions.loadList=["mysql-metadata-storage"]
			 druid.metadata.storage.type=mysql
			 druid.metadata.storage.connector.connectURI=jdbc:mysql://localhost:3306/druid
			 druid.metadata.storage.connector.user=druid
			 druid.metadata.storage.connector.password=druid

			# For PostgreSQL (make sure to additionally include the Postgres extension):
			# druid.metadata.storage.type=postgresql
			# druid.metadata.storage.connector.connectURI=jdbc:postgresql://10.79.99.226:5432/postgres
			# druid.metadata.storage.connector.user=postgres
			# druid.metadata.storage.connector.password=postgres

			#
			# Deep storage
			# changed by Li Yong Ke
			# date: 2018/12/28
			#

			# For local disk (only viable in a cluster if this is a network mount):
			# druid.storage.type=local
			# druid.storage.storageDirectory=var/druid/segments

			# For HDFS (make sure to include the HDFS extension and that your Hadoop config files in the cp):
			# druid.extensions.loadList=["druid-hdfs-storage"]
			 druid.storage.type=hdfs
			 druid.storage.storageDirectory=hdfs://ctrl-secondary:9000/druid/segments

			# For S3:
			# druid.storage.type=s3
			# druid.storage.bucket=your-bucket
			# druid.storage.baseKey=druid/segments
			# druid.s3.accessKey=...
			# druid.s3.secretKey=...

			#
			# Indexing service logs
			# changed by Li Yong Ke
			# date: 2018/12/28
			#

			# For local disk (only viable in a cluster if this is a network mount):
			# druid.indexer.logs.type=file
			# druid.indexer.logs.directory=var/druid/indexing-logs

			# For HDFS (make sure to include the HDFS extension and that your Hadoop config files in the cp):
			 druid.indexer.logs.type=hdfs
			 druid.indexer.logs.directory=hdfs://ctrl-secondary:9000/druid/indexing-logs
			# druid.indexer.logs.directory=var/druid/indexing-logs

			# For S3:
			# druid.indexer.logs.type=s3
			# druid.indexer.logs.s3Bucket=your-bucket
			# druid.indexer.logs.s3Prefix=druid/indexing-logs

			#
			# Service discovery
			#

			druid.selectors.indexing.serviceName=druid/overlord
			druid.selectors.coordinator.serviceName=druid/coordinator

			#
			# Monitoring
			#

			druid.monitoring.monitors=["org.apache.druid.java.util.metrics.JvmMonitor"]
			druid.emitter=logging
			druid.emitter.logging.logLevel=info

			# Storage type of double columns
			# ommiting this will lead to index double as float at the storage layer

			druid.indexing.doubleStorage=double

</details>

3. Start Druid Cluster

   * On master
  
      > bin/
      >>    * coordinator.sh start
      >>    * overlord.sh start
      >>    * historical.sh start
      >>    * middleManager.sh start
      >>    * broker.sh start

   * On slaves

      > bin/
      >>    * historical.sh start
      >>    * middleManager.sh start
      >>    * broker.sh start

   * Or beyond above scripts, start cluster directly by java:

       ```
       nohup java `cat conf/druid/coordinator/jvm.config | xargs` -cp conf/druid/_common:conf/druid/coordinator:lib/* org.apache.druid.cli.Main server coordinator >> /home/liyongke/druid_logs/nohup_coordinator.log 2>&1 &

       nohup java `cat conf/druid/overlord/jvm.config | xargs` -cp conf/druid/_common:conf/druid/overlord:lib/*:extensions/druid-hdfs-storage/* org.apache.druid.cli.Main server overlord >> /home/liyongke/druid_logs/nohup_overlord.log 2>&1 &

       nohup java `cat conf/druid/historical/jvm.config | xargs` -cp conf/druid/_common:conf/druid/historical:lib/*:extensions/druid-hdfs-storage/*  org.apache.druid.cli.Main server historical >> /home/liyongke/druid_logs/nohup_historical.log 2>&1 &

       nohup java `cat conf/druid/middleManager/jvm.config | xargs` -cp conf/druid/_common:conf/druid/middleManager:lib/*:extensions/druid-hdfs-storage/*  org.apache.druid.cli.Main server middleManager >> /home/liyongke/druid_logs/nohup_middleManager.log 2>&1 &

       nohup java `cat conf/druid/broker/jvm.config | xargs` -cp conf/druid/_common:conf/druid/broker:lib/*:extensions/druid-hdfs-storage/*  org.apache.druid.cli.Main server broker >> /home/liyongke/druid_logs/nohup_broker.log 2>&1 &

       ```

4. Druid Console

   > Check on the cluster status from the following console websites.
   
      * Druid Cluster Console : http://\<hostname\>:8081/#/
      
         [Druid Cluster Console sample](http://ctrl-secondary:8081/#/)
    
      * Druid Overload Console : http://\<hostname\>:8090/console.html
   
         [Druid Overload Console sample](http://ctrl-secondary:8090/console.html)
	 	
5. Shutdown Druid Cluster

   * On master
  
      > bin/
      >>    * coordinator.sh stop
      >>    * overlord.sh stop
      >>    * historical.sh stop
      >>    * middleManager.sh stop
      >>    * broker.sh stop

   * On slaves

      > bin/
      >>    * historical.sh stop
      >>    * middleManager.sh stop
      >>    * broker.sh stop

## 5. Varify Druid + HDFS Cluster Enviroment

   * Based on the druid official guides for data ingestion and dataquery
       
      1. [Druid data ingestion](http://druid.io/docs/latest/ingestion/index.html)
      
      2. [Druid data query](http://druid.io/docs/latest/querying/querying.html)
      
        * Monitoring and managing ingestion or query process status and logs through :
          
	     > Druid Overload Console : http://\<hostname\>:8090/console.html *
      
   * Checkout data and files from web console 
      
      3. Checkout data on Druid Cluster Console : http://\<hostname\>:8081/#/ . 
     
      4. Checkout segment files on HDFS Explorer Console : http://\<hostname\>:50070/dfshealth.html#tab-overview .
      

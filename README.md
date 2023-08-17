# Docker Hadoop
This repo, including readme was originally written by [The Apache Software Foundation](https://hub.docker.com/u/apache) on their [Docker Hub](https://hub.docker.com/r/apache/hadoop) and [Github](https://github.com/apache/hadoop/tree/docker-hadoop-3). I am only making a copy of the orginal article and revising some instructions so that any of visitors not be distressed.

## Apache Hadoop
The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

## Quickstart
### Build the Images
```
docker-compose build
```
### Run the Containers
```
docker-compose up -d
```
### Access to the Namenode
User **hadoop** has been granted to execute hadoop and hdfs commands.
```
docker exec --user hadoop -it docker-hadoop_namenode_1 /bin/bash
```
or just run the command without --user flag, **hadoop is default.**
```
docker exec -it docker-hadoop_namenode_1 /bin/bash
```

## In-Depth Guide

### Example building the latest hadoop-3 image
**Create a basic docker-compose.yaml file like:**
```
version: "2"
services:
  namenode:
    image: apache/hadoop:3
    hostname: namenode
    command: ["hdfs", "namenode"]
    ports:
      - 9870:9870
    env_file:
      - ./config
    environment:
      ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
  datanode:
    image: apache/hadoop:3
    command: ["hdfs", "datanode"]
    env_file:
      - ./config      
  resourcemanager:
    image: apache/hadoop:3
    hostname: resourcemanager
    command: ["yarn", "resourcemanager"]
    ports:
      - 8088:8088
    env_file:
      - ./config
    volumes:
      - ./test.sh:/opt/test.sh
  nodemanager:
    image: apache/hadoop:3
    command: ["yarn", "nodemanager"]
    env_file:
      - ./config
```
Change the ```image: apache/hadoop:3``` incase you want to build any other image like ```image: apache/hadoop:3.3.5``` for building Apache Hadoop 3.3.5 image

**Create a Config File Like:**
```
CORE-SITE.XML_fs.default.name=hdfs://namenode
CORE-SITE.XML_fs.defaultFS=hdfs://namenode
HDFS-SITE.XML_dfs.namenode.rpc-address=namenode:8020
HDFS-SITE.XML_dfs.replication=1
MAPRED-SITE.XML_mapreduce.framework.name=yarn
MAPRED-SITE.XML_yarn.app.mapreduce.am.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.map.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.reduce.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
YARN-SITE.XML_yarn.resourcemanager.hostname=resourcemanager
YARN-SITE.XML_yarn.nodemanager.pmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.delete.debug-delay-sec=600
YARN-SITE.XML_yarn.nodemanager.vmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.aux-services=mapreduce_shuffle
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-applications=10000
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-am-resource-percent=0.1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.queues=default
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.user-limit-factor=1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.maximum-capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.state=RUNNING
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_submit_applications=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_administer_queue=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.node-locality-delay=40
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings=
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings-override.enable=false
```
** You can add/replace any new config in the similar format in this file.

### Check the Current Directory (optional)
Do a ls -l on the current directory it should have the two files we created above
```
docker-3 % ls -l
-rw-r--r--  1 hadoop  apache  2547 Jun 23 15:53 config
-rw-r--r--  1 hadoop  apache  1533 Jun 23 16:07 docker-compose.yaml
```

### Run the Docker Containers
Run the docker containers using docker-compose
```
docker-compose up -d
```
The output should look like:
```
docker-hadoop % docker-compose up -d    
Creating network "docker-hadoop_default" with the default driver
Creating docker-hadoop_namenode_1        ... done
Creating docker-hadoop_datanode_1        ... done
Creating docker-hadoop_nodemanager_1     ... done
Creating docker-hadoop_resourcemanager_1 ... done
```

### Accessing the Cluster:
**Login into a node:**
Can login into any node by specifying the container like:
```
docker exec -it docker-hadoop_namenode_1 /bin/bash 
```
Run ```docker ps``` to verify that the container name looks like the example above.

**Running an example Job (Pi Job)**
```
yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 10 15
```
The above will run a Pi Job and similarly any hadoop related command can be run.

### Accessing the UI
The Namenode UI can be accessed at http://localhost:9870/ and the ResourceManager UI can be accessed at http://localhost:8088/

### Shutdown Cluster
The cluster can be shut down via:
```
docker-compose down
```

## How Set-up Hadoop-Book Examples
### Access to the namenode as root user
```
docker exec --user root -it docker-hadoop_namenode_1 /bin/bash
```
### Install git and clone the example repo.
```
yum install git
```
```
git clone https://github.com/tomwhite/hadoop-book.git
```
### Install maven and build
```
yum install maven
```
```
cd hadoop-book \
mvn package -DskipTests
```
### Copy all the example data from local file system to hdfs
```
hdfs dfs -put input hdfs://namenode/user/hadoop/input
```

## Appendix

### Note:
The above example is for Hadoop-3.x line, In case you want to build the Hadoop-2.x, Similar steps but different config & docker-compose.yaml file. Logic can be extracted from: https://github.com/apache/hadoop/tree/docker-hadoop-2

### Docker Source Code:
The docker images are built via special branches & the source code for branch 3 lies at https://github.com/apache/hadoop/tree/docker-hadoop-3 and for branch 2 at https://github.com/apache/hadoop/tree/docker-hadoop-2

### Reaching out us:
Hadoop Developers can be reached via the hadoop mailing lists: https://hadoop.apache.org/mailing_lists.html

### Further Reading
https://hadoop.apache.org/
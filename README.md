# Hadoop-2.10.2-Multi-Node-Cluster
3-node Apache Hadoop 2.10.2 multi-node cluster setup with HDFS, YARN, and MapReduce.

This repository contains the complete setup and configuration for a **3-node Apache Hadoop 2.10.2 multi-node cluster**.

The cluster consists of **one NameNode and two DataNodes**, with **HDFS** used for distributed storage, **YARN** for resource management, and **MapReduce** for distributed data processing.

What's Included

* Java 8 installation
* Hadoop 2.10.2 installation
* Environment variable configuration
* Hadoop configuration files
* Passwordless SSH setup
* Multi-node HDFS configuration
* YARN configuration
* Hadoop cluster startup and verification
* HDFS file upload
* MapReduce WordCount example

### Configuration Files

| File              | Purpose                            |
| ----------------- | ---------------------------------- |
| `hadoop-env.sh`   | Java environment configuration     |
| `core-site.xml`   | Core Hadoop and HDFS configuration |
| `hdfs-site.xml`   | HDFS configuration and replication |
| `mapred-site.xml` | MapReduce configuration            |
| `yarn-site.xml`   | YARN configuration                 |
| `slaves`          | DataNode/worker node configuration |


**Hadoop Version:** 2.10.2
**Java Version:** 8
**Cluster:** 3 Nodes
**Storage:** HDFS
**Resource Management:** YARN
**Processing:** MapReduce

# Hadoop 2.10.2 Multi-Node Cluster Setup

This document provides the step-by-step procedure for setting up a **3-node Apache Hadoop 2.10.2 multi-node cluster** using HDFS, YARN, and MapReduce.

## Cluster Overview

| Node       | Hostname | Private IP         | Role                                         |
| ---------- | -------- | ------------------ | -------------------------------------------- |
| NameNode   | `nn`     | `<PRIVATE_IP_NN>`  | NameNode, SecondaryNameNode, ResourceManager |
| DataNode 1 | `dn1`    | `<PRIVATE_IP_DN1>` | DataNode, NodeManager                        |
| DataNode 2 | `dn2`    | `<PRIVATE_IP_DN2>` | DataNode, NodeManager                        |

> **Important:** Use the **private IP address** assigned to each node in your environment. Do not use public IP addresses for Hadoop's internal cluster communication.

---

# Prerequisites

Before starting Hadoop installation, ensure that all three nodes are running and can communicate with each other over their **private network**.

The required hostname structure is:

```text
nn
dn1
dn2
```

The hostname-to-private-IP mapping should be:

```text
<PRIVATE_IP_NN>  nn
<PRIVATE_IP_DN1> dn1
<PRIVATE_IP_DN2> dn2
```

The same mapping should be configured on **all three nodes**.

---

## 1. Configure `/etc/hosts`

Edit the hosts file:

```bash
sudo vi /etc/hosts
```

Add:

```text
<PRIVATE_IP_NN>  nn
<PRIVATE_IP_DN1> dn1
<PRIVATE_IP_DN2> dn2
```

### What is happening?

The `/etc/hosts` file provides local hostname resolution.

Instead of using private IP addresses throughout the Hadoop configuration, we can use:

```text
nn
dn1
dn2
```

For example, the Hadoop configuration uses:

```text
hdfs://nn:9000
```

The operating system resolves `nn` to the private IP address assigned to the NameNode.

### Important

The hostname must match the intended role:

```text
nn  → NameNode
dn1 → DataNode 1
dn2 → DataNode 2
```

The `/etc/hosts` configuration should be consistent across **all nodes**.

---

## 2. Verify Hostnames

Check the hostname on each machine:

```bash
hostname
```

Expected:

### NameNode

```text
nn
```

### DataNode 1

```text
dn1
```

### DataNode 2

```text
dn2
```

### Why?

Hadoop uses these hostnames to identify and communicate with the different nodes in the cluster.

---

## 3. Verify Hostname Resolution

From each node, verify that the three hostnames resolve correctly.

```bash
getent hosts nn
```

```bash
getent hosts dn1
```

```bash
getent hosts dn2
```

Each hostname should resolve to the corresponding **private IP address**.

You can also test connectivity:

```bash
ping -c 2 nn
```

```bash
ping -c 2 dn1
```

```bash
ping -c 2 dn2
```

### What are we verifying?

We are making sure that every node can correctly resolve:

```text
nn
dn1
dn2
```

before Hadoop services are installed.

---

## 4. Passwordless SSH

Ensure that the NameNode can SSH into both DataNodes without requiring a password.

Required connectivity:

```text
NameNode
   │
   ├── SSH → dn1
   │
   └── SSH → dn2
```

From `nn`, test:

```bash
ssh dn1
```

and:

```bash
ssh dn2
```

### Why is this required?

Hadoop's cluster startup scripts use SSH to communicate with worker nodes.

For example:

```bash
start-dfs.sh
```

and:

```bash
start-yarn.sh
```

use the configured worker nodes to start the appropriate Hadoop services.

---

# Step 1: Install Java 8

**Run on all nodes.**

Hadoop 2.10.2 requires Java because Hadoop services run on the Java Virtual Machine.

Update the package index:

```bash
sudo apt update
```

### What is happening?

Ubuntu refreshes its local package information from the configured repositories.

Install Java 8:

```bash
sudo apt install -y openjdk-8-jdk
```

### What is happening?

Java 8 JDK is installed on the node.

The final environment should have Java 8 available on:

```text
nn
dn1
dn2
```

---

# Step 2: Download & Install Hadoop

**Run on the NameNode.**

Download Hadoop 2.10.2:

```bash
wget [https://archive.apache.org/dist/hadoop/common/hadoop-2.10.2/hadoop-2.10.2.tar.gz](https://archive.apache.org/dist/hadoop/common/hadoop-2.10.2/hadoop-2.10.2.tar.gz)
```

### What is happening?

The Hadoop 2.10.2 compressed archive is downloaded from the Apache archive.

Extract Hadoop into `/opt`:

```bash
sudo tar -xzvf hadoop-2.10.2.tar.gz -C /opt
```

### What is happening?

The compressed Hadoop archive is extracted under `/opt`.

The extracted directory will be:

```text
/opt/hadoop-2.10.2
```

Rename the directory:

```bash
sudo mv /opt/hadoop-2.10.2 /opt/Hadoop
```

### What is happening?

The Hadoop installation directory is renamed to:

```text
/opt/Hadoop
```

This provides a simple and consistent path for the Hadoop installation.

Change ownership:

```bash
sudo chown -R ubuntu:ubuntu /opt/Hadoop
```

### What is happening?

Ownership of the Hadoop installation is assigned to the `ubuntu` user.

This allows Hadoop commands to be managed by the `ubuntu` user during normal operation.

---

# Step 3: Set Environment Variables

**Run on all nodes.**

Edit the user's `.bashrc`:

```bash
vi ~/.bashrc
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/opt/Hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

### What is happening?

Three important environment variables are configured.

### `JAVA_HOME`

```text
/usr/lib/jvm/java-8-openjdk-amd64
```

Tells Hadoop where Java is installed.

### `HADOOP_HOME`

```text
/opt/Hadoop
```

Defines the Hadoop installation directory.

### `PATH`

Adds Hadoop's executable directories to the shell's command search path.

This allows commands such as:

```text
hdfs
hadoop
start-dfs.sh
start-yarn.sh
```

to be executed directly.

Load the updated environment:

```bash
source ~/.bashrc
```

### What is happening?

The updated `.bashrc` is loaded into the current shell session, making the newly configured variables available immediately.

---

# Step 4: Configure Hadoop Files

**Run on the NameNode.**

Move to the Hadoop configuration directory:

```bash
cd /opt/Hadoop/etc/hadoop/
```

This directory contains the configuration files that define how Hadoop operates.

---

## 4.1 `hadoop-env.sh`

Edit:

```bash
sudo vi hadoop-env.sh
```

Set:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

### What is happening?

This explicitly tells Hadoop which Java installation should be used by its internal scripts and services.

---

## 4.2 `core-site.xml`

Edit:

```bash
sudo vi core-site.xml
```

Configuration:

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://nn:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/ubuntu/hdata</value>
  </property>
</configuration>
```

### What is happening?

`core-site.xml` contains core Hadoop configuration.

### `fs.defaultFS`

```text
hdfs://nn:9000
```

Defines HDFS as the default filesystem and identifies `nn` as the NameNode.

The flow is:

```text
Hadoop Command
      ↓
Default Filesystem
      ↓
HDFS
      ↓
NameNode (nn:9000)
```

### `hadoop.tmp.dir`

```text
/home/ubuntu/hdata
```

Defines Hadoop's base temporary directory.

---

## 4.3 `hdfs-site.xml`

Edit:

```bash
sudo vi hdfs-site.xml
```

Configuration:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
</configuration>
```

### What is happening?

The HDFS replication factor is set to `2`.

This means HDFS attempts to maintain two copies of each data block.

With two DataNodes, the basic concept is:

```text
             HDFS Block
                 │
          ┌──────┴──────┐
          ▼             ▼
        dn1            dn2
```

This provides basic redundancy for stored HDFS data.

---

## 4.4 `mapred-site.xml`

Create the MapReduce configuration file:

```bash
cp mapred-site.xml.template mapred-site.xml
```

Edit:

```bash
sudo vi mapred-site.xml
```

Configuration:

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

### What is happening?

This tells MapReduce to use **YARN** as its execution framework.

The relationship is:

```text
MapReduce Job
      ↓
     YARN
      ↓
Cluster Resources
```

---

## 4.5 `yarn-site.xml`

Edit:

```bash
sudo vi yarn-site.xml
```

Configuration:

```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

### What is happening?

This enables the MapReduce shuffle service on the NodeManager.

The shuffle phase transfers intermediate mapper output toward reducers.

Simplified flow:

```text
Mapper
   ↓
Intermediate Data
   ↓
Shuffle
   ↓
Reducer
```

---

## 4.6 `slaves`

**Hadoop 2.x specific configuration.**

Edit:

```bash
vi slaves
```

Remove:

```text
localhost
```

and add:

```text
dn1
dn2
```

### What is happening?

The `slaves` file identifies the worker nodes that belong to this Hadoop cluster.

In this setup:

```text
             nn
          NameNode
             │
       ┌─────┴─────┐
       ▼           ▼
      dn1         dn2
```

The Hadoop startup scripts use this worker-node information when starting cluster services.

---

# Step 5: Copy Hadoop to DataNodes

The Hadoop installation and configuration now need to be available on both DataNodes.

## Copy Hadoop to `dn1`

From the NameNode:

```bash
scp -r /opt/Hadoop ubuntu@dn1:/tmp/
```

### What is happening?

`scp` securely copies the Hadoop installation from the NameNode to `dn1`.

The files are temporarily placed under:

```text
/tmp/Hadoop
```

Connect to `dn1`:

```bash
ssh dn1
```

Move Hadoop into `/opt` and set ownership:

```bash
sudo mv /tmp/Hadoop /opt/ && sudo chown -R ubuntu:ubuntu /opt/Hadoop
```

The Hadoop installation is now available at:

```text
/opt/Hadoop
```

on `dn1`.

---

## Copy Hadoop to `dn2`

From the NameNode:

```bash
scp -r /opt/Hadoop ubuntu@dn2:/tmp/
```

Connect to `dn2`:

```bash
ssh dn2
```

Move Hadoop into `/opt` and set ownership:

```bash
sudo mv /tmp/Hadoop /opt/ && sudo chown -R ubuntu:ubuntu /opt/Hadoop
```

At the end of this step:

```text
nn  → /opt/Hadoop
dn1 → /opt/Hadoop
dn2 → /opt/Hadoop
```

All nodes now have the Hadoop installation.

---

# Step 6: Format & Start Cluster

**Run these commands on the NameNode.**

## 6.1 Format the NameNode

```bash
hdfs namenode -format
```

### What is happening?

This initializes the NameNode's filesystem namespace and metadata for the new HDFS cluster.

The NameNode must be initialized before HDFS can be used for the first time.

> **Warning:** `hdfs namenode -format` should normally be performed only when initializing a new HDFS cluster. Running it on an existing cluster can reinitialize the NameNode filesystem metadata.

---

## 6.2 Start HDFS

```bash
start-dfs.sh
```

### What is happening?

This starts the HDFS services across the cluster.

The expected HDFS architecture is:

```text
              NameNode
                  │
          ┌───────┴───────┐
          ▼               ▼
       DataNode         DataNode
         dn1               dn2
```

---

## 6.3 Start YARN

```bash
start-yarn.sh
```

### What is happening?

This starts the YARN resource management services.

The expected architecture is:

```text
          ResourceManager
                 │
        ┌────────┴────────┐
        ▼                 ▼
   NodeManager       NodeManager
       dn1                dn2
```

At this point:

```text
HDFS  → Distributed Storage
YARN  → Resource Management
```

are running across the cluster.

---

# Step 7: Verification

After starting the cluster, verify the Hadoop services on all nodes.

Run:

```bash
jps
```

## On `nn`

Expected Hadoop processes:

```text
NameNode
SecondaryNameNode
ResourceManager
```

## On `dn1`

Expected:

```text
DataNode
NodeManager
```

## On `dn2`

Expected:

```text
DataNode
NodeManager
```

### Expected Cluster Layout

```text
nn
├── NameNode
├── SecondaryNameNode
└── ResourceManager

dn1
├── DataNode
└── NodeManager

dn2
├── DataNode
└── NodeManager
```

The purpose of this verification is to confirm that the required Hadoop services are running on their intended nodes.

---

# Web UI Verification

## HDFS NameNode UI

Open:

```text
http://nn:50070
```

The NameNode web interface provides information about the HDFS cluster and DataNodes.

## YARN ResourceManager UI

Open:

```text
http://nn:8088
```

The ResourceManager web interface provides information about YARN, cluster resources, nodes, and applications.

---

# Step 8: Test HDFS

Create an input directory in HDFS:

```bash
hdfs dfs -mkdir -p /user/ubuntu/input
```

### What is happening?

This creates the directory:

```text
/user/ubuntu/input
```

inside **HDFS**.

---

Create a local test file:

```bash
echo "Hadoop is awesome Hadoop is powerful" > test.txt
```

### What is happening?

This creates `test.txt` in the **local Linux filesystem**.

At this point the file is not yet stored in HDFS.

---

Upload the file to HDFS:

```bash
hdfs dfs -put test.txt /user/ubuntu/input/
```

### What is happening?

The local file is uploaded into HDFS.

The data flow is:

```text
Local Filesystem
       │
       │ hdfs dfs -put
       ▼
      HDFS
       │
       ▼
   DataNodes
```

---

# Step 9: Run MapReduce WordCount

Run the Hadoop MapReduce example:

```bash
hadoop jar /opt/Hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.2.jar wordcount /user/ubuntu/input /user/ubuntu/output
```

### What is happening?

This submits the WordCount MapReduce application.

The input directory is:

```text
/user/ubuntu/input
```

The output directory is:

```text
/user/ubuntu/output
```

The MapReduce processing flow is:

```text
Input Data
    ↓
   HDFS
    ↓
   Map
    ↓
 Shuffle
    ↓
  Reduce
    ↓
HDFS Output
```

For the input:

```text
Hadoop is awesome Hadoop is powerful
```

the logical word counts are:

```text
Hadoop     2
is         2
awesome    1
powerful   1
```

---

# Final Validation

A successful WordCount job provides an end-to-end test of the Hadoop cluster.

The complete flow is:

```text
              Hadoop Cluster
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
         HDFS                YARN
      Storage Layer      Resource Management
          │                   │
          └─────────┬─────────┘
                    ▼
                MapReduce
                Processing
                    │
                    ▼
                HDFS Output
```

This confirms that the major Hadoop components are working together:

```text
HDFS
 ↓
YARN
 ↓
MapReduce
 ↓
HDFS Output
```

---

# Setup Complete

The Hadoop 2.10.2 multi-node cluster is now configured with:

```text
NameNode        → nn
DataNode 1      → dn1
DataNode 2      → dn2

Storage         → HDFS
Resource Mgmt   → YARN
Processing      → MapReduce
```

The successful execution of the WordCount job confirms that the cluster is able to store data in HDFS and execute a MapReduce workload using YARN.

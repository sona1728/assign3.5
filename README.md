1.)
#HDP 2.0 NAMENODE HIGH AVAILABILITY: FEATURE OVERVIEW

- AUTOMATED FAILOVER: HDP pro-actively detects NameNode host and process failures and will automatically switch to the standby NameNode to maintain availability for the HDFS service. There is no need for human intervention in the process – System Administrators can sleep in peace!

- STANDBY: Both Active and Standby NameNodes have up to date HDFS metadata, ensuring seamless failover even for large clusters – which means no downtime for your HDP cluster!

- FULL STACK RESILIENCY: The entire HDP stack (MapReduce, Hive, Pig, HBase, Oozie) has been certified to handle a NameNode failure scenario without losing data or the job progress. This is vital to ensure long running jobs that are critical to complete on schedule will not be adversely affected during a NameNode failure scenario.

#With HDP 2.0, this entire functionality is built into the HDP product. This means:

- No need for shared storage
- No need for external HA frameworks
- Certified with a secure Kerberos configuration

- In each cluster, two separate machines are configured as NameNodes. In a working cluster, one of the NameNode machine is in the Active state, and the other is in the Standby state.

- The Active NameNode is responsible for all client operations in the cluster. The Standby NameNode maintains enough state to provide a fast failover. In order for the Standby node to keep its state synchronized with the Active node, both nodes communicate through a group of separate daemons called JournalNodes. The file system journal logged by the Active NameNode at the JournalNodes is consumed by the Standby NameNode to keep it’s file system namespace in sync with the Active.

- In order to provide a fast failover, it is also necessary that the Standby node have up-to-date information of the location of blocks in your cluster. DataNodes are configured with the location of both the NameNodes and send block location information and heartbeats to both NameNode machines.

- The ZooKeeper Failover Controller (ZKFC) is responsible for HA Monitoring of the NameNode service and for automatic failover when the Active NameNode is unavailable. There are two ZKFC processes – one on each NameNode machine. ZKFC uses the Zookeeper Service for coordination in determining which is the Active NameNode and in determining when to failover to the Standby NameNode.

- Quorum journal manager (QJM) in the NameNode writes file system journal logs to the journal nodes. A journal log is considered successfully written only when it is written to majority of the journal nodes. Only one of the Namenodes can achieve this quorum write. In the event of split-brain scenario this ensure that the file system metadata will not be corrupted by two active NameNodes.

- In HA setup, HDFS clients are configured with a logical name service URI and the two NameNodes corresponding to it. The clients perform source side failover. When a client cannot connect to a NameNode or if the NameNode is in standby mode, it performs fail over to the other NameNode.


2.)
#CHECK POINTING:

- HDFS metadata can be thought of consisting of two parts: the base filesystem table (stored in a file called fsimage) and the edit log which lists changes made to the base table (stored in a file called edits). Checkpointing is a process of reconciling fsimage with edits to produce a new version of fsimage. There are two benefits arising out of this: a more recent version of fsimage, and a truncated edit log.

*USAGE OF CHECK POINTING:

- fs.checkpoint.period controls how often this reconciliation will be triggered.  3600 means that every hour fsimage will be updated and edit log truncated. Checkpiont is not cheap, so there is a balance between running it too often and letting the edit log grow too large. This parameter should be set to get a good balance assuming typical filesystem use in your cluster.

- fs.checkpoint.size is a size threshold, which, if reached by edits, will trigger an immediate checkpoint regardless of time elapsed since the last checkpoint. This is insurance from edit log getting too large under unusually heavy write traffic to the filesystem metadata.


3.)
#HDFS Federation

- HDFS Federation improves the existing HDFS architecture through a clear separation of namespace and storage, enabling generic block storage layer. It enables support for multiple namespaces in the cluster to improve scalability and isolation. Federation also opens up the architecture, expanding the applicability of HDFS cluster to new implementations and use cases.

- In order to scale the name service horizontally, federation uses multiple independent namenodes/namespaces. The namenodes are federated, that is, the namenodes are independent and don’t require coordination with each other. The datanodes are used as common storage for blocks by all the namenodes. Each datanode registers with all the namenodes in the cluster. Datanodes send periodic heartbeats and block reports and handles commands from the namenodes.

- A Block Pool is a set of blocks that belong to a single namespace. Datanodes store blocks for all the block pools in the cluster.

- It is managed independently of other block pools. This allows a namespace to generate Block IDs for new blocks without the need for coordination with the other namespaces. The failure of a namenode does not prevent the datanode from serving other namenodes in the cluster.

- A Namespace and its block pool together are called Namespace Volume. It is a self-contained unit of management. When a namenode/namespace is deleted, the corresponding block pool at the datanodes is deleted. Each namespace volume is upgraded as a unit, during cluster upgrade.

*BENEFITS:
1. Generic storage service.
2. Design simplicity.


4.)
#The four files that need to be configured explicitly while setting up a single node hadoop cluster are:

- Core-site.xml
- HDFS-site.xml
- YARN-site.xml
- Mapred-site.xml
#Core-site.xml

- The core-site.xml file contains information such as the port number used for Hadoop instance, memory allocated for the file system, memory limit for storing the data, and size of Read/Write buffers.

- Open the core-site.xml and add the following properties in between <configuration>, </configuration> tags.

#HDFS-site.xml

- The hdfs-site.xml file contains information such as the value of replication data, namenode path, and datanode paths of your local file systems. It means the place where you want to store the Hadoop infrastructure.

- Open this file and add the following properties in between the <configuration> </configuration> tags in this file.

#YARN-site.xml

- This file is used to configure yarn into Hadoop. Open the yarn-site.xml file and add the following properties in between the <configuration>, </configuration> tags in this file.

#Mapred-site.xml

- This file is used to specify which MapReduce framework we are using. By default, Hadoop contains a template of yarn-site.xml. First of all, it is required to copy the file from mapred-site,xml.template to mapred-site.xml file using the following command.

- Open mapred-site.xml file and add the following properties in between the <configuration>, </configuration>tags in this file.

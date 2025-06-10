           **MQ Native HA For Container environments for MQ Containers on AWS or CP4I on OpenShift**

**Introduction**: This article demonstrates how to architect Native HA using IBM MQ Container, or IBM MQ Operators using CP4I. 
A Native HA configuration consists of three Kubernetes pods, each with an instance of the queue manager. One instance is the active queue manager, processing messages and writing to its recovery log. Whenever the recovery log is written, the active queue manager sends the data to the other two instances, known as replicas. Each replica writes to its own recovery log, acknowledges the data, and then updates its own queue data from the replicated recovery log. If the pod running the active queue manager fails, one of the replica instances of the queue manager takes over the active role and has current data to operate with.
·                     The log type is known as a 'replicated log'. A replicated log is essentially a linear log, with automatic log management and automatic media images enabled. See Types of logging. You use the same techniques for managing the replicated log that you use for managing a linear log.

Architecture:
To create IBM MQ Native HA architecture deployment, we need three worker nodes, In one worker node, the active instance of Queue Manager will be running and in other two worker nodes the replica instance of the Queue Manager will be on standby. As soon as the worker node 1 goes down, the one of the replica instance will start to process the messages in the queue manager’s queue. The below architecture depicts the similar deployment of Native HA queue manager in three availability zones, one worker node in each availability zone would be part of Native HA queue manager creation.
 ![image](https://github.com/user-attachments/assets/314db7ca-3b59-4b06-9bdf-c4d21186c2f0)


The most common architecture here, is to have a single Kubernetes cluster which contains one or more Nodes in each Availability Zone (AZ).  In the MQ Operator, and all IBM samples MQ containers, IBM MQ queue manager is deployed using a standard Kubernetes StatefulSet.  One of the features of a StatefulSet, is that it will schedule Pods spanning AZs.  So if you have three AZs and three Native HA Pods for a Queue Manager, you’ll automatically get one in each AZ.  MQ Native HA always uses block storage, and MQ queue manager will do the replication between the AZs.  So you can rely on the replication built-in to IBM MQ.
From MQ 9.4.2, you can also use Native HA Cross-Region Replication to get MQ to asynchronously replicate to another region.
Each queue manager instance in AZ has its own persistence volume claim. In one AZ the queue manager will be active and in other two AZ the replica queue manager will be in standby. The Native HA QM will replicate the active QM manager data and recovery logs into standby queue manager persistence volume claim. 
On Amazon EKS, you need EBS – Elastic block storage persistence volume claim created for each AZ instance and the Active queue manager will take care of data replication. 
As soon as the active queue manager goes down, one of the replica Queue Manager will be up to process the message from the queue. 
Native HA - IBM Documentation

What mechanism is used by IBM MQ to find a quorum?

To maintain the quorum minimum two IBM MQ qmgr instance  should be available. 

What mechanism is used by IBM MQ to replicate the recovery data between the active pod and the standby pods?

Native HA implements a consensus algorithm based closely on raft (elections are based on log content, strong leader model, etc). Native HA uses its own proprietary wire protocol to implement the election process and also replicate between the active and replica pods. When using Native HA (including CRR) you should use simple RWO block storage that is not replicated – Native HA is responsible for the replication, not the storage layer.

The data replication is managed by IBM MQ inside the Kubernetes Cluster. 

You need single cluster where the worker nodes are spread across the three availability zones and while creating the Native HA QM, you need to specify the three nodes available in respective zones. Make sure independent persistence volume claim is created for QM instance running in each node. One instance is active and other two are replica. Rest Native QM will take care of replication. 
IBM MQ Helm Chart Sample | mq-helm

Alternatively,  (not considered and not agreed in the proposed design)
if you want to use two different clusters with Native HA high availability, you can implement MQ Native HA CRR (Cross region replication) 
Using helm to deploy IBM MQ Native HA Cross Region Replication

## Partition Reassignment

  The partition reassignment tool can be used to move some topics off of the current set of brokers to the newly added brokers. 
This is typically useful while expanding an existing cluster since it is easier to move entire topics to the new set of 
brokers, than moving one partition at a time. When used to do this, the user should provide a list of topics that should be 
moved to the new set of brokers and a target list of new brokers. The tool then evenly distributes all partitions for the 
given list of topics across the new set of brokers. During this move, the replication factor of the topic is kept constant. 
Effectively the replicas for all partitions for the input list of topics are moved from the old set of brokers to the newly 
added brokers.


* Step 1:

```
[root@c2199-node1 bin]# sh kafka-topics.sh --describe --zookeeper c2199-node3:2181 --topic Akshay
Topic:Akshay PartitionCount:2 ReplicationFactor:2 Configs:
Topic: Akshay Partition: 0 Leader: 1003 Replicas: 1003,1004 Isr: 1003,1004
Topic: Akshay Partition: 1 Leader: 1004 Replicas: 1004,1003 Isr: 1004,1003


[root@c2199-node1 bin]# vim test.json
{
"topics": [{"topic":"Akshay"}],

"version":1
}

```
* Step 2:

```
** Added new broker with 1004 Id.

[root@c2199-node1 bin]# sh kafka-reassign-partitions.sh --zookeeper c2199-node2:2181 --topics-to-move-json-file /test.json --broker-list "1005,1003,1004" --generate
Current partition replica assignment
{"version":1,"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004],"log_dirs":["any","any"]},{"topic":"Akshay","partition":1,"replicas":[1004,1003],"log_dirs":["any","any"]}]}
Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004],"log_dirs":["any","any"]},{"topic":"Akshay","partition":1,"replicas":[1004,1005],"log_dirs":["any","any"]}]}
[root@c2199-node1 bin]#
```

* Step 3:

```
** Copy `Proposed partition reassignment configuration` details from previous command and paste in .json file

[root@c2199-node1 bin]# vim test1.json
{"version":1,"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004],"log_dirs":["any","any"]},{"topic":"Akshay","partition":1,"replicas":[1004,1005],"log_dirs":["any","any"]}]}
```

* Step 4:

```
[root@c2199-node1 bin]# sh kafka-reassign-partitions.sh --zookeeper c2199-node2:2181 --reassignment-json-file test1.json --execute
Current partition replica assignment
{"version":1,"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004],"log_dirs":["any","any"]},{"topic":"Akshay","partition":1,"replicas":[1004,1003],"log_dirs":["any","any"]}]}
Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.
```

* Step 5:
```
[root@c2199-node1 bin]# sh kafka-topics.sh --describe --zookeeper c2199-node2:2181 --topic Akshay
Topic:Akshay PartitionCount:2 ReplicationFactor:2 Configs:
Topic: Akshay Partition: 0 Leader: 1003 Replicas: 1003,1004 Isr: 1003,1004
Topic: Akshay Partition: 1 Leader: 1004 Replicas: 1004,1005 Isr: 1004,1005
[root@c2199-node1 bin]#
```

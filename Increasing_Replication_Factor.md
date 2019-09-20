## Increasing replication factor in Kafka.

  Increasing the replication factor of an existing partition is easy. Just specify the extra replicas in the custom reassignment json file and use it with the --execute option to increase the replication factor of the specified partitions.
For instance, the following example increases the replication factor of partition 0 of topic foo from 1 to 3. Before increasing the replication factor, the partition's only replica existed on broker 5. As part of increasing the replication factor, we will add more replicas on brokers 6 and 7.

* Step 1: 

```The first step is to hand craft the custom reassignment plan in a json file.

[root@c2199-node2 bin]# sh kafka-topics.sh --describe --zookeeper c2199-node2:2181 --topic Akshay
Topic:Akshay	PartitionCount:2	ReplicationFactor:2	Configs:
	Topic: Akshay	Partition: 0	Leader: 1003	Replicas: 1003,1004	Isr: 1003,1004
	Topic: Akshay	Partition: 1	Leader: 1005	Replicas: 1004,1005	Isr: 1005,1004
[root@c2199-node2 bin]#
[root@c2199-node2 bin]# cat increase-replication-factor.json 
{"version":1,
"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004,1005]},
	      {"topic":"Akshay","partition":1,"replicas":[1004,1003,1005]}]
```

* Step 2: 

```
[root@c2199-node2 bin]# sh kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute
Current partition replica assignment

{"version":1,"partitions":[{"topic":"Akshay","partition":0,"replicas":[1003,1004],"log_dirs":["any","any"]},{"topic":"Akshay","partition":1,"replicas":[1004,1005],"log_dirs":["any","any"]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.
```

* Step 3: 
```
Verifying the replication factor :

[root@c2199-node2 bin]# sh kafka-topics.sh --describe --zookeeper c2199-node2:2181 --topic Akshay
Topic:Akshay	PartitionCount:2	ReplicationFactor:3	Configs:
	Topic: Akshay	Partition: 0	Leader: 1003	Replicas: 1003,1004,1005	Isr: 1003,1004,1005
	Topic: Akshay	Partition: 1	Leader: 1004	Replicas: 1004,1003,1005	Isr: 1005,1004,1003
[root@c2199-node2 bin]#
```


## Perfoormace issue while partition reassignment

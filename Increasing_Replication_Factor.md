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
	      {"topic":"Akshay","partition":1,"replicas":[1004,1003,1005]}]}
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


#### Best Practise for Partition Reassignment:

1. You would want to make sure that num.replica.fetchers and num.network.threads is optimized, and num.io.threads if there are a lot of log.dirs.

2. The topics can remain active while the replication is happening, the flow of data doesn't need to be stopped. The new replica will be brought into the ISR, and then the old ones removed, but the leader will still be handling client connections.

3. Performance of other topics could be affected if there is a network bottleneck, or all the network threads/replica fetchers  are busy.

You would need to set the optimal values in Kafka to perform well while doing partition Reassignment and below are the configuration which needs to incrase :-

* num.replica.fetchers (default=1) :- Number of fetcher threads used to replicate messages from a source broker. Increasing this value can increase the degree of I/O parallelism in the follower broker.
* num.network.threads (Default=3) :- Adjust based on the number of producers + number of consumers + replication factor. This is the number of threads for Input Output operations.
* num.io.threads (Default=8) :- should be greater than the number of disks dedicated for Kafka. I strongly recommend to start with same number of disks first.

I would recommand to set the below values in Kafka configuration:

* num.replica.fetchers = 8
* num.network.threads = 12
* num.io.threads = 12 

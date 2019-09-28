### Set Leader Manually For Kafka Topic :

```
* Describing the Topic Akshay :-
===
[root@c2199-node2 bin]# sh kafka-topics.sh --describe --zookeeper localhost:2181 --topic Akshay
Topic:Akshay	PartitionCount:2	ReplicationFactor:3	Configs:
	Topic: Akshay	Partition: 0	Leader: 1003	Replicas: 1003,1004,1005	Isr: 1003,1004,1005
	Topic: Akshay	Partition: 1	Leader: 1004	Replicas: 1004,1003,1005	Isr: 1003,1004,1005
[root@c2199-node2 bin]#

*Zookeeper login :
====

[zk: localhost:2181(CONNECTED) 4] ls /brokers/topics/Akshay
[partitions]

* Partition 0
++++
[zk: localhost:2181(CONNECTED) 11] get /brokers/topics/Akshay/partitions/0/state
{"controller_epoch":21,"leader":1003,"version":1,"leader_epoch":9,"isr":[1003,1004,1005]}
cZxid = 0x5000000ce
ctime = Wed Sep 18 12:20:18 UTC 2019
mZxid = 0x9000000f5
mtime = Sat Sep 28 16:36:37 UTC 2019
pZxid = 0x5000000ce
cversion = 0
dataVersion = 19
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 89
numChildren = 0
[zk: localhost:2181(CONNECTED) 12] 
[zk: localhost:2181(CONNECTED) 4] set /brokers/topics/Akshay/partitions/0/state {"controller_epoch":21,"leader":1005,"version":1,"leader_epoch":9,"isr":[1003,1004,1005]}

* Partition 1
++++
[zk: localhost:2181(CONNECTED) 12] get /brokers/topics/Akshay/partitions/1/state
{"controller_epoch":21,"leader":1004,"version":1,"leader_epoch":17,"isr":[1003,1004,1005]}
cZxid = 0x5000000d1
ctime = Wed Sep 18 12:20:18 UTC 2019
mZxid = 0x900000106
mtime = Sat Sep 28 16:36:37 UTC 2019
pZxid = 0x5000000d1
cversion = 0
dataVersion = 28
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 90
numChildren = 0
[zk: localhost:2181(CONNECTED) 13] 
[zk: localhost:2181(CONNECTED) 7] set /brokers/topics/Akshay/partitions/1/state {"controller_epoch":21,"leader":1003,"version":1,"leader_epoch":17,"isr":[1003,1004,1005]}

* Leader for partitions 0 and 1 have been changed as specified in ZK shell.
===
[root@c2199-node2 bin]# sh kafka-topics.sh --describe --zookeeper localhost:2181 --topic Akshay
Topic:Akshay	PartitionCount:2	ReplicationFactor:3	Configs:
	Topic: Akshay	Partition: 0	Leader: 1005	Replicas: 1003,1004,1005	Isr: 1003,1004,1005
	Topic: Akshay	Partition: 1	Leader: 1003	Replicas: 1004,1003,1005	Isr: 1003,1004,1005
[root@c2199-node2 bin]# 

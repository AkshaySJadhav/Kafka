Source: 

sh kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 4 --topic MirrorMaker

[root@c2199-node2 bin]# sh kafka-topics.sh --describe --zookeeper localhost:2181 --topic MirrorMaker
Topic:MirrorMaker	PartitionCount:4	ReplicationFactor:3	Configs:
	Topic: MirrorMaker	Partition: 0	Leader: 1004	Replicas: 1004,1005,1003	Isr: 1004,1005,1003
	Topic: MirrorMaker	Partition: 1	Leader: 1005	Replicas: 1005,1003,1004	Isr: 1005,1003,1004
	Topic: MirrorMaker	Partition: 2	Leader: 1003	Replicas: 1003,1004,1005	Isr: 1003,1004,1005
	Topic: MirrorMaker	Partition: 3	Leader: 1004	Replicas: 1004,1003,1005	Isr: 1004,1003,1005
[root@c2199-node2 bin]# 

sh kafka-console-producer.sh --broker-list c2199-node2:6667 --topic MirrorMaker

sh kafka-console-consumer.sh --bootstrap-server c2199-node2:6667 --topic MirrorMaker --from-beginning

Destination : 


sh kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 4 --topic MirrorMaker
Created topic "MirrorMaker".
[root@c1199-node2 bin]# 

[root@c1199-node2 bin]# sh kafka-topics.sh --describe --zookeeper localhost:2181 --topic MirrorMaker
Topic:MirrorMaker	PartitionCount:4	ReplicationFactor:3	Configs:
	Topic: MirrorMaker	Partition: 0	Leader: 1001	Replicas: 1001,1003,1004	Isr: 1001,1003,1004
	Topic: MirrorMaker	Partition: 1	Leader: 1002	Replicas: 1002,1004,1001	Isr: 1002,1004,1001
	Topic: MirrorMaker	Partition: 2	Leader: 1003	Replicas: 1003,1001,1002	Isr: 1003,1001,1002
	Topic: MirrorMaker	Partition: 3	Leader: 1004	Replicas: 1004,1002,1003	Isr: 1004,1002,1003
[root@c1199-node2 bin]# 

Kafka Cheat Sheet
-

*Display Topic Information*
```
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
$ bin/kafka-topics.sh --describe --zookeeper localhost:2181
```

*Create topic*
```
$ bin/kafka-topics.sh  --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test

$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: test	Partition: 0	Leader: 1003	Replicas: 1003,1001,1002	Isr: 1003,1001,1002
	Topic: test	Partition: 1	Leader: 1001	Replicas: 1001,1002,1003	Isr: 1001,1002,1003
	Topic: test	Partition: 2	Leader: 1002	Replicas: 1002,1003,1001	Isr: 1002,1003,1001
```
*Sending data to topic (Producing)*
```
$ bin/kafka-console-producer.sh --broker-list localhost:2181 --topic test
> This is first message.
> This is second message.

```

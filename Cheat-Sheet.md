Kafka Cheat Sheet
-

### Cheet-sheet for Normal Kafka:

*Display Topic Information:*
```
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
$ bin/kafka-topics.sh --describe --zookeeper localhost:2181
```

*Create topic:*
```
$ bin/kafka-topics.sh  --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test

$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: test	Partition: 0	Leader: 1003	Replicas: 1003,1001,1002	Isr: 1003,1001,1002
	Topic: test	Partition: 1	Leader: 1001	Replicas: 1001,1002,1003	Isr: 1001,1002,1003
	Topic: test	Partition: 2	Leader: 1002	Replicas: 1002,1003,1001	Isr: 1002,1003,1001
```
*Sending data to topic (Producing):*
```
$ bin/kafka-console-producer.sh --broker-list c3199-node2:6667 --topic test
> This is first message.
> This is second message.

$ bin/kafka-console-producer.sh --broker-list c3199-node2:6667 --topic test < messages.txt 
```

*Consuming the data :*
```
$bin/kafka-console-consumer.sh --bootstrap-server c3199-node2:6667 --topic test --from-beginning
> This is first message.
> This is second message.
```

*Get the Offset location :*
```
# earliest offset :
$ bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list c3199-node2:6667 --topic test --time -2

# Last offset
$ bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list c3199-node2:6667 --topic test --time -1
```

*Change topic retention :*
```
[old Method] $ bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic test --config retention.ms=28800
Updated config for topic "test"

[New Method] $ bin/kafka-configs.sh --zookeeper localhost:2181 --alter  --entity-type topics --entity-name test --add-config retention.ms=1000 
Completed Updating config for entity: topic 'test'.

# This set retention of 8-hours on messages coming to topic test. After 8 hours message will be deleted.
```

*Show under replicated Partitions for topics :*
```
$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe --under-replicated-partitions
	Topic: __consumer_offsets	Partition: 0	Leader: 1003	Replicas: 1002,1003,1001	Isr: 1003,1001
	Topic: __consumer_offsets	Partition: 1	Leader: 1003	Replicas: 1003,1001,1002	Isr: 1003,1001
	Topic: __consumer_offsets	Partition: 2	Leader: 1001	Replicas: 1001,1002,1003	Isr: 1001,1003
	Topic: __consumer_offsets	Partition: 3	Leader: 1001	Replicas: 1002,1001,1003	Isr: 1001,1003
```

*Deleting Topic :*
```
$ bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic test
```

*Consume max messages :*
```
$ bin/kafka-console-consumer.sh --bootstrap-server c3199-node2:6667 --topic test --max-messages 5
```


### Cheet-sheet for Kerbrose-Kafka:

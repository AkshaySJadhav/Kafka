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
$ bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
> This is first message.
> This is second message.

$ bin/kafka-console-producer.sh  --bootstrap-server localhost:9092 --topic test < messages.txt 

In SASL_PLAINTEXT or SASL_SSL, producer.config should have security.protocol=<SASL_PlainText>

$ sh kafka-console-producer.sh  --bootstrap-server localhost:9092 --topic test123 --producer.config producer.config 

```

*Consuming the data :* 
```
$ bin/kafka-console-consumer.sh --from-beginning --bootstrap-server localhost:9092 --topic jdbc3
> This is first message.
> This is second message.

$ sh kafka-console-consumer.sh --broker-list localhost:9092 --topic test123 --consumer.config consumer.config --from-beginning
> This is first message.
> This is second message.

$ cat consumer.config
security.protocol=SASL_PLAINTEXT
```

*Consume max messages :*
```
$ bin/kafka-console-consumer.sh --bootstrap-server c3199-node2:6667 --topic test --max-messages 5
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

### Kafka Debug

```
1. Enable Debug on console :

#/etc/kafka/conf/tools-log4j.properties 
# Replace "log4j.rootLogger=WARN, stderr" with "log4j.rootLogger=DEBUG, stderr"

2. Enable Debug at broker level :

# Ambari >> Kafka >> Config >> Advanced kafka-log4j
# Replace "log4j.rootLogger=INFO, stdout" with "log4j.rootLogger=DEBUG, stdout"

3. Enable Debug for SSL at console level :

For Client -
# export KAFKA_HEAP_OPTS='-Djavax.net.debug=ssl'
# Run the console producer command.

For Server -
# Open kafka-env.sh
# Add KAFKA_HEAP_OPTS='-Djavax.net.debug=ssl' and save the file.

```

###Example of Creating a Consumer Group

```
Creating topic:
#kafka-topics.sh --bootstrap-server localhost:9092 --topic my-topic --create --partitions 3 --replication-factor 1

Creating Consumer group my-first-application:
#kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --group my-first-application

Creating another consumer in same group my-first-application: [You need to run this command in another terminal so that consumer will subscribe to the group]
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --group my-first-application  
```

### List existing topics
 `bin/kafka-topics.sh --zookeeper localhost:2181 --list`

### Purge a topic
 `bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --config retention.ms=1000`
 
... wait a minute ...

 `bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --delete-config retention.ms`
 
### Delete a topic
 `bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic mytopic`

### Get the earliest offset still in a topic
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -2`

### Get the latest offset still in a topic
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1`

### Consume messages with the console consumer
`bin/kafka-console-consumer.sh --new-consumer --bootstrap-server localhost:9092 --topic mytopic --from-beginning`

## Get the consumer offsets for a topic
`bin/kafka-consumer-offset-checker.sh --zookeeper=localhost:2181 --topic=mytopic --group=my_consumer_group`

### Read from __consumer_offsets

Add the following property to `config/consumer.properties`:
`exclude.internal.topics=false`

`bin/kafka-console-consumer.sh --consumer.config config/consumer.properties --from-beginning --topic __consumer_offsets --zookeeper localhost:2181 --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter"`

## Kafka Consumer Groups

### List the consumer groups known to Kafka
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list`  (old api)

`bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list` (new api)

### View the details of a consumer group 
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group <group name>`

## kafkacat

### Getting the last five message of a topic
`kafkacat -C -b localhost:9092 -t mytopic -p 0 -o -5 -e`

## Zookeeper

### Starting the Zookeeper Shell

`bin/zookeeper-shell.sh localhost:2181`

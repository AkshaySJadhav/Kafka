
Configuring Kafka Brokers
===============

   After enabling Kerberos, Ambari sets up a JAAS login configuration file for the Kafka client. Settings in this file will be used for any client (consumer, producer) that connects to a Kerberos-enabled Kafka cluster.

As per the Kafka "console-consumer.sh" script, it search JAAS configuration in "/config/kafka_client_jaas.conf" directory default.

```# check if kafka_jaas.conf in config , only enable client_kerberos_params in secure mode.
KAFKA_HOME="$(dirname $(cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd ))"
KAFKA_JAAS_CONF=$KAFKA_HOME/config/kafka_jaas.conf
if [ -f $KAFKA_JAAS_CONF ]; then
    export KAFKA_CLIENT_KERBEROS_PARAMS="-Djava.security.auth.login.config=$KAFKA_HOME/config/kafka_client_jaas.conf"
fi
exec $(dirname $0)/kafka-run-class.sh kafka.tools.ConsoleConsumer "$@"
````

I have came across an issue where you need to import the JAAS files as $KAFAKA_OPTS variable instead of "KAFKA_CLIENT_KERBEROS_PARAMS". I was getting the "Could not login: the client is being asked for a password" error then I have unset the variable and exported $KAFAKA_OPTS and it worked.


#### 1) Ambari adds the following settings to the file. (Note: serviceName=kafka is required for connections from other brokers.)

Kafka client configuration with keytab, for producers:

```
[root@c1199-node4 bin]# cat /kafka_client_jaas.conf 
KafkaClient {
     com.sun.security.auth.module.Krb5LoginModule required
     useKeyTab=true
     keyTab="/etc/security/keytabs/kafka.service.keytab
     storeKey=true
     useTicketCache=false
     serviceName="kafka"
     principal="kafka/c1199-node4.example.com@HWX.COM";
    };
[root@c1199-node4 bin]#
```

Kafka client configuration without keytab, for producers:

```KafkaClient {
     com.sun.security.auth.module.Krb5LoginModule required
     useTicketCache=true
     renewTicket=true
     serviceName="kafka";
    };
 ```

#### 2) In this scenario we do not need kerbrose ticket to produce and consume the data. Make sure that you are not adding any extra unnecessary parameter in JAAS file. 

```
[root@c1199-node4 bin]# klist 
klist: No credentials cache found (filename: /tmp/krb5cc_0)
[root@c1199-node4 bin]#  export KAFKA_OPTS="-Djava.security.auth.login.config=/kafka_client_jaas.conf"
[root@c1199-node4 bin]# sh  kafka-console-producer.sh --broker-list c1199-node4:6667 --topic Akshay --producer.config /tmp/producer.config
>test1
>Test2
```
You need to pass the security protocol as we are on secure cluster, in this case "/tmp/producer.config" is holding the protocol details.

```
[root@c1199-node2 bin]# cat /tmp/producer.config 
security.protocol=SASL_PLAINTEXT
[root@c1199-node2 bin]# cat /tmp/consumer.config 
security.protocol=SASL_PLAINTEXT
[root@c1199-node2 bin]# 
```

Note : Same configuration goes for the consumer script. 

### Kafka_Debugging :


- you can put the "-x" option to the kafka command as you are having the shells scrips.

e.g 

```[root@c1199-node4 bin]# sh -x kafka-console-producer.sh --broker-list c1199-node4:6667 --topic Akshay --producer.config /tmp/producer.config
+ '[' x = x ']'
+ export KAFKA_HEAP_OPTS=-Xmx512M
+ KAFKA_HEAP_OPTS=-Xmx512M
++++ dirname kafka-console-producer.sh
+++ cd .
+++ pwd
++ dirname /usr/hdf/current/kafka-broker/bin
+ KAFKA_HOME=/usr/hdf/current/kafka-broker
+ KAFKA_JAAS_CONF=/usr/hdf/current/kafka-broker/config/kafka_jaas.conf
+ '[' -f /usr/hdf/current/kafka-broker/config/kafka_jaas.conf ']'
+ export KAFKA_CLIENT_KERBEROS_PARAMS=-Djava.security.auth.login.config=/usr/hdf/current/kafka-broker/config/kafka_client_jaas.conf
+ KAFKA_CLIENT_KERBEROS_PARAMS=-Djava.security.auth.login.config=/usr/hdf/current/kafka-broker/config/kafka_client_jaas.conf
++ dirname kafka-console-producer.sh
+ ./kafka-run-class.sh kafka.tools.ConsoleProducer --broker-list c1199-node4:6667 --topic Akshay --producer.config /tmp/producer.config
>```

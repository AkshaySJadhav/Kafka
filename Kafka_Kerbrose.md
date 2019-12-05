


1) export KAFKA_OPTS="-Djava.security.auth.login.config=/kafka_client_jaas.conf"
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

2) 
```
[root@c1199-node4 bin]# klist 
klist: No credentials cache found (filename: /tmp/krb5cc_0)
[root@c1199-node4 bin]#  export KAFKA_OPTS="-Djava.security.auth.login.config=/kafka_client_jaas.conf"
[root@c1199-node4 bin]# sh  kafka-console-producer.sh --broker-list c1199-node4:6667 --topic Akshay --producer.config /tmp/producer.config
>test1
>Test2
```

Kafka_Debugging :
===============

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

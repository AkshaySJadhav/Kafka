# Setting up single node Kafka server on 2-Way-SSL.


We are going to setup single node Kafka cluster which includes zookeeper server as well on SSL. This repository contains the configuration files for zookeeper, zookeeper clients, Kafka brokers, producers, and consumers for launching a single node kafka cluster with SSL security. 


In this setup, I have downloaded the Kafka 2.7.0 which comes with zookeeper.

Download Link : https://downloads.apache.org/kafka/2.7.0/


1. Create the certificate for Zookeeper :

```
==> Generate CA
#openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650

==> Create Truststore
#keytool -keystore kafka.zookeeper.truststore.jks -alias ca-cert -import -file ca-cert

==> Create Keystore (Make sure you enter the SAN in firstName field i.e localhost)
#keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost

==> Create certificate signing request (CSR)
#keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -certreq -file ca-request-zookeeper

==> Sign the CSR
#openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-zookeeper -out ca-signed-zookeeper -days 3650 -CAcreateserial

==> Import the CA into Keystore
#keytool -keystore kafka.zookeeper.keystore.jks -alias ca-cert -import -file ca-cert

==> Import the signed certificate from step 5 into Keystore
#keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -import -file ca-signed-zookeeper 
```

Once the certificates for zookeeper have been created, you would need update the zookeeper.properties which will allow zookeeper to run on SSL.

#cat zookeeper.properties 
```
secureClientPort=2182
authProvider.x509=org.apache.zookeeper.server.auth.X509AuthenticationProvider
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.trustStore.location=/PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.zookeeper.truststore.jks
ssl.trustStore.password=admin
ssl.keyStore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.zookeeper.keystore.jks
ssl.keyStore.password=admin
ssl.clientAuth=need
```

2. Create certificate for zookeper clients(scripts) :

```
==> Create Truststore
#keytool -keystore kafka.zookeeper.client.jks -alias ca-cert -import -file ca-cert

==> Create Keystore (Make sure you enter the SAN in firstName field i.e localhost)
#keytool -keystore kafka.zookeeper-client.keystore.jks -alias zookeeper-client -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost

==> Create certificate signing request (CSR)
#keytool -keystore kafka.zookeeper-client.keystore.jks -alias zookeeper-client -certreq -file ca-request-zookeeper-client

==> Sign the CSR
#openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-zookeeper-client -out ca-signed-zookeeper-client -days 3650 -CAcreateserial

==> Import the CA into Keystore
#keytool -keystore kafka.zookeeper-client.keystore.jks -alias ca-cert -import -file ca-cert

==> Import the signed certificate from step 5 into Keystore.
#keytool -keystore kafka.zookeeper-client.keystore.jks -alias zookeeper-client -import -file ca-signed-zookeeper-client
```

Once done, you can remove the request and signed certiicate files from the folder. Now we need create a new property file which will contain all the client configuration to connect to SSL Zookeeper.

Just create the file called "zookeeper-client.properties" and update the following properties :

```
#cat zookeeper-client.properties

zookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
zookeeper.ssl.client.enable=true
zookeeper.ssl.protocol=TLSv1.2

zookeeper.ssl.truststore.location=/PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.zookeeper-client.truststore.jks
zookeeper.ssl.truststore.password=admin
zookeeper.ssl.keystore.location=/PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.zookeeper-client.keystore.jks
zookeeper.ssl.keystore.password=admin
```

Now, we are using the same zookeeper-client.properties to connect to the zookeper on SSL port to see if all the certificates are working or not.

```
[root@sdc bin]# ./zookeeper-shell.sh  localhost:2182 -zk-tls-config-file ../config/zookeeper-client.properties
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Connecting to localhost:2182
Welcome to ZooKeeper!
JLine support is disabled

WATCHER::
WatchedEvent state:SyncConnected type:None path:null
ls /brokers/topics
[__consumer_offsets, akshay, kafkasdc, testing, testrun, topicName]
```

3. Create Certificate for Kafka broker and its clients (Producer/Consumer):

```
#keytool -keystore kafka.broker.truststore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.broker.keystore.jks -alias broker -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost
#keytool -keystore kafka.broker.keystore.jks -alias broker -certreq -file ca-request-broker
#openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-broker -out ca-signed-broker -days 3650 -CAcreateserial
#keytool -keystore kafka.broker.keystore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.broker.keystore.jks -alias broker -import -file ca-signed-broker
```

If you have muliple Kafka borker, you would need to create same set of certificates for other brokers. We are almost done on certificate part however in order to produce/consume the data from/to Kafka topic we have to have the certificate for producer/consumer client.

```
#Certificate for Producer:
++++++++++++++++++++++++++
#keytool -keystore kafka.producer.truststore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.producer.keystore.jks -alias producer -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost
#keytool -keystore kafka.producer.keystore.jks -alias producer -certreq -file ca-request-producer
#openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-producer -out ca-signed-producer -days 3650 -CAcreateserial
#keytool -keystore kafka.producer.keystore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.producer.keystore.jks -alias producer -import -file ca-signed-producer

#Certificate for Producer:
++++++++++++++++++++++++++
#keytool -keystore kafka.consumer.truststore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.consumer.truststore.jks -alias consumer -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost
#keytool -keystore kafka.consumer.truststore.jks -alias consumer -certreq -file ca-request-consumer
#openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-consumer -out ca-signed-consumer -days 3650 -CAcreateserial
#keytool -keystore kafka.consumer.truststore.jks -alias ca-cert -import -file ca-cert
#keytool -keystore kafka.consumer.truststore.jks -alias consumer -import -file ca-signed-consumer
````

All set! We are done with SSL certificate part. Now, we would be updating the configuration file server.properties so that Kafka Broker would pick the right protocol and certificates.

```
listeners=SSL://localhost:9092
advertised.listeners=SSL://localhost:9092
zookeeper.connect=localhost:2182

# Properties for SSL Zookeeper Security between Zookeeper and Broker
zookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
zookeeper.ssl.client.enable=true
zookeeper.ssl.protocol=TLSv1.2

zookeeper.ssl.truststore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.broker0.truststore.jks
zookeeper.ssl.truststore.password=admin
zookeeper.ssl.keystore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.broker0.keystore.jks
zookeeper.ssl.keystore.password=admin
zookeeper.set.acl=true


# Properties for SSL Kafka Security between Broker and its clients
ssl.truststore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.broker0.truststore.jks
ssl.truststore.password=admin
ssl.keystore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.broker0.keystore.jks
ssl.keystore.password=admin
ssl.key.password=admin
security.inter.broker.protocol=SSL
ssl.client.auth=required
ssl.protocol=TLSv1.2

```

Upon updating the we would be starting the Kafka broker and will make sure that it pick the certificate we provided in above file. Also, we have created a topic called "ssltest".


Here is the command to produce and consume the data from the specific topic. Make sure that you provide the client certificate while producing/consuming the data.


```
[root@sdc bin]# ./kafka-console-producer.sh --topic ssltest --broker-list localhost:9092 --producer.config ../config/producer.properties
>hello SSL
>THis is my first message
>END

[root@sdc bin]# ./kafka-console-consumer.sh --topic ssltest --from-beginning --consumer.config ../config/consumer.properties --bootstrap-server localhost:9092
END
THis is my first message
hello SSL

# Cat producer.properties

bootstrap.servers=localhost:9092,localhost:9093
security.protocol=SSL
ssl.protocol=TLSv1.2
ssl.truststore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.producer.truststore.jks
ssl.truststore.password=admin
ssl.keystore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.producer.keystore.jks
ssl.keystore.password=admin
ssl.key.password=admin

# Cat  consumer.properties

bootstrap.servers=localhost:9092,localhost:9093
group.id=ssl-consumer
security.protocol=SSL
ssl.protocol=TLSv1.2
ssl.truststore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.consumer.truststore.jks
ssl.truststore.password=admin
ssl.keystore.location=PATH-TO-YOUR-KAFKA-DIR/ssl/kafka.consumer.keystore.jks
ssl.keystore.password=admin
ssl.key.password=admin
```

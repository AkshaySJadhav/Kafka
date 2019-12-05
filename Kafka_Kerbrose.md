



export KAFKA_OPTS="-Djava.security.auth.login.config=/kafka_client_jaas.conf"

[root@c1199-node4 bin]# cat /kafka_client_jaas.conf 
KafkaClient {
     com.sun.security.auth.module.Krb5LoginModule required
     useKeyTab=true
     keyTab="/etc/security/keytabs/kafka.service.keytab"
     storeKey=true
     useTicketCache=false
     serviceName="kafka"
     principal="kafka/c1199-node4.example.com@HWX.COM";
    };
[root@c1199-node4 bin]# 

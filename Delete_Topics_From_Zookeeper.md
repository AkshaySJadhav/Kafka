### Remove Zookeeper entries
=====================

1. Shutdown Kafka


2. Login to Zookeeper
/usr/lib/zookeeper/bin/zkCli.sh -server <zookeeper-node>:2181


3. Remove the topic from /brokers , /admin & /config folder.
```
#cd /usr/hdp/current/zookeeper-server/bin/
#./zkCli.sh -server <FQDN>:2181
  
[zk: FQDN:2181 (connected)] ls /brokers/topics/<topic_to _be_deleted>
[zk: FQDN:2181 (connected)] rmr /brokers/topics/<topic_to _be_deleted>

[zk: FQDN:2181 (connected)] ls /config/topics/<topic_to _be_deleted>
[zk: FQDN:2181 (connected)] rmr /config/topics/<topic_to _be_deleted>

[zk: FQDN:2181 (connected)] ls /admin/delete_topics
[zk: FQDN:2181 (connected)] rmr /admin/delete_topics/<topic_to _be_deleted>
```
4. You can confirm the topic is deleted by running the following and confirming it no longer appears:
#ls /brokers/topics


### Remove the Data files
==================

1. In the broker configuration, you should have the following property which defines where the data is written for the topic:
<log.dirs>/<topic> -<partition>

2. Get a listing of the above directory and look for the topic you have deleted from Zookeeper

3. Move this directory to another location for safe keeping and remove <log.dirs>/<topic>-<partition> on every broker host

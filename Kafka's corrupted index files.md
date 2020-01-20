

### Addressing Kafka's corrupted index files warnings after restarting brokers.



When starting a Kafka Broker, it fails to start with the following exception in the Kafka broker log:

```
[2019-07-08 12:12:58,901] WARN Found a corrupted index file due to requirement failed:
Corrupt index found, index file (/kafka-logs/email-opens/00000000000008365023.index)
  has non-zero size but the last offset is 8365023 which is no larger than the
  base offset 8365023.}. deleting /kafka-logs/email-opens/00000000000008365023.timeindex,
  /kafka-logs/email-opens/00000000000008365023.index and rebuilding index...
  (kafka.log.Log)
```

As the number of messages and topics grow in a Kafka cluster, it takes longer for Kafka server to shutdown safely. Systemd or 
other alternatives that are used to manage Kafka server on the brokers need to the server process enough time to shutdown 
safely.

The issue is caused because of corrupt index files, which is a result of unclean shutdown of the Kafka broker, this can happen
when the kafka JVM process is not able shoutdown properly due to any reason. 

1. Systemd wasn’t letting Kafka shutdown safely.
2. Process kill during shutdonw.
3. OS is overloaded and failed to shudown the process properly.


## Solution :

To resolve this issue, perform the following :

Take backup of the index files mentioned in the log. 
Remove the index file manually from the Kafka logs directory.
Restart the Kafka broker. The index file will be recreated on restarting the Broker.



Referance:
=========
1. https://blog.experteer.engineering/kafka-corrupted-index-file-warnings-after-broker-restart.html
2. https://stackoverflow.com/questions/19394669/why-do-index-files-exist-in-the-kafka-log-directory

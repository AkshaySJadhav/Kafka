

## Addressing Kafka's corrupted index files warnings after restarting brokers.



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


### Solution :
+++++++++++

To resolve this issue, perform the following :

Take backup of the index files mentioned in the log. 
Remove the index file manually from the Kafka logs directory.
Restart the Kafka broker. The index file will be recreated on restarting the Broker.


## Kafka log directory explained || .index/.log/.timeindex Files


For understanding, the log file contains the actual messages structured in a message format. For each message within this file, the first 64bits describe the incremented offset. Now, looking up this file for messages with a specific offset becomes expensive since log files may grow in the range of gigabytes. And to be able to produce messages, the broker actually has to do such kind of lookups to determine the latest offset and be able to further increment incoming messages correctly.

This is why there is an index file. First of all, the structure of the messages within the index file describes only 2 fields, each of them 32bit long:

4 Bytes: Relative Offset
4 Bytes: Physical Position

As described before, the file name represents the base offset. In contrast to the log file where the offset is incremented for each message, the messages within the index files contain a relative offsets to the base offset. The second field represents the physical position of the related log message (base offset + relative offset) and thus, a lookup of O(1) becomes possible.

After all there is to mention, that not every message within a log has it's corresponding message within the index. The configuration parameter index.interval.bytes, which is 4096 bytes by default, sets an index interval which basically describes how frequently (after how many bytes) an index entry will be added.

Regarding the question to size of the .index file there is the following to say: The configuration parameter segment.index.bytes, which is 10MB by default, describes the size of this file. This space is reallocated and will shrink only after log rolls.


Referance:
=========
1. https://blog.experteer.engineering/kafka-corrupted-index-file-warnings-after-broker-restart.html
2. https://stackoverflow.com/questions/19394669/why-do-index-files-exist-in-the-kafka-log-directory

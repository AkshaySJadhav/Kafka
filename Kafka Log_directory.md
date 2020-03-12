

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

You would be seeing following files/folder  under the Log directory :

```
[test@c493-node2 spark2-client]$ ls -lrth /kafka-logs/
total 16K
-rw-r--r-- 1 kafka hadoop   0 Dec  9 05:45 cleaner-offset-checkpoint
-rw-r--r-- 1 kafka hadoop  57 Dec  9 05:45 meta.properties
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-29
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-26
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-23
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-20
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-17
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-14
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-11
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-8
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-2
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-47
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-38
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-35
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-44
drwxr-xr-x 2 kafka hadoop 141 Dec  9 06:17 __consumer_offsets-32
drwxr-xr-x 2 kafka hadoop 161 Dec  9 06:17 __consumer_offsets-41
drwxr-xr-x 2 kafka hadoop 141 Feb  3 07:55 Merger-1
drwxr-xr-x 2 kafka hadoop 161 Feb  3 07:55 Merger-2
drwxr-xr-x 2 kafka hadoop 161 Feb  3 07:55 Merger-0
drwxr-xr-x 2 kafka hadoop 161 Mar 12 07:09 nit-2
drwxr-xr-x 2 kafka hadoop 141 Mar 12 07:09 nit-0
drwxr-xr-x 2 kafka hadoop 141 Mar 12 07:09 nit-1
drwxr-xr-x 2 kafka hadoop 202 Mar 12 07:09 __consumer_offsets-5
drwxr-xr-x 2 kafka hadoop 141 Mar 12 09:55 testing-0
drwxr-xr-x 2 kafka hadoop 141 Mar 12 09:55 testing-2
drwxr-xr-x 2 kafka hadoop 161 Mar 12 09:55 testing-1
drwxr-xr-x 2 kafka hadoop 161 Mar 12 10:30 testing123-2
drwxr-xr-x 2 kafka hadoop 161 Mar 12 10:30 testing123-0
drwxr-xr-x 2 kafka hadoop 141 Mar 12 10:30 testing123-1
```

File : recovery-point-offset-checkpoint : is the internal broker log where Kafka tracks which messages (from-to offset) were successfully checkpointed to disk.

```
cat /kafka-logs/recovery-point-offset-checkpoint

0                             <--- File version i.e 0
28                            <---- Number of topics listed in folder
testing 0 0
__consumer_offsets 8 0
testing 2 0
testing 1 0
__consumer_offsets 35 0
__consumer_offsets 41 0
__consumer_offsets 23 0
__consumer_offsets 47 0
Merger 0 0
testing123 2 0        <--- Here is the topic name is "testing123" 2 is the partition and 0 is latest offset(no data)
Merger 2 0
__consumer_offsets 38 0
__consumer_offsets 17 0
Merger 1 0
__consumer_offsets 11 0
__consumer_offsets 2 0
__consumer_offsets 14 0
nit 1 5
testing123 1 0
nit 2 4
__consumer_offsets 20 0
testing123 0 0
__consumer_offsets 44 0
__consumer_offsets 5 4340
__consumer_offsets 26 0
__consumer_offsets 29 0
nit 0 4
__consumer_offsets 32 0
```

For understanding, the log file contains the actual messages structured in a message format. For each message within this file, the first 64bits describe the incremented offset. Now, looking up this file for messages with a specific offset becomes expensive since log files may grow in the range of gigabytes. And to be able to produce messages, the broker actually has to do such kind of lookups to determine the latest offset and be able to further increment incoming messages correctly.

This is why there is an index file. First of all, the structure of the messages within the index file describes only 2 fields, each of them 32bit long:

1. 4 Bytes: Relative Offset
2. 4 Bytes: Physical Position

As described before, the file name represents the base offset. In contrast to the log file where the offset is incremented for each message, the messages within the index files contain a relative offsets to the base offset. The second field represents the physical position of the related log message (base offset + relative offset) and thus, a lookup of O(1) becomes possible.

After all there is to mention, that not every message within a log has it's corresponding message within the index. The configuration parameter index.interval.bytes, which is 4096 bytes by default, sets an index interval which basically describes how frequently (after how many bytes) an index entry will be added.

Regarding the question to size of the .index file there is the following to say: The configuration parameter segment.index.bytes, which is 10MB by default, describes the size of this file. This space is reallocated and will shrink only after log rolls.



### How to move the data from one Log Directory to another : 

Note : We are moving the topic "testing123 2 0" here :

 1. Stop the broker to avoid any data curruption.
 2 Backup the folders.
 3. Move the folder to another directory. `#mv Testing123-* /tmp`
 4. Edit the `replication-offset-checkpoint` and remove the recently moved topics details and correct the 2 line i.e. no. of toipcs.
 5. Edit the `replication-offset-checkpoint` file from new log directory and add the topics entry and correct the 2nd line i.e. number topics.
 6. Also we need to make the changes in `recovery-point-offset-checkpoint` on both the directory.
 7. Start the broker.
 8. Describe the Topics and check.
 
 
Referance:
=========
1. https://blog.experteer.engineering/kafka-corrupted-index-file-warnings-after-broker-restart.html
2. https://stackoverflow.com/questions/19394669/why-do-index-files-exist-in-the-kafka-log-directory

## Introduction

 Apache Kafka is an open-source distributed event streaming platform. Kafka combines three key capabilities so you can implement your use cases for event streaming end-to-end with a single battle-tested solution:

    To publish (write) and subscribe to (read) streams of events, including continuous import/export of your data from other systems.
    To store streams of events durably and reliably for as long as you want.
    To process streams of events as they occur or retrospectively.

And all this functionality is provided in a distributed, highly scalable, elastic, fault-tolerant, and secure manner. Kafka can be deployed on bare-metal hardware,
virtual machines, and containers, and on-premises as well as in the cloud. You can choose between self-managing your Kafka environments and using fully managed 
services offered by a variety of vendors. 

Kafka basically creating and processing real time data.


Kafka replicates the topic's partitions across the multiple machines for fault-tolerance. Each partition has a leader and one or more number of followers.

Q. I want to know how Kafka chooses the machines that will become the followers of each topic/partition among the possible candidates?

For example, let's say there is 1 topic with 3 partitions {A,B,C} and replication factor is 3. The Kafka is running over 6 machines {1,2,...,6}.

One possible assignment is:

1 2 3 4 5 6
A B C
C A B
B C A
But the following is also possible:

1 2 3 4 5 6
A B C
  A B C
    A B C

ANS: https://github.com/apache/kafka/blob/0.10.0/core/src/main/scala/kafka/admin/AdminUtils.scala#L47-L106

## About

Everything related to kafka and spring kafka

## Block diagram

![alt text](kafka.png?raw=true)

### Architecture

- they are particular streams of data.
- topics are divided into several partitions. we should set them manually
- each partition is ordered
- each message within a partition gets an incremental id, called offset

![Partition diagram](kafka-partition.png?raw=true)

- from the diagram, we can see that offset only have a meaning
  for a specific partition
- order is guaranteed only within a partition. for e.g. in partition 0, we can be sure the message number is 8 has arrived
  after message number 7 in that partition 0. We cannot tell for
  the cross partitions though.
- Data is randomly assigned to partitions unless the partition id
  or key is provided

### Questions

1. When a producer is producing a message, it will specify the topic it wants to send the message to. Is that right? Does it care about partitions?

```
The producer will decide target partition to place any message, depending on:

Partition id, if it's specified within the message
key % num partitions, if no partition id is mentioned
Round robin if neither partition id nor message key is available in the message means only the value is available
```

2. When a subscriber is running - Does it specify its group id so that it can be part of a cluster of consumers of the same topic or several topics that this group of consumers is interested in?

```
The Kafka consumer works by issuing “fetch” requests to the brokers leading the partitions it wants to consume. The consumer offset is specified in the log with each request. The consumer receives back a chunk of log beginning from the offset position. The consumer has significant control over this position and can rewind it to re-consume data if desired.

A consumer group is a set of consumers which cooperate to consume data from some topics. The partitions of all the topics are divided among the consumers in the group. As new group members arrive and old members leave, the partitions are re-assigned so that each member receives a proportional share of the partitions. This is known as rebalancing the group.

This means if a consumer group is set, each consumers will be assign a specific partition of a topic at random if not specified. However, we are able to assign a specific consumer to consume from a specific partition. A specific broker is assigned to manage a consumer group and the partition assignment.
```

3. Does each consumer group have a corresponding partition on the broker or does each consumer have one?

In one consumer group, each partition will be processed by one consumer only.

Possible scenarios:

1. **No. of partitions** > **No. of consumers**

- multiple partitions are assigned to consumers

2. **No. of partitions** === **No. of consumers**

- one to one mapping

3. **No. of partitions** < **No. of consumers**

- one to one mapping and extra consumer is idle so not a good strategy

4. Since this is a queue with an offset for each partition, is it the consumer's responsibility to specify which messages it wants to read? Does it need to save its state?

- No, consumer don't need to be aware of the offset since kafka broker which is assigned as coordinator will handle it. it comes as a response to the consumer. However, the consumer can refetch certain offset topic messages from a partition.

- Kafka (to be specific Group Coordinator) takes care of the offset state by producing a message to an internal \_\_consumer_offsets topic, this behavior can be configurable to manual as well by setting enable.auto.commit to false. In that case consumer.commitSync() and consumer.commitAsync() can help manage offset.

5. What happens when a message is deleted from the queue? - For example, The retention was for 3 hours, then the time passes, how is the offset being handled on both sides?

- If any consumer starts after the retention period, messages will be consumed as per auto.offset.reset configuration which could be latest/earliest. Technically, it's latest (start processing new messages), because all the messages got expired by that time and retention is a topic-level configuration.

5. If i need to consume same messages in two consumers, how do we do it?

- Remember, consumer group is used for parallelism. the group does the same thing. not including the consumers on the same group will do the trick.

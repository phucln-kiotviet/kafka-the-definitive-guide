# Kafka the definitive guide

- The better you understand how Kafka works, the more you can make informed decisions regarding the many trade-offs that are involved in engineering.

## Chapter 1 Meet kafka

- The mapping of a consumer to a partition is often called `ownership` of the partition by the consumer.

- `Producer` could also use a custom partitioner that follow other business rules for mapping message to partitions. Detail in `chapter 3`

- `Consumer` fails, the remaining members of the group will be rebalance the partitions being consumed. Detail in `chapter 4`

- `Broker` owned a partition and it called `leader` of the partition. Partition can replicate to other broker. Figure 1-7. Detail in `chapter 6`.

- `Retention` is the durable storage of messages for some period of time. Default period of time is 7 days or topic reaches a certain size in bytes is 1GB.

- `Stream processing` is covered in `Chapter 11`


- The name: 

```
I thought that since Kafka was a system optimized for writing, using a writer’s name would make sense. I had taken a lot of lit classes in college and liked Franz Kafka. Plus the name sounded cool for an open source project. So basically there is not much of a relationship.
```

## Chapter 2 Installing Kafka

- As figure 2-1: (page 41):

    - `Broker` store broker and topic metadata to Zookeeper

    - `Consume` store consumer metadata and `partition offsets` to Zookeeper

- Create vagrant to setup: 

```sh
vagrant init ubuntu/focal64
```


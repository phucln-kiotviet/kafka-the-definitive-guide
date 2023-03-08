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

- Add `docker-compose.yml` to start:
```sh
# It docker compose not docker-compose cause we are in ubuntu 22.04
docker compose up -d
```

- Find zookeeper container IP:

```sh
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' zookeeper
```

### Broker Configuration

- Some config need to be consider when install broker with cluster:
    - `broker.id`: integer identifier, arbitrary
    - `port`: normal 9092. If port is lower than 1024. Kafka must started as root ( this not recommended)
    - `zookeeper.connect`: to storing broker metadata.
    - `log.dirs`: Kafka persists all messages to this configure.
        - `num.recovery.thread.per.data.dir`: default 1 thread per log directory. If value is 8 and 3 paths specified in `log.dirs` so total is 24 threads.
        - `auto.create.topics.enable`: simple as the name. Tu dong tao topic khi:
            - Producer bat dau ghi message vao topic
            - Consumer bat dau doc message tu topic
            - Bat ky client lay metadata cua topic
    - `Topic` defaults:
        - `num.partitions`: So luong partition trong 1 topic. Default 1. Keep in mind that number of partitions for a topic can only increased, never decreased. How to choose number of partitions:
            - What is throughput you expect to achieve for the topic? Write 100KB or 1 GB per second.
            - What is maximum throughput you expect to achieve when consuming from a single partition. Vi du chung ta luon chi co nhieu nhat 1 consumer. vay neu consumer xu ly nhieu nhat chi duoc 50MB persecond thi partions dap ung throughput 60MB per second is ok.
            - Ve mat ly thuyet tuong tu consumer limit thi producer limit cung co the xem xet de set up partion nhung throughput producer thuong nhanh hon nhieu consumer nen co the bo qua step nay.
            - Neu gui message vao partitions theo keys. Them partitions sau do se rat kho khan. Vay nen tinh toan throughput dua tren tuong lai ko phai hien tai.
            - Xem xet so luong partitions tren moi broker voi luong o dia ( disk) va bang thong mang ( network bandwidth) tren moi broker
        - Example partions: Neu muon throughput 1GB/s. Neu moi consumer xu ly 50MB/s. -> Chung ta can it nhat 20 partitions va 20 consumers de co the doc, ghi 1GB/s.
        - 
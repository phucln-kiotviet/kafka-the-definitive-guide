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
        - `log.retention.ms`: don vi thoi gian nho nhat. Neu ca `.minutes` va `.hours` va `.ms` cung duoc cau hinh, `.ms` se duoc uu tien.
        - `log.retention.minutes`: 
        - `log.retention.hours`: default 168 hours or one week.
        - `log.retention.bytes`: this config per partition. Example 1 topic has 8 partitions and `log.retention.bytes` is 1 GB. So amount of data retained for topic will be 8GB. 
        - Neu ca `log.retention.ms` va `log.retention.bytes` duoc set. 1 trong 2 cau hinh du dieu kien message se bi xoa.
        - `log.segment.bytes`: Default 1GB. log segment dong, `log.retention.ms` moi bat dau hieu luc.
        - `log.segment.ms`: whichever come first Kafka will close a log segment.
        - Disk performance when using time-based segments: Khi dung time base co the xay ra tinh trang nhieu partition ko bao gio dat nguong limit size ( do cau hinh time ngan) va nhieu segment cung start mot thoi diem.

        - `message.max.bytes`: Default 1MB. Lon hon se bi reject
        - `message.max.bytes` can duoc xem set khi cau hinh vi anh huong den: `fetch.message.max` cau hinh lien quan xu ly phia consumer, tuong tu: `replica.fetch.max.bytes` tren cac broker khi cau hinh cluster
### Hardware selection

- Selecting an appropriate hardware configuration for a Kafka broker can be more art than science :v
- `Disk throughput`: hieu suat producer anh huong boi disk throughput cua broker. SSD thi ngon :D, HDD thi kinh te va co the cau hinh RAID de tang hieu nang.
- `Disk Capacity`: nen thua 10% so voi nhu cau su dung, 10% nay dung cho cac file khac.
- `Memory`: Khong nen share kafka voi phan mem khac cung can su dung `page cache`. Dieu nay lam giam performance cua consumer.
- `Networking`:  Bao gom ca write ( producer) , doc ( consumer) va replicate ( cluster)
- `CPU`: CPU ko qua yeu cau cao nhu disk va memory. Message duoc nen de toi uu network va disk. Kafka broker sau do phai giai nen va validate `checksum` va gan `offset`. Sau do can nen lai mot lan nua de ghi vao disk. Day la luc chinh Kafka can CPU. Nhung thuong ko qua cao nhu disk va network.

- `Kafka Cloud`: Nen bat dau xem set tu: `data retention` theo sau la performance cua producer. Neu yeu cau do tre ( latency) la rat thap thi co the xem set SSD, neu ko co the dung vi du AWS EBS. Cuoi cung moi la CPU, memory


### Kafka Cluster
- This section is about config Kafka cluster. Fore more detail replication of data see `Chapter 6`.
- Figure 2-2 A simple Kafka cluster.
- `How many brokers`: Normal it depended on retaining message( how much storage is availabl). Retain 10TB and each broker avai 2TB -> minium is 5 broker. If replication -> will increase at least 100% = 10 broker. 1 nhan to khac can xem xet den la bang thong mang.
- `Broker configuration`: 2 tham so bat buoc khi broker join vao mot cluster:
    - `zookeeper.connect`: Zookeeper cluster va path noi luu metadata phai giong nhau.
    - `broker.id`: ID phai unique.
- `OS Tuning`: place: `/etc/sysctl.conf`
    - `Virtual memory`: 
    - `disk`: outside of select device hardware, or configuration RAID. Filesystem has next largest impact on performance. EXT4 or XFS

### Production concerns
- Garbage collector options:

### Datacenter layout
- Do not place brokers on the same rack, network, power and failed will take down all.

### Colocating Applications on Zookeeper
- Kafka use Zookeeper for storing metadata information about: brokers, topics, and partitions.
- Consumer can use Kafka or Zookeeper for storing offset.



## Chapter 3 Kafka Producers: Writing Messages to Kafka

- Figure 3-1. High-level overview of Kafka producer components. Page 66.
    - Producer: Create `ProducerRecord` which must include: topic and value
    - `ProducerRecord`: May be include [key], [partition]
    - Sau do Producer se `serialize` key va value thanh ByteArrays de gui qua mang.
    - Sau do data duoc gui toi `Partitioner`: Neu chi ro partition trong ProducerRecord thi partitioner se ko lam gi. Neu ko chi ro partitioner se chon partion thuong dua tren ProducerRecord `key`. Cuoi cung producer biet data gui vao partition nao. Luc nay producer se gui batches theo topic, partition toi broker.
    - Neu data ghi vao broker thanh cong producer se nhan ve `RecordMetadata`. Neu ko se nhan ve error va producer se thu ghi vai lan truoc khi tu bo :D

- Constructing a kafka Producer:
    - `bootstrap.servers`: list of `host:port` of brokers. Ko can tat ca vi Producer se lay them thong tin sau khi thiet lap ket noi. Nhung khuyen nghi la 2 trong truong hop 1 broker bi down se con broker khac backup.
    - `key.serializer`:  ????
    - `value.serializer`: ????

- Method send message:
    - Fire and forget: Sent and don't really care if it arrives or not
    - Synchronous send: Sent and wait to check if was success or not
    - Asynchronous send: Sent with callback function, which will triggered when it received response from broker

### Configuring Producers
- Some configuration:
    - `acks`: Bao nhieu partition replicas phai nhan truoc khi producer coi la ghi thanh cong.
        - acks=0: Producer khong cho response to broker. -> Producer ko biet message gui thanh cong hay khong nhung cung gui nhanh nhat co the ( mien mang support).
        - acks=1: Cho 1 success response. Throughput depend on method send: synchronously or asynchronously. Synchronously will increase latency. asynchronously will hidden latency but throughput still limited by number of in-flight messages.
        - acks=all: Can nhan confirm success tu broker khi tat ca da nhan duoc message

    - `buffer.memory`: Amount of memory the producer will be use to buffer message waiting to be sent to brokers. Neu message duoc tao ra tu app nhanh hon bang thong network producer co the out of space.

    - `compression.type`: default message are sent uncompressed. Should use `snappy` invented by Google when performance and bandwidth are concern. Gzip use more CPU but better comprression ratios, recommended when use in cases network and bandwidth is more restricted. 

    - `retries`: default wait 100ms between retries. But we can set `retry.backoff.ms` to control. Thoi gian retry nen duoc test voi thoi gian partition nhan leaders moi.

    - `batch.size`: Producer batch message cung nhau. Producer ko cho batch full moi send. Do do batche loi khong lam cham viec gui message, chi lam su dung nhieu memory cho batch. Set batch qua nho se lam Producer gui message thuong xuyen hon.

    - `linger.ms`: kiem soat thoi gian cho message truoc khi send batch hien tai. Producer send batch khi batch full hoac `linger.ms` dat nguong.

    - `client.id`: just a string
    - `max.in.flight.requests.per.connection`: Bao nhieu message producer se gui den broker ma ko nhan duoc response. Gia tri cao tang su dung memory nhung cung tang throughput. Cao qua cung giam throughput khi batch kem hieu qua. Set 1 se dam bao ghi message vao broker tuan tu.

    - `timeout.ms, request.timeout.ms, and metadata.fetch.timeout.ms`: 
    - `max.block.ms`: Control bao lau producer se block viec gui message.
    - `max.request.size`: size of request Producer will sent ( batch size). nen match voi `message.max.bytes`: size lon nhat broker chap nhan.

### Serializers
- Serializer: Là việc chuyển dữ liệu trên bộ nhớ Heap thành mảng byte để truyền qua mạng
- Custom Serializers: 

### Partition

- Kafka message are key-value. ProducerRecord gom topic name, key, va value. Key co the set null. Key cho 2 muc dich:
    - Key dung de luu message vao partition nao. key null se ghi vao default partion.
    - La thong tin them voi message duoc luu.
- Key null va default partitioner duoc su dung, message duoc luu vao partition available mot cach random. 
- So luong partion cua mot topic ko doi. Thi 1 key luon duoc luu vao mot partition nhat dinh. Dieu nay ko con duoc dam bao khi so luong partition trong topic thay doi.

- Implementing a custom partitioning strategy.

## Chapter 4 Kafka consumers: Reading Data from Kafka

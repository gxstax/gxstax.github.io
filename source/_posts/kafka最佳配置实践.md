---
title: kafka最佳配置实践
date: 2021-12-23 16:01:44
categories:
    - middleware
tags: [middleware]
---

## 1. broker端
 - ### broker集群参数
   1. > **log.dirs**: \
        这是非常重要的参数，指定了 Broker 需要使用的若干个文件目录路径。要知道这个参数是没有默认值的，这说明什么？这说明它必须由你亲自指定。\
        **log.dir**: \
        注意这是 dir，结尾没有 s，说明它只能表示单个路径，它是补充上一个参数用的。\
        <font face="幼圆" color="YellowGreen"> \
        这两个参数应该怎么设置呢？很简单，你只要设置log.dirs，即第一个参数就好了，不要设置log.dir。\
        而且更重要的是，在线上生产环境中一定要为log.dirs配置多个路径，具体格式是一个 CSV 格式，也就是用逗号分隔的多个路径，\
        比如/home/kafka1,/home/kafka2,/home/kafka3这样。如果有条件的话你最好保证这些目录挂载到不同的物理磁盘上。</font>
   2. > **zookeeper.connect** \
        如果我们是单集群，那么很简单，直接设置zookeeper.connect=zk1:2181,zk2:2181,zk3:2181即可。 \
        <font face="幼圆" color="YellowGreen"> \
        这里我们主要考虑多套kafka集群公用一套zookeeper集群, 这时候我们要使用 zookeeper的 chroot，文件挂载方式实现 \    
        如果有两套 Kafka 集群，假设分别叫它们 kafka1 和 kafka2，那么两套集群的zookeeper.connect参数可以这样指定：\
        zk1:2181,zk2:2181,zk3:2181/kafka1和zk1:2181,zk2:2181,zk3:2181/kafka2。\
        切记 chroot 只需要写一次，而且是加到最后的。\
        </font>
        <font color=red>zk1:2181/kafka1,zk2:2181/kafka2,zk3:2181/kafka3</font> 这样的格式是不对的。
   3. > **listeners: PLAINTEXT://host:9092** \
        监听器，其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务。\
        <font face="幼圆" color="YellowGreen"> \
        从构成上来说，它是若干个逗号分隔的三元组，每个三元组的格式为<协议名称，主机名，端口号>。\
        这里的协议名称可能是标准的名字，比如 PLAINTEXT 表示明文传输、SSL 表示使用 SSL 或 TLS 加密传输等；\
        也可能是你自己定义的协议名字，比如CONTROLLER: //localhost:9092。\
        一旦你自己定义了协议名称，你必须还要指定listener.security.protocol.map参数告诉这个协议底层使用了哪种安全协议，\
        比如指定listener.security.protocol.map=CONTROLLER:PLAINTEXT表示CONTROLLER这个自定义协议底层使用明文不加密传输数据。
        </font>
   4. > **advertised.listeners** \
        和 listeners 相比多了个 advertised。Advertised 的含义表示宣称的、公布的，就是说这组监听器是 Broker 用于对外发布的。
   5. > **auto.create.topics.enable=false** \   
        是否允许自动创建 Topic。\
        <font face="幼圆" color="YellowGreen"> \
        建议最好设置成 false，即不允许自动创建 Topic。\
        Topic 应该由运维严格把控，决不能允许自行创建任何 Topic
        </font>
   6. > **unclean.leader.election.enable=false** \
        是否允许 Unclean Leader 选举。\
        <font face="幼圆" color="YellowGreen"> \
        坚决不能让那些落后太多的副本竞选 Leader。</font>
   7. > **auto.leader.rebalance.enable=false** \
        是否允许定期进行 Leader 选举。\
        <font face="幼圆" color="YellowGreen"> \
        换一次 Leader 代价很高的，原本向 A 发送请求的所有客户端都要切换成向 B 发送请求，而且这种换 Leader 本质上没有任何性能收益。\
        所以建议你在生产环境中把这个参数设置成 false。</font>
   8. > **log.retention.{hours|minutes|ms}** \
        控制一条消息数据被保存多长时间。从优先级上来说 ms 设置最高、minutes 次之、hours 最低。\
        <font face="幼圆" color="YellowGreen"> \
        虽然 ms 设置有最高的优先级，但是通常情况下我们还是设置 hours 级别的多一些，比如log.retention.hours=168表示默认保存 7 天的数据，自动删除 7 天前的数据。\
        如果把 Kafka 当作存储来使用，那么这个值就要相应地调大。
        </font>
   9. > **log.retention.bytes** \
        指定 Broker 为消息保存的总磁盘容量大小。\
        <font face="幼圆" color="YellowGreen"> \
        这个值默认是 -1，表明你想在这台 Broker 上保存多少数据都可以，至少在容量方面 Broker 绝对为你开绿灯，不会做任何阻拦。\
        这个参数真正发挥作用的场景其实是在云上构建多租户的 Kafka 集群：设想你要做一个云上的 Kafka 服务，每个租户只能使用 100GB 的磁盘空间，\
        为了避免有个“恶意”租户使用过多的磁盘空间，设置这个参数就显得至关重要了。
        </font>   
   10. > **message.max.bytes** \
         控制 Broker 能够接收的最大消息大小。\
         <font face="幼圆" color="YellowGreen"> \
          默认的 1000012 太少了，还不到 1MB。实际场景中突破 1MB 的消息都是屡见不鲜的，\
          因此在线上环境中设置一个比较大的值还是比较保险的做法。\
          毕竟它只是一个标尺而已，仅仅衡量 Broker 能够处理的最大消息大小，即使设置大一点也不会耗费什么磁盘空间的。
         </font>
 - ### 防消息丢失
    1. > **unclean.leader.election.enable = false** \
         它控制的是哪些 Broker 有资格竞选分区的 Leader。 \
         <font face="幼圆" color="YellowGreen"> \
         如果一个 Broker 落后原先的 Leader 太多，\
         那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。
         </font>
    2. > **replication.factor >= 3** \
         副本数，最好将消息多保存几份，防止消息丢失的主要机制就是冗余。
    3. > **min.insync.replicas > 1** \
         该参数控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。\
         千万不要使用默认值 1。 
    4. > 确保 **replication.factor > min.insync.replicas**\
         如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。\
         推荐设置成 replication.factor = min.insync.replicas + 1。
 - ### 调优吞吐量
    1. > **num.replica.fetchers=不要超过cpu核心数** \
         该参数值表示的是 Follower 副本用多少个线程来拉取消息，默认使用 1 个线程。\
         <font face="幼圆" color="YellowGreen"> \
         如果你的 Broker 端 CPU 资源很充足，不妨适当调大该参数值，加快 Follower 副本的同步速度。\
         因为在实际生产环境中，配置了 acks=all 的 Producer 程序吞吐量被拖累的首要因素，就是副本同步性能。\
         增加这个值后，你通常可以看到 Producer 端程序的吞吐量增加。\
         </font>
    2. > **调优JVM参数，避免频繁fullGC** 
       >  - 设置堆大小(6~8GB) 
             <font face="幼圆" color="YellowGreen"> \
             6~8GB是一个普适的值，可以安心使用，如果想精确调整，建议你可以查看 GC log，\
             特别是关注 Full GC 之后堆上存活对象的总大小，然后把堆大小设置为该值的 1.5～2 倍。\
             如果你发现 Full GC 没有被执行过，手动运行 jmap -histo:live < pid > 就能人为触发 Full GC。\
             </font> 
             &nbsp; 
       >  - GC收集器选择(G1) 
             <font face="幼圆" color="YellowGreen"> \
            竭力避免 Full GC : 如果你的 Kafka 环境中经常出现 Full GC，你可以配置 JVM 参数 -XX:+PrintAdaptiveSizePolicy，来探查一下到底是谁导致的 Full GC。\
            大对象 :  所谓的大对象，一般是指至少占用半个区域（Region）大小的对象。举个例子，如果你的区域尺寸是 2MB，那么超过 1MB 大小的对象就被视为是大对象。要解决这个问题，除了增加堆大小之外，你还可以适当地增加区域大小，设置方法是增加 JVM 启动参数 -XX:+G1HeapRegionSize=N。默认情况下，如果一个对象超过了 N/2，就会被视为大对象，从而直接被分配在大对象区。如果你的 Kafka 环境中的消息体都特别大，就很容易出现这种大对象分配的问题。
             </font>
 - ### 调优延时
    1. > **num.replica.fetchers** \
         <font face="幼圆" color="YellowGreen"> \
         增加 num.replica.fetchers 值以加快 Follower 副本的拉取速度，减少整个消息处理的延时。
         </font>
## 2. producer 端
  - ### 防消息丢失
     1. > 不要使用 producer.send(msg)，而要使用 producer.send(msg, callback)。记住，一定要使用带有回调通知的 send 方法。
     2. > **acks = all** \
          acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。\
          如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
     3. > **retries > 0** \
          设置 retries 为一个较大的值。这里的 retries 为自动重试次数。、
          当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。
  - ### 调优吞吐量
    1. > **batch.size=1048576** \
         该参数值代表生产者一批次发送多少字节大小的数据；\
         <font face="幼圆" color="YellowGreen"> \
         如果你想要增加吞吐量，那么尽量调大该参数值，该值默认16KB，\
         假设你的消息体大小是 1KB，默认一个消息批次也就大约 16 条消息，显然太小了。\
         我们还是希望 Producer 能一次性发送更多的消息。
         </font>
    2. > **linger.ms** \
         该值是和batch.size配对使用的参数，表示每批次缓存时间；\
         <font face="幼圆" color="YellowGreen"> \
         也就是说，如果你设置batch.size=5000，
         如果从上一发送批次发送到现在时间，超过了linger.ms设置的时间，那么即使未达到batch.size设置的5000，这时候也会发送。
         </font>
    3. > **compression.type=lz4/zstd** \
         配置压缩算法，减少I/O传输量，从而提升吞吐量。\
         <font face="幼圆" color="YellowGreen"> \
         当前，和 Kafka 适配最好的两个压缩算法是 LZ4 和 zstd，不妨一试。
         </font>
    4. > **acks=0/1** \
         该参数设置是和防消息丢失配置想违背，如果你追求高吞吐量，那么就要承担消息丢失的风险。\
         <font face="幼圆" color="YellowGreen"> \
         设置为 0, 可以不必等待副本确认就可以直接返回处理下一批数据；\
         设置为 1, 也只需等待一个broker副本确认提交成功，就进行下一批数据处理。
         </font>
    5. > **retries=0** \
         这个参数也和防消息丢失的配置相违背，同样的追求高吞吐量，就必须承担消息丢失风险。\
         <font face="幼圆" color="YellowGreen"> \
         禁用重试当然会提高吞吐量，但是消息发送正确性就得不到保障，这里还是要根据自己的业务做合适的参数调整。
         </font>
    6. > **buffer.memory** \
         <font face="幼圆" color="YellowGreen"> \
         如果你在多个线程中共享一个 Producer 实例，就可能会碰到缓冲区不够用的情形。
         倘若频繁地遭遇 TimeoutException：Failed to allocate memory within the configured max blocking time 这样的异常，
         那么你就必须显式地增加 buffer.memory 参数值，确保缓冲区总是有空间可以申请的。
         </font>
  - ### 调优延时
    1. > **linger.ms=0** \
         <font face="幼圆" color="YellowGreen"> \
         低延时，就是我们希望要把消息尽快的送出去，所以，设置这个参数为 0，意味着只要有消息就发送，不用等待。
         </font>
    2. > **compression.type=none** \
         不要设置压缩算法。
       <font face="幼圆" color="YellowGreen"> \
       毕竟压缩也是要耗费一定的时间的。
       </font>
    3. > **acks=1** \
         该值参数尽量设置的小一点。\
         <font face="幼圆" color="YellowGreen"> \
         Follower 副本同步往往是降低 Producer 端吞吐量和增加延时的首要原因。
         </font>
## 3. consumer 端
  - ### 防消息丢失
     1. > **enable.auto.commit=false** \
          确保消息消费完成再提交。并采用手动提交位移的方式。
          这对于单 Consumer 多线程处理的场景而言是至关重要的。
  - ### 调优吞吐量
     1. > **采用多 Consumer 进程或线程同时消费数据。** \
          适当采用多线程方案增加吞吐量。
          <font face="幼圆" color="YellowGreen"> \
          不过该方案在实施起来是有一定的复杂度的，操作不当还会造成数据丢失，
          我在项目中也写了一个多线程消费者的案例，欢迎大家来讨论。
          </font>
     2. > **fetch.min.bytes** \
          该参数表示 Kafka Broker 端积攒多少字节，就可以返回给 Consumer端。\
          <font face="幼圆" color="YellowGreen"> \
          默认是 1 字节，表示只要 Kafka Broker 端积攒了 1 字节的数据，就可以返回给 Consumer 端，
          这实在是太小了。我们还是让 Broker 端一次性多返回点数据吧。
          </font>
 - ### 调优延时    
     1. > **fetch.min.bytes=1** \
          <font face="幼圆" color="YellowGreen"> \
          在 Consumer 端，我们保持 fetch.min.bytes=1 即可，也就是说，
          只要 Broker 端有能返回的数据，立即令其返回给 Consumer，缩短 Consumer 消费延时。
          </font>

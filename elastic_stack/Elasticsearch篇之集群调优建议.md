### 系统配置要到位
1. 遵照官方建议设置所有的系统参数
2. 参见文档“Setup Elasticsearch -> Important System Configuration”

### ES 设置尽量简洁
1. elasticsearch.yml 中尽量只写必备的参数，其他可以通过 api 动态设置的参数都通过 api 来设定
2. 参见文档“Setup Elasticsearch -> Important Elasticsearch Configuration”
3. 随着 ES 的版本升级，很多网络流传的配置参数已经不再支持，因此不要随便复制别人的集群配置参数

### elasticsearch.yml 中建议设定的基本参数
1. cluster.name
2. node.name
3. node.master/node.data/node.ingest
4. network.host 建议现实指定为内网 ip，不要偷懒直接设为 0.0.0.0
5. discovery.zen.ping.unicast.hosts 设定集群其他节点地址
6. discovery.zen.minimum_master_nodes 一般设定为 2
7. path.data/path.log
8. 除上述参数外再根据需要增加其他的静态配置参数
9. 动态设定的参数有 transient 和 persistent 两种设置，前者在集群重启后会丢失，后者不会，但两种设定都会覆盖 elasticsearch.yml 中的配置
```
PUT /_cluster/settings
{
  "persistent": {
    "discovery.zen.minimum_master_nodes": 2
  },
  "transient": {
    "indices.store.throttle.max_bytes_per_sec": "50mb"
  }
}
```

### 关于 JVM 内存设定
1. 不要超过 31GB
2. 预留一半给操作系统，用来做文件缓存
3. 具体大小根据该 node 要存储的数据量来估算，为来保证性能，在内存和数据量间有一个建议的比例
  - 搜索类项目的比例建议在 1:16 以内
  - 日志类项目的比例建议在 1:48～1:96
4. 假设总数据量大小为 1TB，3 个 node，1 个副本，那么每个 node 要存储的数据量为 2TB/3=666GB，即 700GB 做鱼，做 20% 的预留空间，每个 node 要存储大约 850GB 的数据
  - 如果搜索类项目，每个 node 内存大小为 850GB/16=53GB，大于 31GB。31*16=496，即每个 node 最多存储 496GB 数据，所以需要至少 5 个 node
  - 如果是日志类项目，每个 node 内存大小为 850GB/48=18GB，因此3个节点足够

### ES 写数据过程
1. [refresh](./Elasticsearch篇之分布式特性介绍.md#文档搜索实时性---refresh)
2. [translog](./Elasticsearch篇之分布式特性介绍.md#文档搜索实时性---translog)
3. [flush](./Elasticsearch篇之分布式特性介绍.md#文档搜索实时性---flush)

### 写性能优化
1. 目标是增大写吞吐量-EPS(Events Per Second)越高越好
2. 优化方案
  - 客户端：多线程写，批量写
  - ES：在**高质量数据建模**的前提下，主要是在 refresh、translog 和 flush 之间做文章

### 写性能优化
1. 目标为降低 refresh 的频率
  - 增大 refresh_interval，降低实时性，以增大一次 refresh 处理的文档数，默认是 1s，设置为 -1 直接禁止自动 refresh
  - 增大 index buffer size，参数为 indices.memory.index_buffer_size(静态参数，需要设定在 elasticsearch.yml 中)，默认为 10%
2. 目标是降低 translog 写磁盘的频率，从而提高写效率，但会降低容灾能力
  - index.translog.durability 设置为 async，index.translog.sync_interval 设置需要的大小，比如 120s，那么 translog 会改为每 120s 写一次磁盘
  - index.translog.flush_threshold_size 默认为 512mb，即 translog 超过该大小时会触发一次 flush，那么调大该大小可以避免 flush 的发生
3. 目标为降低 flush 的次数，在 6.x 可优化的点不多，多为 es 自动完成
4. 副本设置为 0，写入完毕再增加
5. 合理的设计 shard 数，并保证 shard 均匀地分配在所有 node 上，充分利用所有 node 的资源
  - index.routing.allocation.total_shards_per_node 限定每个索引在每个 node 上可分配的总主副分片数
  - 5 个 node，某索引有 10 个主分片，1 个副本，上述值应该设置为多少？
    - (10+10)/5=4
    - 实际要设置为 5 个，防止在某个 node 下线时，分片迁移失败的问题
![es-写性能优化.png](./images/es-写性能优化.png)

### 读性能优化
1. 读性能主要受以下几方面影响：
  - 数据模型是否符合业务模型？
  - 数据规模是否过大？
  - 索引配置是否优化？
  - 查询语句是否优化？

### 读性能优化 - 数据建模
1. 高质量的数据建模是优化的基础
  - 将需要通过 script 脚本动态计算的值提前算好作为字段存到文档中
  - 尽量使得数据模型贴近业务模型

### 读性能优化 - 数据规模
1. 根据不同的数据规模设定不同的 SLA
  - 上万条数据与上千万条数据性能肯定存在差异

### 读性能优化 - 索引配置调优
1. 索引配置优化主要包括如下：
  - 根据数据规模设置合理的主分片数，可以通过测试得到最合适的分片数
  - 设置合理的副本数目，不是越多越好

### 读性能优化 - 查询语句调优
1. 查询语句调优主要有以下几种常见手段：
  - 尽量使用 Filter 上下文，减少算分的场景，由于 Filter 有缓冲机制，可以极大提升查询性能
  - 尽量不使用 script 进行字段计算或者算分排序等
  - 结合 profile、explain API 分析慢查询语句的症结所在，然后再去优化数据模型

### 如何设定 Shard 数？
1. ES 的性能基本是线性扩展的，因此我们只要测出1个 Shard 的性能指标，然后根据实际性能需求就能酸楚需要的 Shard 数。比如单 Shard 写入 eps 是 10000，而线上 eps 需求是 50000，那么你需要 5 个 shard。(实际还要考虑副本的情况)
2. 测试 1 个 Shard 的流程如下：
  - 搭建与生产环境相同配置的但节点集群
  - 设定一个但分片零副本的索引
  - 写入实际生产数据进行测试，获取写性能指标
  - 针对数据进行查询请求，获取独行女指标
3. 压测工具可以采用 esrally
4. 燕策的流程还是比较复杂，可以根据经验来设定。如果是搜索引擎场景，单 Shard 大小不要超过 15GB，如果是日志场景，单 Shard 大小不要超过 50GB(Shard 越大，查询性能越低)
5. 此时只要估算出你索引的总数据大小，然后再除以上面的单 Shard 大小也可以得到分片数

### X-Pack Monitoring
1. 官方退出的免费集群监控功能
2. 安装x-pack
```
bin/elasticsearch-plugin install x-pack
bin/kibana-plugin install x-pack
```
3. 然后将 elasticsearch 和 kibana 重启下就行了
4. 因为x-pack有一个账号机制，如果不想用，可以在 elasticsearch.yml 中进行配置
```
vim config/elasticsearch.yml

xpack.security.enabled: false
```
5. Overview
  - Search Rate：查询性能
  - Indexing Rate：写入的性能
  - Search Latency：查询延迟
  - Indexing Latency：写入延迟
6. Node
  - JVM Heap
  - Index Memory：Lucene使用
  - CPU Utilization：CPU占用情况
  - System Load
  - Latency
  - Segment Count：展示这个节点上所有 segment 的数目
  - ......
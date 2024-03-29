
## Codis 的整体架构和基本流程

Codis 集群中包含了 4 类关键组件。

+ `codis server`：这是进行了二次开发的 Redis 实例，其中增加了额外的数据结构，支持数据迁移操作，主要负责处理具体的数据读写请求。
+ `codis proxy`：接收客户端请求，并把请求转发给 `codis server`。
+ `Zookeeper` 集群：保存集群元数据，例如数据位置信息和 `codis proxy` 信息。
+ `codis dashboard` 和 `codis fe`：共同组成了集群管理工具。其中，`codis dashboard` 负责执行集群管理工作，包括增删 `codis server`、`codis proxy` 和进行数据迁移。而 `codis fe` 负责提供 `dashboard` 的 `Web` 操作界面，便于我们直接在 `Web` 界面上进行集群管理。

![Codis架构与关键组件](.pic/2023-03-29-Codis%E6%9E%B6%E6%9E%84%E4%B8%8E%E5%85%B3%E9%94%AE%E7%BB%84%E4%BB%B6.png)

Codis 处理请求过程：

+ 首先，为了让集群能接收并处理请求，要先使用 `codis dashboard` 设置 `codis server` 和 c`odis proxy` 的访问地址，完成设置后，`codis server` 和 `codis proxy` 才会开始接收连接。

+ 然后，当客户端要读写数据时，客户端直接和 `codis proxy` 建立连接。你可能会担心，既然客户端连接的是 `proxy`，是不是需要修改客户端，才能访问 `proxy`？其实，你不用担心，`codis proxy` 本身支持 Redis 的 `RESP` 交互协议，所以，客户端访问 `codis proxy` 时，和访问原生的 Redis 实例没有什么区别，这样一来，原本连接单实例的客户端就可以轻松地和 `Codis` 集群建立起连接了。

+ 最后，`codis proxy `接收到请求，就会查询请求数据和 `codis server` 的映射关系，并把请求转发给相应的 `codis server `进行处理。当 `codis server `处理完请求后，会把结果返回给 `codis proxy`，`proxy` 再把数据返回给客户端。

![Codis处理请求过程](.pic/2023-03-29-Codis%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B.png)


## Codis 的关键技术原理

### 数据如何在集群里分布？

> 在 `Codis` 集群中，一个数据应该保存在哪个 `codis server` 上，这是通过逻辑槽（`Slot`）映射来完成的，具体来说，总共分成两步。

+ 第一步，`Codis` 集群一共有 `1024` 个 `Slot`，编号依次是 `0` 到 `1023`。我们可以把这些 `Slot` 手动分配给 `codis server`，每个 `server` 上包含一部分 `Slot`。当然，我们也可以让 `codis dashboard` 进行自动分配，例如，`dashboard` 把 `1024` 个 `Slot` 在所有 `server` 上均分。

+ 第二步，当客户端要读写数据时，会使用 `CRC32` 算法计算数据 `key` 的哈希值，并把这个哈希值对 `1024` 取模。而取模后的值，则对应 `Slot` 的编号。此时，根据第一步分配的 `Slot` 和 `server` 对应关系，就可以知道数据保存在哪个 `server` 上了。

![数据分布示例](.pic/2023-03-29-%E6%95%B0%E6%8D%AE%E5%88%86%E5%B8%83%E7%A4%BA%E4%BE%8B.png)


数据 `key` 和 `Slot` 的映射关系是客户端在读写数据前直接通过 `CRC32` 计算得到的，而 `Slot` 和 `codis server ` 的映射关系是通过分配完成的，所以就需要用一个存储系统保存下来，否则，如果集群有故障了，映射关系就会丢失。
>  `Slot` 和 `codis server` 的映射关系称为数据路由表（简称路由表）。在 `codis dashboard` 上分配好路由表后，`dashboard` 会把路由表发送给 `codis proxy`，同时，`dashboard` 也会把路由表保存在 `Zookeeper` 中。`codis-proxy` 会把路由表缓存在本地，当它接收到客户端请求后，直接查询本地的路由表，就可以完成正确的请求转发了。

![路由表的分配和使用过程](.pic/2023-03-29-%E8%B7%AF%E7%94%B1%E8%A1%A8%E7%9A%84%E5%88%86%E9%85%8D%E5%92%8C%E4%BD%BF%E7%94%A8%E8%BF%87%E7%A8%8B.png)


## 集群扩容和数据迁移如何进行?
`Codis` 集群扩容包括了两方面：增加 `codis server` 和增加 `codis proxy`。

增加 codis server，这个过程主要涉及到两步操作：

+ 启动新的 `codis server`，将它加入集群；
+ 把部分数据迁移到新的 `server`。

`Codis `集群按照 `Slot` 的粒度进行数据迁移，我们来看下迁移的基本流程。

+ 在源 `server` 上，`Codis` 从要迁移的 `Slot` 中随机选择一个数据，发送给目的 `server`。
+ 目的 `server` 确认收到数据后，会给源 `server` 返回确认消息。这时，源 `server` 会在本地将刚才迁移的数据删除。
+ 第一步和第二步就是单个数据的迁移过程。`Codis` 会不断重复这个迁移过程，直到要迁移的 `Slot` 中的数据全部迁移完成。

![数据迁移流程图](.pic/2023-03-29-%E6%95%B0%E6%8D%AE%E8%BF%81%E7%A7%BB%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

Codis 实现了两种迁移模式，分别是同步迁移和异步迁移：

+ ***同步迁移***
  + 在数据从源 `server` 发送给目的 `server` 的过程中，源 `server` 是 ***阻塞的***，无法处理新的请求操作。这种模式很容易实现，但是迁移过程中会涉及多个操作（`包括数据在源 server 序列化、网络传输、在目的 server 反序列化，以及在源 server 删除`），如果迁移的数据是一个 `bigkey`，源 `server` 就会阻塞较长时间，无法及时处理用户请求。
+ ***异步迁移***：
  + 第一个特点是，当源 `server` 把数据发送给目的 `server` 后，就可以处理其他请求操作了，不用等到目的 `server` 的命令执行完。而目的 `server` 会在收到数据并反序列化保存到本地后，给源 `server` 发送一个 `ACK` 消息，表明迁移完成。此时，源 `server` 在本地把刚才迁移的数据删除。在这个过程中，***迁移的数据会被设置为只读***，所以，源 `server` 上的数据不会被修改，自然也就不会出现“和目的 `server` 上的数据不一致”的问题了。


  + 第二个特点是，对于 `bigkey`，异步迁移采用了 ***拆分指令的方式*** 进行迁移。具体来说就是，对 `bigkey` 中每个元素，用一条指令进行迁移，而不是把整个 `bigkey` 进行序列化后再整体传输。这种 ***化整为零*** 的方式，就避免了 `bigkey` 迁移时，因为要序列化大量数据而阻塞源 `server` 的问题。
    + 但是，当 `bigkey` 迁移了一部分数据后，如果 `Codis` 发生故障，就会导致 `bigkey` 的一部分元素在源 `server`，而另一部分元素在目的 `server`，这就 ***破坏了迁移的原子性***。所以，`Codis` 会在目标 `server` 上，给 `bigkey` 的元素设置一个 ***临时过期时间***。如果迁移过程中发生故障，那么，目标 `server` 上的 `key` 会在过期后被删除，不会影响迁移的原子性。当正常完成迁移后，`bigkey` 元素的临时过期时间会被删除。

---

在 `Codis` 集群中，除了增加 `codis server`，有时还需要增加 `codis proxy`。

> 因为在 `Codis` 集群中，客户端是和 `codis proxy` 直接连接的，所以，当客户端增加时，一个 `proxy` 无法支撑大量的请求操作，此时，就需要增加 `proxy`。直接启动 `proxy`，再通过 `codis dashboard` 把 `proxy` 加入集群就行。
>
> 此时，`codis proxy` 的访问连接信息都会保存在 `Zookeeper` 上。所以，当新增了 `proxy` 后，`Zookeeper `上会有最新的访问列表，客户端也就可以从 `Zookeeper` 上读取 `proxy` 访问列表，把请求发送给新增的 `proxy`。这样一来，客户端的访问压力就可以在多个 `proxy` 上分担处理了。


## 集群客户端需要重新开发吗?

`Codis `集群在设计时，就充分考虑了对现有单实例客户端的兼容性。

`Codis `使用 `codis proxy` 直接和客户端连接，`codis proxy` 是和单实例客户端兼容的。而和集群相关的管理工作（例如请求转发、数据迁移等），都由 `codis proxy`、`codis dashboard `这些组件来完成，不需要客户端参与。

这样一来，业务应用使用 `Codis` 集群时，就不用修改客户端了，可以复用和单实例连接的客户端，既能利用集群读写大容量数据，又避免了修改客户端增加复杂的操作逻辑，保证了业务代码的稳定性和兼容性。

可靠性是实际业务应用的一个核心要求。对于一个分布式系统来说，它的可靠性和系统中的组件个数有关：***组件越多，潜在的风险点也就越多***。和 `Redis Cluster` 只包含 `Redis` 实例不一样，`Codis `集群包含的组件有 `4` 类。

## 怎么保证集群可靠性？

`Codis` 不同组件的可靠性保证方法：
+ `codis server `其实就是 `Redis` 实例，只不过增加了和集群操作相关的命令。`Redis` 的主从复制机制和哨兵机制在 `codis server` 上都是可以使用的，所以，`Codis` 就使用 ***主从集群来保证 `codis server` 的可靠性***。简单来说就是，`Codis` 给每个 `server` 配置从库，并使用哨兵机制进行监控，当发生故障时，主从库可以进行切换，从而保证了 `server` 的可靠性。

+ 在 `Codis` 集群设计时，`proxy `上的信息源头都是来自 `Zookeeper`（例如路由表）。而 `Zookeeper` 集群使用多个实例来保存数据，只要有超过半数的 `Zookeeper` 实例可以正常工作， `Zookeeper` 集群就可以提供服务，也可以保证这些数据的可靠性。

+ 对于 `codis dashboard` 和 `codis fe` 来说，它们主要提供配置管理和管理员手工操作，负载压力不大，所以，它们的可靠性可以不用额外进行保证了。


## 切片集群方案选择建议
![Codis和Redis Cluster切片集群方案区别总结图](.pic/2023-03-29-Codis和Redis%20Cluster切片集群方案区别总结图.png)

对于这两种方案，该怎么选择呢？ 4 条建议：

1. ***从稳定性和成熟度*** 来看，`Codis` 应用得比较早，在业界已经有了成熟的生产部署。虽然 `Codis` 引入了 `proxy` 和 `Zookeeper`，增加了集群复杂度，但是，`proxy` 的无状态设计和 `Zookeeper` 自身的稳定性，也给 `Codis` 的稳定使用提供了保证。而 `Redis Cluster` 的推出时间晚于 `Codis`，相对来说，成熟度要弱于 `Codis`，如果想选择一个成熟稳定的方案，`Codis` 更加合适些。


2. ***从业务应用客户端兼容性*** 来看，连接单实例的客户端可以直接连接 `codis proxy`，而原本连接单实例的客户端要想连接 `Redis Cluster` 的话，就需要开发新功能。所以，如果你的业务应用中大量使用了单实例的客户端，而现在想应用切片集群的话，建议选择 `Codis`，这样可以避免修改业务应用中的客户端。


3. ***从使用 Redis 新命令和新特性*** 来看，`Codis server` 是基于开源的 `Redis 3.2.8 `开发的，所以，`Codis` 并不支持 `Redis` 后续的开源版本中的新增命令和数据类型。另外，`Codis` 并没有实现开源 `Redis` 版本的所有命令，比如 `BITOP、BLPOP、BRPOP`，以及和与事务相关的 `MUTLI、EXEC `等命令。`Codis `官网上列出了不被支持的命令列表，在使用时记得去核查一下。所以，如果想使用开源 `Redis` 版本的新特性，`Redis Cluster` 是一个合适的选择。


4. ***从数据迁移性能维度*** 来看，`Codis` 能支持异步迁移，异步迁移对集群处理正常请求的性能影响要比使用同步迁移的小。所以，如果在应用集群时，数据迁移比较频繁的话，`Codis` 是个更合适的选择。

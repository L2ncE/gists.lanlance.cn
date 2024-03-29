## 场景

### 1. 强制用户下线，让其无法再次登录怎么设计？

**会话管理:** 当用户登录成功时，你可以在服务器端为该用户生成一个唯一的会话 ID，并且在用户的设备上设置一个相应的 cookie。下一次用户请求登录时，服务器会检查这个 cookie 来确认用户的身份。当你想要强制用户下线时，只需要在服务器上销毁这个会话 ID，然后用户的设备就不能再用这个 cookie 来验证自己了。
    
**更改用户状态:** 在用户的数据库记录中添加一个字段，比如叫做 " 是否被禁止 "。当你想要强制用户下线时，将这个字段设置为 " 是 "，那么下次用户试图登录时，系统会因为这个字段是 " 是 " 而拒绝用户的登录请求。
### 2. 多个客户端单一账号怎么禁止登录？

这种问题通常通过服务器控制实现。基本的思路是服务器对每个账号只允许一个活动的 session。这样，当一个新的客户端尝试使用同一账户登录时，服务器可以拒绝该请求，或者结束与旧客户端的 session。以下是这个思路的一种具体实现方式：

1. 核心思想是使用一个集中服务器来存储每个账户的 SessionID。当用户登录时，服务器验证其凭证，并生成一个新的 SessionID。这个 SessionID 将与此用户的账户关联，并存储在服务器上。
2. 当服务器收到客户端的请求时，它会检查该请求的 SessionID。如果 SessionID 匹配存储在服务器上的 ID，那么请求将被处理；否则，请求将被拒绝。
3. 当服务端接收到一个新的登录请求时，它会创建一个新的 SessionID，并改变存储在服务器上的 SessionID。这意味着旧的客户端将不能再通过旧的 SessionID 发送请求，因此它被迫注销。
4. 客户端需要定期发送心跳到服务器以保持其 SessionID 的活跃状态。如果服务器在一定时间内没有收到客户端的心跳，那么它将移除该客户端的 SessionID，迫使其重新登录。

这种方法的一个潜在问题是，如果旧客户端由于无法访问网络而没有收到注销信号，那么它可能会持续认为自己仍在登录状态。为解决此问题，客户端应设计为在检测到网络恢复后尝试向服务器发送心跳。如果心跳被拒，那么客户端应自动注销，并提示用户重新登录。

### 3. 限流器的设计

#### 固定窗口

利用 redis 的原子自增和过期淘汰策略

  - 限流器的计数存放在 redis 中，用 redis 的过期淘汰策略实现限流器的计数的定期更新 
  - 例如针对 接口 A 限流 10000 QPS。redis 的 key 为：“接口 A”，value 为计数值 - 每次接口调用 Redis 用 INC 原子自增命令，自增 1，并设置过期时间为 1s 
  - 初次调用时，因为 redis 中该 key 没有，就直接设置为 1，并设置过期时间为 1s
  - 在这一秒以内的后续调用，每次都自增 1 - 客户端拿到自增后的值如果没有超过限制 10000，就放行 - 如果超过 10000 限制，就不放行，说明超限了
  - 细节实现：为避免超限后无谓的 redis 调用，第一次发现超限时可以记录该值的 TTL 时间，例如只过去 100ms 就有 1w 个请求过来，剩下的 900ms 就不用请求 redis 而是直接返回超限即可。不然这种情况会给 redis 带去额外无谓的流量，例如前面的例子，不做这个细节逻辑的话，redis 的请求量是 10w QPS
  - 精度可调节。假如限流阈值很大，比如 100w，可以把 INC 自增步进/步长调整大一些，例如 100，那么 redis 的 QPS 直接降低 100 倍，为 1w QPS

这样是一个固定窗口的限流器，不能应对突刺情况，可以增加滑动窗口的设计或者使用令牌桶方法。

#### 优化

- 滑动窗口
你可以使用 Redis 提供的有序集合（sorted set）数据结构来存储每一次请求的时间戳，用 score 来表示请求的时间。每次来新的请求，都添加到有序集合中。然后，根据当前的时间戳，删除窗口之外的请求记录，并查看当前窗口内的请求数量是否超过阈值。

- 令牌桶
可以设定每秒钟往桶中放入 100 个令牌，如果有新的请求进来，就从桶中拿走一个令牌，如果桶中没有令牌了，新的请求就需要等待或者被丢弃，这样就可以保护系统不会被突然的大流量压垮。

#### 超高 QPS 优化

使用固定步长 + 本地缓存，每次 Redis 按步长取令牌到本地缓存，本地令牌消耗完后再到 Redis 中进行获取，大幅减少 Redis 压力。如果是突发的高 QPS 流量可以通过动态步长进行实现。

### 4. 短链接系统设计

1. 长网址到短网址转换: 为此，我们可以使用哈希算法，例如 MD5 或 Sha256。将长网址作为输入，获取哈希值，然后从哈希值中获取前若干位作为短网址的唯一标识符。如果同一个长网址多次创建，根据哈希算法的特性，生成的短网址将是相同的。
    
2. 短网址到长网址转换: 需要创建一个数据库或者使用缓存服务器来存储长网址和短网址的映射关系。当用户访问短网址时，系统将查找对应的长网址并进行跳转。
#### 考虑分布式系统和全球部署

要解决跨机房部署的问题，可以使用分布式数据库来存储长短网址的映射，这样可以在跨地域或跨海域部署时实现数据的同步更新，但需要注意数据一致性问题。

为了解决全球部署的问题，可以使用 CDN 来加速全球用户对短网址的访问。
#### 统计 UV 和 PV

可以在每次用户访问短网址时，记录访问日志，收集访问时间，访问的短网址，访问来源等信息。然后可以通过 Hadoop 来做 MapReduce。

### 5. 文章计数系统设计

**使用 Redis 存储统计数据：** 根据业务需求，我们可以为每篇文章创建一个独特的键名，然后将播放数、阅读数、评论数等作为该键的不同的域存储在 Redis 的散列（hash）数据结构中。比如，键名可以是：`article:{文章ID}:count`，然后在这个键下的 hash 中设置播放数（plays）、阅读数（reads）、评论数（comments）等域。
    
**使用到期策略防止内存泄漏：** 对于 Redis 中的每一个键，我们可以设定一个到期时间，这样即使某篇文章长时间没人访问，也不会一直占用内存。

对于大并发的写入压力问题可以采用如下策略：

**消息队列缓冲：** 对于大量的更新统计数据请求，我们可以先放入消息队列中，然后由后台服务逐渐消费这些请求，这样可以平滑处理瞬间的大量请求。

对于去重问题，我们可以采用以下方案：

**使用布隆过滤器：** 对于同一个用户发起的重复的计数请求，我们可以使用布隆过滤器进行去重。布隆过滤器是一种空间效率极高的概率型数据结构，能够判别一个元素是否在集合中。

#### 优化

**1. 分布式存储方案：**  应考虑采用分布式系统，例如，可以使用 Redis Cluster 或者缓存云服务等技术分散服务器的压力，分担存储空间，提高存储和查询功能的性能。

**2. 热点数据与非热点数据分离：**  可以将热点数据（例如，经常被访问或最近被访问的数据）和冷数据（不常被访问的数据）区分开来。热数据依然保存在 Redis 中以维持快速的访问速度，而冷数据可以保存在硬盘上的数据库，如 MySQL 这些，这样既保证了数据的完整性，又有效降低了成本。

### 6. 直播消息服务设计

#### 推模型设计

在推模型设计中，用户在房间内发送消息时，服务器会接收这些消息并将它们异步推送给房间内的所有其他用户，将消息广播到所有客户端。  

优化方法：
1. 对于消息放大问题。考虑使用消息聚合，可以将单个用户的消息汇聚合一起批量发送给其他用户，减少网络压力。同时可以使用多级推送，带宽与网络对此比较敏感的场景，类似于 CDN，通过层级复制和分配，将数据内容分发至边缘节点，通过近端分发的方式解决。
2. 对于房间用户列表的存储和优化，可以通过数据分片和本地缓存来进行优化。也可以使用分布式存储系统，每个子系统只处理部分用户数据。
#### 拉模型设计

拉模型中，当用户在房间内发送消息时，消息会被存储在对应房间的消息列表中。观众的客户端会周期性地轮询服务器，将最新的消息拉到客户端进行展示。

优化方法：
1. 储存消息时，由于每个消息都有其时效性，因此并不需要永久存储所有消息。可以定期对过期的消息进行清理。
2. 轮询方式可以进行优化。例如，增加轮询间隔，或者只有当有新消息时才进行轮询。
3. 拉接口可以进行优化，比如通过设置本地缓存，把一些常用的数据或者重复请求的数据存储到本地，当有请求时先查看本地是否有数据，如果有直接返回，无需每次都去服务端获取，减轻了服务端的压力，同时也提高了响应速度。

### 7. 秒杀系统设计

用户层：用户发出请求后，首先到达系统的前端，我们可以在这个环节加入图形验证码、滑动验证码等反爬虫措施，防止恶意刷单和机器人参与抢购。
    
限流层：用户请求通过反爬虫措施后，接着到达限流层。在这个阶段，我们可以采用比如令牌桶算法来限制流量。如果请求超出了设定的阈值，那么超出部分的请求将不会被处理，直接返回系统繁忙的消息给用户。
    
队列层：限流层通过后，用户请求会进入到队列层, 我们可以使用消息队列（如 Kafka, RabbitMQ) 等技术，异步处理用户请求，缓解高并发带来的压力。
    
业务处理层：这个是秒杀业务处理的核心部分。从队列中取出用户请求，进行库存判断。判断成功后，进行减库操作，并生成订单等业务操作。判断库存和减库操作需要保证原子性，可以采用数据库事务进行处理。为了保证数据一致性，我们可以引入乐观锁或者悲观锁的概念。

#### 库存判断与减库操作

为了保证库存判断和减库操作的原子性，我们需要在数据库级别或者服务级别做控制。

1. 数据库事务： 利用数据库事务的原子性，来保证库存的查询和减库在一个事务中完成。简单说，我们可以先查询库存，然后判断库存数量，最后减少库存，这一整个过程在一个事务中，不会受到其他操作的干扰。此外，此操作中应使用悲观锁或者乐观锁保障并发操作下减库的正确性。
2. 乐观锁：每次先获取商品记录版本号，然后减库操作时候带上版本号。只有当版本号和服务器版本号一致时，才会减库，减库同时升级版本号。因此，只有最早获取版本号的线程可以成功扣库。
3. 悲观锁：悲观锁是乐观锁的一个反面策略，比如读锁和写锁。在读锁期间，所有的写（更新）操作会被挂起，等待读锁消失后再更新；在写锁期间，所有的其他写和读操作都会被挂起，等待写锁消失后再处理。

#### 保证不超卖

1. 库存预扣减：使用缓存如 Redis 等存储商品库存，判断和减库操作都在 Redis 中进行，减库成功后，再异步扣减数据库中的库存。因此，只要保证 Redis 中判断和减库的操作具有原子性就能保证不超卖。
2. 消息队列保证顺序处理： 用消息队列，将用户请求进行排队，每次只处理一次商品减库的操作，保证了线程安全，就可以避免超卖现象。
3. 数据库乐观锁： 通过版本机制，保证库存判断、扣减和版本号升级在一个原子操作里完成，只有版本号一样才能进行扣减，在并发情况下，只有抢到锁的请求才能进行库存扣减操作。

### 8. 海量评论系统

1. **数据分片方案**  
    一种策略是使用一致性哈希算法分配唯一 ID。该方法将 hash 值分配到不同的节点上，当添加或删除节点时，涉及到的重新分配的数据量很小，因此可以避免数据无法访问的问题。也可以考虑使用像 Cassandra 这样的分布式存储系统，它内置了数据分片和复制策略。
    
2. **索引和翻页优化**  
    为了提高翻页效率，可以使用二级索引，将父 ID 和子 ID 关联起来，比如你可以使用<PostID, CommentID 列表>的形式存储评论。然后，对评论进行排序以便于分页。当然，前几页的数据访问频率最高，如果在内存中缓存它们，可以进一步提高读取性能。你还可以利用 Memcached 或 Redis 的 LRU 算法自动策略来管理缓存。
    
3. **冷热数据处理**  
    我们可以设定一个基准让访问频次高于这个基准的数据被标记为热数据，针对读取频繁的热数据，可以利用数据缓存技术，比如 Memcached 或 Redis,把热门数据缓存起来，提高数据读取速度。
    
4. **突发流量的处理**  
    对突发流量这种场景，我们可以考虑使用消息队列，比如 RabbitMQ，Kafka 等。这样，当瞬时流量超过系统处理能力时，请求可以先进入队列中，然后系统按自己的处理能力进行处理，避免系统因为突然的流量增加而崩溃。
    
5. **按照热度排序**  
    对于按照热度排序，可以依靠 Redis 的 Zset 进行处理。Zset 中的元素是唯一的，对于每一篇文章，可以建立一个 Zset，评论 ID 作为 member，热度作为 score 进行存储，这样就能快速地获取热度排名的评论。
    
6. **处理评论删除**  
    对评论的删除，我们可以采取标记删除的方式，也就是并不真实删除数据，只是添加一个标记。等到业务高峰过去，系统空闲的时候，进行删除操作。对于索引的更新，也遵循相同的原则，将修改标记下来，在系统空闲时进行更新。

### 9. 推送去重系统设计

1. 我们需要将用户 ID 和推送的新闻 ID 存储在一个数据结构中。这个数据结构可以是哈希表，其优点是搜索和插入的速度都非常快。
2. 推送新闻前，我们需先查询哈希表来判断用户是否已经接收过这条新闻的推送。如果用户已经接收过，那么我们就跳过这次推送，从而避免发送重复的推送。
3. 数据存储使用分布式 KV 数据库，Redis 或者 TiKV。分布式存储可以有效地处理大规模数据，而 KV 存储由于其读写速度快、延迟低的特性，非常适合用于这种需要频繁读写的场景。
4. 这个系统的设计应该考虑到系统重启或者故障的情况，因此数据需要有一定的持久化手段。我们可以定期将内存中的数据存储到磁盘中，或者利用 Redis 的持久化功能，如定期 Dump、AOF 等。
5. 由于系统涉及到亿级的设备数，我们需要对数据进行一些优化处理。比如，我们可以考虑使用位图 (BitMap) 进行数据存储，位图占用的存储空间小，能有效地节省存储空间。另外，我们还可以选用一些优化的数据结构，如布隆过滤器 (Bloom Filter)，它能有效地解决判断一个元素是否存在于集合中的问题，且占用的内存空间小。
6. 针对数据清除的问题，我们可以采取定期清理的策略，如每天清理一次，将无效的、过期的数据清除出去，保持数据的实时性。
7. 我们在服务端进行去重的同时，设备端也进行去重处理，这样既可以提高去重的效率，也提供了一种安全性保障。对于 Android 设备，可以通过 SharedPreferences 等方式在本地存储已接收的推送信息，对每次推送进行校验，如果已接收，则不再显示。

### 10. 好友共同关注问题

#### 要求

1. 在用户个人界面能看到多少个好友关注他
2. 在用户搜索界面能看到搜索出的所有人分别有多少个好友关注他

#### 实现

对于第一个问题可以将关注关系用以下的 key 表示：uid1_uid2，存在这个 key 则表示 uid1 有关注 uid2。有了这个模型后，可以使用 KV 存储关注关系对；

对于第二个问题，一次检索可以有几十个结果（假设为 20 个），如果当前用户有 500 个好友（关注的人），则至少需要进行 20 * 500=1w 次判断，如果用户有 5000 个好友，则为 10w 次判断，使用 KV 就达不到要求了，这样 KV 压力会非常大。可以使用布隆过滤器，然后将布隆过滤器放在机器内存中，单机存不下的可以对布隆过滤器做分片，这样在内存中判断，并且使用上线程池分发判断任务，速度就非常快了。

即便关注人数不设限，我们也可以认为关注数量高的账号很少，因此可以针对这批账号做特殊处理。


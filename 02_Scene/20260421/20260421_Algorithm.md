# 数据结构与算法 - 类型B 业务场景题（B325-B354）

---

## 一、高并发与缓存场景（6题）

### B325."电商商品详情页的LRU缓存经常被冷数据占满，热点商品频繁被淘汰，如何优化缓存策略？"

**问题分析：**
- LRU仅按时间淘汰，无法区分冷热数据的访问频率
- 突发流量（如爬虫、推荐流量）会将大量冷数据注入缓存，挤走热点
- 纯LRU在面对扫描式访问时效果很差

**解决方案：**
1. **升级为LFU或W-TinyLFU**：Caffeine默认使用W-TinyLFU，结合频率与时间，命中率高15%~30%
2. **热点探测+本地缓存**：通过访问计数识别热点Key，额外放一份到Caffeine本地缓存
3. **分片缓存**：按商品类别分片，避免冷数据挤占热数据的槽位
4. **保护窗口**：新数据先进Window区，只有被多次访问才进入Main区（W-TinyLFU的设计）
5. **布隆过滤器前置**：对确定不存在的Key直接拦截，避免缓存穿透

**方案对比：**

| 方案 | 命中率提升 | 实现复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| W-TinyLFU | 15%~30% | 低（用Caffeine） | 通用 |
| 热点探测+本地 | 20%~40% | 中 | 热点集中的场景 |
| 分片缓存 | 10%~15% | 中 | 数据有明确分片键 |
| 保护窗口 | 15%~20% | 低 | 扫描式访问 |

---

### B326."用户搜索自动补全需要支持百万级词条的实时前缀匹配，后端如何高效实现？"

**问题分析：**
- 传统LIKE查询无法利用索引，百万级数据全表扫描
- 需要支持中文拼音、首字母、模糊匹配
- 要求毫秒级响应

**解决方案：**
1. **Trie前缀树**：内存中构建Trie，前缀匹配O(L)，L为查询词长度
2. **双数组Trie**：压缩存储，空间降至1/10，适合百万级词条
3. **ES倒排索引**：利用edge_ngram分词器自动生成前缀索引
4. **多级缓存**：热门查询→本地Caffeine → Redis → ES
5. **拼音支持**：构建拼音Trie，映射到汉字Trie

**推荐架构：**
```
用户输入 → Caffeine热门查询(1ms)
       → Redis前缀匹配(5ms) 
       → ES edge_ngram搜索(50ms)
```

**关键细节：**
- Trie节点存储Top5热词引用，避免遍历子树
- 用HashMap替代数组子节点，支持中文/拼音混合
- 异步构建Trie，定时全量替换避免阻塞

---

### B327."分布式缓存集群新增节点后，大量请求穿透到数据库，如何用一致性哈希解决？"

**问题分析：**
- 取模哈希（hash%N）在节点变化时，几乎所有Key映射改变
- 新增1个节点，约(N-1)/N的Key需要迁移
- 迁移期间，旧映射的Key在缓存中找不到 → 雪崩式穿透

**解决方案：**
1. **一致性哈希环**：节点和数据都映射到哈希环上，数据顺时针找第一个节点
2. **虚拟节点**：每个物理节点对应150~200个虚拟节点，保证均匀分布
3. **数据迁移量**：新增节点只影响相邻节点的数据，迁移量从O(K)降到O(K/N)
4. **预迁移**：新节点上线前，先将即将迁移的数据异步复制到新节点
5. **双读过渡**：迁移期间，旧节点和新节点都查，确保不穿透

**Java实现要点：**
```java
TreeMap<Integer, String> ring = new TreeMap<>();
// 虚拟节点
for (String node : nodes) {
    for (int i = 0; i < VIRTUAL_COUNT; i++) {
        ring.put(hash(node + "#" + i), node);
    }
}
// 查找
String getNode(String key) {
    Map.Entry<Integer, String> entry = ring.ceilingEntry(hash(key));
    return entry == null ? ring.firstEntry().getValue() : entry.getValue();
}
```

---

### B328."朋友圈信息流需要展示好友最近发布的动态，如何用跳表+定时任务实现高效的时间线查询？"

**问题分析：**
- 信息流核心操作：按时间倒序查询好友动态，分页加载
- 好友数可能上千，动态总量百万级
- 需要支持范围查询（某个时间段的动态）

**解决方案：**
1. **跳表存储时间线**：以时间戳为score，动态ID为value，利用跳表的范围查询能力
2. **写扩散 vs 读扩散**：
   - 写扩散（推模式）：发帖时写入所有好友的时间线 → 读取快但写入放大
   - 读扩散（拉模式）：读取时实时聚合好友动态 → 写入轻但读取慢
3. **混合模式**：大V用拉模式，普通用户用推模式
4. **Redis ZSet**：底层就是跳表，天然支持ZRANGEBYSCORE

**Redis实现：**
```bash
# 发布动态
ZADD user:timeline:{uid} {timestamp} {post_id}
# 查询时间线
ZREVRANGEBYSCORE user:timeline:{uid} +inf -inf WITHSCORES LIMIT 0 20
```

**性能优化：**
- 冷热分离：3天内数据在Redis，更早的在HBase
- 预计算：定时任务提前计算关注人的TopN动态

---

### B329."电商秒杀场景需要快速判断用户是否已参与过，如何用布隆过滤器实现高效去重？"

**问题分析：**
- 百万级用户参与判断，若查数据库或Redis Set，内存开销大（每个用户ID约50字节）
- 要求毫秒级判断，且允许极小误判率（误判=认为已参与，只是少一次机会）

**解决方案：**
1. **布隆过滤器**：每个用户ID经k次哈希映射到位数组
   - 判定不存在 → 一定没参与（0误判）
   - 判定存在 → 可能误判（需二次确认）
2. **内存开销**：100万用户，1%误判率 → 约1.2MB（对比Redis Set需50MB）
3. **二次确认**：布隆过滤器判断存在后，再查Redis/DB确认
4. **定期重建**：布隆过滤器不支持删除，活动结束后重建

**实现方案：**
```java
// Guava BloomFilter
BloomFilter<Long> bloomFilter = BloomFilter.create(
    Funnels.longFunnel(), 
    1_000_000,  // 预期元素数
    0.01        // 1%误判率
);

// 判断流程
if (!bloomFilter.mightContain(userId)) {
    // 一定未参与，放行
} else {
    // 可能已参与，查Redis确认
    if (redis.sismember("seckill:users:{activityId}", userId)) {
        return "已参与";
    }
    // 布隆误判，放行
}
```

---

### B330."热点数据在多级缓存（Caffeine→Redis→DB）中的一致性如何保证？如何处理缓存穿透？"

**问题分析：**
- 多级缓存带来一致性问题：DB更新→Redis→Caffeine的延迟
- 缓存穿透：查询不存在的数据，每级都miss
- 缓存击穿：热点Key过期瞬间大量请求打到DB

**解决方案：**

**一致性保障：**
1. **主动失效**：DB更新后发MQ消息，各节点监听并失效本地缓存
2. **短TTL + 最终一致**：本地缓存TTL设短（如10s），Redis TTL设长（如5min）
3. **版本号校验**：缓存存储数据+版本号，读取时校验版本

**穿透防护：**
1. **布隆过滤器**：前置过滤不存在的Key
2. **缓存空值**：对不存在的Key缓存null值，TTL 5min
3. **接口校验**：参数格式/范围校验在入口拦截

**击穿防护：**
1. **互斥锁**：只允许一个线程回源，其他等待
2. **逻辑过期**：缓存永不过期，但数据中带逻辑过期时间，过期后异步刷新
3. **预加载**：热点Key在过期前主动刷新

```java
// 互斥锁防击穿
public Object get(String key) {
    Object value = caffeine.get(key);
    if (value != null) return value;
    
    String lockKey = "lock:" + key;
    if (redis.setnx(lockKey, "1", 10, SECONDS)) {
        try {
            value = db.query(key);
            caffeine.put(key, value);
            redis.set(key, value, 5, MINUTES);
        } finally {
            redis.del(lockKey);
        }
    } else {
        Thread.sleep(50); // 等待后重试
        return get(key);
    }
    return value;
}
```

---

## 二、数据一致性场景（5题）

### B331."分布式环境中多个服务节点需要判断某个资源是否被独占，如何用分布式锁+Redisson实现？考虑锁续期和可重入"

**问题分析：**
- 多节点并发访问共享资源，需互斥
- 业务执行时间不确定，锁可能提前过期
- 同一线程可能重入同一把锁

**解决方案：**
1. **Redisson分布式锁**：基于SETNX + Lua脚本保证原子性
2. **看门狗续期**：默认锁30s，每10s（1/3租约）检查锁还被持有则续期
3. **可重入**：Redis Hash结构，field=线程ID，value=重入次数
4. **加锁Lua脚本**：判断是否存在→不存在则加锁→存在且是自己则重入次数+1

**Redisson关键源码逻辑：**
```java
// 加锁
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return nil
end
// 解锁
if (redis.call('hexists', KEYS[1], ARGV[2]) == 0) then
    return nil
end
local counter = redis.call('hincrby', KEYS[1], ARGV[2], -1)
if (counter > 0) then
    redis.call('pexpire', KEYS[1], ARGV[1])
    return 0
else
    redis.call('del', KEYS[1])
    redis.call('publish', KEYS[2], ARGV[1])
    return 1
end
```

**注意事项：**
- RedLock在主从切换时可能不安全（Martin Kleppmann的论证）
- 锁超时时间要大于业务执行时间，但看门狗已自动处理

---

### B332."批量导入100万条数据时需要去重判断，如何在O(1)时间复杂度内判断数据是否已存在？"

**问题分析：**
- 100万条数据逐条查DB判断是否存在，最坏1百万次IO
- 用Redis Set存储所有已有数据，内存约50MB（假设每条50字节）
- 需要O(1)判断，且批量场景可容忍极小误判

**解决方案：**
1. **布隆过滤器预过滤**：100万数据，1%误判率 → 仅1.2MB内存
   - 判定不存在 → 直接插入，无需查DB
   - 判定存在 → 查DB二次确认
2. **批量查询优化**：二次确认时用`WHERE id IN (...)`批量查
3. **分批次提交**：每1000条一批，减少事务开销

**优化后流程：**
```
100万条数据
→ 布隆过滤器过滤（1.2MB，O(1)）
→ ~990万条确认不存在 → 直接INSERT
→ ~10万条可能存在 → 批量SELECT确认
→ 真正存在的跳过，误判的INSERT
```

**性能对比：**

| 方案 | DB查询次数 | 内存 |
|------|-----------|------|
| 逐条查DB | 100万 | 0 |
| Redis Set | 100万（O(1)） | 50MB |
| 布隆过滤器 | ~10万 | 1.2MB |
| 布隆+批量查 | ~100次批量 | 1.2MB |

---

### B333."社交平台的好友关系需要判断两人是否为好友，如何用位图或布隆过滤器优化判断？"

**问题分析：**
- 每个用户平均200个好友，1亿用户 = 200亿条关系
- 存MySQL：4亿行（双向存2行），查询单次O(log n)
- 存Redis Set：每个用户一个Set，内存开销约100GB

**解决方案：**
1. **Redis位图**：每个用户一个Bitmap，bit位表示是否与某用户有好友关系
   - 1亿用户 → 每个Bitmap约12MB，全量1.2TB，不现实
2. **好友列表+HashSet**：只存每个人的好友列表，查询时O(1)
   - 好友关系不存在则不可能误判（无假阳性）
3. **分段布隆过滤器**：只对活跃用户构建布隆过滤器
   - 查询"非好友"快速返回，查询"好友"再确认
4. **实际方案**：Redis Set + 冷热分离
   - 活跃用户好友列表在Redis
   - 不活跃用户存DB，按需加载

**优化效果：**
```java
// Redis Set判断好友关系
Boolean isFriend = redis.sismember("friends:{uidA}", uidB);
// O(1)判断，内存：1亿用户×200好友×8字节 ≈ 160GB
// 冷热分离后活跃10%→16GB可接受
```

---

### B334."订单系统需要按时间范围查询并分页，如何利用B+树索引和覆盖索引优化查询？"

**问题分析：**
- `SELECT * FROM orders WHERE user_id=? AND create_time BETWEEN ? AND ? ORDER BY create_time DESC LIMIT 20`
- 如果只有user_id索引，还需回表查create_time，然后filesort
- 百万级订单量下，查询可能秒级

**解决方案：**
1. **联合索引（user_id, create_time）**：
   - 索引已按user_id+create_time排序，范围扫描无需filesort
   - 最左前缀匹配user_id，然后在同一user_id下按create_time有序
2. **覆盖索引**：若只查索引列，无需回表
   ```sql
   SELECT order_id, create_time FROM orders 
   WHERE user_id=? AND create_time BETWEEN ? AND ?
   -- 联合索引(user_id, create_time, order_id)可覆盖
   ```
3. **延迟关联**：先通过覆盖索引查到主键，再回表
   ```sql
   SELECT o.* FROM orders o 
   JOIN (SELECT id FROM orders WHERE user_id=? AND create_time BETWEEN ? AND ? 
         ORDER BY create_time DESC LIMIT 20) t ON o.id = t.id
   ```
4. **分页优化**：深度分页用游标替代OFFSET
   ```sql
   -- 不用OFFSET，用上一页最后一条的create_time作为游标
   WHERE user_id=? AND create_time < ? ORDER BY create_time DESC LIMIT 20
   ```

---

### B335."跨服务的分布式事务中，如何用TCC+幂等表保证最终一致性？补偿失败怎么处理？"

**问题分析：**
- 跨服务调用（如订单→库存→支付），任一环节失败需回滚
- TCC三阶段：Try（预留资源）→ Confirm（确认提交）→ Cancel（释放资源）
- 补偿可能因网络/服务不可用而失败

**解决方案：**
1. **TCC + 幂等表**：
   - 每个服务维护事务日志表（xid, branch_id, status）
   - Try阶段写入状态为"trying"，Confirm/Cancel更新状态
   - 重复调用检查幂等表，已Confirm/Cancel的直接返回成功
2. **补偿失败重试**：
   - 定时任务扫描"trying"超时记录，主动Cancel
   - Confirm/Cancel失败时MQ重试，指数退避
3. **最终一致性保障**：
   - 最大努力通知：重试N次后人工介入
   - 对账补偿：T+1日对账修复不一致数据

**TCC幂等表设计：**
```sql
CREATE TABLE tcc_transaction (
    xid VARCHAR(64) PRIMARY KEY,       -- 全局事务ID
    branch_id VARCHAR(64),             -- 分支事务ID
    status ENUM('trying','confirmed','cancelled'),
    biz_id VARCHAR(64),                -- 业务ID（幂等键）
    retry_count INT DEFAULT 0,
    update_time DATETIME,
    INDEX idx_status_time(status, update_time)  -- 扫描超时
);
```

---

## 三、系统稳定性场景（5题）

### B336."实时排行榜需要支持千万级用户的分数更新和TopN查询，如何用跳表+Redis ZSet实现？"

**问题分析：**
- 排行榜核心操作：更新分数、查询TopN、查询个人排名
- 千万级用户，更新QPS可能10万+
- 要求更新和排名查询均在毫秒级

**解决方案：**
1. **Redis ZSet**：底层跳表+哈希表，天然支持排序
   - ZINCRBY：O(log N)更新分数
   - ZREVRANGE：O(log N + M)查询TopM
   - ZREVRANK：O(log N)查询个人排名
2. **分片排行榜**：按规则分片（如按地区、按时间段）
3. **定时快照**：每分钟快照TopN到MySQL，避免长期依赖Redis
4. **缓存TopN**：TopN结果缓存1秒，减少ZREVRANGE频率

**千万级性能优化：**
```bash
# 更新分数（O(log N)，N=1000万时约24次比较）
ZINCRBY leaderboard 100 "user:12345"

# 查询Top10（O(log N + 10) ≈ 34次比较）
ZREVRANGE leaderboard 0 9 WITHSCORES

# 查询个人排名（O(log N)）
ZREVRANK leaderboard "user:12345"
```

**注意：** 千万级ZSet内存约2GB（每成员约200字节），可接受

---

### B337."海量日志中查找出现频率TopK的IP地址，如何用堆+分治思想在有限内存中完成？"

**问题分析：**
- 100GB日志文件，内存只有4GB
- 需要统计每个IP出现频率并找出TopK
- 不能一次性加载到内存

**解决方案（分治+堆）：**
1. **分治拆分**：顺序读取100GB日志，对每个IP取hash(IP)%100，分发到100个小文件中（每个约1GB）
2. **单文件统计**：对每个小文件，用HashMap统计IP频率（1GB文件约1亿行，HashMap约2GB可装下）
3. **单文件TopK**：每个文件用小顶堆维护TopK
4. **全局TopK**：合并100个文件的TopK（100K个候选），再用小顶堆取最终TopK

**时间复杂度：** O(N)读取 + O(N)统计 + O(NlogK)堆操作

**优化变种：**
- 如果只需Top10，可以流式处理：维护全局小顶堆，但HashMap放不下
- 改用Count-Min Sketch：O(1)计数，O(K)空间，有少量误差

**伪代码：**
```
Phase 1: Split
  for each line in 100GB log:
      file_index = hash(ip) % 100
      write ip to split_files[file_index]

Phase 2: Count & Local TopK
  topK_candidates = []
  for each split_file:
      map = HashMap<IP, Count>()
      for each ip in split_file:
          map[ip]++
      local_topK = min_heap(map, K)
      topK_candidates.addAll(local_topK)

Phase 3: Global TopK
  return min_heap(topK_candidates, K)
```

---

### B338."服务A依赖服务B的返回值做分组聚合，数据量百万级时如何避免OOM？"

**问题分析：**
- 服务A调用服务B获取百万条数据，然后在内存中按某字段分组聚合
- 百万条数据+分组Map → 内存可能超2GB
- 聚合结果可能也很大

**解决方案：**
1. **流式处理**：服务B返回流式数据（分页/游标），服务A逐批处理
2. **下推聚合**：让服务B做聚合（SQL GROUP BY），只返回聚合结果
3. **分片聚合**：按分组键哈希分片，每片独立聚合，最后合并
4. **外部排序**：数据溢写到磁盘，归并排序后分组

**推荐方案（下推聚合）：**
```java
// 不好的做法：全量拉取后内存聚合
List<Order> all = orderService.getAll(userId); // 百万条
Map<String, List<Order>> grouped = all.stream()
    .collect(Collectors.groupingBy(Order::getCategory));

// 好的做法：下推到DB聚合
List<CategorySum> result = orderService.getCategorySummary(userId);
// SQL: SELECT category, SUM(amount) FROM orders WHERE user_id=? GROUP BY category
```

**如果必须内存聚合：**
```java
// 流式分页处理
int page = 0;
Map<String, Long> summary = new HashMap<>();
while (true) {
    List<Order> batch = orderService.getPage(userId, page++, 1000);
    if (batch.isEmpty()) break;
    batch.forEach(o -> summary.merge(o.getCategory(), o.getAmount(), Long::sum));
}
```

---

### B339."接口P99延迟突然从50ms飙到2s，排查发现是HashMap在并发场景下退化为链表导致，如何修复？"

**问题分析：**
- HashMap在JDK7中并发put可能形成环形链表，get时死循环
- JDK8虽修复了环形链表，但仍可能数据丢失、ConcurrentModificationException
- 某个热点HashMap被多线程并发读写，退化为O(n)查找

**解决方案：**
1. **替换为ConcurrentHashMap**：分段锁（JDK7）/ CAS+synchronized（JDK8）
2. **如果读多写少**：CopyOnWrite或volatile+不可变Map
3. **如果是缓存**：用Caffeine/Guava Cache（线程安全）

**修复方案：**
```java
// 问题代码
private Map<String, Data> cache = new HashMap<>();

// 方案1：替换为ConcurrentHashMap
private Map<String, Data> cache = new ConcurrentHashMap<>();

// 方案2：如果读远多于写
private volatile Map<String, Data> cache = new HashMap<>();
public void put(String key, Data value) {
    Map<String, Data> newMap = new HashMap<>(cache);
    newMap.put(key, value);
    cache = newMap; // volatile写，对读线程立即可见
}

// 方案3：用Caffeine
private Cache<String, Data> cache = Caffeine.newBuilder()
    .maximumSize(10000)
    .build();
```

**根因分析：** P99飙高是因为HashMap退化后get从O(1)→O(n)，高并发下多个线程同时在链表上遍历，CPU飙高

---

### B340."定时任务扫描超时订单，数据量百万级，如何避免全表扫描并保证及时性？"

**问题分析：**
- `SELECT * FROM orders WHERE status='UNPAID' AND create_time < NOW() - INTERVAL 30 MINUTE`
- 百万级订单，status和create_time联合索引选择性差（大部分已支付）
- 全表扫描耗时数秒，影响主库性能

**解决方案：**
1. **延迟队列**：创建订单时发延迟消息，30分钟后触发检查
   - RocketMQ延迟消息 / Redis ZSet + 定时轮询
2. **时间轮**：Netty HashedWheelTimer，适合短延迟高并发
3. **索引优化**：`(status, create_time)`联合索引，但status选择性差效果有限
4. **分桶扫描**：按create_time分桶，每次只扫描一个小时间窗口
5. **Binlog监听**：Canal监听订单状态变更，实时处理

**推荐方案（延迟消息）：**
```java
// 创建订单时发送延迟消息
rocketMQTemplate.syncSend("order-timeout", 
    MessageBuilder.withPayload(orderId).build(),
    3000, // 超时
    18    // RocketMQ延迟等级18=2h
);

// 消费者处理
@RocketMQMessageListener(topic = "order-timeout", consumerGroup = "timeout-group")
public class OrderTimeoutListener implements RocketMQListener<String> {
    public void onMessage(String orderId) {
        Order order = orderService.getById(orderId);
        if (order.getStatus() == UNPAID) {
            orderService.cancelOrder(orderId);
        }
    }
}
```

---

## 四、业务设计场景（5题）

### B341."设计一个短链接生成服务，如何保证生成的短码全局唯一？如何处理哈希冲突？"

**问题分析：**
- 长URL映射为短码（如6位Base62 = 568亿种组合）
- 要求短码唯一、不可预测、生成快
- 需要处理哈希冲突（两个长URL生成相同短码）

**解决方案：**
1. **自增ID + Base62编码**：
   - 数据库自增ID → Base62编码 → 短码
   - 优点：简单、唯一、有序
   - 缺点：可预测、暴露业务量
2. **MurmurHash + 冲突检测**：
   - 长URL做MurmurHash，取前6位Base62
   - 查DB检测冲突，冲突则加salt重哈希
3. **预生成+发号器**：
   - 预生成一批短码存Redis，用完再生成
   - Leaf号段模式，每次取1000个号
4. **布隆过滤器防冲突**：先查布隆过滤器，存在则直接返回已有短码

**推荐方案（预生成+发号器）：**
```java
// Leaf号段模式
class ShortCodeGenerator {
    AtomicLong current;
    long maxId;
    
    synchronized String nextCode() {
        if (current == null || current.get() > maxId) {
            // 从DB取一个号段: UPDATE leaf SET max_id = max_id + step
            Segment segment = leafDao.getNextSegment();
            current = new AtomicLong(segment.start);
            maxId = segment.end;
        }
        return Base62.encode(current.getAndIncrement());
    }
}
```

---

### B342."设计一个限流器，如何用滑动窗口算法实现精确的QPS限流？与固定窗口、漏桶的区别？"

**问题分析：**
- 需要精确控制每秒请求数不超过阈值
- 固定窗口有边界突刺问题（窗口边界处2倍流量）
- 需要选择合适的限流算法

**三种算法对比：**

| 算法 | 平滑性 | 突刺 | 实现复杂度 |
|------|--------|------|-----------|
| 固定窗口 | 差 | 有边界突刺 | 最低 |
| 滑动窗口 | 好 | 无 | 中 |
| 漏桶/令牌桶 | 最好 | 无 | 高 |

**滑动窗口实现：**
```java
class SlidingWindowRateLimiter {
    // 每个小窗口的计数器
    TreeMap<Long, Integer> windows = new TreeMap<>();
    int windowSize = 1000; // 1秒
    int subWindowCount = 10; // 10个子窗口
    int limit;
    
    synchronized boolean tryAcquire() {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSize;
        
        // 清除过期窗口
        windows.headMap(windowStart).clear();
        
        // 计算当前总请求数
        int total = windows.values().stream().mapToInt(i -> i).sum();
        
        if (total >= limit) return false;
        
        // 当前子窗口计数+1
        long subWindowKey = now / (windowSize / subWindowCount);
        windows.merge(subWindowKey, 1, Integer::sum);
        return true;
    }
}
```

**工程实践：** Google Guava RateLimiter（令牌桶）、Sentinel（滑动窗口+多种模式）

---

### B343."设计一个分布式唯一ID生成器，对比雪花算法、UUID、号段模式三种方案的优劣？"

**问题分析：**
- 全局唯一、趋势递增、高性能、高可用
- 不同业务场景对有序性、长度、性能要求不同

**三种方案对比：**

| 维度 | 雪花算法 | UUID | 号段模式（Leaf） |
|------|---------|------|-----------------|
| 唯一性 | 强 | 强 | 强 |
| 有序性 | 趋势递增 | 无序 | 严格递增 |
| 长度 | 18位数字 | 36位字符串 | 可变 |
| 性能 | 本地生成，极高 | 本地生成，极高 | 需RPC取号段 |
| 依赖 | 时钟（时钟回拨问题） | 无 | DB |
| 可读性 | 好 | 差 | 好 |
| 扩展性 | 需分配workerId | 无需 | 步长可调 |

**雪花算法时钟回拨解决方案：**
1. 回拨<5ms：等待回拨时间
2. 回拨5ms~500ms：用备选workerId
3. 回拨>500ms：报错或用历史最大时间戳

**Leaf号段模式优化：**
```sql
-- 双Buffer：当前号段用到20%时异步加载下一个号段
UPDATE leaf_alloc SET max_id = max_id + step WHERE biz_tag = 'order'
-- 两个Buffer交替使用，保证取号不阻塞
```

**推荐选择：** 订单ID用号段模式（需严格递增），日志ID用雪花算法（高性能），traceId用UUID（无依赖）

---

### B344."电商系统的库存扣减需要保证不超卖，如何用Redis+Lua脚本实现原子扣减？并发场景如何处理？"

**问题分析：**
- 高并发秒杀场景，库存是共享资源
- 必须保证不超卖：库存不能为负
- 需要原子性：查库存-判断-扣减三步不可分割

**解决方案（Redis + Lua）：**
```lua
-- 扣减库存Lua脚本
local stock = tonumber(redis.call('GET', KEYS[1]))
if stock == nil then
    return -1  -- 库存Key不存在
end
if stock < tonumber(ARGV[1]) then
    return 0   -- 库存不足
end
redis.call('DECRBY', KEYS[1], ARGV[1])
return 1       -- 扣减成功
```

**完整方案：**
1. **预扣库存**：秒杀开始前将库存加载到Redis
2. **Lua原子扣减**：如上脚本
3. **异步落DB**：扣减成功发MQ消息，消费者异步更新DB库存
4. **对账补偿**：定时对比Redis和DB库存

**并发优化：**
- 分key并行：库存10000拆为10个Key，每个1000，随机选一个扣减
- 本地缓存预扣：每节点预分配少量库存，减少Redis访问

```java
// 分key方案
String key = "stock:" + skuId + ":" + (ThreadLocalRandom.current().nextInt(10));
Long result = redisTemplate.execute(luaScript, List.of(key), count);
```

---

### B345."社交平台的"附近的人"功能如何用GeoHash或R树实现？如何处理位置频繁更新？"

**问题分析：**
- 查询附近N米内的用户，支持距离排序
- 位置数据频繁变化（用户移动）
- 要求查询毫秒级，数据量百万级

**解决方案：**
1. **Redis GEO**：底层Sorted Set + GeoHash编码
   - GEOADD：更新位置 O(log N)
   - GEORADIUS：查询附近 O(N+log(N))，N为指定范围内元素数
2. **GeoHash编码**：将经纬度编码为字符串，前缀相同表示位置相近
   - 精度：6位GeoHash约±0.61km
   - 边界问题：附近可能跨越不同GeoHash区域 → 查询周围9个格子
3. **位置更新优化**：
   - 移动距离<50m不更新（避免无效写入）
   - 客户端5秒上报一次位置
   - 冷数据不维护实时位置（离线清除）

**Redis GEO实现：**
```bash
# 更新位置
GEOADD nearby:users 116.397128 39.916527 "user:12345"

# 查询附近5km的用户，按距离排序，限制100个
GEORADIUS nearby:users 116.397128 39.916527 5 km 
    WITHDIST ASC COUNT 100
```

**MySQL方案（小规模）：** `SPATIAL INDEX` + `ST_Distance_Sphere()` 函数

---

## 五、排查优化场景（5题）

### B346."线上接口偶现超时，排查发现某段代码时间复杂度从O(1)退化为O(n)，如何快速定位？"

**问题分析：**
- O(1)→O(n)退化的常见原因：
  1. HashMap并发退化为链表/红黑树重排
  2. ArrayList动态扩容copy
  3. 红黑树插入触发旋转
  4. 数据倾斜导致某条链过长
- 偶现性说明和特定数据/并发条件相关

**排查步骤：**
1. **APM定位**：SkyWalking/Prometheus定位慢方法
2. **日志分析**：慢请求的参数特征（某类Key特别多？）
3. **Arthas诊断**：`trace`命令追踪方法耗时，`watch`观察参数
4. **堆分析**：jmap dump → MAT分析HashMap桶分布

**常见退化场景与修复：**

| 场景 | 退化原因 | 修复方案 |
|------|---------|---------|
| HashMap并发 | 链表环形/O(n)遍历 | ConcurrentHashMap |
| HashSet contains | 哈希冲突严重 | 优化hashCode/换TreeSet |
| String.hashCode() | 短字符串哈希冲突 | 换MurmurHash |
| TreeMap | 逆序输入退化为链 | 换HashMap或洗牌 |
| 正则表达式 | 回溯爆炸 | 预编译+限制回溯 |

**Arthas排查示例：**
```bash
# 追踪方法耗时
trace com.example.Service processData '#cost > 100'

# 观察HashMap大小
watch com.example.Service processData '@java.lang.System@identityHashCode(target.cache)' -n 5
```

---

### B347."大数据量的分组聚合查询在应用层OOM，如何将聚合下推到数据库层？哪些场景必须应用层聚合？"

**问题分析：**
- 应用层将百万条数据拉到内存再groupingBy，内存爆炸
- 需要判断哪些聚合可以下推到SQL，哪些必须应用层处理

**可下推到SQL的聚合：**
- 简单聚合：SUM/COUNT/AVG/MAX/MIN
- GROUP BY + HAVING
- 多表JOIN + GROUP BY

**必须应用层聚合的场景：**
1. **跨数据源**：MySQL + ES + Redis的数据聚合
2. **复杂计算**：中位数、分位数、去重计数（精确）
3. **动态分组**：分组键需要运行时计算
4. **聚合+业务逻辑**：聚合结果需要关联业务规则

**下推示例：**
```sql
-- 应用层聚合（不好）
SELECT * FROM orders WHERE create_time > '2026-01-01'
-- Java: orders.stream().collect(groupingBy(...))

-- SQL下推聚合（好）
SELECT category, SUM(amount), COUNT(*) 
FROM orders WHERE create_time > '2026-01-01'
GROUP BY category
```

**跨数据源聚合方案：**
```java
// 方案1：各数据源先聚合，应用层合并
Map<String, Long> mysqlAgg = orderService.getCategorySum();
Map<String, Long> esAgg = searchService.getCategorySum();
Map<String, Long> merged = Stream.of(mysqlAgg, esAgg)
    .flatMap(m -> m.entrySet().stream())
    .collect(groupingBy(Map.Entry::getKey, summingLong(Map.Entry::getValue)));

// 方案2：数据同步到统一OLAP引擎（ClickHouse/Doris）
```

---

### B348."Redis集群中某个Slot的数据倾斜导致单节点负载过高，如何用一致性哈希+虚拟节点均衡？"

**问题分析：**
- Redis Cluster 16384个Slot，某些热点Key导致个别Slot请求量巨大
- 单节点CPU/内存/带宽过载
- 常见原因：大Key、热点Key、哈希不均匀

**解决方案：**
1. **热点Key打散**：`key:{hash_tag}` → `key:{1~N}`，分散到多个Slot
2. **本地缓存**：热点Key缓存到Caffeine，减少Redis访问
3. **读写分离**：从节点分担读请求
4. **虚拟节点思想**：虽然是Slot预分配，但可通过hash_tag实现逻辑上的虚拟节点

**热点Key发现：**
```bash
# redis-cli --hotkeys
# 或监控instantaneous_ops_per_sec
```

**hash_tag打散示例：**
```java
// 原来一个Key
String key = "product:hot_item_123";  // 全在一个Slot

// 打散为N个Key
String key = "product:hot_item_123:" + (userId % 10);  // 分散到10个Key

// 查询时聚合
List<String> keys = IntStream.range(0, 10)
    .mapToObj(i -> "product:hot_item_123:" + i)
    .collect(toList());
List<String> values = redis.mget(keys);
```

---

### B349."线上CPU突然飙到100%，排查发现是正则回溯导致，如何修复和预防？"

**问题分析：**
- 某些正则表达式在特定输入下会产生指数级回溯
- 典型模式：`(a+)+` 匹配 `aaaaab`，回溯次数O(2^n)
- 单线程CPU 100%，如果线程池中多个线程同时执行则全CPU打满

**常见回溯陷阱：**
1. 嵌套量词：`(a+)+`、`(a*)*`
2. 交替重叠：`(a|a)+`
3. 非贪婪+固定后缀：`.*?end` 在无end时回溯到头部

**修复方案：**
1. **预编译正则**：`Pattern.compile()`，避免重复编译
2. **限制回溯**：设置匹配时间/长度限制
3. **优化正则**：
   - 用占有量词`++`、`*+`、`?+`（不回溯）
   - 用原子组`(?>...)`（整体匹配，不回溯）
   - 简化嵌套量词
4. **替代方案**：用String方法（indexOf/contains）处理简单匹配

**修复代码：**
```java
// 问题正则
Pattern p1 = Pattern.compile("(a+)+b");

// 方案1：占有量词（不回溯）
Pattern p2 = Pattern.compile("(a++)b");

// 方案2：限制输入长度
if (input.length() > 1000) throw new IllegalArgumentException();

// 方案3：设置匹配超时（Java 21+）
// 或者用Re2j（Google的非回溯正则引擎）
```

---

### B350."百万级数据的排序操作在应用层频繁GC，如何用外部排序（归并排序）降低内存占用？"

**问题分析：**
- 百万条数据全量排序需一次性加载到内存
- 每条数据1KB → 1GB内存，排序过程需要额外1GB
- 频繁Full GC，甚至OOM

**解决方案（外部归并排序）：**
1. **分批读取**：每次读10万条，排序后写入临时文件
2. **多路归并**：用小顶堆合并多个有序临时文件
3. **流式输出**：归并结果直接写入输出文件，不占内存

**实现框架：**
```java
void externalSort(File input, File output, int batchSize) {
    // Phase 1: 分批排序
    List<File> tempFiles = new ArrayList<>();
    try (BufferedReader reader = new BufferedReader(new FileReader(input))) {
        List<String> batch = new ArrayList<>(batchSize);
        String line;
        while ((line = reader.readLine()) != null) {
            batch.add(line);
            if (batch.size() == batchSize) {
                batch.sort(null);
                File temp = writeTempFile(batch);
                tempFiles.add(temp);
                batch.clear();
            }
        }
        if (!batch.isEmpty()) {
            batch.sort(null);
            tempFiles.add(writeTempFile(batch));
        }
    }
    
    // Phase 2: 多路归并
    mergeSortedFiles(tempFiles, output);
}

void mergeSortedFiles(List<File> files, File output) {
    // 小顶堆，每文件读一行，取最小写出，再从该文件读下一行
    PriorityQueue<FileLine> heap = new PriorityQueue<>();
    // ... 多路归并逻辑
}
```

**优化：** 归并路数不宜太多（>100路效率下降），可用多级归并（2级→4级→...）

---

## 六、架构演进场景（4题）

### B351."单体应用的数据访问层大量使用HashMap做缓存，微服务拆分后需要替换为分布式缓存，如何平滑迁移？"

**问题分析：**
- 单体时代用HashMap做本地缓存，简单高效
- 微服务拆分后，各实例本地缓存不一致
- 直接全替换为Redis可能引入性能回退（网络延迟）

**迁移策略：**
1. **双读模式**：新代码先读Redis，miss后读HashMap，结果回写Redis
2. **本地缓存保留**：L1=Caffeine(10s TTL) + L2=Redis(5min TTL)
3. **灰度切换**：按接口/用户维度逐步切换
4. **回滚预案**：开关控制，出问题秒切回HashMap

**多级缓存架构：**
```
请求 → Caffeine(10s, 本地)
    → miss → Redis(5min, 分布式)
    → miss → DB
    → 回写Caffeine + Redis
```

**代码改造：**
```java
// 旧代码
private Map<String, Data> cache = new HashMap<>();

// 新代码（多级缓存）
private Cache<String, Data> localCache = Caffeine.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(10, SECONDS)
    .build();

public Data get(String key) {
    // L1: Caffeine
    Data data = localCache.getIfPresent(key);
    if (data != null) return data;
    
    // L2: Redis
    data = redis.get(key);
    if (data != null) {
        localCache.put(key, data);
        return data;
    }
    
    // L3: DB
    data = db.query(key);
    if (data != null) {
        localCache.put(key, data);
        redis.set(key, data, 5, MINUTES);
    }
    return data;
}
```

---

### B352."系统从单机扩展到集群后，原来用synchronized实现的库存扣减失效了，如何用分布式锁替代？"

**问题分析：**
- 单机synchronized只对同一JVM内有效
- 集群多实例，synchronized无法跨JVM互斥
- 导致超卖：两个实例同时读到库存=1，都扣减成功

**分布式锁方案对比：**

| 方案 | 性能 | 可靠性 | 适用场景 |
|------|------|--------|---------|
| Redis SETNX | 高 | 中（主从切换丢锁） | 大部分场景 |
| Redisson | 高 | 中（看门狗续期） | 需要续期/可重入 |
| ZooKeeper | 低 | 高 | 强一致性要求 |
| MySQL锁 | 最低 | 高 | 低并发 |

**推荐方案（Redisson）：**
```java
RLock lock = redissonClient.getLock("stock:" + skuId);
try {
    if (lock.tryLock(3, 10, SECONDS)) {
        // 扣减库存
        int stock = stockDao.getStock(skuId);
        if (stock >= count) {
            stockDao.decreaseStock(skuId, count);
            return true;
        }
        return false;
    }
    throw new BusyException("系统繁忙，请重试");
} finally {
    if (lock.isHeldByCurrentThread()) {
        lock.unlock();
    }
}
```

**注意：** 锁粒度要细（sku级别而非全局），避免不必要串行化

---

### B353."数据量增长到10亿级，单表无法支撑查询，需要按用户ID分库分表，如何保证跨分片的排序和分页？"

**问题分析：**
- 按user_id分片后，全局排序/分页需要跨分片聚合
- 简单方案：各分片查TOP N → 内存归并排序 → 取最终分页
- 深度分页时效率极低

**解决方案：**
1. **浅分页（page < 100）**：各分片取page条，内存排序取TOP page
2. **深度分页**：禁止深度分页，改用游标翻页（after_id）
3. **全局排序**：ES影子表，写入时同步到ES，排序走ES
4. **禁止跨分片ORDER BY非分片键**：如必须则走ES

**分页实现：**
```java
// 浅分页：各分片取limit条，内存归并
List<Order> result = new ArrayList<>();
for (String shard : shards) {
    List<Order> shardResult = orderDao.getByPage(shard, userId, page, size);
    result.addAll(shardResult);
}
result.sort(comparing(Order::getCreateTime).reversed());
return result.subList(0, Math.min(size, result.size()));
```

**ES方案：**
```java
// 写入时Canal同步到ES
// 查询时走ES
SearchResponse response = client.prepareSearch("orders")
    .setQuery(boolQuery().must(termQuery("user_id", userId)))
    .addSort("create_time", SortOrder.DESC)
    .setFrom(page * size).setSize(size)
    .execute().actionGet();
```

---

### B354."系统从HTTP短连接升级到WebSocket长连接后，出现大量CLOSE_WAIT，如何从TCP状态机角度分析？"

**问题分析：**
- CLOSE_WAIT表示对端发了FIN（关闭连接），本端尚未关闭
- 大量CLOSE_WAIT说明：应用层没有正确关闭Socket
- 累积过多会耗尽文件描述符

**TCP状态机分析：**
```
对端调用close() → 发FIN → 本端收到FIN → 回ACK → 进入CLOSE_WAIT
→ 本端应该调用close() → 发FIN → 进入LAST_ACK → 对端回ACK → CLOSED
```

**CLOSE_WAIT堆积原因：**
1. 业务代码没有在finally中关闭Socket/Connection
2. WebSocket的@OnClose没有正确触发
3. 连接空闲超时后，对端关闭但本端未感知
4. 线程池满，关闭连接的任务排队

**排查步骤：**
```bash
# 统计各TCP状态数量
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

# 查看CLOSE_WAIT连接
netstat -n | grep CLOSE_WAIT | head -20
```

**修复方案：**
```java
@ServerEndpoint("/ws")
public class WebSocketHandler {
    @OnClose
    public void onClose(Session session) {
        // 确保清理资源
        sessionCache.remove(session.getId());
        try { session.close(); } catch (Exception ignored) {}
    }
    
    // 添加空闲检测
    @OnMessage
    public void onMessage(String msg, Session session) {
        lastActiveTime.put(session.getId(), System.currentTimeMillis());
    }
}

// 定时扫描清理僵尸连接
scheduledExecutor.scheduleAtFixedRate(() -> {
    sessionCache.forEach((id, session) -> {
        if (System.currentTimeMillis() - lastActiveTime.get(id) > 300_000) {
            sessionCache.remove(id);
            session.close();
        }
    });
}, 0, 1, MINUTES);
```

---

## 知识点覆盖总结

| 子领域 | 题号 | 数量 |
|--------|------|------|
| 高并发与缓存场景 | B325-B330 | 6 |
| 数据一致性场景 | B331-B335 | 5 |
| 系统稳定性场景 | B336-B340 | 5 |
| 业务设计场景 | B341-B345 | 5 |
| 排查优化场景 | B346-B350 | 5 |
| 架构演进场景 | B351-B354 | 4 |
| **合计** | | **30** |

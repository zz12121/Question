# Java后端面试题库（7年经验版）

> 本题库采用**场景化 + 追问递进 + 原理到优化**的出题模式，每道题都包含问题识别、原理追问、方案优化三个层次。

---

## 一、Java基础 & 核心

### 题目1：String拼接性能问题

> 你在代码中大量使用 `String s = str1 + str2` 进行字符串拼接，用于拼接日志或者报文组装。

**问题：**
1. 这个方案在循环场景下有什么性能问题？
2. 如果在高频接口中这样做，JVM 会有什么表现？
3. 你会如何优化？

<details>
<summary>考察点提示</summary>

- StringBuilder vs StringBuffer
- 字符串常量池
- JVM StringTable
- 逃逸分析
</details>

---

### 题目2：List去重方案

> 你有一个 `List<User>` 需要去重，User 有 id、name、age 三个字段。你考虑用 `HashSet<User>` 来去重。

**问题：**
1. 这个方案能正确去重吗？为什么？
2. 如果 User 对象有上百万个，可能有什么性能问题？
3. 你会如何改进？

<details>
<summary>考察点提示</summary>

- hashCode() 和 equals()
- HashSet 底层实现
- 哈希冲突
- 内存占用
</details>

---

### 题目3：日期处理陷阱

> 你在系统中使用 `SimpleDateFormat` 来解析日期字符串，为了方便把它定义为 `static` 变量。

```java
private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
```

**问题：**
1. 这个写法在单线程和多线程环境下分别有什么问题？
2. 如果在高并发接口中使用，会出现什么现象？
3. 你有哪些解决方案？

<details>
<summary>考察点提示</summary>

- SimpleDateFormat 非线程安全
- 内部 Calendar 共享
- ThreadLocal 解决方案
- Java8 DateTimeFormatter
</details>

---

### 题目4：ConcurrentHashMap统计PV

> 你有一个高并发的电商系统，需要统计商品的实时访问量（PV）。你考虑用 `ConcurrentHashMap<String, AtomicLong>` 来实现，key 是商品 ID，value 是访问次数。

**问题：**
1. 这个方案在并发场景下有什么问题？
2. 如果访问量非常大，这个结构可能会引发什么 JVM 层面的问题？
3. 你会如何优化这个方案？

<details>
<summary>考察点提示</summary>

- LongAdder vs AtomicLong
- 伪共享问题
- @Contended 注解
- HighArray 带来的 GC 压力
</details>

---

### 题目5：ArrayList扩容机制

> 你需要在内存中缓存一批数据，使用 `ArrayList` 来存储，大概有 10 万条记录。

**问题：**
1. ArrayList 的扩容机制是什么？每次扩容增长多少？
2. 如果你提前知道数据量是 10 万条，有办法避免扩容吗？
3. 如果数据量突然增长到 1000 万条，JVM 会发生什么？
4. 在内存敏感的场景下，你会如何优化？

<details>
<summary>考察点提示</summary>

- 扩容因子 1.5
- Arrays.copyOf
- OOM 风险
- 预估容量初始化
</details>

---

### 题目6：HashMap死循环

> 你的同事告诉你，Java 8 已经修复了 HashMap 在并发下的死循环问题，所以可以安全地在并发场景使用了。

**问题：**
1. Java 7 和 Java 8 的 HashMap 在并发下的问题有什么区别？
2. Java 8 真的解决了并发安全问题吗？还会有什么问题？
3. 你知道 ConcurrentHashMap 在 Java 7 和 Java 8 的实现有什么区别吗？
4. 如果要在并发场景下使用 map，你会怎么选择？

<details>
<summary>考察点提示</summary>

- Java 7 头插法死循环
- Java 8 尾插法
- 数据覆盖问题
- ConcurrentHashMap 分段锁/无锁
</details>

---

### 题目7：泛型类型擦除

> 你发现一个奇怪的问题，这两行代码都能编译通过：

```java
List<String> list1 = new ArrayList<String>();
List<Integer> list2 = new ArrayList<Integer>();
System.out.println(list1.getClass() == list2.getClass()); // true
```

**问题：**
1. 为什么 `List<String>` 和 `List<Integer>` 的 class 是相同的？
2. 泛型擦除后，原本的类型信息去哪了？
3. 既然泛型被擦除了，为什么 `list.add("123")` 编译报错？
4. 泛型擦除会带来什么坑？你在项目中遇到过吗？

<details>
<summary>考察点提示</summary>

- 类型擦除机制
- 桥接方法
- 泛型限制（不能 new T，不能 new T[]）
- RPC 序列化泛型问题
</details>

---

## 二、JVM 虚拟机

### 题目8：线上OOM排查

> 你的 Java 应用部署在 Linux 服务器上，突然收到报警说应用不可用了。你登录服务器后，发现进程还在，但 CPU 使用率很低，接口全部超时。

**问题：**
1. 你首先会用什么命令确认是否发生了 OOM？
2. 如果确认是 OOM，你会如何定位是哪种类型的 OOM（堆/栈/元空间）？
3. 如何分析 OOM 的原因？你会使用哪些工具？
4. 如果 OOM 是由第三方 SDK 引起的，但源码无法修改，你怎么办？

<details>
<summary>考察点提示</summary>

- jstat/jinfo/jmap 命令
- OOM 原因分类
- MAT/Arthas 分析
- -XX:+HeapDumpOnOutOfMemoryError
</details>

---

### 题目9：GC频繁导致接口抖动

> 你的接口 P99 延迟突然升高，从 50ms 涨到了 500ms。观察监控发现 Young GC 频率从每分钟 2 次变成了每分钟 20 次。

**问题：**
1. GC 频繁为什么会影响接口延迟？
2. 如果每次 Young GC 后很多对象进入了老年代，会导致什么问题？
3. 你会如何排查是哪个环节创建了太多对象？
4. 如何从根本上解决这个问题？

<details>
<summary>考察点提示</summary>

- Stop The World
- 对象分配速率
- Survivor 区
- 逃逸分析、栈上分配
</details>

---

### 题目10：类加载导致的线上问题

> 你的应用在重启后，有时候需要预热很长时间才能达到正常性能，有时候甚至会报错 "ClassNotFound"。

**问题：**
1. 这种"预热"现象背后是什么原理？
2. JIT 编译优化在这个过程中起了什么作用？
3. 如何让系统启动后立即达到高性能？
4. 如果使用 GraalVM Native Image，这个问题会有什么不同？

<details>
<summary>考察点提示</summary>

- JIT 编译
- 热点代码检测
- 类加载延迟
- AOT vs JIT
</details>

---

### 题目11：Metaspace 持续增长

> 你的应用运行一段时间后，Metaspace 的使用量从 100MB 持续增长到了 500MB，接近了 `-XX:MaxMetaspaceSize=512m` 的限制。

**问题：**
1. Metaspace 增长通常是由什么引起的？
2. 你怀疑是动态代理引起的，应该如何验证？
3. 如果确认是动态代理导致的，应该如何优化？
4. 除了动态代理，还有哪些场景会导致 Metaspace 增长？

<details>
<summary>考察点提示</summary>

- Metaspace vs Heap Space
- 动态代理/字节码生成
- CGLIB
- Spring/Gson 反射
</details>

---

### 题目12：Full GC 无法回收

> 你的应用 Full GC 频率很高，几乎每分钟一次，但每次回收的内存很少。更奇怪的是，老年代使用率一直在 90% 以上。

**问题：**
1. 为什么 Full GC 回收不了内存？可能的原因是什么？
2. 如果你怀疑是内存泄漏，如何定位是哪个对象泄漏了？
3. 你知道哪些常见写法会导致内存泄漏？（至少说出 3 种）
4. 如果需要打印 GC 日志来排查，JVM 参数应该怎么配置？

<details>
<summary>考察点提示</summary>

- 内存泄漏排查
- WeakReference vs SoftReference
- 静态集合持有对象
- 资源未关闭（连接/流）
</details>

---

## 三、并发编程

### 题目13：订单号生成方案

> 你的系统需要为每个订单生成唯一ID，你考虑用 `AtomicLong` 来实现：

```java
private static final AtomicLong counter = new AtomicLong(0);
public static long generateId() {
    return System.currentTimeMillis() * 1000 + counter.incrementAndGet();
}
```

**问题：**
1. 这个方案在单机环境下有什么问题？
2. 如果你的系统部署了多个实例，会出现什么情况？
3. 在高并发场景下（QPS 10万+），这个方案会成为瓶颈吗？为什么？
4. 你有什么改进方案？

<details>
<summary>考察点提示</summary>

- 原子性保证
- 时钟回拨
- 分布式ID
- 雪花算法
</details>

---

### 题目14：缓存一致性

> 你有一个订单服务，在查询订单时会先查缓存，如果缓存没有再查数据库：

```java
public Order getOrder(String id) {
    Order order = cache.get(id);
    if (order == null) {
        order = db.findById(id);
        cache.put(id, order);
    }
    return order;
}
```

**问题：**
1. 这个方案在并发场景下有什么问题？（考虑两个线程同时查询）
2. 如果用户修改了订单，这个流程能保证数据一致吗？
3. 你会如何优化这个方案？
4. 如果既要保证性能又要保证一致性，你的方案是什么？

<details>
<summary>考察点提示</summary>

- 缓存穿透/击穿
- Double Check Locking
- 本地缓存
- Cache Aside 模式
</details>

---

### 题目15：连接池配置

> 你的数据库连接池使用的是 HikariCP，默认配置最大连接数是 10。现在系统流量上涨，你把最大连接数调大到了 50。

**问题：**
1. 连接数调大后，可能会引发什么新问题？
2. 如果数据库服务器连接数有限，你们的应用连接数配置过大会有什么问题？
3. 你会如何确定最优的连接数配置？
4. 如果应用是 Spring Boot，你了解哪些关键参数需要调优？

<details>
<summary>考察点提示</summary>

- 数据库连接资源
- 线程池饥饿
- 连接数计算公式
- connectionTimeout/maximumPoolSize
</details>

---

### 题目16：ThreadLocal内存泄漏

> 你的系统使用 ThreadLocal 来存储用户信息，在线程池环境中使用后发现，偶尔会出现用户信息错乱的问题。

**问题：**
1. ThreadLocal 为什么会造成内存泄漏？
2. 为什么在线程池环境中更容易出现问题？
3. 使用 ThreadLocal 时，正确的做法是什么？
4. 你知道 TransmittableThreadLocal 吗？它解决了什么问题？

<details>
<summary>考察点提示</summary>

- ThreadLocalMap 弱引用
-  Entry extends WeakReference<ThreadLocal<?>>
- 线程复用时未清理
- TtlRunnable/TtlCallable
</details>

---

### 题目17：CompletableFuture并发调用

> 你的接口需要调用 5 个下游服务获取数据后再聚合返回，为了提升性能，你使用 CompletableFuture 并行调用：

```java
CompletableFuture.allOf(
    cf1, cf2, cf3, cf4, cf5
).join();
```

**问题：**
1. 如果其中 1 个服务超时或异常了，会发生什么？
2. 如何设置超时时间？超时后如何处理？
3. 如果这 5 个服务的重要性不同，你会如何设计？
4. 你知道 ForkJoinPool 和 CommonPool 的区别吗？

<details>
<summary>考察点提示</summary>

- 异常传播
- exceptionally/handle
- 超时控制
- 线程池隔离
</details>

---

### 题目18：死锁排查

> 线上突然出现接口超时，日志没有任何异常信息，但线程 dump 显示大量线程处于 BLOCKED 状态。

**问题：**
1. 你怀疑是死锁，如何确认？应该 dump 哪些信息？
2. 分析线程 dump 时，你会关注哪些关键信息来定位死锁？
3. 如何避免在代码中写出死锁？有哪些编码规范？
4. 如果死锁是因为第三方框架导致的，有什么临时解决方案？

<details>
<summary>考察点提示</summary>

- jstack -l pid
- 等待关系分析
- 锁顺序、tryLock
- dump 分析工具
</details>

---

## 四、数据库（MySQL）

### 题目19：分页查询性能

> 你的系统有一个订单列表接口，需要支持分页查询：

```sql
SELECT * FROM orders WHERE user_id = ? ORDER BY create_time DESC LIMIT ?, ?
```

**问题：**
1. 当 offset 很大时（比如 offset=1000000），这个查询会有什么性能问题？
2. 如果这个表有 1 亿条数据，查询会走索引吗？为什么？
3. 如何优化这种大 offset 分页查询？
4. 如果需要支持"跳页"功能（用户可以直接跳到第N页），你会如何设计？

<details>
<summary>考察点提示</summary>

- 深度分页问题
- 索引覆盖
- ID回翻 vs 游标分页
- 延迟关联
</details>

---

### 题目20：唯一索引冲突

> 你的系统使用用户手机号作为唯一索引来注册用户：

```sql
CREATE UNIQUE INDEX idx_phone ON users(phone);
```

**问题：**
1. 如果两个请求同时插入相同手机号的用户，会发生什么？
2. MySQL 是如何检测并阻止重复的？
3. 这种冲突在高并发注册场景下会带来什么性能问题？
4. 你有什么优化方案可以减少冲突检测的开销？

<details>
<summary>考察点提示</summary>

- 唯一索引实现
- 死锁场景
- Duplicate Key Error
- 预检查 + insert
</details>

---

### 题目21：批量插入优化

> 你需要每天凌晨导入一批数据（大约 100 万条）到 MySQL 数据库，目前使用的是普通的 INSERT 语句一条一条插入。

**问题：**
1. 这种方式导入 100 万条数据预计需要多长时间？主要耗时在哪里？
2. 你知道有哪些方法可以加速批量插入吗？
3. 如果表有索引，导入时应该先删索引还是后建索引？为什么？
4. 导入过程中如果数据库主从复制开启，可能会造成什么问题？

<details>
<summary>考察点提示</summary>

- 批量 insert 语法
- 事务控制
- 索引维护代价
- binlog 复制延迟
</details>

---

### 题目22：事务隔离级别选择

> 你的系统需要转账功能：

```sql
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
```

**问题：**
1. 如果不加事务控制，可能会出现什么问题？
2. 如果在并发转账场景下，不同事物隔离级别会带来什么影响？
3. `REPEATABLE READ` 下会出现幻读吗？幻读和不可重复读的区别是什么？
4. 从性能角度，你会如何选择隔离级别？说说理由。

<details>
<summary>考察点提示</summary>

- 脏读/不可重复读/幻读
- MVCC 原理
- Next-Key Lock
- 隔离级别与性能权衡
</details>

---

### 题目23：索引失效场景

> 你的查询是这样的：

```sql
SELECT * FROM orders WHERE YEAR(create_time) = 2024 AND status = 1;
```

**问题：**
1. 这个查询能走索引吗？哪个索引会失效？
2. 如果必须保留 `YEAR()` 函数，有哪些优化方案？
3. 常见的索引失效场景还有哪些？（至少说出 5 种）
4. 如何判断查询是否走了索引？你会使用什么工具？

<details>
<summary>考察点提示</summary>

- 函数导致索引失效
- 范围查询后的列
- 前导模糊查询
- OR 条件
- EXPLAIN 分析
</details>

---

### 题目24：SQL死锁分析

> 你的系统在高峰期出现了死锁报警，日志显示：

```
Deadlock found when trying to get lock; try restarting transaction
```

**问题：**
1. 如果让你分析这个死锁，你会获取哪些信息？
2. 从业务角度，为什么转账场景容易出现死锁？
3. 如何设计可以避免这类死锁？（从 SQL 顺序、加锁粒度等角度）
4. 如果死锁无法完全避免，你的系统应该怎么处理？

<details>
<summary>考察点提示</summary>

- SHOW ENGINE INNODB STATUS
- gap lock/next-key lock
- 锁等待图
- 重试机制
</details>

---

## 五、Redis

### 题目25：热key问题

> 你的电商系统有一个商品详情页，商品ID为 `product:12345` 的商品突然上了热搜，大量用户同时访问这个商品页。

**问题：**
1. 这个热 key 会带来什么问题？
2. Redis 单节点处理热 key 请求时，CPU/网络可能会出现什么情况？
3. 你有什么方案可以解决热 key 问题？
4. 如果商品信息更新了，你如何保证缓存一致性？

<details>
<summary>考察点提示</summary>

- 热点 key 探测
- Redis 单线程瓶颈
- 本地缓存 + Redis 二级缓存
- 缓存更新策略
</details>

---

### 题目26：分布式锁误用

> 你的系统需要在秒杀场景下扣减库存，你使用了 Redis 分布式锁：

```java
Boolean success = redis.setnx("stock:lock", "1");
if (success) {
    // 扣减库存
    reduceStock();
    redis.del("stock:lock");
}
```

**问题：**
1. 这个实现有什么问题？（考虑异常情况、超时、误删）
2. 如果服务在持有锁期间进程崩溃了，会发生什么？
3. 你知道 RedLock 算法吗？它解决了什么问题？又有什么争议？
4. 在扣库存场景下，分布式锁真的是最优解吗？有没有更好的方案？

<details>
<summary>考察点提示</summary>

- setnx + expire 原子性
- 锁续期机制
- RedLock 争议
- Redis 原子命令扣库存
</details>

---

### 题目27：缓存过期策略

> 你的系统使用 Redis 做缓存，设置了 30 分钟过期时间。但运营反馈用户经常抱怨"刚看过的数据突然消失了"。

**问题：**
1. 为什么会发生这种现象？用户的操作触发了什么？
2. Redis 的过期删除策略是什么？你了解惰性删除和定期删除吗？
3. 如果大量 key 同一时间过期，会引发什么问题？
4. 你会如何设计缓存过期策略来平衡一致性和用户体验？

<details>
<summary>考察点提示</summary>

- 缓存击穿
- 过期删除策略
- 缓存雪崩
- 随机 TTL + 永不过期
</details>

---

### 题目28：Redis数据持久化

> 你的 Redis 开启了 RDB 和 AOF 两种持久化方式，服务器突然断电了。

**问题：**
1. 重启后，Redis 会加载哪种持久化文件？优先级是什么？
2. RDB 和 AOF 各自的优缺点是什么？
3. 如果 AOF 配置为 `appendfsync = always`，会有什么性能问题？
4. 你通常如何选择 RDB 和 AOF 的配置策略？

<details>
<summary>考察点提示</summary>

- RDB 快照
- AOF 日志
- appendfsync 配置
- 混合持久化
</details>

---

### 题目29：Redis集群脑裂

> 你的 Redis Sentinel 集群出现了网络分区，一部分节点和 master 失联了，另一个从节点被选举为新的 master。当网络恢复后，原来的 master 变成了从节点。

**问题：**
1. 脑裂期间，客户端还能正常写入吗？写入的数据会怎样？
2. 脑裂恢复后，如何处理两个 master 的数据冲突？
3. 如何配置 Redis 来减少脑裂带来的数据丢失风险？
4. 你知道 Redis Cluster 和 Sentinel 在脑裂处理上有什么区别吗？

<details>
<summary>考察点提示</summary>

- min-slaves-to-write
- 脑裂数据丢失
- Raft 协议
- Redis Cluster 槽迁移
</details>

---

### 题目30：Redis vs Memcached

> 你的团队在选择缓存中间件，有人认为 Memcached 更简单，性能也更好。

**问题：**
1. Redis 和 Memcached 在实现上有什么区别？
2. 为什么大多数场景下大家更倾向于选择 Redis？
3. Redis 支持的数据类型，Memcached 支持吗？这些差异在实际业务中有什么影响？
4. 在什么场景下，你会选择 Memcached 而不是 Redis？

<details>
<summary>考察点提示</summary>

- 多线程 vs 单线程
- 数据结构支持
- 持久化能力
- 内存管理
</details>

---

## 六、Spring & 框架

### 题目31：事务失效场景

> 你的代码是这样的：

```java
@Service
public class OrderService {
    @Transactional
    public void createOrder(Order order) {
        orderMapper.insert(order);
        updateUserPoints(order.getUserId());
    }
    
    private void updateUserPoints(Long userId) {
        throw new RuntimeException("积分更新失败");
    }
}
```

**问题：**
1. 当 `updateUserPoints` 抛出异常时，订单会回滚吗？为什么？
2. 如果 `createOrder` 变成了 public void，被其他 Service 调用时，事务会生效吗？
3. 如果 `orderMapper.insert` 后面还有一行 `int i = 1/0`，事务会回滚吗？
4. 你知道 `@Transactional` 还有什么常见失效场景吗？

<details>
<summary>考察点提示</summary>

- private 方法事务不生效
- 自调用问题
- 异常被 catch 吞掉
- 事务传播行为
</details>

---

### 题目32：循环依赖问题

> 你的 Spring Boot 项目启动失败了，日志显示：

```
BeanCurrentlyInCreationException: Error creating bean ... 
requested a bean of dependency 'B' while it is in the process of being created
```

**问题：**
1. 这是什么错误？为什么会发生？
2. Spring 是如何解决单例 Bean 的循环依赖的？三级缓存是什么？
3. 为什么构造函数注入更容易触发循环依赖问题？
4. 如果确实需要 A 依赖 B、B 依赖 A，有哪些解决方案？

<details>
<summary>考察点提示</summary>

- Spring Bean 创建过程
- 三级缓存（singletonObjects、earlySingletonObjects、singletonFactories）
- 构造函数注入 vs setter 注入
- @Lazy 注解
</details>

---

### 题目33：配置中心动态刷新

> 你的系统使用 Nacos 作为配置中心，有一个功能开关：

```yaml
feature:
  new-payment: false
```

**问题：**
1. 改了配置后，Spring Boot 应用能感知到变化吗？需要什么条件？
2. 如果你的代码是 `if (feature.newPayment)` 这种方式，有什么问题？
3. 你知道 `@RefreshScope` 的实现原理吗？
4. 如果开关变化后需要执行一些初始化逻辑，你如何实现？

<details>
<summary>考察点提示</summary>

- @RefreshScope 原理
- 配置绑定
- @ConfigurationProperties
- @PostConstruct vs ApplicationRunner
</details>

---

### 题目34：Spring Boot启动优化

> 你的应用启动时间很长，需要 3 分钟才能完全启动，其中大部分时间花在 Bean 初始化上。

**问题：**
1. 如何定位是哪个 Bean 初始化耗时最长？
2. 有哪些常见的 Bean 初始化优化方案？
3. Spring Boot 2.x 和 3.x 在启动性能上有什么改进？
4. 如果你的应用必须依赖很多 Bean，但又不都在启动时使用，怎么办？

<details>
<summary>考察点提示</summary>

- Bean 初始化耗时分析
- @Lazy 初始化
- BeanPostProcessor 优化
- SpringFactory 优化
</details>

---

### 题目35：Feign调用超时

> 你的微服务使用 Feign 进行调用，偶尔会出现超时错误：

```
ReadTimeout: Read timed out. timeout ...
```

**问题：**
1. 超时是如何配置的？你了解 Ribbon 和 OpenFeign 的超时配置区别吗？
2. 如果是第一次调用就超时，可能是什么原因？
3. 如何实现重试机制？重试在什么场景下是危险的？
4. 如何设计降级方案来保证用户体验？

<details>
<summary>考察点提示</summary>

- 超时配置项
- Hystrix/Sentinel 降级
- 重试策略
- FallbackFactory
</details>

---

## 七、分布式 & 微服务

### 题目36：接口幂等性

> 你的支付系统收到了两个相同的支付请求，订单号都是 `ORDER123`，金额都是 100 元。用户可能是因为网络超时，点击了两次"支付"按钮。

**问题：**
1. 如果不做任何处理，会发生什么？
2. 你有哪些方案来保证接口幂等性？各有什么优缺点？
3. 如果用 Redis 做幂等控制，具体流程是什么？
4. 如果用户支付成功后，异步回调也发送了两次，你的系统能正确处理吗？

<details>
<summary>考察点提示</summary>

- 幂等性保证
- Token 机制
- 唯一索引
- 分布式锁
</details>

---

### 题目37：服务雪崩

> 你的微服务架构是这样的：A → B → C，C 是底层服务。突然 C 服务变慢了，平均响应时间从 10ms 变成了 5s。

**问题：**
1. 在这种情况下，A 和 B 会发生什么？
2. 如果 B 也开始变慢，整个系统会怎样？
3. 你知道有哪些方案可以防止雪崩吗？（至少说出 3 种）
4. Sentinel 和 Hystrix 在熔断策略上有什么不同？

<details>
<summary>考察点提示</summary>

- 线程池耗尽
- 熔断器模式
- 限流策略
- Sentinel vs Hystrix
</details>

---

### 题目38：分布式Session

> 你的系统部署了多个实例，用户登录后 session 存储在实例本地。后来用户反馈"登录后偶尔需要重新登录"。

**问题：**
1. 为什么会发生这种现象？
2. 如果用户第一次请求被路由到实例 A，第二次请求被路由到实例 B，会发生什么？
3. 你有哪些方案可以解决这个问题？
4. 如果使用 JWT 代替 Session，有什么优势和劣势？

<details>
<summary>考察点提示</summary>

- 粘性 session
- Session 共享
- JWT 原理
- Token vs Session
</details>

---

### 题目39：分布式ID冲突

> 你的系统分库分表后，需要为每条记录生成唯一ID。你使用 UUID 作为主键。

**问题：**
1. UUID 作为主键有什么性能问题？
2. 如果用自增ID，在分库分表场景下有什么问题？
3. 你知道哪些分布式ID生成方案？它们各自的优缺点是什么？
4. 雪花算法时钟回拨问题如何处理？

<details>
<summary>考察点提示</summary>

- UUID 性能问题
- 有序性 vs 随机性
- 雪花算法
- 百度UidGenerator
</details>

---

### 题目40：分布式事务一致性

> 你的系统在用户下单时需要同时：1）创建订单 2）扣减库存 3）扣减余额。你使用了 TCC 模式。

**问题：**
1. TCC 的 Try-Confirm-Cancel 三个阶段分别负责什么？
2. 如果在 Confirm 阶段部分成功、部分失败，应该如何处理？
3. TCC 和 Saga 模式有什么区别？各自适合什么场景？
4. 如果网络抖动导致 Confirm 超时但实际执行成功了，会发生什么？

<details>
<summary>考察点提示</summary>

- TCC 三阶段
- 空回滚问题
- 幂等性保证
- Saga 补偿模式
</details>

---

## 八、架构设计

### 题目41：短链接系统设计

> 你需要设计一个短链接服务，把长 URL 转换为短 URL，例如：`https://xxx.com/aBcDeF`。

**问题：**
1. 如何生成短链接？有什么算法？
2. 如果每天有 1 亿次跳转请求，如何设计高并发架构？
3. 短链接和长 URL 的映射关系如何存储？如何保证查询性能？
4. 如果需要统计每个短链接的点击量，你会如何设计？

<details>
<summary>考察点提示</summary>

- 哈希算法 / 自增 ID
- 缓存设计
- 数据库索引
- 异步计数
</details>

---

### 题目42：延迟任务系统

> 你的系统需要在用户下单后 30 分钟，如果用户未支付，自动取消订单。

**问题：**
1. 你有哪些方案可以实现延迟任务？各有什么优缺点？
2. 如果用 Redis 的 ZSet 实现，如何保证定时触发的准确性？
3. 如果系统重启了，未执行的任务会丢失吗？如何保证可靠性？
4. 如果订单量很大（每秒 1 万单），你的系统能支撑吗？

<details>
<summary>考察点提示</summary>

- 定时任务 vs 消息队列
- Redis ZSet 延迟队列
- RocketMQ 延迟消息
- 任务持久化
</details>

---

### 题目43：抢红包系统

> 你需要设计一个春节红包系统，支持用户抢红包。假设红包总金额 100 元，红包数量 10 个。

**问题：**
1. 如何保证红包不会超发？（不能出现第 11 个人抢到红包）
2. 如何设计数据库表结构来记录红包领取记录？
3. 如果同时有 10 万人抢同一个红包，如何保证系统不崩溃？
4. 如何防止一个人抢多个红包？（限制一个人只能抢一次）

<details>
<summary>考察点提示</summary>

- 原子性扣减
- Redis 原子命令
- 数据库乐观锁
- 限流防刷
</details>

---

### 题目44：秒杀系统设计

> 你的电商平台要在双十一做秒杀活动，1000台特价手机，库存有限。

**问题：**
1. 如何防止超卖？（不能卖出超过1000台）
2. 如何应对瞬时高并发？（10万人同时抢购）
3. 如何保证下单接口的可用性和性能？
4. 如果库存提前卖完了，如何快速拦截后续请求？

<details>
<summary>考察点提示</summary>

- Redis 原子扣减
- 消息队列削峰
- 限流熔断
- 提前拦截
</details>

---

### 题目45：搜索接口优化

> 你的搜索接口查询 QPS 达到 10000，需要在 50ms 内返回结果，但目前响应时间需要 500ms。

**问题：**
1. 如何定位性能瓶颈？是 CPU、IO 还是网络？
2. 如果数据库查询是瓶颈，有哪些优化思路？
3. 你了解 Elasticsearch 吗？它和数据库搜索的区别是什么？
4. 如何设计缓存策略来应对热点搜索词？

<details>
<summary>考察点提示</summary>

- 索引优化
- 查询优化
- ES 全文检索
- 搜索缓存
</details>

---

### 题目46：消息队列选型

> 你的系统需要在服务之间传递消息，需要支持消息的可靠投递和顺序性。团队在 RabbitMQ、Kafka、RocketMQ 之间纠结。

**问题：**
1. 这三种消息队列在架构和特性上有什么区别？
2. 如果对消息可靠性要求极高（不能丢消息），你选哪个？为什么？
3. 如果对吞吐量要求极高（每秒百万级），你选哪个？
4. 如果需要支持事务消息（比如先扣库存再发消息），有什么限制？

<details>
<summary>考察点提示</summary>

- 架构模型
- 消息可靠性
- 吞吐量
- 事务消息支持
</details>

---

## 九、中间件 & 工具

### 题目47：Kafka消息丢失

> 你的系统使用 Kafka 传递消息，发现偶尔会有消息丢失的问题。

**问题：**
1. Kafka 在哪些环节可能会丢失消息？
2. 如何配置 Producer 来保证消息不丢失？
3. 如果 Consumer 处理失败导致消息丢失，应该如何解决？
4. 你知道 exactly-once 语义吗？它是如何实现的？

<details>
<summary>考察点提示</summary>

- acks 配置
- 消费者手动提交
- 幂等 Producer
- 事务消息
</details>

---

### 题目48：Docker容器网络

> 你的微服务部署在 Docker 容器中，服务 A 无法访问服务 B，报错 `Connection refused`。

**问题：**
1. 你首先会排查哪些方面？
2. Docker 默认的网络模式是什么？有哪些网络模式？
3. 如果服务 A 和 B 在不同的 Docker 主机上，它们如何通信？
4. Kubernetes 中的 Service 是如何实现服务发现的？

<details>
<summary>考察点提示</summary>

- Docker 网络模式
- bridge/host/overlay
- Kubernetes Service
- DNS 服务发现
</details>

---

### 题目49：线上故障排查流程

> 凌晨2点，你收到报警：用户无法登录。接口返回 502。

**问题：**
1. 首先你应该做什么？获取哪些信息？
2. 如果是 Java 应用，你会使用哪些命令/工具来排查？
3. 如果是数据库问题（比如连接池耗尽），如何快速定位？
4. 故障恢复后，你需要做哪些复盘工作？

<details>
<summary>考察点提示</summary>

- 监控/日志/链路追踪
- Arthas/jstack/jmap
- 数据库连接池
- 故障复盘
</details>

---

### 题目50：接口性能优化

> 你的一个核心接口 P99 延迟是 500ms，目标是优化到 100ms 以内。

**问题：**
1. 你会如何分析这个接口的性能瓶颈？
2. 如果瓶颈在数据库查询，你会从哪些方面优化？
3. 如果需要引入缓存，缓存的读写策略是什么？
4. 如何验证优化效果？需要关注哪些指标？

<details>
<summary>考察点提示</summary>

- APM 工具
- 数据库索引/SQL优化
- 缓存策略
- 压测验证
</details>

---

## 十、场景综合题

### 题目51：系统设计综合

> 你的公司要做一个社区论坛系统，支持用户发帖子、评论、点赞。用户量预估 1000 万DAU，帖子总量 10 亿条。

**问题：**
1. 数据库如何设计？表结构如何规划？
2. 如何设计 Feed 流（用户可以看到关注的人的新帖子）？
3. 如何实现点赞功能的高并发支持？
4. 如果要实现全文搜索，你有什么方案？
5. 如果帖子内容包含图片/视频，如何处理存储问题？

<details>
<summary>考察点提示</summary>

- 分库分表
- Feed 流设计
- 计数器优化
- 搜索引擎
- 对象存储
</details>

---

### 题目52：即时通讯系统

> 你的公司要做一款即时通讯产品，类似微信，需要支持单聊、群聊、消息推送。

**问题：**
1. 消息如何存储？如果用户历史消息有 10 万条，如何快速加载？
2. 如何保证消息的顺序性？
3. 离线消息如何处理？未读数如何同步？
4. 如何实现消息的多端同步？
5. 如果用户 A 在给用户 B 发消息的同时，B 也给 A 发消息，消息流如何设计？

<details>
<summary>考察点提示</summary>

- 消息存储设计
- 顺序消息
- 离线消息
- 多端同步
- websocket 长连接
</details>

---

### 题目53：开放平台设计

> 你的公司要对外开放 API，第三方开发者可以通过 API 访问你们的数据和服务。

**问题：**
1. 如何实现 API 的认证和授权？
2. 如何限制 API 的调用频率（防刷）？
3. 如果第三方开发者的接口响应很慢，会影响你们的服务吗？如何隔离？
4. 如何监控第三方 API 的调用情况？
5. API 版本如何管理？

<details>
<summary>考察点提示</summary>

- OAuth 2.0
- 限流算法
- 资源隔离
- API 网关
- 版本控制
</details>

---

## 十一、计算机网络

### 题目54：TCP三次握手四次挥手

> 你的系统需要建立长连接和服务器通信，客户端突然断开了连接，服务端没有及时感知到。

**问题：**
1. 为什么 TCP 需要三次握手而不是两次？四次不行吗？
2. 为什么挥手需要四次，而握手只需要三次？
3. 如果客户端断开连接后不发送 FIN，服务端如何感知？
4. TCP 的 KeepAlive 机制是什么？HTTP 的 KeepAlive 有什么区别？

<details>
<summary>考察点提示</summary>

- 三次握手状态机
- TIME_WAIT 状态
- 四次挥手流程
- SO_KEEPALIVE
</details>

---

### 题目55：TCP滑动窗口与拥塞控制

> 你的文件上传接口传输一个大文件时，速度开始很快，后来变得越来越慢。

**问题：**
1. 为什么传输速度会逐渐变慢？这是什么机制在起作用？
2. TCP 的滑动窗口机制是什么？发送方和接收方各维护什么？
3. 拥塞控制有哪几种算法？它们分别什么时候触发？
4. 如果网络突然丢包，TCP 会如何反应？

<details>
<summary>考察点提示</summary>

- 慢启动
- 拥塞避免
- 快重传快恢复
- RTT 计算
</details>

---

### 题目56：HTTP与HTTPS

> 你的网站从 HTTP 切换到 HTTPS 后，发现接口响应时间变长了。

**问题：**
1. HTTPS 相比 HTTP 多了哪些步骤？为什么会更慢？
2. TLS 握手过程是怎样的？1.2 和 1.3 版本有什么区别？
3. 什么是 SSL 证书？证书链验证的过程是什么？
4. 如何优化 HTTPS 的性能？（至少说出 3 种）

<details>
<summary>考察点提示</summary>

- TLS 握手
- 对称加密 vs 非对称加密
- Session Resume
- HTTP/2 多路复用
</details>

---

### 题目57：HTTP/2与HTTP/3

> 你的网站启用了 HTTP/2，但测试发现性能提升不明显。

**问题：**
1. HTTP/2 相比 HTTP/1.1 有哪些主要改进？
2. HTTP/2 的多路复用和 HTTP/1.1 的 Pipeline 有什么区别？
3. 什么是 HPACK 头部压缩？它解决了什么问题？
4. HTTP/3 为什么选择 QUIC 协议？QUIC 相比 TCP 有什么优势？

<details>
<summary>考察点提示</summary>

- 多路复用
- Server Push
- HPACK
- UDP + QUIC
</details>

---

### 题目58：DNS解析过程

> 你的用户访问你的网站时，有时很慢，有时很快。怀疑是 DNS 解析的问题。

**问题：**
1. 浏览器输入 URL 到页面加载，DNS 解析经历了哪些步骤？
2. DNS 缓存有哪些层级？浏览器缓存、操作系统缓存、Local DNS 服务器缓存？
3. 什么是 DNS 污染和 DNS 劫持？如何防范？
4. 如何优化 DNS 解析速度？DNS 预解析是什么？

<details>
<summary>考察点提示</summary>

- DNS 分层解析
- 递归查询 vs 迭代查询
- DNS 缓存
- preconnect/prefetch
</details>

---

### 题目59：Socket编程问题

> 你的长连接服务在线上运行一段时间后，发现连接数越来越少，最终无法建立新连接。

**问题：**
1. Linux 系统有哪些参数限制了 socket 连接数？
2. 如果是服务器端，通常需要调整哪些内核参数？
3. 什么是文件描述符？为什么它会影响连接数？
4. 如何排查和解决连接泄漏问题？

<details>
<summary>考察点提示</summary>

- ulimit -n
- /proc/sys/net/ 相关参数
- 文件描述符
- 连接泄漏排查
</details>

---

### 题目60：CDN工作原理

> 你的静态资源部署在 CDN 上，但用户反馈有些地区访问很慢。

**问题：**
1. CDN 的工作原理是什么？它如何决定把用户请求路由到哪个节点？
2. 什么是 CDN 缓存？如何设置合理的缓存策略？
3. 如果源站变更了内容，如何快速刷新 CDN 缓存？
4. CDN 回源流量大会有什么影响？如何优化？

<details>
<summary>考察点提示</summary>

- DNS 调度
- 边缘节点
- 缓存命中率
- 缓存刷新机制
</details>

---

## 十二、操作系统

### 题目61：进程与线程

> 你的 Java 应用启动后，发现创建了很多线程，线程数量一直在增长。

**问题：**
1. 进程和线程的区别是什么？为什么线程更轻量？
2. Java 中的线程和操作系统的线程是什么关系？
3. 线程数量过多会有什么负面影响？如何确定合理的线程池大小？
4. 什么是协程？它和线程有什么区别？Java 支持协程吗？

<details>
<summary>考察点提示</summary>

- 进程内存空间
- 线程共享资源
- 上下文切换
- Quasar/Kotlin 协程
</details>

---

### 题目62：CPU上下文切换

> 你发现服务器的 CPU 使用率很高，但应用吞吐量却很低。怀疑是上下文切换过多。

**问题：**
1. 什么是 CPU 上下文切换？切换时会发生什么？
2. 什么情况会导致上下文切换？（进程切换、线程切换、中断）
3. 如何排查上下文切换是否过度？有哪些监控指标？
4. 如何减少不必要的上下文切换？

<details>
<summary>考察点提示</summary>

- 用户态/内核态
- 寄存器保存
- vmstat 命令
- 减少锁竞争
</details>

---

### 题目63：内存管理

> 你的应用需要处理大量数据，内存使用量很大，但 GC 很频繁。

**问题：**
1. 操作系统的内存管理机制是什么？分页和分段有什么区别？
2. 什么是虚拟内存？为什么需要虚拟内存？
3. 物理内存不足时，操作系统会怎么处理？（swap）
4. Java 的堆外内存是什么？什么情况下会使用堆外内存？

<details>
<summary>考察点提示</summary>

- 虚拟地址空间
- 页表
- swap 分区
- DirectByteBuffer
</details>

---

### 题目64：IO模型

> 你的系统需要支持 10 万并发连接，选择了 NIO，但发现性能不如预期。

**问题：**
1. Unix 有哪几种 IO 模型？阻塞、非阻塞、IO多路复用、异步IO 的区别？
2. select、poll、epoll 的区别是什么？epoll 为什么高效？
3. Java NIO 的 Channel、Buffer、Selector 是什么关系？
4. 为什么 10 万并发连接时，select 模型会比 epoll 差很多？

<details>
<summary>考察点提示</summary>

- IO 多路复用
- 水平触发 vs 边缘触发
- Reactor 模式
- Netty 架构
</details>

---

### 题目65：零拷贝技术

> 你的文件传输服务需要把大文件从磁盘发送到网络，传输速度很慢。

**问题：**
1. 传统的文件传输需要几次拷贝？分别是哪几次？
2. 什么是零拷贝？它是如何减少拷贝次数的？
3. sendfile 和 mmap 的区别是什么？各自适用什么场景？
4. Java 中如何使用零拷贝？NIO 的 FileChannel.transferTo() 是什么原理？

<details>
<summary>考察点提示</summary>

- DMA 拷贝
- 内核空间/用户空间
- Linux sendfile
- Java FileChannel
</details>

---

### 题目66：Linux命令排查问题

> 线上服务器 load 很高，CPU idle 是 0，用户说服务很卡。

**问题：**
1. 你会使用哪些命令来排查问题？排查顺序是什么？
2. top 和 vmstat 显示的信息有什么区别？
3. 如果 Java 进程 CPU 100%，如何定位是哪个线程/方法？
4. 如何分析线程的 CPU 占用情况？

<details>
<summary>考察点提示</summary>

- top/htop
- jstack 分析
- printf 转换
- Arthas profiler
</details>

---

### 题目67：磁盘IO优化

> 你的数据库服务器磁盘 IO 很高，IO Wait 占 CPU 的 30%。

**问题：**
1. 什么是 IO Wait？为什么会产生 IO Wait？
2. 机械硬盘和 SSD 在 IO 特性上有什么区别？
3. 如何优化磁盘 IO？有哪些方案？（至少 4 种）
4. Linux 的 IO 调度算法有哪些？各自适合什么场景？

<details>
<summary>考察点提示</summary>

- IOPS vs 吞吐量
- SSD vs HDD
- RAID 配置
- cfq/deadline/noop
</details>

---

## 十三、数据结构与算法

### 题目68：HashMap实现

> 面试官让你手写一个 HashMap。

**问题：**
1. HashMap 的底层数据结构是什么？为什么选择这个结构？
2. Hash 冲突是如何解决的？开放地址法和拉链法的区别？
3. 为什么 HashMap 容量是 2 的幂次？如何实现的？
4. JDK 1.8 对 HashMap 做了什么优化？

<details>
<summary>考察点提示</summary>

- 数组 + 链表/红黑树
-扰动函数
- 扩容机制
- 红黑树转化
</details>

---

### 题目69：ConcurrentHashMap并发安全

> 面试官让你分析 ConcurrentHashMap 是如何保证线程安全的。

**问题：**
1. JDK 1.7 和 JDK 1.8 的 ConcurrentHashMap 实现有什么区别？
2. Java 8 中是如何实现无锁化的？CAS + synchronized 的使用策略？
3. size() 方法是如何保证准确性的？涉及哪些并发控制？
4. putAll() 是如何实现的？为什么不是原子的？

<details>
<summary>考察点提示</summary>

- Segment 分段锁
- CAS 自旋
- synchronized 锁头节点
- CounterCell
</details>

---

### 题目70：LRU缓存实现

> 你的系统需要实现一个 LRU 缓存，容量是 100。

**问题：**
1. 如何设计数据结构，使得 get 和 put 都是 O(1) 的时间复杂度？
2. 你会使用什么数据结构来实现？
3. 如果需要支持过期时间，你会如何设计？
4. 如何实现线程安全的 LRU 缓存？

<details>
<summary>考察点提示</summary>

- LinkedHashMap
- HashMap + LinkedList
- 双端链表
- ReadWriteLock
</details>

---

### 题目71：分布式一致性算法

> 你的系统需要选出一个主节点来协调操作，了解了 Paxos 和 Raft 算法。

**问题：**
1. Paxos 算法中，有哪些角色？算法流程是什么？
2. Raft 算法相比 Paxos 有什么优势？为什么更容易实现？
3. Raft 中，如果 Follower 收不到心跳，会发生什么？
4. Raft 是如何保证分布式一致性的？日志复制流程是什么？

<details>
<summary>考察点提示</summary>

- Leader/Candidate/Follower
- Term 任期
- 日志复制
- 脑裂问题
</details>

---

### 题目72：排序算法对比

> 你的系统需要对 10 亿条数据进行排序。

**问题：**
1. Java 中的 Arrays.sort() 使用的是什么排序算法？为什么？
2. 对于海量数据排序，有哪些方案？分片 + 归并排序？
3. 如果内存只能容纳 100 万条数据，如何排序 10 亿条？
4. 什么是外部排序？它的基本原理是什么？

<details>
<summary>考察点提示</summary>

- TimSort
- 外部排序
- 多路归并
- 海量数据处理
</details>

---

### 题目73：搜索引擎排名算法

> 你的系统需要实现一个简单的搜索功能，按照相关性排序。

**问题：**
1. 如何计算文档和查询词的相关性？TF-IDF 是什么？
2. Elasticsearch 使用的是什么排序算法？为什么效率高？
3. 如果搜索结果需要按照多个维度排序（相关性 + 时间 + 热度），如何实现？
4. 什么是倒排索引？它和正排索引的区别是什么？

<details>
<summary>考察点提示</summary>

- 倒排索引
- TF-IDF
- BM25
- Lucene 原理
</details>

---

## 十四、设计模式

### 题目74：单例模式

> 面试官让你手写一个线程安全的单例模式。

**问题：**
1. 饿汉式和懒汉式有什么区别？各有什么优缺点？
2. DCL（双重检查锁定）为什么需要 volatile？
3. 除了这两种，还有什么方式可以实现单例？枚举实现单例的原理是什么？
4. 单例模式有什么缺点？在什么情况下不适合使用？

<details>
<summary>考察点提示</summary>

- 线程安全
- 指令重排序
- 枚举单例
- Spring 单例 Bean
</details>

---

### 题目75：工厂模式

> 你的系统需要根据配置创建不同的支付渠道（支付宝、微信、银联）。

**问题：**
1. 简单工厂、工厂方法、抽象工厂分别适合什么场景？
2. 如果使用 Spring，如何利用 Spring 的 BeanFactory 来实现工厂？
3. 如果支付渠道需要共用一些配置（AppID、密钥等），如何设计？
4. 工厂模式和策略模式有什么区别？它们可以一起使用吗？

<details>
<summary>考察点提示</summary>

- 简单工厂
- 工厂方法
- 抽象工厂
- Spring FactoryBean
</details>

---

### 题目76：观察者模式

> 你的系统需要实现一个事件通知功能，当订单状态变化时，通知多个系统。

**问题：**
1. 观察者模式的核心是什么？Subject 和 Observer 的关系？
2. Java 内置了对观察者模式的支持吗？有什么问题？
3. 如果 Observer 的执行顺序很重要，应该如何保证？
4. Spring 的 ApplicationEvent 是观察者模式吗？它是如何实现的？

<details>
<summary>考察点提示</summary>

- 解耦发布订阅
- 同步 vs 异步
- Java Observable
- Spring ApplicationEvent
</details>

---

### 题目77：责任链模式

> 你的系统需要实现一个请求过滤器链：认证 → 日志 → 限流 → 参数校验。

**问题：**
1. 责任链模式和命令模式有什么区别？
2. 如何实现一个可插拔的责任链？
3. 如果某个 Handler 处理失败，应该如何处理？直接返回还是继续传递？
4. Spring 的 FilterChain 是如何工作的？它和责任链模式有什么关系？

<details>
<summary>考察点提示</summary>

- Handler 链表
- doFilter()
- Servlet Filter
- Spring Security 过滤器链
</details>

---

### 题目78：策略模式

> 你的系统需要支持多种促销策略：满减、打折、买赠、积分抵扣。

**问题：**
1. 策略模式的核心是什么？策略和上下文的关系？
2. 如果促销策略需要共享一些数据（如用户等级、购物车内容），如何设计？
3. 策略模式如何和工厂模式结合使用？
4. 策略模式和 if-else 相比，有什么优势？

<details>
<summary>考察点提示</summary>

- 接口隔离
- 运行时切换
- Context 上下文
- 策略选择
</details>

---

## 十五、安全

### 题目79：SQL注入

> 你的系统收到了安全扫描报告，说存在 SQL 注入漏洞。

**问题：**
1. 什么是 SQL 注入？攻击者是如何利用的？
2. 如果使用 `#{}` 和 `${}` 有什么区别？哪个可以防止注入？
3. MyBatis 中使用 `<![CDATA[...]]>` 包裹的 SQL 是否安全？
4. 除了 SQL 注入，还有哪些常见的安全漏洞？

<details>
<summary>考察点提示</summary>

- 参数化查询
- MyBatis #{} vs ${}
- XSS/CSRF
- 预编译语句
</details>

---

### 题目80：接口鉴权

> 你的系统需要对外开放 API，需要实现接口鉴权机制。

**问题：**
1. 如何防止接口被恶意调用？你知道哪些方案？
2. HMAC 签名是如何实现的？它的原理是什么？
3. 如果使用 JWT，JWT 的结构是什么？如何验证签名？
4. access_token 和 refresh_token 有什么区别？如何设计 token 刷新机制？

<details>
<summary>考察点提示</summary>

- HMAC 签名
- JWT 结构
- Token 刷新
- OAuth 2.0
</details>

---

### 题目81：敏感数据加密

> 你的系统需要存储用户的手机号、身份证号等敏感信息。

**问题：**
1. 什么场景下使用对称加密？什么场景下使用非对称加密？
2. AES 和 RSA 有什么区别？各自适合什么场景？
3. 密码应该怎么存储？MD5 安全吗？
4. 如果需要加密字段进行模糊查询（如手机号），有什么方案？

<details>
<summary>考察点提示</summary>

- AES/DES/RSA
- 哈希加盐
- 加密 + 脱敏
- 同态加密
</details>

---

### 题目82：XSS与CSRF

> 安全扫描报告发现了 XSS 和 CSRF 漏洞。

**问题：**
1. XSS 攻击的原理是什么？如何防御？
2. 反射型 XSS、存储型 XSS、DOM 型 XSS 的区别是什么？
3. CSRF 攻击的原理是什么？如何防御？
4. SameSite Cookie 是什么？它能防止 CSRF 吗？

<details>
<summary>考察点提示</summary>

- 输入输出过滤
- HttpOnly Cookie
- CSRF Token
- SameSite 属性
</details>

---

## 十六、消息队列（深入）

### 题目83：RabbitMQ消息丢失

> 你的系统使用 RabbitMQ 传递消息，发现偶尔会有消息丢失。

**问题：**
1. RabbitMQ 的消息持久化机制是什么？publish 和 consumer 各需要做什么？
2. 如果消息发送到 RabbitMQ 后，RabbitMQ 还没持久化就宕机了，会丢失吗？
3. 什么是 publisher confirm？它是如何保证消息可靠发送的？
4. 如果 Consumer 处理失败，消息会怎样？会重发吗？

<details>
<summary>考察点提示</summary>

- 交换机/队列/消息持久化
- mandatory 参数
- publisher confirm
- manual ack
</details>

---

### 题目84：RabbitMQ顺序消费

> 你的系统需要对消息进行顺序消费，但发现顺序经常错乱。

**问题：**
1. RabbitMQ 本身能保证消息顺序吗？什么情况下会乱序？
2. 如果需要保证顺序消费，有哪些方案？
3. 多个 Consumer 消费同一个队列，能保证顺序吗？
4. 什么是消息优先级队列？它能解决顺序问题吗？

<details>
<summary>考察点提示</summary>

- 单队列 + 单 Consumer
- 消息分组
- 消费者线程池
- 消息优先级
</details>

---

### 题目85：Kafka高性能原理

> 你的 Kafka 集群每秒能处理百万级消息，它是如何做到这么高的吞吐量的？

**问题：**
1. Kafka 为什么这么快？它的写入流程是什么？
2. 什么是顺序写？为什么顺序写比随机写快？
3. 什么是零拷贝？Kafka 是如何利用零拷贝的？
4. Kafka 的分区和副本机制是什么？如何保证数据不丢失？

<details>
<summary>考察点提示</summary>

- 顺序写磁盘
- Page Cache
- 零拷贝 sendfile
- ISR 副本同步
</details>

---

### 题目86：Kafka消费模式

> 你的系统需要实现消息的多种消费模式：发布订阅、点对点、广播。

**问题：**
1. Kafka 的 Consumer Group 是什么？它和发布订阅模式有什么关系？
2. 如果 Consumer 数量超过 Partition 数量，会有什么影响？
3. 如何实现消息广播（一个消息所有 Consumer 都收到）？
4. 什么是 Consumer Rebalance？在什么时候会发生？

<details>
<summary>考察点提示</summary>

- Consumer Group
- Partition 分配
- Rebalance 过程
- 订阅模式
</details>

---

## 十七、容器与Kubernetes

### 题目87：Docker镜像构建

> 你的 Dockerfile 构建的镜像很大，超过 2GB，构建时间也很长。

**问题：**
1. 镜像大的原因是什么？Java 应用的 Docker 镜像通常包含哪些内容？
2. 如何优化 Dockerfile 来减小镜像体积？
3. 什么是多阶段构建？它有什么好处？
4. .dockerignore 文件有什么用？

<details>
<summary>考察点提示</summary>

- 基础镜像选择
- 多阶段构建
- 层缓存
- .dockerignore
</details>

---

### 题目88：Kubernetes Pod调度

> 你的 Pod 一直处于 Pending 状态，无法调度。

**问题：**
1. Pod 无法调度的常见原因有哪些？
2. Kubernetes 的调度过程是什么？Scheduler 做了什么？
3. 如果你的节点资源足够，但仍然无法调度，应该检查什么？
4. 什么是污点和容忍？如何使用它们来控制 Pod 调度？

<details>
<summary>考察点提示</summary>

- 资源不足
- 污点/容忍
- 亲和性/反亲和性
- Pod Priority
</details>

---

### 题目89：Kubernetes滚动更新

> 你的应用需要升级版本，想使用滚动更新，但担心更新过程中服务不可用。

**问题：**
1. Kubernetes 的滚动更新是如何工作的？有哪些关键参数？
2. 如何保证滚动更新过程中服务始终可用？
3. 如果新版本有 bug，如何快速回滚？
4. 什么是探针（Probe）？ReadinessProbe 和 LivenessProbe 有什么区别？

<details>
<summary>考察点提示</summary>

- RollingUpdate 策略
- maxSurge/maxUnavailable
- kubectl rollout
- 健康检查
</details>

---

### 题目90：Kubernetes服务发现

> 你的微服务之间需要相互调用，但在代码中硬编码了 IP 地址。

**问题：**
1. Kubernetes 中的服务发现机制是什么？CoreDNS 是如何工作的？
2. 如何在 Pod 中访问另一个 Service？环境变量 vs DNS 哪个更好？
3. ClusterIP、NodePort、LoadBalancer 有什么区别？
4. Ingress 和 Service 的区别是什么？什么场景下使用 Ingress？

<details>
<summary>考察点提示</summary>

- Service 抽象
- CoreDNS 解析
- kube-proxy
- Ingress Controller
</details>

---

## 十八、DevOps与CI/CD

### 题目91：Git分支策略

> 你的团队在开发一个新功能，但由于分支管理混乱，经常出现代码冲突和遗漏。

**问题：**
1. 你了解哪些 Git 分支策略？GitFlow 和 Trunk-based Development 有什么区别？
2. 如果要开发一个需要 2 周的大型功能，你会如何管理分支？
3. 如何设计代码合并流程？PR/MR 的最佳实践是什么？
4. 如何处理长时间未合并的分支？有哪些风险？

<details>
<summary>考察点提示</summary>

- GitFlow
- Trunk-based
- Feature Flag
- 持续集成
</details>

---

### 题目92：持续集成流水线

> 你的团队每天 deploy 多次，但经常出现 build 通过但上线后出问题的情况。

**问题：**
1. 持续集成的核心是什么？为什么要做 CI？
2. 一个完整的 CI/CD 流水线应该包含哪些阶段？
3. 如何保证测试的有效性？单元测试、集成测试、端到端测试的区别？
4. 如何处理 flaky test（不稳定的测试）？

<details>
<summary>考察点提示</summary>

- 自动化构建
- 自动化测试
- 代码质量检查
- 测试金字塔
</details>

---

### 题目93：蓝绿部署与金丝雀

> 你的系统需要升级版本，但不想影响在线用户。

**问题：**
1. 蓝绿部署的原理是什么？它有什么优缺点？
2. 金丝雀发布和灰度发布有什么区别？
3. 如何设计一个金丝雀发布的流量策略？
4. 如何监控金丝雀版本的异常并自动回滚？

<details>
<summary>考察点提示</summary>

- 蓝绿环境
- 流量切换
- 金丝雀比例
- 自动熔断
</details>

---

## 十九、日志与监控

### 题目94：分布式日志方案

> 你的系统部署在多台服务器上，出现问题时需要查看各台服务器的日志，很不方便。

**问题：**
1. 传统的日志管理有什么问题？为什么需要集中式日志？
2. ELK Stack 的架构是什么？各组件的作用是什么？
3. 如何实现日志的采集、传输、存储、搜索？
4. 如果日志量很大，如何设计日志的存储策略和生命周期？

<details>
<summary>考察点提示</summary>

- ELK Stack
- Logstash/Fluentd
- Elasticsearch
- 日志级别和生命周期
</details>

---

### 题目95：APM监控系统

> 你的接口响应慢，但不知道是哪个环节的问题。

**问题：**
1. APM（应用性能监控）和传统监控有什么区别？
2. 你了解哪些 APM 工具？SkyWalking、Pinpoint、CAT 有什么区别？
3. 分布式追踪的原理是什么？traceId 是如何在服务间传递的？
4. 如何判断接口慢是数据库问题还是网络问题？

<details>
<summary>考察点提示</summary>

- 链路追踪
- Span/Trace
- Agent 无侵入
- 采样策略
</details>

---

### 题目96：Prometheus监控

> 你的系统需要接入 Prometheus 监控。

**问题：**
1. Prometheus 的数据模型是什么？Metric 类型有哪些？
2. PushGateway 和 Pull 模式的区别是什么？各自适用什么场景？
3. 如何在 Java 应用中暴露自定义指标？
4. AlertManager 是如何工作的？如何配置告警规则？

<details>
<summary>考察点提示</summary>

- Metric Types
- PushGateway
- Micrometer
- 告警规则
</details>

---

## 二十、场景综合题

### 题目97：停车场系统设计

> 你的公司要做智慧停车场系统，支持车辆入场、出场、计时计费、月卡管理。

**问题：**
1. 如何实现车牌识别和车辆进场？
2. 如果停车场有 1000 个车位，如何设计实时车位显示？
3. 如何处理计费逻辑？临时车、月卡车的计费规则有何不同？
4. 如何实现无感支付（自动扣费）？
5. 如果系统出现故障导致闸机无法抬起，如何处理？

<details>
<summary>考察点提示</summary>

- 车牌识别
- 实时数据同步
- 支付集成
- 降级方案
</details>

---

### 题目98：医疗挂号系统

> 你的公司要做在线挂号系统，支持患者挂号、医生排班、候诊管理。

**问题：**
1. 如何设计医生排班系统？支持临时停诊、调诊等操作？
2. 如何防止号贩子恶意抢号？
3. 如何实现候诊排队功能？实时通知患者候诊进度？
4. 如果患者挂号后不来（爽约），如何处理？
5. 如何实现一人多号、恶意占号等异常检测？

<details>
<summary>考察点提示</summary>

- 排班算法
- 限流防刷
- WebSocket 推送
- 风控策略
</details>

---

### 题目99：物流配送系统

> 你的公司要做同城配送系统，支持订单下单、骑手接单、实时配送、订单完成。

**问题：**
1. 如何实现订单和骑手的智能匹配？考虑距离、骑手状态等因素？
2. 如何实现骑手位置的实时追踪？GPS 数据如何处理和存储？
3. 如何计算骑手的配送费用？阶梯计价还是其他方式？
4. 如果骑手中途取消订单，系统如何处理？
5. 如何实现订单的异常监控（如长时间未取货）？

<details>
<summary>考察点提示</summary>

- LBS 定位
- 路径规划
- 实时推送
- 异常处理
</details>

---

### 题目100：游戏后端系统

> 你的公司要做一款在线游戏，后端需要支持用户登录、房间匹配、对战同步、数据存储。

**问题：**
1. 如何设计游戏的登录认证？考虑 token 有效期、强制下线等？
2. 如何实现房间匹配？匹配算法是什么？
3. 如何保证游戏对战的实时性和一致性？帧同步 vs 状态同步？
4. 如果玩家中途退出，如何处理？
5. 如何防止游戏外挂和作弊？

<details>
<summary>考察点提示</summary>

- 实时通信
- WebSocket
- 帧同步
- 反作弊机制
</details>

---

### 题目101：电商秒杀系统（完整版）

> 双十一秒杀活动，10000台手机，100万人抢购。

**问题：**
1. 如何设计秒杀架构？整体架构分层是怎样的？
2. 如何防止超卖？Redis 原子操作和数据库乐观锁各有什么优缺点？
3. 如何应对瞬时高并发？10万人同时请求，如何扛住？
4. 如何设计限流策略？令牌桶和滑动窗口的区别？
5. 库存扣减后，如果用户支付超时，库存如何回滚？
6. 如何设计秒杀的黑名单和反作弊机制？

<details>
<summary>考察点提示</summary>

- CDN 静态化
- 消息队列削峰
- Redis 库存预扣
- 限流熔断
- 库存回滚
- 风控识别
</details>

---

### 题目102：数据平台架构设计

> 你的公司需要构建一个数据平台，支持数据采集、清洗、存储、分析。

**问题：**
1. 如何设计数据采集层？支持埋点日志、业务数据、第三方数据？
2. 如何处理数据的实时性和一致性？Lambda 架构 vs Kappa 架构？
3. 数据仓库的分层设计是怎样的？ODS、DWD、DWS、ADS 的区别？
4. 如何实现数据质量监控？数据血缘追踪？
5. 如果数据出现异常，如何快速定位问题数据？

<details>
<summary>考察点提示</summary>

- 数据采集
- ETL 管道
- 数据仓库
- 数据血缘
- 数据治理
</details>

---

## 📊 题目模式总结

| 模式 | 示例 | 考察维度 |
|------|------|----------|
| 1️⃣ 给方案 | 用 ConcurrentHashMap 统计 PV | 技术选型 |
| 2️⃣ 问问题 | 并发场景有什么问题？ | 问题识别 |
| 3️⃣ 追问原理 | JVM 层面会怎样？ | 原理理解 |
| 4️⃣ 要求优化 | 如何改进？ | 方案设计 |

## 📈 题目分类统计

| 分类 | 题目数量 | 新增内容 |
|------|----------|----------|
| Java基础 & 核心 | 7 | - |
| JVM 虚拟机 | 5 | - |
| 并发编程 | 6 | - |
| 数据库 MySQL | 6 | - |
| Redis | 6 | - |
| Spring & 框架 | 5 | - |
| 分布式 & 微服务 | 5 | - |
| 架构设计 | 6 | - |
| 中间件 & 工具 | 4 | - |
| 场景综合题 | 3 | - |
| **计算机网络** | **7** | 🆕 TCP/HTTP/DNS/ Socket |
| **操作系统** | **7** | 🆕 进程/IO/内存/磁盘 |
| **数据结构与算法** | **6** | 🆕 HashMap/LRU/排序/一致性算法 |
| **设计模式** | **5** | 🆕 单例/工厂/观察者/责任链 |
| **安全** | **4** | 🆕 SQL注入/鉴权/加密/XSS |
| **消息队列（深入）** | **4** | 🆕 RabbitMQ/Kafka |
| **容器与Kubernetes** | **4** | 🆕 Docker/K8s |
| **DevOps与CI/CD** | **3** | 🆕 Git/流水线/部署 |
| **日志与监控** | **3** | 🆕 ELK/APM/Prometheus |
| **场景综合题（续）** | **6** | 🆕 停车场/医疗/物流/游戏/数据平台 |
| **总计** | **102** | |

---

## 🎯 按难度分布

| 难度 | 数量 | 描述 |
|------|------|------|
| ⭐⭐⭐ | 约40% | 基础知识，需深入理解原理 |
| ⭐⭐⭐⭐ | 约45% | 进阶应用，需结合场景分析 |
| ⭐⭐⭐⭐⭐ | 约15% | 高级架构，需系统设计能力 |

---

> 本题库由 WorkBuddy 面试助手生成 | 适合 5-8 年 Java 后端工程师面试准备
> 最后更新：2026-04-08 | 共 102 道场景化面试题

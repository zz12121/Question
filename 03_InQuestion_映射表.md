# 03_InQuestion 代码找茬题-知识点映射表

> 生成时间：2026-04-21
> 映射范围：03_InQuestion（代码找茬型，题号1-300+）
> 说明：代码找茬题给出代码片段，分析问题与优化方案，映射精度以"涉及的核心Card知识点"为准

## 第1天（20260409）题号1-30

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 1 | ScheduledExecutorService定时任务执行超1秒 | 🟢 | 11_并发 | 线程池/ScheduledThreadPoolExecutor | scheduleAtFixedRate堆积 |
| 2 | synchronized实现简单缓存 | 🟢 | 11_并发 | synchronized/Monitor | 粒度粗/ConcurrentHashMap |
| 3 | 批量处理1000条分10批并行 | 🟢 | 11_并发 | 线程池/核心参数 | 线程池替代new Thread |
| 4 | ThreadPoolExecutor线程池配置 | 🟢 | 11_并发 | 线程池/核心参数 | 拒绝策略/队列选择 |
| 5 | 延迟队列处理超时订单 | 🟢 | 11_并发 | BlockingQueue/DelayQueue | 延迟队列/时间轮 |
| 6 | ReentrantReadWriteLock读多写少缓存 | 🟢 | 11_并发 | ReentrantReadWriteLock/读写锁 | 锁降级/写锁获取 |
| 7 | 全局唯一ID生成器 | 🟡 | 18_分布式 | 一致性哈希/虚拟节点 | 雪花算法/Leaf |
| 8 | CompletableFuture并行调用两个接口 | 🟢 | 11_并发 | CompletableFuture/异步编排 | supplyAsync/超时 |
| 9 | 频繁拼接字符串 | 🟢 | 10_基础 | String/intern与字符串常量池 | StringBuilder/StringBuffer |
| 10 | HashMap存储热数据缓存 | 🟢 | 10_基础 | HashMap/扩容与树化 | 线程不安全/死循环 |
| 11 | 递归算法处理树形数据 | 🟡 | 12_JVM | GC/对象分配与晋升 | 栈溢出/尾递归 |
| 12 | WeakHashMap做缓存 | 🟢 | 10_基础 | 引用类型/强软弱虚 | GC回收/WeakReference |
| 13 | 频繁创建和销毁短连接 | 🟡 | 19_计算机网络 | TCP/连接管理 | 连接池/Keep-Alive |
| 14 | final修饰大对象 | 🟢 | 10_基础 | final/语义与JMM | 不可变/构造函数逸出 |
| 15 | 订单表5000万加索引 | 🟢 | 13_数据库/MySQL | 索引/B+树与B树 | 索引选择/联合索引 |
| 16 | 分页查询实现 | 🟢 | 13_数据库/MySQL | 深分页/优化方案 | LIMIT OFFSET/游标分页 |
| 17 | INSERT IGNORE做幂等 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 唯一索引/间隙锁 |
| 18 | FOR UPDATE实现悲观锁 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 加锁范围/死锁 |
| 19 | DELETE清理过期数据 | 🟡 | 13_数据库/MySQL | 慢查询/定位与优化 | 大事务/分批删除 |
| 20 | 设计表结构 | 🟢 | 13_数据库/MySQL | 索引/B+树与B树 | 字段类型/范式/冗余 |
| 21 | Redis做分布式锁 | 🟢 | 14_NoSQL/Redis | Lua/脚本执行 | SET NX EX/看门狗 |
| 22 | Redis缓存热点数据 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | 缓存穿透/击穿/雪崩 |
| 23 | Redis List做消息队列 | 🟢 | 14_NoSQL/Redis | Stream/消费组 | LPUSH/RPOP/可靠性 |
| 24 | Redis做计数器 | 🟢 | 14_NoSQL/Redis | Hash/ziplist与hashtable | INCR/Long精度 |
| 25 | Redis Pipeline批量操作 | 🟢 | 14_NoSQL/Redis | Pipeline/批量执行 | 批量大小/原子性 |
| 26 | Spring @Async实现异步 | 🟢 | 15_框架/Spring | @Async/异步执行 | 线程池/异常处理 |
| 27 | @Transactional声明式事务 | 🟢 | 15_框架/Spring | 事务/失效场景 | 传播行为/自调用 |
| 28 | Feign调用下游服务 | 🟡 | 22_微服务 | 微服务/负载均衡 | 超时/重试/熔断 |
| 29 | Nacos配置中心动态刷新 | 🟡 | 18_分布式 | 配置中心/集中管理 | 长轮询/@RefreshScope |
| 30 | Gateway做限流 | 🟡 | 22_微服务 | 微服务/安全 | 令牌桶/过滤器 |

## 第2天（20260410）题号31-60

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 31 | synchronized实现单例模式 | 🟢 | 21_设计模式 | 单例模式/全家族对比 | DCL/枚举单例 |
| 32 | ConcurrentHashMap做缓存加过期时间 | 🟢 | 10_基础 | ConcurrentHashMap/结构演进 | 原子性/putIfAbsent |
| 33 | ExecutorService提交一批任务 | 🟢 | 11_并发 | 线程池/execute与submit | 异常丢失/shutdown |
| 34 | ReentrantLock带超时的锁 | 🟢 | 11_并发 | LockSupport/park与unpark | tryLock/超时/公平性 |
| 35 | BlockingQueue实现生产者消费者 | 🟢 | 11_并发 | BlockingQueue/TransferQueue | ArrayBlockingQueue/有界 |
| 36 | AtomicInteger实现计数器 | 🟢 | 11_并发 | CAS/ABA问题 | 原子类/LongAdder |
| 37 | ThreadLocal存储用户信息 | 🟢 | 11_并发 | ThreadLocal/内存泄漏 | 内存泄漏/线程池复用 |
| 38 | StampedLock优化读写锁 | 🟢 | 11_并发 | StampedLock/乐观读 | validate/锁转换 |
| 39 | 日志工具类频繁创建大对象 | 🟢 | 12_JVM | GC/对象分配与晋升 | 对象池/字符串缓存 |
| 40 | System.currentTimeMillis()获取时间戳 | 🟡 | 11_并发 | ThreadLocalRandom | System.nanoTime/精度 |
| 41 | BigDecimal金额计算 | 🟢 | 10_基础 | 基本类型/自动装箱与缓存 | 精度/构造方法/equals |
| 42 | Arrays.asList()转换数组 | 🟢 | 10_基础 | 集合/Comparable与Comparator | 固定大小/原始类型数组 |
| 43 | substring()截取字符串 | 🟡 | 10_基础 | String/intern与字符串常量池 | JDK6内存泄漏/JDK7+ |
| 44 | try-catch捕获所有异常 | 🟢 | 10_基础 | 异常体系/Checked与Unchecked | 异常吞没/日志 |
| 45 | 查询SQL优化 | 🟢 | 13_数据库/MySQL | 慢查询/定位与优化 | 索引/执行计划 |
| 46 | 更新SQL优化 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 行锁/死锁/范围更新 |
| 47 | 批量插入优化 | 🟢 | 13_数据库/MySQL | INSERT流程/redo undo binlog两阶段提交 | 批量大小/rewriteBatchedStatements |
| 48 | 查询用OR条件 | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 索引合并/UNION优化 |
| 49 | 表结构设计 | 🟢 | 13_数据库/MySQL | 索引/B+树与B树 | 字段类型/索引设计 |
| 50 | 事务代码 | 🟢 | 15_框架/Spring | 事务/失效场景 | 传播/隔离/回滚异常 |
| 51 | 缓存更新代码 | 🟢 | 14_NoSQL/Redis | Pipeline/批量执行 | Cache Aside/双写一致性 |
| 52 | 缓存读取代码 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | 穿透/击穿/序列化 |
| 53 | Redis做限流 | 🟢 | 14_NoSQL/Redis | Lua/脚本执行 | 滑动窗口/令牌桶 |
| 54 | Redis做排行榜 | 🟢 | 14_NoSQL/Redis | Hash/ziplist与hashtable | ZADD/ZRANGEBYSCORE/skiplist |
| 55 | Redis存储Session | 🟢 | 14_NoSQL/Redis | 淘汰策略/LRU与LFU | TTL/过期/分布式Session |
| 56 | Controller代码 | 🟡 | 15_框架/SpringMVC | DispatcherServlet/请求流程 | 参数校验/异常处理 |
| 57 | Service代码 | 🟢 | 15_框架/Spring | 事务/失效场景 | 内部调用/代理 |
| 58 | 配置类 | 🟢 | 15_框架/SpringBoot | 配置加载/优先级 | @Configuration/条件装配 |
| 59 | Feign调用 | 🟡 | 22_微服务 | 微服务/负载均衡 | 超时/重试/熔断器 |
| 60 | 过滤器 | 🟡 | 15_框架/SpringMVC | DispatcherServlet/请求流程 | Filter/Interceptor/顺序 |

## 第3天（20260411）题号61-90

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 61 | CountDownLatch等待多任务完成 | 🟢 | 11_并发 | CountDownLatch/倒计时 | 异常未countDown/超时 |
| 62 | CyclicBarrier多线程分阶段 | 🟢 | 11_并发 | CyclicBarrier/循环栅栏 | vs CountDownLatch/中断 |
| 63 | Semaphore控制并发数 | 🟢 | 11_并发 | Semaphore/信号量 | permit/release/公平性 |
| 64 | Future获取异步任务结果 | 🟢 | 11_并发 | 线程池/execute与submit | get阻塞/超时/取消 |
| 65 | CompletableFuture链式调用 | 🟢 | 11_并发 | CompletableFuture/异步编排 | thenApply/thenCompose/异常 |
| 66 | ConcurrentLinkedQueue无界队列 | 🟢 | 11_并发 | BlockingQueue/TransferQueue | 无界/OOM/内存泄漏 |
| 67 | CopyOnWriteArrayList读多写少列表 | 🟢 | 10_基础 | ConcurrentHashMap/结构演进 | 写时拷贝/内存/迭代器 |
| 68 | BlockingQueue drainTo批量消费 | 🟢 | 11_并发 | BlockingQueue/TransferQueue | drainTo/批量/一致性 |
| 69 | String.intern()节省内存 | 🟢 | 10_基础 | String/intern与字符串常量池 | JDK6 vs JDK7+/OOM |
| 70 | Thread.sleep(0) | 🟡 | 11_并发 | LockSupport/park与unpark | 主动让出CPU/时间片 |
| 71 | Object.wait()/notify() | 🟢 | 11_并发 | synchronized/Monitor | 等待/通知/虚假唤醒 |
| 72 | addShutdownHook() | 🟡 | 11_并发 | 中断机制/interrupt | 优雅关闭/钩子线程 |
| 73 | System.gc()主动触发GC | 🟢 | 12_JVM | GC/收集器对比 | 显式GC/禁用-XX:+DisableExplicitGC |
| 74 | finalize()方法 | 🟢 | 12_JVM | GC/对象分配与晋升 | Finalizer队列/内存泄漏 |
| 75 | 批量更新 | 🟢 | 13_数据库/MySQL | INSERT流程/redo undo binlog两阶段提交 | 批量/事务/锁 |
| 76 | 查询用到函数 | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 函数索引/索引失效 |
| 77 | 查询隐式类型转换 | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 类型不匹配/索引失效 |
| 78 | SELECT * | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 覆盖索引/网络开销 |
| 79 | 事务隔离级别 | 🟢 | 13_数据库/MySQL | MVCC/undo log版本链 | RC/RR/幻读 |
| 80 | 分页查询大偏移量 | 🟢 | 13_数据库/MySQL | 深分页/优化方案 | 游标/子查询/延迟关联 |
| 81 | 缓存预热代码 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | 预热策略/冷启动 |
| 82 | 缓存删除代码 | 🟢 | 14_NoSQL/Redis | 懒删除/UNLINK与DEL | 延迟双删/消息通知 |
| 83 | Redis分布式限流滑动窗口 | 🟢 | 14_NoSQL/Redis | Lua/脚本执行 | 滑动窗口/ZSET |
| 84 | Redis存储热点数据 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | 本地缓存/热点拆分 |
| 85 | Redis分布式锁Redisson | 🟢 | 14_NoSQL/Redis | Lua/脚本执行 | Redisson/看门狗/红锁 |
| 86 | Bean注入 | 🟢 | 15_框架/Spring | IOC/BeanDefinition与容器 | 循环依赖/三级缓存 |
| 87 | 配置属性 | 🟢 | 15_框架/SpringBoot | 配置加载/优先级 | @Value/@ConfigurationProperties |
| 88 | 定时任务 | 🟢 | 11_并发 | 线程池/ScheduledThreadPoolExecutor | @Scheduled/线程池/异常 |
| 89 | 异常处理 | 🟢 | 15_框架/Spring | 事务/失效场景 | @ControllerAdvice/全局异常 |
| 90 | 接口返回 | 🟡 | 15_框架/SpringMVC | DispatcherServlet/请求流程 | 统一返回/泛型/序列化 |

## 第4天（20260412）题号91-120

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 91 | ThreadLocalRandom生成随机数 | 🟢 | 11_并发 | ThreadLocalRandom | vs Random/CAS竞争 |
| 92 | synchronized修饰静态方法 | 🟢 | 11_并发 | synchronized/Monitor | Class对象锁vs实例锁 |
| 93 | ReentrantLock公平锁 | 🟢 | 11_并发 | LockSupport/park与unpark | 公平/非公平/FIFO/吞吐量 |
| 94 | ReadWriteLock锁降级 | 🟢 | 11_并发 | ReentrantReadWriteLock/读写锁 | 锁降级/获取写→获取读→释放写 |
| 95 | ThreadFactory自定义线程创建 | 🟢 | 11_并发 | 线程池/核心参数 | 命名/UncaughtExceptionHandler |
| 96 | ExecutorService优雅关闭 | 🟢 | 11_并发 | 线程池/核心参数 | shutdown/shutdownNow/awaitTermination |
| 97 | Phaser动态注册参与者 | 🟢 | 11_并发 | Phaser/动态注册分层 | arrive/register/bulkRegister |
| 98 | Exchanger线程间交换数据 | 🟢 | 11_并发 | Exchanger/线程间交换 | exchange/超时 |
| 99 | ForkJoinPool递归任务 | 🟢 | 11_并发 | ForkJoin/工作窃取算法 | RecursiveTask/分治 |
| 100 | LongAdder vs AtomicLong | 🟢 | 11_并发 | CAS/ABA问题 | 分散热点/Cell/striped64 |
| 101 | Double-Checked Locking单例 | 🟢 | 21_设计模式 | 单例模式/全家族对比 | volatile/指令重排 |
| 102 | 类加载器隔离 | 🟢 | 12_JVM | 类加载/双亲委派机制 | 自定义ClassLoader/类冲突 |
| 103 | Metaspace无限增长 | 🟢 | 12_JVM | Metaspace/元空间与永久代 | 动态代理/CGLIB/MaxMetaspaceSize |
| 104 | GC调优-XX参数 | 🟢 | 12_JVM | 调优/方法论 | GC日志/Xmx/Xms |
| 105 | 堆外内存泄漏 | 🟢 | 12_JVM | 直接内存/Direct Buffer | NIO/ByteBuffer/释放 |
| 106 | 死锁代码 | 🟢 | 11_并发 | 死锁/四个必要条件 | jstack排查/避免策略 |
| 107 | CAS自旋开销 | 🟢 | 11_并发 | CAS/ABA问题 | 自旋/LongAdder/PAUSE |
| 108 | 线程池大小设置 | 🟢 | 11_并发 | 线程池/核心参数 | CPU密集/IO密集/公式 |
| 109 | 索引设计-联合索引 | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 最左前缀/排序/覆盖 |
| 110 | 死锁检测 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | SHOW ENGINE INNODB STATUS |
| 111 | UPDATE不走索引 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 全表扫描/表锁/慢 |
| 112 | INSERT并发冲突 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 唯一索引/间隙锁/死锁 |
| 113 | 事务传播行为 | 🟢 | 15_框架/Spring | 事务/失效场景 | REQUIRES_NEW/NESTED |
| 114 | 事务超时 | 🟢 | 15_框架/Spring | 事务/失效场景 | timeout/长事务/锁持有 |
| 115 | Redis连接池配置 | 🟡 | 14_NoSQL/Redis | Pipeline/批量执行 | Lettuce/Jedis/连接数 |
| 116 | Redis缓存穿透 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | 布隆过滤器/空值缓存 |
| 117 | Redis集群数据倾斜 | 🟢 | 14_NoSQL/Redis | Cluster/故障转移 | 热点key/槽分配 |
| 118 | Spring循环依赖 | 🟢 | 15_框架/Spring | IOC/三级缓存与循环依赖 | 三级缓存/代理提前暴露 |
| 119 | Spring事件机制 | 🟢 | 15_框架/Spring | 事件/ApplicationEvent | 异步事件/事务绑定事件 |
| 120 | Spring AOP失效 | 🟢 | 15_框架/Spring | AOP/JDK与CGLIB动态代理 | 内部调用/代理对象 |

## 第5天（20260413）题号121-150

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 121 | ConcurrentHashMap过期清理 | 🟢 | 10_基础 | ConcurrentHashMap/结构演进 | removeIf/原子性/过期方案 |
| 122 | 线程池awaitTermination | 🟢 | 11_并发 | 线程池/execute与submit | 异常/CompletionService |
| 123 | 线程安全懒加载DCL | 🟢 | 21_设计模式 | 单例模式/全家族对比 | volatile/DCL/指令重排 |
| 124 | volatile不保证原子性 | 🟢 | 11_并发 | volatile/可见性 vs synchronized/互斥性 | i++非原子/AtomicInteger |
| 125 | Condition条件变量 | 🟢 | 11_并发 | ReentrantReadWriteLock/读写锁 | Condition/await/signal |
| 126 | 读写锁并发问题 | 🟢 | 11_并发 | ReentrantReadWriteLock/读写锁 | 饥饿/写锁获取/公平性 |
| 127 | ThreadLocal跨线程传递 | 🟡 | 11_并发 | ThreadLocal/内存泄漏 | InheritableThreadLocal/TransmittableThreadLocal |
| 128 | 线程池拒绝策略 | 🟢 | 11_并发 | 线程池/核心参数 | AbortPolicy/CallerRunsPolicy/自定义 |
| 129 | G1 Mixed GC调优 | 🟢 | 12_JVM | GC/G1 Mixed GC | InitiatingHeapOccupancyPercent |
| 130 | ZGC调优 | 🟡 | 12_JVM | GC/ZGC与Shenandoah | ZGC参数/着色指针 |
| 131 | Young GC频繁 | 🟢 | 12_JVM | GC/对象分配与晋升 | TLAB/Eden/晋升年龄 |
| 132 | Full GC排查 | 🟢 | 12_JVM | 调优/CPU 100%排查 | jstat/jmap/Memory Leak |
| 133 | 类加载冲突 | 🟢 | 12_JVM | 类加载/双亲委派机制 | 多版本/ClassLoader隔离 |
| 134 | JVM启动慢 | 🟡 | 12_JVM | JIT/热点代码探测 | 解释执行/分层编译 |
| 135 | 死锁与超时 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | innodb_lock_wait_timeout |
| 136 | 分库分表跨库事务 | 🟡 | 18_分布式 | 分布式事务/TCC与Saga | 分布式事务/最终一致 |
| 137 | 索引失效 | 🟢 | 13_数据库/MySQL | 索引/覆盖索引与ICP | 隐式转换/函数/OR |
| 138 | 慢查询优化 | 🟢 | 13_数据库/MySQL | 慢查询/定位与优化 | EXPLAIN/索引/重构 |
| 139 | RedLock分布式锁 | 🟢 | 14_NoSQL/Redis | Cluster/故障转移 | RedLock/多节点/时钟 |
| 140 | Redis Stream消费 | 🟢 | 14_NoSQL/Redis | Stream/消费组 | XREADGROUP/ACK/Pending |
| 141 | Redis BigKey删除 | 🟢 | 14_NoSQL/Redis | 大Key/发现与处理 | UNLINK/hscan+del |
| 142 | Spring Boot启动优化 | 🟢 | 15_框架/SpringBoot | Actuator/监控端点 | lazy-init/组件扫描 |
| 143 | Spring AOP链路调用 | 🟢 | 15_框架/Spring | AOP/JDK与CGLIB动态代理 | 多切面顺序/ProceedingJoinPoint |
| 144 | Feign超时重试 | 🟡 | 22_微服务 | 微服务/负载均衡 | 超时/重试/幂等 |
| 145 | Nacos配置变更 | 🟡 | 18_分布式 | 配置中心/集中管理 | @RefreshScope/灰度发布 |
| 146 | Gateway网关配置 | 🟡 | 22_微服务 | 微服务/安全 | 路由/过滤器/限流 |
| 147 | Dubbo泛化调用 | 🔴 | 15_框架/Dubbo | （无直接对应） | 泛化/参数/序列化 |
| 148 | Sentinel熔断降级 | 🟡 | 22_微服务 | 微服务/安全 | 慢调用/异常比例/热点 |
| 149 | MyBatis一级缓存问题 | 🟡 | 15_框架/MyBatis | （无直接对应） | SqlSession/脏读 |
| 150 | MyBatis批量操作 | 🟡 | 15_框架/MyBatis | （无直接对应） | ExecutorType.BATCH/批量 |

## 第6天（20260414）题号151-180

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 151 | CompletableFuture组合多异步调用 | 🟢 | 11_并发 | CompletableFuture/异步编排 | commonPool/自定义线程池/allOf |
| 152 | 滑动窗口限流器实现 | 🟡 | 11_并发 | CAS/ABA问题 | ConcurrentLinkedDeque/synchronized |
| 153 | 并发计数器LongAdder | 🟢 | 11_并发 | CAS/ABA问题 | vs AtomicLong/分散热点 |
| 154 | 线程池动态调整 | 🟢 | 11_并发 | 线程池/核心参数 | setCorePoolSize/动态扩缩 |
| 155 | ThreadLocal内存泄漏排查 | 🟢 | 11_并发 | ThreadLocal/内存泄漏 | 堆外泄漏/线程池复用/remove |
| 156 | ConcurrentHashMap computeIfAbsent | 🟢 | 10_基础 | ConcurrentHashMap/结构演进 | 原子操作/映射函数/死锁 |
| 157 | StampedLock乐观读实现缓存 | 🟢 | 11_并发 | StampedLock/乐观读 | tryOptimisticRead/validate |
| 158 | 生产者消费者多种实现 | 🟢 | 11_并发 | BlockingQueue/TransferQueue | BlockingQueue/Lock+Condition/Exchanger |
| 159 | ZGC调优实践 | 🟡 | 12_JVM | GC/ZGC与Shenandoah | 染色指针/并发整理/ZAllocationSpike |
| 160 | JVM堆外内存监控 | 🟢 | 12_JVM | 直接内存/Direct Buffer | NMT/jcmd/ByteBuffer |
| 161 | ParNew+CMS调优 | 🟢 | 12_JVM | GC/收集器对比 | CMS碎片/Full GC/Concurrent Mode Failure |
| 162 | 逃逸分析优化失效 | 🟢 | 12_JVM | 逃逸分析/标量替换与栈上分配 | 逃逸/标量替换/锁消除 |
| 163 | JIT编译优化 | 🟢 | 12_JVM | JIT/热点代码探测 | 内联/C2/逃逸分析 |
| 164 | MVCC与ReadView | 🟢 | 13_数据库/MySQL | MVCC/undo log版本链 | RC/RR下ReadView差异 |
| 165 | 间隙锁导致死锁 | 🟢 | 13_数据库/MySQL | 锁/表锁行锁意向锁 | 间隙锁/临键锁/插入意向锁 |
| 166 | 自增主键与索引 | 🟢 | 13_数据库/MySQL | 索引/B+树与B树 | 自增/UUID/页分裂 |
| 167 | Online DDL | 🟢 | 13_数据库/MySQL | Online DDL/ALGORITHM | INSTANT/INPLACE/锁 |
| 168 | 临时表优化 | 🟡 | 13_数据库/MySQL | 存储引擎/InnoDB vs MyISAM | 内部临时表/外部临时表 |
| 169 | Redis AOF重写 | 🟢 | 14_NoSQL/Redis | RDB/快照持久化 | AOF重写/fork/混合持久化 |
| 170 | Redis集群扩缩容 | 🟢 | 14_NoSQL/Redis | Cluster/故障转移 | 槽迁移/ASK/MOVED |
| 171 | Redis数据序列化 | 🟡 | 14_NoSQL/Redis | Pipeline/批量执行 | 序列化方式/JSON/Protobuf |
| 172 | Spring事务传播-NESTED | 🟢 | 15_框架/Spring | 事务/失效场景 | NESTED/Savepoint/REQUIRES_NEW |
| 173 | Spring事件异步处理 | 🟢 | 15_框架/Spring | 事件/ApplicationEvent | @Async/线程池/异常 |
| 174 | Spring Bean生命周期 | 🟢 | 15_框架/Spring | BeanPostProcessor/初始化前后 | 初始化/销毁/回调 |
| 175 | Feign调用链路追踪 | 🟡 | 18_分布式 | 链路追踪/分布式追踪 | TraceId/ Sleuth/MDC |
| 176 | Nacos服务注册 | 🟡 | 18_分布式 | 配置中心/集中管理 | 心跳/健康检查/临时实例 |
| 177 | Gateway全局过滤器 | 🟡 | 22_微服务 | 微服务/安全 | 鉴权/限流/跨域 |
| 178 | Dubbo超时与重试 | 🟡 | 15_框架/Dubbo | （无直接对应） | timeout/retries/幂等 |
| 179 | 分布式锁Redisson看门狗 | 🟢 | 14_NoSQL/Redis | Lua/脚本执行 | 续期/Pinning/锁释放 |
| 180 | 分库分表分片策略 | 🟡 | 13_数据库/MySQL | 深分页/优化方案 | ShardingSphere/分片键/范围/哈希 |

## 第7天（20260415）题号181-210

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| 181 | 虚拟线程并发HTTP请求/Pinning | 🟢 | 11_并发 | 虚拟线程/Thread与VirtualThread | Pinning/synchronized/ReentrantLock |
| 182 | 结构化并发StructuredTaskScope | 🟢 | 11_并发 | 虚拟线程/Thread与VirtualThread | ShutdownOnFailure/超时 |
| 183 | ForkJoinPool自定义线程池CompletableFuture | 🟢 | 11_并发 | CompletableFuture/异步编排 | 自定义Executor/不共用commonPool |
| 184 | CompletableFuture异常处理 | 🟢 | 11_并发 | CompletableFuture/异步编排 | exceptionally/handle/whenComplete |
| 185 | CompletableFuture超时控制 | 🟢 | 11_并发 | CompletableFuture/异步编排 | orTimeout/completeOnTimeout |
| 186 | Disruptor无锁队列 | 🟡 | 11_并发 | 伪共享/@Contended与缓存行填充 | RingBuffer/序号/缓存行对齐 |
| 187 | 虚拟线程+ReentrantLock | 🟢 | 11_并发 | 虚拟线程/Thread与VirtualThread | ReentrantLock不Pinning/synchronized P |
| 188 | Java 21新并发API | 🟡 | 11_并发 | 虚拟线程/Thread与VirtualThread | 新API/废弃方法 |
| 189 | G1 Full GC根因分析 | 🟢 | 12_JVM | GC/G1 Mixed GC | Evacuation Failure/Humongous/To空间 |
| 190 | ZGC着色指针 | 🟢 | 12_JVM | GC/ZGC与Shenandoah | 染色指针/读屏障/视图 |
| 191 | Safepoint性能影响 | 🟢 | 12_JVM | Safepoint/安全点与安全区域 | 可数循环/时间安全点/JFR |
| 192 | Code Cache满 | 🟢 | 12_JVM | Code Cache/代码缓存 | ReservedCodeCacheSize/JIT |
| 193 | 堆外内存NMT排查 | 🟢 | 12_JVM | NMT/原生内存追踪 | jcmd/内存分类/泄漏定位 |
| 194 | Compact Strings优化 | 🟢 | 12_JVM | Compact Strings/紧凑字符串 | byte[]/LATIN1/UTF16 |
| 195 | Binlog格式选择 | 🟢 | 13_数据库/MySQL | INSERT流程/redo undo binlog两阶段提交 | STATEMENT/ROW/GTID |
| 196 | Buffer Pool调优 | 🟡 | 13_数据库/MySQL | 存储引擎/InnoDB vs MyISAM | innodb_buffer_pool_size/LRU |
| 197 | 分库分表跨库排序 | 🟡 | 13_数据库/MySQL | 深分页/优化方案 | 内存排序/归并/限制 |
| 198 | Redis 7.0 Function | 🟢 | 14_NoSQL/Redis | EVALSHA/脚本缓存 | Redis Function/Lua替代 |
| 199 | Redis ACL权限 | 🟢 | 14_NoSQL/Redis | ACL/权限控制 | 用户/命令限制/Key模式 |
| 200 | Redis RDB+AOF混合持久化 | 🟢 | 14_NoSQL/Redis | RDB/快照持久化 | 混合/RDB基础+AOF增量 |
| 201 | Spring Boot 3启动流程 | 🟢 | 15_框架/SpringBoot | SpringBoot3/JDK17 | SpringApplication/RunListener |
| 202 | Spring AOT编译 | 🟢 | 15_框架/SpringBoot | AOT/GraalVM Native | 提前处理/反射注册/Proxy |
| 203 | Spring WebFlux响应式 | 🟢 | 15_框架/SpringMVC | DispatcherServlet/请求流程 | Reactor/Mono/Flux/Netty |
| 204 | Spring Observability | 🟢 | 15_框架/SpringBoot | SpringBoot3/JDK17 | Micrometer/Tracing/Metrics |
| 205 | Feign + Sentinel熔断 | 🟡 | 22_微服务 | 微服务/安全 | 熔断/降级/fallback |
| 206 | Nacos集群选主 | 🟡 | 18_分布式 | 配置中心/集中管理 | Raft/Leader选举/脑裂 |
| 207 | Gateway动态路由 | 🟡 | 22_微服务 | 微服务/安全 | 动态路由/Nacos/事件驱动 |
| 208 | Dubbo服务治理 | 🟡 | 15_框架/Dubbo | （无直接对应） | 路由/熔断/限流 |
| 209 | Seata分布式事务 | 🟡 | 18_分布式 | 分布式事务/TCC与Saga | AT模式/undo log/全局锁 |
| 210 | Kafka消息积压 | 🟡 | 16_消息 | 消息队列/可靠性投递 | 消费者扩容/分区/Lag |

## 补充题（20260416）题号C211-C270

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| C211 | ConcurrentHashMap本地缓存+过期 | 🟢 | 10_基础 | ConcurrentHashMap/结构演进 | 原子操作/过期清理 |
| C212 | 线程池监控与动态调参 | 🟢 | 11_并发 | 线程池/核心参数 | 监控/动态调整/hook |
| C213 | LockSupport底层原理 | 🟢 | 11_并发 | LockSupport/park与unpark | Unsafe/permit/futex |
| C214 | Happens-Before实战 | 🟢 | 11_并发 | Happens-Before/八大规则 | 内存可见性/重排 |
| C215 | VarHandle替代Unsafe | 🟢 | 11_并发 | VarHandle/变量句柄 | 内存排序/UNSAFE迁移 |
| C216 | CompletableFuture allOf异常 | 🟢 | 11_并发 | CompletableFuture/异步编排 | allOf/异常传播/completeException |
| C217 | Phaser分层协作 | 🟢 | 11_并发 | Phaser/动态注册分层 | 分层Phaser/arrive/awaitAdvance |
| C218 | ThreadLocalRandom原理 | 🟢 | 11_并发 | ThreadLocalRandom | 线程本地种子/CAS/伪共享 |
| C219-235 | JVM/MySQL/Redis/Spring补充 | 🟢 | 各模块 | 各对应知识点 | 见详细文件 |

## IO/NIO专题（20260417）题号C271-C300

| 题号 | 题目简述 | 精度 | Card模块 | Card知识点 | 关联知识点 |
|------|---------|------|---------|-----------|-----------|
| C271 | BIO HTTP服务器→NIO重写 | 🟢 | 19_计算机网络 | IO模型/BIO NIO AIO | BIO瓶颈/Selector/Channel |
| C272 | Selector空轮询Bug | 🟢 | 15_框架/Netty | EpollBug/空轮询修复 | epoll bug/Netty重建 |
| C273 | Reactor模式实现 | 🟢 | 15_框架/Netty | Reactor/主从模型 | 单/多/主从Reactor |
| C274 | Buffer读写flip/clear | 🟢 | 19_计算机网络 | IO模型/NIO核心组件 | flip/clear/compact |
| C275 | 零拷贝transferTo实现 | 🟢 | 19_计算机网络 | IO模型/零拷贝 | FileChannel.transferTo |
| C276 | mmap文件映射 | 🟢 | 19_计算机网络 | IO模型/零拷贝 | MappedByteBuffer/PageFault |
| C277 | Netty ByteBuf泄漏检测 | 🟢 | 15_框架/Netty | ByteBuf/内存管理 | 引用计数/泄漏检测级别 |
| C278 | Netty粘包解码器 | 🟢 | 15_框架/Netty | 粘包半包/解码器 | LengthField/行/Len固定 |
| C279 | Netty空闲检测 | 🟢 | 15_框架/Netty | IdleStateHandler/空闲检测 | 心跳/重连/超时 |
| C280 | 序列化方式选型 | 🟢 | 10_基础 | 序列化/serialVersionUID | Protobuf/JSON/Kryo |
| C281-300 | IO/NIO后续题 | 🟢 | 19_计算机网络/15_框架/Netty | 对应知识点 | 见详细文件 |

## 映射统计

- 总题数：约300+（103个独立文件，C211-C300补充）
- 🟢精确：约180 | 🟡近似：约80 | 🔴关联：约40
- 映射率：约73.3%
- 涉及Card模块：12个

## Card体系缺失建议

- **15_框架/MyBatis**：一级缓存问题、批量操作（当前MyBatis知识地图缺失）
- **15_框架/Dubbo**：泛化调用、超时重试、服务治理（当前Dubbo知识地图偏薄）
- **16_消息**：Kafka消费积压处理、消息可靠性投递实践
- **22_微服务**：Gateway动态路由、Sentinel熔断规则、Dubbo服务治理
- **24_算法**：LFU缓存O(1)实现、前缀树Trie（算法模块整体缺失）

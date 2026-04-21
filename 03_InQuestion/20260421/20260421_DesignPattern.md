# 设计模式 - 代码/方案设计题（C361-C390）

> 第12天 · 2026-04-21 · 共30题

---

## 创建型模式代码题（6题）

### C361. "以下单例实现存在并发问题和序列化漏洞，请分析并给出线程安全且防反射攻击的完整实现。"

```java
public class DatabaseConnectionPool {
    private static DatabaseConnectionPool instance;
    private Connection[] connections;
    
    private DatabaseConnectionPool() {
        connections = new Connection[10];
        // 初始化连接池...
    }
    
    public static DatabaseConnectionPool getInstance() {
        if (instance == null) {
            instance = new DatabaseConnectionPool();
        }
        return instance;
    }
    
    public Connection getConnection() {
        // 返回可用连接...
        return null;
    }
}
```

**问题：**
1. 懒汉式在多线程下为什么不安全？给出至少2种并发场景导致创建多个实例。
2. 给出双重检查锁（DCL）的完整实现，说明volatile为什么必须加在instance字段上。
3. 枚举单例如何防御反射攻击？给出枚举单例的完整代码。

---

### C362. "以下工厂方法实现每新增一种产品就要修改工厂类，违反开闭原则。请用抽象工厂+注册表模式重构，使新增产品零修改工厂。"

```java
public class PaymentFactory {
    public static PaymentService create(String type) {
        switch (type) {
            case "alipay": return new AlipayService();
            case "wechat": return new WechatPayService();
            case "unionpay": return new UnionPayService();
            default: throw new IllegalArgumentException("Unsupported: " + type);
        }
    }
}
```

**问题：**
1. 为什么这个工厂违反开闭原则？
2. 用Map注册表+反射机制实现自动注册工厂，新增产品只需一个注解。
3. 如果每个支付渠道的创建过程很复杂（需要读取配置、初始化SDK），如何结合建造者模式？

---

### C363. "以下建造者模式实现中，builder的参数校验在build()方法中进行，但某些参数之间有依赖关系（如endDate必须大于startDate），如何优雅地实现参数校验？"

```java
public class QueryRequest {
    private final String userId;
    private final LocalDate startDate;
    private final LocalDate endDate;
    private final int page;
    private final int size;
    
    private QueryRequest(Builder builder) {
        this.userId = builder.userId;
        this.startDate = builder.startDate;
        this.endDate = builder.endDate;
        this.page = builder.page;
        this.size = builder.size;
    }
    
    public static class Builder {
        private String userId;
        private LocalDate startDate;
        private LocalDate endDate;
        private int page = 1;
        private int size = 20;
        
        public Builder userId(String userId) { this.userId = userId; return this; }
        public Builder startDate(LocalDate startDate) { this.startDate = startDate; return this; }
        public Builder endDate(LocalDate endDate) { this.endDate = endDate; return this; }
        public Builder page(int page) { this.page = page; return this; }
        public Builder size(int size) { this.size = size; return this; }
        
        public QueryRequest build() {
            // 校验逻辑应该怎么写？
            return new QueryRequest(this);
        }
    }
}
```

**问题：**
1. 当参数有依赖关系时，在build()中校验和使用Java Bean Validation（@Valid）各有什么优劣？
2. 如何实现Builder的"必填参数在构造器中传入，可选参数通过链式设置"的混合模式？
3. 如果QueryRequest被广泛使用（100+处调用），如何做兼容迁移？

---

### C364. "以下代码用简单工厂创建不同类型的日志记录器，但每种日志器的初始化过程完全不同。请用抽象工厂模式重构，使创建过程统一且可扩展。"

```java
public class LoggerFactory {
    public static Logger createLogger(String type) {
        switch (type) {
            case "file":
                Logger logger = new FileLogger();
                ((FileLogger) logger).setFilePath("/var/log/app.log");
                ((FileLogger) logger).setMaxSize(100 * 1024 * 1024);
                return logger;
            case "kafka":
                KafkaLogger kLogger = new KafkaLogger();
                kLogger.setBootstrapServers("kafka:9092");
                kLogger.setTopic("app-logs");
                return kLogger;
            case "elasticsearch":
                ESLogger esLogger = new ESLogger();
                esLogger.setHosts("http://es:9200");
                esLogger.setIndex("app-logs");
                return esLogger;
            default:
                throw new IllegalArgumentException("Unknown logger type");
        }
    }
}
```

**问题：**
1. 给出抽象工厂的完整重构代码，包括LoggerFactory接口和各具体工厂。
2. 如果日志器创建时需要从配置中心（如Nacos）动态读取配置，工厂如何与配置中心集成？
3. 如何结合Spring的依赖注入来管理LoggerFactory？

---

### C365. "以下原型模式的深拷贝实现用序列化方式，性能差且不支持不可序列化的字段。请给出基于BeanUtils、JSON、手工拷贝三种方式的对比和实现。"

```java
public class Order implements Cloneable {
    private String orderId;
    private List<OrderItem> items;
    private User buyer;
    
    @Override
    public Order clone() {
        try {
            return (Order) super.clone(); // 浅拷贝！
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**问题：**
1. super.clone()为什么只做浅拷贝？items和buyer字段会有什么问题？
2. 用Java序列化实现深拷贝的代码是什么？性能瓶颈在哪里？
3. 对比Serialization、Kryo、JSON序列化（Jackson）三种深拷贝方式的性能和限制。

---

### C366. "以下多例模式（Multiton Pattern）实现有并发问题，请修复并说明多例模式的使用场景。"

```java
public class CachePool {
    private static final Map<String, CachePool> instances = new HashMap<>();
    
    private CachePool(String name) {}
    
    public static CachePool getInstance(String name) {
        if (!instances.containsKey(name)) {
            instances.put(name, new CachePool(name));
        }
        return instances.get(name);
    }
}
```

**问题：**
1. 为什么这个多例实现不是线程安全的？给出修复方案。
2. 多例模式与享元模式有什么区别？
3. Spring的Bean作用域（singleton/prototype/request/session）与多例模式有什么关系？

---

## 结构型模式代码题（6题）

### C367. "以下装饰器模式实现日志记录功能，但装饰器嵌套过多时代码难以维护。请用动态代理（JDK Proxy）简化，并分析两种方式的优劣。"

```java
public class LoggingPaymentService implements PaymentService {
    private final PaymentService delegate;
    
    public LoggingPaymentService(PaymentService delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public PayResult pay(PayRequest request) {
        log.info("Pay request: {}", request);
        PayResult result = delegate.pay(request);
        log.info("Pay result: {}", result);
        return result;
    }
}

// 使用：需要日志+限流+监控时
PaymentService service = new MonitoringPaymentService(
    new RateLimitPaymentService(
        new LoggingPaymentService(
            new AlipayServiceImpl()
        )
    )
);
```

**问题：**
1. 用JDK动态代理实现同样的功能，代码量能减少多少？
2. 装饰器模式和动态代理在AOP场景中各有什么优劣？Spring AOP用的是哪种？
3. 如果要按条件决定是否启用某个装饰器（如通过配置控制），装饰器模式和动态代理哪种更灵活？

---

### C368. "以下代理模式实现缓存功能，但缓存更新策略有问题。请修复并扩展为支持TTL和LRU淘汰的完整方案。"

```java
public class CachedUserService implements UserService {
    private final UserService delegate;
    private final Map<Long, User> cache = new HashMap<>();
    
    public CachedUserService(UserService delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public User getUser(Long id) {
        if (cache.containsKey(id)) return cache.get(id);
        User user = delegate.getUser(id);
        cache.put(id, user);
        return user;
    }
}
```

**问题：**
1. 这个缓存代理有什么问题？（无限增长、无过期、无容量限制）
2. 用Caffeine替换HashMap，实现带TTL和LRU的缓存代理。
3. 如何用动态代理实现一个通用的缓存切面（Aspect），对任意Service方法生效？

---

### C369. "以下适配器模式实现新旧系统对接，但适配逻辑中混入了业务逻辑。请分离适配层和业务层，并说明适配器模式与Facade模式的区别。"

```java
public class LegacyOrderAdapter {
    public OrderDTO adapt(LegacyOrder legacy) {
        OrderDTO dto = new OrderDTO();
        dto.setOrderId(legacy.getOrderNo());
        dto.setAmount(legacy.getMoney() / 100.0); // 分转元
        dto.setStatus(convertStatus(legacy.getState()));
        // ... 20多行转换逻辑
        return dto;
    }
    
    private String convertStatus(int state) {
        switch (state) {
            case 1: return "PENDING";
            case 2: return "PAID";
            case 3: return "SHIPPED";
            default: return "UNKNOWN";
        }
    }
}
```

**问题：**
1. 如何用MapStruct自动生成适配器代码，替代手写的转换逻辑？
2. 当新旧系统的字段名相同但语义不同（如旧系统amount含税，新系统不含税）时，适配器如何处理？
3. 适配器模式与Facade模式的本质区别是什么？什么场景下用适配器，什么场景用Facade？

---

### C370. "以下责任链模式实现的参数校验框架中，每个校验器返回不同的错误信息格式。请统一错误处理，并支持校验器的动态编排。"

```java
public interface Validator {
    String validate(Order order);
}

public class AmountValidator implements Validator {
    @Override
    public String validate(Order order) {
        if (order.getAmount() <= 0) return "金额必须大于0";
        return null;
    }
}

public class StockValidator implements Validator {
    @Override
    public String validate(Order order) {
        if (!stockService.check(order.getProductId(), order.getQuantity())) {
            return "库存不足";
        }
        return null;
    }
}

// 使用
List<Validator> validators = Arrays.asList(new AmountValidator(), new StockValidator());
for (Validator v : validators) {
    String error = v.validate(order);
    if (error != null) throw new BusinessException(error);
}
```

**问题：**
1. 当校验规则需要根据业务场景动态组合（如下单场景校验A+B+C，退款场景校验B+D）时，如何实现动态编排？
2. 改为标准的责任链模式（每个Validator决定是否继续传递），对比两种方式的优劣。
3. 与Spring Validation（@Valid + JSR-303注解）相比，手写责任链校验有什么优势和劣势？

---

### C371. "以下组合模式实现的商品分类树不支持排序和过滤，请扩展。"

```java
public abstract class Category {
    protected String name;
    public abstract void display();
    public abstract int count();
}

public class LeafCategory extends Category {
    public LeafCategory(String name) { this.name = name; }
    @Override public void display() { System.out.println(name); }
    @Override public int count() { return 1; }
}

public class CompositeCategory extends Category {
    private List<Category> children = new ArrayList<>();
    public CompositeCategory(String name) { this.name = name; }
    public void add(Category c) { children.add(c); }
    @Override public void display() {
        System.out.println(name);
        for (Category c : children) c.display();
    }
    @Override public int count() {
        return children.stream().mapToInt(Category::count).sum();
    }
}
```

**问题：**
1. 如何添加 `sortBySales()` 方法，使整个分类树按销量排序？
2. 如何实现 `filter(predicate)` —— 过滤掉不符合条件的子分类？
3. 当分类树层级很深（如10级）时，递归遍历可能导致栈溢出，如何改为迭代实现？

---

### C372. "以下桥接模式实现的支付渠道中，支付方式和通知方式混在一起。请分离抽象和实现，并用桥接模式重构。"

```java
public class AlipayService {
    public void pay() { /* Alipay支付逻辑 */ }
    public void notify() { /* Alipay回调通知 */ }
}

public class WechatPayService {
    public void pay() { /* 微信支付逻辑 */ }
    public void notify() { /* 微信回调通知 */ }
}
// 每增加一种支付方式，就要重复写通知逻辑
```

**问题：**
1. 用桥接模式将"支付方式"和"通知方式"分离为两个独立的维度。
2. 当支付方式和通知方式的组合数增多时（3种支付×4种通知=12种组合），桥接模式能减少多少类？
3. 桥接模式与策略模式的区别是什么？什么场景下选桥接，什么场景选策略？

---

## 行为型模式代码题（10题）

### C373. "以下策略模式的策略切换逻辑在业务代码中，导致业务代码与策略管理耦合。请将策略管理抽取为独立的策略工厂，支持运行时动态切换。"

```java
public class PricingService {
    private PricingStrategy strategy;
    
    public void setStrategy(PricingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public BigDecimal calculatePrice(Order order) {
        return strategy.calculate(order);
    }
}

// 业务代码中
if (user.isVip()) {
    pricingService.setStrategy(new VipPricingStrategy());
} else if (order.getAmount() > 1000) {
    pricingService.setStrategy(new BulkPricingStrategy());
} else {
    pricingService.setStrategy(new NormalPricingStrategy());
}
```

**问题：**
1. 用策略工厂+注解实现策略的自动注册和查找。
2. 如果策略的选择逻辑很复杂（需要考虑多个条件），如何结合规则引擎（如Drools或LiteFlow）？
3. 策略对象如果是有状态的（如维护计数器），在多线程环境下会有什么问题？

---

### C374. "以下模板方法模式的子类只需要重写部分步骤，但有些子类把模板方法本身也重写了，破坏了算法骨架。如何防止子类破坏模板方法？"

```java
public abstract class DataExporter {
    // 模板方法：final防止子类重写
    public final void export(String data) {
        String formatted = formatData(data);
        byte[] compressed = compress(formatted);
        send(compressed);
    }
    
    protected abstract String formatData(String data);
    protected abstract byte[] compress(String formatted);
    protected abstract void send(byte[] data);
}
```

**问题：**
1. 为什么模板方法应该加final修饰？如果不加，子类重写模板方法会导致什么问题？
2. 如果某些步骤是可选的（子类可以选择跳过），如何引入钩子方法？
3. 模板方法模式 vs 策略模式：什么场景下选模板方法，什么场景选策略？给出决策标准。

---

### C375. "以下观察者模式的实现中，通知是同步的，如果某个观察者处理慢会阻塞其他观察者。请改为异步通知，并考虑异常处理。"

```java
public class OrderEventPublisher {
    private List<OrderEventListener> listeners = new ArrayList<>();
    
    public void addListener(OrderEventListener listener) {
        listeners.add(listener);
    }
    
    public void publish(Order order) {
        for (OrderEventListener listener : listeners) {
            listener.onOrderCreated(order); // 同步调用！
        }
    }
}
```

**问题：**
1. 用线程池实现异步通知，但保证通知顺序（A在B之前注册，A的通知应该先于B完成）。
2. 如果观察者A处理成功、B处理失败，应该怎么处理？全部回滚还是只回滚B？
3. 对比自研观察者 vs Spring ApplicationEvent vs Guava EventBus，各自如何处理异步和异常？

---

### C376. "以下状态模式的订单状态机缺少非法状态转换的防御。请实现一个类型安全的状态机，确保只有合法的状态转换才能执行。"

```java
public class Order {
    private OrderState state = OrderState.CREATED;
    
    public void pay() {
        if (state == OrderState.CREATED) {
            state = OrderState.PAID;
            // ...
        } else {
            throw new IllegalStateException("Cannot pay in state: " + state);
        }
    }
    
    public void ship() {
        if (state == OrderState.PAID) {
            state = OrderState.SHIPPED;
        } else {
            throw new IllegalStateException("Cannot ship in state: " + state);
        }
    }
    // ... 每个操作都要判断当前状态
}
```

**问题：**
1. 用状态模式重构：每个状态是一个类，只有该状态定义的转换方法才能调用。
2. 设计一个状态转换表（Map<Pair<State, Event>, State>），实现声明式的状态机配置。
3. 如果要支持"从任意状态取消订单"，但不同状态的取消逻辑不同（已支付需要退款、已发货需要拦截物流），如何设计？

---

### C377. "以下命令模式实现的操作历史记录不支持持久化，应用重启后Undo/Redo丢失。请设计持久化方案。"

```java
public interface Command {
    void execute();
    void undo();
}

public class CommandHistory {
    private Deque<Command> undoStack = new ArrayDeque<>();
    private Deque<Command> redoStack = new ArrayDeque<>();
    
    public void execute(Command cmd) {
        cmd.execute();
        undoStack.push(cmd);
        redoStack.clear();
    }
    
    public void undo() {
        Command cmd = undoStack.pop();
        cmd.undo();
        redoStack.push(cmd);
    }
    
    public void redo() {
        Command cmd = redoStack.pop();
        cmd.execute();
        undoStack.push(cmd);
    }
}
```

**问题：**
1. 如何将Command序列化到数据库，实现持久化的Undo/Redo？
2. 如果Command的undo操作需要回滚数据库事务，但数据库不支持嵌套事务，如何处理？
3. 在分布式场景下（多个用户同时操作同一个文档），命令模式如何保证一致性？（OT算法 vs CRDT）

---

### C378. "以下迭代器模式实现用于分页查询，但每次都重新查询数据库。请改为支持流式遍历的惰性迭代器。"

```java
public class UserIterator implements Iterator<User> {
    private List<User> currentPage;
    private int index = 0;
    private int pageNum = 1;
    
    public UserIterator() {
        loadPage();
    }
    
    private void loadPage() {
        currentPage = userDao.findByPage(pageNum, 100);
        index = 0;
    }
    
    @Override
    public boolean hasNext() {
        if (index < currentPage.size()) return true;
        if (currentPage.size() < 100) return false; // 最后一页
        pageNum++;
        loadPage();
        return !currentPage.isEmpty();
    }
    
    @Override
    public User next() {
        return currentPage.get(index++);
    }
}
```

**问题：**
1. 这个迭代器有什么问题？（每次hasNext都可能触发数据库查询）
2. 如何改为：先遍历当前页，遍历完后再加载下一页，避免多余的数据库查询？
3. 用Java 8的Spliterator将这个分页迭代器改为支持Stream API的流式处理。

---

### C379. "以下责任链模式实现的请求处理管道中，如果某个Handler抛出异常，后续Handler不会执行。请设计一个灵活的异常处理策略。"

```java
public abstract class RequestHandler {
    private RequestHandler next;
    
    public RequestHandler setNext(RequestHandler next) {
        this.next = next;
        return next;
    }
    
    public void handle(Request request) {
        doHandle(request);
        if (next != null) next.handle(request);
    }
    
    protected abstract void doHandle(Request request);
}
```

**问题：**
1. 设计三种异常处理策略：①遇到异常立即终止 ②跳过当前Handler继续执行 ③记录异常但继续，最后汇总所有异常
2. 如何在Handler中支持"短路"——某些Handler可以决定不再传递给后续Handler（如权限校验失败直接返回）？
3. 对比这种手写责任链与Spring Interceptor的异常处理机制，Spring是怎么做的？

---

### C380. "以下中介者模式实现的聊天室中，所有消息都经过中介者，中介者成为单点和性能瓶颈。如何优化？"

```java
public class ChatMediator {
    private Map<String, User> users = new ConcurrentHashMap<>();
    
    public void register(User user) { users.put(user.getName(), user); }
    
    public void sendMessage(String from, String to, String message) {
        User target = users.get(to);
        if (target != null) {
            target.receive(from + ": " + message);
        }
    }
}
```

**问题：**
1. 当在线用户达到百万级时，单个ChatMediator如何水平扩展？（提示：按用户ID分片）
2. 如何实现群聊消息的"扇出"优化——用MQ广播替代中介者逐一发送？
3. 中介者模式在什么场景下适合？什么场景下应该用事件驱动（观察者模式）替代？

---

### C381. "以下备忘录模式保存的备忘录对象过大（包含完整的业务对象），导致内存占用高。请优化备忘录的存储策略。"

```java
public class Document {
    private String content;
    private String font;
    private int fontSize;
    // ... 50多个字段
    
    public Memento save() {
        return new Memento(this.content, this.font, this.fontSize /* ...所有字段 */);
    }
    
    public void restore(Memento m) {
        this.content = m.content;
        this.font = m.font;
        this.fontSize = m.fontSize;
        // ... 恢复所有字段
    }
    
    public static class Memento {
        // 所有字段的快照...
    }
}
```

**问题：**
1. 改为增量快照（只保存变化的字段），如何实现？需要引入什么额外的数据结构？
2. 如果要支持100步Undo，但每步的完整快照需要1MB，如何优化内存？（提示：命令模式+逆向执行）
3. 备忘录模式 vs Event Sourcing（事件溯源）：两种方式实现状态恢复各有什么优劣？

---

### C382. "以下访问者模式实现的表达式求值器，在新增表达式类型时需要修改所有访问者。请分析这个'开放-封闭'的权衡，并给出实用的替代方案。"

```java
interface Expression {
    <T> T accept(ExpressionVisitor<T> visitor);
}

class NumberExpr implements Expression {
    double value;
    public <T> T accept(ExpressionVisitor<T> visitor) { return visitor.visit(this); }
}

class AddExpr implements Expression {
    Expression left, right;
    public <T> T accept(ExpressionVisitor<T> visitor) { return visitor.visit(this); }
}

interface ExpressionVisitor<T> {
    T visit(NumberExpr expr);
    T visit(AddExpr expr);
    // 每新增一种Expression，所有Visitor都要加方法
}
```

**问题：**
1. 访问者模式的"双分派"原理是什么？为什么Java不支持多分派？
2. 当Expression类型经常新增（开放扩展），但操作相对稳定时，访问者模式还合适吗？什么模式更好？
3. 用Java 14+的sealed class + pattern matching替代访问者模式，给出代码对比。

---

## 综合设计题（8题）

### C383. "设计一个可扩展的规则引擎：运营可以通过配置界面配置促销规则（满减、折扣、赠品、阶梯价），规则可以自由组合、设置优先级，且支持实时生效。请给出完整的架构设计和核心代码骨架。"

**考察点：** 策略模式+责任链模式+工厂模式+动态配置

---

### C384. "设计一个插件化的数据导出框架：支持导出为Excel/CSV/PDF/JSON，支持自定义列、条件过滤、分页导出，且支持第三方扩展新的导出格式。请给出架构设计和核心接口定义。"

**考察点：** 策略模式+建造者模式+SPI机制+模板方法模式

---

### C385. "设计一个多级缓存框架：支持本地缓存（Caffeine）+ 分布式缓存（Redis）+ 数据库三级缓存，缓存策略可配置，且支持缓存预热、缓存刷新、缓存穿透保护。请给出核心设计。"

**考察点：** 代理模式+策略模式+装饰器模式+观察者模式

---

### C386. "设计一个通用的幂等框架：通过注解声明接口的幂等键（如基于请求参数的MD5），框架自动拦截重复请求。幂等记录存储支持MySQL和Redis两种实现。请给出完整设计。"

**考察点：** 代理模式+策略模式+模板方法模式+AOP

---

### C387. "设计一个灰度发布路由框架：支持按用户ID哈希、用户标签、地理位置、随机比例等多种路由规则，规则可以自由组合（AND/OR），且规则变更实时生效。请给出架构设计。"

**考察点：** 策略模式+责任链模式+组合模式+观察者模式

---

### C388. "设计一个分布式事务的最终一致性方案：基于本地消息表+消息队列，支持消息的可靠发送、消费幂等、失败重试和死信队列。请给出核心设计，并对比Saga、TCC和本地消息表三种方案的适用场景。"

**考察点：** 模板方法模式+策略模式+命令模式+状态模式

---

### C389. "设计一个API版本管理框架：同一个接口支持多个版本（v1/v2/v3），客户端通过Header指定版本，框架自动路由到对应的Handler。不同版本的字段映射通过适配器完成。请给出完整设计。"

**考察点：** 适配器模式+工厂模式+策略模式+代理模式

---

### C390. "设计一个可观测性（Observability）框架：统一管理日志、链路追踪、指标采集，开发者在代码中只需调用统一API，框架负责将数据分发到不同的后端（ELK、Jaeger、Prometheus）。请给出核心设计，并说明用到了哪些设计模式。"

**考察点：** 外观模式+策略模式+观察者模式+装饰器模式+工厂模式

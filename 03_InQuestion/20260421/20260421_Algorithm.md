# 数据结构与算法 - 代码/方案设计题（C331-C360）

> 第11天 · 2026-04-21 · 共30题

---

### C331. "给定以下手写LRU缓存的实现，存在并发安全和性能问题，请分析并优化。"

```java
public class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, V> map = new HashMap<>();
    private final LinkedList<K> list = new LinkedList<>();

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public synchronized V get(K key) {
        if (!map.containsKey(key)) return null;
        list.remove(key);
        list.addFirst(key);
        return map.get(key);
    }

    public synchronized void put(K key, V value) {
        if (map.containsKey(key)) {
            list.remove(key);
            list.addFirst(key);
            map.put(key, value);
        } else {
            if (list.size() >= capacity) {
                K oldest = list.removeLast();
                map.remove(oldest);
            }
            list.addFirst(key);
            map.put(key, value);
        }
    }
}
```

**问题：**
1. LinkedList的remove操作时间复杂度是多少？整体get/put的时间复杂度是多少？
2. synchronized导致所有操作串行，如何提升并发度？
3. 给出基于LinkedHashMap和基于ConcurrentLinkedHashMap两种优化方案的代码。

---

### C332. "以下布隆过滤器实现有误，请找出问题并修复。"

```java
public class BloomFilter {
    private final BitSet bitSet;
    private final int[] seeds = {3, 5, 7, 11, 13};
    private final int size;

    public BloomFilter(int expectedInsertions, double fpp) {
        this.size = (int) (-expectedInsertions * Math.log(fpp) / (Math.log(2) * Math.log(2)));
        this.bitSet = new BitSet(size);
    }

    public void add(String value) {
        for (int seed : seeds) {
            int hash = (value.hashCode() * seed) % size;
            bitSet.set(Math.abs(hash));
        }
    }

    public boolean mightContain(String value) {
        for (int seed : seeds) {
            int hash = (value.hashCode() * seed) % size;
            if (!bitSet.get(Math.abs(hash))) return false;
        }
        return true;
    }
}
```

**问题：**
1. 仅使用String.hashCode()做哈希有什么问题？为什么不同seed相乘不能真正模拟多个独立哈希函数？
2. 整数溢出后取模会产生什么问题？
3. 给出使用MurmurHash3 + 双哈希公式（h1 + i * h2）的正确实现。

---

### C333. "以下跳表实现用于替代有序Map，但查询和插入性能不达标，请分析并优化。"

```java
public class SkipList<K extends Comparable<K>, V> {
    private static final int MAX_LEVEL = 16;
    private final Node head = new Node(null, null, MAX_LEVEL);
    private int level = 1;
    private final Random random = new Random();

    static class Node {
        K key;
        V value;
        Node[] forward;

        Node(K key, V value, int level) {
            this.key = key;
            this.value = value;
            this.forward = new Node[level];
        }
    }

    public V get(K key) {
        Node cur = head;
        for (int i = level - 1; i >= 0; i--) {
            while (cur.forward[i] != null && cur.forward[i].key.compareTo(key) < 0) {
                cur = cur.forward[i];
            }
        }
        if (cur.forward[0] != null && cur.forward[0].key.compareTo(key) == 0) {
            return cur.forward[0].value;
        }
        return null;
    }

    public void put(K key, V value) {
        Node[] update = new Node[MAX_LEVEL];
        Node cur = head;
        for (int i = level - 1; i >= 0; i--) {
            while (cur.forward[i] != null && cur.forward[i].key.compareTo(key) < 0) {
                cur = cur.forward[i];
            }
            update[i] = cur;
        }
        // ... 插入逻辑省略
    }
}
```

**问题：**
1. 当前实现中，每次查询都从最高层开始，但大部分元素的层数远低于MAX_LEVEL。如何优化平均查询路径？
2. 随机层数生成策略（每次50%概率提升一层）有什么问题？如何设置概率使跳表更平衡？
3. 如何实现范围查询（range query）？给出 `entrySet(K from, K to)` 的实现。

---

### C334. "以下一致性哈希实现用于分库分表路由，虚拟节点机制有缺陷，请分析并修复。"

```java
public class ConsistentHash<T> {
    private final TreeMap<Integer, T> ring = new TreeMap<>();
    private final int virtualNodes = 100;

    public void addNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            int hash = (node.toString() + i).hashCode();
            ring.put(hash, node);
        }
    }

    public void removeNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            int hash = (node.toString() + i).hashCode();
            ring.remove(hash);
        }
    }

    public T route(String key) {
        if (ring.isEmpty()) throw new IllegalStateException("No nodes");
        int hash = key.hashCode();
        Map.Entry<Integer, T> entry = ring.ceilingEntry(hash);
        if (entry == null) entry = ring.firstEntry();
        return entry.getValue();
    }
}
```

**问题：**
1. String.hashCode()的分布均匀性如何？为什么可能造成数据倾斜？推荐什么哈希函数？
2. 当节点数量很少（如2个节点）时，即使有100个虚拟节点，仍然可能分布不均。如何评估和改善？
3. 添加一个 `routeNodes(String key, int replicas)` 方法，用于数据冗余备份场景——选出N个不同的物理节点。

---

### C335. "以下前缀树（Trie）实现用于自动补全功能，但内存占用过大且不支持模糊搜索，请优化。"

```java
public class Trie {
    static class TrieNode {
        Map<Character, TrieNode> children = new HashMap<>();
        boolean isEnd;
    }

    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            node.children.putIfAbsent(c, new TrieNode());
            node = node.children.get(c);
        }
        node.isEnd = true;
    }

    public List<String> search(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            if (!node.children.containsKey(c)) return Collections.emptyList();
            node = node.children.get(c);
        }
        List<String> results = new ArrayList<>();
        dfs(node, prefix, results);
        return results;
    }

    private void dfs(TrieNode node, String current, List<String> results) {
        if (node.isEnd) results.add(current);
        for (Map.Entry<Character, TrieNode> entry : node.children.entrySet()) {
            dfs(entry.getValue(), current + entry.getKey(), results);
        }
    }
}
```

**问题：**
1. 每个TrieNode都有一个HashMap，空指针占比高。如何用数组/压缩技术降低内存？
2. 搜索时如果输入有错别字（如"aple"代替"apple"），如何支持模糊匹配？给出编辑距离≤1的前缀搜索。
3. 如何实现Top-K自动补全——每个节点维护热度计数，返回最热门的K个结果？

---

### C336. "以下快速排序实现在特定输入下会栈溢出，请分析原因并给出优化版本。"

```java
public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}

private int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            swap(arr, i, j);
            i++;
        }
    }
    swap(arr, i, high);
    return i;
}
```

**问题：**
1. 当输入数组已有序时，递归深度是多少？为什么会导致栈溢出？
2. 给出三路快排（Dijkstra's 3-way partitioning）的实现，使其对大量重复元素更高效。
3. 如何用"尾递归优化+小数组插入排序"的组合策略进一步降低栈深度？

---

### C337. "以下归并排序实现用于外部排序（大文件排序），但合并阶段内存溢出，请分析并修复。"

```java
public void externalSort(File inputFile, File outputFile, int chunkSize) throws IOException {
    // 阶段1：分割
    List<File> chunks = new ArrayList<>();
    try (BufferedReader reader = new BufferedReader(new FileReader(inputFile))) {
        String line;
        List<String> buffer = new ArrayList<>();
        while ((line = reader.readLine()) != null) {
            buffer.add(line);
            if (buffer.size() >= chunkSize) {
                Collections.sort(buffer);
                File chunk = File.createTempFile("chunk", ".txt");
                writeChunk(chunk, buffer);
                chunks.add(chunk);
                buffer.clear();
            }
        }
        if (!buffer.isEmpty()) {
            Collections.sort(buffer);
            File chunk = File.createTempFile("chunk", ".txt");
            writeChunk(chunk, buffer);
            chunks.add(chunk);
        }
    }

    // 阶段2：合并
    mergeChunks(chunks, outputFile);
}

private void mergeChunks(List<File> chunks, File outputFile) throws IOException {
    // 每个chunk对应一个BufferedReader
    PriorityQueue<FileEntry> pq = new PriorityQueue<>();
    for (File chunk : chunks) {
        BufferedReader reader = new BufferedReader(new FileReader(chunk));
        String line = reader.readLine();
        if (line != null) pq.add(new FileEntry(line, reader));
    }

    try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile))) {
        while (!pq.isEmpty()) {
            FileEntry entry = pq.poll();
            writer.write(entry.line);
            writer.newLine();
            String next = entry.reader.readLine();
            if (next != null) pq.add(new FileEntry(next, entry.reader));
            else entry.reader.close();
        }
    }
}
```

**问题：**
1. 当chunk文件数量达到数千个时，PriorityQueue内存和BufferedReader资源有什么问题？
2. 如何实现多路归并——不是一次性合并所有chunk，而是K路归并后递归合并？
3. 如果每个chunk大小不一致（有的1MB，有的100MB），如何优化合并顺序（Huffman编码思想）？

---

### C338. "以下二分查找变体实现查找第一个等于目标值的元素，但有边界错误，请找出并修复。"

```java
public int findFirst(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] >= target) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    if (left < arr.length && arr[left] == target) return left;
    return -1;
}

public int findLast(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    if (right >= 0 && arr[right] == target) return right;
    return -1;
}
```

**问题：**
1. 当数组为空或所有元素都大于target时，findFirst会抛什么异常？如何防御？
2. 如何用二分查找实现"查找最后一个小于target的元素"和"查找第一个大于等于target的元素"？
3. 在一个旋转有序数组 [4,5,6,7,0,1,2] 中，如何用O(log n)查找target？给出完整实现。

---

### C339. "以下最小堆实现用于Top-K问题，但插入和删除操作有性能问题，请分析并优化。"

```java
public class MinHeap {
    private int[] heap;
    private int size;

    public MinHeap(int capacity) {
        this.heap = new int[capacity];
        this.size = 0;
    }

    public void insert(int val) {
        if (size >= heap.length) throw new RuntimeException("Heap is full");
        heap[size] = val;
        siftUp(size);
        size++;
    }

    private void siftUp(int index) {
        while (index > 0) {
            int parent = (index - 1) / 2;
            if (heap[index] < heap[parent]) {
                swap(index, parent);
                index = parent;
            } else break;
        }
    }

    public int extractMin() {
        if (size == 0) throw new RuntimeException("Heap is empty");
        int min = heap[0];
        heap[0] = heap[size - 1];
        size--;
        siftDown(0);
        return min;
    }

    private void siftDown(int index) {
        while (true) {
            int left = 2 * index + 1;
            int right = 2 * index + 2;
            int smallest = index;
            if (left < size && heap[left] < heap[smallest]) smallest = left;
            if (right < size && heap[right] < heap[smallest]) smallest = right;
            if (smallest != index) {
                swap(index, smallest);
                index = smallest;
            } else break;
        }
    }
}
```

**问题：**
1. insert方法中 `siftUp(size)` 和 `size++` 的顺序是否正确？如果size先自增会怎样？
2. 如何用这个最小堆实现"从10亿个数中找出最大的100个数"？时间和空间复杂度是多少？
3. 给出堆化的两种方式：Floyd建堆法（O(n)）和逐个插入法（O(n log n)），并证明为什么Floyd方法更优。

---

### C340. "以下字典树+优先队列实现热搜词Top-K，但实时更新时性能差，请分析并优化。"

```java
public class HotSearch {
    private final Map<String, Integer> wordCount = new ConcurrentHashMap<>();
    private final int k;

    public HotSearch(int k) {
        this.k = k;
    }

    public void addSearch(String word) {
        wordCount.merge(word, 1, Integer::sum);
    }

    public List<String> topK() {
        return wordCount.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .limit(k)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

**问题：**
1. topK() 每次调用都对全量数据排序，时间复杂度O(n log n)。当n=1000万、QPS=100时如何优化？
2. 给出基于ConcurrentSkipListMap的方案，实现O(log n)插入和O(k)的Top-K查询。
3. 如何实现"最近1小时"的滑动窗口热搜——过期数据自动清理且不影响查询性能？

---

### C341. "以下红黑树插入修复实现有遗漏，请补全并解释旋转规则。"

```java
enum Color { RED, BLACK }

class RBNode {
    int key;
    Color color = Color.RED;
    RBNode left, right, parent;
}

public void insert(RBNode root, RBNode z) {
    // BST标准插入（省略）
    z.color = Color.RED;
    insertFixup(z);
}

private void insertFixup(RBNode z) {
    while (z.parent != null && z.parent.color == Color.RED) {
        if (z.parent == z.parent.parent.left) {
            RBNode uncle = z.parent.parent.right;
            if (uncle != null && uncle.color == Color.RED) {
                // Case 1: 叔叔是红色
                z.parent.color = Color.BLACK;
                uncle.color = Color.BLACK;
                z.parent.parent.color = Color.RED;
                z = z.parent.parent;
            } else {
                // Case 2 & 3: 叔叔是黑色
                if (z == z.parent.right) {
                    // Case 2: 左旋
                    z = z.parent;
                    leftRotate(z);
                }
                // Case 3: 变色 + 右旋
                z.parent.color = Color.BLACK;
                z.parent.parent.color = Color.RED;
                rightRotate(z.parent.parent);
            }
        } else {
            // 对称情况（省略）
        }
    }
    root.color = Color.BLACK; // 假设有root引用
}
```

**问题：**
1. 补全右侧（parent是右子节点）的对称情况处理代码。
2. 为什么新插入的节点必须是红色？插入红色比插入黑色有什么好处？
3. 对照TreeMap的源码，Java中红黑树在删除节点时的修复逻辑比插入复杂多少？核心区别是什么？

---

### C342. "以下B+树插入实现用于理解MySQL InnoDB索引结构，请分析分裂策略。"

```java
class BPlusNode {
    boolean isLeaf;
    List<Integer> keys = new ArrayList<>();
    List<BPlusNode> children = new ArrayList<>(); // 内部节点
    List<Object> values = new ArrayList<>();       // 叶子节点
    BPlusNode next; // 叶子节点链表指针

    void insert(int key, Object value, int order) {
        if (isLeaf) {
            // 找到插入位置并插入
            int pos = findPosition(keys, key);
            keys.add(pos, key);
            values.add(pos, value);
            if (keys.size() > order - 1) {
                splitLeaf(order);
            }
        } else {
            // 找到子节点并递归
            int pos = findPosition(keys, key);
            children.get(pos).insert(key, value, order);
        }
    }

    private void splitLeaf(int order) {
        BPlusNode newLeaf = new BPlusNode();
        newLeaf.isLeaf = true;
        int mid = keys.size() / 2;
        // 分裂逻辑...
    }
}
```

**问题：**
1. 叶子节点分裂时，中间key应该上提到父节点还是留在叶子节点？为什么？
2. InnoDB的B+树页大小固定为16KB，行记录大小不固定时如何处理？给出页内空间管理的思路。
3. 与B树相比，B+树的范围查询为什么更快？叶子节点的链表指针有什么作用？

---

### C343. "以下图的BFS实现用于社交关系中的六度分隔查找，但性能不达标，请优化。"

```java
public int degreesOfSeparation(Map<String, List<String>> graph, String start, String target) {
    if (start.equals(target)) return 0;
    Set<String> visited = new HashSet<>();
    Queue<String> queue = new LinkedList<>();
    queue.offer(start);
    visited.add(start);
    int degree = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        degree++;
        for (int i = 0; i < size; i++) {
            String current = queue.poll();
            for (String friend : graph.getOrDefault(current, Collections.emptyList())) {
                if (friend.equals(target)) return degree;
                if (!visited.contains(friend)) {
                    visited.add(friend);
                    queue.offer(friend);
                }
            }
        }
    }
    return -1;
}
```

**问题：**
1. 当社交网络有10亿节点时，单机BFS无法运行。如何设计分布式BFS？给出MapReduce或Pregel模型下的实现思路。
2. 如何实现双向BFS来加速查找？什么情况下双向BFS比单向BFS快？
3. 如果需要记录最短路径（而不只是距离），如何修改实现使得空间复杂度仍为O(n)？

---

### C344. "以下并查集实现用于朋友圈合并，但find操作没有路径压缩，请优化。"

```java
class UnionFind {
    private int[] parent;
    private int[] rank;

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]); // 路径压缩
        }
        return parent[x];
    }

    public void union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) return;
        if (rank[rootX] < rank[rootY]) parent[rootX] = rootY;
        else if (rank[rootX] > rank[rootY]) parent[rootY] = rootX;
        else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
    }
}
```

**问题：**
1. 路径压缩+按秩合并的时间复杂度上界是多少？为什么实际运行中几乎为O(1)？
2. 如何用并查集判断图中是否存在环？给出算法思路。
3. 实现一个支持"删除操作"的并查集——删除某个元素后，该元素与其他元素的连通关系断开。（提示：不能直接从parent数组中删除）

---

### C345. "以下滑动窗口实现用于计算字符串中不含重复字符的最长子串，但方法可以推广到更多场景，请扩展。"

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> window = new HashMap<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (window.containsKey(c) && window.get(c) >= left) {
            left = window.get(c) + 1;
        }
        window.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**问题：**
1. 用同样的滑动窗口框架，实现"最小覆盖子串"（Minimum Window Substring）——找到s中包含t所有字符的最短子串。
2. 用滑动窗口实现"最多包含K个不同字符的最长子串"。
3. 滑动窗口的时间复杂度为什么是O(n)而不是O(n²)？每个元素最多进出窗口几次？

---

### C346. "以下单调栈实现用于计算股票价格的"下一个更大元素"，但面试官要求扩展到环形数组场景，请实现。"

```java
public int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = i;
        }
        stack.push(i);
    }
    return result;
}
```

**问题：**
1. 如何修改实现，使其处理环形数组（如 [1,2,1] 的结果是 [2,-1,2]）？不使用2n长度的扩展数组，用取模实现。
2. 用单调栈实现"接雨水"（Trapping Rain Water）问题——计算柱状图中能接多少雨水。
3. 用单调栈实现"每日温度"问题——对每个温度，输出需要等多少天才能遇到更温暖的温度。

---

### C347. "以下动态规划实现0-1背包问题，但面试官要求改为空间优化的版本，请分析并改造。"

```java
public int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(dp[i - 1][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][capacity];
}
```

**问题：**
1. 将二维dp优化为一维（滚动数组），并解释为什么内层循环必须逆序遍历。
2. 如果是"完全背包"（每种物品可以用无限次），内层循环应该正序还是逆序？为什么？
3. 如何在求最大价值的同时，还原出具体选了哪些物品？给出回溯方案的代码。

---

### C348. "以下LFU缓存实现使用TreeMap按频率排序，但O(log n)的频率更新有瓶颈，请分析并优化为O(1)。"

```java
public class LFUCache {
    private final int capacity;
    private final Map<Integer, Integer> keyToVal = new HashMap<>();
    private final Map<Integer, Integer> keyToFreq = new HashMap<>();
    private final TreeMap<Integer, LinkedHashSet<Integer>> freqToKeys = new TreeMap<>();
    private int minFreq;

    public LFUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key) {
        if (!keyToVal.containsKey(key)) return -1;
        increaseFreq(key);
        return keyToVal.get(key);
    }

    public void put(int key, int value) {
        if (capacity <= 0) return;
        if (keyToVal.containsKey(key)) {
            keyToVal.put(key, value);
            increaseFreq(key);
            return;
        }
        if (keyToVal.size() >= capacity) {
            evict();
        }
        keyToVal.put(key, value);
        keyToFreq.put(key, 1);
        freqToKeys.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
        minFreq = 1;
    }

    private void increaseFreq(int key) {
        int freq = keyToFreq.get(key);
        freqToKeys.get(freq).remove(key);
        if (freqToKeys.get(freq).isEmpty() && freq == minFreq) {
            minFreq++;
        }
        keyToFreq.put(key, freq + 1);
        freqToKeys.computeIfAbsent(freq + 1, k -> new LinkedHashSet<>()).add(key);
    }

    private void evict() {
        LinkedHashSet<Integer> keys = freqToKeys.get(minFreq);
        int evictKey = keys.iterator().next();
        keys.remove(evictKey);
        keyToVal.remove(evictKey);
        keyToFreq.remove(evictKey);
    }
}
```

**问题：**
1. TreeMap的`computeIfAbsent`和`get(freq).isEmpty()`在并发场景下有什么线程安全问题？
2. 如何用"双向链表嵌套双向链表"的数据结构将所有操作优化到O(1)？描述链表结构。
3. 对比LRU和LFU：在高并发缓存场景中，哪个更适合？各自最差情况下的命中率表现如何？

---

### C349. "以下位运算实现常用技巧，但面试官要求用位运算解决一个实际问题：实现一个支持Add、Delete和GetRandom的数据结构，且三个操作都是O(1)。"

```java
// 题目：设计一个数据结构支持以下操作，时间复杂度均为O(1)：
// - insert(val): 插入元素（已存在则返回false）
// - remove(val): 删除元素（不存在则返回false）
// - getRandom(): 随机返回一个元素
```

**问题：**
1. 为什么HashMap不能单独满足要求？ getRandom为什么不能是O(1)？
2. 给出HashMap + ArrayList的组合方案，重点说明delete操作如何保持O(1)。
3. 如何扩展这个数据结构使其支持"允许重复元素"？ getRandom应按频率返回。

---

### C350. "以下Redis Cluster的哈希槽路由用Java模拟实现，但需要支持在线扩容，请设计方案。"

```java
public class SlotRouter {
    // 16384个槽位，每个槽映射到一个节点
    private final int[] slotToNode = new int[16384];

    public SlotRouter() {
        // 初始3个节点，平均分配
        for (int i = 0; i < 16384; i++) {
            slotToNode[i] = i % 3;
        }
    }

    public int route(String key) {
        int slot = crc16(key) % 16384;
        return slotToNode[slot];
    }

    private int crc16(String key) {
        // CRC16实现省略
        return 0;
    }
}
```

**问题：**
1. 从3个节点扩容到5个节点时，如何最小化数据迁移量？给出迁移策略和代码。
2. 迁移过程中如何保证请求的正确性——哪些key应该访问旧节点，哪些访问新节点？
3. 如何实现"询问式迁移"（ASK重定向）？客户端和节点各需要什么改动？

---

### C351. "以下哈希表实现（开放寻址法）有聚集问题，请分析并用更好的探测策略优化。"

```java
public class OpenAddressHashMap<K, V> {
    private final Entry[] table;
    private final int capacity;
    private int size;

    static class Entry<K, V> {
        K key;
        V value;
        boolean deleted;

        Entry(K key, V value) { this.key = key; this.value = value; }
    }

    public OpenAddressHashMap(int capacity) {
        this.capacity = capacity;
        this.table = new Entry[capacity];
    }

    public void put(K key, V value) {
        int index = key.hashCode() % capacity;
        int i = 0;
        while (i < capacity) {
            int idx = (index + i) % capacity; // 线性探测
            if (table[idx] == null || table[idx].deleted) {
                table[idx] = new Entry<>(key, value);
                size++;
                return;
            }
            if (table[idx].key.equals(key)) {
                table[idx].value = value;
                return;
            }
            i++;
        }
    }

    public V get(K key) {
        int index = key.hashCode() % capacity;
        for (int i = 0; i < capacity; i++) {
            int idx = (index + i) % capacity;
            if (table[idx] == null) return null;
            if (table[idx].key.equals(key)) return table[idx].value;
        }
        return null;
    }
}
```

**问题：**
1. 线性探测为什么会产生一次聚集和二次聚集？
2. 分别用二次探测（quadratic probing）和双重哈希（double hashing）替换线性探测，给出实现。
3. ThreadLocal在JDK7中用的就是开放寻址法（线性探测），JDK8改为Entry数组+链表。为什么？

---

### C352. "以下Dijkstra最短路径实现使用优先队列，但面试官要求处理负权边，请分析并给出Bellman-Ford方案。"

```java
public int[] dijkstra(Map<Integer, List<int[]>> graph, int src, int n) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
    pq.offer(new int[]{src, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], d = cur[1];
        if (d > dist[u]) continue;
        for (int[] edge : graph.getOrDefault(u, Collections.emptyList())) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.offer(new int[]{v, dist[v]});
            }
        }
    }
    return dist;
}
```

**问题：**
1. Dijkstra为什么不能处理负权边？给出一个反例说明。
2. 给出Bellman-Ford算法的实现，并分析其时间复杂度O(VE)。
3. Bellman-Ford如何检测负权环？在什么应用场景需要检测负权环（如金融、网络路由）？

---

### C353. "以下字符串匹配实现使用暴力法，面试官要求实现KMP算法并分析。"

```java
public int bruteForce(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    for (int i = 0; i <= n - m; i++) {
        int j;
        for (j = 0; j < m; j++) {
            if (text.charAt(i + j) != pattern.charAt(j)) break;
        }
        if (j == m) return i;
    }
    return -1;
}
```

**问题：**
1. 暴力法最坏时间复杂度是多少？给出一个最坏case（如text="AAAA...AAB", pattern="AAAAB"）。
2. 实现KMP算法，重点实现next数组（最长公共前后缀）的构建过程。
3. 对比KMP、Rabin-Karp（滚动哈希）、BM（Boyer-Moore）三种算法的适用场景和平均/最坏时间复杂度。

---

### C354. "以下限流算法使用固定窗口计数器，面试官要求分别实现滑动窗口和令牌桶。"

```java
public class FixedWindowRateLimiter {
    private final int limit;
    private final long windowSizeMs;
    private final Map<String, WindowCount> windows = new ConcurrentHashMap<>();

    public FixedWindowRateLimiter(int limit, long windowSizeMs) {
        this.limit = limit;
        this.windowSizeMs = windowSizeMs;
    }

    public boolean allow(String key) {
        long windowId = System.currentTimeMillis() / windowSizeMs;
        windows.compute(key, (k, wc) -> {
            if (wc == null || wc.windowId != windowId) {
                return new WindowCount(windowId, 1);
            }
            wc.count++;
            return wc;
        });
        return windows.get(key).count <= limit;
    }

    static class WindowCount {
        long windowId;
        int count;
        WindowCount(long windowId, int count) {
            this.windowId = windowId;
            this.count = count;
        }
    }
}
```

**问题：**
1. 固定窗口的"临界突发"问题是什么？举例说明（如限流100次/分钟，在00:00:59到00:01:01之间可以瞬间通过200次）。
2. 实现滑动窗口限流——将每个窗口分成多个小格子，用圆形数组存储。
3. 实现令牌桶算法（Token Bucket），允许一定程度的突发流量，给出完整实现。

---

### C355. "以下递归实现计算斐波那契数列，面试官要求用多种方法优化。"

```java
public int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

**问题：**
1. 递归版的时间复杂度和空间复杂度分别是多少？为什么？
2. 分别给出以下三种优化的实现：①自底向上DP（O(n)时间O(1)空间） ②矩阵快速幂（O(log n)时间） ③通项公式法（黄金比例）
3. 当n非常大（如n=10^18）时，要求结果对10^9+7取模，矩阵快速幂方法如何处理？

---

### C356. "以下拓扑排序实现用于任务依赖管理，但需要检测环路并支持增量添加任务。"

```java
public List<Integer> topologicalSort(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());

    for (int[] pre : prerequisites) {
        graph.get(pre[1]).add(pre[0]);
        inDegree[pre[0]]++;
    }

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) queue.offer(i);
    }

    List<Integer> result = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result.add(node);
        for (int neighbor : graph.get(node)) {
            if (--inDegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }
    return result.size() == numCourses ? result : Collections.emptyList();
}
```

**问题：**
1. 当result.size() < numCourses时，说明存在环路。如何找出具体是哪些节点构成了环？
2. 如果需求变为"每次添加新任务和依赖关系时，动态判断是否会产生环"（而不是每次全量计算），如何设计增量拓扑排序？
3. 如果任务有执行时间，如何用拓扑排序+优先队列（最长路径/关键路径）计算最短完成时间？

---

### C357. "以下分治法实现归并两个有序数组的中位数查找，但面试官要求推广到任意数量的有序数组。"

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int m = nums1.length, n = nums2.length;
    if (m > n) return findMedianSortedArrays(nums2, nums1);
    int low = 0, high = m;
    while (low <= high) {
        int partition1 = (low + high) / 2;
        int partition2 = (m + n + 1) / 2 - partition1;

        int maxLeft1 = partition1 == 0 ? Integer.MIN_VALUE : nums1[partition1 - 1];
        int minRight1 = partition1 == m ? Integer.MAX_VALUE : nums1[partition1];
        int maxLeft2 = partition2 == 0 ? Integer.MIN_VALUE : nums2[partition2 - 1];
        int minRight2 = partition2 == n ? Integer.MAX_VALUE : nums2[partition2];

        if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
            if ((m + n) % 2 == 0) return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0;
            return Math.max(maxLeft1, maxLeft2);
        } else if (maxLeft1 > minRight2) {
            high = partition1 - 1;
        } else {
            low = partition1 + 1;
        }
    }
    throw new IllegalArgumentException();
}
```

**问题：**
1. 解释二分搜索的partition含义——为什么partition1 + partition2 = (m+n+1)/2？
2. 如何将这个算法扩展到K个有序数组的中位数查找？给出思路（提示：每次排除K/2个元素）。
3. 在分布式系统中，如何从N个分片（每个分片数据有序）中高效查找全局中位数？给出Reduce阶段的设计。

---

### C358. "以下贪心算法实现区间调度（活动选择问题），面试官要求证明贪心选择的正确性。"

```java
public int maxActivities(int[][] activities) {
    // activities[i] = {start, end}
    Arrays.sort(activities, Comparator.comparingInt(a -> a[1])); // 按结束时间排序
    int count = 0;
    int lastEnd = Integer.MIN_VALUE;
    for (int[] act : activities) {
        if (act[0] >= lastEnd) {
            count++;
            lastEnd = act[1];
        }
    }
    return count;
}
```

**问题：**
1. 用"交换论证法"（Exchange Argument）证明"按结束时间排序贪心"是最优的。
2. 如果每个活动有一个权重（价值），问题变为"加权区间调度"，贪心还适用吗？给出DP解法。
3. 有N个会议室和M个会议（每个会议有起止时间），如何用贪心算法分配最少的会议室？（提示：最小堆）

---

### C359. "以下倒排索引用于全文搜索，但需要支持布尔查询（AND/OR/NOT），请扩展设计。"

```java
public class InvertedIndex {
    private final Map<String, Set<Integer>> index = new HashMap<>();

    public void addDocument(int docId, String content) {
        String[] words = content.toLowerCase().split("\\s+");
        for (String word : words) {
            index.computeIfAbsent(word, k -> new HashSet<>()).add(docId);
        }
    }

    public Set<Integer> search(String word) {
        return index.getOrDefault(word.toLowerCase(), Collections.emptySet());
    }
}
```

**问题：**
1. 实现 `searchAnd(String term1, String term2)` —— 返回同时包含两个term的文档集合。如何高效计算交集？
2. 实现 `searchOr(String term1, String term2)` 和 `searchNot(String term)`。
3. 对于大规模索引（10亿文档），Set<Integer>的内存占用太大。如何用Roaring Bitmap替代？解释Roaring Bitmap的容器类型（Array Container / Run Container / Bitmap Container）。

---

### C360. "综合设计题：设计一个支持范围查询的分布式计数器服务。"

**需求：**
- 支持数十亿个计数器，每个计数器有唯一key
- 支持 `increment(key, delta)` —— 原子增加
- 支持 `get(key)` —— 获取当前值
- 支持 `rangeQuery(prefix, start, end)` —— 获取key在某个前缀下、时间戳在[start, end]范围内的聚合值
- QPS：100万 increment + 10万 rangeQuery

**问题：**
1. 底层存储选什么？为什么Redis Sorted Set + HyperLogLog不适合这个场景？
2. 数据分片策略——按key hash分片 vs 按时间范围分片 vs 二级索引（key+时间），各自的优劣？
3. 给出基于LSM-Tree的存储引擎（如RocksDB/Cassandra）的实现方案，重点说明compaction策略如何影响rangeQuery性能。

# Java笔记3.0


<!--more-->
# Java 笔记3.0
## 八、常用类与集合框架：Java 开发的“工具箱”（续）
### 二、集合框架：Java 存储数据的“容器”（续）
#### 3. List 接口：有序可重复集合
List 接口的核心是“有序、可重复、支持索引访问”，常用实现类为 `ArrayList` 和 `LinkedList`，两者底层结构不同，导致性能差异显著。

##### （1）ArrayList：动态数组实现（查询优先）
- **底层结构**：基于**动态数组**（JDK 8 无初始容量时默认空数组，第一次添加元素时扩容为 10；JDK 7 初始容量默认 10），数组满时自动扩容（扩容机制：JDK 8 及后为原容量的 1.5 倍，JDK 7 为 1.5 倍+1）。
- **核心特点**：
  - 优势：支持通过索引（`get(int index)`）直接访问元素，**查询效率高**（时间复杂度 O(1)）；
  - 劣势：增删元素时需移动数组元素（如在中间插入元素，需后移后续所有元素），**增删效率低**（时间复杂度 O(n)）；
  - 线程安全：线程不安全（多线程并发修改可能抛出 `ConcurrentModificationException`）。
- **常用方法**（除 Collection 通用方法外）：
  - `E get(int index)`：通过索引获取元素；
  - `E set(int index, E element)`：替换指定索引的元素；
  - `void add(int index, E element)`：在指定索引插入元素；
  - `E remove(int index)`：删除指定索引的元素；
  - `int indexOf(Object o)`：返回元素首次出现的索引（无则返回 -1）。

##### 代码示例（ArrayList 用法）
```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListDemo {
    public static void main(String[] args) {
        // 1. 创建 ArrayList 集合（泛型指定存储 String 类型）
        List<String> list = new ArrayList<>();

        // 2. 添加元素
        list.add("Java");
        list.add("Python");
        list.add("C++");
        System.out.println("初始集合：" + list); // 输出：[Java, Python, C++]

        // 3. 索引访问
        String first = list.get(0);
        System.out.println("索引 0 的元素：" + first); // 输出：Java

        // 4. 插入元素（索引 1 位置）
        list.add(1, "Go");
        System.out.println("插入后集合：" + list); // 输出：[Java, Go, Python, C++]

        // 5. 修改元素（索引 2 位置）
        list.set(2, "JavaScript");
        System.out.println("修改后集合：" + list); // 输出：[Java, Go, JavaScript, C++]

        // 6. 删除元素（索引 3 位置）
        list.remove(3);
        System.out.println("删除后集合：" + list); // 输出：[Java, Go, JavaScript]

        // 7. 遍历集合（三种方式）
        // 方式 1：普通 for 循环（利用索引）
        System.out.println("普通 for 循环遍历：");
        for (int i = 0; i < list.size(); i++) {
            System.out.print(list.get(i) + " "); // 输出：Java Go JavaScript
        }

        // 方式 2：增强 for 循环（foreach）
        System.out.println("\n增强 for 循环遍历：");
        for (String lang : list) {
            System.out.print(lang + " ");
        }

        // 方式 3：迭代器（Iterator）
        System.out.println("\n迭代器遍历：");
        java.util.Iterator<String> it = list.iterator();
        while (it.hasNext()) { // 判断是否有下一个元素
            String lang = it.next(); // 获取下一个元素
            System.out.print(lang + " ");
        }
    }
}
```


##### （2）LinkedList：双向链表实现（增删优先）
- **底层结构**：基于**双向链表**（每个节点包含 `prev`（前驱）、`element`（元素）、`next`（后继）），无需连续内存空间。
- **核心特点**：
  - 优势：增删元素时只需修改链表节点的 `prev` 和 `next` 指针，**增删效率高**（尤其是首尾操作，时间复杂度 O(1)）；
  - 劣势：查询元素需从链表头/尾遍历，**查询效率低**（时间复杂度 O(n)）；
  - 线程安全：线程不安全；
  - 额外功能：实现 `Deque` 接口，可作为队列（FIFO）或栈（LIFO）使用。
- **常用方法**（除 List 通用方法外，新增首尾操作）：
  - `void addFirst(E e)`：在链表头部添加元素；
  - `void addLast(E e)`：在链表尾部添加元素（等同于 `add(E e)`）；
  - `E getFirst()`：获取头部元素；
  - `E getLast()`：获取尾部元素；
  - `E removeFirst()`：删除头部元素；
  - `E removeLast()`：删除尾部元素。

##### 代码示例（LinkedList 作为队列和栈）
```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Deque;

public class LinkedListDemo {
    public static void main(String[] args) {
        // 1. 作为队列（先进先出，FIFO）
        Queue<String> queue = new LinkedList<>();
        queue.offer("A"); // 入队（尾部添加）
        queue.offer("B");
        queue.offer("C");
        System.out.println("队列初始：" + queue); // 输出：[A, B, C]

        String pollElem = queue.poll(); // 出队（头部删除）
        System.out.println("出队元素：" + pollElem); // 输出：A
        System.out.println("出队后队列：" + queue); // 输出：[B, C]

        // 2. 作为栈（先进后出，LIFO）
        Deque<String> stack = new LinkedList<>();
        stack.push("X"); // 入栈（头部添加）
        stack.push("Y");
        stack.push("Z");
        System.out.println("栈初始：" + stack); // 输出：[Z, Y, X]

        String popElem = stack.pop(); // 出栈（头部删除）
        System.out.println("出栈元素：" + popElem); // 输出：Z
        System.out.println("出栈后栈：" + stack); // 输出：[Y, X]
    }
}
```


##### （3）ArrayList vs LinkedList 对比选择
| 对比维度       | ArrayList                          | LinkedList                        |
|----------------|------------------------------------|-----------------------------------|
| 底层结构       | 动态数组                           | 双向链表                          |
| 查询效率       | 高（O(1)，索引访问）               | 低（O(n)，遍历访问）              |
| 增删效率       | 低（O(n)，移动元素）               | 高（O(1)，首尾操作；O(n)，中间操作）|
| 内存占用       | 可能有冗余（数组扩容预留空间）     | 无冗余（节点按需创建）            |
| 适用场景       | 频繁查询、少量增删（如数据展示）   | 频繁增删、少量查询（如队列/栈）   |


#### 4. Set 接口：无序不可重复集合
Set 接口的核心是“无序（存储顺序与添加顺序无关）、不可重复（元素唯一）、无索引”，常用实现类为 `HashSet`、`LinkedHashSet`、`TreeSet`。

##### （1）HashSet：哈希表实现（无序去重）
- **底层结构**：JDK 8 前为“数组+链表”，JDK 8 及后为“数组+链表/红黑树”（当链表长度超过 8 且数组容量≥64 时，链表转为红黑树，提升查询效率）。
- **去重原理**：基于 `hashCode()` 和 `equals()` 方法，核心逻辑：
  1. 新增元素时，先调用元素的 `hashCode()` 方法，计算哈希值，确定在数组中的存储位置；
  2. 若该位置为空，直接存入元素；
  3. 若该位置不为空，调用该位置已有元素的 `equals()` 方法与新增元素比较：
     - 若 `equals()` 返回 `true`，视为重复元素，不存入；
     - 若 `equals()` 返回 `false`，存入链表（或红黑树）。
- **重要约定**：若重写 `equals()` 方法，必须同时重写 `hashCode()` 方法，确保“相等的对象必须有相等的哈希值”（否则会导致 HashSet 无法去重）。
- **核心特点**：无序、不可重复、查询效率高（平均 O(1)）、线程不安全。

##### 代码示例（HashSet 去重与重写方法）
```java
import java.util.HashSet;
import java.util.Set;

// 自定义实体类（需重写 equals() 和 hashCode() 实现去重）
class Student {
    private String id;
    private String name;

    public Student(String id, String name) {
        this.id = id;
        this.name = name;
    }

    // 重写 equals()：id 相同则视为同一学生
    @Override
    public boolean equals(Object o) {
        if (this == o) return true; // 同一对象，直接相等
        if (o == null || getClass() != o.getClass()) return false; // 类型不同，不相等
        Student student = (Student) o;
        return id.equals(student.id); // 比较 id
    }

    // 重写 hashCode()：id 相同则哈希值相同
    @Override
    public int hashCode() {
        return id.hashCode(); // 基于 id 计算哈希值
    }

    // 重写 toString()：便于打印
    @Override
    public String toString() {
        return "Student{id='" + id + "', name='" + name + "'}";
    }
}

public class HashSetDemo {
    public static void main(String[] args) {
        Set<Student> studentSet = new HashSet<>();

        // 添加学生（id=001 的两个学生视为重复）
        studentSet.add(new Student("001", "张三"));
        studentSet.add(new Student("002", "李四"));
        studentSet.add(new Student("001", "张三（重复）")); // 重复元素，不存入

        // 遍历 Set（无序，无索引，只能用 foreach 或迭代器）
        System.out.println("HashSet 中的学生：");
        for (Student student : studentSet) {
            System.out.println(student);
        }
        // 输出结果（顺序可能不同，但仅两个学生）：
        // Student{id='001', name='张三'}
        // Student{id='002', name='李四'}
    }
}
```


##### （2）LinkedHashSet：哈希表+链表（有序去重）
- **底层结构**：在 HashSet 基础上，增加了一条“双向链表”，记录元素的**插入顺序**。
- **核心特点**：
  - 有序：遍历顺序与元素添加顺序一致；
  - 去重：同 HashSet，基于 `hashCode()` 和 `equals()`；
  - 效率：查询效率略低于 HashSet（需维护链表），但高于 TreeSet；
  - 线程不安全。
- **适用场景**：需要“去重且保留插入顺序”的场景（如日志记录、历史记录）。

##### 代码示例（LinkedHashSet 有序性）
```java
import java.util.LinkedHashSet;
import java.util.Set;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<>();
        set.add("Apple");
        set.add("Banana");
        set.add("Orange");
        set.add("Apple"); // 重复元素，不存入

        // 遍历：顺序与添加顺序一致
        System.out.println("LinkedHashSet 遍历：");
        for (String fruit : set) {
            System.out.print(fruit + " "); // 输出：Apple Banana Orange
        }
    }
}
```


##### （3）TreeSet：红黑树实现（排序去重）
- **底层结构**：基于**红黑树**（一种自平衡的二叉搜索树），确保元素按“自然顺序”或“定制顺序”排列。
- **去重与排序原理**：
  - 排序：通过比较元素大小确定存储位置（自然排序需元素实现 `Comparable` 接口，定制排序需传入 `Comparator` 比较器）；
  - 去重：若比较结果为 0（视为元素相等），则不存入。
- **核心特点**：有序（排序后的顺序）、不可重复、查询效率高（O(log n)）、线程不安全。
- **适用场景**：需要“去重且排序”的场景（如成绩排名、按时间排序的任务列表）。

##### 代码示例（TreeSet 自然排序与定制排序）
```java
import java.util.TreeSet;
import java.util.Comparator;
import java.util.Set;

// 1. 自然排序：实现 Comparable 接口
class Score implements Comparable<Score> {
    private String name;
    private int score;

    public Score(String name, int score) {
        this.name = name;
        this.score = score;
    }

    // 重写 compareTo()：按分数降序排序（分数相同则按姓名升序）
    @Override
    public int compareTo(Score o) {
        if (this.score != o.score) {
            return o.score - this.score; // 降序（o.score - this.score）
        } else {
            return this.name.compareTo(o.name); // 姓名升序
        }
    }

    @Override
    public String toString() {
        return "Score{name='" + name + "', score=" + score + "}";
    }
}

public class TreeSetDemo {
    public static void main(String[] args) {
        // 方式 1：自然排序（依赖 Score 实现的 Comparable 接口）
        Set<Score> scoreSet1 = new TreeSet<>();
        scoreSet1.add(new Score("张三", 95));
        scoreSet1.add(new Score("李四", 88));
        scoreSet1.add(new Score("王五", 95)); // 分数相同，按姓名排序
        System.out.println("自然排序（按分数降序）：");
        for (Score s : scoreSet1) {
            System.out.println(s);
        }
        // 输出：
        // Score{name='王五', score=95}
        // Score{name='张三', score=95}
        // Score{name='李四', score=88}

        // 方式 2：定制排序（传入 Comparator 比较器，不依赖元素类）
        Set<Score> scoreSet2 = new TreeSet<>(new Comparator<Score>() {
            // 按分数升序排序
            @Override
            public int compare(Score o1, Score o2) {
                return o1.score - o2.score;
            }
        });
        scoreSet2.add(new Score("张三", 95));
        scoreSet2.add(new Score("李四", 88));
        scoreSet2.add(new Score("王五", 95));
        System.out.println("\n定制排序（按分数升序）：");
        for (Score s : scoreSet2) {
            System.out.println(s);
        }
        // 输出：
        // Score{name='李四', score=88}
        // Score{name='张三', score=95}
        // Score{name='王五', score=95}
    }
}
```


#### 5. Map 接口：双列集合（键值对存储）
Map 接口的核心是“存储键值对（key-value）、key 唯一、value 可重复”，常用实现类为 `HashMap`、`LinkedHashMap`、`TreeMap`。

##### （1）HashMap：哈希表实现（无序键值对）
- **底层结构**：JDK 8 前为“数组+链表”，JDK 8 及后为“数组（哈希桶）+链表/红黑树”（链表长度＞8 且数组容量≥64 时转红黑树）。
- **key 特性**：
  - 唯一性：基于 `hashCode()` 和 `equals()` 去重（同 HashSet）；
  - 允许为 `null`：但仅能有一个 `null` key（重复添加会覆盖 value）；
  - 无序：key 的存储顺序与添加顺序无关。
- **value 特性**：可重复、可为 `null`。
- **核心特点**：查询/添加/删除效率高（平均 O(1)）、线程不安全、初始化容量为 16（默认）、负载因子为 0.75（当元素个数超过“容量×负载因子”时，数组扩容为原来的 2 倍）。
- **常用方法**：
  - `V put(K key, V value)`：添加键值对（key 存在则覆盖 value，返回旧 value；不存在则返回 null）；
  - `V get(Object key)`：通过 key 获取 value（key 不存在则返回 null）；
  - `V remove(Object key)`：通过 key 删除键值对，返回删除的 value；
  - `boolean containsKey(Object key)`：判断是否包含指定 key；
  - `boolean containsValue(Object value)`：判断是否包含指定 value；
  - `Set<K> keySet()`：获取所有 key 的 Set 集合（用于遍历 key）；
  - `Set<Map.Entry<K, V>> entrySet()`：获取所有键值对的 Set 集合（用于遍历键值对）；
  - `int size()`：返回键值对个数；
  - `void clear()`：清空所有键值对。

##### 代码示例（HashMap 常用操作与遍历）
```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class HashMapDemo {
    public static void main(String[] args) {
        // 1. 创建 HashMap（泛型指定 key 为 String，value 为 Integer）
        Map<String, Integer> scoreMap = new HashMap<>();

        // 2. 添加键值对
        scoreMap.put("张三", 95);
        scoreMap.put("李四", 88);
        scoreMap.put("王五", 92);
        scoreMap.put("张三", 98); // key 重复，覆盖 value（旧值 95，新值 98）
        scoreMap.put(null, 100); // 允许 null key
        System.out.println("HashMap 初始：" + scoreMap);
        // 输出（顺序可能不同）：{null=100, 李四=88, 张三=98, 王五=92}

        // 3. 获取 value
        Integer zhangScore = scoreMap.get("张三");
        System.out.println("张三的分数：" + zhangScore); // 输出：98

        // 4. 判断是否包含 key/value
        boolean hasLi = scoreMap.containsKey("李四");
        boolean has92 = scoreMap.containsValue(92);
        System.out.println("是否包含 key '李四'：" + hasLi); // 输出：true
        System.out.println("是否包含 value 92：" + has92); // 输出：true

        // 5. 遍历 HashMap（三种方式）
        // 方式 1：遍历 key，再通过 key 获取 value（keySet()）
        System.out.println("\n方式 1：遍历 key -> value");
        Set<String> keys = scoreMap.keySet();
        for (String key : keys) {
            Integer value = scoreMap.get(key);
            System.out.println(key + "：" + value);
        }

        // 方式 2：直接遍历键值对（entrySet()，推荐，效率高）
        System.out.println("\n方式 2：遍历 entrySet（键值对）");
        Set<Map.Entry<String, Integer>> entries = scoreMap.entrySet();
        for (Map.Entry<String, Integer> entry : entries) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println(key + "：" + value);
        }

        // 方式 3：Lambda 表达式遍历（Java 8+）
        System.out.println("\n方式 3：Lambda 遍历");
        scoreMap.forEach((key, value) -> System.out.println(key + "：" + value));

        // 6. 删除键值对
        scoreMap.remove(null);
        System.out.println("\n删除 null key 后：" + scoreMap);
        // 输出（顺序可能不同）：{李四=88, 张三=98, 王五=92}
    }
}
```


##### （2）LinkedHashMap：哈希表+链表（有序键值对）
- **底层结构**：在 HashMap 基础上，增加一条“双向链表”，记录键值对的**插入顺序**或**访问顺序**（默认插入顺序）。
- **核心特点**：
  - 有序：遍历顺序与插入顺序一致（或访问顺序，需构造时指定 `accessOrder=true`）；
  - key/value 特性：同 HashMap（允许 null key/value，key 唯一）；
  - 效率：略低于 HashMap（需维护链表）；
  - 线程不安全。
- **适用场景**：需要“保留键值对顺序”的场景（如缓存、最近访问记录）。

##### 代码示例（LinkedHashMap 有序性与访问顺序）
```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapDemo {
    public static void main(String[] args) {
        // 1. 默认：按插入顺序排序
        Map<String, String> insertOrderMap = new LinkedHashMap<>();
        insertOrderMap.put("A", "1");
        insertOrderMap.put("B", "2");
        insertOrderMap.put("C", "3");
        System.out.println("插入顺序遍历：" + insertOrderMap); // 输出：{A=1, B=2, C=3}

        // 2. 按访问顺序排序（构造时指定 accessOrder=true）
        // 访问顺序：调用 get() 或 put() 会将元素移到链表尾部
        Map<String, String> accessOrderMap = new LinkedHashMap<>(16, 0.75f, true);
        accessOrderMap.put("A", "1");
        accessOrderMap.put("B", "2");
        accessOrderMap.put("C", "3");

        accessOrderMap.get("A"); // 访问 A，A 移到尾部
        accessOrderMap.put("B", "22"); // 覆盖 B，B 移到尾部
        System.out.println("访问顺序遍历：" + accessOrderMap); // 输出：{C=3, A=1, B=22}
    }
}
```


##### （3）TreeMap：红黑树实现（排序键值对）
- **底层结构**：基于红黑树，key 按“自然顺序”或“定制顺序”排列。
- **key 特性**：
  - 排序：同 TreeSet（需实现 `Comparable` 或传入 `Comparator`）；
  - 唯一性：比较结果为 0 则视为 key 重复，覆盖 value；
  - 不允许 null key（会抛出 `NullPointerException`）；
  - value 可重复、可为 null。
- **核心特点**：有序（排序后的顺序）、查询效率高（O(log n)）、线程不安全。
- **适用场景**：需要“按 key 排序”的键值对场景（如按日期排序的订单记录、按 ID 排序的用户信息）。

##### 代码示例（TreeMap 定制排序）
```java
import java.util.TreeMap;
import java.util.Comparator;
import java.util.Map;

public class TreeMapDemo {
    public static void main(String[] args) {
        // 定制排序：按 key 的长度升序（key 为 String 类型）
        Map<String, Integer> treeMap = new TreeMap<>(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.length() - o2.length(); // 按 key 长度升序
            }
        });

        treeMap.put("Apple", 5);
        treeMap.put("Banana", 6);
        treeMap.put("Pear", 4);
        treeMap.put("Orange", 6); // key 长度 6，与 Banana 相同，覆盖 value（若 key 不同但长度相同，可共存）

        // 遍历：按 key 长度升序
        System.out.println("TreeMap 遍历（按 key 长度升序）：");
        for (Map.Entry<String, Integer> entry : treeMap.entrySet()) {
            System.out.println(entry.getKey() + "（长度：" + entry.getKey().length() + "）：" + entry.getValue());
        }
        // 输出：
        // Pear（长度：4）：4
        // Apple（长度：5）：5
        // Banana（长度：6）：6
        // Orange（长度：6）：6
    }
}
```


#### 6. 集合的线程安全问题与解决方案
大部分集合（ArrayList、LinkedList、HashMap、HashSet 等）都是线程不安全的，多线程并发修改时可能抛出 `ConcurrentModificationException`（并发修改异常）。解决方案主要有以下 3 种：

##### （1）使用 Collections 工具类的同步方法
`java.util.Collections` 提供 `synchronizedXXX()` 方法，将线程不安全的集合包装为线程安全的集合：
- `Collections.synchronizedList(List<T> list)`：包装为线程安全的 List；
- `Collections.synchronizedSet(Set<T> set)`：包装为线程安全的 Set；
- `Collections.synchronizedMap(Map<K,V> m)`：包装为线程安全的 Map。

**代码示例**：
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class SynchronizedCollectionDemo {
    public static void main(String[] args) {
        // 线程不安全的 ArrayList
        List<String> unsafeList = new ArrayList<>();
        // 包装为线程安全的 List
        List<String> safeList = Collections.synchronizedList(unsafeList);

        // 多线程环境下使用 safeList，无需手动加锁
        safeList.add("Java");
        safeList.add("Python");
    }
}
```


##### （2）使用并发集合类（Java 5+ 推荐）
JDK 5 引入 `java.util.concurrent` 包，提供专门的并发集合类，性能优于 Collections 包装的集合：
- `CopyOnWriteArrayList`：线程安全的 List（写时复制，适合读多写少场景）；
- `CopyOnWriteArraySet`：线程安全的 Set（基于 CopyOnWriteArrayList）；
- `ConcurrentHashMap`：线程安全的 Map（JDK 8 用 CAS+ synchronized 实现，效率高，适合高并发场景）；
- `ConcurrentLinkedQueue`：线程安全的 Queue（无锁实现，高并发场景推荐）。

**代码示例（ConcurrentHashMap）**：
```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapDemo {
    public static void main(String[] args) {
        // 线程安全的 ConcurrentHashMap
        Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();

        // 多线程并发添加
        new Thread(() -> concurrentMap.put("A", 1)).start();
        new Thread(() -> concurrentMap.put("B", 2)).start();

        System.out.println(concurrentMap); // 输出：{A=1, B=2}（或顺序相反）
    }
}
```


##### （3）手动加锁（synchronized 或 Lock）
在多线程操作集合时，手动为集合的操作加锁（如 `synchronized` 代码块），确保同一时间只有一个线程修改集合。

**代码示例**：
```java
import java.util.ArrayList;
import java.util.List;

public class ManualLockDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Object lock = new Object(); // 锁对象

        // 线程 1 添加元素
        new Thread(() -> {
            synchronized (lock) { // 手动加锁
                for (int i = 0; i < 5; i++) {
                    list.add("Thread1-" + i);
                }
            }
        }).start();

        // 线程 2 添加元素
        new Thread(() -> {
            synchronized (lock) { // 手动加锁
                for (int i = 0; i < 5; i++) {
                    list.add("Thread2-" + i);
                }
            }
        }).start();

        // 等待线程执行完成
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("集合大小：" + list.size()); // 输出：10（线程安全）
    }
}
```


### 三、工具类：简化开发的“小助手”
#### 1. Collections 工具类（集合操作）
`java.util.Collections` 是集合操作的工具类，提供大量静态方法，用于排序、查找、同步集合等：
- **排序**：`void sort(List<T> list)`（自然排序）、`void sort(List<T> list, Comparator<? super T> c)`（定制排序）；
- **查找**：`int binarySearch(List<? extends Comparable<? super T>> list, T key)`（二分查找，需先排序）、`T max(Collection<? extends T> coll)`（获取最大值）；
- **修改**：`void reverse(List<?> list)`（反转集合）、`void shuffle(List<?> list)`（随机打乱集合）、`void fill(List<? super T> list, T obj)`（填充集合）；
- **同步**：`synchronizedList(List<T> list)`（线程安全包装）。

**代码示例**：
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CollectionsDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(3);
        list.add(1);
        list.add(2);
        list.add(5);
        list.add(4);

        // 1. 自然排序（升序）
        Collections.sort(list);
        System.out.println("排序后：" + list); // 输出：[1, 2, 3, 4, 5]

        // 2. 二分查找（查找 3 的索引）
        int index = Collections.binarySearch(list, 3);
        System.out.println("3 的索引：" + index); // 输出：2

        // 3. 反转集合
        Collections.reverse(list);
        System.out.println("反转后：" + list); // 输出：[5, 4, 3, 2, 1]

        // 4. 随机打乱
        Collections.shuffle(list);
        System.out.println("打乱后：" + list); // 输出：随机顺序（如 [3, 1, 5, 2, 4]）

        // 5. 获取最大值
        Integer max = Collections.max(list);
        System.out.println("最大值：" + max); // 输出：5
    }
}
```


#### 2. Arrays 工具类（数组操作）
`java.util.Arrays` 是数组操作的工具类，提供数组排序、查找、转换为集合等方法：
- **排序**：`void sort(int[] a)`（数组排序，基本类型升序）、`void sort(Object[] a)`（对象数组自然排序）；
- **查找**：`int binarySearch(int[] a, int key)`（二分查找）；
- **转换**：`List<T> asList(T... a)`（将数组转换为 List，注意：返回的 List 不可修改，底层是数组）；
- **填充**：`void fill(int[] a, int val)`（填充数组）；
- **比较**：`boolean equals(int[] a, int[] a2)`（比较两个数组是否相等）。

**代码示例**：
```java
import java.util.Arrays;
import java.util.List;

public class ArraysDemo {
    public static void main(String[] args) {
        // 1. 数组排序
        int[] arr = {3, 1, 2, 5, 4};
        Arrays.sort(arr);
        System.out.println("排序后数组：" + Arrays.toString(arr)); // 输出：[1, 2, 3, 4, 5]

        // 2. 二分查找
        int keyIndex = Arrays.binarySearch(arr, 4);
        System.out.println("4 的索引：" + keyIndex); // 输出：3

        // 3. 数组转 List（注意：返回的 List 不可修改）
        String[] strArr = {"Java", "Python", "C++"};
        List<String> strList = Arrays.asList(strArr);
        System.out.println("数组转 List：" + strList); // 输出：[Java, Python, C++]
        // strList.add("Go"); // 抛出 UnsupportedOperationException（不可修改）

        // 4. 填充数组
        int[] fillArr = new int[5];
        Arrays.fill(fillArr, 10);
        System.out.println("填充后数组：" + Arrays.toString(fillArr)); // 输出：[10, 10, 10, 10, 10]
    }
}
```


## 九、Java 关键字汇总（全）
Java 关键字是对编译器有特殊含义的保留字符，共 53 个（其中 `const` 和 `goto` 是保留关键字，无实际用途），按功能分类如下：

| 分类         | 关键字列表                                                                 |
|--------------|--------------------------------------------------------------------------|
| 访问修饰符   | public、protected、private、default（默认，无关键字）                     |
| 类与接口相关 | class、interface、abstract、final、extends、implements、new、instanceof  |
| 数据类型     | byte、short、int、long、float、double、char、boolean、void               |
| 流程控制     | if、else、switch、case、default、break、continue、for、while、do、return |
| 异常处理     | try、catch、finally、throw、throws                                       |
| 静态与实例   | static、this、super                                                      |
| 线程相关     | synchronized、volatile                                                   |
| 其他         | package、import、enum、assert、strictfp、transient、native、strictfp    |
| 保留关键字   | const、goto（无实际用途，不可用作标识符）                                 |

**注意**：关键字不能用作变量名、类名、方法名等标识符；所有关键字均为小写（如 `Int` 不是关键字，可作为标识符，但不推荐）。


## 十、总结：Java 核心知识体系图

```
Java 核心知识
├─ 基础语法
│  ├─ 变量与数据类型（8种基本类型+引用类型）
│  ├─ 运算符与表达式（算术、逻辑、比较、三元等）
│  ├─ 流程控制（if-else、switch、for、while）
│  └─ 关键字（53个，按功能分类）
│
├─ 面向对象（OOP）
│  ├─ 三大特性：封装、继承（extends）、多态（父类引用指向子类对象）
│  ├─ 核心概念：类与对象、构造器、this/super、方法重写/重载
│  ├─ 修饰符：abstract（抽象）、final（不可变）、static（静态）
│  ├─ 抽象类与接口：abstract class、interface、implements
│  └─ 内部类：成员内部类、局部内部类、匿名内部类、静态内部类
│
├─ 集合框架
│  ├─ Collection：List（ArrayList、LinkedList）、Set（HashSet、TreeSet）
│  ├─ Map：HashMap、LinkedHashMap、TreeMap
│  ├─ 工具类：Collections、Arrays
│  └─ 并发集合：CopyOnWriteArrayList、ConcurrentHashMap
│
├─ 异常处理
│  ├─ 异常体系：Throwable（Error、Exception）
│  ├─ 处理机制：try-catch-finally、throw、throws
│  └─ 自定义异常：继承 Exception/RuntimeException
│
├─ 常用类与 API
│  ├─ 字符串：String、StringBuffer、StringBuilder
│  ├─ 日期时间：Date、Calendar、LocalDateTime（Java 8+）
│  ├─ 工具类：Math、Random、UUID、File
│  └─ 包装类：Integer、Double 等（基本类型→引用类型）
│
└─ 进阶内容
   ├─ 线程编程：Thread、Runnable、synchronized、volatile
   ├─ IO 流：字节流（InputStream/OutputStream）、字符流（Reader/Writer）
   ├─ 反射：Class、Constructor、Method、Field
   └─ 泛型：List<T>、Map<K,V>（类型安全，避免强制转换）


---

> Author: 7M7  
> URL: http://localhost:1313/posts/86f3bbb/  


# JavaSE 基础笔记（二）：继承、多态、异常与集合


这一篇接着整理 JavaSE 后半段：`this` / `super`、`static`、继承、多态、抽象类、接口、异常、常用类和集合框架。重点放在“什么时候用”和“容易错在哪里”。

<!--more-->

# JavaSE 基础笔记（二）：继承、多态、异常与集合

## 1. `this` 与 `super`

`this` 指当前对象，`super` 指当前对象中的父类部分。

| 对比 | `this` | `super` |
| --- | --- | --- |
| 代表 | 当前对象 | 父类部分 |
| 访问属性 | `this.name` | `super.name` |
| 调用方法 | `this.show()` | `super.show()` |
| 调用构造 | `this(...)` | `super(...)` |
| 使用前提 | 非静态上下文 | 有继承关系 |

构造方法里要注意：

- `this(...)` 和 `super(...)` 都必须放在构造方法第一行。
- 二者不能同时出现。
- 子类构造方法默认会先调用父类无参构造。
- 如果父类没有无参构造，子类必须显式调用父类有参构造。

```java
class Father {
    public Father(String name) {
        System.out.println("Father: " + name);
    }
}

class Son extends Father {
    public Son() {
        super("张三");
        System.out.println("Son");
    }
}
```

## 2. `static`

`static` 修饰的成员属于类，不属于某个具体对象。

| 对比 | 静态成员 | 非静态成员 |
| --- | --- | --- |
| 所属 | 类 | 对象 |
| 访问方式 | `类名.成员` | `对象.成员` |
| 内存特点 | 类加载时初始化 | 创建对象时初始化 |
| 是否能用 `this` | 不能 | 能 |

```java
class Counter {
    static int count = 0;

    public Counter() {
        count++;
    }
}
```

静态方法不能直接访问非静态成员，因为静态方法执行时不一定有对象。

```java
class Demo {
    int age = 18;

    public static void test() {
        // System.out.println(age); // 错误
    }
}
```

## 3. 继承与方法重写

继承用于表达“is-a”关系，让子类复用父类的属性和方法。

```java
class Animal {
    public void eat() {
        System.out.println("吃东西");
    }
}

class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }
}
```

重写规则：

- 方法名、参数列表必须相同。
- 返回值类型要相同或更小。
- 子类访问权限不能比父类更严格。
- `private` 方法不能被重写。
- `static` 方法不参与多态，只是隐藏。

## 4. 多态

多态的写法是父类引用指向子类对象：

```java
Animal animal = new Dog();
animal.eat();
```

多态的核心：

- 编译看左边：能不能调用，看引用类型。
- 运行看右边：实际执行哪个方法，看对象类型。
- 成员方法有多态，成员变量没有多态。
- 静态方法没有多态。

向下转型前最好先判断类型：

```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
    dog.eat();
}
```

## 5. `final`、抽象类与接口

`final` 表示“不可变方向”：

- 修饰类：不能被继承。
- 修饰方法：不能被重写。
- 修饰变量：只能赋值一次。

抽象类用于抽取共同模板：

```java
abstract class Shape {
    abstract double area();
}
```

接口用于定义能力或规范：

```java
interface Flyable {
    void fly();
}
```

简单区分：

| 场景 | 更适合 |
| --- | --- |
| 有共同父类和共享代码 | 抽象类 |
| 只想约定能力 | 接口 |
| 一个类需要多个能力 | 接口 |

## 6. 异常处理

异常分两类：

- 编译时异常：必须处理，比如 `IOException`。
- 运行时异常：可以不显式处理，比如 `NullPointerException`、`ArrayIndexOutOfBoundsException`。

基本写法：

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("除数不能为 0");
} finally {
    System.out.println("无论是否异常都会执行");
}
```

方法可以用 `throws` 把异常交给调用者：

```java
public static void readFile() throws IOException {
    // ...
}
```

自己主动抛异常用 `throw`：

```java
if (age < 0) {
    throw new IllegalArgumentException("年龄不能小于 0");
}
```

## 7. 常用类

常见类按用途记：

| 类 | 用途 |
| --- | --- |
| `String` | 不可变字符串 |
| `StringBuilder` | 可变字符串，适合频繁拼接 |
| `Math` | 数学工具 |
| `Arrays` | 数组工具 |
| `Objects` | 空值安全比较 |
| `Date` / `Calendar` | 旧日期 API |
| `LocalDateTime` | 新日期 API |

字符串拼接多时优先用 `StringBuilder`：

```java
StringBuilder builder = new StringBuilder();
builder.append("Hello");
builder.append(" Java");
System.out.println(builder.toString());
```

## 8. 集合框架总览

集合用于存储对象。先记两条主线：

```text
Collection
├── List：有序、可重复、有索引
└── Set：无序或排序、不可重复、无索引

Map：键值对，key 不可重复
```

## 9. `List`

`List` 有序、可重复、支持索引。

| 实现类 | 底层 | 特点 | 适合 |
| --- | --- | --- | --- |
| `ArrayList` | 动态数组 | 查询快，增删慢 | 读多写少 |
| `LinkedList` | 双向链表 | 首尾增删快，查询慢 | 队列、栈 |

```java
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");
list.set(1, "Go");
System.out.println(list.get(0));
```

`LinkedList` 也可以当队列或栈用：

```java
Deque<String> deque = new LinkedList<>();
deque.offer("A");
deque.offer("B");
System.out.println(deque.poll());

deque.push("C");
System.out.println(deque.pop());
```

## 10. `Set`

`Set` 不允许重复元素。

| 实现类 | 特点 |
| --- | --- |
| `HashSet` | 无序，查找快，基于 `hashCode()` 和 `equals()` |
| `LinkedHashSet` | 保留插入顺序 |
| `TreeSet` | 自动排序，基于比较规则 |

自定义对象放进 `HashSet` 时，如果要按内容去重，必须同时重写 `equals()` 和 `hashCode()`。

```java
class Student {
    private String id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;
        Student student = (Student) o;
        return id.equals(student.id);
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }
}
```

## 11. `Map`

`Map` 存键值对，key 不能重复。

| 实现类 | 特点 |
| --- | --- |
| `HashMap` | 常用，key 无序 |
| `LinkedHashMap` | 保留插入顺序 |
| `TreeMap` | 按 key 排序 |
| `Hashtable` | 老类，线程安全但不常用 |

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("张三", 90);
scores.put("李四", 85);

for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

## 12. `Collections` 工具类

`Collections` 是集合工具类，常用方法：

```java
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 5, 2));

Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

## 13. 关键字速查

| 关键字 | 作用 |
| --- | --- |
| `this` | 当前对象 |
| `super` | 父类部分 |
| `static` | 属于类 |
| `final` | 不可变、不可继承、不可重写 |
| `abstract` | 抽象 |
| `interface` | 接口 |
| `extends` | 继承类 |
| `implements` | 实现接口 |
| `try/catch/finally` | 异常处理 |
| `throw/throws` | 抛出异常 |

## 14. 小结

JavaSE 后半段可以按这条线复习：

1. `this`、`super`、`static` 先搞清楚对象、类、父类的边界。
2. 继承和多态解决代码复用与运行时行为变化。
3. 抽象类和接口用来抽取模板或能力。
4. 异常处理让程序能面对错误输入和运行失败。
5. 集合框架是日常写 Java 最常用的数据容器。


---

> 作者: 7M7  
> URL: http://localhost:1313/posts/54c6db0/  


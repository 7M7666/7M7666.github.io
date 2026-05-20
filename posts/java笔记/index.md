# JavaSE 基础笔记（一）：输入、方法、数组与面向对象入门


这篇只保留 JavaSE 入门阶段最容易忘、最容易写错的部分：控制台输入、可变参数、内存模型、数组，以及面向对象的基本写法。它不是完整教材，更像复习时能快速翻到重点的笔记。

<!--more-->

# JavaSE 基础笔记（一）：输入、方法、数组与面向对象入门

## 1. Scanner：控制台输入

`Scanner` 用来从控制台读取用户输入。基本流程是导包、创建对象、读取数据，最后关闭资源。

```java
import java.util.Scanner;

public class ScannerDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("请输入姓名：");
        String name = scanner.nextLine();
        System.out.println("你好，" + name);

        scanner.close();
    }
}
```

常用方法：

| 方法 | 作用 | 适合场景 |
| --- | --- | --- |
| `hasNext()` | 判断是否还有非空白输入 | 读取单词 |
| `hasNextLine()` | 判断是否还有一整行输入 | 读取一句话 |
| `hasNextInt()` | 判断下一个输入是否为整数 | 读取年龄、编号 |
| `next()` | 读取到空白字符前 | 用户名、命令 |
| `nextLine()` | 读取整行 | 地址、备注 |
| `nextInt()` | 读取整数 | 数字输入 |

最常见的坑是混用 `nextInt()` 和 `nextLine()`：

```java
int age = scanner.nextInt();
String name = scanner.nextLine(); // 会读到上一行剩下的回车
```

解决方式有两个：

```java
int age = scanner.nextInt();
scanner.nextLine();       // 先吃掉回车
String name = scanner.nextLine();
```

或者统一用 `nextLine()` 读字符串，再手动转换：

```java
int age = Integer.parseInt(scanner.nextLine());
String name = scanner.nextLine();
```

## 2. 可变参数

可变参数适合“参数个数不固定，但类型相同”的方法。本质上它还是数组。

规则：

- 一个方法最多只能有一个可变参数。
- 可变参数必须放在参数列表最后。
- 调用时可以传 0 个、1 个、多个参数，也可以传数组。

```java
public static void printMax(double... numbers) {
    if (numbers.length == 0) {
        System.out.println("未传入参数");
        return;
    }

    double max = numbers[0];
    for (double number : numbers) {
        if (number > max) {
            max = number;
        }
    }

    System.out.println("最大值：" + max);
}
```

## 3. Java 内存模型入门

入门阶段先记住三块区域：

| 区域 | 主要存什么 | 特点 |
| --- | --- | --- |
| 栈 | 局部变量、引用变量、方法调用栈帧 | 随方法调用创建和销毁 |
| 堆 | `new` 出来的对象和数组 | 被 GC 管理 |
| 方法区 | 类信息、静态成员、运行时常量池 | 随类加载存在 |

例子：

```java
class Pet {
    String name;
    int age;

    public void shout() {
        System.out.println(name + "在叫");
    }
}

public class Demo {
    public static void main(String[] args) {
        Pet cat = new Pet();
        cat.name = "旺财";
        cat.age = 3;
    }
}
```

可以这样理解：

- `cat` 这个引用变量在栈里。
- `new Pet()` 创建的对象在堆里。
- `Pet` 类的信息和 `shout()` 方法字节码在方法区里。
- `cat.name`、`cat.age` 是堆中对象自己的成员变量。

## 4. 数组

数组是引用类型。它的长度在创建时确定，后面不能直接改变。

```java
int[] a = new int[3];        // 动态初始化，默认值为 0
int[] b = new int[]{1, 2, 3};
int[] c = {1, 2, 3};         // 只能声明时这样写
```

数组常见操作：

```java
int[] nums = {3, 1, 4, 1, 5};

for (int i = 0; i < nums.length; i++) {
    System.out.println(nums[i]);
}

for (int num : nums) {
    System.out.println(num);
}
```

注意：

- 索引范围是 `0 ~ length - 1`。
- 越界会抛出 `ArrayIndexOutOfBoundsException`。
- 数组变量保存的是数组对象的引用。

## 5. 稀疏数组

稀疏数组适合保存“大部分位置都是默认值”的二维数据，比如棋盘。

普通二维数组：

```text
0 0 0 0
0 1 0 0
0 0 2 0
```

稀疏数组只记录非零位置：

```text
行  列  值
3   4   2   // 原数组 3 行 4 列，共 2 个有效值
1   1   1
2   2   2
```

核心思路：

1. 遍历原数组，统计有效值个数。
2. 创建 `sum + 1` 行、3 列的稀疏数组。
3. 第一行存原数组规模和有效值个数。
4. 后续每行存一个有效值的位置和值。

## 6. 面向对象入门

类是模板，对象是根据模板创建出来的实例。

```java
class Student {
    String name;
    int age;

    public void study() {
        System.out.println(name + "正在学习");
    }
}

public class Demo {
    public static void main(String[] args) {
        Student student = new Student();
        student.name = "张三";
        student.age = 18;
        student.study();
    }
}
```

入门阶段先抓住这几个点：

- 成员变量属于对象，有默认值。
- 局部变量属于方法，必须先赋值再使用。
- 方法可以操作当前对象的成员变量。
- `new` 负责在堆中创建对象。

## 7. 构造方法与封装

构造方法在创建对象时执行，常用于初始化成员变量。

```java
class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

封装的基本做法：

- 成员变量用 `private` 修饰。
- 通过 `getXxx()` 和 `setXxx()` 暴露必要访问。
- 对外隐藏实现细节，只保留稳定接口。

`this` 表示当前对象，常用来区分成员变量和局部变量：

```java
public User(String name) {
    this.name = name;
}
```

## 8. 小结

这一篇对应 JavaSE 的入门层：

- `Scanner` 解决输入问题。
- 可变参数解决参数数量不固定的问题。
- 栈、堆、方法区帮助理解对象和引用。
- 数组是固定长度的引用类型。
- 类、对象、构造方法、封装是面向对象的第一层基础。

下一篇继续整理继承、多态、异常、常用类和集合框架。


---

> 作者: 7M7  
> URL: http://localhost:1313/posts/java%E7%AC%94%E8%AE%B0/  


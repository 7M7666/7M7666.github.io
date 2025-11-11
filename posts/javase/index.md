# Javase

# Javase基础笔记1.0——基于《狂神说Java》



## 1. 人机交互第一步：Scanner类（Java5新特性）
### 1.1 核心定位：解决“程序没人交互”的痛点
在学Scanner之前，我们写的程序都是“自说自话”（比如固定打印`Hello World`），而`java.util.Scanner`是Java提供的**原生输入工具**，能获取用户从控制台输入的文本、数字等内容。  


### 1.2 完整使用流程（含资源关闭）
Scanner的使用分5步，缺一不可（尤其是关闭资源，避免内存泄漏）：
```java
import java.util.Scanner; // 1. 导入Scanner包（IDEA会自动提示）

public class ScannerDemo {
    public static void main(String[] args) {
        // 2. 创建Scanner对象，关联系统输入流（System.in = 控制台输入）
        Scanner scanner = new Scanner(System.in);

        // 3. 判断是否有输入（避免读取空数据抛异常）
        System.out.print("请输入你的姓名：");
        if (scanner.hasNextLine()) { // 判断是否有完整行输入
            // 4. 读取输入内容
            String name = scanner.nextLine();
            System.out.println("你好，" + name + "！");
        }

        // 5. 关闭Scanner，释放输入流资源（必须做！）
        scanner.close();
    }
}
```
**运行效果**：
```
请输入你的姓名：张三
你好，张三！
```

### 1.3 关键方法：判断与读取的“黄金搭档”
Scanner的核心是“先判断，再读取”，避免直接读取导致的`InputMismatchException`（输入类型不匹配）。以下是常用方法对比：

| 方法类别       | 方法名                | 功能描述                                                                 | 适用场景                     |
|----------------|-----------------------|--------------------------------------------------------------------------|------------------------------|
| **判断输入**   | `hasNext()`           | 判断后续是否有“有效字符”（非空格/Tab/Enter），返回boolean                 | 读取不含空格的字符串         |
| **判断输入**   | `hasNextLine()`       | 判断后续是否有“完整行”（以Enter为结束符），返回boolean                   | 读取含空格的字符串（如姓名） |
| **判断输入**   | `hasNextInt()`        | 判断后续输入是否为整数，返回boolean                                      | 读取年龄、分数等int类型数据  |
| **判断输入**   | `hasNextDouble()`     | 判断后续输入是否为浮点数，返回boolean                                    | 读取工资、身高等double数据  |
| **读取输入**   | `next()`              | 读取“有效字符到空白字符”的字符串（不含空白）                             | 读取用户名（无空格）         |
| **读取输入**   | `nextLine()`          | 读取“当前位置到Enter前”的所有字符（含空白）                             | 读取地址、备注（含空格）     |
| **读取输入**   | `nextInt()`           | 读取整数并返回int，输入非整数会抛异常                                    | 读取整数                     |
| **读取输入**   | `nextDouble()`        | 读取浮点数并返回double，输入非浮点数会抛异常                              | 读取浮点数                   |

### 1.4 深坑预警：`next()`与`nextLine()`的“兼容性问题”
这是入门最容易踩的坑！当`next()`和`nextLine()`连续使用时，`nextLine()`会“吃掉”`next()`残留的Enter，导致读取空字符串。

#### 问题复现：
```java
Scanner scanner = new Scanner(System.in);
System.out.print("请输入年龄：");
int age = scanner.nextInt(); // 输入“25”后按Enter，缓冲区残留Enter
System.out.print("请输入姓名：");
String name = scanner.nextLine(); // 直接读取残留的Enter，返回空字符串
System.out.println("年龄：" + age + "，姓名：" + name); // 姓名为空
```
**运行效果**：
```
请输入年龄：25
请输入姓名：
年龄：25，姓名：
```

#### 解决方案（两种）：
1. **方案1：额外调用`nextLine()`消耗Enter**  
   在`nextInt()`后加一行`scanner.nextLine();`，手动清空缓冲区：
   ```java
   int age = scanner.nextInt();
   scanner.nextLine(); // 关键：消耗残留的Enter
   String name = scanner.nextLine(); // 正常读取姓名
   ```

2. **方案2：统一用`nextLine()`读取，再手动转类型**  
   避免混合使用不同读取方法，先读字符串再转成目标类型（更推荐，通用性强）：
   ```java
   System.out.print("请输入年龄：");
   String ageStr = scanner.nextLine(); // 先读字符串
   int age = Integer.parseInt(ageStr); // 转成int
   System.out.print("请输入姓名：");
   String name = scanner.nextLine(); // 正常读取
   ```


## 2. 方法进阶：可变参数——解决“参数数量不确定”
### 2.1 核心场景：当方法参数“可多可少”时
比如实现“求任意个数字的最大值”，如果没有可变参数，需要重载多个方法（`max(int a)`、`max(int a,int b)`、`max(int a,int b,int c)`），麻烦且不灵活。  
JDK1.5引入的**可变参数**，允许方法接收“数量不固定的同类型参数”，本质是**数组的语法糖**（底层仍用数组存储）。

### 2.2 语法规则：3个“必须遵守”
1. 声明格式：在**参数类型后加省略号（`...`）**，参数名自定义（如`numbers`）；  
2. 数量限制：一个方法**只能有1个可变参数**（多了会歧义）；  
3. 位置限制：可变参数必须是方法的**最后一个参数**（普通参数需在前面）。

#### 错误与正确示例对比：
| 语法示例                          | 是否正确 | 原因分析                                                                 |
|-----------------------------------|----------|--------------------------------------------------------------------------|
| `public static void printMax(double... numbers)` | ✅        | 符合“类型+...+参数名”，无其他参数                                       |
| `public static void test(String name, int... scores)` | ✅        | 可变参数在最后，普通参数在前面                                           |
| `public static void test(int... scores, String name)` | ❌        | 可变参数不是最后一个参数                                                 |
| `public static void test(int... a, double... b)` | ❌        | 一个方法有2个可变参数，无法区分参数边界                                   |

### 2.3 实战案例：求任意个double数字的最大值
```java
public class VariableArgsDemo {
    public static void main(String[] args) {
        // 调用可变参数方法：可传0个、1个、多个参数，或直接传数组
        printMax(); // 无参数
        printMax(1.5); // 1个参数
        printMax(3.2, 5.8, 2.9, 4.1); // 多个参数
        printMax(new double[]{1.1, 2.2, 3.3}); // 传数组（底层兼容）
    }

    // 可变参数方法：求最大值
    public static void printMax(double... numbers) {
        // 1. 判断是否传入参数（numbers本质是数组，length=0表示无参数）
        if (numbers.length == 0) {
            System.out.println("⚠️ 未传入任何参数");
            return; // 结束方法，避免后续空指针
        }

        // 2. 初始化最大值（取第一个参数）
        double max = numbers[0];

        // 3. 遍历数组，对比更新最大值（原笔记标注“排序!”，遍历更高效）
        for (double num : numbers) {
            if (num > max) {
                max = num;
            }
        }

        // 4. 输出结果
        System.out.println("当前最大值：" + max);
    }
}
```
**运行效果**：
```
⚠️ 未传入任何参数
当前最大值：1.5
当前最大值：5.8
当前最大值：3.3
```


## 3. 底层核心：Java内存分析——搞懂“对象存在哪里”
这是Java的“灵魂知识点”，很多人学完面向对象还不清楚“new的对象存在哪”“引用变量是什么”。结合原笔记的ProcessOn配图，我们用`Pet`类实例拆解堆、栈、方法区的分工。

### 3.1 内存三分区：职责分明
Java内存主要分为**堆（Heap）、栈（Stack）、方法区（Method Area，JDK8+为元空间）**，各分区存储内容和特性完全不同：

| 分区名称       | 存储内容（超详细版）                                                                 | 线程共享性       | 生命周期                                                                 |
|----------------|------------------------------------------------------------------------------------|------------------|--------------------------------------------------------------------------|
| **堆（Heap）** | 1. 所有`new`创建的**对象**（如`new Pet()`、`new Person()`）；<br>2. 所有**数组**（如`new int[5]`）；<br>3. 对象的**成员变量**（随对象存储，有默认值） | 可被所有线程共享 | 随对象创建而分配，随GC（垃圾回收）回收而释放（无需手动管理）               |
| **栈（Stack）** | 1. **基本类型变量**（如`int age=25`，存储“25”这个具体数值）；<br>2. **引用类型变量**（如`Pet cat=new Pet()`，存储“对象在堆中的地址”）；<br>3. 方法调用的**栈帧**（存储局部变量、操作数栈、返回地址） | 线程私有（每个线程独立栈） | 栈帧随方法调用创建，方法执行完出栈；变量随作用域（如代码块）结束而释放     |
| **方法区（Method Area）** | 1. 类的**字节码信息**（类名、属性、方法定义）；<br>2. **static静态变量/方法**（随类加载，不依赖对象）；<br>3. 运行时常量池（如字符串“旺财”、整数常量123） | 可被所有线程共享 | 随类加载而分配，随类卸载（极少发生）而释放                               |

### 3.2 实例拆解：`Pet`对象的内存分布
我们用一段简单代码，逐区分析内存中“数据到底存在哪”：

#### 前提代码：
```java
// 定义Pet类
class Pet {
    String name; // 成员变量（默认null）
    int age;     // 成员变量（默认0）

    // 成员方法
    public void shout() {
        System.out.println(name + "在叫~");
    }
}

// 测试类：创建2个Pet对象
public class PetMemoryDemo {
    public static void main(String[] args) {
        Pet cat = new Pet(); // 第一个对象：cat
        cat.name = "旺财";   // 给cat的name赋值
        cat.age = 3;        // 给cat的age赋值

        Pet dog = new Pet(); // 第二个对象：dog
        // dog未赋值，name默认null，age默认0
    }
}
```

#### 内存分布详图（逐区解析）：
1. **方法区（元空间）**：  
   - 存储`Pet`类字节码：包含`name`（String类型）、`age`（int类型）的属性定义，`shout()`方法的字节码；  
   - 存储`PetMemoryDemo`类字节码：包含`main()`方法的字节码；  
   - 常量池：存储字符串“旺财”（`cat.name = "旺财"`时，name引用指向此常量）；  
   - 静态区域：无static变量/方法，暂空（若有`static String type = "宠物"`，则存在这里）。

2. **堆（Heap）**：  
   - 第一个对象（地址`0x0001`）：  
     - 类型：`Pet`对象；  
     - 成员变量：`name`（存储常量池“旺财”的地址）、`age`（存储数值3）；  
     - 方法引用：`shout()`方法的地址（指向方法区的`shout()`字节码）；  
   - 第二个对象（地址`0x0002`）：  
     - 类型：`Pet`对象；  
     - 成员变量：`name`（默认null，无指向）、`age`（默认0，存储数值0）；  
     - 方法引用：`shout()`方法的地址（和第一个对象共享方法区的字节码，不重复存储）。

3. **栈（Stack，main线程栈）**：  
   - 栈帧：`main()`方法调用时创建的栈帧（包含局部变量表、操作数栈）；  
   - 局部变量1：`cat`（引用变量，存储堆中第一个对象的地址`0x0001`）；  
   - 局部变量2：`dog`（引用变量，存储堆中第二个对象的地址`0x0002`）；  
   - 操作数栈：执行`cat.name = "旺财"`时，临时存储“旺财”的地址（从常量池到堆对象）。

#### 关键结论（必须记住）：
- 引用变量（如`cat`、`dog`）在**栈**中，仅存“对象地址”，不存对象本身；  
- 对象的属性和方法引用在**堆**中，方法的字节码在**方法区**（所有对象共享方法，不重复存）；  
- 字符串常量优先存在**方法区的常量池**（避免重复创建，节省内存）；  
- 基本类型的“变量+值”在栈中（局部变量）或堆中（成员变量，随对象）。


## 4. 基础数据结构：数组——“相同类型的有序集合”
### 4.1 本质：数组是“引用类型的对象”
很多人误以为数组是“基本类型”，但实际上：  
- 数组是Java中的**引用数据类型**（即使存储基本类型，数组本身也是对象）；  
- 数组的“长度”是对象的成员变量（`arr.length`），固定不可变（创建后不能改长度）；  
- 原笔记标注“字符串是数组，数组名是指针”——Java无“指针”，但数组名本质是“引用”，指向堆中的数组对象。

### 4.2 核心特性（3个“必须知道”）
1. **长度固定**：创建时必须指定长度（如`new int[5]`），长度一旦确定，无法扩容（需扩容需手动创建新数组复制元素）；  
2. **类型统一**：所有元素必须是“同一种数据类型”（如`int[]`只能存int，`Object[]`可存任意对象，但仍属Object类型）；  
3. **越界异常**：访问索引超出`[0, 长度-1]`时，抛`ArrayIndexOutOfBoundsException`（运行时异常，编译不报错）。

### 4.3 3种初始化方式（附代码示例）
| 初始化方式       | 语法示例                                  | 特点                                                                 | 适用场景                     |
|------------------|-------------------------------------------|----------------------------------------------------------------------|------------------------------|
| **动态初始化**   | `int[] arr = new int[5];`                 | 只指定长度，元素按默认值初始化（int默认0，String默认null）           | 知道长度，暂不确定元素值     |
| **静态初始化**   | `int[] arr = new int[]{1, 2, 3, 4};`      | 指定元素值，长度由元素个数自动确定（`new int[]`不可省略）             | 知道所有元素值，无需指定长度 |
| **简化静态初始化** | `int[] arr = {1, 2, 3, 4};`               | 静态初始化的简化写法，**只能在声明时使用**（分开声明和赋值会报错）     | 代码简洁，声明时确定元素值   |

#### 错误示例（简化初始化的坑）：
```java
int[] arr;
arr = {1, 2, 3}; // 错误！简化初始化只能在声明时使用（如int[] arr={...}）
arr = new int[]{1, 2, 3}; // 正确！分开声明时，需用完整静态初始化
```

### 4.4 实战：数组遍历与常见操作
#### 1. 遍历（2种方式）
- 普通for循环：可通过索引修改元素（推荐需修改元素时用）；  
- 增强for循环（foreach）：仅遍历元素，无法修改索引（推荐只读遍历）。

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[] arr = {10, 20, 30, 40};

        // 1. 普通for循环（可修改元素）
        System.out.println("普通for循环遍历：");
        for (int i = 0; i < arr.length; i++) {
            arr[i] += 5; // 修改元素（加5）
            System.out.print(arr[i] + " "); // 输出：15 25 35 45
        }

        // 2. 增强for循环（只读）
        System.out.println("\n增强for循环遍历：");
        for (int num : arr) { // 依次取arr中的元素赋值给num
            // num += 5; // 仅修改副本，不影响原数组
            System.out.print(num + " "); // 输出：15 25 35 45
        }
    }
}
```

#### 2. 常见操作：数组复制（手动+工具类）
```java
import java.util.Arrays; // 导入Arrays工具类

public class ArrayCopyDemo {
    public static void main(String[] args) {
        int[] arr1 = {1, 2, 3, 4};
        int[] arr2 = new int[arr1.length];

        // 1. 手动复制（基础）
        for (int i = 0; i < arr1.length; i++) {
            arr2[i] = arr1[i];
        }
        System.out.println("手动复制后arr2：" + Arrays.toString(arr2)); // [1,2,3,4]

        // 2. 工具类复制（推荐，JDK自带）
        int[] arr3 = Arrays.copyOf(arr1, arr1.length); // 复制arr1的所有元素
        int[] arr4 = Arrays.copyOf(arr1, 6); // 复制arr1，新数组长度6，多余元素默认0
        System.out.println("Arrays.copyOf复制arr3：" + Arrays.toString(arr3)); // [1,2,3,4]
        System.out.println("Arrays.copyOf复制arr4：" + Arrays.toString(arr4)); // [1,2,3,4,0,0]
    }
}
```

## 稀疏数组（Sparse Array）—— 压缩大规模稀疏数据
### 1.1 核心定义与使用场景
- **适用场景**：当一个数组中**大部分元素为0**，或者为**同一默认值**时，使用稀疏数组可以大幅减少内存占用，缩小程序规模。
- **处理逻辑**：
  1. 记录原始数组的**总行数、总列数**，以及**有效元素（非默认值）的个数**；
  2. 将每个有效元素的**行号、列号、值**存储在一个小规模的二维数组中（稀疏数组）。

### 1.2 图示对比：原始数组 vs 稀疏数组
| 原始数组（以示例矩阵为例） | 稀疏数组（存储有效元素的“行、列、值”） |
|---------------------------|---------------------------------------|
| $\begin{pmatrix} 0 & 0 & 0 & 22 & 0 & 0 & 15 \\ 0 & 11 & 0 & 0 & 0 & 17 & 0 \\ 0 & 0 & 0 & -6 & 0 & 0 & 0 \\ 0 & 0 & 0 & 0 & 0 & 39 & 0 \\ 91 & 0 & 0 & 0 & 0 & 0 & 0 \\ 0 & 0 & 28 & 0 & 0 & 0 & 0 \end{pmatrix}$ | | 行（row） | 列（col） | 值（value） |
| --- | --- | --- | --- |
| [0] | 6 | 7 | 8 |
| [1] | 0 | 3 | 22 |
| [2] | 0 | 6 | 15 |
| [3] | 1 | 1 | 11 |
| [4] | 1 | 5 | 17 |
| [5] | 2 | 3 | -6 |
| [6] | 3 | 5 | 39 |
| [7] | 4 | 0 | 91 |
| [8] | 5 | 2 | 28 |
|  |  |  |  |

> 说明：稀疏数组的**第一行**存储原始数组的“总行数、总列数、有效元素个数”，后续每一行存储一个有效元素的“行号、列号、值”。


### 1.3 代码实现：原始数组 ↔ 稀疏数组（完整流程）
#### 步骤1：原始数组转稀疏数组
```java
public class SparseArrayDemo {
    public static void main(String[] args) {
        // 1. 创建原始二维数组（模拟棋盘/矩阵，0为默认值）
        int[][] originalArray = new int[11][11]; // 11x11的棋盘
        originalArray[1][2] = 1; // 示例有效元素1
        originalArray[2][3] = 2; // 示例有效元素2
        originalArray[3][4] = 3; // 示例有效元素3

        // 2. 统计有效元素的个数
        int count = 0;
        for (int i = 0; i < originalArray.length; i++) {
            for (int j = 0; j < originalArray[i].length; j++) {
                if (originalArray[i][j] != 0) {
                    count++;
                }
            }
        }
        System.out.println("有效元素个数：" + count);

        // 3. 创建稀疏数组（行数=count+1，列数=3）
        int[][] sparseArray = new int[count + 1][3];
        // 稀疏数组第一行存储原始数组的“总行数、总列数、有效元素个数”
        sparseArray[0][0] = originalArray.length;
        sparseArray[0][1] = originalArray[0].length;
        sparseArray[0][2] = count;

        // 4. 遍历原始数组，将有效元素存入稀疏数组
        int index = 1; // 稀疏数组的行索引（从第1行开始存储有效元素）
        for (int i = 0; i < originalArray.length; i++) {
            for (int j = 0; j < originalArray[i].length; j++) {
                if (originalArray[i][j] != 0) {
                    sparseArray[index][0] = i; // 行号
                    sparseArray[index][1] = j; // 列号
                    sparseArray[index][2] = originalArray[i][j]; // 值
                    index++;
                }
            }
        }

        // 输出稀疏数组
        System.out.println("稀疏数组内容：");
        for (int i = 0; i < sparseArray.length; i++) {
            System.out.printf("%d\t%d\t%d\n", 
                sparseArray[i][0], sparseArray[i][1], sparseArray[i][2]);
        }
    }
}
```
**输出结果**：
```
有效元素个数：3
稀疏数组内容：
11	11	3
1	2	1
2	3	2
3	4	3
```


#### 步骤2：稀疏数组还原为原始数组
```java
public class SparseArrayDemo {
    public static void main(String[] args) {
        // （承接上文，假设已创建稀疏数组sparseArray）
        // 1. 从稀疏数组第一行获取原始数组的行列信息
        int rows = sparseArray[0][0];
        int cols = sparseArray[0][1];

        // 2. 创建原始数组
        int[][] originalArray = new int[rows][cols];

        // 3. 遍历稀疏数组，还原有效元素到原始数组
        for (int i = 1; i < sparseArray.length; i++) {
            int row = sparseArray[i][0];
            int col = sparseArray[i][1];
            int value = sparseArray[i][2];
            originalArray[row][col] = value;
        }

        // 输出还原后的原始数组
        System.out.println("还原后的原始数组：");
        for (int[] row : originalArray) {
            for (int num : row) {
                System.out.print(num + "\t");
            }
            System.out.println();
        }
    }
}
```
**输出结果**：
```
还原后的原始数组：
0	0	0	0	0	0	0	0	0	0	0	
0	0	1	0	0	0	0	0	0	0	0	
0	0	0	2	0	0	0	0	0	0	0	
0	0	0	0	3	0	0	0	0	0	0	
0	0	0	0	0	0	0	0	0	0	0	
...（剩余行均为0，此处省略）...
```


### 1.4 应用场景延伸
稀疏数组常用于**棋盘类游戏的存档/读档**（如五子棋、象棋，大部分位置无棋子，仅需存储有棋子的位置）、**大规模稀疏矩阵的存储与计算**（如科学计算中的稀疏矩阵），核心优势是**节省内存空间**，避免存储大量无意义的默认值。


## 二、回顾方法及加深——从定义到调用的完整逻辑
### 2.1 方法的定义（语法与构成）
方法是类的“行为单元”，用于封装可复用的逻辑。其定义格式为：
```java
修饰符 返回值类型 方法名(参数列表) throws 异常列表 {
    方法体;
    return 返回值; // 返回值类型为void时可省略
}
```

#### 各组成部分详解：
- **修饰符**：控制方法的访问权限（如`public`、`private`、`protected`、`static`等）；
- **返回值类型**：方法执行后返回的数据类型（如`int`、`String`、`void`（无返回值））；
- **break与return的区别**：
  - `break`：用于**跳出循环**或**switch分支**，不终止方法；
  - `return`：用于**终止方法执行**，并可选地返回一个值（返回值类型需匹配）；
- **方法名**：遵循驼峰命名法（如`calculateSum`），见名知意；
- **参数列表**：方法接收的输入数据，格式为`数据类型 参数名`（多个参数用逗号分隔）；
- **异常抛出**：用`throws`声明方法可能抛出的异常（如`IOException`），调用者需处理或继续抛出。


### 2.2 方法的调用（静态 vs 非静态，值传递 vs 引用传递）
#### 1. 静态方法 vs 非静态方法
- **静态方法**：用`static`修饰，**属于类**，无需创建对象即可调用（格式：`类名.方法名()`）；
  ```java
  public class MathUtil {
      public static int add(int a, int b) {
          return a + b;
      }
  }
  // 调用静态方法
  int result = MathUtil.add(1, 2); // 输出3
  ```
- **非静态方法**：无`static`修饰，**属于对象**，必须通过“对象.方法名()”调用；
  ```java
  public class Person {
      public void eat() {
          System.out.println("吃饭");
      }
  }
  // 调用非静态方法
  Person p = new Person();
  p.eat(); // 输出“吃饭”
  ```


#### 2. 形参 vs 实参
- **形参**：方法定义时的“形式参数”，仅为占位符（如`add(int a, int b)`中的`a`、`b`）；
- **实参**：方法调用时传递的“实际参数”，是具体的数值或对象（如`add(1, 2)`中的`1`、`2`）。


#### 3. 值传递 vs 引用传递（Java只有值传递）
Java中**所有参数传递都是“值传递”**——传递的是“实参值的副本”：
- **基本数据类型**：传递“数值的副本”，形参修改不影响实参；
  ```java
  public static void change(int a) {
      a = 100; // 仅修改形参副本
  }
  int num = 10;
  change(num);
  System.out.println(num); // 输出10（实参未变）
  ```
- **引用数据类型**：传递“对象地址的副本”，形参修改“对象属性”会影响实参（因指向同一对象），但修改“形参的指向”不影响实参；
  ```java
  class Student {
      String name;
  }
  public static void changeName(Student s) {
      s.name = "张三"; // 修改对象属性，实参受影响
      s = new Student(); // 修改形参指向，实参不受影响
      s.name = "李四";
  }
  Student stu = new Student();
  changeName(stu);
  System.out.println(stu.name); // 输出“张三”
  ```


#### 4. `this`关键字
`this`代表**当前对象**，用于：
- 区分**成员变量**和**方法参数**（重名时）；
- 调用**本类的其他构造器**（需放在构造器第一行）；
- 调用**本类的其他方法**；


```java
public class Person {
    private String name;

    // 区分成员变量和参数
    public void setName(String name) {
        this.name = name; // this.name指成员变量，name指参数
    }

    // 调用本类构造器
    public Person() {
        this("默认姓名"); // 调用有参构造器Person(String name)
    }

    public Person(String name) {
        this.name = name;
    }
}
```


### 2.3 方法重载（Overload）与重写（Override）区分
| 对比维度 | 方法重载（Overload） | 方法重写（Override） |
|----------|----------------------|----------------------|
| 定义位置 | 同一类中             | 子类继承父类后        |
| 方法名   | 必须相同             | 必须相同             |
| 参数列表 | 必须不同（个数、类型、顺序） | 必须相同             |
| 返回值类型 | 可相同可不同         | 子类返回值需≤父类（兼容） |
| 修饰符   | 无限制               | 子类修饰符需≥父类（访问权限） |

示例（重载）：
```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; } // 参数类型不同，重载
    public int add(int a, int b, int c) { return a + b + c; } // 参数个数不同，重载
}
```

示例（重写）：
```java
class Animal {
    public void eat() {
        System.out.println("动物吃饭");
    }
}
class Dog extends Animal {
    @Override // 注解表示重写
    public void eat() {
        System.out.println("狗吃骨头"); // 重写父类方法
    }
}
```


## 三、学习总结
- 稀疏数组是**空间优化的经典思路**，核心是“只存有效数据”，在稀疏场景下能大幅降低内存消耗；
- 方法是Java代码复用的核心单元，理解“定义、调用、值传递、this、重载/重写”是掌握面向对象编程的基础；


## 5. 面向对象（OOP）核心：从“模板”到“实例”
OOP是Java的灵魂，原笔记用“类是模板，对象是实例”精准概括。这部分我们从“类与对象的关系”入手，逐步拆解属性、方法、构造器。

### 5.1 类与对象的关系（类比理解）
- **类（Class）**：抽象的“模板”，定义了对象的“属性（数据）”和“方法（行为）”。比如“Person类”定义了人的“姓名、年龄”属性和“吃饭、睡觉”方法；  
- **对象（Object）**：类的“具体实例”，是真实存在的实体。比如“张三”是“Person类”的对象，“旺财”是“Pet类”的对象；  
- **类比**：类 = 汽车设计图，对象 = 按设计图造的具体汽车（同一设计图可造多辆汽车，同一类可创建多个对象）。

### 5.2 属性（成员变量）：对象的“数据”
#### 5.2.1 定义与默认初始化
属性定义在“类中、方法外”，若不手动赋值，会按类型默认初始化（和数组元素默认值一致）：

| 数据类型分类       | 具体类型                | 默认值          | 示例代码                     |
|--------------------|-------------------------|-----------------|------------------------------|
| 整数类型           | byte、short、int、long   | 0               | `int age; // age默认0`       |
| 浮点类型           | float、double           | 0.0             | `double score; // 默认0.0`  |
| 字符类型           | char                    | `\u0000`（空字符） | `char sex; // 默认\u0000`    |
| 布尔类型           | boolean                 | false           | `boolean isStudent; // 默认false` |
| 引用类型           | String、类、数组等       | null            | `String name; // 默认null`   |

#### 5.2.2 访问修饰符：控制属性的“访问权限”
为了实现封装，属性通常用`private`修饰（仅类内部访问），外部通过getter/setter访问。4种修饰符的权限对比：

| 修饰符       | 同一类中 | 同一包中 | 子类中 | 任意类中 | 常用场景                 |
|--------------|----------|----------|--------|----------|--------------------------|
| private      | ✅        | ❌        | ❌      | ❌        | 封装属性（推荐）         |
| 默认（无修饰）| ✅        | ✅        | ❌      | ❌        | 同包内共享（少用）       |
| protected    | ✅        | ✅        | ✅      | ❌        | 子类继承（框架中常用）   |
| public       | ✅        | ✅        | ✅      | ✅        | 公开常量（如`public static final int MAX = 100`） |

### 5.3 方法（成员方法）：对象的“行为”
方法定义了对象的“动作”，比如`Person`类的`eat()`方法表示“吃饭”行为。

#### 方法定义格式：
```java
修饰符 返回值类型 方法名(参数列表) {
    方法体; // 具体逻辑
    return 返回值; // 若返回值类型为void，可省略return
}
```

#### 方法分类（按参数和返回值）：
| 方法类型       | 示例代码                                  | 特点                                                                 |
|----------------|-------------------------------------------|----------------------------------------------------------------------|
| 无参无返回值   | `public void sleep() { System.out.println("睡觉"); }` | 无参数，无返回值，仅执行逻辑                                         |
| 无参有返回值   | `public String getName() { return name; }` | 无参数，返回指定类型值（如String）                                   |
| 有参无返回值   | `public void setName(String name) { this.name = name; }` | 有参数（如String name），无返回值，常用于修改属性                     |
| 有参有返回值   | `public int add(int a, int b) { return a + b; }` | 有参数，返回计算结果（如int）                                         |

### 5.4 构造器：“创建对象的必经之路”
原笔记强调“new本质在调用构造器”——构造器是创建对象的“特殊方法”，负责初始化对象的属性。

#### 5.4.1 构造器的3个特性：
1. 方法名与**类名完全相同**（大小写一致，如`Person`类的构造器名是`Person`）；  
2. 无返回值类型（无需写`void`，写了就成普通方法了）；  
3. 不能被`static`、`final`、`abstract`修饰（仅用于创建对象）。

#### 5.4.2 构造器的分类与重载：
- **无参构造器**：若类中未显式定义构造器，Java会自动生成“默认无参构造器”（方法体为空）；  
- **有参构造器**：显式定义，用于创建对象时直接给属性赋值（避免创建后再调用setter）；  
- **重载**：多个构造器的参数列表不同（个数、类型、顺序不同），允许用不同方式创建对象。

#### 实战示例：Person类的构造器重载
```java
class Person {
    // 属性（private封装）
    private String name;
    private int age;

    // 1. 无参构造器（显式定义，避免被有参构造器覆盖）
    public Person() {
        // 可手动初始化属性（也可默认）
        this.name = "未知姓名";
        this.age = 0;
    }

    // 2. 有参构造器1（仅传name）
    public Person(String name) {
        this.name = name; // this.name指成员变量，name指参数
        this.age = 0;
    }

    // 3. 有参构造器2（传name和age）
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getter/setter（外部访问属性的接口）
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        // 合法性校验（封装的体现）
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            System.out.println("⚠️ 年龄非法（0-150）");
        }
    }

    // 成员方法：自我介绍
    public void introduce() {
        System.out.println("我叫" + name + "，今年" + age + "岁。");
    }
}

// 测试构造器
public class PersonConstructorDemo {
    public static void main(String[] args) {
        // 用无参构造器创建对象
        Person p1 = new Person();
        p1.introduce(); // 输出：我叫未知姓名，今年0岁。

        // 用有参构造器1创建对象（仅传name）
        Person p2 = new Person("张三");
        p2.setAge(25); // 用setter修改age
        p2.introduce(); // 输出：我叫张三，今年25岁。

        // 用有参构造器2创建对象（传name和age）
        Person p3 = new Person("李四", 30);
        p3.setAge(200); // 年龄非法，不修改
        p3.introduce(); // 输出：我叫李四，今年30岁。
    }
}
```

#### 5.4.3 关键注意点（坑！）：
- **有参构造器覆盖默认无参**：若显式定义了有参构造器，Java不再自动生成默认无参构造器；若需用`new Person()`创建对象，**必须显式定义无参构造器**（否则编译报错）；  
- **`this`关键字的2个用法**：  
  1. `this.属性名`：区分成员变量和参数（如`this.name = name`）；  
  2. `this(参数)`：调用本类的其他构造器（必须放在构造器的第一行），示例：
     ```java
     public Person() {
         this("未知姓名", 0); // 调用有参构造器Person(String name, int age)
     }
     public Person(String name) {
         this(name, 0); // 调用有参构造器Person(String name, int age)
     }
     ```


## 6. 关键字static：“属于类，不属于对象”
原笔记总结得很到位：“静态化的内容不经过实例化就可以调用；非静态可以调用静态，但是静态不可以调用非静态”。

### 6.1 static的核心含义：
`static`修饰的内容（变量/方法/代码块）属于“类”，不属于“对象”，即**与类一起加载**（早于对象创建），所有对象共享静态内容。

### 6.2 静态变量（类变量）：
#### 特点：
- 存储位置：方法区的静态区域（不随对象存储）；  
- 访问方式：`类名.变量名`（推荐，体现静态属性），也可`对象.变量名`（不推荐，易混淆）；  
- 共享性：所有对象共享同一静态变量（修改一个对象的静态变量，其他对象访问时会看到修改后的值）。

#### 实战示例：
```java
class Student {
    static String school = "北京大学"; // 静态变量（所有学生共享）
    String name; // 非静态变量（每个学生独有）
}

public class StaticVariableDemo {
    public static void main(String[] args) {
        Student s1 = new Student();
        Student s2 = new Student();

        // 非静态变量：每个对象独有
        s1.name = "张三";
        s2.name = "李四";
        System.out.println("s1姓名：" + s1.name); // 张三
        System.out.println("s2姓名：" + s2.name); // 李四

        // 静态变量：所有对象共享
        System.out.println("初始学校：" + Student.school); // 北京大学
        s1.school = "清华大学"; // 修改s1的静态变量（本质是修改类的变量）
        System.out.println("s1的学校：" + s1.school); // 清华大学
        System.out.println("s2的学校：" + s2.school); // 清华大学（共享修改后的值）
        System.out.println("类访问学校：" + Student.school); // 清华大学（推荐方式）
    }
}
```

### 6.3 静态方法（类方法）：
#### 特点：
- 访问方式：`类名.方法名`（推荐，如`Math.max(10,20)`）；  
- 限制条件（核心！）：  
  1. 静态方法**不能直接访问非静态成员**（非静态成员属于对象，静态方法加载时对象未创建）；  
  2. 静态方法**不能使用this关键字**（this代表当前对象，静态方法无对象关联）；  
  3. 非静态方法**可以直接访问静态成员**（静态成员已随类加载）。

#### 实战示例：
```java
class StaticMethodDemo {
    static String staticStr = "静态变量"; // 静态变量
    String nonStaticStr = "非静态变量"; // 非静态变量

    // 静态方法
    public static void staticMethod() {
        System.out.println("静态方法访问静态变量：" + staticStr); // ✅ 允许
        // System.out.println("静态方法访问非静态变量：" + nonStaticStr); // ❌ 不允许
        // nonStaticMethod(); // ❌ 静态方法不能调用非静态方法
        // System.out.println(this.staticStr); // ❌ 静态方法不能用this
    }

    // 非静态方法
    public void nonStaticMethod() {
        System.out.println("非静态方法访问静态变量：" + staticStr); // ✅ 允许
        System.out.println("非静态方法访问非静态变量：" + nonStaticStr); // ✅ 允许
        staticMethod(); // ✅ 非静态方法可以调用静态方法
    }

    public static void main(String[] args) {
        // 调用静态方法（无需创建对象）
        StaticMethodDemo.staticMethod();

        // 调用非静态方法（必须创建对象）
        StaticMethodDemo demo = new StaticMethodDemo();
        demo.nonStaticMethod();
    }
}
```

### 6.4 静态代码块：“类加载时执行一次”
静态代码块用`static { ... }`定义，在**类加载时执行（仅执行一次）**，用于初始化静态变量或执行静态逻辑（比构造器先执行）。

#### 执行顺序示例：
```java
class StaticBlockDemo {
    // 静态代码块
    static {
        System.out.println("1. 静态代码块执行（类加载时）");
    }

    // 构造代码块（非静态，每个对象创建时执行）
    {
        System.out.println("2. 构造代码块执行（对象创建时）");
    }

    // 构造器
    public StaticBlockDemo() {
        System.out.println("3. 构造器执行（对象创建时）");
    }

    public static void main(String[] args) {
        System.out.println("4. main方法开始");
        new StaticBlockDemo(); // 创建第一个对象
        new StaticBlockDemo(); // 创建第二个对象
    }
}
```
**运行效果**（注意顺序）：
```
1. 静态代码块执行（类加载时）
4. main方法开始
2. 构造代码块执行（对象创建时）
3. 构造器执行（对象创建时）
2. 构造代码块执行（对象创建时）
3. 构造器执行（对象创建时）
```


## 7. 参数传递：Java只有“值传递”（纠正误区）
原笔记提到“值传递：传递的是值；引用传递：传递的是地址”，但**Java中只有“值传递”**——所谓的“引用传递”本质是“传递引用变量的值（即对象地址）”。

### 7.1 两种传递场景：
#### 1. 基本数据类型的传递：传递“值的副本”
- 原理：方法调用时，将实参的“具体数值”复制一份给形参，形参的修改**不影响实参**（实参和形参是独立变量）。

#### 示例：
```java
public class ValuePassBasic {
    public static void main(String[] args) {
        int a = 10; // 实参a（栈中存储10）
        change(a); // 传递a的值副本（10）给形参
        System.out.println("main中的a：" + a); // 输出10（实参未变）
    }

    public static void change(int a) { // 形参a（栈中存储10的副本）
        a = 20; // 修改形参a为20（仅修改副本）
        System.out.println("change中的a：" + a); // 输出20
    }
}
```

#### 2. 引用数据类型的传递：传递“地址的副本”
- 原理：方法调用时，将实参（引用变量）存储的“对象地址”复制一份给形参，形参和实参指向**堆中同一个对象**；  
- 关键：修改“形参指向的对象的属性”，会影响实参（同一对象）；但修改“形参本身的指向”（如`形参 = new 对象()`），不会影响实参。

#### 示例（原笔记Demo05修正）：
```java
class Person {
    String name; // 成员变量
}

public class ValuePassReference {
    public static void main(String[] args) {
        Person p = new Person(); // 实参p（栈中存储地址0x0001）
        p.name = "初始姓名";
        System.out.println("修改前name：" + p.name); // 初始姓名

        // 场景1：修改形参指向的对象的属性
        changeName(p);
        System.out.println("场景1后name：" + p.name); // 秦疆（对象属性被修改）

        // 场景2：修改形参本身的指向（new新对象）
        changePerson(p);
        System.out.println("场景2后name：" + p.name); // 秦疆（实参仍指向原对象）
    }

    // 场景1：修改对象属性
    public static void changeName(Person p) { // 形参p存储地址副本0x0001
        p.name = "秦疆"; // 修改堆中0x0001对象的name（实参也指向此对象）
    }

    // 场景2：修改形参指向
    public static void changePerson(Person p) { // 形参p存储地址副本0x0001
        p = new Person(); // 形参指向新对象（地址0x0002）
        p.name = "张三"; // 修改新对象的name（实参仍指向0x0001）
    }
}
```

### 7.2 核心结论（必须记住）：
- Java中只有“值传递”，传递的是“实参值的副本”；  
- 基本类型传递“数值副本”，形参修改不影响实参；  
- 引用类型传递“地址副本”，形参修改“对象属性”影响实参，修改“形参指向”不影响实参。


## 8. 面向对象三大特性之一：封装——“高内聚，低耦合”
原笔记用“活到老学到老”强调封装的重要性，封装是JavaBean规范的核心，目的是“保护数据安全，隐藏实现细节”。

### 8.1 封装的4个优点：
1. **提高安全性**：禁止外部直接修改属性（如用private修饰，避免赋值非法值）；  
2. **隐藏细节**：外部无需知道属性如何存储、方法如何实现，只需调用接口；  
3. **统一接口**：所有外部访问通过固定的getter/setter，修改内部逻辑不影响外部；  
4. **增强可维护性**：代码结构清晰，修改时只需改类内部，无需改所有调用处。

### 8.2 封装的实现步骤（JavaBean规范）：
1. **用private修饰属性**：禁止外部直接访问；  
2. **提供public的getter方法**：用于外部获取属性值（方法名：`get+属性名首字母大写`）；  
3. **提供public的setter方法**：用于外部修改属性值（方法名：`set+属性名首字母大写`），可加合法性校验；  
4. **类是public的**，有默认无参构造器（符合JavaBean规范）。

### 8.3 实战示例：封装Person类（含敏感信息处理）
```java
// 符合JavaBean规范的封装类
public class Person {
    // 1. private修饰属性（隐藏数据）
    private String name; // 姓名
    private int age;     // 年龄
    private String idCard; // 身份证号（敏感信息）

    // 2. 默认无参构造器（JavaBean规范）
    public Person() {
    }

    // 3. 有参构造器（可选，方便创建对象）
    public Person(String name, int age, String idCard) {
        this.name = name;
        this.age = age;
        this.idCard = idCard;
    }

    // 4. getter方法（获取属性值，敏感信息特殊处理）
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // 身份证号 getter：隐藏中间8位（敏感信息保护）
    public String getIdCard() {
        if (idCard != null && idCard.length() == 18) {
            return idCard.substring(0, 6) + "********" + idCard.substring(14);
        }
        return "未知";
    }

    // 5. setter方法（修改属性值，加合法性校验）
    public void setName(String name) {
        // 校验：姓名不为null且长度1-10
        if (name != null && name.length() >= 1 && name.length() <= 10) {
            this.name = name;
        } else {
            System.out.println("⚠️ 姓名非法（长度1-10）");
        }
    }

    public void setAge(int age) {
        // 校验：年龄0-150
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            System.out.println("⚠️ 年龄非法（0-150）");
        }
    }

    public void setIdCard(String idCard) {
        // 校验：身份证号为18位数字（正则表达式）
        if (idCard != null && idCard.matches("\\d{18}")) {
            this.idCard = idCard;
        } else {
            System.out.println("⚠️ 身份证号非法（18位数字）");
        }
    }

    // 6. 业务方法（封装逻辑）
    public void showInfo() {
        System.out.println("姓名：" + name + "，年龄：" + age + "，身份证号：" + getIdCard());
    }

    // 测试
    public static void main(String[] args) {
        Person p = new Person();
        p.setName("张三"); // 合法
        p.setAge(25);     // 合法
        p.setIdCard("110101199001011234"); // 合法
        p.showInfo(); // 输出：姓名：张三，年龄：25，身份证号：110101********1234

        // 测试非法赋值
        p.setName(""); // 非法
        p.setAge(200); // 非法
        p.setIdCard("123456"); // 非法
        p.showInfo(); // 输出不变（属性未被修改）
    }
}
```
>先更新到这里，下次更新继承和多态还有error相关的内容，准备刷牛客

---

> Author: 7M7  
> URL: http://localhost:1313/posts/javase/  


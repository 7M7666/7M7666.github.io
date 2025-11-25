# Java笔记2.0


<!--more-->
# Java 笔记2.0



## 一、super 与 this 关键字：对象与父类的“桥梁”
### 1. 核心本质与作用
- **this**：代表**当前对象实例**的引用，本质是“指向当前正在执行方法的对象”，用于访问本类的属性、方法、构造器。
- **super**：代表**父类对象的引用**（非父类实例，而是子类对象中父类的“部分”），用于访问父类的属性、方法、构造器，仅存在于继承场景。


### 2. 构造方法中的使用（重点）
#### （1）调用规则
- 无论是 `this()`（调用本类构造）还是 `super()`（调用父类构造），**必须放在构造方法的第一行**（否则编译报错）。
- `super()` 和 `this()` **不能同时出现**（因为第一行只能有一个，且两者都需占第一行）。
- 若子类构造中未显式调用 `super()`，编译器会默认添加 `super()`（调用父类无参构造），若父类无无参构造，则必须显式调用父类有参构造。

#### （2）代码示例
```java
// 父类
class Father {
    private String name;

    // 父类无参构造
    public Father() {
        System.out.println("Father 无参构造执行");
    }

    // 父类有参构造
    public Father(String name) {
        this.name = name;
        System.out.println("Father 有参构造执行，name=" + name);
    }
}

// 子类
class Son extends Father {
    private int age;

    // 子类无参构造
    public Son() {
        // 编译器默认添加 super()，调用父类无参构造
        System.out.println("Son 无参构造执行");
    }

    // 子类有参构造
    public Son(String fatherName, int age) {
        super(fatherName); // 显式调用父类有参构造（必须在第一行）
        this.age = age;    // this 访问本类属性
        System.out.println("Son 有参构造执行，age=" + age);
    }

    // this 调用本类其他构造
    public Son(int age) {
        this("张三", age); // 调用本类的 Son(String, int) 构造（必须在第一行）
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        Son son1 = new Son(); 
        // 输出：Father 无参构造执行 → Son 无参构造执行

        Son son2 = new Son("李四", 18); 
        // 输出：Father 有参构造执行，name=李四 → Son 有参构造执行，age=18
    }
}
```


### 3. 访问属性与方法的用法
| 场景                | 语法示例                  | 说明                                  |
|---------------------|---------------------------|---------------------------------------|
| 访问本类属性        | this.属性名               | 若局部变量与属性重名，必须用 this 区分 |
| 访问父类属性        | super.属性名              | 需保证父类属性访问权限（非 private）  |
| 调用本类方法        | this.方法名(参数)         | 可省略 this（无歧义时）               |
| 调用父类方法        | super.方法名(参数)        | 用于调用父类未被重写的方法            |

#### 代码示例
```java
class Father {
    protected String familyName = "张"; // 父类受保护属性

    public void sayFamily() {
        System.out.println("家族姓氏：" + familyName);
    }
}

class Son extends Father {
    private String familyName = "李"; // 子类与父类属性重名（隐藏父类属性）

    @Override
    public void sayFamily() {
        System.out.println("子类姓氏：" + this.familyName); // 访问子类属性
        System.out.println("父类姓氏：" + super.familyName); // 访问父类属性
        super.sayFamily(); // 调用父类方法
    }
}

public class Test {
    public static void main(String[] args) {
        Son son = new Son();
        son.sayFamily();
        // 输出：
        // 子类姓氏：李
        // 父类姓氏：张
        // 家族姓氏：张
    }
}
```


### 4. 核心区别对比
| 对比维度       | this                          | super                        |
|----------------|-------------------------------|------------------------------|
| 代表对象       | 当前对象实例                  | 父类对象的引用（子类中的父类部分） |
| 使用前提       | 无继承也可使用（只要是对象）  | 必须存在继承关系              |
| 访问构造器     | this()：调用本类构造          | super()：调用父类构造         |
| 访问属性/方法   | 优先访问本类，无则向上找      | 直接访问父类                  |
| 与构造器的关系 | 可调用本类其他构造，需在第一行 | 可调用父类构造，需在第一行    |


### 5. 易错点总结
1. **super() 必须在构造第一行**：若放在后面，编译直接报错（Java 语法强制）。
2. **父类无无参构造时，子类必须显式调用父类有参构造**：否则编译器默认添加的 `super()` 会找不到父类无参构造，报 `NoSuchMethodError`。
3. **this 不能在静态方法中使用**：静态方法属于类，无对象实例，而 this 依赖对象；super 同理，也不能在静态方法中使用。


## 二、静态（static）与非静态成员：类与对象的“边界”
### 1. 核心概念与内存模型
- **静态成员（static 修饰）**：包括静态属性、静态方法，属于**类本身**，存储在 **方法区的静态区**，仅加载一次（类加载时初始化），所有对象共享。
- **非静态成员**：包括非静态属性、非静态方法，属于**对象实例**，存储在 **堆内存**，每个对象有独立副本，需创建对象后访问。


### 2. 静态成员的特性与用法
#### （1）静态属性（类变量）
- 初始化时机：类加载时（早于对象创建），默认值与非静态属性一致（如 int 为 0，String 为 null）。
- 访问方式：`类名.静态属性名`（推荐）或 `对象.静态属性名`（不推荐，易混淆）。
- 应用场景：存储共享数据（如计数器、常量）。

#### （2）静态方法（类方法）
- 访问限制：**不能直接访问非静态成员**（非静态成员依赖对象，静态方法无对象），但可访问静态成员；不能使用 this/super。
- 访问方式：`类名.静态方法名(参数)`（推荐）或 `对象.静态方法名(参数)`。
- 应用场景：工具类方法（如 `Math.random()`、`Arrays.sort()`）、无状态方法（不依赖对象属性）。

#### 代码示例
```java
class Tool {
    // 静态属性（共享计数器）
    public static int count = 0;

    // 静态方法（工具方法）
    public static int add(int a, int b) {
        count++; // 静态方法访问静态属性
        return a + b;
    }

    // 非静态方法
    public void showCount() {
        System.out.println("调用次数：" + count); // 非静态方法可访问静态属性
    }
}

public class Test {
    public static void main(String[] args) {
        // 静态方法直接通过类名调用
        int sum1 = Tool.add(1, 2);
        int sum2 = Tool.add(3, 4);

        // 静态属性通过类名访问
        System.out.println("静态属性 count：" + Tool.count); // 输出 2

        // 非静态方法需创建对象调用
        Tool tool = new Tool();
        tool.showCount(); // 输出 调用次数：2
    }
}
```


### 3. 静态 vs 非静态：关键区别
| 对比维度       | 静态成员（static）            | 非静态成员                    |
|----------------|-------------------------------|-------------------------------|
| 所属对象       | 类本身                        | 对象实例                      |
| 内存位置       | 方法区静态区                  | 堆内存                        |
| 加载时机       | 类加载时                      | 对象创建时                    |
| 访问方式       | 类名.成员（推荐）             | 对象.成员（必须）              |
| 访问权限       | 不能访问非静态成员            | 可访问静态/非静态成员         |
| this/super     | 不能使用                      | 可以使用                      |
| 共享性         | 所有对象共享                  | 每个对象独立                  |


### 4. 静态方法与多态的关系（重点易错点）
**静态方法不参与多态**：方法调用时，编译器根据“引用的类型”（而非对象实际类型）确定调用的方法，即“静态绑定”（编译时确定）。

#### 代码示例（结合继承）
```java
class Father {
    public static void test() {
        System.out.println("Father 静态 test()");
    }
}

class Son extends Father {
    public static void test() {
        System.out.println("Son 静态 test()");
    }
}

public class Test {
    public static void main(String[] args) {
        Father f1 = new Father();
        Father f2 = new Son(); // 父类引用指向子类对象
        Son s1 = new Son();

        f1.test(); // 输出 Father 静态 test()（引用类型是 Father）
        f2.test(); // 输出 Father 静态 test()（引用类型是 Father，不触发多态）
        s1.test(); // 输出 Son 静态 test()（引用类型是 Son）
    }
}
```
**原因**：静态方法属于类，调用时与对象无关，仅看引用的编译时类型（左边定义的类型）。


### 5. 静态代码块（静态初始化块）
- 作用：初始化静态属性，仅执行一次（类加载时），优先级高于构造器。
- 语法：`static { 代码逻辑 }`。
- 执行顺序：多个静态代码块按定义顺序执行；静态代码块 → 构造代码块 → 构造器。

#### 代码示例
```java
class Person {
    static String country;

    // 静态代码块（初始化静态属性）
    static {
        country = "中国";
        System.out.println("静态代码块执行：初始化 country=" + country);
    }

    // 构造代码块（初始化非静态属性，每个对象创建时执行）
    {
        System.out.println("构造代码块执行");
    }

    // 构造器
    public Person() {
        System.out.println("构造器执行");
    }
}

public class Test {
    public static void main(String[] args) {
        Person p1 = new Person();
        // 输出：
        // 静态代码块执行：初始化 country=中国
        // 构造代码块执行
        // 构造器执行

        Person p2 = new Person();
        // 输出：
        // 构造代码块执行
        // 构造器执行（静态代码块仅执行一次）
    }
}
```


## 三、继承与方法重写：代码复用与个性化
### 1. 继承的核心概念与语法
- **定义**：子类通过 `extends` 关键字继承父类，复用父类的属性和方法，同时可扩展新功能。
- **顶层父类**：所有类默认继承 `java.lang.Object`（若未显式指定父类），即 Object 是 Java 类的根。
- **访问权限**：子类只能访问父类的非 private 成员（public > protected > default（同包）），private 成员需通过父类的 getter/setter 访问。
- **单继承限制**：Java 不支持多继承（一个类不能继承多个父类），但支持多层继承（如 A → B → C）。

#### 代码示例
```java
// 顶层父类（默认继承 Object）
class Animal {
    protected String name;
    private int age; // 私有属性，子类不能直接访问

    // 父类方法
    public void eat() {
        System.out.println(name + " 在吃东西");
    }

    // getter/setter 访问私有属性
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

// 子类继承 Animal
class Dog extends Animal {
    // 子类扩展新属性
    private String breed;

    // 子类扩展新方法
    public void bark() {
        System.out.println(name + "（品种：" + breed + "）在叫");
    }

    // 子类访问父类私有属性（通过 getter/setter）
    public void showAge() {
        System.out.println(name + " 的年龄：" + getAge());
    }
}

public class Test {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.name = "旺财"; // 访问父类 protected 属性
        dog.setAge(3);    // 通过父类 setter 设置私有属性
        dog.breed = "金毛";// 访问子类属性

        dog.eat();        // 调用父类方法
        dog.bark();       // 调用子类方法
        dog.showAge();    // 子类方法访问父类私有属性

        // 输出：
        // 旺财 在吃东西
        // 旺财（品种：金毛）在叫
        // 旺财 的年龄：3
    }
}
```


### 2. 方法重写（Override）：子类的“个性化改造”
#### （1）定义与目的
- 子类在继承父类后，对父类的**非静态、非 private、非 final** 方法进行重新实现，以满足子类的特定需求。
- 核心目的：在复用父类方法名的同时，修改方法逻辑（如 Animal 的 eat() 方法，Dog 重写为“吃骨头”，Cat 重写为“吃鱼”）。

#### （2）重写的严格规则（“两同两小一大”）
1. **两同**：
   - 方法名相同；
   - 参数列表（参数个数、类型、顺序）完全相同（若参数列表不同，是重载而非重写）。
2. **两小**：
   - 返回值类型：子类方法返回值 ≤ 父类方法返回值（若父类返回 Object，子类可返回 String；若父类返回 int，子类必须返回 int，基本类型需完全一致）；
   - 抛出异常：子类方法抛出的异常 ≤ 父类方法抛出的异常（异常范围更小，或相同）。
3. **一大**：
   - 访问修饰符：子类方法修饰符范围 ≥ 父类方法修饰符（如父类是 protected，子类可改为 public；父类是 public，子类必须是 public）。

#### （3）代码示例（符合规则的重写）
```java
import java.io.IOException;
import java.io.FileNotFoundException;

// 父类
class Father {
    // 父类方法：返回 Object，抛出 IOException，protected 修饰
    protected Object test(int num) throws IOException {
        System.out.println("父类 test()，num=" + num);
        return num;
    }
}

// 子类
class Son extends Father {
    // 重写：方法名、参数列表相同
    // 返回值：String（≤ Object），访问修饰符：public（≥ protected）
    // 抛出异常：FileNotFoundException（≤ IOException，是其子类）
    @Override // 注解：强制检查重写规则，不符合则编译报错（推荐添加）
    public String test(int num) throws FileNotFoundException {
        System.out.println("子类 test()，num=" + num);
        return "子类返回：" + num;
    }
}

public class Test {
    public static void main(String[] args) throws FileNotFoundException {
        Father father = new Son(); // 父类引用指向子类对象
        Object result = father.test(10);
        System.out.println(result); // 输出：子类返回：10
    }
}
```

#### （4）不能重写的方法
| 方法类型                | 原因                                  | 示例                          |
|-------------------------|---------------------------------------|-------------------------------|
| 静态方法（static）      | 属于类，不依赖对象，无多态性          | `public static void test()`   |
| 私有方法（private）     | 子类无法访问，无法重写                | `private void test()`         |
| final 方法              | final 修饰的方法禁止重写              | `public final void test()`    |
| 构造方法                | 子类有自己的构造方法，父类构造不继承  | 无                            |


### 3. 重写 vs 重载（Overload）：易混淆对比
| 对比维度       | 重写（Override）                  | 重载（Overload）                  |
|----------------|-----------------------------------|-----------------------------------|
| 定义场景       | 子类与父类之间                    | 同一类中（或子类与父类中，但不推荐） |
| 方法名         | 必须相同                          | 必须相同                          |
| 参数列表       | 必须相同                          | 必须不同（个数、类型、顺序）       |
| 返回值类型     | 子类 ≤ 父类（兼容）               | 可任意（与重载无关）              |
| 访问修饰符     | 子类 ≥ 父类                       | 可任意                            |
| 抛出异常       | 子类 ≤ 父类                       | 可任意                            |
| 与多态的关系   | 参与多态（动态绑定）              | 不参与多态（静态绑定）            |
| 核心目的       | 个性化改造父类方法                | 同一方法名处理不同参数场景        |

#### 代码示例（重载）
```java
class Calculator {
    // 重载 1：两个 int 参数
    public int add(int a, int b) {
        return a + b;
    }

    // 重载 2：三个 int 参数（参数个数不同）
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // 重载 3：两个 double 参数（参数类型不同）
    public double add(double a, double b) {
        return a + b;
    }
}

public class Test {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.add(1, 2));      // 调用重载 1，输出 3
        System.out.println(calc.add(1, 2, 3));   // 调用重载 2，输出 6
        System.out.println(calc.add(1.5, 2.5));  // 调用重载 3，输出 4.0
    }
}
```


### 4. 继承中的构造器执行顺序（重点）
- **核心规则**：子类构造器执行前，必须先执行父类构造器（保证父类的属性初始化完成）。
- **执行流程**：
  1. 执行父类静态代码块 → 子类静态代码块（类加载时，仅一次）；
  2. 执行父类构造代码块 → 父类构造器；
  3. 执行子类构造代码块 → 子类构造器。

#### 代码示例
```java
class Grandfather {
    static {
        System.out.println("Grandfather 静态代码块");
    }

    {
        System.out.println("Grandfather 构造代码块");
    }

    public Grandfather() {
        System.out.println("Grandfather 构造器");
    }
}

class Father extends Grandfather {
    static {
        System.out.println("Father 静态代码块");
    }

    {
        System.out.println("Father 构造代码块");
    }

    public Father() {
        System.out.println("Father 构造器");
    }
}

class Son extends Father {
    static {
        System.out.println("Son 静态代码块");
    }

    {
        System.out.println("Son 构造代码块");
    }

    public Son() {
        System.out.println("Son 构造器");
    }
}

public class Test {
    public static void main(String[] args) {
        new Son(); // 创建子类对象，触发所有初始化流程
    }
}
```
**输出结果**：
```
Grandfather 静态代码块
Father 静态代码块
Son 静态代码块
Grandfather 构造代码块
Grandfather 构造器
Father 构造代码块
Father 构造器
Son 构造代码块
Son 构造器
```


## 四、多态：Java 面向对象的“灵魂”
### 1. 多态的定义与核心条件
- **定义**：同一行为（方法调用），在不同对象上表现出不同的结果（如“吃”行为，Dog 吃骨头，Cat 吃鱼）。
- **本质**：**动态绑定**（运行时根据对象的实际类型，确定调用哪个类的方法）。
- **三大条件**（缺一不可）：
  1. 存在继承关系（子类继承父类）；
  2. 子类重写父类的非静态方法；
  3. 父类引用指向子类对象（`Father f = new Son();`，即“向上转型”）。


### 2. 多态的代码演示（核心场景）
```java
// 父类
class Animal {
    public void eat() {
        System.out.println("动物在吃东西");
    }

    public void sleep() {
        System.out.println("动物在睡觉");
    }
}

// 子类 1：Dog
class Dog extends Animal {
    @Override
    public void eat() { // 重写 eat()
        System.out.println("狗在吃骨头");
    }

    // 子类特有方法
    public void bark() {
        System.out.println("狗在叫");
    }
}

// 子类 2：Cat
class Cat extends Animal {
    @Override
    public void eat() { // 重写 eat()
        System.out.println("猫在吃鱼");
    }
}

public class Test {
    // 多态的核心应用：方法参数统一为父类类型，接收任意子类对象
    public static void feedAnimal(Animal animal) {
        animal.eat(); // 运行时根据实际对象类型，调用对应子类的 eat()
        animal.sleep(); // 未重写，调用父类方法
    }

    public static void main(String[] args) {
        // 父类引用指向子类对象（向上转型）
        Animal animal1 = new Dog();
        Animal animal2 = new Cat();

        // 调用方法：运行时绑定实际对象的方法
        feedAnimal(animal1); 
        // 输出：狗在吃骨头 → 动物在睡觉

        feedAnimal(animal2); 
        // 输出：猫在吃鱼 → 动物在睡觉

        // 注意：父类引用不能直接调用子类特有方法（需向下转型）
        // animal1.bark(); // 编译报错（Animal 类无 bark() 方法）
    }
}
```


### 3. 多态的三大注意事项
#### （1）多态仅针对方法，属性无多态
- 属性的访问由“引用的编译时类型”（左边定义的类型）决定，与对象实际类型无关。

#### 代码示例
```java
class Father {
    public String name = "Father"; // 父类属性
}

class Son extends Father {
    public String name = "Son"; // 子类属性（隐藏父类属性）
}

public class Test {
    public static void main(String[] args) {
        Father f = new Son();
        System.out.println(f.name); // 输出 Father（属性无多态，看引用类型）
    }
}
```

#### （2）避免类型转换异常（ClassCastException）
- 若父类引用指向的对象，与强制转换的子类类型不匹配，会抛出 `ClassCastException`。
- **解决方案**：转换前用 `instanceof` 关键字判断“对象是否是某个类的实例”，避免异常。

#### 代码示例
```java
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

public class Test {
    public static void main(String[] args) {
        Animal animal = new Dog(); // 父类引用指向 Dog 对象

        // 正确转换：对象实际类型是 Dog，转换为 Dog
        if (animal instanceof Dog) {
            Dog dog = (Dog) animal; // 向下转型（强制转换）
            System.out.println("转换为 Dog 成功");
        }

        // 错误转换：对象实际类型是 Dog，不能转换为 Cat
        if (animal instanceof Cat) {
            Cat cat = (Cat) animal;
        } else {
            System.out.println("animal 不是 Cat 的实例，无法转换");
        }
    }
}
```

#### （3）不参与多态的方法
- 静态方法、private 方法、final 方法：这些方法不支持重写，调用时由引用类型决定（静态绑定），不参与多态。


### 4. 多态中的类型转换（向上转型 vs 向下转型）
#### （1）向上转型（自动转换，安全）
- **语法**：`父类类型 引用名 = new 子类类型();`（如 `Animal a = new Dog();`）。
- **特点**：
  - 自动完成，无需强制转换；
  - 安全（子类是父类的“子集”）；
  - 局限性：父类引用不能访问子类的特有属性和方法。
- **应用场景**：统一方法参数类型（如 `feedAnimal(Animal animal)` 接收所有 Animal 子类对象）。

#### （2）向下转型（强制转换，需谨慎）
- **语法**：`子类类型 引用名 = (子类类型) 父类引用;`（如 `Dog d = (Dog) a;`）。
- **特点**：
  - 必须显式强制转换；
  - 不安全（需保证父类引用指向的对象是该子类类型）；
  - 目的：恢复子类特有属性和方法的访问权限。
- **核心原则**：只有当父类引用实际指向的是“该子类对象”时，向下转型才安全，否则抛 `ClassCastException`。


### 5. 多态的实际应用场景
1. **统一接口**：如定义 `List` 接口引用，指向 `ArrayList` 或 `LinkedList` 对象，切换实现类时无需修改调用代码（`List list = new ArrayList();` 或 `List list = new LinkedList();`）。
2. **简化代码**：如方法参数用父类类型，可接收所有子类对象，避免重载多个相似方法（如 `feedAnimal(Animal animal)` 无需写 `feedDog(Dog dog)`、`feedCat(Cat cat)`）。
3. **扩展性强**：新增子类时，无需修改原有代码（如新增 `Bird` 类继承 `Animal`，`feedAnimal` 方法可直接接收 `Bird` 对象）。


## 五、final 修饰符：Java 中的“不可变”关键字
### 1. final 的核心语义
- **中文含义**：最终的、不可变的。
- **作用对象**：可修饰类、方法、变量（包括局部变量、成员变量、静态变量），不同对象的作用不同。


### 2. final 修饰类：禁止继承（“断子绝孙”类）
- **语法**：`final class 类名 {}`。
- **特点**：该类不能被其他类继承，所有方法默认是 final 方法（无需显式修饰）。
- **应用场景**：确保类的不可变性，防止子类篡改核心逻辑（如 `java.lang.String`、`java.lang.Math` 都是 final 类）。

#### 代码示例
```java
final class StringUtil { // final 类，不能被继承
    public void reverse(String str) {
        // 字符串反转逻辑
    }
}

// class SubStringUtil extends StringUtil {} // 编译报错：无法继承 final 类
```


### 3. final 修饰方法：禁止重写
- **语法**：`public final 返回值类型 方法名(参数) {}`。
- **特点**：子类可以继承该方法，但不能重写（修改方法逻辑）。
- **应用场景**：父类中核心逻辑的方法（如工具类的核心算法），不允许子类修改。

#### 代码示例
```java
class Father {
    // final 方法，禁止子类重写
    public final void calculate() {
        System.out.println("父类核心计算逻辑");
    }
}

class Son extends Father {
    // @Override
    // public void calculate() {} // 编译报错：无法重写 final 方法
}
```


### 4. final 修饰变量：不可变（常量）
- **核心规则**：变量一旦初始化赋值，就不能再修改其值（基本类型）或引用（引用类型）。
- **分类与初始化要求**：

#### （1）final 局部变量
- 定义在方法、代码块中的变量，必须在**使用前**初始化（赋值），且赋值后不能修改。

#### 代码示例
```java
public class Test {
    public static void main(String[] args) {
        final int num1 = 10; // 声明时初始化
        // num1 = 20; // 编译报错：final 变量不能修改

        final int num2; // 声明时不初始化
        num2 = 30; // 使用前初始化
        // num2 = 40; // 编译报错
    }
}
```

#### （2）final 成员变量（非静态）
- 定义在类中、方法外的变量，必须在**对象创建完成前**初始化，有三种初始化方式：
  1. 声明时直接赋值；
  2. 在构造代码块中赋值；
  3. 在所有构造器中赋值（确保每个构造器都初始化）。

#### 代码示例
```java
class Person {
    // 方式 1：声明时直接赋值
    final String id = "123456";

    // 方式 2：构造代码块中赋值
    final int age;
    {
        age = 18;
    }

    // 方式 3：构造器中赋值（需确保所有构造器都赋值）
    final String name;
    public Person() {
        name = "张三";
    }
    public Person(String name) {
        this.name = name;
    }
}
```

#### （3）final 静态变量（类变量）
- 定义在类中、用 `static final` 修饰的变量（通常称为“常量”），必须在**类加载完成前**初始化，有两种方式：
  1. 声明时直接赋值；
  2. 在静态代码块中赋值。
- **命名规范**：常量名全部大写，多个单词用下划线分隔（如 `MAX_AGE`、`DB_URL`）。

#### 代码示例
```java
class Constants {
    // 方式 1：声明时直接赋值（推荐）
    public static final int MAX_AGE = 120;
    public static final String DB_URL = "jdbc:mysql://localhost:3306/test";

    // 方式 2：静态代码块中赋值
    public static final String VERSION;
    static {
        VERSION = "1.0.0";
    }
}

public class Test {
    public static void main(String[] args) {
        System.out.println(Constants.MAX_AGE); // 直接通过类名访问常量
    }
}
```

#### （4）final 修饰引用类型变量
- **注意**：不可变的是“引用地址”，而非引用指向的对象的属性。即引用不能指向新对象，但对象的属性可以修改。

#### 代码示例
```java
class User {
    private String name;
    public User(String name) {
        this.name = name;
    }
    // getter/setter
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

public class Test {
    public static void main(String[] args) {
        final User user = new User("张三"); // final 修饰引用类型

        // 允许：修改对象的属性
        user.setName("李四");
        System.out.println(user.getName()); // 输出 李四

        // 禁止：修改引用地址（指向新对象）
        // user = new User("王五"); // 编译报错
    }
}
```


### 5. final 的常见易错点
1. **final 成员变量未初始化**：若未在声明时、构造代码块、构造器中赋值，编译报错。
2. **final 引用类型变量的误解**：认为“final 修饰的对象不可变”，实际是引用不可变，对象属性可修改。
3. **final 与 static 的结合**：`static final` 是类级别的常量，仅加载一次；`final` 非静态是对象级别的常量，每个对象可能不同（如 `final String id`，每个对象的 id 不同，但赋值后不可改）。


## 六、抽象类与接口：Java 中的“规范与约束”
### 一、抽象类（abstract class）：约束与复用的结合
#### 1. 抽象类的定义与核心语义
- **语法**：用 `abstract` 关键字修饰的类，称为抽象类；用 `abstract` 修饰的方法，称为抽象方法（只有方法声明，无方法体）。
- **核心语义**：
  - 抽象类是“不完整的类”，不能直接创建对象（需子类继承并实现抽象方法）；
  - 抽象方法是“未实现的方法”，强制子类必须实现（除非子类也是抽象类）；
  - 作用：提供“约束”（抽象方法）和“复用”（普通方法、属性）的结合。

#### 2. 抽象类的核心规则
1. **抽象类不能直接 new**：`new AbstractClass();` 编译报错，必须通过子类实例化。
2. **有抽象方法的类必须是抽象类**：反之，抽象类可以没有抽象方法（全是普通方法）。
3. **子类继承抽象类的义务**：
   - 若子类不是抽象类，必须重写抽象类中的**所有**抽象方法；
   - 若子类是抽象类，可选择性重写抽象方法（未重写的抽象方法继续由子类的子类实现）。
4. **抽象类的组成**：可包含普通方法、抽象方法、属性（静态/非静态）、构造器（用于子类初始化父类属性）。

#### 3. 代码示例（抽象类的定义与使用）
```java
// 抽象类（有抽象方法）
abstract class Shape {
    // 普通属性
    protected String color;

    // 抽象类的构造器（用于子类初始化父类属性）
    public Shape(String color) {
        this.color = color;
    }

    // 抽象方法（强制子类实现）
    public abstract double calculateArea(); // 计算面积，无方法体

    // 普通方法（子类可直接复用）
    public void showColor() {
        System.out.println("形状颜色：" + color);
    }
}

// 子类 1：Circle（非抽象类，必须实现所有抽象方法）
class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color); // 调用抽象类的构造器
        this.radius = radius;
    }

    // 重写抽象方法：计算圆的面积
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// 子类 2：Rectangle（非抽象类，必须实现所有抽象方法）
class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    // 重写抽象方法：计算矩形的面积
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Test {
    public static void main(String[] args) {
        // 抽象类不能直接 new，需通过子类实例化
        Shape circle = new Circle("红色", 5);
        Shape rectangle = new Rectangle("蓝色", 4, 6);

        circle.showColor(); // 复用抽象类的普通方法
        System.out.println("圆的面积：" + circle.calculateArea()); // 调用重写的抽象方法

        rectangle.showColor();
        System.out.println("矩形的面积：" + rectangle.calculateArea());
    }
}
```
**输出结果**：
```
形状颜色：红色
圆的面积：78.53981633974483
形状颜色：蓝色
矩形的面积：24.0
```

#### 4. 抽象类的应用场景
- **模板方法模式**：抽象类定义“模板流程”（普通方法），将流程中的可变步骤定义为抽象方法，由子类实现（如定义“考试流程”：普通方法 `startExam()` 包含“发卷→答题→交卷”，其中“答题”是抽象方法，学生/老师子类实现不同逻辑）。
- **约束子类行为**：强制所有子类实现统一的方法（如 Shape 抽象类强制所有形状子类实现 `calculateArea()` 方法）。


### 二、接口（interface）：纯粹的“规范”
#### 1. 接口的定义与核心语义
- **语法**：用 `interface` 关键字定义，接口中的成员有特殊规则（Java 8 及以后有扩展）。
- **核心语义**：接口是“纯粹的规范”，不提供任何实现（Java 8 前），仅定义“必须做什么”，不关心“怎么做”。
- **作用**：解决 Java 单继承的限制（一个类可实现多个接口），实现“多继承”的效果；定义跨类的统一规范（如 `Runnable` 接口定义“可运行”的规范）。

#### 2. 接口的成员规则（按 Java 版本划分）
| Java 版本 | 成员类型                | 规则                                                                 |
|-----------|-------------------------|----------------------------------------------------------------------|
| Java 7 及前 | 抽象方法                | 默认为 `public abstract`（可省略，强制公开），无方法体                |
|           | 静态常量                | 默认为 `public static final`（可省略），必须初始化                    |
| Java 8+   | 默认方法（default）     | 用 `default` 修饰，有方法体，实现类可重写或直接使用                   |
|           | 静态方法（static）      | 用 `static` 修饰，有方法体，只能通过接口名调用（实现类不能继承）       |
| Java 9+   | 私有方法（private）     | 用 `private` 修饰，有方法体，仅接口内部的默认方法/静态方法可调用       |

#### 3. 接口的实现规则
1. **类实现接口**：用 `implements` 关键字，语法 `class 类名 implements 接口1, 接口2,... {}`。
2. **实现类的义务**：
   - 若类是普通类（非抽象类），必须重写接口中的**所有抽象方法**（包括继承的抽象方法）；
   - 若类是抽象类，可选择性重写抽象方法。
3. **多实现**：一个类可实现多个接口，解决单继承限制，但需注意“接口冲突”（如两个接口有同名抽象方法，实现类只需重写一次；若有同名默认方法，必须显式重写）。

#### 4. 代码示例（接口的定义与实现）
```java
// 接口 1：ShapeInterface（定义“形状”的规范）
interface ShapeInterface {
    // 静态常量（默认 public static final）
    double PI = Math.PI;

    // 抽象方法（默认 public abstract）
    double calculateArea(); // 计算面积
    double calculatePerimeter(); // 计算周长
}

// 接口 2：ColorInterface（定义“颜色”的规范）
interface ColorInterface {
    // 抽象方法
    void setColor(String color);
    String getColor();
}

// 类实现多个接口（多实现）
class Circle implements ShapeInterface, ColorInterface {
    private double radius;
    private String color;

    public Circle(double radius) {
        this.radius = radius;
    }

    // 重写 ShapeInterface 的抽象方法
    @Override
    public double calculateArea() {
        return PI * radius * radius; // 访问接口的静态常量
    }

    @Override
    public double calculatePerimeter() {
        return 2 * PI * radius;
    }

    // 重写 ColorInterface 的抽象方法
    @Override
    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String getColor() {
        return color;
    }
}

public class Test {
    public static void main(String[] args) {
        Circle circle = new Circle(5);
        circle.setColor("红色");

        System.out.println("颜色：" + circle.getColor());
        System.out.println("面积：" + circle.calculateArea());
        System.out.println("周长：" + circle.calculatePerimeter());
    }
}
```
**输出结果**：
```
颜色：红色
面积：78.53981633974483
周长：31.41592653589793
```

#### 5. Java 8+ 接口的新特性（默认方法与静态方法）
```java
interface MyInterface {
    // 抽象方法
    void abstractMethod();

    // 默认方法（Java 8+）：有方法体，实现类可直接使用
    default void defaultMethod() {
        System.out.println("接口的默认方法");
        privateMethod(); // 调用接口的私有方法（Java 9+）
    }

    // 静态方法（Java 8+）：只能通过接口名调用
    static void staticMethod() {
        System.out.println("接口的静态方法");
    }

    // 私有方法（Java 9+）：仅接口内部使用
    private void privateMethod() {
        System.out.println("接口的私有方法（辅助默认方法）");
    }
}

// 实现类
class MyImpl implements MyInterface {
    @Override
    public void abstractMethod() {
        System.out.println("实现类重写的抽象方法");
    }

    // 可选：重写默认方法
    @Override
    public void defaultMethod() {
        System.out.println("实现类重写的默认方法");
    }
}

public class Test {
    public static void main(String[] args) {
        MyImpl impl = new MyImpl();
        impl.abstractMethod(); // 调用重写的抽象方法
        impl.defaultMethod();  // 调用重写的默认方法

        // 调用接口的静态方法（必须通过接口名）
        MyInterface.staticMethod();
    }
}
```
**输出结果**：
```
实现类重写的抽象方法
实现类重写的默认方法
接口的静态方法
```


### 三、抽象类 vs 接口：核心区别
| 对比维度       | 抽象类（abstract class）              | 接口（interface）                          |
|----------------|---------------------------------------|-------------------------------------------|
| 关键字         | `abstract class`                      | `interface`                               |
| 继承/实现      | 子类用 `extends` 继承（单继承）        | 类用 `implements` 实现（多实现）          |
| 成员组成       | 可包含：抽象方法、普通方法、属性、构造器 | Java 7-：抽象方法、静态常量；Java 8+：新增默认/静态/私有方法 |
| 方法访问权限   | 可自定义（public/protected/default）   | 抽象方法默认 public；默认方法默认 public   |
| 构造器         | 有（用于子类初始化父类属性）          | 无（不能创建对象）                        |
| 实例化         | 不能直接 new，需子类实例化            | 不能直接 new，需实现类实例化              |
| 核心语义       | 约束 + 复用（既有规范，又有实现）      | 纯粹的规范（Java 8 前无实现，后有默认方法）|
| 应用场景       | 有共同属性和方法的类（如 Shape 及其子类） | 跨类的统一规范、多实现场景（如 `Runnable`、`Comparable`） |


## 七、异常处理：Java 程序的“容错机制”
### 1. 异常的定义与分类
- **定义**：异常是程序运行时发生的“不正常情况”（如除以零、空指针、文件未找到），会中断程序的正常执行流程。
- **Java 异常体系**：所有异常的根类是 `java.lang.Throwable`，分为两大分支：`Error`（错误）和 `Exception`（异常）。

#### （1）Throwable 体系结构
```
Throwable
├─ Error（错误）：JVM 层面的严重问题，程序无法处理
│  ├─ StackOverflowError（栈溢出）：如递归调用无终止条件
│  ├─ OutOfMemoryError（内存溢出）：如创建过多对象耗尽内存
│  └─ AWTError（GUI 相关错误）
│
└─ Exception（异常）：程序可处理的问题，分为检查性异常和运行时异常
   ├─ 检查性异常（Checked Exception）：编译时必须处理（try-catch 或 throws）
   │  ├─ IOException（IO 异常）：如文件未找到、读取失败
   │  ├─ SQLException（数据库异常）：如连接失败、SQL 语法错误
   │  └─ ClassNotFoundException（类未找到异常）
   │
   └─ 运行时异常（Unchecked Exception）：编译时可忽略，运行时可能发生
      ├─ NullPointerException（空指针异常）：调用 null 对象的方法/属性
      ├─ ArithmeticException（算术异常）：如除以零
      ├─ ArrayIndexOutOfBoundsException（数组下标越界）
      ├─ ClassCastException（类型转换异常）
      └─ IllegalArgumentException（非法参数异常）
```

#### （2）核心区别：Error vs Exception
| 对比维度       | Error（错误）                        | Exception（异常）                      |
|----------------|-------------------------------------|---------------------------------------|
| 性质           | JVM 层面的严重问题                  | 程序逻辑或环境问题                    |
| 处理能力       | 程序无法处理，只能避免              | 程序可处理（try-catch/throws）        |
| 编译检查       | 编译时不检查                        | 检查性异常编译时必须处理；运行时异常不检查 |
| 示例           | StackOverflowError、OutOfMemoryError | NullPointerException、IOException     |


### 2. 异常处理的核心关键字（5个）
Java 提供 `try`、`catch`、`finally`、`throw`、`throws` 五个关键字处理异常，核心流程是“监控异常→捕获异常→处理异常→善后工作”。

#### （1）try-catch：捕获并处理异常
- **语法**：
  ```java
  try {
      // 监控可能发生异常的代码块（“监控区”）
  } catch (异常类型1 异常变量名) {
      // 捕获并处理“异常类型1”的异常
  } catch (异常类型2 异常变量名) {
      // 捕获并处理“异常类型2”的异常（可多个 catch）
  }
  ```
- **规则**：
  - `try` 块不能单独存在，必须配合 `catch` 或 `finally`；
  - 多个 `catch` 块的异常类型需按“从小到大”排列（子类异常在前，父类异常在后），否则编译报错；
  - 异常变量名可通过 `e.getMessage()` 获取异常信息，`e.printStackTrace()` 打印异常堆栈（定位错误位置）。

#### 代码示例（try-catch 处理运行时异常）
```java
public class Test {
    public static void main(String[] args) {
        int a = 10;
        int b = 0;

        try {
            int result = a / b; // 可能发生 ArithmeticException（除以零）
            System.out.println("结果：" + result); // 异常后，此代码不执行
        } catch (ArithmeticException e) {
            // 处理算术异常
            System.out.println("捕获到算术异常：" + e.getMessage()); // 输出：/ by zero
            e.printStackTrace(); // 打印异常堆栈（推荐，便于调试）
        }

        System.out.println("程序继续执行（异常已处理）"); // 异常处理后，此代码正常执行
    }
}
```

#### （2）finally：善后工作（必执行）
- **语法**：在 `try-catch` 后添加 `finally` 块，无论 `try` 块是否发生异常，`finally` 块**一定执行**（除非 JVM 退出，如 `System.exit(0)`）。
- **应用场景**：释放资源（如关闭文件流、数据库连接、网络连接），避免资源泄漏。

#### 代码示例（try-catch-finally）
```java
import java.io.FileInputStream;
import java.io.IOException;

public class Test {
    public static void main(String[] args) {
        FileInputStream fis = null; // 定义在 try 外，确保 finally 可访问
        try {
            fis = new FileInputStream("test.txt"); // 可能发生 FileNotFoundException
            // 读取文件逻辑
        } catch (IOException e) {
            System.out.println("IO 异常：" + e.getMessage());
        } finally {
            // 无论是否发生异常，都关闭流（释放资源）
            if (fis != null) { // 避免空指针异常
                try {
                    fis.close(); // close() 也可能抛 IOException，需单独处理
                    System.out.println("文件流已关闭");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### （3）throw：主动抛出异常
- **语法**：在方法体中，用 `throw new 异常类(参数);` 主动抛出异常对象（通常用于“业务异常”，如参数非法）。
- **注意**：`throw` 抛出的是“异常对象”，而非异常类型；抛出后，后续代码不执行（需配合 `try-catch` 或 `throws` 处理）。

#### （4）throws：声明异常（交给调用者处理）
- **语法**：在方法声明处，用 `方法返回值类型 方法名(参数) throws 异常类型1, 异常类型2... {}`，声明该方法可能抛出的异常。
- **作用**：将异常“向上传递”，由方法的调用者处理（调用者需用 `try-catch` 捕获，或继续用 `throws` 向上传递）；仅适用于“检查性异常”和“未处理的运行时异常”。

#### 代码示例（throw 与 throws 结合）
```java
// 自定义业务异常（继承 Exception，属于检查性异常）
class IllegalAgeException extends Exception {
    public IllegalAgeException(String message) {
        super(message); // 调用父类构造器，传递异常信息
    }
}

class Person {
    private int age;

    // 方法声明可能抛出 IllegalAgeException（交给调用者处理）
    public void setAge(int age) throws IllegalAgeException {
        if (age < 0 || age > 150) {
            // 主动抛出异常（业务异常：年龄非法）
            throw new IllegalAgeException("年龄必须在 0-150 之间，当前：" + age);
        }
        this.age = age;
    }
}

public class Test {
    public static void main(String[] args) {
        Person person = new Person();
        try {
            person.setAge(200); // 调用声明异常的方法，必须处理
        } catch (IllegalAgeException e) {
            // 处理业务异常
            System.out.println("业务异常：" + e.getMessage()); // 输出：年龄必须在 0-150 之间，当前：200
        }
    }
}
```


### 3. 自定义异常：业务场景的“专属异常”
- **定义**：Java 内置异常无法满足所有业务需求（如“年龄非法”“余额不足”），需自定义异常类，继承 `Exception`（检查性异常）或 `RuntimeException`（运行时异常）。
- **步骤**：
  1. 创建自定义异常类，继承 `Exception` 或 `RuntimeException`；
  2. 提供构造器（通常重载两个：无参、带异常信息的有参，调用父类构造器）；
  3. 在业务逻辑中，用 `throw` 抛出自定义异常；
  4. 用 `try-catch` 捕获或 `throws` 声明处理。

#### 代码示例（完整自定义异常流程）
```java
// 1. 自定义运行时异常（继承 RuntimeException，编译时不检查）
class InsufficientBalanceException extends RuntimeException {
    // 无参构造器
    public InsufficientBalanceException() {
        super();
    }

    // 带异常信息的构造器（推荐）
    public InsufficientBalanceException(String message) {
        super(message);
    }
}

// 业务类：银行账户
class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    // 取款业务：余额不足时抛出自定义异常
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("取款金额必须大于 0"); // 内置运行时异常
        }
        if (amount > balance) {
            // 抛出自定义运行时异常
            throw new InsufficientBalanceException("余额不足！当前余额：" + balance + "，取款金额：" + amount);
        }
        balance -= amount;
        System.out.println("取款成功！剩余余额：" + balance);
    }
}

public class Test {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000);

        try {
            account.withdraw(1500); // 触发自定义异常
        } catch (InsufficientBalanceException e) {
            // 处理自定义异常
            System.out.println("取款失败：" + e.getMessage());
        } catch (IllegalArgumentException e) {
            // 处理内置异常
            System.out.println("参数错误：" + e.getMessage());
        }
    }
}
```
**输出结果**：
```
取款失败：余额不足！当前余额：1000.0，取款金额：1500.0
```


### 4. 异常处理的最佳实践
1. **避免吞掉异常**：不要在 `catch` 块中只打印日志却不处理，或空 `catch` 块（`catch (Exception e) {}`），会导致问题无法定位。
2. **优先捕获具体异常**：多个 `catch` 块按“子类异常在前，父类异常在后”排列，避免用 `catch (Exception e)` 捕获所有异常（不精准）。
3. **使用 try-with-resources 简化资源关闭**：Java 7+ 提供 `try-with-resources`，自动关闭实现 `AutoCloseable` 接口的资源（如流、连接），无需手动在 `finally` 关闭。
   ```java
   // try-with-resources 示例（自动关闭 FileInputStream）
   try (FileInputStream fis = new FileInputStream("test.txt")) {
       // 读取文件逻辑
   } catch (IOException e) {
       e.printStackTrace();
   }
   ```
4. **区分检查性异常与运行时异常**：
   - 检查性异常：用于“可恢复的异常”（如文件未找到，可提示用户检查路径）；
   - 运行时异常：用于“程序逻辑错误”（如空指针，需通过代码修复避免）。
5. **自定义异常用业务语义命名**：如 `InsufficientBalanceException`（余额不足）、`IllegalAgeException`（年龄非法），便于理解异常含义。


## 八、常用类与集合框架：Java 开发的“工具箱”
### 一、字符串相关类：String、StringBuffer、StringBuilder
#### 1. String 类：不可变的字符串
- **核心特性**：`String` 类被 `final` 修饰，不可继承；内部用 `char[]`（Java 9 前）或 `byte[]`（Java 9+）存储字符串，且数组被 `private final` 修饰，因此字符串**不可变**（一旦创建，值不能修改）。
- **不可变性的影响**：
  - 优点：线程安全（多线程访问无并发问题）、可缓存哈希值（`hashCode()` 计算后缓存，提升 `HashMap` 效率）；
  - 缺点：字符串拼接（`+`）会创建大量临时对象，效率低（推荐用 `StringBuilder`/`StringBuffer`）。

#### 2. StringBuffer：线程安全的可变字符串
- **核心特性**：继承 `AbstractStringBuilder`，内部用 `char[]` 存储，数组无 `final` 修饰，因此**可变**；方法用 `synchronized` 修饰，**线程安全**，但效率较低。
- **常用方法**：`append(...)`（追加）、`insert(int offset, ...)`（插入）、`reverse()`（反转）、`delete(int start, int end)`（删除）。

#### 3. StringBuilder：非线程安全的可变字符串
- **核心特性**：与 `StringBuffer` 同属 `AbstractStringBuilder` 子类，**可变**；但方法无 `synchronized` 修饰，**线程不安全**，效率比 `StringBuffer` 高（约 10-15 倍）。
- **应用场景**：单线程环境下的字符串拼接（如普通业务逻辑中的字符串组装）。

#### 4. 三者对比与选择
| 类名         | 可变性 | 线程安全 | 效率   | 底层存储（Java 9+） | 应用场景                  |
|--------------|--------|----------|--------|---------------------|---------------------------|
| String       | 不可变 | 安全     | 低（拼接时） | byte[]              | 字符串常量、少量拼接      |
| StringBuffer | 可变   | 安全（synchronized） | 中     | char[]              | 多线程环境（如服务器日志）|
| StringBuilder| 可变   | 不安全   | 高     | char[]              | 单线程环境（推荐优先用）  |

#### 5. 代码示例（字符串拼接效率对比）
```java
public class Test {
    public static void main(String[] args) {
        // 1. String 拼接（效率低，创建大量临时对象）
        long start1 = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < 10000; i++) {
            str += i; // 每次拼接都创建新 String 对象
        }
        long end1 = System.currentTimeMillis();
        System.out.println("String 拼接耗时：" + (end1 - start1) + "ms");

        // 2. StringBuilder 拼接（效率高）
        long start2 = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            sb.append(i); // 直接在原有数组上修改，无临时对象
        }
        long end2 = System.currentTimeMillis();
        System.out.println("StringBuilder 拼接耗时：" + (end2 - start2) + "ms");

        // 3. StringBuffer 拼接（效率中）
        long start3 = System.currentTimeMillis();
        StringBuffer sbf = new StringBuffer();
        for (int i = 0; i < 10000; i++) {
            sbf.append(i);
        }
        long end3 = System.currentTimeMillis();
        System.out.println("StringBuffer 拼接耗时：" + (end3 - start3) + "ms");
    }
}
```
**输出结果（参考）**：
```
String 拼接耗时：200ms
StringBuilder 拼接耗时：1ms
StringBuffer 拼接耗时：3ms
```


### 二、集合框架：Java 存储数据的“容器”
Java 集合框架（Collection Framework）是一组用于存储和操作对象的接口和类，位于 `java.util` 包下，核心分为 `Collection` 和 `Map` 两大体系。

#### 1. 集合框架体系图（核心部分）
```
java.util
├─ Collection（单列集合：存储单个对象）
│  ├─ List（有序、可重复、有索引）
│  │  ├─ ArrayList（动态数组，查询快、增删慢，线程不安全）
│  │  ├─ LinkedList（双向链表，增删快、查询慢，线程不安全）
│  │  └─ Vector（动态数组，线程安全，效率低，已过时）
│  │
│  ├─ Set（无序、不可重复、无索引）
│  │  ├─ HashSet（哈希表实现，无序，查询快，基于 equals() 和 hashCode() 去重）
│  │  ├─ LinkedHashSet（哈希表+链表，有序（插入顺序），去重）
│  │  └─ TreeSet（红黑树实现，自然排序或定制排序，去重）
│  │
│  └─ Queue（队列：先进先出，FIFO）
│     ├─ LinkedList（实现 Queue 接口，可作队列/栈）
│     └─ PriorityQueue（优先队列，按优先级出队）
│
└─ Map（双列集合：存储键值对（key-value））
   ├─ HashMap（哈希表实现，key 无序、唯一，线程不安全，查询快，JDK 8+ 底层是数组+链表/红黑树）
   ├─ LinkedHashMap（哈希表+链表，key 有序（插入顺序），唯一）
   ├─ TreeMap（红黑树实现，key 自然排序或定制排序，唯一）
   └─ Hashtable（哈希表实现，key 无序、唯一，线程安全，效率低，已过时）
```

#### 2. Collection 接口核心方法（List/Set 通用）
| 方法名                  | 作用                                  |
|-------------------------|---------------------------------------|
| `boolean add(E e)`      | 添加元素到集合                        |
| `boolean remove(Object o)` | 从集合中移除指定元素                  |
| `void clear()`          | 清空集合所有元素                      |
| `boolean contains(Object o)` | 判断集合是否包含指定元素              |
| `int size()`            | 返回集合元素个数                      |
| `boolean isEmpty()`     | 判断集合是否为空                      |
| `Object[] toArray()`    | 将集合转换为数组                      |
| `Iterator<E> iterator()`| 返回迭代器，用于遍历集合              |



---

> Author: 7M7  
> URL: http://localhost:1313/posts/54c6db0/  


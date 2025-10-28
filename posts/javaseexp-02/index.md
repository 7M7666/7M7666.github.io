# Javaseexp 02

# javaseexp-02
首先得有花色和牌面吧，这俩都是固定的，所以用了枚举。花色就叫Suit，放了黑桃、红桃、梅花、方块；牌面是Face，从二到A，每个牌面还带了个value，比如A是14，这样后面比大小方便。代码大概是这样的：

```java
public enum Face {
    二(2), 三(3), ..., A(14); // 中间省略了其他牌面
    private int value;
    Face(int value) {
        this.value = value;
    }
    public int getValue() { return value; }
}
```

然后是单张牌，用Card类来表示，里面就俩属性：suit（花色）和face（牌面）。构造方法传这俩参数，还得有getter方法拿属性。最开始写toString的时候直接用`suit + face`，结果打印出来不对，后来才知道枚举得用name()方法，改成`suit.name() + face.name()`就正常了，能显示“红桃七”这种格式。

```java
public class Card {
    private Suit suit;
    private Face face;
    public Card(Suit suit, Face face) {
        this.suit = suit;
        this.face = face;
    }
    // getter方法省略
    public String toString() {
        return suit.name() + face.name(); // 这里踩过坑，刚开始没加name()
    }
}
```

牌组的话，写了个Poke类，负责生成牌。构造方法里用嵌套循环，外层控制几副牌，中间循环花色，内层循环牌面，每次循环就new一个Card加进列表里。这样不管要1副还是2副，传个参数就行。

发牌和洗牌的功能，有Action接口，定义了shuffle、distribute、display这几个方法，然后让Game类去实现。洗牌简单，用Collections.shuffle直接打乱列表就行。发牌的时候卡了一下，怎么让牌轮流分给每个玩家呢？后来想到用取余，index是当前牌的序号，`index % player`就能得到该发给第几个玩家，比如3个玩家的话，0%3=0（玩家1），1%3=1（玩家2），2%3=2（玩家3），3%3=0（又回到玩家1），代码是这样的：

```java
public void distribute(ArrayList<Card> cards, ArrayList<ArrayList<Card>> playercards, int player) {
    int index = 0;
    for (Card card : cards) {
        playercards.get(index % player).add(card); // 取余实现轮流发牌
        index++;
    }
}
```

不过刚开始运行的时候报了空指针异常，调试了半天才发现，playercards里的每个玩家列表没初始化，只是new了个外层列表，里面的小列表还是null。后来在main里加了循环，给每个玩家new一个ArrayList，就好了：

```java
// 初始化玩家手牌列表，这里刚开始漏掉了，导致空指针
for (int i = 0; i < players; i++) {
    playercards.add(new ArrayList<>());
}
```

比大小的时候，直接拿两张牌的face.getValue()比较就行，这个逻辑简单，没出什么问题。

整个流程就是：让用户输入几副牌、几个玩家，然后生成牌组、洗牌、发牌，最后显示每个玩家的牌，再随机抽两张比大小。运行起来还挺顺畅的，就是写的时候有些细节没注意，踩了几个小坑，调试完感觉对枚举、接口还有集合的用法更熟了。

最重要的是理解：
- 先枚举
- 再定义接口
- 在重写实现方法

# 实验结果截图如下
![实验结果](https://youke1.picui.cn/s1/2025/10/28/6900a82995fe1.png)

---

> Author: 7M7  
> URL: http://localhost:1313/posts/javaseexp-02/  


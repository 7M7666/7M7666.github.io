# Java 扑克牌实验：枚举、集合与轮流发牌


这次实验要用 Java 模拟扑克牌，主要包括生成牌组、洗牌、发牌、显示手牌和比较大小。写的时候踩了几个小坑，调试完以后，对枚举、接口和集合的用法熟了不少。

<!--more-->

## 用枚举表示花色和牌面

花色和牌面都是固定集合，所以我先用了枚举。花色叫 `Suit`，包括黑桃、红桃、梅花、方块；牌面叫 `Face`，从二到 A，每个牌面带一个数值，后面比较大小会方便很多。

```java
public enum Face {
    二(2), 三(3), 四(4), 五(5), 六(6), 七(7),
    八(8), 九(9), 十(10), J(11), Q(12), K(13), A(14);

    private final int value;

    Face(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

## 单张牌与牌组

单张牌用 `Card` 类表示，里面是 `suit` 和 `face` 两个属性。默认情况下，枚举的 `toString()` 会返回枚举名称；这里显式调用 `name()`，是为了明确输出格式。

```java
public class Card {
    private final Suit suit;
    private final Face face;

    public Card(Suit suit, Face face) {
        this.suit = suit;
        this.face = face;
    }

    public Face getFace() {
        return face;
    }

    @Override
    public String toString() {
        return suit.name() + face.name();
    }
}
```

牌组用 `Poke` 类生成。构造方法里用嵌套循环：外层控制牌的副数，中间遍历花色，内层遍历牌面，每次创建一张 `Card` 放进列表。这样要一副还是两副牌，只需要改变构造参数。

## 洗牌和轮流发牌

我用 `Action` 接口定义 `shuffle`、`distribute` 和 `display`，再让 `Game` 实现。洗牌直接调用 `Collections.shuffle()`。

发牌时一开始没想好怎样轮流分给每个玩家，后来用牌的序号对玩家数取余：三个玩家时，`0 % 3`、`1 % 3`、`2 % 3` 分别对应前三个玩家，下一张牌的 `3 % 3` 又回到第一个玩家。

```java
public void distribute(
        ArrayList<Card> cards,
        ArrayList<ArrayList<Card>> playerCards,
        int players) {
    int index = 0;
    for (Card card : cards) {
        playerCards.get(index % players).add(card);
        index++;
    }
}
```

## 空指针问题

刚开始运行时遇到了空指针异常。外层的 `playerCards` 虽然创建了，但每个玩家自己的手牌列表还没有初始化。后来在发牌前补上循环：

```java
for (int i = 0; i < players; i++) {
    playerCards.add(new ArrayList<>());
}
```

比大小时，直接比较两张牌的 `face.getValue()` 即可。完整流程就是让用户输入牌的副数和玩家数，然后生成牌组、洗牌、发牌、显示手牌，最后随机抽两张牌比较大小。

这次实验最重要的不是把功能堆出来，而是把顺序理清楚：先用枚举表示固定数据，再设计牌和牌组，最后通过接口组织洗牌、发牌和显示行为。

## 实验结果

原文使用的外部图床已经无法解析，实验过程和关键输出先由上面的代码与文字保留。找到原始截图后，再补成本地静态资源，避免继续依赖失效外链。

## 相关文章

- [JavaSE 基础笔记（一）：输入、方法、数组与面向对象入门]({{< relref "posts/java笔记.md" >}})
- [JavaSE 基础笔记（二）：继承、多态、异常与集合]({{< relref "posts/java笔记2.0.md" >}})


---

> 作者: 7M7  
> URL: https://7m7666.github.io/posts/javaseexp-02/  


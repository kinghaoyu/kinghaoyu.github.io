---
title: Java设计模式之策略模式
date: 2018-09-30 11:21:24
tags:
---

​	

​	策略模式，它其实是**定义了一系列的算法簇，然后分别封装起来，让它们之间可以互相替换，此设计模式的变化独立于使用算法的客户。**

​	举一个例子：现在 Animal（动物类）,它有 看，呼吸，叫等的动作。现在让以面向对象的思想编写 Dolphin(海豚)，GoldFish（金鱼）类的实现。我们可能会想到这样：

```java
/**
 * 父类采用抽象类，把已经确定实现的方法给予实现，将不确定的暴露给子类去实现
 */
abstract class Animal{
    public void look(){
        System.out.println("look");
    }
    public abstract void breathe();
    public abstract void call();
}
```

​	那么 Animal 的子类将这样实现：

```java
class Dolphin extends Animal{

    @Override
    public void breathe() {
        System.out.println("Dolphin ... breathe");
    }

    @Override
    public void call() {
        System.out.println("Dolphin... call");
    }
}

class GoldFish extends Animal{

    @Override
    public void breathe() {
        System.out.println("GoldFish... breathe");
    }

    @Override
    public void call() {
        System.out.println("GoldFish... call");
    }
}
```

​	嗯~~~，这样看起来很完美，我才用了继承，将 look() 方法（不变的方法）从父类中继承下来，这样我就不用在写重复的代码啦！！！

​	但是，真的是这样吗？

​	现在是两个实现类，这样写还看不出来什么问题。假如说，有一天，老板说给我把动物园里的动物类都实现一遍？这时我们应该怎么办？

​	如果是上面的需求的话，会发现，我们会写大量的重复性代码（把 相同的 breathe(), call() 一遍又一遍的实现）。这时我们可以 **定义两个接口 BreatheBehavior,CallBehavior ，将它们作为 Animal 的属性添加进来，这样我们就可以委托这两个接口的实现类来完成 breathe 和 call 操作。而我们不需要知道它的具体实现。**代码如下：

1. 定义 BreatheBehavior,CallBehavior 接口

```java
public interface BreatheBehavior {
    /**
     * 呼吸
     */
    void breathe();
}

public interface CallBehavior {
    /**
     * 叫
     */
    void call();
}
```

1. 下面是它的实现类

```java
/**
 * 鳃呼吸
 */
public class LamellaBreatheBehavior implements BreatheBehavior {
    @Override
    public void breathe() {
        System.out.println("breathe with lamella");
    }
}


/**
 * 肺呼吸
 */
public class LungBreatheBehavior implements BreatheBehavior {
    @Override
    public void breathe() {
        System.out.println("breathe with lung");
    }
}
//=========================================================================================

/**
 * 不出声
 */
public class NoCallBehavior implements CallBehavior {
    @Override
    public void call() {
        System.out.println("no call");
    }
}

/**
 * 呱呱叫
 */
public class QuackCallBehavior implements CallBehavior {
    @Override
    public void call() {
        System.out.println("quack,quack,quack");
    }
}


/**
 * 尖叫
 */
public class SqueakCallBehavior implements CallBehavior {
    @Override
    public void call() {
        System.out.println("squeak,squeak,squeak");
    }
}
```

1. 动物类，**我们添加两个 set 方法 （**setBreatheBehavior(), setCallBehavior()**）将两个接口的实现类赋值给它们，其实用到的是多态。==在这里我们为什么不用构造方法为两个属性赋值呢？ 原因就是，这样做可以在程序运行时动态的改变对象的行为（调用 set 方法）==**

```java
/**
 * 动物类
 */
public class Animal {
    private BreatheBehavior breatheBehavior;
    private CallBehavior callBehavior;

    public void look(){
        System.out.println("look");
    }
    public void breathe(){
        breatheBehavior.breathe();
    }
    public void call(){
        callBehavior.call();
    }

    public void setBreatheBehavior(BreatheBehavior breatheBehavior) {
        this.breatheBehavior = breatheBehavior;
    }
    public void setCallBehavior(CallBehavior callBehavior){
        this.callBehavior = callBehavior;
    }
}
```

1. 下面就来看看我们的劳动成果吧！

```java
/**
 * @author wanghaoyu
 * @date 2018/9/29 - 19:07
 */
public class Main {
    public static void main(String[] args) {
        Animal goldFish = new GoldFish();
        goldFish.breathe();
        goldFish.call();
        //程序中临时指定，金鱼的叫声。
        goldFish.setCallBehavior(new SqueakCallBehavior());
        goldFish.call();
    }
}
```

​	执行结果：

![](/Java设计模式之策略模式/run.jpg)

​	那么这样设计有什么好处呢？ 我们来看，呼吸和叫的行为可以被其他对象复用，因为我们的动物，与那些叫和呼吸的行为其实本来没有太大的关系。而我们新增加或者减少那些行为类，其实那些对象也感知不到。我们要将变化的部分取出来并封装起来，好让其他部分不受影响。

​	设计模式有这样一个设计原则：**找出应用中可能变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起。**

​	我们如何将他分开来呢？我们可以利用接口来代表每个行为，比如 BreatheBehavior 与 CallBehavior ，而行为的每个实现都必须实现这些接口之一。所以这些动物不需要理会动作是怎么实现的，我只需要委托你这个接口在帮我实现就好。**针对接口编程，而不是这对实现编程。**在我们刚开始使用继承时，就是面向实现编程的，这样会导致代码强耦合，不利于扩展和维护。

​	**“有一个” 可能比 “是一个” 更好** ，使用组合建立系统具有很大的弹性，不仅可以将算法封装成类，更可以在**运行时动态地改变行为**，只要组合的行为对象符合正确的接口标准即可

​	以下就是策略模式的结构：

![](/Java设计模式之策略模式/strategy.png)

​	**构成**

- **Strategy**

  ---定义所有支持的算法的公共接口。Context 使用这个接口来调用某个 ConcreteStrategy 定义的算法。

- **ConcreteStrategy**

  ---以 Strategy 接口实现某具体算法。

- **Context**

  ---用一个 ConcreteStrategy 对象来配置。

  ---维护一个对 Strategy 对象的引用。

  ---可定义一个接口来让 Stategy 访问它的数据

**下面我们来看看它的适用性**

- 许多相关的类仅仅是行为有异。“策略” 提供了一种用多个行为中的一种来配置一个类的方法。

- 当程序需要实现一个特定行为，但该行为有多种实现方式时。

- 一个类定义了多种行为，并且这些行为在这个类的操作中以多个 if...else 。

  ​

**策略模式的优点**

- **相关算法系列**  Strategy 类层次为 Context 定义里一系列的可供重用的算法或行为。继承有助于析取出这些算法的公共功能。（例如 上例中 look() 方法）

- **可以替代继承的方法**  继承提供了另一种支持多种算法或行为的方法。你可以直接生成一个 Context 类的子类，从而给它以不同的行为。但者会将行为强加到 Context 中， 使算法的实现与 Context 的实现混合起来，从而使 Context 难以理解，难以维护，难以扩展，而且还不能动态的改变算法。最后你得到一堆相关的类，它们之间的唯一差别是它们所使用的算法或行为。将算法封装在独立的 Strategy 类中使得你可以独立于其 Context 改变它，使它易于切换，易于理解，易于扩展。

- **消除了一些条件语句**  Strategy 模式提供了用条件语句选择所需的行为之外的另一种选择。当不同的行为对其在一个类中时，很难避免使用条件语句来选择合适的行为。将行为封装在一个个独立的 Strategy 类中消除了这些条件语句。

- **实现的选择**  Strategy 模式可以提供相同行为的不同实现。客户可以根据不同时间/空间权衡取舍要求从不同策略中进行选择。

  ​	

**策略模式的缺点**

- **客户端必须了解不同的Strategy** 本模式有一个潜在的缺点，就是一个客户要选择一个合适的 Strategy 就必须知道这些 Strategy 到底有何不同。此时可能不得不向客户暴露具体的实现问题。因此仅当这些不同行为变体与客户相关的行为时，才需要使用 Strategy 模式。
- **Strategy 和 Context 之间的通信开销**   无论各个 ConcreteStrategy 实现的算法是简单还是复杂，它们都共享 Strategy 定义的接口。因此很可能某些 ConacreteStrategy 不会都用到所用通过这个接口传递给它们的信息；简单的 ConcreteStrategy 可能不使用其中的任何信息！这意味着有时 Context 会创建和初始化一些永远不会用到的参数。如果存在这样的问题，那么将需要在 Strategy 和 Context 之间更进行紧密的耦合。
- **在该模式下，会产生非常多的行为簇，导致项目中类的数目过多。**有时你可以将 Strategy 实现为可供各 Context 共享的无状态的对象来减少这一开销。任何其余状态都有 Context 维护。Context 在每次对 Strategy 对象的请求中对将这个状态传递过去，共享的 Strategy 不应在各次调用之间维护状态。

好的，讲完啦。



原文链接：https://www.baeldung.com/java-state-design-pattern

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)


## 1、概述

本教程，介绍一种行为型GOF设计模式：状态模式。
首先，我们介绍一下此模式的目的和阐述其试图解决的问题。然后看一下状态模式的UML类图和一个实例。

## 2、状态设计模式

状态模式的主要思想是：_允许object在不改变class的情况下改变自身的行为_。同时，通过实现状态模式，代码可以消除过多的if/else语句，能够保持代码整洁。

想象一下我们有一个Package送去邮局，Package可以自己排队，然后递送邮局并最终被客户签收。现在，基于真实状态，我们想要打印Package的递送状态。

最简单的方法就是在class的各个方法中用一些boolean标识和简单的if/else语句实现。这样不会使简单场景复杂化。然而，如果我们需要处理的状态很多，这种方式会使逻辑、代码很混乱，不易管理。

除此之外，每种状态的逻辑会贯穿整个方法。这种时候就可以考虑使用状态模式。借助状态模式，我们可以将逻辑封装到独立的class中，符合[单一责任原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)和[开放/封闭原则](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)，并使代码简洁易于维护。

## 3、UML类图

![UML类图](http://www.baeldung.com/wp-content/uploads/2018/08/State-1.png)

UML类图中，Context class拥有一个可以在程序执行时改变的关联State。

Context_代理State实现的行为_。换句话说，所有进入Context的请求由具体的State执行。

以上设计使得逻辑是分离的，并且很容易增加新state，只需要增加一个State实现。

## 4、实现

我们设计一个应用。如上所属，Package可以排队、传递和接收，从而我们我们可以设计三种State和Context class。
首先，定义Context，这是个Package类。

```java
public class Package {
 
    private PackageState state = new OrderedState();
 
    // getter, setter
 
    public void previousState() {
        state.prev(this);
    }
 
    public void nextState() {
        state.next(this);
    }
 
    public void printStatus() {
        state.printStatus();
    }
}
```

Package类包含一个管理State的引用，注意：我们通过previousState(),nextState()和printStatus()方法为State对象分派任务。State可以互相转换，_State对象的前两个方法可以基于传入的Context设置State的转换_。

Client与Package交互，我们不必处理State的转换，所有Client只需要执行下一个或者前一个State。

下面，我们抽象一个PackageState interface出来，用于定义三个行为。

```java
public interface PackageState {
 
    void next(Package pkg);
    void prev(Package pkg);
    void printStatus();
}
```

所有具体State class需要实现这个interface。

第一个具体State是OrderState:

```java
public class OrderedState implements PackageState {
 
    @Override
    public void next(Package pkg) {
        pkg.setState(new DeliveredState());
    }
 
    @Override
    public void prev(Package pkg) {
        System.out.println("The package is in its root state.");
    }
 
    @Override
    public void printStatus() {
        System.out.println("Package ordered, not delivered to the office yet.");
    }
}
```

从以上代码中，我们指定Package排序之后的下一个State。我们明确标明排序State是起始State，从这两个方法能够看到State如何转移。

下面是DeliveredState class:

```java
public class DeliveredState implements PackageState {
 
    @Override
    public void next(Package pkg) {
        pkg.setState(new ReceivedState());
    }
 
    @Override
    public void prev(Package pkg) {
        pkg.setState(new OrderedState());
    }
 
    @Override
    public void printStatus() {
        System.out.println("Package delivered to post office, not received yet.");
    }
}
```

同样能够看到State的联系。Package从排序State转移到投递State，printStatus()方法的消息也随之改变。

最后一个State是ReceivedState：

```java
public class ReceivedState implements PackageState {
 
    @Override
    public void next(Package pkg) {
        System.out.println("This package is already received by a client.");
    }
 
    @Override
    public void prev(Package pkg) {
        pkg.setState(new DeliveredState());
    }
}
```

这是最后一个State,我们只能回滚到上一个State。

由于一个State需要了解另一个State的情况，我们付出了一定的紧密耦合的代价。

## 5、测试

首先，来看一下对象的行为方式，是否如预期一般进行转换。

```java
@Test
public void givenNewPackage_whenPackageReceived_thenStateReceived() {
    Package pkg = new Package();
 
    assertThat(pkg.getState(), instanceOf(OrderedState.class));
    pkg.nextState();
 
    assertThat(pkg.getState(), instanceOf(DeliveredState.class));
    pkg.nextState();
 
    assertThat(pkg.getState(), instanceOf(ReceivedState.class));
}
```

然后，快速检查Package是否能够回退State。

```java
@Test
public void givenDeliveredPackage_whenPrevState_thenStateOrdered() {
    Package pkg = new Package();
    pkg.setState(new DeliveredState());
    pkg.previousState();
 
    assertThat(pkg.getState(), instanceOf(OrderedState.class));
}
```

最后，验证在运行时Package改变State后，其printStatus()方法的行为已经改变。

```java
public class StateDemo {
 
    public static void main(String[] args) {
 
        Package pkg = new Package();
        pkg.printStatus();
 
        pkg.nextState();
        pkg.printStatus();
 
        pkg.nextState();
        pkg.printStatus();
 
        pkg.nextState();
        pkg.printStatus();
    }
}
```

最终输出：

- Package ordered, not delivered to the office yet.
- Package delivered to post office, not received yet.
- Package was received by client.
- This package is already received by a client.
- Package was received by client.


当我们改变Context的State，Context的行为发生改变但是class并没有改变。

同时，当State转换后，class改变了他的State，从而改变了他的行为。

## 6、缺点

状态模式的缺点是在实现状态之间的转换时的代价。这使得State硬编码，这在一般情况下是不好的做法。

但是，根据我们的需求和要求，这可能是也可能不是问题。

## 7、状态模式vs策略模式

两者很相似，但相同的类图后面隐藏着细微的思想差别。

首先，[策略模式](https://www.baeldung.com/java-strategy-pattern)_定义了一组可互换的算法_。通常这些算法可达到同样的目的，但是实现有区别，比如排序和渲染算法。
状态模式，基于真实状态，_行为可以完全改变_。

并且，策略模式中，_调用者必须了解不同的策略并且明确的去改变使用_。反之，状态模式，状态互相连接，就像有限状态机一样转换。

## 8、总结

当我们想要_避免原始的if/else语句_时，状态设计模式很好用。我们_提取逻辑来分离class_，让我们的_Context对象代理_state class中实现的方法。此外，我们可以利用State之间的转换，其中一个State可以改变Context的State。

一般来说，这种设计模式对于相对简单的应用程序来说非常有用，但是对于更高级的程序，我们可以看一下[Spring的State Machine教程](https://www.baeldung.com/spring-state-machine)。

像往常一样，完整的代码可以在[GitHub项目](https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns)中找到。
        

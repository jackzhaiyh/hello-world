# Java8中的策略设计模式

原文链接：https://www.baeldung.com/java-state-design-pattern

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1、简介
在本文中，我们将了解如何在Java 8中实现策略设计模式。

首先，我们将概述该模式，并解释它是如何在旧版本的Java中传统实现的。

接下来，我们将再次尝试该模式，这次只使用Java 8 lambdas，减少了代码的冗长。


## 2、策略模式
本质上，_策略模式允许我们在运行时更改算法的行为。_

通常，我们将从用于应用算法的接口开始，然后针对每种可能的算法多次实现它。

假设我们要求在购买时应用不同类型的折扣，具体取决于是圣诞节，复活节还是新年。 首先，让我们创建一个Discounter接口，它将由我们的每个策略实现：
```java
public interface Discounter {
    BigDecimal applyDiscount(BigDecimal amount);
}
```
那么假设我们想要在复活节期间享受50％的折扣，在圣诞节期间享受10％的折扣。 让我们为每个策略实现我们的接口：

```java
public static class EasterDiscounter implements Discounter {
    @Override
    public BigDecimal applyDiscount(final BigDecimal amount) {
        return amount.multiply(BigDecimal.valueOf(0.5));
    }
}

public static class ChristmasDiscounter implements Discounter {
   @Override
   public BigDecimal applyDiscount(final BigDecimal amount) {
       return amount.multiply(BigDecimal.valueOf(0.9));
   }
}
```
最后，让我们在测试中尝试一个策略：
```java
Discounter easterDiscounter = new EasterDiscounter();

BigDecimal discountedValue = easterDiscounter
  .applyDiscount(BigDecimal.valueOf(100));

assertThat(discountedValue)
  .isEqualByComparingTo(BigDecimal.valueOf(50));
```
这很有效，但问题是必须为每个策略创建一个具体的类可能会有点痛苦。 另一种方法是使用匿名内部类型，但这仍然非常冗长，并且不像以前的解决方案那么方便：
```java
Discounter easterDiscounter = new Discounter() {
    @Override
    public BigDecimal applyDiscount(final BigDecimal amount) {
        return amount.multiply(BigDecimal.valueOf(0.5));
    }
};
```

## 3、借助Java8
自从Java 8发布以来，lambdas的引入使匿名内部类型或多或少变得多余。 这意味着在线创建策略现在变得更加清洁和简单。

此外，函数式编程的声明式风格使我们能够实现以前不可能的模式。

### 3.1减少代码详细程度
让我们尝试创建一个内联EasterDiscounter，这次只使用lambda表达式：
```java
Discounter easterDiscounter = amount -> amount.multiply(BigDecimal.valueOf(0.5));
```
正如我们所看到的，我们的代码现在更清晰，更易于维护，实现与以前相同，但只需一行。 从本质上讲，_lambda可以被视为匿名内部类型的替代品。_

当我们想要在线声明更多Discounters时，这个优势变得更加明显：

```java
List<Discounter> discounters = newArrayList(
  amount -> amount.multiply(BigDecimal.valueOf(0.9)),
  amount -> amount.multiply(BigDecimal.valueOf(0.8)),
  amount -> amount.multiply(BigDecimal.valueOf(0.5))
);
```

当我们想要定义大量的折扣器时，我们可以在一个地方静态地声明它们。 如果我们愿意，Java 8甚至可以让我们在接口中定义静态方法。

因此，不要在具体类或匿名内部类型之间进行选择，而是尝试在单个类中创建lambdas：

```java
public interface Discounter {
    BigDecimal applyDiscount(BigDecimal amount);

    static Discounter christmasDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.9));
    }

    static Discounter newYearDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.8));
    }

    static Discounter easterDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.5));
    }
}
```
正如我们所看到的，我们在一个不太多的代码中取得了很多成就。

### 3.2利用功能组合
让我们修改我们的Discounter接口，使其扩展UnaryOperator接口，然后添加combine（）方法：
```java
public interface Discounter extends UnaryOperator<BigDecimal> {
    default Discounter combine(Discounter after) {
        return value -> after.apply(this.apply(value));
    }
}
```
本质上，我们正在重构我们的Discounter，并利用一个事实，即应用折扣是一个将BigDecimal实例转换为另一个BigDecimal实例的函数，允许我们访问预定义的方法。 _由于UnaryOperator附带了apply（）方法，我们可以用它替换applyDiscount。_

combine（）方法只是将一个Discounter应用于此结果的抽象。 它使用内置的函数apply（）来实现这一点。

现在，让我们尝试累计应用多个折扣器到一定数量。 我们将通过使用函数reduce（）和combine（）来完成此操作：

```java
Discounter combinedDiscounter = discounters
  .stream()
  .reduce(v -> v, Discounter::combine);

combinedDiscounter.apply(...);
```
特别注意第一个reduce参数。 如果没有提供折扣，我们需要返回未更改的值。 这可以通过提供身份功能作为默认折扣器来实现。

这是执行标准迭代的有用且不那么详细的替代方法。 如果我们考虑我们开箱即用的功能组合方法，它还可以免费提供更多功能。

## 4、总结
在本文中，我们已经解释了策略模式，并且还演示了如何使用lambda表达式以更简洁的方式实现它。

可以在[GitHub上找到这些示例的实现](https://github.com/eugenp/tutorials/tree/master/core-java-8)。 这是一个基于Maven的项目，所以应该很容易按原样运行。

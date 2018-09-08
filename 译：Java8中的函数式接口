# Java8中的函数式接口

原文链接：https://www.baeldung.com/java-8-functional-interfaces

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1、简介
本文是Java 8中不同功能的函数式接口的指南，它们的一般用例和标准JDK库中的用法。

## 2、Java8 Lambdas
Java 8以lambda表达式的形式带来了强大的新语法改进。 lambda是一个匿名函数，可以作为变量处理，例如传递给方法或从方法返回。

在Java 8之前，您通常会为需要封装单个功能的每种情况创建一个类。 这暗示了很多不必要的样板代码来定义充当原始函数表示的东西。

文章：[Lambda Expressions and Functional Interfaces: Tips and Best Practices]（https://www.baeldung.com/java-8-lambda-expressions-tips）中描述了Lambda，函数式接口和使用它们的最佳实践。 本指南重点介绍java.util.function包中存在的一些特定的函数式接口。

## 3、函数式接口
所有函数式接口都有一个@FunctionalInterface注解。 这不仅清楚地传达了此接口的用途，而且如果带注解的接口不满足条件，也允许编译器生成错误。

任何具有SAM（单一抽象方法）的接口都是一个函数式接口，其实现可以被当作lambda表达式处理。

请注意，Java 8的默认方法不是抽象的，不包括：功能接口可能仍然有多个默认方法。 您可以通过查看Function的文档来观察这一点。

## 4、函数
lambda的最简单和一般的情况是一个函数接口，其方法接收一个值并返回另一个值。 单个参数的此函数由Function接口表示，该接口由其参数类型和返回值参数化：

```java
public interface Function<T, R> { … }
```
标准库中Function类型的一个用法是Map.computeIfAbsent方法，该方法通过键从映射返回值，但如果映射中尚未存在键，则计算值。 要计算值，它使用传递的Function实现：

```java
Map<String, Integer> nameMap = new HashMap<>();
Integer value = nameMap.computeIfAbsent("John", s -> s.length());
```
在这种情况下，值将通过将函数应用于键，放入映射并从方法调用返回来计算。 顺便说一句，我们可以用匹配传递和返回值类型的方法引用替换lambda。

请记住，调用该方法的对象实际上是方法的隐式第一个参数，它允许将实例方法长度引用转换为Function接口：
```java
Integer value = nameMap.computeIfAbsent("John", String::length);
```
Function接口还有一个默认的compose方法，它允许将多个函数合并为一个并按顺序执行它们：

```java
Function<Integer, String> intToString = Object::toString;
Function<String, String> quote = s -> "'" + s + "'";

Function<Integer, String> quoteIntToString = quote.compose(intToString);

assertEquals("'5'", quoteIntToString.apply(5));
```
quoteIntToString函数是应用于intToString函数结果的quote函数的组合。

## 5、原始函数专业化
由于基本类型不能是泛型类型参数，因此对于大多数使用的基本类型double，int，long以及它们在参数和返回类型中的组合，都有函数式接口的版本：
- IntFunction，LongFunction，DoubleFunction：参数是指定类型，返回类型是参数化的
- ToIntFunction，ToLongFunction，ToDoubleFunction：返回类型是指定类型，参数是参数化的
- DoubleToIntFunction，DoubleToLongFunction，IntToDoubleFunction，IntToLongFunction，LongToIntFunction，LongToDoubleFunction - 将参数和返回类型定义为基本类型，由其名称指定

没有开箱即用的函数式接口，例如，输入一个short并返回一个byte的函数，但没有什么能阻止你自己编写一个：
```java
@FunctionalInterface
public interface ShortToByteFunction {

    byte applyAsByte(short s);

}
```
现在我们可以编写一个方法，使用ShortToByteFunction定义的规则将short数组转换为byte数组：
```java
public byte[] transformArray(short[] array, ShortToByteFunction function) {
    byte[] transformedArray = new byte[array.length];
    for (int i = 0; i < array.length; i++) {
        transformedArray[i] = function.applyAsByte(array[i]);
    }
    return transformedArray;
}
```
以下是我们如何使用它将short数组乘以2之后转换为byte数组：
```java
short[] array = {(short) 1, (short) 2, (short) 3};
byte[] transformedArray = transformArray(array, s -> (byte) (s * 2));

byte[] expectedArray = {(byte) 2, (byte) 4, (byte) 6};
assertArrayEquals(expectedArray, transformedArray);
```

## 6、双精度函数专业化
要使用两个参数定义lambda，我们必须使用名称中包含“Bi”关键字的其他接口：BiFunction，ToDoubleBiFunction，ToIntBiFunction和ToLongBiFunction。

BiFunction同时具有参数和返回类型，而ToDoubleBiFunction和其他允许您返回原始值。

在标准API中使用此接口的典型示例之一是Map.replaceAll方法，该方法允许使用某个计算值替换映射中的所有值。

让我们使用BiFunction实现，它接收一个键和一个旧值来计算工资的新值并返回它。
```java
Map<String, Integer> salaries = new HashMap<>();
salaries.put("John", 40000);
salaries.put("Freddy", 30000);
salaries.put("Samuel", 50000);

salaries.replaceAll((name, oldValue) ->
  name.equals("Freddy") ? oldValue : oldValue + 10000);
```

## 7、Suppliers
Supplier函数式接口是另一个不带任何参数的专有函数。 它通常用于延迟生成值。 例如，让我们定义一个对double值进行平方的函数。 它本身不会获得价值，而是具有此价值的Supplier：
```java
public double squareLazy(Supplier<Double> lazyValue) {
    return Math.pow(lazyValue.get(), 2);
}
```
这允许我们懒惰地使用Supplier实现生成用于调用此函数的参数。 如果生成此参数需要相当长的时间，这可能很有用。 我们将使用Guava的sleepUninterruptibly方法模拟：
```java
Supplier<Double> lazyValue = () -> {
    Uninterruptibles.sleepUninterruptibly(1000, TimeUnit.MILLISECONDS);
    return 9d;
};

Double valueSquared = squareLazy(lazyValue);
```
Supplier的另一个用例是定义序列生成的逻辑。 为了演示它，让我们使用静态Stream.generate方法来创建Fibonacci数字流：
```java
int[] fibs = {0, 1};
Stream<Integer> fibonacci = Stream.generate(() -> {
    int result = fibs[1];
    int fib3 = fibs[0] + fibs[1];
    fibs[0] = fibs[1];
    fibs[1] = fib3;
    return result;
});
```
传递给Stream.generate方法的函数实现了Supplier功能接口。 请注意，Supplier通常需要某种外部状态作为生成器。 在这种情况下，其状态由两个最后的斐波纳契序列号组成。

为了实现这个状态，我们使用数组而不是几个变量，_因为Lambda中使用的所有外部变量必须是有效的_。

Supplier函数接口的其他特例包括BooleanSupplier，DoubleSupplier，LongSupplier和IntSupplier，其返回类型是相应的原语。

## 8、Consumers
与Supplier相反，Consumer接受一个普遍的论点并且不返回任何内容。 它是一种代表副作用的函数。

例如，让我们通过在控制台中打印问候语来问候名单中的每个人。 传递给List.forEach方法的lambda实现了Consumer函数接口：
```java
List<String> names = Arrays.asList("John", "Freddy", "Samuel");
names.forEach(name -> System.out.println("Hello, " + name));
```
还有一些特殊版本的Consumer - DoubleConsumer，IntConsumer和LongConsumer--它们接收原始值作为参数。 更有趣的是BiConsumer界面。 其中一个用例是遍历地图的条目：
```java
Map<String, Integer> ages = new HashMap<>();
ages.put("John", 25);
ages.put("Freddy", 24);
ages.put("Samuel", 30);

ages.forEach((name, age) -> System.out.println(name + " is " + age + " years old"));
```
另一组专有的BiConsumer版本由ObjDoubleConsumer，ObjIntConsumer和ObjLongConsumer组成，它们接收两个参数，其中一个是基本的，另一个是基本类型。

## 9、Predicates
在数学逻辑中，predicate是一个接收值并返回布尔值的函数。

Predicate函数接口是一个Function的特化，它接收一个泛化值并返回一个布尔值。 Predicate lambda的典型用例是过滤一组值：
```java
List<String> names = Arrays.asList("Angela", "Aaron", "Bob", "Claire", "David");

List<String> namesWithA = names.stream()
  .filter(name -> name.startsWith("A"))
  .collect(Collectors.toList());
```
在上面的代码中，我们使用Stream API过滤列表，并仅保留以字母“A”开头的名称。 过滤逻辑封装在Predicate实现中。

与前面的所有示例一样，此函数的IntPredicate，DoublePredicate和LongPredicate版本接收原始值。

## 10、Operators
Operator接口是接收和返回相同值类型的函数的特殊情况。 UnaryOperator接口接收单个参数。 Collections API中的一个用例是用一些相同类型的计算值替换列表中的所有值：
```java
List<String> names = Arrays.asList("bob", "josh", "megan");

names.replaceAll(name -> name.toUpperCase());
```
List.replaceAll函数返回void，因为它替换了适当的值。 为了达到目的，用于转换列表值的lambda必须返回与它接收的结果类型相同的结果类型。 这就是UnaryOperator在这里很有用的原因。

当然，您可以简单地使用方法引用代替name - > name.toUpperCase（）：
```java
names.replaceAll(String::toUpperCase);
```
BinaryOperator最有趣的用例之一是还原操作。 假设我们想要以所有值的总和聚合整数集合。 使用Stream API，我们可以使用收集器来完成此操作，但更通用的方法是使用reduce方法：
```java
List<Integer> values = Arrays.asList(3, 5, 8, 9, 12);

int sum = values.stream()
  .reduce(0, (i1, i2) -> i1 + i2);
```
reduce方法接收初始累加器值和BinaryOperator函数。 此函数的参数是一对相同类型的值，函数本身包含一个逻辑，用于将它们连接到同一类型的单个值中。 _传递的函数必须是关联的_，这意味着值聚合的顺序无关紧要，即以下条件应该成立：

```java
op.apply(a, op.apply(b, c)) == op.apply(op.apply(a, b), c)
```
BinaryOperator操作符函数的关联属性允许轻松并行化还原过程。

当然，还有一些可以与原始值一起使用的UnaryOperator和BinaryOperator的特化，即DoubleUnaryOperator，IntUnaryOperator，LongUnaryOperator，DoubleBinaryOperator，IntBinaryOperator和LongBinaryOperator。

## 11、传统函数接口
并非所有函数接口都出现在Java 8中。以前版本的Java中的许多接口符合FunctionalInterface的约束，可以用作lambdas。 一个突出的例子是在并发API中使用的Runnable和Callable接口。 在Java 8中，这些接口也标有@FunctionalInterface注释。 这使我们可以大大简化并发代码：
```java
Thread thread = new Thread(() -> System.out.println("Hello From Another Thread");
thread.start();
```

## 12、总结
在本文中，我们描述了Java 8 API中存在的可用作lambda表达式的不同函数接口。 该文章的源代码可在[GitHub上获得](https://github.com/eugenp/tutorials/tree/master/core-java-8)。

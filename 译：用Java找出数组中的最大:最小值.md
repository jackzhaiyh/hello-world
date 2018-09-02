# 用Java找出数组中的最大/最小值

原文链接：https://www.baeldung.com/java-array-min-max

作者：[Biju Kunjummen](https://dzone.com/users/1229843/biju.kunjummen.html)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1、简介
在这个简短的教程中，我们将看到如何使用Java 8的Stream API查找数组中的最大值和最小值。

我们首先在整数数组中找到最小值，然后我们将在对象数组中找到最大值。

## 2、概述
有许多方法可以在无序数组中找到最小值或最大值，它们看起来像：
```c
SET MAX to array[0]
FOR i = 1 to array length - 1
  IF array[i] > MAX THEN
    SET MAX to array[i]
  ENDIF
ENDFOR
```
我们将看看_Java 8如何隐藏这些细节_。 但是，在Java的API不适合我们的情况下，我们总是可以回到这个基本算法。

因为我们需要检查数组中的每个值，所有实现都是O（n）。

## 3、
_java.util.stream.IntStream接口提供了min方法，它可以很好地用于我们的目的_。

因为我们只使用整数，所以min不需要比较器：
```java
@Test
public void whenArrayIsOfIntegerThenMinUsesIntegerComparator() {
    int[] integers = new int[] { 20, 98, 12, 7, 35 };

    int min = Arrays.stream(integers)
      .min()
      .getAsInt();

    assertEquals(7, min);
}
```
注意我们如何使用Arrays中的流静态方法创建Integer流对象。 每种基本数组类型都有等效的流方法。

由于数组可能为空，因此min返回一个Optional，因此要将其转换为int，我们使用getAsInt。

## 4、总结
让我们创建一个简单的POJO：
```java
@Test
public void whenArrayIsOfIntegerThenMinUsesIntegerComparator() {
    int[] integers = new int[] { 20, 98, 12, 7, 35 };

    int min = Arrays.stream(integers)
      .min()
      .getAsInt();

    assertEquals(7, min);
}
```
然后我们可以再次使用Stream API来查找汽车数组中最快的汽车：
```java
@Test
public void whenArrayIsOfCustomTypeThenMaxUsesCustomComparator() {
    Car porsche = new Car("Porsche 959", 319);
    Car ferrari = new Car("Ferrari 288 GTO", 303);
    Car bugatti = new Car("Bugatti Veyron 16.4 Super Sport", 415);
    Car mcLaren = new Car("McLaren F1", 355);
    Car[] fastCars = { porsche, ferrari, bugatti, mcLaren };

    Car maxBySpeed = Arrays.stream(fastCars)
      .max(Comparator.comparing(Car::getTopSpeed))
      .orElseThrow(NoSuchElementException::new);

    assertEquals(bugatti, maxBySpeed);
}
```
在这种情况下，_Arrays的静态方法流返回接口java.util.stream.Stream <T>的实例，其中方法max需要Comparator_。

我们可以构建自己的自定义Comparator，但[Comparator.comparing更容易](https://www.baeldung.com/java-8-comparator-comparing)。

请再次注意，max返回Optional实例的原因与之前相同。

我们可以获得此值，或者我们可以使用Optionals执行其他任何操作，例如orElseThrow如果max不返回值则抛出异常。

## 5、总结
我们在这篇简短的文章中看到了使用Java 8的Stream API在数组上查找max和min是多么简单和紧凑。

有关此库的更多信息，请参阅[Oracle文档](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)。

可以在[GitHub上找到所有这些示例和代码片段的实现](https://github.com/eugenp/tutorials/tree/master/core-java-8)。

## Spring状态机指南

原文链接：https://www.baeldung.com/spring-state-machine

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1.简介

本文主要关注Spring的State Machine项目 - 该项目可用于表示工作流或任何其他类型的有限状态自动机表示问题。

## 2. Maven Dependency

首先增加Maven依赖

```java
<dependency>
    <groupId>org.springframework.statemachine</groupId>
    <artifactId>spring-statemachine-core</artifactId>
    <version>1.2.3.RELEASE</version>
</dependency>
```

最新版本在[这儿]（https://search.maven.org/）

## 3.状态机设置

首先我们定义一个简单状态机

```java
@Configuration
@EnableStateMachine
public class SimpleStateMachineConfiguration 
  extends StateMachineConfigurerAdapter<String, String> {
 
    @Override
    public void configure(StateMachineStateConfigurer<String, String> states) 
      throws Exception {
  
        states
          .withStates()
          .initial("SI")
          .end("SF")
          .states(
            new HashSet<String>(Arrays.asList("S1", "S2", "S3")));
 
    }
 
    @Override
    public void configure(
      StateMachineTransitionConfigurer<String, String> transitions) 
      throws Exception {
  
        transitions.withExternal()
          .source("SI").target("S1").event("E1").and()
          .withExternal()
          .source("S1").target("S2").event("E2").and()
          .withExternal()
          .source("S2").target("SF").event("end");
    }
}
```

请注意，此类注释为传统的Spring配置以及状态机。 它还需要扩展StateMachineConfigurerAdapter，以便可以调用各种初始化方法。 在其中一种配置方法中，我们定义状态机的所有可能状态，另一种是事件如何改变当前状态。

上面的配置设置了一个非常简单的直线过渡状态机，应该很容易遵循。

![UML状态图](https://www.baeldung.com/wp-content/uploads/2017/04/simple.png)

现在我们需要启动一个Spring context并获取对我们配置定义的状态机的引用：

```java
@Autowired
private StateMachine<String, String> stateMachine;
```
一旦得到状态机，我们需要启动它。

```java
stateMachine.start();
```
现在我们的机器处于初始状态，我们可以发送事件并因此触发转换：

```java
stateMachine.sendEvent("E1");
```
我们可以随时检查状态机当前的状态。

```java
stateMachine.getState();
```
## 4.Actions

让我们添加一些围绕状态转换执行的操作。 首先，我们在同一个配置文件中将我们的操作定义为Spring bean。

```java
@Bean
public Action<String, String> initAction() {
    return ctx -> System.out.println(ctx.getTarget().getId());
}
```
然后我们可以在配置类中的转换上注册上面创建的操作：

```java
@Override
public void configure(
  StateMachineTransitionConfigurer<String, String> transitions)
  throws Exception {
  
    transitions.withExternal()
      transitions.withExternal()
      .source("SI").target("S1")
      .event("E1").action(initAction())
```
当通过事件E1从SI转换到S1时，将执行此操作。 行动可以附在state自己身上：

```java
@Bean
public Action<String, String> executeAction() {
    return ctx -> System.out.println("Do" + ctx.getTarget().getId());
}
 
states
  .withStates()
  .state("S3", executeAction(), errorAction());
```
此状态定义函数接受在机器处于目标状态时执行的操作，并且可选地接受错误操作处理程序。

错误操作处理程序与任何其他操作没有太大区别，但如果在评估状态操作期间随时抛出异常，则会调用它：


```java
@Bean
public Action<String, String> errorAction() {
    return ctx -> System.out.println(
      "Error " + ctx.getSource().getId() + ctx.getException());
}
```

也可以为进入，执行和退出状态转换注册单个动作：

```java
@Bean
public Action<String, String> entryAction() {
    return ctx -> System.out.println(
      "Entry " + ctx.getTarget().getId());
}
 
@Bean
public Action<String, String> executeAction() {
    return ctx -> 
      System.out.println("Do " + ctx.getTarget().getId());
}
 
@Bean
public Action<String, String> exitAction() {
    return ctx -> System.out.println(
      "Exit " + ctx.getSource().getId() + " -> " + ctx.getTarget().getId());
}
```

```java
states
  .withStates()
  .stateEntry("S3", entryAction())
  .stateDo("S3", executeAction())
  .stateExit("S3", exitAction());
```
将对相应的状态转换执行相应的操作。 例如，我们可能想要在进入时验证一些前置条件或在退出时触发一些报告。

## 5.全局Listeners
可以为状态机定义全局事件侦听器。 这些侦听器将在状态转换发生时调用，并可用于日志记录或安全性等操作。

首先，我们需要添加另一个配置方法 - 一个不处理状态和转换但是配置状态机本身的配置方法。

我们需要通过扩展StateMachineListenerAdapter来定义一个监听器：
```java
public class StateMachineListener extends StateMachineListenerAdapter {

    @Override
    public void stateChanged(State from, State to) {
        System.out.printf("Transitioned from %s to %s%n", from == null ?
          "none" : from.getId(), to.getId());
    }
}
```

在这里我们只覆盖stateChanged

## 6.扩展state

Spring State Machine会跟踪其状态，但为了跟踪我们的应用程序状态，无论是一些计算值，来自管理员的条目还是来自调用外部系统的响应，我们都需要使用所谓的扩展状态。

假设我们要确保帐户申请经过两级审批。 我们可以使用存储在扩展状态中的整数来跟踪批准计数：

```java
@Bean
public Action<String, String> executeAction() {
    return ctx -> {
        int approvals = (int) ctx.getExtendedState().getVariables()
          .getOrDefault("approvalCount", 0);
        approvals++;
        ctx.getExtendedState().getVariables()
          .put("approvalCount", approvals);
    };
}
```

## 7.Guards
在执行到状态的转换之前，可以使用guard来验证某些数据。 一个guard看起来非常类似于一个action：

```java
@Bean
public Guard<String, String> simpleGuard() {
    return ctx -> (int) ctx.getExtendedState()
      .getVariables()
      .getOrDefault("approvalCount", 0) > 0;
}
```

这里明显的区别是一个guard返回一个真或假，它将告知状态机是否允许发生状态转换。

支持SPeL表达式作为guard存在。 上面的例子也可以写成：
```java
.guardExpression("extendedState.variables.approvalCount > 0")
```

## 8.状态机构建者
StateMachineBuilder可用于在不使用Spring注解或创建Spring上下文的情况下创建状态机：

```java
StateMachineBuilder.Builder<String, String> builder
  = StateMachineBuilder.builder();
builder.configureStates().withStates()
  .initial("SI")
  .state("S1")
  .end("SF");

builder.configureTransitions()
  .withExternal()
  .source("SI").target("S1").event("E1")
  .and().withExternal()
  .source("S1").target("SF").event("E2");

StateMachine<String, String> machine = builder.build();
```

## 9.状态分层
可以通过将多个withStates（）与parent（）结合使用来配置分层状态：
```java
states
  .withStates()
    .initial("SI")
    .state("SI")
    .end("SF")
    .and()
  .withStates()
    .parent("SI")
    .initial("SUB1")
    .state("SUB2")
    .end("SUBEND");
```
这种设置允许状态机具有多个状态，因此对getState（）的调用将产生多个ID。 例如，启动后立即生成以下表达式：
```java
stateMachine.getState().getIds()
["SI", "SUB1"]
```
## 10.交汇点（选择）
到目前为止，我们已经创建了本质上是线性的状态转换。 这不仅无趣，而且也不能反映开发人员要求实现的现实用例。很可能条件路径需要实现，而Spring状态机的交汇点（选择）允许我们这样做。

首先，我们需要在状态定义中将状态标记为交汇点（选择）：
```java
states
  .withStates()
  .junction("SJ")
```
然后在转换中，我们定义对应于if-then-else结构的first/then/last结构：

```java
.withJunction()
  .source("SJ")
  .first("high", highGuard())
  .then("medium", mediumGuard())
  .last("low")
```
首先，然后采取第二个参数，这是一个常规guard，将被调用，以找出应该采取的路径：
```java
@Bean
public Guard<String, String> mediumGuard() {
    return ctx -> false;
}

@Bean
public Guard<String, String> highGuard() {
    return ctx -> false;
}
```
请注意，转换不会在交汇节点停止，而是立即执行定义的guard并转到其中一条指定的路径。

在上面的例子中，指示状态机转换到SJ将导致实际状态变低，因为两个guard装置都返回false。

最后要注意的是，API提供了交汇点和选择。 但是，从功能上讲，它们在各方面都是相同的。

## 11.Fork
有时，有必要将执行分成多个独立的执行路径。 这可以使用fork功能来实现。

首先，我们需要将节点指定为fork节点，并创建状态机将执行拆分的分层区域：

```java
states
  .withStates()
  .initial("SI")
  .fork("SFork")
  .and()
  .withStates()
    .parent("SFork")
    .initial("Sub1-1")
    .end("Sub1-2")
  .and()
  .withStates()
    .parent("SFork")
    .initial("Sub2-1")
    .end("Sub2-2");
```
然后定义fork转换：

```java
.withFork()
  .source("SFork")
  .target("Sub1-1")
  .target("Sub2-1");
```
## 12.Join
fork操作的补充是连接。 它允许我们设置一个状态转换，依赖于完成其他一些状态：

![UML状态图](https://www.baeldung.com/wp-content/uploads/2017/04/forkjoin.png)

与fork一样，我们需要在状态定义中指定一个join节点：
```java
states
  .withStates()
  .join("SJoin")
```
然后在转换中，我们定义需要完成哪些状态以启用我们的join状态：

```java
transitions
  .withJoin()
    .source("Sub1-2")
    .source("Sub2-2")
    .target("SJoin");
```
使用此配置，当实现Sub1-2和Sub2-2时，状态机将转换为SJoin。

## 13.枚举而不是字符串
在上面的示例中，为了清晰和简单，我们使用字符串常量来定义状态和事件。 在现实世界的生产系统中，人们可能希望使用Java的枚举来避免拼写错误并获得更多的类型安全性。

首先，我们需要在系统中定义所有可能的状态和事件：

```java
public enum ApplicationReviewStates {
    PEER_REVIEW, PRINCIPAL_REVIEW, APPROVED, REJECTED
}

public enum ApplicationReviewEvents {
    APPROVE, REJECT
}
```
我们还需要在扩展配置时将我们的枚举作为通用参数传递：

```java
public class SimpleEnumStateMachineConfiguration
  extends StateMachineConfigurerAdapter<ApplicationReviewStates, ApplicationReviewEvents>
```
一旦定义，我们可以使用我们的枚举常量而不是字符串。 例如，要定义转换：

```java
transitions.withExternal()
  .source(ApplicationReviewStates.PEER_REVIEW)
  .target(ApplicationReviewStates.PRINCIPAL_REVIEW)
  .event(ApplicationReviewEvents.APPROVE)
```
## 14.总结

本文探讨了Spring状态机的一些特性。

与往常一样，您可以在[GitHub上找到示例源代码](https://github.com/eugenp/tutorials/tree/master/spring-state-machine)。

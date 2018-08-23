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

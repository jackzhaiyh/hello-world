1.简介

本文主要关注Spring的State Machine项目 - 该项目可用于表示工作流或任何其他类型的有限状态自动机表示问题。

2. Maven Dependency

首先增加Maven依赖

'''
<dependency>
    <groupId>org.springframework.statemachine</groupId>
    <artifactId>spring-statemachine-core</artifactId>
    <version>1.2.3.RELEASE</version>
</dependency>
'''

最新版本在[这儿]（https://search.maven.org/）

3.状态机设置

首先我们定义一个简单状态机
'''java
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
'''

请注意，此类注释为传统的Spring配置以及状态机。 它还需要扩展StateMachineConfigurerAdapter，以便可以调用各种初始化方法。 在其中一种配置方法中，我们定义状态机的所有可能状态，另一种是事件如何改变当前状态。

上面的配置设置了一个非常简单的直线过渡状态机，应该很容易遵循。

![UML类图](https://www.baeldung.com/wp-content/uploads/2017/04/simple.png)

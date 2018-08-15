�룺Java�е�״̬���ģʽ

ԭ�����ӣ�https://www.baeldung.com/java-state-design-pattern

���ߣ�[baeldung](https://www.baeldung.com/author/baeldung/)

���ߣ�[jackzhaiyh](https://github.com/jackzhaiyh)


## 1������

���̳̣�����һ����Ϊ��GOF���ģʽ��״̬ģʽ��
���ȣ����ǽ���һ�´�ģʽ��Ŀ�ĺͲ�������ͼ��������⡣Ȼ��һ��״̬ģʽ��UML��ͼ��һ��ʵ����

## 2��״̬���ģʽ

״̬ģʽ����Ҫ˼���ǣ�_����object�ڲ��ı�class������¸ı��������Ϊ_��ͬʱ��ͨ��ʵ��״̬ģʽ������������������if/else��䣬�ܹ����ִ������ࡣ

����һ��������һ��Package��ȥ�ʾ֣�Package�����Լ��Ŷӣ�Ȼ������ʾֲ����ձ��ͻ�ǩ�ա����ڣ�������ʵ״̬��������Ҫ��ӡPackage�ĵ���״̬��

��򵥵ķ���������class�ĸ�����������һЩboolean��ʶ�ͼ򵥵�if/else���ʵ�֡���������ʹ�򵥳������ӻ���Ȼ�������������Ҫ�����״̬�ܶ࣬���ַ�ʽ��ʹ�߼�������ܻ��ң����׹���

����֮�⣬ÿ��״̬���߼���ᴩ��������������ʱ��Ϳ��Կ���ʹ��״̬ģʽ������״̬ģʽ�����ǿ��Խ��߼���װ��������class�У�����[��һ����ԭ��](https://en.wikipedia.org/wiki/Single_responsibility_principle)��[����/���ԭ��](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)����ʹ����������ά����

## 3��UML��ͼ

![UML��ͼ](http://springforall.ufile.ucloud.com.cn/static/img/9f2b6f6038c380d70ab2e7b80fd953881534345)

UML��ͼ�У�Context classӵ��һ�������ڳ���ִ��ʱ�ı�Ĺ���State��

Context_����Stateʵ�ֵ���Ϊ_�����仰˵�����н���Context�������ɾ����Stateִ�С�

�������ʹ���߼��Ƿ���ģ����Һ�����������state��ֻ��Ҫ����һ��Stateʵ�֡�

## 4��ʵ��

�������һ��Ӧ�á�����������Package�����Ŷӡ����ݺͽ��գ��Ӷ��������ǿ����������State��Context class��
���ȣ�����Context�����Ǹ�Package�ࡣ

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

Package�����һ������State�����ã�ע�⣺����ͨ��previousState(),nextState()��printStatus()����ΪState�����������State���Ի���ת����_State�����ǰ�����������Ի��ڴ����Context����State��ת��_��

Client��Package���������ǲ��ش���State��ת��������Clientֻ��Ҫִ����һ������ǰһ��State��

���棬���ǳ���һ��PackageState interface���������ڶ���������Ϊ��

```java
public interface PackageState {
 
    void next(Package pkg);
    void prev(Package pkg);
    void printStatus();
}
```

���о���State class��Ҫʵ�����interface��

��һ������State��OrderState:

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

�����ϴ����У�����ָ��Package����֮�����һ��State��������ȷ��������State����ʼState���������������ܹ�����State���ת�ơ�

������DeliveredState class:

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

ͬ���ܹ�����State����ϵ��Package������Stateת�Ƶ�Ͷ��State��printStatus()��������ϢҲ��֮�ı䡣

���һ��State��ReceivedState��

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

�������һ��State,����ֻ�ܻع�����һ��State��

����һ��State��Ҫ�˽���һ��State����������Ǹ�����һ���Ľ�����ϵĴ��ۡ�

## 5������

���ȣ�����һ�¶������Ϊ��ʽ���Ƿ���Ԥ��һ�����ת����

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

Ȼ�󣬿��ټ��Package�Ƿ��ܹ�����State��

```java
@Test
public void givenDeliveredPackage_whenPrevState_thenStateOrdered() {
    Package pkg = new Package();
    pkg.setState(new DeliveredState());
    pkg.previousState();
 
    assertThat(pkg.getState(), instanceOf(OrderedState.class));
}
```

�����֤������ʱPackage�ı�State����printStatus()��������Ϊ�Ѿ��ı䡣

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

���������

- Package ordered, not delivered to the office yet.
- Package delivered to post office, not received yet.
- Package was received by client.
- This package is already received by a client.
- Package was received by client.


�����Ǹı�Context��State��Context����Ϊ�����ı䵫��class��û�иı䡣

ͬʱ����Stateת����class�ı�������State���Ӷ��ı���������Ϊ��

## 6��ȱ��

״̬ģʽ��ȱ������ʵ��״̬֮���ת��ʱ�Ĵ��ۡ���ʹ��StateӲ���룬����һ��������ǲ��õ�������

���ǣ��������ǵ������Ҫ���������Ҳ���ܲ������⡣

## 7��״̬ģʽvs����ģʽ

���ߺ����ƣ�����ͬ����ͼ����������ϸ΢��˼����

���ȣ�[����ģʽ](https://www.baeldung.com/java-strategy-pattern)_������һ��ɻ������㷨_��ͨ����Щ�㷨�ɴﵽͬ����Ŀ�ģ�����ʵ�������𣬱����������Ⱦ�㷨��
״̬ģʽ��������ʵ״̬��_��Ϊ������ȫ�ı�_��

���ң�����ģʽ�У�_�����߱����˽ⲻͬ�Ĳ��Բ�����ȷ��ȥ�ı�ʹ��_����֮��״̬ģʽ��״̬�������ӣ���������״̬��һ��ת����

## 8���ܽ�

��������Ҫ_����ԭʼ��if/else���_ʱ��״̬���ģʽ�ܺ��á�����_��ȡ�߼�������class_�������ǵ�_Context�������_state class��ʵ�ֵķ��������⣬���ǿ�������State֮���ת��������һ��State���Ըı�Context��State��

һ����˵���������ģʽ������Լ򵥵�Ӧ�ó�����˵�ǳ����ã����Ƕ��ڸ��߼��ĳ������ǿ��Կ�һ��[Spring��State Machine�̳�](https://www.baeldung.com/spring-state-machine)��

������һ���������Ĵ��������[GitHub��Ŀ](https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns)���ҵ���
        
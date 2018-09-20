# Java和Zookeeper入门

原文链接：https://www.baeldung.com/java-zookeeper

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1、简介
_Apache ZooKeeper是一种分布式协调服务_，可以简化分布式应用程序的开发。 它被Apache Hadoop，HBase等项目用于不同的用例，例如领导者选举，配置管理，节点协调，服务器租用管理等。

_ZooKeeper集群中的节点将其数据存储在共享的层级命名空间中_，该命名空间类似于标准文件系统或树数据结构。

在本文中，我们将探讨如何使用Apache Zookeeper的Java API来存储，更新和删除ZooKeeper中存储的信息。

## 2、Setup
可以在此处找到最新版本的Apache ZooKeeper [Java库](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.zookeeper%22%20AND%20a%3A%22zookeeper%22)：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

## 3、ZooKeeper数据模型 - ZNode
ZooKeeper有一个分层命名空间，很像分布式文件系统，它存储协调数据，如状态信息，协调信息，位置信息等。这些信息存储在不同的节点上。

_ZooKeeper树中的每个节点都称为ZNode。_

每个ZNode都维护所有数据和ACL变更的版本号和时间戳。 此外，这允许ZooKeeper验证缓存并协调更新。

## 4、安装
### 4.1安装
最新的ZooKeeper版本可以从[这里下载](https://www.apache.org/dyn/closer.cgi/zookeeper/)。 在此之前，我们需要确保满足此处描述的[系统要求](https://zookeeper.apache.org/doc/r3.3.5/zookeeperAdmin.html#sc_systemReq)。

### 4.2独立模式
对于本文，我们将以独立模式运行ZooKeeper，因为它需要最少的配置。 请按照[此处文档](https://zookeeper.apache.org/doc/r3.3.5/zookeeperStarted.html#sc_InstallingSingleMode)中描述的步骤操作。

注意：在独立模式下，没有复制，因此如果ZooKeeper进程失败，服务将会关闭。

## 5、ZooKeeper CLI示例
我们现在将使用ZooKeeper命令行界面（CLI）与ZooKeeper进行交互：

`bin/zkCli.sh -server 127.0.0.1:2181`

以上命令在本地启动独立实例。 现在让我们看看如何在ZooKeeper中创建ZNode并存储信息：

>[zk: localhost:2181(CONNECTED) 0] create /MyFirstZNode ZNodeVal
Created /FirstZnode

我们刚刚在ZooKeeper分层命名空间的根目录下创建了一个ZNode“MyFirstZNode”，并为其编写了“ZNodeVal”。

由于我们没有传递任何标志，因此创建的ZNode将是持久的。

现在让我们发出一个'get'命令来获取数据以及与ZNode相关的元数据：

>[zk: localhost:2181(CONNECTED) 1] get /FirstZnode

>“Myfirstzookeeper-app”
cZxid = 0x7f
ctime = Sun Feb 18 16:15:47 IST 2018
mZxid = 0x7f
mtime = Sun Feb 18 16:15:47 IST 2018
pZxid = 0x7f
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 22
numChildren = 0

我们可以使用set操作更新现有ZNode的数据。

例如：

`set /MyFirstZNode ZNodeValUpdated`

这会将MyFirstZNode上的数据从ZNodeVal更新为ZNodeValUpdated。

## 6、ZooKeeper Java API示例
现在让我们看看Zookeeper Java API并创建一个节点，更新节点并检索一些数据。

### 6.1 Java包

ZooKeeper Java绑定主要由两个Java包组成：

1.org.apache.zookeeper: which defines the main class of the ZooKeeper client library along with many static definitions of the ZooKeeper event types and states
2.org.apache.zookeeper.data: that defines the characteristics associated with ZNodes, such as Access Control Lists (ACL), IDs, stats, and so on

还有一些ZooKeeper Java API用于服务器实现，例如org.apache.zookeeper.server，org.apache.zookeeper.server.quorum和org.apache.zookeeper.server.upgrade。

但是，它们超出了本文的范围。

### 6.2 连接到ZooKeeper实例

现在让我们创建ZKConnection类，它将用于连接和断开已经运行的ZooKeeper：
```java
public class ZKConnection {
    private ZooKeeper zoo;
    CountDownLatch connectionLatch = new CountDownLatch(1);

    // ...

    public ZooKeeper connect(String host)
      throws IOException,
      InterruptedException {
        zoo = new ZooKeeper(host, 2000, new Watcher() {
            public void process(WatchedEvent we) {
                if (we.getState() == KeeperState.SyncConnected) {
                    connectionLatch.countDown();
                }
            }
        });

        connectionLatch.await();
        return zoo;
    }

    public void close() throws InterruptedException {
        zoo.close();
    }
}
```
要使用ZooKeeper服务，应用程序必须首先实例化ZooKeeper类的对象，这是ZooKeeper客户端库的主要类。

在connect方法中，我们实例化一个ZooKeeper类的实例。 此外，我们已经注册了一个回调方法来处理来自ZooKeeper的WatchedEvent以进行连接接受，并相应地使用CountDownLatch的倒计时方法完成连接方法。

建立与服务器的连接后，会话ID将分配给客户端。 为了保持会话有效，客户端应定期向服务器发送心跳。

只要会话ID保持有效，客户端应用程序就可以调用ZooKeeper API。

### 6.3 客户端操作

我们现在将创建一个ZKManager接口，该接口公开不同的操作，如创建ZNode和保存一些数据，获取和更新ZNode数据：
```java
public interface ZKManager {
    public void create(String path, byte[] data)
      throws KeeperException, InterruptedException;
    public Object getZNodeData(String path, boolean watchFlag);
    public void update(String path, byte[] data)
      throws KeeperException, InterruptedException;
}
```
现在让我们看一下上面接口的实现：

```java
public class ZKManagerImpl implements ZKManager {
    private static ZooKeeper zkeeper;
    private static ZKConnection zkConnection;

    public ZKManagerImpl() {
        initialize();
    }

    private void initialize() {
        zkConnection = new ZKConnection();
        zkeeper = zkConnection.connect("localhost");
    }

    public void closeConnection() {
        zkConnection.close();
    }

    public void create(String path, byte[] data)
      throws KeeperException,
      InterruptedException {

        zkeeper.create(
          path,
          data,
          ZooDefs.Ids.OPEN_ACL_UNSAFE,
          CreateMode.PERSISTENT);
    }

    public Object getZNodeData(String path, boolean watchFlag)
      throws KeeperException,
      InterruptedException {

        byte[] b = null;
        b = zkeeper.getData(path, null, null);
        return new String(b, "UTF-8");
    }

    public void update(String path, byte[] data) throws KeeperException,
      InterruptedException {
        int version = zkeeper.exists(path, true).getVersion();
        zkeeper.setData(path, data, version);
    }
}
```
在上面的代码中，connect和disconnect调用被委托给先前创建的ZKConnection类。 我们的create方法用于从字节数组数据创建给定路径的ZNode。 仅出于演示目的，我们保持ACL完全开放。

创建后，ZNode是持久的，并且在客户端断开连接时不会被删除。

在我们的getZNodeData方法中从ZooKeeper获取ZNode数据的逻辑非常简单。 最后，使用update方法，我们检查给定路径上是否存在ZNode，并在存在时获取它。

除此之外，为了更新数据，我们首先检查ZNode是否存在并获取当前版本。 然后，我们使用ZNode的路径，数据和当前版本作为参数调用setData方法。 仅当传递的版本与最新版本匹配时，ZooKeeper才会更新数据。

## 7、总结
在开发分布式应用程序时，Apache ZooKeeper作为分布式协调服务发挥着关键作用。 特别适用于存储共享配置，选择主节点等用例。

ZooKeeper还提供了一个优雅的基于Java的API，用于客户端应用程序代码，以便与ZooKeeper ZNodes无缝通信。

和往常一样，本教程的所有资源都可以在[Github上找到](https://github.com/eugenp/tutorials/tree/master/apache-zookeeper)。

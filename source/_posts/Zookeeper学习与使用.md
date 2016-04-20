title: Zookeeper学习与使用(转载)
date: 2015-03-08 23:09:00
categories: 转载
tags: Distributed
---
## 1. ZooKeeper的学习与应用
 
### 1.1. 概述

ZooKeeper是Apache在很多云计算项目中的一个，与Hadoop密切相关，这种情况导致我一开始认为ZooKeeper的搭建需要Hadoop项目作为支持，但是最后发现完全不需要，它是可以单独运行的一个项目。

在网上看到了一个很不错的关于ZooKeeper的介绍： 顾名思义<span style="color:red">动物园管理员，他是拿来管大象(Hadoop) 、 蜜蜂(Hive) 、 小猪(Pig)  的管理员</span>， Apache Hbase和 Apache Solr 以及LinkedIn sensei  等项目中都采用到了 Zookeeper。ZooKeeper是一个分布式的，<span style="color:red">开放源码的分布式应用程序协调服务</span>，ZooKeeper是以Fast Paxos算法为基础，实现<span style="color:red">同步服务，配置维护和命名服务</span>等分布式应用。

<!--more-->
从介绍可以看出，ZooKeeper更倾向于对大型应用的协同维护管理工作。IBM则给出了IBM对ZooKeeper的认知： Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些<span style="color:red">数据管理问题</span>，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。

总之，我认为它的核心词就是一个单词，协调。
 
## 1.2. ZooKeeper的特征
 
 在Hadoop权威指南中看到了关于ZooKeeper的一些核心特征，阅读之后感觉总结的甚是精辟，在这里引用并总结。
 
### 1.2.1. 简易

ZooKeeper的最重要核心就是一个精简文件系统，提供一些简单的操作以及附加的抽象（例如排序和通知）。
 
### 1.2.2. 易表达

ZooKeeper的原型是一个丰富的集合，它们是一些已建好的块，可以用来构建大型的协作数据结构和协议，例如：分布式队列、分布式锁以及一组对等体的选举。
 
### 1.2.3. 高可用性

ZooKeeper运行在一些集群上，被设计成可用性较高的，因此应用程序可以依赖它。ZooKeeper可以帮助你的系统<span style="color:red">避免单点故障</span>，从而建立一个可靠的应用程序。
 
### 1.2.4. 松散耦合

ZooKeeper的交互支持参与者之间并不了解对方。例如：ZooKeeper可以被当做一种公共的机制，使得进程彼此不知道对方的存在也可以相互发现并且交互，对等方可能甚至不是同步的。

这一特点我感觉最能体现在集群的部署启动过程中。像Hadoop当把配置文件写好之后，然后运行启动脚本，则251，241，242中作为集群的虚拟机是同步启动的，也就是<span style="color:red">DataNode，NameNode，TaskTracker，以及JobTracker的启动并运行时在一次启动过程中启动的，就是运行一次启动脚本文件，则都启动起来</span>。但是ZooKeeper的启动过程却不是这样的。我在251，241，242部署了ZooKeeper集群，并进行启动，则启动的过程是这样的：首先ssh到251然后启动，这时候251的集群节点启动起来，但是控制台一直报错，大概的含义就是没有检测到其他两个结点。接着分别ssh到241，242，分别启动集群中的剩下的结点，当241启动起来时，回到251查看，发现报错的信息减少，意思是只差一个结点。当251，241，242三台服务器的结点全部启动起来，则三台的服务器的控制台打印出正常的信息。
 
### 1.2.5. ZooKeeper是一个库

ZooKeeper提供了一个开源的、共享的执行存储，以及通用协作的方法，分担了每个程序员写通用协议的负担。随着时间的推移，人们可以增加和改进这个库来满足自己的需求。
 
## 1.3. Zookeeper基本知识
 
在这一小结，我介绍关于ZooKeeper的一些基本理论知识，以便对ZooKeeper有一个基本感性的认识吧，由于学习的时间不长，有些的认识可能是比较片面的，之后如果有了更深层次的认识，会补充于之后的月总结中。
 
### 1.3.1. 层次化的名字空间

ZooKeeper的整个名字空间的结构是层次化的，和一般的Linux文件系统结构非常相似，一颗很大的树。这也就是ZooKeeper的数据结构情况。名字空间的层次由斜杠/来进行分割，在名称空间里面的每一个结点的名字空间唯一由这个结点的路径来确定。
![](/img/distributed _systems/zookeeper/3_1.jpg)
图3.1 ZooKeeper的层次化名字空间（原图丢失，自己配图）

每一个节点拥有自身的一些信息，包括：数据、数据长度、创建时间、修改时间等等。从这样一类既含有数据，又作为路径表标示的节点的特点中，可以看出，ZooKeeper的节点<span style="color:red">既可以被看做是一个文件，又可以被看做是一个目录，它同时具有二者的特点</span>。为了便于表达，今后我们将使用Znode来表示所讨论的ZooKeeper节点。
 
### 1.3.2. Znode

Znode维护着<span style="color:red">数据、ACL（access control list，访问控制列表）、时间戳等交换版本号等数据结构</span>，它通过对这些数据的管理来让缓存生效并且令协调更新。每当Znode中的数据更新后它所维护的版本号将增加，这非常类似于数据库中计数器时间戳的操作方式。

另外Znode还具有<span style="color:red">原子性</span>操作的特点：命名空间中，每一个Znode的数据将被原子地读写。读操作将读取与Znode相关的所有数据，写操作将替换掉所有的数据。除此之外，每一个节点都有一个访问控制列表，这个访问控制列表规定了用户操作的权限。

ZooKeeper中同样存在临时节点。这些节点与session同时存在，当session生命周期结束，这些临时节点也将被删除。临时节点在某些场合也发挥着非常重要的作用。

### 1.3.3. Watch机制

Watch机制就和单词本身的意思一样，看。看什么？具体来讲就是某一个或者一些Znode的变化。官方给出的定义：一个Watch事件是一个一次性的触发器，当被设置了Watch的数据发生了改变的时候，则服务器将这个改变发送给设置了Watch的客户端，以便通知它们。

Watch机制主要有以下三个特点：

1、一次性的触发器（one-time trigger）
    当数据改变的时候，那么一个Watch事件会产生并且被发送到客户端中。但是客户端只会收到一次这样的通知，如果以后这个数据再次发生改变的时候，之前设置Watch的客户端将不会再次收到改变的通知，因为Watch机制规定了它是一个一次性的触发器。

2、发送给客户端
    这个表明了Watch的通知事件是从服务器发送给客户端的，是异步的，这就表明不同的客户端收到的Watch的时间可能不同，但是ZooKeeper有保证：<span  style="color:red">当一个客户端在看到Watch事件之前是不会看到结点数据的变化的</span>。例如：A=3，此时在上面设置了一次Watch，如果A突然变成4了，那么客户端会先收到Watch事件的通知，然后才会看到A=4。

3、被设置Watch的数据
    这表明了一个结点可以变换的不同方式。一个Znode变化方式有两种，<span style="color:red">结点本身数据的变化以及结点孩子的变化</span>。因此Watch也可以设置为这个Znode的结点数据，当然也可以设置为Znode结点孩子。
 
### 1.3.4. ACL访问控制列表

这是另外一个和Linux操作系统非常相似的地方，ZooKeeper使用ACL来控制对旗下Znode结点们的访问。ACL（access control list，访问控制列表）的实现和Linux文件系统的</span>访问权限</span>十分类似：它通过设置权限为来表明是否允许对一个结点的相关内容的改变。

但是与传统Linux机制不太相同，一个结点的数据没有类似“拥有者，组用户，其他用户”的概念，在ZooKeeper中，ACL通过设置ID以及与其关联的权限来完成访问控制的。ACL的权限组成语法是：
(scheme:expression, perms)
前者表明设置的ID，逗号后面表示的是ID相关的权限，例如：
 (ip:172.16.16.1, READ)
指明了IP地址为如上的用户的权限为只读。
 
以下列举以下ACL所具有的权限
> * CREATE：表明你可以创建一个Znode的子结点。
> * READ：你可以得到这个结点的数据以及列举该结点的子结点情况。
> * WRITE：设置一个结点的数据。
> * DELETE：可以删除一个结点
> * ADMIN：对一个结点设置权限。
 
## 1.4. ZooKeeper的部署以及简单使用
 
要想使用ZooKeeper，首先就要把它部署在服务器上跑起来，就想Apache，Tomcat，FtpServer等服务器一样。ZooKeeper的部署方式主要有三种，单机模式、伪集群模式、集群模式。其实剩下的两种模式都是集群模式的特殊情况。
 
### 1.4.1. 基本的环境变量配置

Java大型的项目中，环境变量的配置很重要，如果没有很好的配置环境变量的话，甚至项目连启动都是难事。

```java
export ZOOKEEPER_HOME=/home/zookeeper-3.3.3
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
```
### 1.4.2. ZooKeeper的单机模式部署

ZooKeeper的单机模式通常是用来快速测试客户端应用程序的，在实际过程中不可能是单机模式。单机模式的配置也比较简单。
#### l、编写配置文件zoo.cfg

zookeeper-3.3.3/conf文件夹下面就是要编写配置文件的位置了。在文件夹下面新建一个文件zoo.cfg。ZooKeeper的运行默认是读取zoo.cfg文件里面的内容的。以下是一个最简单的配置文件的样例：
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181

在这个文件中，我们需要指定 dataDir 的值，它指向了一个目录，这个目录在开始的时候需要为空。下面是每个参数的含义：
tickTime ：基本事件单元，以毫秒为单位。这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是<span style="color:red">每个 tickTime 时间就会发送一个心跳。</span> 
dataDir ：存储内存中数据库快照的位置，顾名思义就是 <span style="color:red">Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。</span> 
<span style="color:red">clientPort ：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。</span>

使用单机模式时用户需要注意：这种配置方式下没有 ZooKeeper 副本，所以如果 ZooKeeper 服务器出现故障， ZooKeeper 服务将会停止。
#### 2、 执行运行脚本

在zookeeper-3.3.3/bin文件夹下面运行zkServer.sh即可，运行完毕之后则ZooKeeper服务变启动起来。
./zkServer.sh start
脚本默认调用zoo.cfg里面的配置，因此程序正常启动。
 
### 1.4.3. ZooKeeper的集群模式部署

ZooKeeper的集群模式下，多个Zookeeper服务器在工作前会选举出一个Leader，在接下来的工作中这个被选举出来的Leader死了，而剩下的Zookeeper服务器会知道这个Leader死掉了，在活着的Zookeeper集群中会继续选出一个Leader，选举出Leader的目的是为了<span style="color:red">可以在分布式的环境中保证数据的一致性。</span>
如图所示：
![](/img/distributed _systems/zookeeper/3_2.jpg)
图3.2 ZooKeeper集群模式图（原图丢失，自己配图）
#### 1、确认集群服务器的数量

由于ZooKeeper集群中，会有一个Leader负责管理和协调其他集群服务器，因此服务器的数量通常都是单数，例如3，5，7...等，这样2n+1的数量的服务器就可以允许最多n台服务器的失效。

#### 2、编写配置文件
配置文件需要在每台服务器中都要编写，以下是一个配置文件的样本：
# Filename zoo.cfg
tickTime=2000
dataDir=/var/zookeeper/
clientPort=2181
initLimit=5
syncLimit=2
server.1=202.115.36.251:2888:3888
server.2=202.115.36.241:2888:3888
server.3=202.115.36.242:2888:3888

initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端<span style="color:red">不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器</span>）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒。

syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
    server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
#### 3、创建myid文件

除了修改 zoo.cfg 配置文件，集群模式下还要配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面就只有一个数据就是 A 的值，Zookeeper 启动时会读取这个文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是那个 server。

#### 4、执行运行脚本

和单机模式下的运行方式基本相同，值得注意的地方就是要分别在不同服务器上执行一次，例如分别在251，241，242上运行：
./zkServer.sh start
这样才能使得整个集群启动起来。
 
### 1.4.4. ZooKeeper的集群伪分布

其实在企业中式不会存在的，另外为了测试一个客户端程序也没有必要存在，只有在物质条件比较匮乏的条件下才会存在的模式。
集群伪分布模式就是在单机下模拟集群的ZooKeeper服务，在一台机器上面有多个ZooKeeper的JVM同时运行。

#### 1、确认集群伪服务器的数量

2n+1，和之前的集群分布相同。

#### 2、编写配置文件

在/conf文件夹新建三个配置文件，zoo1.cfg，zoo2.cfg以及zoo3.cfg。配置文件分别如下编写：
zoo1.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/users/jiangnan04/tmp/zookeeper_test/d_1
clientPort=2181
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889

zoo2.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/users/jiangnan04/tmp/zookeeper_test/d_2
clientPort=2182
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
zoo3.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/users/jiangnan04/tmp/zookeeper_test/d_3
clientPort=2183
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889

由于三个服务都在同一台电脑上，因此这里要保证地址的唯一性，因此要特别注意IP地址和端口号不要互相冲突，以免影响程序的正确执行。

#### 3、创建myid文件

这个同集群模式部署，在各自的文件夹下面创建。

------

除了修改 zoo.cfg 配置文件，集群模式下还要配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面就只有一个数据就是 A 的值，Zookeeper 启动时会读取这个文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是那个 server。

#### 4、执行运行脚本

由于所有的配置文件都在/conf文件夹下面，因此要执行三次，而且要加文件名的参数，不然会默认执行zoo.cfg这个文件，如下：
./zkServer.sh start zoo1.cfg
./zkServer.sh start zoo2.cfg
./zkServer.sh start zoo3.cfg
执行完毕后，将完成ZooKeeper的集群伪分布的启动。
 
### 1.4.5. 通过ZooKeeper命令行工具访问ZooKeeper

ZooKeeper命令行工具类似于Linux的shell环境，不过功能肯定不及shell啦，但是使用它我们可以简单的对ZooKeeper进行访问，数据创建，数据修改等操作。

当启动 ZooKeeper 服务成功之后，输入下述命令，连接到 ZooKeeper 服务：
zkCli.sh –server 127.0.0.1:2181
连接成功后，系统会输出 ZooKeeper 的相关环境以及配置信息，并在屏幕输出“ Welcome to ZooKeeper ”等信息。
命令行工具的一些简单操作如下：
> * 使用 ls 命令来查看当前 ZooKeeper 中所包含的内容：
[zk: 10.94.44.42:2181(CONNECTED) 1] ls /
> * 创建一个新的 znode ，使用 create /zk myData 。这个命令创建了一个新的 znode 节点“ zk ”以及与它关联的字符串：
[zk: 10.94.44.42:2181(CONNECTED) 2] create /zk "myData"
> * 我们运行 get 命令来确认 znode 是否包含我们所创建的字符串：
[zk: 10.94.44.42:2181(CONNECTED) 3] get /zk
> * 下面我们通过 set 命令来对 zk 所关联的字符串进行设置：
[zk: 10.94.44.42:2181(CONNECTED) 4] set /zk "zsl"
> * 下面我们将刚才创建的 znode 删除：
[zk: 10.94.44.42:2181(CONNECTED) 5] delete /zk
 
### 1.4.6. 使用API来访问ZooKeeper

API访问ZooKeeper才是客户端主要的使用手段，通过在客户端编写丰富多彩的程序，来达到对ZooKeeper的利用。这里给出一个简单的例子：（深入的还没能力给出啊，例子是从网上找的很清晰明了）

```java
import java.io.IOException;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.ZooKeeper;

public class demo {
    // 会话超时时间，设置为与系统默认时间一致
    private static final int SESSION_TIMEOUT=30000;
    
    // 创建 ZooKeeper 实例
    ZooKeeper zk;
   
    // 创建 Watcher 实例
    Watcher wh=new Watcher(){
           public void process(org.apache.zookeeper.WatchedEvent event)
           {
                   System.out.println(event.toString());
           }
    };
   
    // 初始化 ZooKeeper 实例
    private void createZKInstance() throws IOException
    {             
           zk=new ZooKeeper("localhost:2181",demo.SESSION_TIMEOUT,this.wh);
          
    }
   
    private void ZKOperations() throws IOException,InterruptedException,KeeperException
    {
           System.out.println("\n1. 创建 ZooKeeper 节点 (znode ： zoo2, 数据： myData2 ，权限： OPEN_ACL_UNSAFE ，节点类型： Persistent");
           zk.create("/zoo2","myData2".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
          
           System.out.println("\n2. 查看是否创建成功： ");
           System.out.println(new String(zk.getData("/zoo2",false,null)));
                          
           System.out.println("\n3. 修改节点数据 ");
           zk.setData("/zoo2", "shenlan211314".getBytes(), -1);
          
           System.out.println("\n4. 查看是否修改成功： ");
           System.out.println(new String(zk.getData("/zoo2", false, null)));
                          
           System.out.println("\n5. 删除节点 ");
           zk.delete("/zoo2", -1);
          
           System.out.println("\n6. 查看节点是否被删除： ");
           System.out.println(" 节点状态： ["+zk.exists("/zoo2", false)+"]");
    }
   
    private void ZKClose() throws  InterruptedException
    {
           zk.close();
    }
   
    public static void main(String[] args) throws IOException,InterruptedException,KeeperException {
           demo dm=new demo();
           dm.createZKInstance( );
           dm.ZKOperations();
           dm.ZKClose();
    }
}
```

此类包含两个主要的 ZooKeeper 函数，分别为 createZKInstance （）和 ZKOperations （）。其中 createZKInstance （）函数负责对 ZooKeeper 实例 zk 进行初始化。 ZooKeeper 类有两个构造函数，我们这里使用  “ ZooKeeper （ String connectString, ， int sessionTimeout, ， Watcher watcher ）”对其进行初始化。因此，我们需要提供初始化所需的，连接字符串信息，会话超时时间，以及一个 watcher 实例。 17 行到 23 行代码，是程序所构造的一个 watcher 实例，它能够输出所发生的事件。

ZKOperations （）函数是我们所定义的对节点的一系列操作。它包括：创建 ZooKeeper 节点（ 33 行到 34 行代码）、查看节点（ 36 行到 37 行代码）、修改节点数据（ 39 行到 40 行代码）、查看修改后节点数据（ 42 行到 43 行代码）、删除节点（ 45 行到 46 行代码）、查看节点是否存在（ 48 行到 49 行代码）。另外，需要注意的是：在创建节点的时候，需要提供节点的名称、数据、权限以及节点类型。此外，使用 exists 函数时，如果节点不存在将返回一
个 null 值。
 
## 1.5. 小结
    对于ZooKeeper的认识目前处在比较浅显的状态，了解到了基本的服务的部署以及大概ZooKeeper的工作原理。很多东西都是只懂得皮毛，现在能够深深地感受到“只有结合具体的应用才能使你对一个东西有较深的了解”这句话的深刻含义了。

转载自 ： [http://blog.csdn.net/rengq126/article/details/7393227](http://blog.csdn.net/rengq126/article/details/7393227)

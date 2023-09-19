## 1 	Zookeeper 介绍

![image-20230910201804245](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230910201804245.png)

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670234250903.png)

1. Zookeeper是根据谷歌的论文《The Chubby Lock Service for Loosely Couple Distribute System》所做的开源实现
2. Zookeeper提供了**统一配置**、**统一命名**、**分布式锁**等功能

## **2 	 zookepper配置**

### 2.1 规划

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670571083548.png)

nn1 nn2 nn3

其中：一个是leader，剩余的是Follower

### 2.2 制作 zookeeper 机器批量分布脚本

**1）拷贝并修改带有ip的文件**

```shell
cp /home/hadoop/bin/ips /home/hadoop/bin/ips_zookeeper
```

修改 ips_zookeeper 文件 的主机名称只留下nn1、nn2、nn3 三台。

```shell
vim /home/hadoop/bin/ips_zookeeper 
```

```shell
nn1
nn2
nn3
```

![2023-09-17_095625](C:\Users\15202\Desktop\pic\2023-09-17_095625.png)

**2）拷贝脚本**

```shell
#复制三个zk的脚本
cp /home/hadoop/bin/scp_all.sh /home/hadoop/bin/scp_all_zookeeper.sh
cp /home/hadoop/bin/ssh_all.sh /home/hadoop/bin/ssh_all_zookeeper.sh
cp /home/hadoop/bin/ssh_root.sh /home/hadoop/bin/ssh_root_zookeeper.sh
```

**3）修改脚本**

```shell
vim /home/hadoop/bin/scp_all_zookeeper.sh 
vim /home/hadoop/bin/ssh_all_zookeeper.sh 
vim /home/hadoop/bin/ssh_root_zookeeper.sh 
```

并将三个脚本中的**ips**改为**ips_zookeeper**

**4）**分发给nn1、nn2、nn3

```shell
scp /home/hadoop/bin/scp_all_zookeeper.sh hadoop@nn2:/home/hadoop/bin/ 
scp /home/hadoop/bin/scp_all_zookeeper.sh hadoop@nn3:/home/hadoop/bin/
scp_all_zookeeper.sh /home/hadoop/bin/ssh_all_zookeeper.sh /home/hadoop/bin/
scp_all_zookeeper.sh /home/hadoop/bin/ssh_root_zookeeper.sh /home/hadoop/bin/
```



### 2.3 zookeeper安装

**1）**zookeeper安装包下载

链接：https://pan.baidu.com/s/1hRI3JMI_ObXqQCgX1e3EdA?pwd=1111 

```shell
su - 
rz  #上传zookeeper-3.4.8.tar.gz
cd /tmp
scp_all_zookeeper.sh /tmp/zookeeper-3.4.8.tar.gz /tmp
```

**2）**在nn1、nn2、nn3下把zookeeper的tar包解压到/usr/local目录下

```shell
ssh_root_zookeeper.sh tar -zxvf /tmp/zookeeper-3.4.8.tar.gz -C /usr/local/
```

**3）**修改zookeeper目录所属用户和组为Hadoop，创建软链接

```shell
ssh_root_zookeeper.sh chown -R hadoop:hadoop /usr/local/zookeeper-3.4.8
ssh_root_zookeeper.sh  ln -s /usr/local/zookeeper-3.4.8/ /usr/local/zookeeper
```

2.4 zookeeper配置

conf/zoo.cfg

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670235071096.png)

2）修改zoo.cfg 配置文件，并批量拷贝到每台机器

```shell
#备份一下
ssh_all_zookeeper.sh cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo_sample.cfg.bakcup
#修改配置文件名称
mv zoo_sample.cfg zoo.cfg
#编辑zoo.cfg
vim /usr/local/zookeeper/conf/zoo.cfg
#增加此内容
server.1=nn1:2888:3888
server.2=nn2:2888:3888
server.3=nn3:2888:3888
#其中1 2 3分别代表的节点编号 2888节点之间的通信端口 3888节点间的选举端口
dataDir=/data/zookeeper
#zookeeper运行时候的数据存储位置
scp_all_zookeeper.sh /usr/local/zookeeper/conf/zoo.cfg /usr/local/zookeeper/conf/
```

![2023-09-15_220534](C:\Users\15202\Desktop\pic\2023-09-15_220534.png)

### 2.4	 修改输出日志配置文件所在目录并分发配置

修改zookeeper/bin/zkEnv.sh 脚本，在脚本中给 ZOO_LOG_DIR 设置日志所在目录 

```shell
vim /usr/local/zookeeper/bin/zkEnv.sh
```

<img src="C:\Users\15202\Desktop\pic\2023-09-15_220833.png" alt="2023-09-15_220833" style="zoom:67%;" />



```shell
#拷贝zkEnv.sh到每台机器的zookeeper的bin目录下
scp_all_zookeeper.sh /usr/local/zookeeper/bin/zkEnv.sh /usr/local/zookeeper/bin/
```

###  2.5 在新建的/data/zookeeper目录下生成myid文件

**1)**创建zookeeper目录

nn1上

```shell
su -
mkdir -p /data/zookeeper
chown -R hadoop:hadoop /data/zookeeper
exit
echo "1" > /data/zookeeper/myid 
```

nn2上

```shell
su -
mkdir -p /data/zookeeper
chown -R hadoop:hadoop /data/zookeeper
exit
echo "2" > /data/zookeeper/myid 
```

nn3上

```shell
su -
mkdir -p /data/zookeeper
chown -R hadoop:hadoop /data/zookeeper
exit
echo "3" > /data/zookeeper/myid 
```

### 2.6	 给5个机器设置好环境变量

```shell
su - 
echo 'export ZOOKEEPER_HOME=/usr/local/zookeeper' >> /etc/profile.d/myenv.sh
echo 'export PATH=$PATH:$ZOOKEEPER_HOME/bin' >> /etc/profile.d/myenv.sh
```

添加完分发到其他两台机器上

```shell
#批量分发
scp_all_zookeeper.sh /etc/profile.d/myenv.sh /tmp
#曲线救国将数据从tmp目录提取出来放入到/etc目录下
ssh_root_zookeeper.sh mv /tmp/myenv.sh /etc/profile.d
#批量验证
ssh_root_zookeeper.sh tail /etc/profile.d/myenv.sh
#批量source
ssh_root_zookeeper.sh source /etc/profile
```

### 2.7  配置防火墙放行规则

在nn1、nn2、nn3上配置

```shell
sudo systemctl status firewalld # 查看防火墙是否开启
sudo yum install firewalld   # 如果没有防火墙那么进行安装
sudo systemctl start firewalld  #启动防火墙
sudo systemctl enable firewalld # 设置防火墙开机自启
sudo firewall-cmd --permanent --add-port=2888/tcp # 添加2888端口放行
sudo firewall-cmd --permanent --add-port=3888/tcp # 添加3888端口放行
sudo firewall-cmd --reload # 重新加载配置文件使生效
sudo firewall-cmd --list-ports # 列出放行列表，验证规则是否生效
```

### 2.8   在每个机器上启动zookeeper服务并查看启动结果

#### 2.8.1    在每个机器上启动zookeeper服务

```shell
#批量启动
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh start
#批量查看java进程用jps
ssh_all_zookeeper.sh jps
#或者用ps 查看
ps aux|grep zookeeper 
ps -ef|grep zookeeper
```

启动成功后查看状态

```shell
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh status
```

![2023-09-16_083833](C:\Users\15202\Desktop\pic\2023-09-16_083833.png)



**前台运行 vs 后台运行 前台运行：ctrl+c 可以关闭的，也就是说 shell 客户端 和 服务器连着呢。**
想让一个程序在后台运行可以使用nohup +&的方式运行

1.nohup

用途：不挂断地运行命令。

2.&用途

用途：在后台运行

首先我们生命定义一个执行脚本

```shell
vim ~/f1.sh
#输入以下内容
#! /bin/bash
name='shenjian'
echo $name
sleep 20
echo 'end'
```

```shell
#增加执行权限
chmod +x ~/f1.sh
#执行脚本命令
~/f1.sh
```

**程序前台执行处于卡死状态**

```shell
#使用nohup执行
nohup ~/f1.sh &
```

发现程序处于后台执行过程中

**默认执行输出文件内容位置nohup.out中**

```shell
#所以程序运行我们要选择文件执行日志输出位置
#linux进程中2代表是错误输出结果 1代表是正常输出结果 2>&1
nohup ~/f1.sh >>test.log 2>&1 &
#后台执行并且输出到test.log中
#其中输出日志结果可以写出到 /dev/null中，这个位置是系统黑洞，什么都会消失
#所以如果不看重日志那么可以输出到这个位置
nohup ~/f1.sh >> /dev/null 2>&1 &
```

#### 2.8.2	查看ZK输出日志和进程信息

```bash
#日志输出文件
cat /data/zookeeper/zookeeper.out
```

#### 2.8.3	zookeeper使用

```shell
#启动zk服务
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh start
#查看每个机器ZK运行的状态
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh status
#整体停止服务
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh stop 
#重启zk服务
ssh_all_zookeeper.sh /usr/local/zookeeper/bin/zkServer.sh restart
```

启动zookeeper客户端Client

```shell
#启动zkclient，并连接zookeeper集群
/usr/local/zookeeper/bin/zkCli.sh -server nn1:2181
```

## 3 zookeeper特点

- ZooKeeper本质上是一个分布式的小文件存储系统；

- Zookeeper是一个树状结构，每一个节点都称之为Znode节点，因此这棵树也称之为Znode树，根节点为/

- 每一个节点都必须存储数据，这个数据可以是对节点的描述信息或者是集群的统一配置信息

- **强调一点**：ZooKeeper 主要是用来协调服务的，而不是用来存储业务数据的，所以不要放比较大的数据在 znode 上，ZooKeeper 给出的上限是每个结点的数据大小最大是 1M。
- 每一个持久节点都可以挂载子节点，但是临时节点不能挂载子节点

- 节点类型：

  - 持久节点 ：客户端断开节点还在
    `-- create /test` 

  - 临时节点：客户端断开节点消失

    `--create -e /test` 

  - 持久顺序节点：创建带有事务编号的节点 ，永久存在
    `--create -s /test` 

  - 临时顺序节点 ：创建带有事务编号的节点，客户端断开节点消失
    `--create -s -e /test` 

- 每一个节点的路径都是唯一的，所以基于这一个特点，可以做集群的统一命名服务

- Zookeeper的树状结构是维系在内存中的，即每一个节点中的数据也是维系在内存中，这样做的目的是方便快速查找；同时Zookeeper的树状结构也会以快照的形式维系在磁盘上，在磁盘上的存储位置由dataDir和dataLogDir 属性来决定

- 在Zookeeper中，会将每一次的写操作(创建节点，删除节点 ，修改节点)看作是一个事务，会为每一个事务分配一个**全局递增**的编号，这个编号称之为事务id，简写为Zxid

| 属性           | 解释                                                         |
| :------------- | :----------------------------------------------------------- |
| cZxid          | 节点被创建的事务id                                           |
| ctime          | 节点被创建的时间                                             |
| mZxid          | 节点被修改的事务id                                           |
| mtime          | 节点被修改的时间                                             |
| pZxid          | 子节点个数变化的事务id                                       |
| cversion       | 子节点个数的变化次数                                         |
| dataVersion    | 节点数据的变化次数                                           |
| aclVersion     | 节点的权限变化次数                                           |
| ephemeralOwner | 如果是持久节点，则此项值为0 如果是临时节点，则此项值为sessionid |
| dataLength     | 节点数据的字节个数                                           |
| numChildren    | 子节点个数                                                   |

| 常见命令           | 说明                               |
| :----------------- | :--------------------------------- |
| create /video ''   | 创建一个/vedio的持久节点，数据为空 |
| get /video         | 查看/video节点的信息               |
| set /video 'hello' | 修改节点数据为‘hello’              |
| create -e /demo01  | 创建一个临时节点/demo01            |
| create -s /demo02  | 创建一个顺序节点                   |
| ls /               | 查看子节点                         |
| rmr /test          | 删除节点                           |
| ls /demo04 watch   | 监听 /demo04下子节点的变化         |
| get /demo04 watch  | 监听/demo04数据的变化              |

------

## 4 	选举机制

### 4.1 规则

1. 在选举刚开始的时候，每一个节点都会进入选举状态，并且都会推荐自己成为leader，然后将自己的选举信息发送给其他的节点
2. 节点之间进行两两比较，经过多轮比较之后，最终胜出的节点成为leader
3. 选举信息：
   - 自己所拥有的最大事务id - Zxid
   - 自己的选举id - myid
4. 比较原则
   - 先比较两个节点的最大事务id，谁大谁赢
   - 如果最大事务id一致，则比较myid，谁大谁赢
   - 经过多轮比较之后，一个节点如果胜过了一半及以上的节点，则这个节点就会成为leader - 过半性
5. 在Zookeeper集群中，一旦选举出来leader，那么新添的节点的事务id或者myid是多大，都只能成为follower
6. 在Zookeeper集群中，当leader宕机之后，会自动的重新选举出一个新的leader，所以不存在单点故障
7. 节点状态
   1. Looking - 选举状态
   2. follower - 追随者
   3. leader - 领导者
   4. observer - 观察者
8. 如果在Zookeeper集群中出现了2个及以上的leader，则这种现象称之为脑裂

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670588853415.png)

### 4.2	异常状态解决

**4.2.1	脑裂及解决方案**

**原因：**集群产生了分裂，分裂之后进行了选举
**解决方案：**
在Zookeeper集群中，会对每次选举出来的leader分配一个唯一的全局递增的编号，称之为epochid。如果Zookeeper集群中存在了多个leader，那么会自动的将epochid较低的节点切换为follower状态

**4.2.2	集体宕机**

如果一个Zookeeper集群中，存活(能够相互通信)的节点个数不足一半，则剩余的存活节点则停止服务(对外停止服务，对内停止投票)

## 5 	ZAB协议

### 5.1	ZAB协议概述

1. ZAB（Zookeeper Atomic Broadcast）协议是为分布式协调服务ZooKeeper专门设计的一种支持**崩溃恢复**的**原子广播**协议
2. ZAB协议是一种特别为ZooKeeper设计的崩溃可恢复的原子消息广播算法。这个算法是一种类2PC算法，在2PC基础上做的改进
3. 2PC算法的核心思想就是一票否决，效率不高，从而引入Paxos算法--过半性
4. Paxos算法是一个分布式一致性算法，可以让分布式环境下的数据达到一致性的状态

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670595356286.png)**ZAB协议包括两种基本的模式**，分别是：

- 消息原子广播（保证数据一致性）
- 崩溃恢复

### **5.2	消息原子广播**

1. 在ZooKeeper中，主要依赖ZAB协议来实现分布式数据一致性，基于该协议，ZooKeeper实现了一种主备模式的系统架构来保持集群中各副本之间数据的一致性，实现分布式数据一致性的这一过程称为消息广播（原子广播）
2. ZAB协议的消息广播过程使用的是原子广播协议，类似于一个二阶段提交过程。但是相较于2PC算法，不同的是ZAB协议引入了过半性思想
   ![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670306282882.png)

1. ZooKeeper使用一个单一的主进程（leader服务器）来接收并处理客户端的所有事务请求，并采用ZAB的原子广播协议，将服务器数据的状态变更以事务Proposal的形式广播到所有的副本进程（follower或observer)上去。即：所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为leader服务器，而余下的其他服务器则成为follower服务器或observer
2. 具体流程：
   -  客户端的事务请求，leader服务器会先将该事务写到本地的log文件中
   - 然后，leader服务器会为这次请求生成对应的事务Proposal并且为这个事务Proposal分配一个全局递增的唯一的事务ID，即Zxid
   - leader服务器会为每一个follower服务器都各自分配一个单独的队列，将需要广播的事务Proposal依次放入队列中，发送给每一个follower
   -  每一个follower在收到队列之后，会从队列中依次取出事务Proposal，写到本地的事务日志中。如果写成功了，则给leader返回一个ACK消息
   - 当leader服务器接收到半数的follower的ACK相应之后，就会广播一个Commit消息给所有的follower以通知其进行事务提交，同时leader自身也进行事务提交
   - leader在收到Commit消息后完成事务提交
3. 当然，在这种简化了的二阶段提交模型下，是无法处理Leader服务器崩溃退出而带来的数据不一致问题的，因此在ZAB协议中添加了另一个模式，即采用**崩溃恢复**模式来解决这个问题
4. 整个消息广播协议是基于具有FIFO特性的TCP协议来进行网络通信的，因此能够很容易地保证消息广播过程中消息接收与发送的顺序性

### **5.3	崩溃恢复**

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670599143492.png)

1. 当leader服务器出现崩溃、重启等场景，或者因为网络问题导致过半的follower不能与leader服务器保持正常通信的时候，Zookeeper集群就会进入崩溃恢复模式
2. 进入崩溃恢复模式后，只要集群中存在过半的服务器能够彼此正常通信，那么就可以选举产生一个新的leader
3. 每次新选举的leader会自动分配一个全局递增的编号，即epochid
4. 当选举产生了新的leader服务器，同时集群中已经有过半的机器与该leader服务器完成了状态同步之后，ZAB协议就会退出恢复模式。其中，所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和leader服务器的数据保持一致
5. 当集群中已经有过半的follower服务器完成了和leader服务器的状态同步，那么整个服务框架就可以进入消息广播模式了
6. 当一台同样遵守ZAB协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个Leader服务器在负责进行消息广播，那么新加入的服务器就会自觉地进入数据恢复模式：找到leader所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中

**ZAB协议是基于2PC算法和Paxos算法改进得来，解决了2PC一票否决效率低以及Paxos算法leader宕机以及follower宕机之后重启数据怎么恢复的问题。**

## 6 	观察者

### 6.1	概述

1. 观察者（observer）在zookeeper中即不参与选举也不参与投票，但是会监听选举和投票的结果，根据结果做相应的操作
2. observer可以看成一个 没有选举权和投票权的follower。只有干活的义务，没有选举的权利
3. 观察者的数量的多少不影响投票的性能。因此观察者不是Zookeeper集群整体的主要组件。 因此如果观察者产生故障或者从集群断开连接都不会影响Zookeeper服务的可用性
4. 对于当前的Zookeeper架构而言，如果添加了更多的投票成员，则会导致写入性能下降：一个写入操作要求共识协议至少是整体的一半，因此投票的成本随着投票者越多会显著增加
5. 在实际开发过程中，如果集群规模比较大，受网络波动影响的考虑，一般会把集群的90%-97%的节点设置为observer
6. 观察者不参与投票，它只监听投票的结果，但是观察者可以和追随者一样运行 ，即客户端可能链接他们并发送读取和写入请求。 观察者像追随者一样转发这些请求到领导者，而他们只是简单的等待监听投票的结果
7. 在实际使用中，观察者可以连接到比追随者更不可靠的网络。事实上，观察者可以用于从其他数据中心和Zookeeper服务通信。观察者的客户端会看到快速的读取，因为所有的读取都在本地，并且写入导致最小的网络开销，因为投票协议所需的消息数量更小

### 6.2	配置

1. 进入到Zookeeper安装目录下的conf目录：cd zookeeper-3.4.8/conf
2. 编辑zoo.cfg文件：vim zoo.cfg
3. 在zoo.cfg中添加如下属性：peerType=observer
4. 在要配置为观察者的主机后添加观察者标记。例如：

```php
server.1=nn1:2888:3888

server.2=nn2:2888:3888

server.3=nn3:2888:3888:observer #表示将该节点设置为观察者
```

1. 进入到Zookeeper的bin目录下启动该服务器端

```shell
cd ../bin

sh zkServer.sh start
```

## 7 	zookeeper特性总结

1. 数据一致性：客户端不论连接到哪个Zookeeper节点上，展示给它都是同一个视图，即查询的数据都是一样的。这是Zookeeper最重要的性能
2. 原子性：对于事务决议的更新，只能是成功或者失败两种可能，没有中间状态。 要么都更新成功，要么都不更新。即，要么整个集群中所有机器都成功应用了某一事务，要么都没有应用，一定不会出现集群中部分机器应用了改事务，另外一部分没有应用的情况
3. 可靠性：一旦Zookeeper服务端成功的应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会一直保留下来，除非有另一个事务又对其进行了改变
4. 实时性：Zookeeper保证客户端将在非常短的时间间隔范围内获得服务器的更新信息，或者服务器失效的信息，或者指定监听事件的变化信息。（前提条件是：网络状况良好）
5. 顺序性：如果在一台服务器上消息a在消息b前发布，则在所有服务器上消息a都将在消息b前被发布。客户端在发起请求时，都会跟一个递增的命令号，根据这个机制，Zookeeper会确保客户端执行的顺序性。底层指的是Zxid
6. 过半性：Zookeeper集群必须有半数以上的机器存活才能正常工作。因为只有满足过半性，才能满足选举机制选出leader。因为只有过半，在做事务决议时，事务才能更新。所以一般来说，Zookeeper集群的数量最好是奇数个
   1. 过半选举
   2. 过半存活
   3. 过半操作

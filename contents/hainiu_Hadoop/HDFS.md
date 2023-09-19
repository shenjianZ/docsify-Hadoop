## 1 HDFS原理及搭建

前提：HDFS在Hadoop中是用来对数据进行分布式存储的，提供了数据的容错能力。在没有Hadoop之前，我们的数据存储在哪？以及怎样保证数据的容错特性呢?

### 1.1 单块硬盘

 单机时代：早期互联网刚发展的时候，各种硬件资源相对缺乏，成本较高。我们将数据写入到磁盘中，一块不够再来一块，如果把数据一块磁盘一块磁盘的写，有如下问题：

 1）单块磁盘写，磁盘读写速度上不去，读写慢；

 2）数据写入单块磁盘，一旦磁盘故障导致数据丢失；

 在这种情况下，RAID技术就应运而生了。

### 1.2 RAID

 RAID （ Redundant Array of Independent Disks ）即独立磁盘冗余阵列，简称为「磁盘阵列」，其实就是用多个独立的磁盘组成在一起形成一个大的磁盘系统，从而实现比单块磁盘更好的存储性能和更高的可靠性。

根据 RAID 算法的不同，RAID 有很多种。下面主要介绍 RAID0、RAID1、RAID10。

 **1.2.1 RAID0** 

RAID0 是一种非常简单的的方式，它将多块磁盘组合在一起形成一个大容量的存储。

 假设阵列中有N块磁盘，当写数据时，会将数据分为N份，然后分别写入N块 磁盘中。因此，RAID0将提供非常优秀的读写性能

 **优点：**

 并行写入读取快，空间利用率高。

 如果你要读取/写入 2G 的数据，在普通硬盘上，要以单盘的速度读取/写入 2G 的数据。

 如果在 4 盘 RAID0 阵列中，每个盘只需读取/写入 500MB 的数据，四个盘可以并行读取/写入，因此理论的读写速度将是单块硬盘的4倍。

 **缺点：**

 只要阵列中有一块硬盘坏掉，由于这块硬盘保存着所有数据（每个文件）的某一部分，因此所有数据都将无法读取，整个阵列中的数据将宣告报废。

**1.2.2 RAID1**

 RAID1 是磁盘阵列中单位成本最高的一种方式。因为它的原理是在往磁盘写数据的时候，将同一份数据无差别的写两份到磁盘，分别写到工作磁盘和镜像磁盘，那么它的实际空间使用率只有50%了，两块磁盘当做一块用，这是一种比较昂贵的方案。

 **优点：**

 数据写入两个磁盘，通过冗余存储实现容错。

 **缺点：**

 空间利用率低，读写速度慢。

**1.2.3 RAID10**

 RAID10其实就是RAID1与RAID0的一个合体。

 先将磁盘阵列组成 RAID1， 再将多个RAID1 组成 RAID0。

 读写数据时，可以将数据分N块，并行写入，而且每块数据都有备份。

 这样既可以通过冗余存储容错，也可以提高读写的效率。

## 2 HDFS组件详解

> HDFS是Hadoop Distribute File System 的简称，也就是Hadoop的一个分布式文件系统，用来解决存储问题。

> HDFS 可以使用低成本的硬件来搭建。

> HDFS 可以对海量的数据进行分布式存储。

> HDFS 实现软件RAID10。

### 2.1 HDFS 组件介绍-namenode、datanode

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670654861986.png)

在 HDFS 体系结构中有两类结点：

 一类是 NameNode，又叫“名称结点”

 另一类是 DataNode，又叫“数据结点”。

**NameNode**：

1）用来**存**储元数据。接收客户端的读写请求 ，namenode元数据保存到了内存中和磁盘中，保存到内存中是为了快速查询数据信息，保存到磁盘中是为了数据安全。

元数据包括：

1. 文件名称
2. 文件大小
3. 文件权限
4. 文件所有者
5. 文件切了几块
6. 副本数量
7. 每一块数据存到了哪一个datanode上
8. 等

2）一般有一个active状态的namenode，有两个standby状态的namenode，其中，active状态的NameNode负责所有的客户端操作，standby状态的NameNode处于从属地位，维护着数据状态，随时准备切换。

3）一个数据块的元数据信息占用namenode内存存储空间为150字节。块越小读取的速度就越快，但是整体占用namenode的空间就越大。

**DataNode**：

 干活的。负责存储client发来的数据块block；执行数据块的读写操作。

### 2.2 namenode环境搭建

**1.规划**

- namenode所在服务器:nn1 nn2 nn3
- datanode所在服务器:s1 s2 s3

**2.启动6台服务器，并打开所有服务器的shell终端，通过批量命令切换到hadoop用户**

Hadoop安装包下载

链接：https://pan.baidu.com/s/1hRI3JMI_ObXqQCgX1e3EdA?pwd=1111 

```shell
#切换用户，批量命令
su - hadoop
cd /tmp
rz #上传安装包
```

```shell
#解压hadoop到 /usr/local目录中，批量命令
sudo tar -zxvf /tmp/hadoop-3.1.4.tar.gz -C /usr/local/
#查询解压完毕的安装包
ssh_root.sh ls /usr/local/|grep hadoop-3.1.4
```

```shell
#修改安装包的用户权限归属为hadoop用户，因为以后我们都使用hadoop用户安装和使用
ssh_root.sh chown  -R hadoop:hadoop  /usr/local/hadoop-3.1.4/
#创建软连接，使用起来更方便 连接文件不用修改权限
ssh_root.sh ln -s /usr/local/hadoop-3.1.4/ /usr/local/hadoop
```

**3打开hadoop安装目录**

![2023-09-16_114847](C:\Users\15202\Desktop\pic\2023-09-16_114847.png)

（1）bin目录：存放对Hadoop相关服务（hdfs，yarn，mapred）进行操作的脚本

（2）etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件

（3）lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）

（4）sbin目录：存放启动或停止Hadoop相关服务的脚本

（5）share目录：存放Hadoop的依赖jar包、文档、和官方案例

```shell
# 配置环境变量
echo 'export HADOOP_HOME=/usr/local/hadoop' >> /etc/profile.d/myenv.sh
echo 'export PATH=$PATH:$HADOOP_HOME/bin' >> /etc/profile.d/myenv.sh
echo 'export PATH=$PATH:$HADOOP_HOME/sbin' >> /etc/profile.d/myenv.sh
#分发所有机器上
scp_all.sh /etc/profile.d/myenv.sh /tmp
ssh_root.sh  mv /tmp/myenv.sh /etc/profile.d
#让环境变量生效
ssh_all.sh source /etc/profile
#检查Hadoop环境变量是否配置成功
ssh_all.sh cat /etc/profile.d/myenv.sh
```

**4	修改配置文件启动namenode和datanode**

​		1.进入hadoop的配置文件目录

```shell
cd /usr/local/hadoop/etc/hadoop
```

![](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911093627832.png)

​		2.增添下列内容

**core-site.xml**
在<configuration> </configuration>标签内添加

```shell
 vim /usr/local/hadoop/etc/hadoop/core-site.xml 
```

```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://ns1</value>
    <description>默认文件服务的协议和NS逻辑名称，和hdfs-site.xml里的对应此配置替代了1.0里的fs.default.name</description>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/data/tmp</value>
    <description>数据存储目录</description>
</property>
<property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>hadoop</value>
        <description>配置root(超级用户)允许通过代理用户所属组</description>
</property>
<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>localhost</value>
    <description>配置root(超级用户)允许通过代理访问的主机节点</description>
</property> 
```

**hdfs-site.xml**

在<configuration> </configuration>标签内添加

```shell
 vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml 
```

```xml
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/data/namenode</value>
    <description>namenode本地文件存放地址</description>
</property>
<property>
    <name>dfs.nameservices</name>
    <value>ns1</value>
    <description>提供服务的NS逻辑名称，与core-site.xml里的对应</description>
</property>
<!--主要的-->
<property>
    <name>dfs.ha.namenodes.ns1</name>
    <value>nn1,nn2,nn3</value>
    <description>列出该逻辑名称下的NameNode逻辑名称</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.rpc-address.ns1.nn1</name>
    <value>nn1:9000</value>
    <description>指定NameNode的RPC位置</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.http-address.ns1.nn1</name>
    <value>nn1:50070</value>
    <description>指定NameNode的Web Server位置</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.rpc-address.ns1.nn2</name>
    <value>nn2:9000</value>
    <description>指定NameNode的RPC位置</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.http-address.ns1.nn2</name>
    <value>nn2:50070</value>
    <description>指定NameNode的Web Server位置</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.rpc-address.ns1.nn3</name>
    <value>nn3:9000</value>
    <description>指定NameNode的RPC位置</description>
</property>
<!--主要的-->
<property>
    <name>dfs.namenode.http-address.ns1.nn3</name>
    <value>nn3:50070</value>
    <description>指定NameNode的Web Server位置</description>
</property>
<property>
    <name>dfs.namenode.handler.count</name>
    <value>77</value>
    <description>namenode的工作线程数</description>
</property>
```

**hadoop-env.sh**

在第98行位置处插入

```shell
vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh 
```

```shell
# The maximum amount of heap to use, in MB. Default is 1000.
# java虚拟机使用的最大内存
source /etc/profile
export HADOOP_HEAPSIZE_MAX=2048  #注意此处内存分配视情况而定
```

![image-20230911094441783](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911094441783.png)

​	3.分发以上配偶的文件到其他机器中 scp_all.sh 

```shell
scp_all.sh  /usr/local/hadoop/etc/hadoop/core-site.xml /usr/local/hadoop/etc/hadoop/ 
scp_all.sh  /usr/local/hadoop/etc/hadoop/hdfs-site.xml /usr/local/hadoop/etc/hadoop/ 
scp_all.sh  /usr/local/hadoop/etc/hadoop/hadoop-env.sh /usr/local/hadoop/etc/hadoop/
```

​	4.将/data目录所有者和属组改为hadoop

```shell
 ssh_root.sh chown -R hadoop:hadoop  /data
```

​	5.格式化第一台namenode

```shell
#在第一台机器上面进行数据的格式化
/usr/local/hadoop/bin/hdfs namenode -format
```

![2023-09-16_121108](C:\Users\15202\Desktop\pic\2023-09-16_121108.png)

格式化成功！

6. 格式化成功之后会看到/data/namenode/current目录下有一个fsimage文件
   **fsimage文件是namenode元数据的镜像文件，相当于内存中元数据的快照。**

![](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911095643308.png)

7. 启动第一台主机的hdfs

```shell
/usr/local/hadoop/sbin/hadoop-daemon.sh start namenode
```

8. 格式化第二台和第三台namenode

```shell
#在nn2和nn3上执行
/usr/local/hadoop/bin/hdfs namenode -bootstrapStandby
```

格式化第二台namenode的时候竟然没有成功

分析原因：当我格式化第二台namenode的时候需要从第一台同步元数据，多台namenode要保证数据一致，此时需要另外一个组件journalnode

### 2.3 journalnode环境搭建

**2.3.1	步骤**

![吃饭](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670662615226.png)

1. journalnode的配置-在hdfs-site.xml中添加如下配置

   ```shell
   vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml 
   ```


```XML
    <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://nn1:8485;nn2:8485;nn3:8485/ns1</value>
    <description>指定用于HA存放edits的共享存储，通常是namenode的所在机器</description>
    </property>
    <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/data/journaldata/</value>
    <description>journaldata服务存放文件的地址</description>
    </property>
    <property>
    <name>ipc.client.connect.max.retries</name>
    <value>10</value>
    <description>namenode和journalnode的链接重试次数10次</description>
    </property>
    <property>
    <name>ipc.client.connect.retry.interval</name>
    <value>10000</value>
    <description>重试的间隔时间10s</description>
    </property>
```

```shell
#将hdfs-site.xml发送到其他节点
scp_all.sh /usr/local/hadoop/etc/hadoop/hdfs-site.xml  /usr/local/hadoop/etc/hadoop/
```

2.因为journalnode是用来进行namenode数据同步的，所以需要先启动三台journalnode再对namenode进行格式化

启动journalnode之前先将nn1上namenode停掉，并将格式化生成的/data/namenode目录删除掉

```shell
hadoop-daemon.sh stop namenode
rm -rf /data/namenode
```

```shell
#启动三个主机的journalnode
#nn1 nn2 nn3
su - hadoop
ssh_all_zookeeper.sh hadoop-daemon.sh start journalnode
```

![image-20230911181453502](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911181453502.png)

3.添加journalnode放行端口8485、NameNode IPC服务端口9000、Web Server页面访问端口50070

```shell
# 分别在nn1、nn2、nn3执行
sudo firewall-cmd --permanent --add-port=8485/tcp 
sudo firewall-cmd --permanent --add-port=9000/tcp 
sudo firewall-cmd --permanent --add-port=50070/tcp 
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

4.journalnode已经准备完毕，那么再次格式化第一台namenode并启动，

```shell
#在nn1机器上面进行namenode的格式化
hdfs namenode -format
#启动nn1上的namenode
hadoop-daemon.sh start namenode
```

5.格式化并启动第二、三台namenode（nn2、nn3），并访问页面进行测试

```shell
#格式化第二台和第三台namenode
hdfs namenode -bootstrapStandby
#启动第二台和第三台namenode
hadoop-daemon.sh start namenode
```

![image-20230911182023059](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911182023059.png)

**1）**先启动zookeeper

![image-20230911182358256](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911182358256.png)

**2）**本地浏览器访问nn2:50070

![2023-09-16_165934](C:\Users\15202\Desktop\pic\2023-09-16_165934.png)

**2.3.2	journalnode元数据的同步**

Namenode 除了内存存储元数据外，磁盘也存储，主要维护两个文件，一个是fsimage，一个是 editlog

**editlog（简称edits）：**

 操作日志文件,记录了NameNode所要执行的操作；

 **fsimage：**

 元数据镜像文件。存储某NameNode元数据信息，并不是实时同步内存中的数据。

 一般而言，fsimage中的元数据是落后于内存中的元数据的；

 当有写请求时，NameNode会首先写该操作先写到磁盘上的edits文件中，当edits文件写成功后才会修改NameNode内存的元数据，内存修改成功后向客户端返回成功信号。

请求操作记录除了会写入到activenamenode中的edits文件中，还会给journalnode中写入edits文件，然后journalnode会将本地的edits文件中的请求操作，在standbynamenode中进行同步。这样就可以进行元数据的同步了

**2.3.3	 namenode元数据合并**

- 随着edits记录的操作越来越多，内存中的元数据越来越多，这样的话内存中的元数据存在风险，一旦宕机，内存数据丢失，所以为了安全起见，需要将内存中的元数据，周期性的写入到fsimage文件中。如果数据出错可以通过 fsimage + edits 恢复。
- 因为ActiveNamenode相对来说比较忙，所以选择相对轻松的StandbyNamenode来做元数据同步这个事情

**2.3.4	合并周期是什么呢？**

- StandByNamenode检查是否达到checkpoint条件：离上一次checkpoint操作是否已经有一个小时，或者HDFS已经进行了100万次操作。

- StandByNamenode检查达到checkpoint条件后，将该元数据以fsimage.ckpt_txid格式保存到StandByNamenode的磁盘上，并且随之生成一个MD5文件。然后将该fsimage.ckpt_txid文件重命名为fsimage_txid。

- 然后StandByNamenode通过HTTP联系ActiveNameNode。

- ActiveNameNode通过StandByNamenode从StandByNamenode获取最新的fsimage_txid文件并保存为fsimage.ckpt_txid，然后也生成一个MD5，将这个MD5与StandByNamenode的MD5文件进行比较，确认ANN已经正确获取到了StandByNamenode最新的fsimage文件。然后将fsimage.ckpt_txid文件重命名为fsimage_txit。 　　

  通过上面一系列的操作，SBNN上最新的FSImage文件就成功同步到了ANN上

### 2.4 	ZKFC环境搭建

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670684800979.png)

1. 三台namenode已经全部启动成功，设置nn1是active也就是老大，另外两个namenode是standby是备份

   ```shell
   #手动切换nn1的节点是主节点 ，注意这种方式只是临时生效，下次重启失效
   hdfs haadmin -transitionToActive nn1
   #查看节点状态
   hdfs haadmin -getServiceState nn1
   ```

远程桌面浏览器访问nn1:50070

![image-20230911184341106](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911184341106.png)

1. 如果每次都是程序员自己切换的话，那也太不灵活了，所以需要引入一个组件，就是zkfc
2. zkfc本质上就是一个进程，**全称是ZKFailoverController**，需要在三台namenode上启动。它的主要任务就是一边联系同服务器的namenode，一边连接zookeeper。我们三个namenode启动之后，怎么通过zkfc选出一个active的namenode呢？那就看哪台节点的zkfc先在zookeeper中创建节点。谁创建成功，哪台服务器就是active的namenode，具体过程如下:

**主备切换过程**

- ① 启动NameNode，ZKFC，此时三个NameNode的状态都是竞选状态
- ② 三个ZKFC分别通过ActiveStandbyElector发起NameNode的选举
  通过zookeeper的写一致性以及临时节点来实现
- ③ 发起主备选举的时候，ActiveStandbyElector会尝试在zookeeper的/hadoop-ha/ns1创建一个临时节点，zookeeper的写一致性会保证只有一个节点创建成功
- ④ 创建成功的ActiveStandbyElector通过回调方式通知ZKFC，将对应的NameNode切换为Active状态；创建失败的也通过同样方式将NameNode切换为Standby状态
- ⑤ 无论是否创建成功，这些ActiveStandbyElector都会监听/hadoop-ha/ns1；
  当Active NameNode对应的HealthMonitor监控到NameNode异常时，会告知ZKFC，ZKFC通过ActiveStandbyElector删除所创建的临时节点
- ⑥ 此时处于Standby的NameNode会监控到这个消息它首先会通过判断节点是否存在来确认情况
  如果是正常关闭的，则发起主备选举，成功创建临时节点，并且将NameNode的状态切换为Active

1. - 如果是正常关闭的，则发起主备选举，成功创建临时节点，并且将NameNode的状态切换为Active

**2.1.1.	zkfc安装**

**core-site.xml**中添加一下内容

```shell
vim /usr/local/hadoop/etc/hadoop/core-site.xml 
```

```xml
<property>
    <name>ha.zookeeper.quorum</name>
    <value>nn1:2181,nn2:2181,nn3:2181</value>
    <description>HA使用的zookeeper地址</description>
</property>
```

**hdfs-site.xml**增加如下配置

```shell
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml 
```

```xml
<property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
    <description>指定HA做隔离的方法，缺省是ssh，可设为shell，稍后详述</description>
</property>

<property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
    <description>杀死命令脚本的免密配置秘钥</description>
</property>
<property>
        <name>dfs.client.failover.proxy.provider.ns1</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>

<property>
     <name>dfs.client.failover.proxy.provider.auto-ha</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property> 

<property>
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
</property>
```

```shell
#分发脚本配置到各个节点
scp_all.sh /usr/local/hadoop/etc/hadoop/core-site.xml /usr/local/hadoop/etc/hadoop/
scp_all.sh /usr/local/hadoop/etc/hadoop/hdfs-site.xml /usr/local/hadoop/etc/hadoop/
```

启动zkfc之前需要在zookeeper中初始化zkfc的节点

```shell
#在nn1节点执行
hdfs zkfc -formatZK
```

分别在nn1,nn2,nn3 机器启动zkfc

```shell
#nn1 nn2 nn3启动zkfc
ssh_all_zookeeper.sh hadoop-daemon.sh start zkfc
```

![image-20230911200703492](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911200703492.png)

重启三台namenode查看是否会选举active namenode

```shell
#三个机器分别重启namenode
ssh_all_zookeeper.sh hadoop-daemon.sh stop namenode
ssh_all_zookeeper.sh hadoop-daemon.sh start namenode
```

![2023-09-17_103428](C:\Users\15202\Desktop\pic\2023-09-17_103428.png)

当nn1的namenode宕掉，看看是否可以进行故障切换



![2023-09-17_103615](C:\Users\15202\Desktop\pic\2023-09-17_103615.png)

nn1变成active的namenode

![2023-09-17_103651](C:\Users\15202\Desktop\pic\2023-09-17_103651.png)

### 2.5	Datanode环境搭建

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670687040001.png)

1. 数据是切块进行分布式存储的，而每个block块都需要存储在datanode上。
2. namenode的作用是**存**元数据和***管理**datanode
3. 每隔3sdatanode就会向namenode汇报自身情况，如果超过10min中没有收到datanode的心跳信息，namenode就会认为此datanode丢失了，将其身上的数据拷贝到其他服务器中。
4. datanode在启动成功之后会接收namenode同步过来的clusterid，之后的通信过程中，datanode都会带着clusterid去和namenode通信，所以切记不要频繁的格式化namenode，因为每一次格式化namenode都会重新产生新的clusterid，造成namenode和datanode起不来的情况出现。（解决方法是删除文件重新格式化）

**2.5.1	安装 datanode**

1. 修改**hdfs-site.xml**

   ```shell
   vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml 
   ```

```xml
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/data/datanode</value>
    <description>datanode本地文件存放地址</description>
</property>
<property>
    <name>dfs.replication</name>
    <value>3</value>
    <description>文件复本数</description>
</property>
<property>
    <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
    <value>false</value>
</property>
<property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
</property>
<property>
    <name>dfs.datanode.use.datanode.hostname</name>
    <value>true</value>
</property>
```

2. 修改**workers**

   ```shell
   vim /usr/local/hadoop/etc/hadoop/workers 
   ```

```xml
s1
s2
s3
```

```shell
#分发配置文件到各个节点中
 scp_all.sh /usr/local/hadoop/etc/hadoop/hdfs-site.xml /usr/local/hadoop/etc/hadoop/
 scp_all.sh /usr/local/hadoop/etc/hadoop/workers /usr/local/hadoop/etc/hadoop/
 #启动各个节点的datanode,在s1、s2、s3分别执行
hadoop-daemon.sh start datanode
```

```shell
ssh_all.sh ls /data/datanode | grep datanode
```

![2023-09-17_104615](C:\Users\15202\Desktop\pic\2023-09-17_104615.png)

到此为止，hdfs所有组件全部启动成功。

以后启动可以不用一个一个启，我们可以一起全部启动

```shell
# 关闭hdfs的所有进程
stop-dfs.sh
```

![2023-09-17_104720](C:\Users\15202\Desktop\pic\2023-09-17_104720.png)

```shell
#启动hdfs的所有进程
start-dfs.sh 
```

![2023-09-17_104720](C:\Users\15202\Desktop\pic\2023-09-17_104720.png)

3.测试hdfs的可用性

```shell
# 在hdfs的根路径创建dir001目录
hadoop  fs -mkdir /001
# echo "hello hdfs" >> /tmp/a.txt
hadoop fs -put  /tmp/a.txt /001
```

在active的节点处点击Utilities，在点击Browse the file system，输入/，点击go，即可查看

![image-20230911204402967](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911204402967.png)

datanode数据存储路径：

 /data/datanode/current/BP-1348735818-11.112.227.54-1694427374120/current/finalized/subdir0/subdir0/blk_1073741825

![image-20230911204613117](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230911204613117.png)

**总结**

**NameNode**：用来**管**理datanode，并用来**存**储元数据

**journalnode**：负责两个状态的namenode进行数据同步，保持数据一致。

**ZKFC**：作用是HA自动切换。会将NameNode的active状态信息保存到zookeeper。

**DataNode**：负责存储client发来的数据块block；执行数据块的读写操作。

### 2.6	HDFS的高级配置

**core-site.xml中进行配置**

```shell
vim /usr/local/hadoop/etc/hadoop/core-site.xml 
```

```shell
#开启本地库对压缩的支持
<property>
    <name>io.native.lib.available</name>
    <value>true</value>
    <description>开启本地库支持</description>
</property>
#支持的压缩格式
<property>
    <name>io.compression.codecs</name>    <value>org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.SnappyCodec</value>
    <description>相应编码的操作类</description>
</property>
#SequenceFiles在读写中可以使用的缓存大小
<property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
    <description>SequenceFiles在读写中可以使用的缓存大小</description>
</property>
# 设置mr输入到hdfs中的数据的压缩是按照块压缩
<property>
    <name>mapreduce.output.fileoutputformat.compress.type</name>
    <value>BLOCK</value>
</property>
# 出入到hdfs中的文件是按照块为一个整体进行压缩
<property>
    <name>io.seqfile.compressioin.type</name>
    <value>BLOCK</value>
</property>
# 客户端连接超时时间
<property>
    <name>ipc.client.connection.maxidletime</name>
    <value>60000</value>
</property>
```

**hdfs-site.xml中的配置**

```shell
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml 
```

```shell
# hdfs开启支持文件追加操作
#关闭文件系统权限
#开启垃圾箱，删除的文件不会消失会移除到垃圾箱中
<property>
    <name>dfs.support.append</name>
    <value>true</value>
    <description>是否支持追加</description>
</property>
<property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
    <description>是否开启目录权限</description>
</property>
<property>
    <name>fs.trash.interval</name>
    <value>2880</value>
    <description>回收周期</description>
</property>
# datanode在读写本地文件的时候设置最大机器文件打开数
<property>
        <name>dfs.datanode.max.transfer.threads</name>
        <value>8192</value>
        <description>相当于linux下的打开文件最大数量，文档中无此参数，当出现DataXceiver报错的时候，需要调大。默认256</description>
</property>
# 在hdfs多个节点中数据均衡的时候能够用到的最大系统带宽，防止占用太多带宽
<property>
        <name>dfs.datanode.balance.bandwidthPerSec</name>
        <value>104857600</value>
</property>
# 设置每个机器的磁盘要预留两个G的数据，不能全部都给hdfs使用
# 设置存储的datanode机器的选择策略，优先以机器剩余磁盘存储两个G以上
<property>
    <name>dfs.datanode.du.reserved</name>
    <value>2147483648</value>
    <description>每个存储卷保留用作其他用途的磁盘大小</description>
</property>
<property>
    <name>dfs.datanode.fsdataset.volume.choosing.policy</name>
    <value>org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy</value>
    <description>存储卷选择策略</description>
</property>

<property>
    <name>dfs.datanode.available-space-volume-choosing-policy.balanced-space-threshold</name>
    <value>2147483648</value>
    <description>允许的卷剩余空间差值，2G</description>
</property>
# 设置客户端读取数据
# 如果读取数据的客户端和datanode在同一个机器上那么可以直接从本地读取数据，不需要走远程IO
<property>
    <name>dfs.client.read.shortcircuit</name>
    <value>true</value>
</property>
<property>
    <name>dfs.domain.socket.path</name>
    <value>/data/dn_socket_PORT</value>
</property>
```

复制以上内容到core-site.xml和hdfs-site.xml中，将这个文件分发到不同的机器中

```shell
scp_all.sh /usr/local/hadoop/etc/hadoop/hdfs-site.xml /usr/local/hadoop/etc/hadoop/
scp_all.sh /usr/local/hadoop/etc/hadoop/core-site.xml /usr/local/hadoop/etc/hadoop/
#重启集群
stop-dfs.sh
start-dfs.sh
```

## 3	HDFS命令

### 3.1 配置操作机

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670752399192.png)

hadoop集群在正常公司集群部署的时候我们是不能直接使用的，这也是为了集群安全来考虑，所以需要一个操作机去代理我们将请求发送到hadoop集群。

这就需要让操作机能够识别我们的操作命令。需要给操作机配置相应的环境

```shell
#进入op机器，并且创建hadoop用户
useradd hadoop
#设置hadoop密码
passwd hadoop
#使用root用户解压hadoop到/usr/local下面
tar -zxvf /public/software/bigdata/hadoop-3.1.4.tar.gz -C /usr/local
```

```shell
#删除原来的软连接
#使用root用户创建软连接
rm -rf /usr/local/hadoop
ln -s /usr/local/hadoop-3.1.4/ /usr/local/hadoop
```

```shell
#删除hadoop的配置文件
rm -rf /usr/local/hadoop/etc/hadoop/*
# 将nn1配置好的配置文件放入到op机中
scp -r root@11.237.80.49:/usr/local/hadoop/etc/hadoop/ /usr/local/hadoop/etc/hadoop/
```

```shell
# 修改hadoop文件夹的权限给hadoop用户使用
chown hadoop:hadoop -R /usr/local/hadoop-3.1.4/
```

```shell
# 切换用户，配置环境变量
su - hadoop
vim ~/.bash_profile 
#增加如下配置
export JAVA_HOME=/usr/local/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
# source配置文件生效
source ~/.bash_profile
```

### 3.2	hdfs常用命令

**1)ls命令**

```php
# 标准写法
hadoop fs -ls hdfs://ns1/
# 简写（推荐）
hadoop fs -ls /
# -h：文件大小显示为最大单位，更加人性化
hadoop fs -ls -h /
# -R：递归显示
hadoop fs -ls -R /  
```

**2） 上传文件/目录 put**

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670755970017.png)

```shell
#创建本地文件
echo hello yyds ttt >> /tmp/b.txt
#创建001目录
hadoop fs -mkdir /001
#标准写法：put 左面：是本地，右面是hdfs集群
hadoop fs -put /tmp/b.txt hdfs://ns1/001/
#简写：右面默认找hdfs （推荐）
hadoop fs -put -f /tmp/b.txt /001
#当上传时要对文件进行重名
hadoop fs -put  /tmp/b.txt /b_alias.txt
#在本地创建多个文件
echo 'hello' >> /tmp/c.txt
echo 'world' >> /tmp/d.txt
#一次上传多个文件到HDFS路径
hadoop fs -put -f /tmp/a.txt /tmp/b.txt  /001  
#上传目录
mkdir /tmp/test01
touch /tmp/test01/aa.txt /tmp/test01/bb.txt
hadoop fs -put /tmp/test01 /    
# -f覆盖上传
hadoop fs -put -f /tmp/test01 /
```

![2023-09-17_132432](C:\Users\15202\Desktop\pic\2023-09-17_132432.png)

**3） 读取文件 cat**

HDFS的读流程

![file](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670761068907.png)

```shell
#如果文件太大，不要用cat读取文件
hadoop fs -cat /001/a.txt /001/b.txt 
```

**4) 下载文件/目录 get**

```shell
#下载hdfs文件到本地目录
hadoop fs -get /001/a.txt /tmp
#下载hdfs文件到本地目录并重命名
hadoop fs -get /001/a.txt /tmp/rename.txt
ls -l /tmp
```

**5) 拷贝文件/目录 cp**

```shell
# 创建新的文件
touch /tmp/word.txt
# 上传本地文件使用file:开头
hadoop fs -cp file:/tmp/word.txt /test01
# 查看上传
hadoop fs -ls /test01/word.txt
# 从hdfs进行拷贝
hadoop fs -cp /test01/word.txt /test01/word1.txt
# 查询
hadoop fs -ls /test01
```

![2023-09-17_132752](C:\Users\15202\Desktop\pic\2023-09-17_132752.png)

**6） 剪切文件 mv**

```shell
#剪切
hadoop fs -mv /test01/word.txt  /001
#查询
hadoop fs -ls /001/word.txt
```

**7）删除文件/目录 rm**

执行-rm 命令后，默认是把文件移动到 user/hadoop/.Trash/Current 下，会根据配置文件配置的清理周期定期清理。

```shell
#删除文件
hadoop fs -rm /001/word.txt  # 这将会使得word.txt放到回收站中，即hdfs://ns1/user/hadoop/.Trash/Current/001/word.txt
#报错 rm只能删除文件
hadoop fs -rm /test01
#强制删除，并且递归删除文件夹中的内容
hadoop fs -rmr /test01
#删除之后不放到回收站  
hadoop fs -rm -skipTrash /001/a.txt
#查看hdfs所有文件
hadoop fs -ls -R /
#清空回收站
hadoop fs -rmr /user/hadoop/.Trash/Current/*
#再次查看hdfs所有文件
hadoop fs -ls -R /
```

![2023-09-17_133156](C:\Users\15202\Desktop\pic\2023-09-17_133156.png)

**8）创建空文件 touchz**

```shell
hadoop fs -touchz /001/ff.txt
```

**9) 创建目录 mkdir**

```shell
#可以同时创建多个目录
hadoop fs -mkdir /tmp1 /tmp2   
#同时创建父级目录
hadoop fs -mkdir -p /dir1/dir2/dir3
#查看hdfs所有文件
hadoop fs -ls -R /
```

**10） 读取文件尾部 tail**

```shell
#查看尾部1K字节   
hadoop fs -tail /001/b.txt
```

**11) 追加写入文件 appendToFile**

```shell
#本地创建文件 note.txt
echo 'it is my cat ' >> /tmp/note1.txt
#本地创建文件 new.txt
echo 'what is that ' >> /tmp/note2.txt
#将note.txt上传到hdfs中
hadoop fs -put /tmp/note1.txt /001
#将note2.txt的内容追加到note1.txt中
hadoop fs -appendToFile /tmp/note2.txt /001/note1.txt
#查看上传追加结果
hadoop fs -cat /001/note1.txt
```

**12 获取逻辑空间文件/目录大小 du**

```shell
#显示HDFS根目录中各文件和文件夹大小
hadoop fs -du / 
#以最大单位显示HDFS根目录中各文件和文件夹大小
hadoop fs -du -h / 
#仅显示HDFS根目录大小。即各文件和文件夹大小之和
hadoop fs -du -s -h /
```

**13 改变文件副本数 setrep**

```shell
#-R 递归改变目录下所有文件的副本数。
#-w 等待副本数调整完毕后返回。可理解为加了这个参数就是阻塞式的了。
hadoop fs -setrep -R -w 2 /001/b.txt
```

![2023-09-17_133909](C:\Users\15202\Desktop\pic\2023-09-17_133909.png)

**14 ） 获取HDFS目录的物理空间信息 count**

```shell
hadoop fs -count / #显示HDFS根目录在物理空间的信息
```

### 3.3 hdfs高级命令

**hdfs dfsadmin**

**1） -report：**

​    查看文件系统的基本信息和统计信息。

**2）-safemode ：**

​    安全模式命令。安全模式是NameNode的一种状态，在这种状态下，NameNode不接受对元数据的更改（只读）；不复制或删除块。NameNode在启动时自动进入安全模式，当配置块的最小百分数满足最小副本数-的条件时，会自动离开安全模式。enter是进入，leave是离开 。

```bash
#进入安全模式
hadoop dfsadmin  -safemode enter
#离开安全模式
hadoop dfsadmin -safemode leave
#获取安全模式信息
hadoop dfsadmin -safemode get
```

安全模式下可以查询元数据信息，但是不能对文件做任何的修改

------

## 4 	Fsimage和Edits文件详解

**1） 首先我们看一下两个文件存放的目录/data/namenode/current**

![2023-09-17_134122](C:\Users\15202\Desktop\pic\2023-09-17_134122.png)

**2） VERSION是java属性文件**

![2023-09-17_134234](C:\Users\15202\Desktop\pic\2023-09-17_134234.png)

1. namespaceID是文件系统唯一标识符
2. clusterID是系统生成或手动指定的集群ID
3. cTime表示Namenode存储的创建时间
4. storageType表示这个文件存储的是什么进程的数据结构信息
5. blockpoolID表示每一个namenode对应的块池id，这个id包括了其对应的Namenode节点的ip地址。
6. layoutVersion表示HDFS永久性数据结构的版本信息，只要数据结构变更，版本号也要递减

**3）edits_文件**

edits文件中存放的是客户端执行的所有更新命名空间的操作。

```shell
# hdfs oev 解析操作日志文件并且输入到相应的目录中 -i 输入 -o 输出 -p 默认就是xml
hdfs oev -i edits文件 -o ~/edits.txt
#查看edits文件
vim ~/edits.txt
```

**4） seen_txid文件**

这个文件中保存了一个事务id，这个事务id并不是Namenode内存中最新的事务id。这个文件的作用在于Namenode启动时，利用这个文件判断是否有edits文件丢失，Namenode启动时会检查seen_txid并确保内存中加载的事务id至少超过seen_txid，否则Namenode将终止启动操作。

**5） fsimage_文件**

```shell
#先进入安全模式
hadoop dfsadmin  -safemode enter
#首先我们手动触发合并元数据
hadoop dfsadmin -saveNamespace
#然后将fsimage数据导出到文本中
hdfs oiv -i fsimage文件 -o ~/fs.xml -p xml
```

fs.xml里面有

- version：version描述的一些版本信息
- NameSection：描述的是命名空间的一些信息
- ErasureCodingSection：纠删码
- INodeSection：INodeSection 由一段一段inode组成，是image中内容最大部分，除了上述的几个片段，image中剩下的内容全部都是inode。

**6） fsimage_.md5文件**

md5校验文件，用于确保fsimage文件的正确性，可以作用于磁盘异常导致文件损坏的情况。


















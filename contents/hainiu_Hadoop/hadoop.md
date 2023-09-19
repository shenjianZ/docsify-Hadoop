



## 1. 大数据

### 1.1	 什么是大数据?

**大数据的主要特点（4V）**

**1）Volume：数据体量巨大**

随着移动端和互联网的兴起，数据集合的规模不断扩大，已经从 GB 级增加到 TB 级再增加到 PB 级，近年来，数据量甚至开始以 EB 和 ZB 来计数。

**2）Velocity：大数据的数据产生、处理和分析的速度在持续加快**

加速的原因是数据创建的实时性特点，以及将流数据结合到业务流程和决策过程中的需求。数据处理速度快，处理模式已经开始从批处理转向流处理。

**3）Variety：大数据的数据类型繁多**

现在的数据类型不再只是结构化数据，更多的是半结构化或者非结构化数据，如 XML、邮件、博客、即时消息、视频、照片、点击流、 日志文件等。

**4）Value：表示大数据的数据价值密度低，商业价值高**

大数据由于体量不断加大，单位数据的价值密度在不断降低，然而数据的整体价值在提高。以监控视频为例，在一小时的视频中，有用的数据可能仅仅只有一两秒，但是却会非常重要。

### 1.2 	大数据产生

**PC端应用系统**、**手机端应用系统**、**工业设备**、**数据爬取**

### 1.3 大数据的作用?

**1）**电商、零售行业，通过对用户购买行为的分析，细分市场、精准营销。

**2）**企业通过大数据技术，来分析生产的各个环节，提高生产效率。

**3）**通过数据采集技术，抓取互联网上的信息，并提取，比如：新闻采集。

### 1.4 	大数据的数据处理流程

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670495138468.png)

**第一阶段 : 数据抽取与集成**

将不同数据源、不同类型的数据统一格式，存储到分布式文件系统中。

没有大数据技术前：存储到MySQL、oracle这种关系型数据库（单机瓶颈）。

有大数据技术后：存储到如**HDFS**分布式文件系统中（可多机存储和分析）。

**第二阶段 : 数据分析**

数据分析是整个大数据处理流程的核心。

数据分析包括 普通统计分析、算法分析（数据挖掘），通过分析数据产生价值。

数据分析通常有两种：

**1）离线处理分析：**

对一段时间内海量的离线数据进行统一的处理，对应的处理框架有 MapReduce、Spark。 数据有范围，对时间性要求不高。

**2）实时处理分析：**

 对运动中的数据进行处理，即在接收数据的同时就对其进行处理，对应的处理框架有Storm、Spark Streaming、Flink。数据没有范围（随着时间的流逝数据在产生），对时间性要求高。随着处理框架的完善，也可以用SQL来对数据进行统计分析，如hiveSQL， sparkSQL。

**第三阶段 : 数据解释**

通过数据可视化，将分析的结果形象的展现在数据大屏上，使用户更易于理解和接受。

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/0.8354713393412858.png)

### 1.5 	大数据处理技术

 对大数据技术的基本概念进行简单介绍，包括服务器集群，分布式存储和分布式计算 大数据技术。

**1.5.1 	服务器集群**

 单台服务器会有性能瓶颈，比如磁盘，CPU，内存等。

服务器集群是一种提升服务器整体计算能力的解决方案。它是由互相连接在一起的服务器群组成的一个并行式或分布式系统。

尽管单台服务器的运算能力有限，但是将成百上千的服务器组成服务器集群后，整个系统就具备了强大的运算能力，可以支持大数据分析的运算负荷。

 Google、Amazon， 阿里巴巴的计算中心里的服务器集群都达到了 5000 台服务器的规模。

**1.5.2 	分布式存储**

单机版的时候是将数据放到一台机器的磁盘中，但是随着数据量的增多，一台机器的磁盘空间不够，所以需要使用服务器集群在存储超大文件，只需要将大文件拆分成多份小文件，分别存放到不同的服务器节点。

**1.5.3 分布式计算**

随着数据越来越大，单机的计算能力遇到瓶颈，处理时间太长，需要多台服务器利用更多的资源对数据进行处理，每台服务器节点计算数据的一部分， 而且是多台计算机同时计算，处理数据的速度会远高于单个计算机。

------

## 2. 了解Hadoop

### 2.1	 Hadoop简介

![image-20230910165045825](C:\Users\15202\AppData\Roaming\Typora\typora-user-images\image-20230910165045825.png)

Hadoop是一个**分布式的**、**可靠的**、**可扩展的**用于存储超大文件的存储和计算的项目

### 2.2 	hadoop的历史起源

创始人： **Doug Cutting** 和 **Mike Cafarella !**

### 2.3 	hadoop核心组件和框架

1）**Hadoop Common**：

1. common 包括Hadoop常用的工具类，由原来的Hadoopcore部分更名而来。主要包括系统配置工具Configuration、远程过程调用RPC、序列化机制和Hadoop抽象文件系统FileSystem等

2）**Hadoop Distributed FileSystem（HDFS）**:解决存储问题

3）**Hadoop MapReduce**（分布式计算框架）--> 解决计算问题
4）**Hadoop YARN**（分布式资源管理器）--> 解决资源管理

### 2.4 Hadoop 生态圈

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670502806298.png)

Hadoop 生态圈包括以下主要组件。

**1）HDFS（Hadoop分布式文件系统）**

 HDFS是一种数据分布式保存机制，数据被保存在计算机集群上。数据写入一次，读取多次。HDFS 为HBase等工具提供了基础。

**2）MapReduce（分布式计算框架）**

 MapReduce是一种分布式计算模型，用以进行大数据量的计算，是一种离线计算框架。

 这个 MapReduce 的计算过程简而言之，就是将大数据集分解为成若干个小数据集，每个(或若干个)数据集分别由集群中的一个结点(一般就是一台主机)进行处理并生成中间结果，然后将每个结点的中间结果进行合并, 形成最终结果。

**3）HBASE（分布式缓存数据库）**

 HBase是一个建立在HDFS之上，面向列的NoSQL数据库，用于快速读/写大量数据。HBase使用Zookeeper进行管理，确保所有组件都正常运行。

**4）Flink(分布式处理引擎）**

Apache Flink是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。Flink设计为在所有常见的集群环境中运行，以内存速度和任何规模执行计算。

**5）flume（分布式日志收集系统）**

 Flume是一个分布式、可靠、和高可用的海量日志聚合的系统，如日志数据从各种网站服务器上汇集起来存储到HDFS，HBase等集中存储器中。

**6）Storm（流示计算、实时计算）**

 Storm是一个免费开源、分布式、高容错的实时计算系统。Storm令持续不断的流计算变得容易，弥补了Hadoop批处理所不能满足的实时要求。Storm经常用于在实时分析、在线机器学习、持续计算、分布式远程调用和ETL等领域。

**7）Zookeeper（分布式协作服务）**

 Hadoop的许多组件依赖于Zookeeper，它运行在计算机集群上面，用于管理Hadoop操作。

作用：解决分布式环境下的数据管理问题：统一命名，状态同步，集群管理，配置同步等。

**9）Hive（数据仓库）**

 Hive定义了一种类似SQL的查询语言(HQL),将SQL转化为MapReduce任务在Hadoop上执行。通常用于离线分析。

 HQL用于运行存储在Hadoop上的查询语句，Hive让不熟悉MapReduce开发人员也能编写数据查询语句，然后这些语句被翻译为Hadoop上面的MapReduce任务。

**10）Spark(内存计算模型)**

 Spark提供了一个更快、更通用的数据处理平台。和Hadoop相比，Spark可以让你的程序在内存中运行时速度提升100倍，或者在磁盘上运行时速度提升10倍。

**11）DolphinScheduler(工作流调度器）**

 DolphinScheduler可以把多个Map/Reduce作业组合到一个逻辑工作单元中，从而完成更大型的任务。

**12）Mahout（数据挖掘算法库）**

 Mahout的主要目标是创建一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。

**13）Hadoop YARN（分布式资源管理器）**

 YARN是下一代MapReduce，即MRv2，是在第一代MapReduce基础上演变而来的，主要是为了解决原始Hadoop扩展性较差，不支持多计算框架而提出的。

其核心思想：

将MR1中JobTracker的资源管理和作业调用两个功能分开，分别由ResourceManager和ApplicationMaster进程来实现。

1）ResourceManager：负责整个集群的资源管理和调度

2）ApplicationMaster：负责应用程序相关事务，比如任务调度、任务监控和容错等

**14）Tez(DAG计算模型)**

 一个运行在YARN之上支持DAG（有向无环图）作业的计算框架。

 Tez的目的就是帮助Hadoop处理这些MapReduce处理不了的用例场景，如机器学习。

### 2.5	 Hadoop 发行版本

**Apache：** 企业实际使用并不多。最原始（基础）版本。这是学习hadoop的基础。

**cloudera -->** **CDH版（后学这个）：**

- 对hadoop的升级，打包，开发了很多框架。flume、hue、impala都是这个公司开发

- CDH是Cloudera的Hadoop发行版，完全开源，比Apache Hadoop在兼容性，安全 性，稳定性上有所增强。

- Cloudera Manager是集群的软件分发及管理监控平台，可以在几个小时内部署 好一个Hadoop集群，并对集群的节点及服务进行实时监控。Cloudera Support即 是对Hadoop的技术支持。

**Hortonworks -->** **HDP+Ambari ：**

- 2011年成立的Hortonworks是雅虎与硅谷风投公司Benchmark Capital合资组建

------

## 3.环境准备

### 3.1 VMWare的配置

**1）**点击此处的编辑

![2023-09-15_165819](C:\Users\15202\Desktop\pic\2023-09-15_165819.png)

**2）**再点击虚拟网格编辑器，

![2023-09-15_165942](C:\Users\15202\Desktop\pic\2023-09-15_165942.png)

**3）**点击更改设置

![2023-09-15_170136](C:\Users\15202\Desktop\pic\2023-09-15_170136.png)

**4）**选择VMnet8，并更改子网IP为192.168.1.0，再点击NAT设置

![2023-09-15_172007](C:\Users\15202\Desktop\pic\2023-09-15_172007.png)

**5）**更改网关IP为192.168.1.2，并确定

![2023-09-15_171124](C:\Users\15202\Desktop\pic\2023-09-15_171124.png)

**6）**点击应用设置，完成

![2023-09-15_171336](C:\Users\15202\Desktop\pic\2023-09-15_171336.png)



### 3.2  Centos-7.9的安装

http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso

**1）**虚拟机命名为hadoop-nn1,创建用户为iboom，密码为123456

**2)**克隆5台虚拟机作为集群其他节点

![2023-09-15_172754](C:\Users\15202\Desktop\pic\2023-09-15_172754.png)



![2023-09-15_173950](C:\Users\15202\Desktop\pic\2023-09-15_173950.png)



![2023-09-15_174612](C:\Users\15202\Desktop\pic\2023-09-15_174612.png)



![2023-09-15_175414](C:\Users\15202\Desktop\pic\2023-09-15_175414.png)

可以看到已经完成了

3.3 静态IP的设置

**1)**运行6台Centos虚拟机，登录后打开终端，输入命令

```shell
su # 登录到root用户，密码为123456
vim /etc/sysconfig/network-scripts/ifcfg-ens33 
```

按下 i 进入输入模式

![2023-09-15_180258](C:\Users\15202\Desktop\pic\2023-09-15_180258.png)

配置为

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="4ae2d76b-6a8d-447c-adee-4bc0332d3d02"
DEVICE="ens33"
ONBOOT="yes"
IPADDR=192.168.1.51
NETMASK=255.255.255.0
GATEWAY=192.168.1.2
DNS1=192.168.1.2
DNS2=8.8.8.8
DNS3=114.114.114.114                     
```

2)使配置生效

```shell
systemctl restart network.service #重启网络
ifconfig # 查看是否生效
```

![2023-09-15_182050](C:\Users\15202\Desktop\pic\2023-09-15_182050.png)

ping网络测试

```shell
ping www.baidu.com
```

![2023-09-15_183621](C:\Users\15202\Desktop\pic\2023-09-15_183621.png)



接下来其他四台虚拟机依次配置即可

3.3 SSH远程连接配置

**1)**配置Windows本地DNS映射

找到hosts文件，位于 C:\Windows\System32\drivers\etc

<img src="C:\Users\15202\Desktop\pic\2023-09-15_184235.png" alt="2023-09-15_184235" style="zoom:50%;" />

hosts文件右键属性->安全->在组织或用户名中找到你的当前登录用户，选中它，并点击右下方的高级添加修改、写入权限

<img src="C:\Users\15202\Desktop\pic\2023-09-15_190510.png" alt="2023-09-15_190510" style="zoom:50%;" />

接着编辑此文件，添加如下内容

```text
192.168.1.51  nn1
192.168.1.52  nn2
192.168.1.53  nn3
192.168.1.54  s1
192.168.1.55  s2
192.168.1.56  s3
```

保存退出

**2）**使用远程连接客户端ssh登录到6台虚拟机

可使用CMD、Xshell等，这里使用**XShell 7**

<img src="C:\Users\15202\Desktop\pic\2023-09-15_191420.png" alt="2023-09-15_191420" style="zoom:80%;" />



![2023-09-15_191533](C:\Users\15202\Desktop\pic\2023-09-15_191533.png)

点击左侧的用户身份验证，填写信息连接

![2023-09-15_191836](C:\Users\15202\Desktop\pic\2023-09-15_191836.png)

![2023-09-15_192250](C:\Users\15202\Desktop\pic\2023-09-15_192250.png)

![2023-09-15_192413](C:\Users\15202\Desktop\pic\2023-09-15_192413.png)

可以看到连接成功了！

**3）**其他5台虚拟机依次进行配置

------

## 4.  服务器集群搭建

### 4.1 	集群部署规划

  集群部署：完全分布式。

**1）**选择6台linux组件并配置cpu和内存

 下载 hadoop3.x 安装包 我们使用的hadoop版本是 3.1.4

**2）**节点规划

存储节点：

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670222223891.png)

计算节点：

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670222603835.png)

配置方案：

![](http://www.hainiubl.com/uploads/md_images/202301/03/22/1670223342960.png)

集群部署规划，6台服务器：

 一个主节点：nn1

 两个从节点：nn2，nn3

 三个工作节点：s1、s2、s3

### 4.2	初始化环境

先在nn1上配置

**1）安装sz rz工具，用于以后用rz sz上传下载文件**（XShell 可以已经存在此工具，不用安装）

```shell
yum install -y lrzsz
```

**2）下载 repo 文件**

下载地址： http://mirrors.aliyun.com/repo/Centos-7.repo

**3）用 rz 将下载的** **Centos-7.repo** **文件上传到Linux系统的/tmp目录下**

```shell
cd /tmp 
rz
```

![2023-09-15_193305](C:\Users\15202\Desktop\pic\2023-09-15_193305.png)

**4）备份并替换系统的repo文件**

```shell
cp Centos-7.repo /etc/yum.repos.d/ 
cd /etc/yum.repos.d/ 
mv CentOS-Base.repo CentOS-Base.repo.bak 
mv Centos-7.repo CentOS-Base.repo
```

![2023-09-15_193556](C:\Users\15202\Desktop\pic\2023-09-15_193556.png)

**5）执行yum源更新命令**

```shell
yum clean all   #用于清理Yum包管理器的缓存
yum makecache   #服务器的包信息下载到本地电脑缓存起来
yum update -y   #用于升级系统上已安装的软件包
```

配置完毕。

6）在其他5台虚拟机上依次配置

### 4.3   安装常用软件

```shell
yum install -y openssh-server vim gcc gcc-c++ glibc-headers bzip2-devel lzo-devel curl wget openssh-clients zlib-devel autoconf automake cmake libtool openssl-devel fuse-devel snappy-devel telnet unzip zip net-tools.x86_64 firewalld systemd ntp unrar bzip2 
```

### 4.4安装JDK

**1 JDK 下载地址**

**方法一：**官网下载，需要创建一个Oracle账号登录后下载

https://download.oracle.com/otn/java/jdk/8u202-b08/1961070e4c9b4e26a04e7f5a083f551e/jdk-8u202-linux-x64.rpm

**方法二：**网盘链接

链接：https://pan.baidu.com/s/1hRI3JMI_ObXqQCgX1e3EdA?pwd=1111 

**2 安装JDK**

```shell
cd /tmp
rz #上传下载的JDK8
#默认安装到了/usr目录下
rpm -ivh jdk-8u202-linux-x64.rpm  # -ivh参数安装时显示安装进度
```

 **3配置JDK 环境变量**

```shell
echo 'export JAVA_HOME=/usr/java/jdk1.8.0_202-amd64' >> /etc/profile.d/myenv.sh
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile.d/myenv.sh
#使修改生效
source /etc/profile
#查看系统变量值
echo $PATH
#检查JDK 配置情况
java -version
```

### 4.5  修改主机名

```shell
#所有服务器节点执行修改主机名
vim /etc/hostname
#修改完后用hostname可查看当前主机名
hostname
#并配置hosts文件 添加主机名和ip地址的映射
vim /etc/hosts
# 修改第一台机器的ip映射文件，然后分发都各个不同的机器上面
scp /etc/hosts root@nn2:/etc
scp /etc/hosts root@nn3:/etc
scp /etc/hosts root@s1:/etc
scp /etc/hosts root@s2:/etc
scp /etc/hosts root@s3:/etc
```

执行reboot 重启可使主机名显示生效

### 4.6  创建hadoop 用户并设置密码

```shell
#创建hadoop用户
useradd hadoop
#给hadoop用户设置密码: 12345678
echo "12345678"|passwd hadoop --stdin
```

### 4.7 	禁止非 wheel 组用户切换到root

> 通常情况下，一般用户通过执行“su -”命令、输入正确的root密码，可以登录为root用户来对系统进行管理员级别的配置。
>  但是，为了更进一步加强系统的安全性，有必要建立一个管理员的组，只允许这个组的用户来执行 “su -” 命令登录为 root 用户，而让其他组的用户即使执行 “su -” 、输入了正确的 root 密码，也无法登录为 root 用户。在UNIX和Linux下，这个组的名称通常为 “wheel” 。

**1）修改/etc/pam.d/su配置**

要求使用su命令时必须是wheel组的成员

将此文件的两行注释取消

```shell
sed -i 's/#auth\t\trequired\tpam_wheel.so/auth\t\trequired\tpam_wheel.so/g' '/etc/pam.d/su'
sed -i 's/#auth\t\tsufficient\tpam_wheel.so/auth\t\tsufficient\tpam_wheel.so/g' '/etc/pam.d/su' 
```

**2）修改/etc/login.defs文件**

设置只有wheel组可以`su` 登录到root

**3）修改/etc/login.defs文件**，只有wheel组可以su 到root

```shell
# 先备份一个
cp /etc/login.defs /etc/login.defs_back    
# 把“SU_WHEEL_ONLY yes”字符串追加到/etc/login.defs文件底部
echo "SU_WHEEL_ONLY yes" >> /etc/login.defs 
```

**4）添加hadoop用户到wheel组**

```shell
#把hadoop用户加到wheel组里
gpasswd -a hadoop wheel
#查看wheel组里是否有hadoop用户
cat /etc/group | grep wheel
```

### 4.8	给hadoop用户，配置SSH密钥，免登录到hadoop

配置SSH密钥的目的：使得多台机器间可以免密登录。实现原理：公钥加密私钥解密，称为非对称加密。
加密和解密是同一套秘钥，称为对称加密。 使用ssh-keygen在nn1 上生成private和public密钥，将生成的public密钥注册到远程机器，这样nn1就可以远程连接其他服务器了

```shell
# 6台主机均切换到hadoop用户
su - hadoop                         
#在nn1上生成ssh公私钥，三次回车                         
ssh-keygen  
#将公钥注册给其他服务器    
ssh-copy-id nn2
ssh-copy-id nn3
ssh-copy-id s1
ssh-copy-id s2
ssh-copy-id s3
```

如果想要其他服务器可以连接nn1，并且实现两两互联，需要将private秘钥发送至其他服务器，并且nn1需要给自己注册一份public秘钥

```shell
# 在nn1上执行命令
#nn1给自己注册一份公钥
ssh-copy-id nn1
#将私钥远程拷贝到其他服务器  
scp  /home/hadoop/.ssh/id_rsa hadoop@nn2:/home/hadoop/.ssh/
scp  /home/hadoop/.ssh/id_rsa hadoop@nn3:/home/hadoop/.ssh/
scp  /home/hadoop/.ssh/id_rsa hadoop@s1:/home/hadoop/.ssh/
scp  /home/hadoop/.ssh/id_rsa hadoop@s2:/home/hadoop/.ssh/
scp  /home/hadoop/.ssh/id_rsa hadoop@s3:/home/hadoop/.ssh/
```

### 4.9 给hadoop用户，配置SSH密钥，免登录到root

```shell
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@nn1
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@nn2
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@nn3
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@s1
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@s2
ssh-copy-id  -i /home/hadoop/.ssh/id_rsa.pub  root@s3
```

------

## 5. 	命令说明及使用

### 4.1	命令说明

**4.1.1	scp 命令**

scp 就是secure copy，一个在linux下用来进行远程拷贝文件的命令。

格式：

scp 文件名 登录用户名@目标机器IP或主机名:目标目录

**4.1.2  ssh命令**

ssh 登录用户名@目标ip或主机名

**4.2.3  eval命令**

> 如何动态执行拼接的SSH 命令， 通过 eval 命令 eval命令会计算(evalue)它的参数，这些参数作为表达式计算后重新组合为一个字符串，然后作为一个命令被执行。eval最常见的用法是将动态生成的命令行计算并执行

```shell
name=hadoop
cmd="echo hello $name\! "
eval $cmd
```

### 4.3 	批量脚本说明及使用

1. ips ：用于存放要操作的主机列表，用回车或空格隔开
2. scp_all.sh ：用hadoop用户拷贝当前机器的文件到其他操作机（多机分发脚本）
3. ssh_all.sh ：用hadoop 用户可登陆其他操作机执行相应操作（多机操作脚本）
4. exe.sh ： 执行su 命令，与ssh_root.sh 配套使用 ssh_root.sh ： 用hadoop 用户登录其他操作机，并su 到 root 用户，以root 用户执行相应操作，与exe.sh 配套使用（root权限多机操作脚本）

将/home/hadoop/bin加入到hadoop的环境变量中

```shell
su - 
echo 'export PATH=$PATH:/home/hadoop/bin/' >> /etc/profile
source /etc/profile
```

**1）ips ：**用于存放要操作的主机列表，用回车或空格隔开

**在hadoop的家目录下创建bin目录**

```shell
mkdir /home/hadoop/bin
```

**在bin目录中编写ips文件**

```shell
vim /home/hadoop/bin/ips
```

```shell
nn1
nn2
nn3
s1
s2
s3
```

scp分发给其他主机

```shell
scp /home/hadoop/bin/ips hadoop@nn2:/home/hadoop/bin
scp /home/hadoop/bin/ips hadoop@nn3:/home/hadoop/bin
scp /home/hadoop/bin/ips hadoop@s1:/home/hadoop/bin
scp /home/hadoop/bin/ips hadoop@s2:/home/hadoop/bin
scp /home/hadoop/bin/ips hadoop@s3:/home/hadoop/bin
```

**2）ssh_all.sh：**用hadoop 用户可登陆其他操作机执行相应操作

**在bin目录中编写ssh_all.sh**

```shell
vim /home/hadoop/bin/ssh_all.sh
```

```shell
#! /bin/bash
# 进入到当前脚本所在目录
cd `dirname $0`
# 获取当前脚本所在目录
dir_path=`pwd`
#echo $dir_path
# 读ips文件得到数组(里面是一堆主机名)
ip_arr=(`cat $dir_path/ips`)
# 遍历数组里的主机名
for ip in ${ip_arr[*]}
do
        # 拼接ssh命令: ssh hadoop@nn1.hadoop ls
        cmd_="ssh hadoop@${ip} \"$*\"  "
        echo $cmd_
        # 通过eval命令 执行 拼接的ssh 命令
        if eval ${cmd_} ; then
                echo "=========================SUCCESS============================="
        else
                echo "==========================FAIL=============================="
        fi
done
```

scp分发给其他主机

```shell
scp /home/hadoop/bin/ssh_all.sh hadoop@nn2:/home/hadoop/bin
scp /home/hadoop/bin/ssh_all.sh hadoop@nn3:/home/hadoop/bin
scp /home/hadoop/bin/ssh_all.sh hadoop@s1:/home/hadoop/bin
scp /home/hadoop/bin/ssh_all.sh hadoop@s2/home/hadoop/bin
scp /home/hadoop/bin/ssh_all.sh hadoop@s3:/home/hadoop/bin
```

**3）  scp_all.sh：** 用hadoop用户拷贝当前机器的文件到其他操作机

**在bin目录中编写scp_all.sh**

```shell
vim /home/hadoop/bin/scp_all.sh
```

```shell
#! /bin/bash
# 进入到当前脚本所在目录
cd `dirname $0`
# 获取当前脚本所在目录
dir_path=`pwd`
#echo $dir_path
# 读ips文件得到数组(里面是一堆主机名)
ip_arr=(`cat $dir_path/ips`)
# 源
source_=$1
# 目标
target=$2
# 遍历数组里的主机名
for ip in ${ip_arr[*]}
do
        # 拼接scp命令: scp 源 hadoop@nn1.hadoop:目标
        cmd_="scp -r ${source_} hadoop@${ip}:${target}"
        echo $cmd_
        # 通过eval命令 执行 拼接的scp 命令
        if eval ${cmd_} ; then
                echo "=========================SUCCESS============================="
        else
                echo "==========================FAIL=============================="
        fi
done
```

scp分发给其他主机

```shell
scp /home/hadoop/bin/scp_all.sh hadoop@nn2:/home/hadoop/bin
scp /home/hadoop/bin/scp_all.sh hadoop@nn3:/home/hadoop/bin
scp /home/hadoop/bin/scp_all.sh hadoop@s1:/home/hadoop/bin
scp /home/hadoop/bin/scp_all.sh hadoop@s2:/home/hadoop/bin
scp /home/hadoop/bin/scp_all.sh hadoop@s3:/home/hadoop/bin
```

**4）exe.sh:**执行su 命令，与ssh_root.sh 配套使用 

**在bin目录中编写exe.sh**

```shell
vim /home/hadoop/bin/exe.sh
```

```shell
#切换到root用户执行cmd命令
cmd=$*
su - << EOF
$cmd
EOF
```

使用scp_all.sh分发给其他主机，路径需要使用绝对路径，否则出错

```shell
chmod +x /home/hadoop/bin/scp_all.sh #赋予执行权限
scp_all.sh /home/hadoop/bin/exe.sh /home/hadoop/bin
```

**5）ssh_root.sh:**  su 到 root 用户，执行相应操作

**在bin目录中编写ssh_root..sh**

```shell
vim /home/hadoop/bin/ssh_root.sh
```

```shell
#! /bin/bash
# 进入到当前脚本所在目录
cd `dirname $0`
# 获取当前脚本所在目录
dir_path=`pwd`
#echo $dir_path
# 读ips文件得到数组(里面是一堆主机名)
ip_arr=(`cat $dir_path/ips`)
# 遍历数组里的主机名
for ip in ${ip_arr[*]}
do
        # 拼接ssh命令: ssh hadoop@nn1.hadoop ls
        cmd_="ssh  root@${ip} ~/bin/exe.sh \"$*\""
        echo $cmd_
        # 通过eval命令 执行 拼接的ssh 命令
        if eval ${cmd_} ; then
                echo "=========================SUCCESS============================="
        else
                echo "==========================FAIL=============================="
        fi
done
```

使用scp_all.sh分发给其他主机，路径需要使用绝对路径，否则出错

```shell
scp_all.sh /home/hadoop/bin/ssh_root.sh /home/hadoop/bin/ssh_root.sh
```

**6）**将exe.sh传到其他主机的/home/hadoop/bin目录下

```shell
scp_all.sh /home/hadoop/bin/exe.sh /home/hadoop/bin/
```

7)赋予其他脚本的可执行文件,并重新分发

```shell
chmod +x /home/hadoop/bin/ssh_all.sh
ssh_all.sh chmod +x /home/hadoop/bin/ssh_all.sh
ssh_all.sh chmod +x /home/hadoop/bin/scp_all.sh
ssh_all.sh chmod +x /home/hadoop/bin/exe.sh
ssh_all.sh chmod +x /home/hadoop/bin/ssh_root.sh
```


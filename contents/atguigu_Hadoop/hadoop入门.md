## Hadoop概述

#### 了解Hadoop

**Hadoop**是一个分布式系统基本架构，解决海量数据的存储和海量数据的计算

**Hadoop三大发行版本：**Apache、Cloudera、Hortonworks。

**Hadoop**优势：

- **高可靠性：**维护了多个副本，即使某个节点数据丢失，也会有备份的数据
- **高扩展性：**集群间分配任务，可动态管理节点
- **高效性：**并行处理任务，速度快
- **高容错性：**任务失败会重新将任务分配给节点

#### Hadoop组成

**HDFS架构**

- **NameNode(nm)：**存储元数据，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限），以及每个文件的**块列表**和**块所在的DataNode**等
- **DataNode(dn)：**在本地文件系统存储文件块数据，以及块数据的校验和
- **Secondary NameNode(2nn)：**每隔一段时间对元数据备份

**YARN架构**

- **ResourceManager(RM)：**管理分配集群资源（CPU、内存）
- **NodeManager(NM)：**管理单节点服务器的资源（CPU、内存）
- **AppliactionMaster(AM)：**管理单任务进行
- **Container：容器**，相当于服务器，封装了运行任务所需的资源
- **包含关系:RM->NM->Container->AM->MapTask、ReduceTask**

**MapReduce架构**

- 待分析数据   -->  Map阶段  --> Reduce阶段  -->  汇总服务器

**HDFS、YARN、MapReduce三者的关系**

Client  -->  ResouceManager (Container(MapTask、ReduceTesk)  --> DataNode  --> NameNode)

#### 大数据的生态技术体系

- **Sqoop：**在Hadoop、Hive与传统的数据库Mysql之间进行数据的传递
- **Flume：**高可用、高可靠的日志传输系统
- **Kafka：**一种高吞吐量的分布式发布订阅系统
- **Spark：**开源大数据内存计算框架，可基于Hadoop上存储的大数据进行计算
- **Flink：**开源大数据内存计算框架，用于实时计算
- **Oozie：**管理Hadoop作业的工作流程调度管理系统
- **Hbase：**分布式的、面向列的开源数据库，适合于非结构化数据存储的数据库
- **Hive：**基于Hadoop的一个数据仓库工具，将结构化数据映射为数据表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行，十分适合数据仓库的统计分析
- **Zooper：**是一个对于大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式同步、组服务等

------

## Hadoop运行环境的搭建

#### 模版虚拟机准备

**1**）hadoop100虚拟机配置要求如下（本文Linux**系统全部以CentOS-7.5-x86-1804**为例）

(1)虚拟机要能够联网

`[root@hadoop100 ~]# ping www.baidu.com`

`PING www.baidu.com (14.215.177.39) 56(84) bytes of data.`

`64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=1 ttl=128 time=8.60 ms`

`64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=2 ttl=128 time=7.72 ms`

(2)安装epel-release

注：Extra Packages for Enterprise Linux是为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux。相当于是一个软件仓库，大多数rpm包在官方 repository 中是找不到的）

`[root@hadoop100 ~]# yum install -y epel-release`

(3)注意：如果Linux安装的是最小系统版，还需要安装如下工具；如果安装的是Linux桌面标准版，不需要执行如下操作

 net-tool：工具包集合，包含ifconfig等命令

`[root@hadoop100 ~]# yum install -y net-tools` 

vim：编辑器

`[root@hadoop100 ~]# yum install -y vim`

**2）**关闭防火墙，关闭防火墙开机自启

`[root@hadoop100 ~]# systemctl stop firewalld`

`[root@hadoop100 ~]# systemctl disable firewalld.service`

**3**）创建atguigu用户，并修改atguigu用户的密码

`[root@hadoop100 ~]# useradd atguigu`

`[root@hadoop100 ~]# passwd atguigu`

**4**）配置atguig用户具有root权限，方便后期加sudo执行root权限的命令

`[root@hadoop100 ~]# vim /etc/sudoers`

```shell
## Allow root to run any commands anywhere

root  ALL=(ALL)   ALL

 

\## Allows people in group wheel to run all commands

%wheel ALL=(ALL)    ALL

atguigu  ALL=(ALL)   NOPASSWD:ALL
```

atguigu  ALL=(ALL)   NOPASSWD:ALL

注意：atguigu这一行不要直接放到root行下面，因为所有用户都属于wheel组，你先配置了atguigu具有免密功能，但是程序执行到%wheel行时，该功能又被覆盖回需要密码。所以atguigu要放到%wheel这行下面。

**5)**在**/op**t目录下创建文件夹，并修改所属主、所属组

（1）在/opt目录下创建module、software文件夹

`[root@hadoop100 ~]# mkdir /opt/module`

`[root@hadoop100 ~]# mkdir /opt/software`

（2）修改module、software文件夹的所有者和所属组均为atguigu用户 

`[root@hadoop100 ~]# chown atguigu:atguigu /opt/module` 

`[root@hadoop100 ~]# chown atguigu:atguigu /opt/software`

**6**）卸载虚拟机自带的JDK**

​    注意：如果你的虚拟机是最小化安装不需要执行这一步。

[root@hadoop100 ~]# rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps 

 `rpm -qa：`查询所安装的所有rpm软件包

 `grep -i`：忽略大小写

`xargs -n1:`表示每次只传递一个参数

`rpm -e –nodeps：`强制卸载软件

**7**）重启虚拟机

`[root@hadoop100 ~]# reboot`

#### 克隆虚拟机

**1**）利用模板机hadoop100，克隆三台虚拟机：**hadoop102 hadoop103 hadoop104**

​    注意：克隆时，要先关闭hadoop100

**2**）修改克隆机IP，以下以hadoop102举例说明

（1）修改克隆虚拟机的静态IP

[root@hadoop100 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33

改成

```shell
DEVICE=ens33
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
NAME="ens33"
IPADDR=192.168.10.102
PREFIX=24
GATEWAY=192.168.10.2
DNS1=192.168.10.2
```

（2）查看Linux虚拟机的虚拟网络编辑器，编辑->虚拟网络编辑器->VMnet8

修改子网IP为**192.168.100.0**，网关IP为**192.168.100.2**

（3）查看Windows系统适配器VMware Network Adapter VMnet8的IP地址

注意**子网IP、网关IP**修改一致，DNS服务器使用**192.168.100.2 、8.8.8.8**

（4）保证Linux系统ifcfg-ens33文件中IP地址、虚拟网络编辑器地址和Windows系统VM8网络IP地址相同。

**3）**修改克隆主机名，一hadoop102为例

（1）修改主机名称

`[root@hadoop100 ~]# vim /etc/hostname`

`hadoop102`

（2）配置Linux克隆机主机名称映射hosts文件，打开/etc/hosts

`[root@hadoop100 ~]# vim /etc/hosts`

添加如下内容

`192.168.10.100 hadoop100`

`192.168.10.101 hadoop101`

`192.168.10.102 hadoop102`

`192.168.10.103 hadoop103`

`192.168.10.104 hadoop104`

`192.168.10.105 hadoop105`

`192.168.10.106 hadoop106`

`192.168.10.107 hadoop107`

`192.168.10.108 hadoop108`

**5**）修改windows的主机映射文件（hosts文件）

先拷贝出来，修改保存以后，再覆盖即可

（a）进入C:\Windows\System32\drivers\etc路径

（b）拷贝hosts文件到桌面

（c）打开桌面hosts文件并添加如下内容

`192.168.10.100 hadoop100`

`192.168.10.101 hadoop101`

`192.168.10.102 hadoop102`

`192.168.10.103 hadoop103`

`192.168.10.104 hadoop104`

`192.168.10.105 hadoop105`

`192.168.10.106 hadoop106`

`192.168.10.107 hadoop107`

`192.168.10.108 hadoop108`

（d）将桌面hosts文件覆盖C:\Windows\System32\drivers\etc路径hosts文件

#### 在Hadoop安装JDK（Java8）

注意：安装JDK前，一定确保提前删除了虚拟机自带的JDK。详细步骤见问文档3.1节中卸载JDK步骤。

**1**）用XShell传输工具将JDK导入到**/op**t目录下面的**software**文件夹下面

**2**）解压JDK到**/opt/module**目录下

`[atguigu@hadoop102 software]$ tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/`

**5**）配置JDK**环境变量**

​    （1）新建/etc/profile.d/my_env.sh文件

`[atguigu@hadoop102 ~]$ sudo vim /etc/profile.d/my_env.sh`

添加如下内容

```shell
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin
```

(2）保存后退出  :wq

(3）source一下/etc/profile文件，让新的环境变量PATH生效

`[atguigu@hadoop102 ~]$ source /etc/profile`

**6**）测试JDK是否安装成功

`[atguigu@hadoop102 ~]$ java -version`

#### 在Hadoop102上安装Hadoop

Hadoop下载地址：[https://archive.apache.org/dist/hadoop/common/hadoop-3.1.3/](https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/)

**1**）用XShell文件传输工具将**hadoop-3.1.3.tar.gz**导入到**opt**目录下面的**software**文件夹下面

**2**）解压安装文件到**/opt/module**下面

`[atguigu@hadoop102 software]$ tar -zxvf /opt/software/hadoop-3.1.3.tar.gz -C /opt/module/`

**3**）查看是否解压成功

`[atguigu@hadoop102 software]$ ls /opt/module/`

`hadoop-3.1.3`

**4**）将Hadoop添加到环境变量

（1）打开/etc/profile.d/my_env.sh文件

`[atguigu@hadoop102 hadoop-3.1.3]$ sudo vim /etc/profile.d/my_env.sh`

 在my_env.sh文件末尾添加如下内容：（shift+g）

\`#HADOOP_HOME`

`export HADOOP_HOME=/opt/module/hadoop-3.1.3`

`export PATH=$PATH:$HADOOP_HOME/bin`

`export PATH=$PATH:$HADOOP_HOME/sbin`

保存并退出： **:wq**

（2）让修改后的文件生效

`[atguigu@hadoop102 hadoop-3.1.3]$ source /etc/profile`

**6**）测试是否安装成功

`[atguigu@hadoop102 hadoop-3.1.3]$ hadoop version`

`Hadoop 3.1.3`

#### Hadoop的目录结构

**1**）查看Hadoop**目录结构**

`[atguigu@hadoop102 hadoop-3.1.3]$ ll`

`总用量 52`

`drwxr-xr-x. 2 atguigu atguigu 4096 5月 22 2017 bin`

`drwxr-xr-x. 3 atguigu atguigu 4096 5月 22 2017 etc`

`drwxr-xr-x. 2 atguigu atguigu 4096 5月 22 2017 include`

`drwxr-xr-x. 3 atguigu atguigu 4096 5月 22 2017 lib`

`drwxr-xr-x. 2 atguigu atguigu 4096 5月 22 2017 libexec`

`-rw-r--r--. 1 atguigu atguigu 15429 5月 22 2017 LICENSE.txt`

`-rw-r--r--. 1 atguigu atguigu  101 5月 22 2017 NOTICE.txt`

`-rw-r--r--. 1 atguigu atguigu 1366 5月 22 2017 README.txt`

`drwxr-xr-x. 2 atguigu atguigu 4096 5月 22 2017 sbin`

`drwxr-xr-x. 4 atguigu atguigu 4096 5月 22 2017 share`

**2）**重要目录

（1）bin ：存放对Hadoop相关服务（hdfs、yarn、mapred）进行操作的脚本

（2）etc ：存放Hadoop的配置文件

（3）lib ：存放Hadoop的本地库（对数据进行压缩解压缩功能）

（4）sbin ：存放启动、停止Hadoop相关服务的脚本

（5）share：存放Hadoop的依赖jar包、文档和官方用例

------

## Hadoop运行模式

**Hadoop运行模式**包括：本地模式、伪分布式模式、完全分布式模式

- **本地模式：**单机运行，实际生产不用
- **伪分布式模式：**单机运行但具有Hadoop集群的所有功能，实际生成不用
- **完全分布式模式：**多台服务器组成分布式环境，实际生产使用

这里仅实现完全分布式模式

分析：

​    1）准备3台客户机（关闭防火墙、静态IP、主机名称）

​    2）安装JDK

​    3）配置环境变量

​    4）安装Hadoop

​    5）配置环境变量

​	6）配置集群

​	7）单点启动

​	8）配置ssh

​	9）群起并测试集群

#### **编写集群分发脚本xsync**

**1）** **scp安全拷贝**

**前提：**在hadoop102、hadoop103、hadoop104都已经创建好的**/opt/module** 、 **/opt/software**两个目录，并且已经把这两个目录修改为atguigu:atguigu

`[atguigu@hadoop102 ~]$ sudo chown atguigu:atguigu -R /opt/module`

（a）在**hadoop102**上，将hadoop102中/opt/module/jdk1.8.0_212目录拷贝到hadoop103上。

`[atguigu@hadoop102 ~]$ scp -r /opt/module/jdk1.8.0_212 atguigu@hadoop103:/opt/module`

（b）在**hadoop103**上，将hadoop102中/opt/module/hadoop-3.1.3目录拷贝到hadoop103上。

`[atguigu@hadoop103 ~]$ scp -r atguigu@hadoop102:/opt/module/hadoop-3.1.3 /opt/module/`

（c）在**hadoop103**上操作，将hadoop102中/opt/module目录下所有目录拷贝到hadoop104上。

`[atguigu@hadoop103 opt]$ scp -r atguigu@hadoop102:/opt/module/* atguigu@hadoop104:/opt/module`

**2）** **rsync远程同步工具**

sync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

（2）案例实操

​    a）删除hadoop103中/opt/module/hadoop-3.1.3/wcinput

  `[atguigu@hadoop103 hadoop-3.1.3]$ rm -rf wcinput/`

  b）同步hadoop102中的/opt/module/hadoop-3.1.3到hadoop103

   `[atguigu@hadoop102 module]$ rsync -av hadoop-3.1.3/ atguigu@hadoop103:/opt/module/hadoop-3.1.3/`

**3）** **xsync分发脚本**

（a）期望脚本在任何路径都能使用（脚本放在声明了全局环境变量的路径）

```shell
[atguigu@hadoop102 ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin:/opt/module/jdk1.8.0_212/bin
```

（a）在/home/atguigu/bin目录下创建xsync文件

`[atguigu@hadoop102 ~]$ mkdir /home/atguigubin`

`[atguigu@hadoop102 bin]$ vim /home/atguigu/bin/xsync`

在该文件中编写如下代码

```
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
  echo ==================== $host ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
   #4. 判断文件是否存在
    if [ -e $file ]
      then
        \#5. 获取父目录
        pdir=$(cd -P $(dirname $file); pwd)
        \#6. 获取当前文件的名称
        fname=$(basename $file)
        ssh $host "mkdir -p $pdir"
        rsync -av $pdir/$fname $host:$pdir
      else
        echo $file does not exists!
     fi
  done
done


```

（b）修改脚本 xsync 具有执行权限

`[atguigu@hadoop102 bin]$ chmod +x xsync`

（d）将脚本复制到/bin中，以便全局调用

`[atguigu@hadoop102 bin]$ sudo cp xsync /bin/`

（e）同步环境变量配置（root所有者）

`[atguigu@hadoop102 ~]$ sudo ./bin/xsync /etc/profile.d/my_env.sh`

注意：如果用了sudo，那么xsync一定要给它的路径补全。

让环境变量生效

`[atguigu@hadoop103 bin]$ source /etc/profile`

`[atguigu@hadoop104 opt]$ source /etc/profile`

#### SSH免密登录

（1）免密登录原理：A服务器`ssh-key-gen` 命令生成密钥对，A服务器`ssh-copy-id hadoop102`将公钥注册到B服务器的**Authorized_keys**中进行授权，
当A要ssh登录到B时，A使用公钥加密发生数据给B，B接收到后去**Authorized_keys**寻找对应的公钥，B使用公钥加密发生数据A，A接收到后使用私钥进行解密发送数据给B，B进行验证后允许Assh远程登录。

（2）生成公钥和私钥

`[atguigu@hadoop102 .ssh]$ pwd`

`/home/atguigu/.ssh`

`[atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa`

然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

（3）将公钥拷贝到要免密登录的目标机器上

`[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102`

`[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103`

`[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104`

注意：

还需要在hadoop103上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

还需要在hadoop104上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104；

**3**）.ssh文件夹下（~/.ssh）的文件功能解释

**known_hosts   记录ssh访问过计算机的公钥（public  key）** 
**id_rsa          生成的私钥** 
**id_rsa.pub     生成的公钥** 
**authorized_keys 存放授权过的无密登录服务器公钥**


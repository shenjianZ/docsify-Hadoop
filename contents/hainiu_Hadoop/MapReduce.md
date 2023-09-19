## 1 mapreduce

 引入 MapReduce 框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将 分布式计算中的复杂性交由框架来处理

### 1.1 mapreduce介绍

1. MapReduce是一种分布式计算模型
2. Doug Cutting根据《MapReduce: Simplified Data Processing on Large Clusters》设计实现了Hadoop中基于HDFS的MapReduce
3. MapReduce是由两个阶段组成：Map和Reduce，用户只需要实现map以及reduce两个函数，即可实现分布式计算，这样做的目的是简化分布式程序的开发和调试周期
4. JobTracker/ResourceManager：任务调度者，管理多个NodeManager。ResourceManager是Hadoop2.0版本之后引入Yarn之后用于替代JobTracke部分功能的机制
5. TaskTracker/NodeManager：任务执行者

### 1.2 mapreduce 作用？

**mapreduce的核心思想：化大为小，分而治之。**

 mapreduce 的主要组成部分：Mapper 和 Reducer。

 Mapper： 负责“分”，即把复杂的任务分解为若干个“简单的任务”来处理。

 Reducer： 负责对map阶段的结果进行汇总。

## 2 配置开发环境

### 2.1 创建一个maven工程

![2023-09-17_221523](C:\Users\15202\Desktop\pic\2023-09-17_221523.png)



![2023-09-17_222204](C:\Users\15202\Desktop\pic\2023-09-17_222204.png)

新建一个模块**MapReduceTest**

![2023-09-17_222204](C:\Users\15202\Desktop\pic\2023-09-17_222204.png)

在**MapReduceTest**的**pom.xm**l文件增添hadoop依赖

 添加在<dependencies> </dependencies>标签中

```xml
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>3.1.4</version>
</dependency>
```

右上角刷新一下

### 2.2  导入集群的配置文件

![2023-09-17_223423](C:\Users\15202\Desktop\pic\2023-09-17_223423.png)

使用XFTP传输hdfs-site.xml、core-site.xml到本地桌面，再将他们拖入**MapReduceTest**的**resources**中

![2023-09-17_223818](C:\Users\15202\Desktop\pic\2023-09-17_223818.png)

### 2.3  编写代码

1）自定义的mapreduce类

2）继承Mapper类，实现map函数

3）继承Reducer类，实现reduce函数

4）main()中设置Job相关信息 和 提交Job运行

**Mapper代码**

```java
package com.hainiu.wordcount;

import org.apache.hadoop.io.*;
import org.apache.hadoop.io.serializer.Serializer;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * @auther chenzhe
 * @date 2022/10/20
 */
/*
* KEYIN：输入到mapper类的key的类型
* VALUEIN：输入到mapper类的value的类型
*  KEYOUT：mapper类输出的key的类型
* VALUEOUT：mapper类输出的value的类型
*
* hadoop要求mapreduce计算过程中，所有的数据必须经过序列化，基本类型是没有序列化的，所以hadoop帮我们将基本类型重新实现了序列化
* hadoop用的序列化框架是avro 比java原生的序列化框架性能更高
* */
public class WordCountMapper extends Mapper <LongWritable, Text, Text, IntWritable> {

    //--没读取一行数据就会调用一次map方法
    //-- key 行首偏移量
    //-- value 一行数据
    //-- context是 mr的上下文对象
   Text keyout=new Text();
   IntWritable valueout=new IntWritable();
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        System.out.println("map执行了。。。。。");
        //--value 就是一行数据
        String line = value.toString();
        //--切分 【hello world】
        String[] words = line.split(" ");
        for (String word : words) {
            //--将每个单词进行打标记1输出
            //--hadoop会根据你map输出的key帮我们进行聚合
            keyout.set(word);
            valueout.set(1);
            context.write(keyout,valueout);
        }
    }
}
```

**Reducer**

```java
public class WordCountReducer  extends Reducer<Text, IntWritable,Text,IntWritable> {
    @Override
    //--key就是map输出之后，hadoop框架聚合之后的key values就是相同的key聚合后的数据
    //--hadoop框架聚合之后，有多少个key就会调用多少次reduce方法
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int sum=0;
        for (IntWritable value : values) {
            int i = value.get();
            sum+=i;
        }

        //--输出
        context.write(key,new IntWritable(sum));
    }
}
```

**Driver类代码**

创建Job任务，定义输入输出文件目录

```java
package com.hainiu.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @auther chenzhe
 * @date 2022/10/20
 * 怎么通过参数动态传递的方式运行mapreduce
 */
public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        System.out.println(args[0]);
        System.out.println(args[1]);

        //--创建hadoop的配置对象
        Configuration conf=new Configuration();
        //--定义一个job用来启动一个任务
        Job  job = Job.getInstance(conf, "wc");
        //--定义任务的入口类
        job.setJarByClass(WordCountDriver.class);
        //--定义mapper类
        job.setMapperClass(WordCountMapper.class);
        //--定义mapper类的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        //--定义reduce类
        job.setReducerClass(WordCountReduce.class);
        //--定义reduce的输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //--处理文件
        FileInputFormat.addInputPath(job,new Path("hdfs://ns1/word/words.txt"));
        //--定义结果目录
        FileOutputFormat.setOutputPath(job,new Path(""hdfs://ns1/wcresult""));

        //--启动job运行
        boolean b = job.waitForCompletion(true);
        System.exit(b?0:1);
    }
}
```





## 3 Mapreduce 原理

### **3.1 map任务的输入文件是怎么分割的**

 mapreduce中生成多少个MapTask是由输入文件的输入分片决定的，有多少输入分片，就会生成多少MapTask。

**输入文件分片方法：**

1）首先查看输入文件的压缩格式是否支持split，如果不支持，则一个文件对应一个split；

2）如果支持split，会默认按照一个hdfs块大小对应一个split。

3）如果文件剩余块大小/分块大小>1.1，那会生成两个split；

如果文件剩余大小/splitsize<=1.1，剩余的部分作为一个split。

MapReduce 通过 org.apache.hadoop.mapreduce.InputSplit 类提供数据切片方法(抽象类)。

1）文件个数

2）文件是否支持split

![](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221214175627682.png)

3）hdfs块大小

4）1.1 的系数

在 org.apache.hadoop.mapreduce.lib.input.FileInputFormat 这个类中有一个神奇的参数 :

```php
private static final double SPLIT_SLOP = 1.1;   // 10% slop
```

### 3.2  MapReduce灵魂shuffle 过程

**一个Reducer的情况**

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/shuffle.png)

**多个Reducer的情况（2个为例）**

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/shuffle2-1672303114622.png)

有多少个reducer将来就有多少个输出文件，一般reducer的数量和分区的数量是一致的。

如何确定哪些数据写入哪个Partition？

假设：reduce数量是2

partitionId = key的hash值% reduce的数量

partitionA = key的hash值尾数是0

partitionB = key的hash值尾数是1

如果reduce数量是3

partitionA = key的hash值尾数是0

partitionB = key的hash值尾数是1

partitionC = key的hash值尾数是2

4 怎么设置输出压缩

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221229154047733.png)

在shuffle过程中，reducer端 拉去数据并进行merge数据 占整个reducer运行进度的33%，但可能因为map阶段文件分布不均导致该阶段耗费50-70%的时间，**怎么减少reducer从map拉取的数据量**，我们可以设置压缩。

压缩分为map阶段输出压缩，以及reduce阶段输出压缩

**可以在配置文件中进行配置默认压缩**

```xml
  <property>
        <name>mapreduce.map.output.compress</name> 
        <value>true</value>
        <description>map是否开启输出压缩</description>
    </property>

    <property>
        <name>mapreduce.map.output.compress.codec</name>
        <value>org.apache.hadoop.io.compress.Bzip2Codec</value>
        <description>map输出默认的算法</description>
    </property>
 <property>
        <name>mapreduce.output.fileoutputformat.compress</name> 
        <value>true</value>
        <description>reduce是否开启输出压缩</description>
    </property>

    <property>
        <name>mapreduce.output.fileoutputformat.compress.codec</name>
        <value>org.apache.hadoop.io.compress.Bzip2Codec</value>
        <description>reduce输出默认的算法</description>
    </property>
```

1）设置reducer压缩

```java
// ---设置reduce输出压缩---
// 1）开启reduce输出压缩
FileOutputFormat.setCompressOutput(job, true);
// 2）设置输出压缩格式--gzip
FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class);
```

**2）设置mapper 压缩**

```java
//方式1：
// 开启map输出压缩
conf.set(MRJobConfig.MAP_OUTPUT_COMPRESS, "true");
// 设置输出压缩格式是 Bzip压缩
conf.set(MRJobConfig.MAP_OUTPUT_COMPRESS_CODEC, Bzip2Codec.class.getName());
// 创建运行mapreduce任务的Job对象
// 当创建job对象时，会把 conf里面的所有数据 拷贝到 job对象的Configuration里
Job job = Job.getInstance(conf, "wordcount");

//方式2：
// 开启map输出压缩
job.getConfiguration().set(mapreduce.map.output.compress", "true");
                           // 设置输出压缩格式是 Bzip压缩
                           job.getConfiguration().set("mapreduce.map.output.compress.codec", Bzip2Codec.class.getName());
```





## 4 设置map阶段Combiner

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221229163207963.png)

提前在map节点进行合并计算，这样输出到reducer的数据量就会变少，reducer的计算压力也会减小，

因为combiner是发生在map输出到缓冲区，在缓冲区完成的。要实现的需求是提前进行累加计算，所以和reduce的功能是一样的，需要实现的方法是reduce方法。

Combiner代码：

```java
public class WordCountCombiner  extends Reducer<Text, IntWritable,Text,IntWritable> {
    @Override
    //--key就是map输出之后，hadoop框架聚合之后的key values就是相同的key聚合后的数据
    //--hadoop框架聚合之后，有多少个key就会调用多少次reduce方法
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int sum=0;
        for (IntWritable value : values) {
            int i = value.get();
            sum+=i;
        }

        //--输出
        context.write(key,new IntWritable(sum));
    }
}
```

并在job中进行设置

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221214201322032.png)

**增加combiner和不加的区别：**

没有提前combiner的：

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221214201513679.png)

增加combiner的

![file](http://www.hainiubl.com/uploads/md_images/202301/03/23/image-20221214201627469.png)

**注意：类似于单词统计这样的需求可以提前对单词在map节点进行统计，适合于提前combiner。但是不适用于所有业务**
+++
draft = true
date = 2022-04-21T19:10:10+08:00
title = "OpenLooKeng性能测试方案"
description = "对包括基本使用、环境部署、二次开发以及大厂实践经验进行简单介绍。"
slug = ""
tags = ["性能测试"]
categories = ["openlookeng"]
externalLink = ""
series = []
+++
<a name="ikR01"></a>
## 背景
OpenLooKeng是一款高效的数据虚拟化融合分析引擎，旨在让大数据变简单。
<a name="wtYAm"></a>
## 目的
本次测试的目的是检查OpenLooKeng核心模块的性能情况。为了保证后期在数据量不断增长的情况下系统能够稳定运行，需要对大数据量下系统运行的服务器指标和系统指标进行收集，并在后期就行比对调优，最终为生产环境的稳定运行提供参考。
<a name="aNkkv"></a>
## 环境
<a name="jsZqX"></a>
### 硬件信息
| 操作系统 | CPU | 内存 | 磁盘 | 备注 |
| --- | --- | --- | --- | --- |
| Centos 7 | 8核 | 16GB | 500GB | coordinator、worker |
| Centos 7 | 8核 | 16GB | 500GB | coordinator、worker |
| Centos 7 | 8核 | 16GB | 500GB | worker |

<a name="z9dng"></a>
## 方案
本次测试主要针对以下几个方面：

1. 使用官方提供的presto-benchmark-driver-1.5.0-executable.jar对OpenLooKeng进行性能测试；
1. 使用hive-testbench生成测试数据和测试语句，并对Hive进行性能测试；
1. 使用JMeter对OpenLooKeng和Hive的查询引擎分别进行并发吞吐量测试。
> 注：本文以Hive为例。

<a name="rPJu2"></a>
### 测试工具
<a name="mvBbv"></a>
#### presto-benchmark-driver
工具名称：presto-benchmark-driver<br />下载地址：[https://repo1.maven.org/maven2/io/hetu/core/presto-benchmark-driver/1.5.0/presto-		benchmark-driver-1.5.0-executable.jar](https://repo1.maven.org/maven2/io/hetu/core/presto-benchmark-driver/1.5.0/presto-benchmark-driver-1.5.0-executable.jar)<br />使用教程：[https://mumu-presto.readthedocs.io/zh/latest/installation/benchmark.html](https://mumu-presto.readthedocs.io/zh/latest/installation/benchmark.html)
<a name="ollbj"></a>
#### hive-testbench
工具名称：hive-testbench<br />下载地址Github：[https://github.com/hortonworks/hive-testbench/](https://github.com/hortonworks/hive-testbench/)<br />使用教程：[http://t.zoukankan.com/cyanrose-p-11881251.html](http://t.zoukankan.com/cyanrose-p-11881251.html)
<a name="ro4Kh"></a>
#### JMeter
工具名称：jmeter<br />下载地址Github：[https://github.com/apache/jmeter](https://github.com/apache/jmeter)<br />使用教程：[https://www.cnblogs.com/yhtboke/p/14836966.html](https://www.cnblogs.com/yhtboke/p/14836966.html)
<a name="Eb5f3"></a>
### 测试方法
<a name="lxHvY"></a>
#### presto-benchmark-driver

1. 按照使用教程安装配置presto-benchmark-driver；
1. 分别在OpenLooKeng和Hive上执行查询语句，并统计第一次查询耗时、第二次查询耗时、CPU以及内存负载等参数（可借助大数据平台性能监控组件进行数据统计）；
1. 将统计数据生成可视化图表，对OpenLooKeng和Hive数据进行对比分析；
1. 根据分析结果产生结论。
   <a name="VzPal"></a>
#### JMeter

1. 数据集使用hive-testbench生成或自定义即可；
1. 启动JMeter并配置连接OpenLooKeng和Hive数据源，执行测试脚本；
1. 结合聚合报告，统计分析测试结果；
1. 根据分析结果产生结论。
   <a name="Huela"></a>
## 数据集
<a name="rqfx7"></a>
### 生成测试数据
通过 hive-testbench 工具设置参数10生成的测试集，其规模如下，称为hive-testbench-10。
> 注：hive-testbench生成的测试数据和测试语句全文共用。

```shell
#生成10GB数据和查询语句脚本
sh {hive-testbench}/tpcds-setup.sh 10
```
等待执行成功后通过beeline登录到hive查看生成结果
```shell
beeline -n hive -u 'jdbc:hive2://emr200.dtwave.com:2181,emr225.dtwave.com:2181,emr97.dtwave.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2?tez.queue.name=default'

show databases;
```
hive-testbench-10会生成名为tpcds_bin_partitioned_orc_10的数据库<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1650263890211-8ae381e4-70e7-47ec-a883-13eb95c5bf37.png#clientId=u7060a19b-46c8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=173&id=u2015ed89&margin=%5Bobject%20Object%5D&name=image.png&originHeight=426&originWidth=1512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=122827&status=done&style=none&taskId=u277aedc1-0b5a-49db-84a8-fa3ffe5c456&title=&width=614)
<a name="AElVB"></a>
### 测试语句
```sql
Q1:select cd_gender, cd_marital_status, cd_education_status, cd_purchase_estimate, cd_credit_rating, cd_dep_count, cd_dep_employed_count, cd_dep_college_count, from customer c,customer_address ca,customer_demographics where c.c_current_addr_sk = ca.ca_address_sk and cd_demo_sk = c.c_current_cdemo_sk;
Q2:select  i_item_id, ca_country, ca_state, ca_county from catalog_sales, customer_demographics cd1, customer_demographics cd2, customer, customer_address, date_dim, item where cs_sold_date_sk = d_date_sk and cs_item_sk = i_item_sk and cs_bill_cdemo_sk = cd1.cd_demo_sk and cs_bill_customer_sk = c_customer_sk and c_current_cdemo_sk = cd2.cd_demo_sk and c_current_addr_sk = ca_address_sk;
Q3:select iss.i_brand_id brand_id ,iss.i_class_id class_id ,iss.i_category_id category_id from store_sales ,item iss ,date_dim d1 where ss_item_sk = iss.i_item_sk and ss_sold_date_sk = d1.d_date_sk;
```
<a name="Veaay"></a>
### 执行测试
<a name="ne7qB"></a>
#### presto-benchmark-driver测试

1. 安装配置完成后如下图

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1649929026268-f3fa6e45-7d86-4e9b-a7e1-9bdf48ab15ed.png#clientId=u90a4a42f-121b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=221&id=uef417bb8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=276&originWidth=1095&originalType=binary&ratio=1&rotation=0&showTitle=false&size=225659&status=done&style=none&taskId=u52c5cbff-a60c-4b57-ad5d-8b3d4f446db&title=&width=876)

2. 执行测试语句
```shell
./presto-benchmark-driver --server localhost:9090 --debug  --catalog hive_561019627216896_1648125091607ivlw --schema tpcds_bin_partitioned_orc_10
```

3. 执行结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1649923606033-b6f8f049-d71e-47ae-82d2-3eed5035739b.png#clientId=u90a4a42f-121b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=253&id=cXySC&margin=%5Bobject%20Object%5D&name=image.png&originHeight=316&originWidth=1557&originalType=binary&ratio=1&rotation=0&showTitle=false&size=271091&status=done&style=none&taskId=ubdbd4aeb-2ff1-4244-b223-501d59a5c58&title=&width=1245.6)<br />参数说明：

- WallTime：用户要可以看到查询结果要等待的时间；
- processCpuTime：整个集群为助力查询而消耗的CPU时间，包含一些垃圾回收的时间；
- queryCpuTime：整个集群为助力查询而消耗的CPU时间；
> 注：一共包含三种时间WallTime,ProcessCpuTime,queryCpuTime,每种时间都包含P50,Mean,stand三个值。

<a name="NUN4G"></a>
#### hive-testbench测试
进入Hive命令行执行上文生成或自定义的测试语句，统计查询性能数据即可。
```shell
 cd sample-queries-tpcds
 hive -i testbench.settings
 hive> use tpcds_bin_partitioned_orc_10;
 hive> select cd_gender, cd_marital_status, cd_education_status, cd_purchase_estimate, cd_credit_rating, cd_dep_count, cd_dep_employed_count, cd_dep_college_count, from customer c,customer_address ca,customer_demographics where c.c_current_addr_sk = ca.ca_address_sk and cd_demo_sk = c.c_current_cdemo_sk limit 1000;
```
<a name="dAkXu"></a>
#### JMeter测试
分别对Hive和OpenLooKeng进行并发吞吐量测试：分别将Hive和OpenLooKeng驱动包添加到{jmeter}/lib下，重启JMeter并创建JDBC Connection Configuration、JDBC Request，并配置相关参数：

1. 测试Hive并发吞吐量：
    1. 设置并发数：采用分组测试，分别设置Number of Threads为5、10、20；
> 注：由于测试机内存限制，线程数设置较为保守。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1649841655550-20d964f1-8aed-46a6-975e-db262d7fc549.png#clientId=u77025df6-a597-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=880&id=u50fa9897&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1100&originWidth=2304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=480854&status=done&style=none&taskId=ubb658b5f-b386-40cb-a44a-3c371a2d25a&title=&width=1843.2)

2. 填写数据源连接信息
```yaml
#Hive
database URL: jdbc:hive2://localhost:10000/test
JDBC Driver class: org.apache.hive.jdbc.HiveDriver
Username: root
Password: root

#OpenLooKeng
database URL: jdbc:lk://localhost:8123/hiveDB/oem_12
JDBC Driver class: io.hetu.core.jdbc.OpenLooKengDriver
Username: root
Password: root
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1649843319457-8ffeb2b2-e24d-4fa2-9510-5f5baaee8e2c.png#clientId=u77025df6-a597-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1152&id=u440e3800&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1440&originWidth=2304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=621772&status=done&style=none&taskId=u75d2bcb4-3a30-4d85-8701-af5491f4564&title=&width=1843.2)

3. 在JDBC Request中将生成的测试脚本拷贝进去，执行查看结果。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2707056/1649843376913-e43c49bc-8360-459f-acb8-aef519aa8a8a.png#clientId=u77025df6-a597-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=534&id=u465107e6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=668&originWidth=2304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=512370&status=done&style=none&taskId=uf9421289-c611-49f8-a7cd-fb43cf52b7b&title=&width=1843.2)

2. 测试OpenLooKeng并发吞吐量：同Hive流程，修改数据源配置即可。
   <a name="tUGxQ"></a>
## 测试结果
<a name="b0N2n"></a>
### 执行查询语句
| 参数\\组件 | OpenLooKeng | Hive |
| --- | --- | --- |
| 第一次执行耗时 | 3.12 s | 51.713 s |
| 第二次执行耗时 | 2.58 s | 51.193 s |
| CPU | 70% + | 90% + |
| 内存 | 40% + | 20% + |
| 执行语句 | select cd_gender, cd_marital_status, cd_education_status, cd_purchase_estimate, cd_credit_rating, cd_dep_count, cd_dep_employed_count, cd_dep_college_count, from customer c,customer_address ca,customer_demographics where c.c_current_addr_sk = ca.ca_address_sk and cd_demo_sk = c.c_current_cdemo_sk <br />limit 1000; | select cd_gender, cd_marital_status, cd_education_status, cd_purchase_estimate, cd_credit_rating, cd_dep_count, cd_dep_employed_count, cd_dep_college_count, from customer c,customer_address ca,customer_demographics where c.c_current_addr_sk = ca.ca_address_sk and cd_demo_sk = c.c_current_cdemo_sk <br />limit 1000; |
| 备注 | 当去掉limit分页限制，OpenLooKeng的查询耗时会增加至30 s左右 | 当去掉limit分页限制，Hive的查询耗时基本不变 |

<a name="YTWqo"></a>
### JMeter并发吞吐量测试
| 参数\\组件 | OpenLooKeng |  |  | Hive |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| Samples | 5 | 10 | 20 | 5 | 10 | 20 |
| Average | 2945 | 2627 | 2541 |  |  |  |
| Min | 2753 | 2215 | 2215 |  |  |  |
| Max | 3048 | 3048 | 3048 |  |  |  |
| Throughput | 4.4/sec | 3.4/sec | 2.2/sec |  |  |  |
| Received KB/sec | 2.81 | 0.22 | 0.14 |  |  |  |

> 上表参数解释如下：
> - samples：请求线程数
> - average：平均响应时间（单位：毫秒）
> - min：最小相应时间（单位：毫秒）
> - max：最大响应时间（单位：毫秒）
> - throughput：吞吐量--默认情况下表示每秒完成的请求数
> - received KB/sec：每秒从服务端接收到的数据量

<a name="VChD8"></a>
## 结论
        由以上测试结果可以看出，同等条件下在查询性能方面，OpenLooKeng明显要比Hive快，但数据量和内存对OpenLooKeng的性能会有一定程度上的影响，而对Hive来说影响不大；在并发性能方面，OpenLooKeng优于Hive，OpenLooKeng并发性能同样会受到数据量和内存的影响，但两者都容易出现服务挂掉的情况。

+++
draft = true
date = 2022-03-25T22:23:15+08:00
title = "OpenLooKeng性能测试报告"
description = "对包括基本使用、环境部署、二次开发以及大厂实践经验进行简单介绍。"
slug = ""
tags = ["性能测试"]
categories = ["openlookeng"]
externalLink = ""
series = []
+++
<a name="nBr1t"></a>
## 一、测试环境
<a name="JcuDU"></a>
### 1.1 硬件信息
| 操作系统 | CPU | 内存 | 磁盘 |
| --- | --- | --- | --- |
| Centos 7 | 6核 | 32GB | 1.5TB |
| Centos 7 | 6核 | 32GB | 1.5TB |
| Centos 7 | 6核 | 32GB | 1.5TB |

<a name="k4Hjq"></a>
### 1.2 软件信息
| 组件名称 | 版本 |
| --- | --- |
| JDK | 1.8.0_322 |
| OpenLooKeng | 1.5.0 |
| Jmeter | 5.4.3 |
| presto-benchmark-driver | 1.5.0 |
| hive-testbench | / |
| Nginx | 1.20.2 |

<a name="bJ8Oo"></a>
## 二、测试方法
本次测试主要针对以下两个方面：

1. 使用官方提供的presto-benchmark-driver-1.5.0-executable.jar对OpenLooKeng进行性能测试。
    1. Scan query
    1. Aggregation query
    1. Join query
2. 使用JMeter对OpenLooKeng的查询引擎进行并发吞吐量测试，分别设置并发数为10、50、100进行。
> 注：本文以Hive数据源为例，OpenLooKeng使用Hive Connector进行连接。

<a name="VN2zq"></a>
## 三、测试数据
使用开源数据生成工具[hive_testbench](https://github.com/hortonworks/hive-testbench/)生成测试数据，生成数据总量为10GB，本次用到的测试数据大约2GB。
<a name="SxMt4"></a>
## 四、执行性能测试
<a name="sD9Yx"></a>
### 4.1 SQL性能测试
<a name="ev0Sb"></a>
#### 4.1.1 Scan query
```sql
select
    ss_sold_time_sk,
    ss_item_sk,
    ss_customer_sk
from
    store_sales
where
    ss_cdemo_sk > 20000
limit
    1000;
```
<a name="PjW6W"></a>
#### 4.1.2 Aggregation query
```sql
select
    dt.d_year,
    item.i_brand_id brand_id,
    item.i_brand brand,
    sum(ss_sales_price) sum_agg
from
    date_dim dt,
    store_sales,
    item
where
    dt.d_date_sk = store_sales.ss_sold_date_sk
    and store_sales.ss_item_sk = item.i_item_sk
    and item.i_manufact_id = 816
    and dt.d_moy = 11
group by
    dt.d_year,
    item.i_brand,
    item.i_brand_id
order by
    dt.d_year,
    sum_agg desc,
    brand_id
limit
    1000;
```
<a name="D9Z2h"></a>
#### 4.1.3 Join query
```sql
select cd_gender, cd_marital_status, cd_education_status, cd_purchase_estimate, cd_credit_rating, cd_dep_count, cd_dep_employed_count, cd_dep_college_count from customer c,customer_address ca,customer_demographics where c.c_current_addr_sk = ca.ca_address_sk and cd_demo_sk = c.c_current_cdemo_sk limit 1000;
select
    cd_gender,
    cd_marital_status,
    cd_education_status,
    cd_purchase_estimate,
    cd_credit_rating,
    cd_dep_count,
    cd_dep_employed_count,
    cd_dep_college_count
from
    customer c,
    customer_address ca,
    customer_demographics
where
    c.c_current_addr_sk = ca.ca_address_sk
    and cd_demo_sk = c.c_current_cdemo_sk
limit
    1000;

```
<a name="q7kTb"></a>
### 4.2 JMeter测试
Jdbc请求脚本统一用：
```sql
select
    cd_gender,
    cd_marital_status,
    cd_education_status,
    cd_purchase_estimate,
    cd_credit_rating,
    cd_dep_count,
    cd_dep_employed_count,
    cd_dep_college_count
from
    customer c,
    customer_address ca,
    customer_demographics
where
    c.c_current_addr_sk = ca.ca_address_sk
    and cd_demo_sk = c.c_current_cdemo_sk
limit
    1000
```
<a name="xSfYQ"></a>
#### 4.2.1 并发数10
结果：<br />![adfa.png](../../../images/adfa.png)
<a name="m1pT0"></a>
#### 4.2.1 并发数50
结果：<br />![ssgdgdsg](../../../images/ssgdgdsg.png)
<a name="rSkba"></a>
#### 4.2.1 并发数100
结果：<br />![fsdshehe.png](../../../images/fsdshehe.png)
<a name="zzAek"></a>
## 五、结果总结
<a name="fdlKG"></a>
### 5.1 SQL性能测试
| 参数\\组件 | OpenLooKeng |  |  |
| --- | --- | --- | --- |
|  | Query scan | Aggregation scan | Join scan |
| 第一次执行耗时 | 5.83 s | 36.06 s | 12.58 s |
| 第二次执行耗时 | 5.54 s | 32.01 s | 11.23 s |
| CPU | 50% + | 70% + | 50% + |
| 内存 | 10% + | 20% + | 20% + |

<a name="oY7Al"></a>
### 5.2 JMeter测试
| 参数\\组件 | OpenLooKeng |  |  |
| --- | --- | --- | --- |
| Samples | 10 | 50 | 100 |
| Average | 14893 | 72955 | 47878 |
| Min | 14046 | 66442 | 37451 |
| Max | 15513 | 76075 | 54587 |
| Throughput | 0.62/sec | 0.68/sec | 1.8/sec |
| Received KB/sec | 28.54 | 30.02 | 83.96 |

> 上表参数解释如下：
> - samples：请求线程数
> - average：平均响应时间（单位：毫秒）
> - min：最小相应时间（单位：毫秒）
> - max：最大响应时间（单位：毫秒）
> - throughput：吞吐量--默认情况下表示每秒完成的请求数
> - received KB/sec：每秒从服务端接收到的数据量

<a name="WblH2"></a>
## 六、参考
【1】OpenLooKeng性能测试方案：[https://dtwave.yuque.com/wm7zom/xxznbo/nlgz0f](https://dtwave.yuque.com/wm7zom/xxznbo/nlgz0f)

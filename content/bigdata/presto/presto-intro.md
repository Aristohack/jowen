---
title: "Presto架构原理与实践经验"
date: 2022-01-29T10:46:53+08:00
draft: true
---
## 介绍
Presto是由 Facebook 推出的一个**基于Java开发的开源分布式SQL查询引擎**，适用于交互式分析查询，数据量支持TB到PB字节。Presto本身并不存储数据，但是可以接入多种数据源，并且支持跨数据源的级联查询。
> 注：Presto 不是通用的关系数据库。它不能替代 MySQL、PostgreSQL 或 Oracle 等数据库。设计Presto的目的并不是处理OLTP型事务。

### 1.基本概念
#### 1.1 术语
[请参考官网。](https://prestodb.io/docs/current/overview/concepts.html)
#### 1.2 特点
Presto引擎相较于其他引擎的特点正如⽂章标题描述的这样，多源就是它可以⽀持跨不同数据源的联邦查询，即席即实时计算，将要做的查询任务实时拉取到本地进⾏现场计算，然后返回计算结果。除此之外，对于引擎本身，它有⼏个值得关注的特点：

- 多租户：它⽀持并发执⾏数百个内存、I/O以及CPU密集型的负载查询，并⽀持集群规模扩展到上千个节点；
- 联邦查询：它可以由开发者利⽤开放接⼝⾃定义开发针对不同数据源的连接器（Connector),从⽽⽀持跨多种不同数据源的联邦数据查询；
- 内在特性：为了保证引擎的⾼效性，Presto还进⾏了一些优化，例如基于JVM运⾏，Code-Generation等。
#### 1.3 应用场景
Presto 的应⽤场景⾮常⼴泛，接下来我们主要介绍⼏种使⽤⽐较⼴泛的场景进⾏介绍。

- 交互式分析：交互式查询是Presto主打的应⽤场景，Presto的即席计算特性和内部设计机制就是为了能够更好地⽀持⽤户进⾏交互式分析。可以类⽐⽤户基于Hive交互式查询HDFS中的数据，⽤户可以基于Presto查询各种不同的数据源的数据；
- 批量 ETL；
- Facebook 的 A/B Test 基础架构也是基于Presto 构建的。
### 2.整体架构图
Presto查询引擎是一个Master-Slave的架构，由一个Coordinator节点，一个Discovery Server节点，多个Worker节点组成，Discovery Server通常内嵌于Coordinator节点中。Coordinator负责解析SQL语句，生成执行计划，分发执行任务给Worker节点执行。Worker节点负责实际执行查询任务。Worker节点启动后向Discovery Server服务注册，Coordinator从Discovery Server获得可以正常工作的Worker节点。如果配置了Hive Connector，需要配置一个Hive MetaStore服务为Presto提供Hive元信息，Worker节点与HDFS交互读取数据。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2707056/1640501799769-d498dc34-e73e-4901-b2b6-eef480c2fd29.png#clientId=u7e3b3ccf-ffa5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=321&id=u137ed407&margin=%5Bobject%20Object%5D&name=image.png&originHeight=641&originWidth=1270&originalType=binary&ratio=1&rotation=0&showTitle=true&size=138027&status=done&style=none&taskId=ueb8e4524-1eb3-4317-9eab-27b6014d8d3&title=Presto%E6%9E%B6%E6%9E%84%E5%9B%BE&width=635 "Presto架构图")
## 安装部署
### 1.单机部署
### 2.集群部署
### 3.容器化部署
## 安全认证
### 1.[Coordinator Kerberos 身份验证](https://prestodb.io/docs/current/security/server.html)
#### 1.1 config.properties
Kerberos 身份验证在协调器节点的config.properties文件中配置 。下面列出了需要添加的条目。
```
http-server.authentication.type=KERBEROS

http.server.authentication.krb5.service-name=presto
http.server.authentication.krb5.service-hostname=presto.example.com
http.server.authentication.krb5.keytab=/etc/presto/presto.keytab
http.authentication.krb5.config=/etc/krb5.conf

http-server.https.enabled=true
http-server.https.port=7778

http-server.https.keystore.path=/etc/presto_keystore.jks
http-server.https.keystore.key=keystore_password
```
| **配置项** | **描述** |
| --- | --- |
| http-server.authentication.type | Presto Coordinator的身份验证类型。必须设置为KERBEROS. |
| http.server.authentication.krb5.service-name | Presto Coordinator的 Kerberos 服务名称。必须与 Kerberos 主体匹配。 |
| http.server.authentication.krb5.principal-hostname | Presto Coordinator的 Kerberos 主机名。必须与 Kerberos 主体匹配。该参数是可选的。如果包含，Presto 将在 Kerberos 主体的主机部分中使用此值，而不是机器的主机名。 |
| http.server.authentication.krb5.keytab | 可用于验证 Kerberos 主体的密钥表的位置。 |
| http.authentication.krb5.config | Kerberos 配置文件的位置。 |
| http-server.https.enabled | 为 Presto 协调器启用 HTTPS 访问。应设置为true. |
| http-server.https.port | HTTPS 服务器端口。 |
| http-server.https.keystore.path | 将用于保护 TLS 的 Java 密钥库文件的位置。 |
| http-server.https.keystore.key | 密钥库的密码。这必须与您在创建密钥库时指定的密码相匹配。 |

#### 1.2 故障排查
### 2.[CLI Kerberos 身份验证](https://prestodb.io/docs/current/security/cli.html)
### 3.[LDAP 认证](https://prestodb.io/docs/current/security/ldap.html)
### 4.[密码文件认证](https://prestodb.io/docs/current/security/password-file.html)
### 5.[Java 密钥库和信任库](https://prestodb.io/docs/current/security/tls.html)
### 6.[内置系统访问控制](https://prestodb.io/docs/current/security/built-in-system-access-control.html)
### 7.[安全的内部通信](https://prestodb.io/docs/current/security/internal-communication.html)
### 8.[授权](https://prestodb.io/docs/current/security/authorization.html)
## 性能优化
## 二次开发
## 参考
[https://prestodb.io/docs/current/](https://prestodb.io/docs/current/)（官网）
[https://tech.meituan.com/2014/06/16/presto.html](https://tech.meituan.com/2014/06/16/presto.html)（presto实现原理和美团最佳实践）
[Presto原理调优.pdf](https://dtwave.yuque.com/attachments/yuque/0/2021/pdf/2707056/1640508573878-7e674cb1-684b-4db5-9670-44935c2d8c7b.pdf?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fdtwave.yuque.com%2Fattachments%2Fyuque%2F0%2F2021%2Fpdf%2F2707056%2F1640508573878-7e674cb1-684b-4db5-9670-44935c2d8c7b.pdf%22%2C%22name%22%3A%22Presto%E5%8E%9F%E7%90%86%E8%B0%83%E4%BC%98.pdf%22%2C%22size%22%3A3634679%2C%22type%22%3A%22application%2Fpdf%22%2C%22ext%22%3A%22pdf%22%2C%22status%22%3A%22done%22%2C%22taskId%22%3A%22u2e09aca8-050a-47d8-8c5b-3177ee1b008%22%2C%22taskType%22%3A%22upload%22%2C%22id%22%3A%22ua100e946%22%2C%22card%22%3A%22file%22%7D)
[https://tanggaomeng.blog.csdn.net/article/details/112967839](https://tanggaomeng.blog.csdn.net/article/details/112967839)
[https://trino.io/docs/current/](https://trino.io/docs/current/)

+++
draft = true
date = 2021-05-21T19:10:10+08:00
title = "《Atlas技术调研》"
description = "对包括基本使用、环境部署、二次开发以及大厂实践经验进行简单介绍。"
slug = ""
tags = []
categories = []
externalLink = ""
series = []
+++

| 序号  | 作者  | 版本  | 时间  | 备注  |
| --- | --- | --- | --- | --- |
| 1   | 乔文文  | 1.0.0 | 2021-11-23 | 第一个版本 |

# 一、Atlas和Ranger介绍

## 1.1 Atlas High Level Architecture

Atlas 是一个可伸缩且功能丰富的数据管理系统，深度集成了 Hadoop 大数据组件。简单理解就是一个跟 Hadoop 关系紧密的，可以用来做元数据管理的一个系统。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638789705519-2bdf81c1-fbc5-49c5-a5eb-341e5323abd0.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=966&id=u01e91945&margin=%5Bobject%20Object%5D&name=image.png&originHeight=966&originWidth=1432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=243722&status=done&style=stroke&taskId=u883ce02d-5de3-4d0f-bbcf-d44cbd104fe&title=&width=1432)
Hive中库操作、表操作、运行的SQL都会通过Kafka Messaging 近实时同步到Atlas中，Atlas会存储此元数据以及血缘关系。

## 1.2 Atlas名词解释

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637646796986-9b6f00f9-1e1c-45ec-b12b-7046f77e8350.png#clientId=u15a8b532-4119-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=164&id=uabdc584a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=164&originWidth=691&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9720&status=done&style=none&taskId=u50298840-2b1c-487d-a75f-8bf91d6d82b&title=&width=691)
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637646838228-2a2b93ad-dc19-408c-9fdc-d0e1e2072d28.png#clientId=u15a8b532-4119-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=424&id=u3ec4739f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=424&originWidth=767&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24367&status=done&style=none&taskId=u7968e00a-2e49-4e5a-a6a9-462df17b4b4&title=&width=767)

> 注意点:
>
> - Classification间有层级关系，且拥有父classification的用户(组)可以访问子classification的entity。
> - Term和Categories共享一级目录，Term和Category的关系只能是在一级目录范围内关联。
> - Term可以在 Related Terms设置Terms间的关系，此时能选择其他一级目录下的Term。
> - Term增加Classifications时，如果勾选Propagate，则Term关联的Entity会自动关联此Classification；

反之Entity不会关联到Classification

> - 血缘传递的Classification不能删除。

Entity的两种属性:

1. Entity关联Classification，一个Entity可以添加多个Classification。

  ```
    ![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638795724372-6406d1f6-ec94-4a95-8193-4b0c43447d32.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=190&id=u934069bf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=380&originWidth=568&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21146&status=done&style=none&taskId=u9345c684-bf14-42ab-a19e-5bc4ae60fce&title=&width=284)
  ```


2. Entity关联Term，一个Term可以添加多个Classifacation，一个Entity可以添加多个Term。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638795646490-203234a2-7c6c-4882-9390-3cd97388ac02.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=408&id=u5b586c85&margin=%5Bobject%20Object%5D&name=image.png&originHeight=816&originWidth=728&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42244&status=done&style=none&taskId=u7bac7ed2-2c16-470e-9bef-e1831aa71fe&title=&width=364)
本质上也是Entity通过关联Term的方式间接关联到Classification。

## 1.3 Atlas和Ranger组合使用流程图

> 注意点:
>
> - Ranger Tag Based Policies 包含Acces 和Masking两种策略，不支持 Row Level Filter策略。
> - 一个Tag Policy下只能选择一个Classification(Tag)
> - 一个Classfication只能用于一个Access Policy中。
> - 一个Classfication只能用于一个Masking Policy中。

### 1.3.1 新建Ranger Tag Based Access Policy

- Atlas Entity关联Classification后，在Ranger中定义Tag Based Access Policy。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638791652814-64090a7b-d32b-4e58-a376-d38a3c50bf82.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=802&id=ue4cea8dc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=802&originWidth=1294&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74073&status=done&style=none&taskId=u42bfad04-6b98-465a-ae8b-9090d74f640&title=&width=1294)

- Atlas Entity关联Term后，在Ranger中定义Tag Based Access Policy。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638795876199-2791c6c9-6a22-4fbd-a216-6c4b70635622.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=798&id=u2acb9d8e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=798&originWidth=1298&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91584&status=done&style=none&taskId=u527c7ebd-d58a-4b9e-be25-52f9ebb5b63&title=&width=1298)

### 1.3.2 新建Ranger Tag Based Masking Policy

- Atlas Entity关联Classification后，在Ranger中定义Tag Based Masking Policy，其中Component Permissions只支持Hive的select permission。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638794706689-3b508ebb-9c79-4b34-ac3d-3009aa5a7ebf.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=866&id=u84b446a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=866&originWidth=1324&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92638&status=done&style=none&taskId=u1c4296a4-d330-419f-ae4e-7c0d95eac6a&title=&width=1324)

- Atlas Entity关联Term后，在Ranger中定义Tag Based Masking Policy。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638795961450-0b89d020-5740-4be4-adb3-3fe23aab1f08.png#clientId=u5d87398a-ad73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=874&id=uc6620727&margin=%5Bobject%20Object%5D&name=image.png&originHeight=874&originWidth=1308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105196&status=done&style=none&taskId=u3678c370-42de-4c10-bc59-9e935255e06&title=&width=1308)

## 1.4 Apache Ranger Policy Evaluation Flow with Tags

当同时有Tag Policy和Resource Policy的时候，Ranger是按照如下流程进行决策的。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638271354945-eb0720d7-1039-4446-8378-c0223246424b.png#clientId=u43964b9d-4fd6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=457&id=ua843b6ca&margin=%5Bobject%20Object%5D&name=image.png&originHeight=914&originWidth=1688&originalType=binary&ratio=1&rotation=0&showTitle=false&size=179703&status=done&style=stroke&taskId=u3428d5aa-27ee-45ad-8f3f-bea5a7f2881&title=&width=844)
Tag Policy的deny或者allow优先级高于Resource Policy。
​

# 二、产品规划

## 2.1 数据安全3.0

> 1.0，BI同学**直接访问**数据
> 2.0，数据开发、BI、挖掘建模同学通过**虚拟机单向访问**数据。

3.0时代，在“向数据要价值”大旗下，**全民开始使用数据**。以数据分级分类为基础，然后进行权限管控，例如权限控制、脱敏、加密、审计等。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638797171393-9412e9ee-6e6d-4cce-b491-ce3efd7a90ce.png#clientId=udbea385f-5a43-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=389&id=u564eaf08&margin=%5Bobject%20Object%5D&name=image.png&originHeight=389&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49547&status=done&style=none&taskId=u1bdd7d9d-641b-461e-90d5-99e4a896b65&title=&width=582)

## 2.2 基于数据分级的数据安全管理体系

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638803632422-399ce359-ff43-443f-b477-8dfa1d080f41.png#clientId=u88114c1b-7b45-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=440&id=u020565a5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=440&originWidth=819&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48516&status=done&style=none&taskId=u7d079f71-6523-4e83-93bc-1c1ccc3c7af&title=&width=819)

## 2.3 产品核心功能点

### 2.3.1 产品核心功能大图

- 对象、用户、标签、级别间的关系如下：
    - 一个对象可以设置多个级别
    - 一个对象可以设置多个标签
    - 一个用户可以设置多个级别
    - 一个用户可以设置多个标签

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638843720413-040c8892-7418-4c0c-a7ac-50d03d9b46a8.png#clientId=u01f9e259-892e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=700&id=u15e70fef&margin=%5Bobject%20Object%5D&name=image.png&originHeight=700&originWidth=1464&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75359&status=done&style=none&taskId=uea7952c5-def7-484c-ad37-8be1836e455&title=&width=1464)

- 标签授权

    - 当组织成员较多时，使用标签授权可以避免对用户、用户组单独授权，实现一次性为所有用户授权，降低成本和复杂度，方便后续管理。
    - ​
- 数据分级的技术方案

    - 参考《测试Sub-classification》章节即可。

- 标签的技术方案
    - 产品上新建标签时，技术在Atlas上新建一个Term，并且自动创建一个或多个Classification并关联到此Term。
    - 产品上配置对象和标签的关系，技术上在Atlas上配置Entity和Term的关系。
    - 产品上配置标签和用户组的关系，技术上在Ranger上配置Term对应的Classification和用户/用户组的关系。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638883414694-4c24ffee-c2e8-44be-9853-8d34277accde.png#clientId=uf5d8230d-a37e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=455&id=u518f3a53&margin=%5Bobject%20Object%5D&name=image.png&originHeight=910&originWidth=1644&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63907&status=done&style=none&taskId=ud0f17a8c-61ce-44c8-89a0-c3a7add6404&title=&width=822)

### 2.3.2 产品核心流程

#### 2.3.2.1 数据分级

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638844423183-755e1054-dd00-4adf-aede-c569f929a697.png#clientId=u57221412-bd81-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=320&id=ub50add88&margin=%5Bobject%20Object%5D&name=image.png&originHeight=640&originWidth=950&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34539&status=done&style=none&taskId=u5d63addf-e700-40dc-8273-428db562ae6&title=&width=475)

#### 2.3.2.2 标签式授权-列级权限-访问策略

```
                 ![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638845185784-da8f53a9-16fe-48b4-9389-430958e931b5.png#clientId=u57221412-bd81-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=385&id=ua6bec161&margin=%5Bobject%20Object%5D&name=image.png&originHeight=770&originWidth=1168&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74038&status=done&style=none&taskId=u56e4c2e8-5cf2-410f-a4ac-7b1d8952c7a&title=&width=584)
```

​

#### 2.3.2.3 标签式授权-列级权限-屏蔽策略

```
                      ![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638845406553-13865977-51fd-4106-a0c9-5e1099412696.png#clientId=u57221412-bd81-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=372&id=u0e7facc4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=744&originWidth=868&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43779&status=done&style=none&taskId=u5ee9b142-94c3-4705-b523-8cc50f2919f&title=&width=434)
```

#### 2.3.2.4 标签式授权-行级权限

暂不支持

#### 2.3.2.5 数据服务-行级权限

```
  ![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638882601270-12ec5826-4c43-4fad-b2d9-0f8fe94a9ea5.png#clientId=uf5d8230d-a37e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=479&id=u8d6263c1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=958&originWidth=870&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55850&status=done&style=none&taskId=u09c511d5-bc60-4989-8e06-598108e39e2&title=&width=435)
```

#### 2.3.2.6 数据标准-标签式授权

```
   ![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638863552765-188afe16-ec11-49fd-87d8-3ae76dccc301.png#clientId=u93730dea-c676-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=385&id=uf29c0205&margin=%5Bobject%20Object%5D&name=image.png&originHeight=770&originWidth=1026&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58921&status=done&style=none&taskId=u95a1ef19-2afc-4111-bccd-d761efaa249&title=&width=513)
```

## 2.4 沟通问题解答

### 2.4.1 集群用户、项目用户及级别继承关系

- 现有平台用户和集群用户映射关系

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1640680490716-13368b3d-9c4a-4ec1-aa19-6510c54484e7.png#clientId=u1a0f8777-c1a6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=155&id=uff010bca&margin=%5Bobject%20Object%5D&name=image.png&originHeight=310&originWidth=638&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19850&status=done&style=none&taskId=u12fec088-c3b8-4e84-8ce2-27eab4b9111&title=&width=319)

- 5.5 平台用户和集群用户映射关系，绿色表示这次要做的(下同)。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1640680518194-f4a5bad3-e9d9-49c3-9f25-34895d4fd550.png#clientId=u1a0f8777-c1a6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=420&id=ub69d949f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=840&originWidth=934&originalType=binary&ratio=1&rotation=0&showTitle=false&size=83841&status=done&style=none&taskId=ubd6946b5-b755-46c7-ad8d-6070585b00f&title=&width=467)

- 级别继承关系

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1640680544401-af38d89c-067c-4e8a-834e-78a21106a54c.png#clientId=u1a0f8777-c1a6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=710&id=u63ce2c70&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1420&originWidth=1306&originalType=binary&ratio=1&rotation=0&showTitle=false&size=163419&status=done&style=none&taskId=u5d154ebb-cc4a-4448-9995-83b385b1edd&title=&width=653)

### 2.4.2 数栖5.5信息架构（非最终稿）

[数栖平台5.5-数据分级、类别、行级权限.xmind](https://dtwave.yuque.com/attachments/yuque/0/2021/xmind/499433/1640680740559-b59f289b-42ec-436b-8879-041a9473f838.xmind?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fdtwave.yuque.com%2Fattachments%2Fyuque%2F0%2F2021%2Fxmind%2F499433%2F1640680740559-b59f289b-42ec-436b-8879-041a9473f838.xmind%22%2C%22name%22%3A%22%E6%95%B0%E6%A0%96%E5%B9%B3%E5%8F%B05.5-%E6%95%B0%E6%8D%AE%E5%88%86%E7%BA%A7%E3%80%81%E7%B1%BB%E5%88%AB%E3%80%81%E8%A1%8C%E7%BA%A7%E6%9D%83%E9%99%90.xmind%22%2C%22size%22%3A91631%2C%22type%22%3A%22application%2Fvnd.xmind.workbook%22%2C%22ext%22%3A%22xmind%22%2C%22status%22%3A%22done%22%2C%22taskId%22%3A%22u37182a2f-ab3e-4e44-9ead-23f6e3ffea9%22%2C%22taskType%22%3A%22upload%22%2C%22id%22%3A%22u70016c3e%22%2C%22card%22%3A%22file%22%7D)
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1640680602918-e8b65605-f847-4484-b121-053552354a6e.png#clientId=u1a0f8777-c1a6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=716&id=uad110d4d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1432&originWidth=1498&originalType=binary&ratio=1&rotation=0&showTitle=false&size=209736&status=done&style=none&taskId=u92fb870c-9a42-4c2d-bb9e-5a47601e7ab&title=&width=749)
​

其他待解决问题？
问题2:标签要设置数据级别吗？
​

问题3: 由于Ranger中Tag策略优先，会出现没有Resource Policy 的情况下，高级别的人能访问所有已设置低级别密级的数据？
​

## 2.5 优化考虑点

- 考虑 标签表-用户在标签表中的值，有些来来自于组织架构，已有属性的约束值。

# 三、实战准备工作

## 3.1 测试环境

数栖EMR开发环境-安全集群: [http://192.168.90.112:8080](http://192.168.90.112:8080/)
用户名: admin
密码: admin

## 3.2 新建LDAP组和用户

### 3.2.1 LDAP操作工具

操作LDAP目前有两种方式: 网页版和客户端，本文采用客户端的形式。

> 网页版地址: [http://192.168.90.112/ldapadmin/](http://192.168.90.112/ldapadmin/)
> 用户名: ldapadmin
> 密码: root

客户端工具ApacheDirectoryStudio，安装包[ApacheDirectoryStudio安装包.zip](https://dtwave.yuque.com/attachments/yuque/0/2021/zip/499433/1640680740725-1d8ea01d-a105-4280-8599-8efe4b1d04be.zip?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fdtwave.yuque.com%2Fattachments%2Fyuque%2F0%2F2021%2Fzip%2F499433%2F1640680740725-1d8ea01d-a105-4280-8599-8efe4b1d04be.zip%22%2C%22name%22%3A%22ApacheDirectoryStudio%E5%AE%89%E8%A3%85%E5%8C%85.zip%22%2C%22size%22%3A138893747%2C%22type%22%3A%22application%2Fzip%22%2C%22ext%22%3A%22zip%22%2C%22status%22%3A%22done%22%2C%22taskId%22%3A%22u0c19700e-5f31-48aa-91ac-b1b734f4a65%22%2C%22taskType%22%3A%22upload%22%2C%22id%22%3A%22u6913f434%22%2C%22card%22%3A%22file%22%7D)

### 3.2.2 登录LDAP

新建LDAP Connection，Hostname: 192.168.90.112
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637676457264-e3a395e9-65f9-4168-bddb-9b82f0115562.png#clientId=u0acf11c9-046e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=552&id=iwzkq&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1104&originWidth=1488&originalType=binary&ratio=1&rotation=0&showTitle=false&size=155597&status=done&style=stroke&taskId=u320dbdbd-b1a8-46d7-92ed-01f3d1cc203&title=&width=744)
Bind DN or user: uid=ldapadmin,ou=people,dc=hadoop,dc=com
Bind password: root
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637676468923-ac8623e2-6ed3-4ec4-8641-13d0cdce6998.png#clientId=u0acf11c9-046e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1102&id=SfJ21&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1102&originWidth=1490&originalType=binary&ratio=1&rotation=0&showTitle=false&size=148630&status=done&style=stroke&taskId=ud85984f1-087a-44f3-8999-96cd7982408&title=&width=1490)

### 3.2.3 新建组

在ou=group下 New Entity，cn: atlas_c_group，gidNumber: 5000014， objectClass: posixGroup和top。

> 注: 此处的gidNumber以及下文用户的uidNumber 可任意写，但是不要跟已有id冲突。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637673451486-8d4920d9-28fe-44cd-92b8-58f63ea8f3af.png#clientId=u0acf11c9-046e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=266&id=eKJmD&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=1504&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62647&status=done&style=stroke&taskId=ubd74eeb3-9d44-459b-a045-263b2a5d924&title=&width=1504)

### 3.2.4 新建用户

在ou=people下 New Entity，cn: atlas_c1，uidNumber: 5000024, gidNumber 配置为上步骤新建的5000014，其他配置参考截图。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637672591363-75ba1236-60b2-413e-8b32-59f54726a568.png#clientId=u0acf11c9-046e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=444&id=H36Ev&margin=%5Bobject%20Object%5D&name=image.png&originHeight=444&originWidth=1516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92011&status=done&style=stroke&taskId=u910fc61e-a171-4e8e-9a21-27b0ccb21ce&title=&width=1516)
同理新建cn: atlas_c2, uidNumber: 5000025 和 cn: atlas_c3, uidNumber: 5000026 两个用户。

### 3.2.5 给Group配置memberUid

给cn: atlas_c_group的配置增加新建三个用户的memberUid, 记录如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637673959232-5dd25922-d90b-4fbc-9556-c641f9b4eb19.png#clientId=u0acf11c9-046e-4&crop=0&crop=0.6922&crop=1&crop=1&from=paste&height=366&id=wrGXN&margin=%5Bobject%20Object%5D&name=image.png&originHeight=366&originWidth=1512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77930&status=done&style=stroke&taskId=ud4b875d0-ffa9-471d-8ed6-0f23f96955d&title=&width=1512)

## 2.3 用户生成keytab

> 在机器192.168.90.112上用 root 用户操作。

### 3.3.1 三个用户生成keytab

keytab文件存储到etc/security/keytabs目录下。

```shell
kadmin.local -q "addprinc -pw 123456  atlas_c1"
kadmin.local -q "xst -k /etc/security/keytabs/atlas_c1.keytab atlas_c1@HADOOP.COM"

kadmin.local -q "addprinc -pw 123456  atlas_c2"
kadmin.local -q "xst -k /etc/security/keytabs/atlas_c2.keytab atlas_c2@HADOOP.COM"

kadmin.local -q "addprinc -pw 123456  atlas_c3"
kadmin.local -q "xst -k /etc/security/keytabs/atlas_c3.keytab atlas_c3@HADOOP.COM"
```

生成完成后，选择atlas_c1.keytab、atlas_c2.keytab、atlas_c3.keytab验证下。

> 注: kinit前设置KRB5CCNAME目录用于多个用户同时认证不冲突。

```shell
# atlas_c1用户
export KRB5CCNAME=/tmp/atlas_c1 && kinit -kt /etc/security/keytabs/atlas_c1.keytab atlas_c1@HADOOP.COM

# atlas_c2用户
export KRB5CCNAME=/tmp/atlas_c2 && kinit -kt /etc/security/keytabs/atlas_c2.keytab atlas_c2@HADOOP.COM

# atlas_c3用户
export KRB5CCNAME=/tmp/atlas_c3 && kinit -kt /etc/security/keytabs/atlas_c3.keytab atlas_c3@HADOOP.COM
```

## 3.4 同步用户到Ranger中

新建完成后，在Ranger Admin UI中 Settings->Users/Groups下搜索上面新建的组和用户，检索结果均为空。
这是由于Ranger是默认一小时从LDAP中同步一次用户，由配置项ranger.usersync.sleeptimeinmillisbetweensynccycle控制，默认值3600000。可在Audit->User Sync 页面中查看同步记录。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637850779556-46e97e6e-6994-4b4a-a2f7-4bccbf2e10e3.png#clientId=u72d2e971-9b8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1152&id=ue69fc834&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1152&originWidth=2678&originalType=binary&ratio=1&rotation=0&showTitle=false&size=284096&status=done&style=stroke&taskId=u34a89d5a-5e88-4c3e-b2a9-ba30c7d110e&title=&width=2678)
​

一小时太慢，此值也不建议修改默认值，因此本文选择重启Ranger来同步。重启后推荐看Audit->User Sync中的是否新增有同步记录，如有再去检索验证。
​

同步后，在Settings->Users/Groups的Group下检索 atlas_c_group，结果如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637850999407-03dec224-2232-4964-96db-d1628869d314.png#clientId=u72d2e971-9b8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=592&id=ub5e75a5b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=592&originWidth=2708&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151201&status=done&style=stroke&taskId=ub757523d-f8db-4e52-92c8-083110f6d1f&title=&width=2708)
在Users检索 atlas_c，结果如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637851057287-71b96260-b391-4d46-b398-85f2ec95c123.png#clientId=u72d2e971-9b8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=720&id=u0a8e078c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=720&originWidth=2740&originalType=binary&ratio=1&rotation=0&showTitle=false&size=204423&status=done&style=stroke&taskId=u98c6f74e-a3a0-48ca-9d6b-4adbbcd9666&title=&width=2740)
至此，组和用户已同步到Ranger中。

> 由于是手动在LDAP中新建组和用户，容易出错，会导致组和用户无法同步到Ranger中。针对此种情况建议查看分析用户同步日志。
> 在[Ranger Usersync](http://192.168.90.112:8080/#) 所在机器 192.168.90.128的/var/log/ranger/usersync 目录下查看usersync.log日志。

## 3.5 准备测试库和表

> 在机器192.168.90.112上用 root 用户操作。

### 3.5.1 认证登录

采用hive用户新建库和表用于下面的验证，但是会报下面错误:

```shell
# hive
export KRB5CCNAME=/tmp/hive && kinit -kt /etc/security/keytabs/hive.keytab hive@HADOOP.COM
kinit: Password incorrect while getting initial credentials
```

根据参考资料[Kerberos报错：kinit: Password incorrect while getting initial credentials](https://blog.csdn.net/ZhouyuanLinli/article/details/78540299) 可知，是由于每次生成秘钥文件时密码可能会改变导致。
​

解决办法是生成keytab时采用随机密码，命令如下:

```shell
kadmin.local -q "xst -k /etc/security/keytabs/hive.keytab -norandkey hive@HADOOP.COM"
```

然后再重新认证。
​

### 3.5.2 新建数据库atlas_ranger_db

用beeline进入hive客户端进行操作。

```shell
beeline
```

【可选】在192.168.90.122机器上执行下面命令消费kafka Topic ATLAS_HOOK 中的数据。

```shell
# 用atlas用户认证
export KRB5CCNAME=/tmp/atlas && kinit -kt /etc/security/keytabs/atlas.service.keytab atlas/emr122.dtwave.com@HADOOP.COM

# 消费Topic ATLAS_HOOK观察输出数据
/usr/hdp/3.1.0.0-78/kafka/bin/kafka-console-consumer.sh --bootstrap-server emr122.dtwave.com:9092,emr115.dtwave.com:9092,emr114.dtwave.com:9092  --topic ATLAS_HOOK --consumer.config /usr/hdp/3.1.0.0-78/kafka/config/consumer.properties
```

新建数据库，执行SQL: create database atlas_ranger_db;

```shell
0: jdbc:hive2://emr114.dtwave.com:2181,emr128> create database atlas_ranger_db;
INFO  : Compiling command(queryId=hive_20211125225745_ff9bbc05-bb94-4a4c-bd11-3faab1c3b8b2): create database atlas_ranger_db
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:null, properties:null)
INFO  : Completed compiling command(queryId=hive_20211125225745_ff9bbc05-bb94-4a4c-bd11-3faab1c3b8b2); Time taken: 0.066 seconds
INFO  : Executing command(queryId=hive_20211125225745_ff9bbc05-bb94-4a4c-bd11-3faab1c3b8b2): create database atlas_ranger_db
INFO  : Starting task [Stage-0:DDL] in serial mode
INFO  : Completed executing command(queryId=hive_20211125225745_ff9bbc05-bb94-4a4c-bd11-3faab1c3b8b2); Time taken: 0.228 seconds
INFO  : OK
No rows affected (0.419 seconds)
```

### 3.5.3 Atlas中检索数据库

在hive中进行新建数据库、表等操作后，数据会通过kafka实时采集到Atlas中。因此在Atlas DashBoard中检索上述新建的数据库atlas_ranger_db，结果如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637852681826-b3030fb9-106e-4bf5-b943-68e4d7a9533b.png#clientId=u9fa33a77-4a9f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1022&id=u3c192975&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1022&originWidth=2770&originalType=binary&ratio=1&rotation=0&showTitle=false&size=471407&status=done&style=stroke&taskId=ue6e6472f-5b77-4f13-a4c9-72092b84253&title=&width=2770)

### 3.5.4 【可选】查看Topic ATLAS_HOOK中的数据

在控制台上会输出如下内容:

```json
{
    "version":{
        "version":"1.0.0",
        "versionParts":[
            1
        ]
    },
    "msgCompressionKind":"NONE",
    "msgSplitIdx":1,
    "msgSplitCount":1,
    "msgSourceIP":"192.168.90.115",
    "msgCreatedBy":"hive",
    "msgCreationTime":1637852423252,
    "message":{
        "type":"ENTITY_CREATE_V2",
        "user":"hive",
        "entities":{
            "referredEntities":{

            },
            "entities":[
                {
                    "typeName":"hive_db",
                    "attributes":{
                        "owner":"hive",
                        "ownerType":"USER",
                        "qualifiedName":"atlas_ranger_db@emr_dev",
                        "clusterName":"emr_dev",
                        "name":"atlas_ranger_db",
                        "description":null,
                        "location":"hdfs://mycluster/warehouse/tablespace/managed/hive/atlas_ranger_db.db",
                        "parameters":{

                        }
                    },
                    "guid":"-19348701935700",
                    "version":0,
                    "proxy":false
                }
            ]
        }
    }
}
```

### 3.5.5 新建数据表demo_user

新建数据表，执行下述SQL。

```sql
use atlas_ranger_db;

CREATE TABLE demo_user( user_id BIGINT COMMENT '用户ID' , user_name STRING COMMENT '姓名' , age BIGINT COMMENT '年龄' , id_card STRING COMMENT '身份证' , salary BIGINT COMMENT '薪水' , email STRING COMMENT '邮箱' , address STRING COMMENT '地址') COMMENT '用户表';

INSERT INTO demo_user VALUES( 1 , '张林静' , 23 , '330110198901013412' , 4500 , 'linjing.qlj@shuqi.com' , '杭州市金域湖庭小区') ,( 2 , '王勇强' , 54 , '610122199003054324' , 6500 , 'yongqiang.wyq@shuqi.com' , '杭州市万科锦程小区') ,( 3 , '李志刚' , 56 , '411033199112015423' , 6700 , 'zhigang.lzg@shuqi.com' , '杭州市金色城市小区') ,( 4 , '赵晓丽' , 23, '510230199209235123', 1200, 'xiaoli.zxl@shuqi.com', '杭州市万科红郡小区' ), ( 5, '杜孟娟', 67, '566092199308070904', 9000, 'mengjuan.dmj@shuqi.com', '杭州市翡翠国际小区' );
```

### 3.5.6 Atlas中检索表

在Atlas DashBoard中检索上述新建的表atlas_ranger_db.demo_user，结果如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638174580906-72e60233-785c-4243-bc76-865aec5e1965.png#clientId=ue2cccb16-fc26-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=451&id=ucdb48790&margin=%5Bobject%20Object%5D&name=image.png&originHeight=902&originWidth=2762&originalType=binary&ratio=1&rotation=0&showTitle=false&size=527370&status=done&style=stroke&taskId=u278df7a9-a630-4892-8832-70e73572ce5&title=&width=1381)
点击「demo_user」查看表的详情如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638174690571-e3446ec0-b6ff-480b-8a13-ba47885f8834.png#clientId=ue2cccb16-fc26-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=619&id=u3b494243&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1238&originWidth=2054&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151940&status=done&style=stroke&taskId=u8ab7d710-3c91-4ccb-a123-d9164b713d3&title=&width=1027)

# 四、测试Classification结合Ranger Tag

## 4.1 新建Classification(Tag)

在「Atlas DashBoard」->「CLASSFICATION」下新建PID classfication，描述: "Private Identifiable Data"。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638175341153-345a9737-a5d7-4cd4-a447-d169228fcec0.png#clientId=ue2cccb16-fc26-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=438&id=uc007e3b6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=876&originWidth=1196&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87497&status=done&style=stroke&taskId=u64c6e900-bd10-4824-91d7-d181adf1347&title=&width=598)

## 4.2 给字段salary配置Classification

给atlas_ranger_db.demo_user表的salary字段配置PID classfication。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638175544272-2afafecf-9fc7-4164-aacc-1a2af2532305.png#clientId=ue2cccb16-fc26-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=263&id=u4b8efc8e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=526&originWidth=1790&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42977&status=done&style=stroke&taskId=u91b0cb4c-b7cd-4cc8-b4ac-0ff2ffc583d&title=&width=895)
配置完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638260321358-15227c38-9eb4-4883-93d2-6c0d5e77497a.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=761&id=ud476f9dd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1522&originWidth=2048&originalType=binary&ratio=1&rotation=0&showTitle=false&size=391159&status=done&style=stroke&taskId=ubbd3047c-4855-445c-993e-4d6d31e6992&title=&width=1024)

## 4.3 新建Tag Based Access Policy

在Ranger 「AccessManager」->「Tag Base Policies」的[hivetag](http://emr128.dtwave.com:6080/index.html#!/service/24/policies/0) 下新建Access policy。

- Policy Name: PID_Policy
- TAG: PID

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638179005287-6108047b-5eca-4aa6-86ce-c2913a92cd90.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=450&id=u26f3d6a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=900&originWidth=2664&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117103&status=done&style=stroke&taskId=u46dc9bd3-a14f-4504-b4e5-d75a5fd8b3e&title=&width=1332)

- Allow conditions:
    - User: atlas_c1
    - Component Permissions: select, update for hive component

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638178800341-5608a00a-a3e0-4f47-92ec-8a7b8212a12c.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=240&id=u34b38f8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=480&originWidth=1406&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55685&status=done&style=stroke&taskId=u2b23ebfb-58d6-4fd6-9ce7-d8d882fa66f&title=&width=703)
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638178975059-3f53ec43-0076-443a-96a3-24c19616123a.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=441&id=u08f64fd5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=882&originWidth=2562&originalType=binary&ratio=1&rotation=0&showTitle=false&size=135575&status=done&style=stroke&taskId=u61a80651-28c7-48f3-89d4-90a90dff1ed&title=&width=1281)

- Deny conditions:
    - User: atlas_c2
    - Component Permissions: all for hive component

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638179037549-c5a98d94-a851-4a40-8e09-04de593fde18.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=242&id=u24b2a1d5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=1404&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56251&status=done&style=stroke&taskId=uf5c5d3a9-422d-4f87-80bd-aa3fe0d4d2e&title=&width=702)
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638179049615-58f2da6d-8f1e-4058-bc24-bebcd5dcbd00.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=436&id=ua2a025d6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=872&originWidth=2558&originalType=binary&ratio=1&rotation=0&showTitle=false&size=135381&status=done&style=stroke&taskId=u379439bf-4f2c-4962-85a9-997a4a500bf&title=&width=1279)
上述新建完成后，PID_Policy如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638179184786-21a6e465-81f6-413e-bd91-59e86eaa34b6.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=425&id=u3ae6c9a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=850&originWidth=2730&originalType=binary&ratio=1&rotation=0&showTitle=false&size=570950&status=done&style=stroke&taskId=u4a7c8467-0669-439f-8e47-f9f8744ebf6&title=&width=1365)
注意: **一个Classfication只能用于一个Access Policy中**，例如下图新建一个配置和PID_Policy一样的PID_Policy_2(只是Policy Name不一样)会报如下的错误:

```shell
Error Code : 3010 Another policy already exists for matching resource: policy-name=[PID_Policy], service=[hivetag]
```

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638274963649-d2536174-37df-441d-879e-462081b4d6b1.png#clientId=u43964b9d-4fd6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=476&id=u336ca88e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=952&originWidth=1883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=140026&status=done&style=stroke&taskId=u069a113d-7671-4a09-a6ed-f2b45ae2825&title=&width=941.5)

## 4.4 测试表的Tag Access Policy

上述给atlas_ranger_db.demo_user的salary设置PID Classfication，根据[Apache Ranger Policy Evaluation Flow with Tags](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.1/authorization-ranger/content/tags_and_policy_evaluation.html)可知，atlas_c1用户此时只能访问salary字段，而atlas_c2用户则任何字段都不能访问。测试结果如下图:

1. atlas_c1用户访问

  ```shell
  # atlas_c1用户认证
  export KRB5CCNAME=/tmp/atlas_c1 && kinit -kt /etc/security/keytabs/atlas_c1.keytab atlas_c1@HADOOP.COM
  ```


# 进入hive客户端

beeline

# 执行SQL

SELECT user_id, user_name, age, id_card, salary, email, address FROM atlas_ranger_db.demo_user;
SELECT salary FROM atlas_ranger_db.demo_user;
SELECT user_id FROM atlas_ranger_db.demo_user;

````
![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638271706327-9b1e3813-ebba-4d7c-9216-0e5944dc00ee.png#clientId=u43964b9d-4fd6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=478&id=ud9b7f3bb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=956&originWidth=2212&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1620193&status=done&style=none&taskId=u7ed0602c-77b7-4ec5-9d7b-2b4f888c763&title=&width=1106)

2. atlas_c2用户访问
```shell
# atlas_c2用户认证
export KRB5CCNAME=/tmp/atlas_c2 && kinit -kt /etc/security/keytabs/atlas_c2.keytab atlas_c1@HADOOP.COM

# 进入hive客户端
beeline

# 执行SQL
SELECT user_id, user_name, age, id_card, salary, email, address FROM atlas_ranger_db.demo_user;
SELECT salary FROM atlas_ranger_db.demo_user;
SELECT user_id FROM atlas_ranger_db.demo_user;
````

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638271774690-7a97ad47-99e9-4496-a97e-40538113e061.png#clientId=u43964b9d-4fd6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=271&id=u64b66e3d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=542&originWidth=2216&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1165509&status=done&style=none&taskId=u9974ffa1-f4d1-4b73-9305-203a1e560fe&title=&width=1108)

## 4.5 新建 Resource Based Access Policy

在Ranger 「AccessManager」->「Resource Base Policies」->「Hive」的[emr_dev_hive](http://emr128.dtwave.com:6080/index.html#!/service/4/policies/0)下新建 Access policy。

- Policy Name: atlas_ranger_db_demo_user_policy
- Database： atlas_ranger_db
- Table: demo_user
- Hive Column: *
- Allow Conditions:
    - user: atlas_c1,atlas_c2
    - Permissions: select,update

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638179655223-d05ada13-a3d5-4023-a0b7-2f5cd86a1bf1.png#clientId=u266dd5e4-b8ef-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=796&id=u22b11c2c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1592&originWidth=2698&originalType=binary&ratio=1&rotation=0&showTitle=false&size=284856&status=done&style=stroke&taskId=uccb645da-03cf-4ca7-971d-e94e7d3b0bc&title=&width=1349)

## 4.6 混合测试Tag Access Policy和Resource Access Policy

1. 用户atlas_c1访问表

  ```shell
  # atlas_c1用户认证
  export KRB5CCNAME=/tmp/atlas_c1 && kinit -kt /etc/security/keytabs/atlas_c1.keytab atlas_c1@HADOOP.COM
  ```


# 进入hive客户端

beeline

# 执行SQL

SELECT user_id, user_name, age, id_card, salary, email, address FROM atlas_ranger_db.demo_user;

````
结果如下，能正常访问atlas_ranger_db.demo_user表的所有字段。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638261582747-32d8a263-0f0b-4c2f-b29a-f8bfd7e0f5c1.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=394&id=u4a7d01e3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=788&originWidth=2216&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1729954&status=done&style=none&taskId=uf3b5a534-9b81-4271-9c73-fcb9080fdb1&title=&width=1108)

2. 用户atlas_c2访问表
```shell
# atlas_c2用户认证
export KRB5CCNAME=/tmp/atlas_c2 && kinit -kt /etc/security/keytabs/atlas_c2.keytab atlas_c2@HADOOP.COM

# 进入hive客户端
beeline

# 执行SQL
SELECT user_id, user_name, age, id_card, salary, email, address FROM atlas_ranger_db.demo_user;
````

会报如下错误:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638261832442-14b81307-d501-41fa-bcbb-fa261f0e975e.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=88&id=uf2158aaf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=176&originWidth=2220&originalType=binary&ratio=1&rotation=0&showTitle=false&size=421486&status=done&style=none&taskId=u8f097bd0-aee2-4166-9601-e039b8f1e74&title=&width=1110)
因为atlas_c2用户没有salary字段的select权限，因此去掉salary字段再次执行。

```shell
SELECT user_id, user_name, age, id_card, email, address FROM atlas_ranger_db.demo_user;
```

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638261825772-49f1fe1c-84f0-4c3c-ae76-5c10e918266b.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=373&id=u36828921&margin=%5Bobject%20Object%5D&name=image.png&originHeight=746&originWidth=2212&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1613980&status=done&style=none&taskId=u4a09f301-4de1-49b9-be38-e59e73b7175&title=&width=1106)

# 五、测试Tag Based Masking Policy

## 5.1 字段id_card配置PID

给atlas_ranger_db.demo_user表的id_card字段配置PID。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281680333-5dde62ab-4fbf-475b-a966-fc037dc9abaf.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=262&id=u1f093abe&margin=%5Bobject%20Object%5D&name=image.png&originHeight=524&originWidth=1794&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43902&status=done&style=stroke&taskId=ud2efe2b7-8b15-490d-b95b-66634955e26&title=&width=897)
配置完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638278169349-7210d374-fb0e-48af-9b7d-535fac6f860c.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=767&id=udbf0ffd0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1534&originWidth=2034&originalType=binary&ratio=1&rotation=0&showTitle=false&size=412168&status=done&style=stroke&taskId=u8e0cb67e-e875-4ec1-8b95-5bcb6fafb1b&title=&width=1017)

## 5.2 新建Tag Based Masking Policy

在Ranger 「AccessManager」->「Tag Base Policies」的[hivetag](http://emr128.dtwave.com:6080/index.html#!/service/24/policies/0) 下新建Masking policy。

- Policy Name: PID_Masking_Policy
- TAG: PID
- Allow conditions:
    - User: atlas_c1
    - Access Type: select for hive component
    - Select Masking Option: Partial mast: show first 4

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638280137577-5c216dd6-d413-4006-8d8b-6c93195cf6df.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=622&id=u2c4f912e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1244&originWidth=2668&originalType=binary&ratio=1&rotation=0&showTitle=false&size=269827&status=done&style=stroke&taskId=ue243bc18-61d3-47e8-8d7d-248ecc86b13&title=&width=1334)

注意: **一个Classfication只能用于一个Masking Policy中。**

## 5.3 测试表的Tag Masking Policy

用户atlas_c1访问表atlas_ranger_db.demo_user的user_id、id_card、salary字段。

```shell
# 执行SQL
SELECT user_id, id_card, salary  FROM atlas_ranger_db.demo_user;
```

结果显示身份证mask策略已生效，只显示前4位。salary由于是BIGINT类型，对Partial策略不生效。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281057648-5bc8dd9e-4cbe-4bca-9ff7-4b4f1e07798e.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=372&id=u873be02a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=744&originWidth=2214&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1194096&status=done&style=none&taskId=u56f23315-5090-4e12-975b-1b47d071215&title=&width=1107)
另外，由于此时id_card字段已绑定PID策略，用户atlas_c2也无权限访问id_card字段。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281117387-8e6f5ab9-09eb-4aa6-9b98-8033af4fced7.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=86&id=u35a42744&margin=%5Bobject%20Object%5D&name=image.png&originHeight=172&originWidth=2196&originalType=binary&ratio=1&rotation=0&showTitle=false&size=65702&status=done&style=none&taskId=ue4074138-b0e1-4043-8645-5a96d7c59f3&title=&width=1098)

# 六、测试Classification的血缘传递

## 6.1 新建demo_complaint和demo_user_complaint表

采用hive用户新建数据表，执行下述SQL。

```sql
use atlas_ranger_db;

# 新建投诉表 demo_complaint
CREATE TABLE demo_complaint( user_id BIGINT COMMENT '用户ID' , content STRING COMMENT '投诉内容' , complaint_time STRING COMMENT '投诉时间') COMMENT '投诉表';

# 给投诉表中插入数据
INSERT INTO demo_complaint(user_id , content , complaint_time) VALUES( 5 , '你好,我家厨房的地漏堵着了' , '2017-07-01') ,( 5 , '客服人员,卧室门吸吸不住,很影响使用' , '2017-03-23') ,( 4 , '厨房窗户关不上' , '2017-06-12') ,( 4 , '卫生间水管渗水,房子都不能住了' , '2017-07-09') ,( 3 , '我要投诉,我家的进户门门框开裂严重,很长时间了还不来维修' , '2017-08-01') ,( 3 , '投诉好久了我家的主卧地板发黑,现在都没人管!' , '2017-05-12') ,( 2 , '客服你好,厨房台面有点开裂' , '2017-07-25') ,( 2 , '卫生间马桶下水慢,请快速解决' , '2017-07-03') ,( 1 , '我家洗手间的门槛石缺损了,请尽快维修' , '2017-07-23') ,( 1 , '浴室门上都是划痕' , '2017-07-04') ,( 1 , '卫生间门把手松动,门都发不开,快点来维修' , '2017-06-23') ,( 1 , '你好,卧室门有污渍,清理不掉' , '2017-04-15');

# 创建用户-投诉表 demo_user_complaint
CREATE TABLE demo_user_complaint(user_id BIGINT COMMENT '用户ID' , user_name STRING COMMENT '姓名' , age BIGINT COMMENT '年龄' , id_card STRING COMMENT '身份证' , salary BIGINT COMMENT '薪水' , email STRING COMMENT '邮箱' , address STRING COMMENT '地址' , content STRING COMMENT '投诉内容' , complaint_time STRING COMMENT '投诉时间') COMMENT '用户-投诉表';
```

在Atlas DashBoard中查看数据库atlas_ranger_db下面表。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281491734-8be66e9a-8e59-4eae-918f-89c2ea39b259.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=584&id=u205a3730&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1168&originWidth=2072&originalType=binary&ratio=1&rotation=0&showTitle=false&size=190337&status=done&style=none&taskId=u465baa2b-9d6f-4431-9954-f709f6ccd57&title=&width=1036)

## 6.2 双表Join生成血缘

采用hive用户新建数据表，执行下述SQL。

```sql
INSERT overwrite TABLE demo_user_complaint SELECT a.user_id, a.user_name, a.age, a.id_card, a.salary, a.email, a.address, b.content, b.complaint_time FROM demo_user a JOIN demo_complaint b ON a.user_id = b.user_id;
```

在Atlas DashBoard中查看表demo_user_complaint的Lineage，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281606415-02de244e-79f5-4087-932e-c240b49872f4.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=700&id=uad67c8b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1400&originWidth=2052&originalType=binary&ratio=1&rotation=0&showTitle=false&size=230379&status=done&style=none&taskId=u5346a7e4-1592-44af-9a5c-9f59ece2cf9&title=&width=1026)

## 6.3 id_card配置PID并勾选Propagate

由于Atlas不支持字段上已配置Classification的属性修改，因此删除id_card字段上已有的PID。然后重新添加Classification，并勾选Propagate和Remove Propagationg on entity delete 选项。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638281988952-a9e5bd4b-9ff1-4675-a9b5-8cec808db808.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=297&id=u266eba69&margin=%5Bobject%20Object%5D&name=image.png&originHeight=594&originWidth=1792&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54936&status=done&style=stroke&taskId=u22d13bff-3f0e-44a7-a3bb-c0239ec5571&title=&width=896)

## 6.4 Atlas查看血缘传递

打开demo_user_complaint表，发现id_card字段已通过血缘传递配置有PID Classification，但注意此Classification不能直接在demo_user_complaint表上手动删除。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638282122787-abb26f9c-6708-4f91-94b6-18904977216f.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=744&id=ud1f1eb0c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1488&originWidth=2048&originalType=binary&ratio=1&rotation=0&showTitle=false&size=427177&status=done&style=stroke&taskId=u97025d42-4fd8-4927-8f81-b20f2847d20&title=&width=1024)
在Atlas CLASSIFICATION页查看PID已关联的hive_column和[hive_column_lineage](http://emr122.dtwave.com:21000/index.html#!/search/searchResult?query=hive_column_lineage%20&searchType=dsl&dslChecked=true) 信息，如下。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638282252422-b326af2e-c04c-4523-8e1a-03761d9c54e1.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=618&id=u12b3cdf1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1236&originWidth=2066&originalType=binary&ratio=1&rotation=0&showTitle=false&size=176187&status=done&style=stroke&taskId=u365fdf45-b775-4423-a192-4a1adf40040&title=&width=1033)

## 6.5 测试表的血缘传递

用户atlas_c1访问表atlas_ranger_db.demo_user_complaint的id_card字段。

```shell
# 执行SQL
SELECT id_card FROM atlas_ranger_db.demo_user_complaint;
```

结果显示身份证mask策略已生效，只显示前4位。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638282467147-1c1c1870-a5d8-4b3c-a1d0-9958e73531cd.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=475&id=u5f12c80a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=950&originWidth=2208&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1163716&status=done&style=stroke&taskId=u8776e8f0-f969-42a4-9547-24799b20c2e&title=&width=1104)

# 七、测试Glossary(Category和Terms)

## 7.1 新建Category

在「Atlas DashBoard」->「GLOSSARY」下新建Category，Name是"智慧物业"。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283326958-d5c6c03d-b715-43d8-bbfd-1335332f139d.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=344&id=u2e1b2dae&margin=%5Bobject%20Object%5D&name=image.png&originHeight=688&originWidth=1196&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58682&status=done&style=stroke&taskId=u7812ea50-dc9c-48f8-b4eb-2c53b3e2c0c&title=&width=598)
然后在"智慧物业"下再点击“+Create Catagory”，Name时“用户基本信息”。新建完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283432813-bd3d4925-6d90-434d-875e-b865b6682923.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=383&id=u0033674f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=766&originWidth=2226&originalType=binary&ratio=1&rotation=0&showTitle=false&size=267203&status=done&style=stroke&taskId=u0ce33f41-882b-41da-87c7-aba03b985a5&title=&width=1113)

## 7.2 新建Term

切换到Terms视图，在“智慧物业” Category下点击“Create Term”。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283511730-938739ae-dda0-4298-b8ab-2ac5531c5531.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=365&id=ua2fe4359&margin=%5Bobject%20Object%5D&name=image.png&originHeight=730&originWidth=2074&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130907&status=done&style=none&taskId=u9a73e21a-c5de-4f5d-91ea-9fe514b7632&title=&width=1037)
Name是"邮箱"，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283001589-ab3bd862-d7fd-4f12-ab83-166c4141824b.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=343&id=u04459fc2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=686&originWidth=1192&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54934&status=done&style=stroke&taskId=u3c10c008-e783-46b0-b326-e21c10701e9&title=&width=596)
新建完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283094718-7efcdc5c-0081-4dc9-b849-6c58e36f4e98.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=538&id=ub0ebeac6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1076&originWidth=2774&originalType=binary&ratio=1&rotation=0&showTitle=false&size=208817&status=done&style=stroke&taskId=ubd07c85e-f5e2-401e-b815-ffe097b99b5&title=&width=1387)
然后给邮箱绑定“用户基本信息”的Catagory。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638283607505-9f6ae72b-92eb-4032-b10b-03e2894cb885.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=484&id=u44de8a6c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=968&originWidth=2058&originalType=binary&ratio=1&rotation=0&showTitle=false&size=122529&status=done&style=stroke&taskId=uddff46a2-3178-4889-8453-f2bdb5e3930&title=&width=1029)

## 7.3 新建Classification

在「Atlas DashBoard」->「CLASSFICATION」下新建Email classfication，Name是“Email”。

## 7.4 Term配置Email Classification

给Term“邮箱”添加Classification添加上面新建的Email Tag，并勾选Propagate和Remove Propagationg on entity delete 选项。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638284053018-f17078a4-5e51-4a3d-bfdd-f5610957615a.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=488&id=u33f3ba36&margin=%5Bobject%20Object%5D&name=image.png&originHeight=976&originWidth=2072&originalType=binary&ratio=1&rotation=0&showTitle=false&size=125551&status=done&style=stroke&taskId=u2dbd93d8-1e5f-47fd-b146-acea69493e0&title=&width=1036)

## 7.5 表字段配置Term

给demo_user表的email字段关联邮箱”Term“，结果如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638284334763-5fa61cb8-5e31-418a-b5ea-ac12aa3468fa.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=445&id=u9d079e40&margin=%5Bobject%20Object%5D&name=image.png&originHeight=890&originWidth=2086&originalType=binary&ratio=1&rotation=0&showTitle=false&size=362810&status=done&style=stroke&taskId=u66d34a47-cd4c-4198-accc-291225604fa&title=&width=1043)

## 7.6 新建Tag Based Policy

在Ranger 「AccessManager」->「Tag Base Policies」的[hivetag](http://emr128.dtwave.com:6080/index.html#!/service/24/policies/0) 下新建Masking policy。

- Policy Name: Email_Masking_Policy
- TAG: Email
- Allow conditions:
    - User: atlas_c3
    - Access Type: select for hive component
    - Select Masking Option: Hash

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638284587874-8e6be96a-6e90-439d-9fde-d6172515eb24.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=767&id=u6644c388&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1534&originWidth=2706&originalType=binary&ratio=1&rotation=0&showTitle=false&size=287535&status=done&style=stroke&taskId=ucc358abb-e076-4a89-871b-691c819c176&title=&width=1353)
上述只有Masking策略，atlas_c3还是无权限访问demo_user表的email字段的。
​

此时再在hivetag下新建Access Policy:

- Policy Name: Email_Policy
- TAG: Email
- Allow conditions:
    - User: atlas_c3
    - Component Permissions: select for hive component

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638284707304-1b8a460c-67de-405c-801a-850506e2b1ac.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=716&id=u48add2c5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1432&originWidth=2716&originalType=binary&ratio=1&rotation=0&showTitle=false&size=253400&status=done&style=stroke&taskId=u0d774c2d-ef26-4f3c-baf2-fcf06784e63&title=&width=1358)

## 7.7 测试表的policy

用户atlas_c3访问表atlas_ranger_db.demo_user的email字段。

```shell
# 执行SQL
select email from atlas_ranger_db.demo_user;
```

结果显示策略已生效，email的值已被Hash掉。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638284980373-3f9bd81d-d8d9-47f3-9081-426dd31c6c65.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=325&id=u428c10a9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=650&originWidth=2128&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1253596&status=done&style=none&taskId=ub0a7183d-6ccb-4d8b-af52-237805c3ccc&title=&width=1064)
由于血缘传递关系，demo_user_complaint表的email字段也会被绑定Email Classification，但是不会传递绑定“邮箱”Term。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638285352882-98601602-52f0-4d17-a711-adb3eab7f972.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=745&id=uc62c36f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1490&originWidth=2050&originalType=binary&ratio=1&rotation=0&showTitle=false&size=316548&status=done&style=stroke&taskId=ueaca067a-29b1-454a-8b6a-807ed490a00&title=&width=1025)
此时用atlas_c3访问表atlas_ranger_db.demo_user_complaint的email字段。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638285449468-e2535de6-f99f-411d-a46d-974341a61088.png#clientId=u89760ba5-82ee-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=447&id=uf1f83ed7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=894&originWidth=2210&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1697235&status=done&style=none&taskId=udfa52c83-6b17-4414-82df-ae5529254c1&title=&width=1105)

# 八、测试Sub-classification

测试目的: 测试拥有父classification的用户(组)能否访问子classification的entity。

## 8.1 新建Classification

在「Atlas DashBoard」->「CLASSFICATION」下按照下图的层次关系新建classification。

- Data_Classification
    - TopSecret
        - Confidential
            - Mimi_Tag
                - Gongkai_Tag

                  > 注: 测试过程中用Secret、Open中出现策略不生效问题，应该是跟系统关键字有冲突。因此秘密、公开采用拼音的形式。


![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353270976-c26777e3-e9bb-44af-9144-3dd2a3db635d.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=378&id=u4af494a2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=756&originWidth=1668&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58300&status=done&style=none&taskId=u1c9f7d45-de91-431f-a2f5-2c3d4857ce2&title=&width=834)
新建完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353383593-525c35ea-7fe5-4e39-b94f-9d9342a847ad.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=379&id=u3f55fff9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=758&originWidth=676&originalType=binary&ratio=1&rotation=0&showTitle=false&size=160332&status=done&style=none&taskId=ua225f28b-de9a-44dd-839e-f4358a22b88&title=&width=338)

## 8.2 新建数据表demo_user_classification

新建数据表，用hive用户执行下述SQL。

```sql
use atlas_ranger_db;

CREATE TABLE demo_user_classification( user_id BIGINT COMMENT '用户ID' , user_name STRING COMMENT '姓名' , age BIGINT COMMENT '年龄' , id_card STRING COMMENT '身份证' , salary BIGINT COMMENT '薪水' , email STRING COMMENT '邮箱' , address STRING COMMENT '地址') COMMENT '用户表';

INSERT INTO demo_user_classification VALUES( 1 , '张林静' , 23 , '330110198901013412' , 4500 , 'linjing.qlj@shuqi.com' , '杭州市金域湖庭小区') ,( 2 , '王勇强' , 54 , '610122199003054324' , 6500 , 'yongqiang.wyq@shuqi.com' , '杭州市万科锦程小区') ,( 3 , '李志刚' , 56 , '411033199112015423' , 6700 , 'zhigang.lzg@shuqi.com' , '杭州市金色城市小区') ,( 4 , '赵晓丽' , 23, '510230199209235123', 1200, 'xiaoli.zxl@shuqi.com', '杭州市万科红郡小区' ), ( 5, '杜孟娟', 67, '566092199308070904', 9000, 'mengjuan.dmj@shuqi.com', '杭州市翡翠国际小区' );
```

## 8.3 表字段配置Classsification

按照下表给字段绑定Classification，并勾选Propagate和Remove Propagationg on entity delete 选项。

| 表名  | 字段  | 绑定的classification |
| --- | --- | --- |
| demo_user_classification | id_card | TopSecret |
| demo_user_classification | salary | Confidential |
| demo_user_classification | email | Mimi_Tag |
| demo_user_classification | user_name | Gongkai_Tag |

配置完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353448932-7ad31608-af13-427d-8705-546811ed5c56.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=721&id=u3a34b627&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1442&originWidth=2044&originalType=binary&ratio=1&rotation=0&showTitle=false&size=205180&status=done&style=none&taskId=u44c2b920-6b37-4788-82ef-16bcfa49e75&title=&width=1022)
在CLASSIFICATION页面查看如下:
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353548574-6fb3f69e-b7b9-4826-999b-6ca5b0d7389e.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=664&id=ufa05e34b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1328&originWidth=2752&originalType=binary&ratio=1&rotation=0&showTitle=false&size=625484&status=done&style=none&taskId=u7834e6db-1ae1-436a-b678-23d7b19824e&title=&width=1376)
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353576059-0059b840-e3d5-44a4-bac8-4c4cf9239163.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=566&id=u765dfaad&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1132&originWidth=2744&originalType=binary&ratio=1&rotation=0&showTitle=false&size=514997&status=done&style=none&taskId=u031f707e-8260-4877-97d7-3f42455cc76&title=&width=1372)

## 8.4 新建Tag Based Access Policy

在Ranger 「AccessManager」->「Tag Base Policies」的[hivetag](http://emr128.dtwave.com:6080/index.html#!/service/24/policies/0) 下，按照下表新建Access policy。

| Policy Name | TAG | Allow conditions |     |
| --- | --- | --- | --- |
|     |     | User | Component Permissions |
| TopSecret_Policy | TopSecret | atlas_c1 | select for hive component |
| Confidential_Policy | Confidential | atlas_c1 | select for hive component |
| Mimi_Tag_Policy | Mimi_Tag | atlas_c2 | select for hive component |
| Gongkai_Tag_Policy | Gongkai_Tag | atlas_c3 | select for hive component |

注: 一个Tag Policy下只能选择一个Classification(Tag)，因此虽然atlas_c1用户能同时访问绝密TopSecret和机密Confidential数据，也需要给每个Tag新建Policy。
新建完成后，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353868279-1b755f7a-05d2-4bcd-b7fb-3567c71c2e39.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=558&id=ufd254b60&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1116&originWidth=2746&originalType=binary&ratio=1&rotation=0&showTitle=false&size=823493&status=done&style=none&taskId=u157dfe12-dc1a-46a9-8793-44504230509&title=&width=1373)

## 8.5 测试Tag Access Policy

1. 用户atlas_c1访问表

  ```shell
  # 查询id_card和salary字段
  SELECT id_card, salary FROM atlas_ranger_db.demo_user_classification;
  ```


# 查询email字段，sub_classification Secret对应的字段

SELECT email FROM atlas_ranger_db.demo_user_classification;

# 查询user_name字段, sub_sub_classification Open对应的字段

SELECT user_name FROM atlas_ranger_db.demo_user_classification;

````
![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638339441879-d11933a6-5b82-4421-a868-37b67fea044d.png#clientId=u737d33ef-a8e3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=676&id=u72ee98c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1352&originWidth=2314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2167462&status=done&style=none&taskId=uaaf6db49-f3b0-427b-bdc7-6774e5d592f&title=&width=1157)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/499433/1638339466274-c41b1566-c0fb-48e4-a6d0-fd2984cfcb14.png#clientId=u737d33ef-a8e3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=358&id=uf737d2c9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=716&originWidth=2314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=969658&status=done&style=none&taskId=u45b09e3b-e614-4f4b-9b81-da217823a64&title=&width=1157)

2. 用户atlas_c2访问表
```shell
# 查询email字段
SELECT email FROM atlas_ranger_db.demo_user_classification;

# 查询user_name字段, sub_classification Open对应的字段
SELECT user_name FROM atlas_ranger_db.demo_user_classification;
````

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353645370-c95670fc-2120-4de1-a518-8f7f428d1a87.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=374&id=u912ab94c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=748&originWidth=2300&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1160265&status=done&style=none&taskId=u94c86abf-6985-4aab-8d37-39623becfa1&title=&width=1150)

3. 用户atlas_c3访问表

  ```shell
  # 查询user_name字段
  SELECT user_name FROM atlas_ranger_db.demo_user_classification;
  ```

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353680108-2c68e8d4-dcfa-4d49-bca2-c15cc939f6f7.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=356&id=u78ae7a3a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=712&originWidth=2282&originalType=binary&ratio=1&rotation=0&showTitle=false&size=970170&status=done&style=none&taskId=u74b1ff0c-e02b-4167-ac23-49481938d7f&title=&width=1141)
如果用atlas_c3访问父Classification对应的email字段，则会报无权限。

  ```shell
  # 查询email字段
  SELECT email FROM atlas_ranger_db.demo_user_classification;
  ```

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638353775681-f6d06ef0-956f-4aa1-810f-83db1de03256.png#clientId=ufcf1b8ff-e9b1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=54&id=u76a93a15&margin=%5Bobject%20Object%5D&name=image.png&originHeight=108&originWidth=2316&originalType=binary&ratio=1&rotation=0&showTitle=false&size=271414&status=done&style=none&taskId=ubf7ddc1c-b104-410d-80f3-a6ade383b4c&title=&width=1158)

## 8.6 测试结论

测试拥有父classification的用户(组)可以访问子classification的entity，跟官网文档[Atlas Authorization Model](https://atlas.apache.org/1.2.0/Atlas-Authorization-Model.html)中的结论一致。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638328047772-da9347d3-7402-4ffc-96fc-c3600d692223.png#clientId=u737d33ef-a8e3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=395&id=u14083557&margin=%5Bobject%20Object%5D&name=image.png&originHeight=790&originWidth=2322&originalType=binary&ratio=1&rotation=0&showTitle=false&size=388608&status=done&style=stroke&taskId=u7073ad3f-72d8-411b-b3d5-273394a6ef1&title=&width=1161)

# 九、实战笔记

## 9.1 Kafka常用命令

命令中的文件producer.properties和consumer.properties的配置请参考[kerberos认证下kafka报错Bootstrap broker host:ip disconnected](https://blog.csdn.net/github_39319229/article/details/114777296)
​

- 查看topic列表命令

  ```shell
  /usr/hdp/3.1.0.0-78/kafka/bin/kafka-topics.sh --zookeeper emr114.dtwave.com,emr128.dtwave.com,emr112.dtwave.com/kafka  --list
  ```

- 查看指定topic信息

  ```shell
  /usr/hdp/3.1.0.0-78/kafka/bin/kafka-topics.sh --zookeeper emr112.dtwave.com:2181,emr114.dtwave.com:2181,emr128.dtwave.com:2181/kafka --describe --topic test 
  ```

- 在console写入数据

  ```shell
  /usr/hdp/3.1.0.0-78/kafka/bin/kafka-console-producer.sh --broker-list emr122.dtwave.com:9092,emr115.dtwave.com:9092,emr114.dtwave.com:9092  --topic test  --producer.config /usr/hdp/3.1.0.0-78/kafka/config/producer.properties
  ```

- 在console消费数据

  ```shell
  /usr/hdp/3.1.0.0-78/kafka/bin/kafka-console-consumer.sh --bootstrap-server emr122.dtwave.com:9092,emr115.dtwave.com:9092,emr114.dtwave.com:9092  --topic test --consumer.config /usr/hdp/3.1.0.0-78/kafka/config/consumer.properties --from-beginning 
  ```

  ## 9.2 Kerberos常用命令

- 查看principle

  ```shell
  klist -ket /etc/security/keytabs/hdfs.keytab 
  ```

- 清空缓存

  ```shell
  kdestroy 
  ```

  ## 9.3 排查指南-Atlas中未查询到Hive中新建的库或表

  **问题**: 在Hive中新建数据库或者表后，但在Atlas却找搜索不到。
  ​


**排查步骤**:

1. 建议排查Kafka的ATLAS_HOOK Topic是否有数据，通过下面命令观察。

  ```shell
  # 消费Topic ATLAS_HOOK观察输出数据
  /usr/hdp/3.1.0.0-78/kafka/bin/kafka-console-consumer.sh --bootstrap-server emr122.dtwave.com:9092,emr115.dtwave.com:9092,emr114.dtwave.com:9092  --topic ATLAS_HOOK --consumer.config /usr/hdp/3.1.0.0-78/kafka/config/consumer.properties
  ```

2. 查看Atlas日志分析


/var/log/atlas/application.log

## 9.4 排查指南-Ranger中新建Policy后Kafka未生效

**问题**: 由于用户A对Topic B没有权限，于是在Ranger emr_dev_kafka Policies中新建Policy进行授权后，但是在命令中执行还是报用户A对Topic B没有权限查看。
​

**排查步骤**:

1. 在Ranger->Audit->Plugin Status中检查emr_dev_kafka service的状态，观察Plugin插件的状态。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637854522404-77ea4d98-50f5-4810-8bed-6a3228afe4b4.png#clientId=u9fa33a77-4a9f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=739&id=u9bc54601&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1478&originWidth=1996&originalType=binary&ratio=1&rotation=0&showTitle=false&size=954733&status=done&style=stroke&taskId=uf6829f5f-aed4-4baf-9aef-e1ab3bd9108&title=&width=998)
上图中第一个红框中的两条记录Active的时间是最近的且感叹号⚠️警示表示是正常，最后一条则相反。
​

2. 在Kafka机器上查看策略文件的时间。

Ranger会定期把策略文件同步到Kafka机器上，存储在文件 /etc/ranger/emr_dev_kafka/policycache/kafka_emr_dev_kafka.json中。

```shell
ll  /etc/ranger/emr_dev_kafka/policycache/kafka_emr_dev_kafka.json
```

3. 在Ranger->Audit->Plugin 中检查emr_dev_kafka service的状态，观察Plugin插件的状态。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637854879741-ed6b18f0-300a-46c6-988d-da5764217c59.png#clientId=ued46dc4a-37a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1430&id=u428db8ea&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1430&originWidth=2696&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1029330&status=done&style=stroke&taskId=ud5b57270-0eb8-4498-b558-bf1bf18586b&title=&width=2696)
上图红框中是正常记录，如果查看不到，则表明策略同步是有问题的。
​

4. 查看Kafka的日志 /var/log/kafka/ranger_kafka.log

5. 查看Ranger的日志 /var/log/ranger/admin/xa_portal.log

本文是遇到如下错误:

```shell
2021-11-25 00:00:03,266 [http-bio-6080-exec-5] ERROR org.apache.ranger.rest.ServiceREST (ServiceREST.java:2856) - getSecureServicePoliciesIfUpdated(emr_dev_kafka, 6) failed as User doesn't have permission to download Policy
2021-11-25 00:00:03,268 [http-bio-6080-exec-19] ERROR org.apache.ranger.rest.ServiceREST (ServiceREST.java:2856) - getSecureServicePoliciesIfUpdated(emr_dev_kafka, -1) failed as User doesn't have permission to download Policy
2021-11-25 00:00:03,269 [http-bio-6080-exec-5] INFO  org.apache.ranger.common.RESTErrorUtil (RESTErrorUtil.java:345) - Request failed. loginId=spark, logMessage=User doesn't have permission to download policy
javax.ws.rs.WebApplicationException
        at org.apache.ranger.common.RESTErrorUtil.createRESTException(RESTErrorUtil.java:337)
        at org.apache.ranger.rest.ServiceREST.getSecureServicePoliciesIfUpdated(ServiceREST.java:2873)
        at org.apache.ranger.rest.ServiceREST$$FastClassBySpringCGLIB$$92dab672.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:736)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:282)
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
```

日志显示是由于spark用户没有download policy的权限，因此在Ranger [emr_dev_kafka](http://emr128.dtwave.com:6080/#!/service/25/policies/0)中给spark用户加权限。
把policy.download.auth.users的值配置为spark，如下图所示。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637855310134-29a62d1c-1bc1-4027-9b27-8532013c449d.png#clientId=ued46dc4a-37a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1528&id=u943b8779&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1528&originWidth=2678&originalType=binary&ratio=1&rotation=0&showTitle=false&size=568822&status=done&style=stroke&taskId=u67e42bbb-96b8-4ae1-b86d-12ea970ae82&title=&width=2678)

> 注: 理论上应该是kafka用户，至于此处为什么是spark用户因时间关系还未去排查。
> 建议通过查看Ranger源码中来分析。

## 9.5 配置atlas用户能写入到ATLAS_HOOK中

编辑Ranger->emr_dev_kafka service，给default-policy.1.policyItem.1.users追加用户atlas 。
![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1637855603302-8f8b8243-518c-4a4c-afd8-bcb7940fb6ca.png#clientId=ued46dc4a-37a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1532&id=u52424c21&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1532&originWidth=2686&originalType=binary&ratio=1&rotation=0&showTitle=false&size=615893&status=done&style=stroke&taskId=u8d938459-e53b-40c6-9556-3e8aecb04d2&title=&width=2686)

## 9.6 排查指南-Atlas中新建Tag未同步到Ranger中

> 在机器192.168.90.115上用 root 用户操作。

**基础知识: **Ranger会启动ranger-tagsync进程同步Atlas上信息到Ranger中，会存储到/etc/ranger/emr_dev_hive/policycache目录中的hiveServer2_emr_dev_hive_tag.json文件中，此文件还包含从Ranger同步过来的Tag Based Policies。

```shell
[root@emr115 policycache]# cd /etc/ranger/emr_dev_hive/policycache
[root@emr115 policycache]# ll
总用量 660
-rw-r--r-- 1 hive hadoop 623273 11月 30 14:20 hiveServer2_emr_dev_hive.json
-rw-r--r-- 1 hive hadoop  45795 11月 30 14:59 hiveServer2_emr_dev_hive_tag.json
```

> hiveServer2_emr_dev_hive.json 文件是Hive从Ranger同步过来的Resource Based Policies。

[hiveServer2_emr_dev_hive_tag.json](https://dtwave.yuque.com/attachments/yuque/0/2021/json/499433/1640680740957-344c5992-60fd-4b23-869e-54573a7832a8.json?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fdtwave.yuque.com%2Fattachments%2Fyuque%2F0%2F2021%2Fjson%2F499433%2F1640680740957-344c5992-60fd-4b23-869e-54573a7832a8.json%22%2C%22name%22%3A%22hiveServer2_emr_dev_hive_tag.json%22%2C%22size%22%3A50431%2C%22type%22%3A%22application%2Fjson%22%2C%22ext%22%3A%22json%22%2C%22status%22%3A%22done%22%2C%22taskId%22%3A%22u9e30662b-21de-4f86-b55b-4136478e4ad%22%2C%22taskType%22%3A%22upload%22%2C%22id%22%3A%22u55a85aa2%22%2C%22card%22%3A%22file%22%7D)

**问题: **查看ranger-tagsync服务的日志/var/log/ranger/tagsync/tagsync.log，报如下错误:

```sql
29 十一月 2021 19:27:35 DEBUG AtlasRESTTagSource [Thread-6] - 313 <== getAtlasActiveEntities()
29 十一月 2021 19:27:35 DEBUG AtlasRESTTagSource [Thread-6] - 194 Sleeping for [60000] milliSeconds
29 十一月 2021 19:28:35 DEBUG AtlasRESTTagSource [Thread-6] - 238 ==> getAtlasActiveEntities()
29 十一月 2021 19:28:35 ERROR AtlasRESTTagSource [Thread-6] - 261 failed to download tags from Atlas
org.apache.atlas.AtlasServiceException: Metadata service API org.apache.atlas.AtlasClientV2$API_V2@7ce009c8 failed with status 401 (Unauthorized) Response Body ()
        at org.apache.atlas.AtlasBaseClient.callAPIWithResource(AtlasBaseClient.java:418)
        at org.apache.atlas.AtlasBaseClient.callAPIWithResource(AtlasBaseClient.java:344)
        at org.apache.atlas.AtlasBaseClient.callAPI(AtlasBaseClient.java:227)
        at org.apache.atlas.AtlasClientV2.facetedSearch(AtlasClientV2.java:383)
        at org.apache.ranger.tagsync.source.atlasrest.AtlasRESTTagSource.getAtlasActiveEntities(AtlasRESTTagSource.java:255)
        at org.apache.ranger.tagsync.source.atlasrest.AtlasRESTTagSource.synchUp(AtlasRESTTagSource.java:209)
        at org.apache.ranger.tagsync.source.atlasrest.AtlasRESTTagSource.run(AtlasRESTTagSource.java:192)
        at java.lang.Thread.run(Thread.java:748)
```

上述是认证错误，ranger-tagsync 目前是通过Atlas Rest的方式调用API来获取Tag等信息，用户名和密码都是admin。
​

**排查步骤: **

1. 对Ranger 「AccessManager」->「Resource Base Policies」->「Atlas」的[emr_dev_atlas](http://emr128.dtwave.com:6080/index.html#!/service/22/policies/0)中进行编辑，以下三个配置项中均增加admin用户。

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638259416953-141b04ae-fdb1-48af-b05b-139a5852925e.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=718&id=u69145256&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1436&originWidth=2710&originalType=binary&ratio=1&rotation=0&showTitle=false&size=392783&status=done&style=stroke&taskId=u77d8e465-d03e-449c-bfff-38dc24e232a&title=&width=1355)

2. 同时在Atlas的自定义 application-properties中添加如下三个配置项，

  ```shell
  atlas.client.type=rest
  atlas.client.username=admin
  atlas.client.password=admin
  ```

![imagepng](https://cdn.nlark.com/yuque/0/2021/png/499433/1638260077393-e7f50c82-c11f-462e-a2bf-9f5977c747d6.png#clientId=ua9496610-306e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=687&id=ud11159fa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1374&originWidth=2118&originalType=binary&ratio=1&rotation=0&showTitle=false&size=357589&status=done&style=stroke&taskId=u1dd5ad3e-90b4-4aa2-9379-298da3f2290&title=&width=1059)​

3. 再重启Atlas、Ranger验证，可通过查看/etc/ranger/emr_dev_hive/policycache/hiveServer2_emr_dev_hive_tag.json 文件的更新时间和内容来验证，也可测试下面的命令。

  ```shell
  curl -v -u admin:admin http://192.168.90.122:21000/api/atlas/admin/version
  ```

# 十、参考资料

4. [How to mask Hive columns using Atlas tags and Ranger?](https://www.youtube.com/watch?v=3XbZ6tga52Q)

5. [Kerberos报错：kinit: Password incorrect while getting initial credentials](https://blog.csdn.net/ZhouyuanLinli/article/details/78540299)

6. [https://github.com/emaxwell-hw/Atlas-Ranger-Tag-Security](https://github.com/emaxwell-hw/Atlas-Ranger-Tag-Security)

7. [kerberos认证下kafka报错Bootstrap broker host:ip disconnected](https://blog.csdn.net/github_39319229/article/details/114777296)

8. [Apache Ranger Policy Evaluation Flow with Tags](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.1/authorization-ranger/content/tags_and_policy_evaluation.html)

9. [Atlas Authorization Model](https://atlas.apache.org/1.2.0/Atlas-Authorization-Model.html)

10. [https://atlas.apache.org/](https://atlas.apache.org/)

11. [行级权限 - 数栈帮助文档.pdf](https://dtwave.yuque.com/attachments/yuque/0/2021/pdf/499433/1640680741115-f5d1384a-fa93-41ce-b58d-9c256b4f3711.pdf?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fdtwave.yuque.com%2Fattachments%2Fyuque%2F0%2F2021%2Fpdf%2F499433%2F1640680741115-f5d1384a-fa93-41ce-b58d-9c256b4f3711.pdf%22%2C%22name%22%3A%22%E8%A1%8C%E7%BA%A7%E6%9D%83%E9%99%90+-+%E6%95%B0%E6%A0%88%E5%B8%AE%E5%8A%A9%E6%96%87%E6%A1%A3.pdf%22%2C%22size%22%3A347429%2C%22type%22%3A%22application%2Fpdf%22%2C%22ext%22%3A%22pdf%22%2C%22status%22%3A%22done%22%2C%22taskId%22%3A%22ud703c5f1-491f-4c53-b1dd-ca824077d47%22%2C%22taskType%22%3A%22upload%22%2C%22id%22%3A%22u6cea16b2%22%2C%22card%22%3A%22file%22%7D)

12. [Quick BI-权限管理-数据行列权限-列级权限-数据脱敏](https://help.aliyun.com/document_detail/325480.html?spm=a2c4g.11186623.0.0.6aee4d2erSADtY)

13. [Quick BI-权限管理-数据行列权限-行级权限-标签授权](https://help.aliyun.com/document_detail/325414.html?spm=a2c4g.11186623.0.0.9a736e69OKaxvi)​
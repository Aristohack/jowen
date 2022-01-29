---
title: "Atlas调研"
date: 2022-01-29T10:52:13+08:00
draft: true
---
带着问题看:


1.元数据怎么同步(批量还是实时?)


2.元数据怎么存储


3.支持那些元数据


4.血缘怎么做


5.标签,术语怎么使用


6.标签是否可以传递


7.基于标签的安全是怎么做的


8.基于标签的数据过滤是怎么做的


### 参考:
[http://atlas.apache.org/2.1.0/index#/Architecture](http://atlas.apache.org/2.1.0/index#/Architecture) (官网)
[https://www.jianshu.com/p/4eee91bc926c](https://www.jianshu.com/p/4eee91bc926c)
[https://www.jianshu.com/p/b92575a8127e?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation](https://www.jianshu.com/p/b92575a8127e?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
[https://www.it610.com/article/1288017827889983488.htm](https://www.it610.com/article/1288017827889983488.htm)
[https://www.cnblogs.com/mantoudev/p/9965869.html](https://www.cnblogs.com/mantoudev/p/9965869.html)
### 1.介绍
		Atlas 是一组**可扩展和可扩展的核心基础治理服务**——使企业能够有效和高效地满足其在 Hadoop 中的合规性要求，并允许与整个企业数据生态系统集成。
		Apache Atlas 为组织提供开放的元数据管理和治理功能，以构建其数据资产的目录，对这些资产进行分类和治理，并为数据科学家、分析师和数据治理团队提供围绕这些数据资产的协作功能。
		Apache Atlas是Hadoop社区为解决Hadoop生态系统的**元数据治理问题**而产生的开源项目，它为Hadoop集群提供了包括**数据分类、集中策略引擎、数据血缘、安全和生命周期管理**在内的元数据治理核心能力。
当前最新版本2.1。emr集成版本1.1。


#### 1.1整体架构图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631702964900-eea8268d-aca3-4d3e-ba1c-487a29a041d9.png#clientId=u022f7a5c-acf4-4&from=paste&height=349&id=u8ae8dbf2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1046&originWidth=1854&originalType=binary&ratio=1&size=654567&status=done&style=none&taskId=u5f35645e-3779-4e77-830e-cd3d593e8d0&width=618)
KAFKA:数据交互通道
Hbase:数据存储
Solr:数据索引
#### 1.2核心模块
##### Core
此类别包含实现 Atlas 功能核心的组件，包括：


- Type System: Atlas 允许用户为他们想要管理的元数据对象定义一个模型。该模型由称为 "类型" 的定义组成。"类型" 的 实例被称为 "实体" 表示被管理的实际元数据对象。类型系统是一个组件，允许用户定义和管理类型和实体。由 Atlas 管理的所有元数据对象（例如Hive表）都使用类型进行建模，并表示为实体。要在 Atlas 中存储新类型的元数据，需要了解类型系统组件的概念。


- Ingest/Export：Ingest 组件允许将元数据添加到 Atlas。类似地，Export 组件暴露由 Atlas 检测到的元数据更改，以作为事件引发，消费者可以使用这些更改事件来实时响应元数据更改。


- Graph Engine：在内部，Atlas 通过使用图形模型管理元数据对象。以实现元数据对象之间的巨大灵活性和丰富的关系。图形引擎是负责在类型系统的类型和实体之间进行转换的组件，以及基础图形模型。除了管理图形对象之外，图形引擎还为元数据对象创建适当的索引，以便有效地搜索它们。


- Titan：目前，Atlas 使用 Titan 图数据库来存储元数据对象。 Titan 使用两个存储：默认情况下元数据存储配置为 HBase ，索引存储配置为 Solr。也可以通过构建相应的配置文件使用BerkeleyDB存储元数据存储和使用ElasticSearch存储 Index。元数据存储用于存储元数据对象本身，索引存储用于存储元数据属性的索引，其允许高效搜索。


##### Integration


用户可以使用两种方法管理 Atlas 中的元数据：


- API： Atlas 的所有功能都可以通过 REST API 提供给最终用户，允许创建，更新和删除类型和实体。它也是查询和发现通过 Atlas 管理的类型和实体的主要方法。


- Messaging：除了 API 之外，用户还可以选择使用基于 Kafka 的消息接口与 Atlas 集成。这对于将元数据对象传输到 Atlas 以及从 Atlas 使用可以构建应用程序的元数据更改事件都非常有用。如果希望使用与 Atlas 更松散耦合的集成，这可以允许更好的可扩展性，可靠性等，消息传递接口是特别有用的。Atlas 使用 Apache Kafka 作为通知服务器用于钩子和元数据通知事件的下游消费者之间的通信。事件由钩子(hook)和 Atlas 写到不同的 Kafka 主题:


- ATLAS_HOOK: 来自 各个组件的Hook 的元数据通知事件通过写入到名为 ATLAS_HOOK 的 Kafka topic 发送到 Atlas


- ATLAS_ENTITIES：从 Atlas 到其他集成组件（如Ranger）的事件写入到名为 ATLAS_ENTITIES 的 Kafka topic


##### Metadata source


Atlas 支持与许多元数据源的集成，将来还会添加更多集成。目前，Atlas 支持从以下数据源获取和管理元数据：


- Hive：通过hive bridge， atlas可以接入Hive的元数据，包括hive_db/hive_table/hive_column/hive_process


- Sqoop：通过sqoop bridge，atlas可以接入关系型数据库的元数据，包括sqoop_operation_type/ sqoop_dbstore_usage/sqoop_process/sqoop_dbdatastore


- Falcon：通过falcon bridge，atlas可以接入Falcon的元数据，包括falcon_cluster/falcon_feed/falcon_feed_creation/falcon_feed_replication/ falcon_process


- Storm：通过storm bridge，atlas可以接入流式处理的元数据，包括storm_topology/storm_spout/storm_bolt


Atlas集成大数据组件的元数据源需要实现以下两点：


- 首先，需要基于atlas的类型系统定义能够表达大数据组件元数据对象的元数据模型(例如Hive的元数据模型实现在org.apache.atlas.hive.model.HiveDataModelGenerator)；


- 然后，需要提供hook组件去从大数据组件的元数据源中提取元数据对象，实时侦听元数据的变更并反馈给atlas；


##### Applications


- Atlas Admin UI: 该组件是一个基于 Web 的应用程序，允许数据管理员和科学家发现和注释元数据。Admin UI提供了搜索界面和 类SQL的查询语言，可以用来查询由 Atlas 管理的元数据类型和对象。Admin UI 使用 Atlas 的 REST API 来构建其功能。


- Tag Based Policies: Apache Ranger 是针对 Hadoop 生态系统的高级安全管理解决方案，与各种 Hadoop 组件具有广泛的集成。通过与 Atlas 集成，Ranger 允许安全管理员定义元数据驱动的安全策略，以实现有效的治理。 Ranger 是由 Atlas 通知的元数据更改事件的消费者。


- Business Taxonomy:从元数据源获取到 Atlas 的元数据对象主要是一种技术形式的元数据。为了增强可发现性和治理能力，Atlas 提供了一个业务分类界面，允许用户首先定义一组代表其业务域的业务术语，并将其与 Atlas 管理的元数据实体相关联。业务分类法是一种 Web 应用程序，目前是 Atlas Admin UI 的一部分，并且使用 REST API 与 Atlas 集成。


- 在Atlas > Configs > Advanced > Custom application-properties中添加atlas.feature.taxonomy.enable=true并重启atlas服务来开启


### 2.核心特性


#### 2.1 元数据定义与表示


##### type(类型)


- **Type**：Atlas中的 “类型” 定义了如何存储和访问特定类型的元数据对象。类型表示了所定义元数据对象的一个或多个属性集合。具有开发背景的用户可以将 “类型” 理解成面向对象的编程语言的 “类” 定义的或关系数据库的 “表模式”。类型具有元类型，元类型表示 Atlas 中此模型的类型：


Hive表定义了以下属性：


```
Name:         hive_table
TypeCategory: Entity
SuperTypes:   DataSet
Attributes:
    name:             string
    db:               hive_db
    owner:            string
    createTime:       date
    lastAccessTime:   date
    comment:          string
    retention:        int
    sd:               hive_storagedesc
    partitionKeys:    array<hive_column>
    aliases:          array<string>
    columns:          array<hive_column>
    parameters:       map<string,string>
    viewOriginalText: string
    viewExpandedText: string
    tableType:        string
    temporary:        boolean
```


从上面的例子中可以注意到以下几点：


-  Atlas中的类型(Type)由`name`唯一标识
-  Type具有元类型。Atlas中有以下元类型：
    - **原始元类型(Primitive metatypes)**：boolean，byte，short，int，long，float，double，biginteger，bigdecimal，string，date
    - **枚举元型(Enum metatypes)**
    - **集合元类型(Collection metatypes:)**：array, map
    - **复合元类型(Composite metatypes)**：Entity, Struct, Classification, Relationship
-  实体(Entity)和分类(Classification)类型可以从其他类型继承，称为“超类型/父类型”(supertype) ，它包括在超类型中定义的属性。这允许建模者在一组相关类型等中定义公共属性。类似于面向对象语言如何为类定义父类。 Atlas中的类型也可以从多个超类型扩展。
    - 在此示例中，每个配置单元表都从称为`DataSet`的预定义超类型扩展。稍后将提供有关此预定义类型的更多详细信息。
-  具有元类型`Entity`，`Struct`，`Classification`或`Relationship`的类型可以具有属性的集合。每个属性都有一个名称（例如: `name`）和一些其他相关属性。可以使用表达式`type_name.attribute_name`引用属性。值得注意的是，属性本身是使用Atlas元类型定义的。
    - 在此示例中，hive_table.name是String，hive_table.aliases是一个字符串数组，hive_table.db是指一个名为hive_db的类型的实例，依此类推。
-  属性中的类型引用（如hive_table.db）特别有趣，使用这样的属性，我们可以定义Atlas中定义的两种类型之间的任意关系，从而构建丰富的模型。此外，还可以将引用列表收集为属性类型（例如，hive_table.columns，表示从hive_table到hive_column类型的引用列表）
   ![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703028901-b9203f17-6ac3-4619-bf13-ab61c81fcf13.png#clientId=u022f7a5c-acf4-4&from=paste&height=457&id=u9675715c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1372&originWidth=1806&originalType=binary&ratio=1&size=1179779&status=done&style=none&taskId=u1340d11c-11aa-403e-8aef-4c7b98975f1&width=602)
##### Entities(实体)
Atlas中的`entity`是`type`的特定值或实例，因此表示现实世界中的特定元数据对象。用我们对面向对象编程语言的类比，`实例(instance)`是某个`类(Class)`的`对象(Object)`。
实体的其中一个示例就是Hive表。Hive在'default'数据库中有一个名为'customers'的表。该表是hive_table类型的Atlas中的“实体”。由于是实体类型的实例，它将具有作为Hive表'type'的一部分的每个属性的值，例如：
```
guid:     "9ba387dd-fa76-429c-b791-ffc338d3c91f"
typeName: "hive_table"
status:   "ACTIVE"
values:
    name:             “customers”
    db:               { "guid": "b42c6cfc-c1e7-42fd-a9e6-890e0adf33bc", "typeName": "hive_db" }
    owner:            “admin”
    createTime:       1490761686029
    updateTime:       1516298102877
    comment:          null
    retention:        0
    sd:               { "guid": "ff58025f-6854-4195-9f75-3a3058dd8dcf", "typeName": "hive_storagedesc" }
    partitionKeys:    null
    aliases:          null
    columns:          [ { "guid": ""65e2204f-6a23-4130-934a-9679af6a211f", "typeName": "hive_column" }, { "guid": ""d726de70-faca-46fb-9c99-cf04f6b579a6", "typeName": "hive_column" }, ...]
    parameters:       { "transient_lastDdlTime": "1466403208"}
    viewOriginalText: null
    viewExpandedText: null
    tableType:        “MANAGED_TABLE”
    temporary:        false
```
从上面的例子中可以注意到以下几点：

- 实体类型的每个实例都由唯一标识符GUID标识。此GUID由Atlas服务器在定义对象时生成，并在实体的整个生命周期内保持不变。在任何时间点，都可以使用其GUID访问此特定实体。
    - 在此示例中，默认数据库中的“customers”表由GUID“9ba387dd-fa76-429c-b791-ffc338d3c91f”唯一标识。
- 实体具有给定类型，并且类型的名称随实体定义一起提供。
    - 在此示例中，'customers'表是'hive_table'类型。
- 该实体的值是hive_table类型定义中定义的属性的所有属性名称及其值的映射。
  属性值将根据属性的数据类型。实体类型属性将具有AtlasObjectId类型的值

有了实体的这个设计，我们现在可以看到Entity和Struct元类型之间的区别。实体(Entity)和结构(Entity)都构成其他类型的属性。但是，实体类型的实例具有标识(具有GUID值)，并且可以从其他实体引用（例如，从hive_table实体引用hive_db实体）。 Struct类型的实例没有自己的标识。 Struct类型的值是在实体本身内“嵌入”的属性集合。
##### Attributes(属性)
我们已经看到，属性(attributes)是在实体(Entity)，结构(Struct)，分类(Classification)和关系(Relationship)等元类型中定义的。但我们将属性列举为具有名称和元类型值。然而，Atlas中的attributes具有一些properties，这些properties定义了与类型系统相关的更多概念。
attributes具有以下properties：
```
name:        string,
    typeName:    string,
    isOptional:  boolean,
    isIndexable: boolean,
    isUnique:    boolean,
    cardinality: enum
```
上述属性具有以下含义：

- `name`: 属性的名称
- `dataTypeName`: 属性的元类型名称（native, collection, composite)）
- `isComposite`:
    - 该标志表示建模的一个方面。如果将属性定义为复合(composite)，则意味着它不能具有独立于其所包含的实体的生命周期。这个概念的一个很好的示例是构成hive表的一部分的列集。由于列在hive表外部没有意义，因此它们被定义为复合属性。
    - 必须在Atlas中创建复合属性及其包含的实体。即，必须与hive表一起创建配置单元列。
- `isIndexable`
    - 标志指示是否应该对此属性建立索引，以便可以使用属性值作为谓词来执行查找，并且可以有效地执行查找。
- `isUnique`
    - 同样与索引相关。如果指定为唯一，则表示在JanusGraph中为此属性创建了一个特殊索引，允许基于相等的查找。
    - 具有该标志的真值的任何属性都被视为主键，以将该实体与其他实体区分开。因此，应该注意确保此属性确实在现实世界中为唯一属性建模。
        - 对于例如考虑hive_table的name属性。在单独的情况下，名称不是hive_table的唯一属性，因为具有相同名称的表可以存在于多个数据库中。如果Atlas在多个集群中存储hive表的元数据，那么即使是一对（数据库名称，表名）也不是唯一的。在物理世界中，只有集群位置，数据库名称和表名称才能被视为唯一。
- `multiplicity`: 标示该属性是必选(required)，可选(optional)的还是可以是多值的(multi-valued)。如果实体的属性值定义与类型定义中的多重性声明不匹配，则这将违反约束，并且实体添加将失败。因此，该字段可用于定义元数据信息的一些约束。

根据上面的内容，让我们展开下面的hive表的一个attributes的属性定义。让我们看一下名为'db'的属性，它表示hive表所属的数据库：
```
db:
    "name":        "db",
    "typeName":    "hive_db",
    "isOptional":  false,
    "isIndexable": true,
    "isUnique":    false,
    "cardinality": "SINGLE"
```
请注意“isOptional = true”约束 - 如果没有db引用，则无法创建表实体。
```
columns:
    "name":        "columns",
    "typeName":    "array<hive_column>",
    "isOptional":  optional,
    "isIndexable": true,
    “isUnique":    false,
    "constraints": [ { "type": "ownedRef" } ]
```
请注意列的“ownedRef”约束。通过这样，我们指出定义的列实体应始终绑定到它们所定义的表实体。
通过此描述和示例，您将能够意识到属性定义可用于影响Atlas系统强制执行的特定建模行为（约束，索引等）。
##### 系统特定类型及含义
Atlas自带了一些预定义的系统类型。我们在前面的部分中看到了一个示例（DataSet）。在本节中，我们将看到更多这些类型并了解它们的重要性。

- **Referenceable**：该类型表示可以使用名为qualifiedName的唯一属性搜索的所有实体。
- **Asset**：该类型扩展了Referenceable并添加了名称，描述和所有者等属性。 Name是必需属性（isOptional = false），其他属性是可选的。

Referenceable和Asset的目的是为建模者提供在定义和查询自己类型的实体时强制一致性的方法。拥有这些固定的属性集允许应用程序和用户界面基于约定做出关于默认情况下它们可以期望类型的属性的假设。

- **Infrastructure**：该类型继承自Asset，通常可用作基础结构元数据对象（如集群，主机等）的常见超类型。
- **DataSet**：该类型继承自Referenceable。从概念上讲，它可以用于表示存储数据的类型。在Atlas中，hive表，hbase_tables等都是从DataSet扩展的类型。扩展DataSet的类型可以预期具有Schema，因为它们具有定义该数据集的属性的属性。对于例如hive_table中的columns属性。此外，扩展DataSet的类型实体参与数据转换，Atlas可以通过血缘）图了解到转换过程。
- **Process**：该类型继承自Asset。从概念上讲，它可以用于表示任何数据转换操作。例如，将具有原始数据的配置单元表转换为存储某些聚合的另一个配置单元表的ETL过程可以是扩展Process类型的特定类型。流程类型有两个特定属性，即输入和输出。输入和输出都是DataSet实体的数组。因此，Process类型的实例可以使用这些输入和输出来捕获DataSet的血缘如何演变。
#### 2.2 高可用
      通过zookeeper支持主备高可用
#### 2.3 dsl查询支持
[http://atlas.apache.org/2.1.0/index#/SearchBasic](http://atlas.apache.org/2.1.0/index#/SearchBasic)
[http://atlas.apache.org/2.1.0/index#/SearchAdvance](http://atlas.apache.org/2.1.0/index#/SearchAdvance)
#### 2.4 分级(Classifications)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703125489-c0e95d67-1c29-4659-94aa-c9b76422e26d.png#clientId=u022f7a5c-acf4-4&from=paste&height=141&id=ub25692c0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=424&originWidth=1936&originalType=binary&ratio=1&size=88103&status=done&style=none&taskId=u90349057-ce12-4086-ae7b-394985f8dc6&width=645.3333333333334)
分级可以设置属性Attributes。在entitys页面可以设置分级及属性值，如下:
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703145969-0417e991-fdb3-4b1e-b8d2-c2c0770e1c69.png#clientId=u022f7a5c-acf4-4&from=paste&height=458&id=u2f60dd80&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1374&originWidth=1940&originalType=binary&ratio=1&size=219893&status=done&style=none&taskId=uef6d33fc-9aaa-467e-b53a-17742b4f5a2&width=646.6666666666666)
#### 2.5 术语(Glossary)
这里需要先建Glossary，在Glossary下新建Terms和Category。 Glossary更像一个目录。Terms和Category 共享Glossary。
Terms添加Category时，只能添加同一目录下的。Category添加Terms时一样。
##### 术语(Terms)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703181642-bed890ea-b7a1-4ffe-b914-db027ebaaec5.png#clientId=u022f7a5c-acf4-4&from=paste&height=205&id=MrUAQ&margin=%5Bobject%20Object%5D&name=image.png&originHeight=614&originWidth=1934&originalType=binary&ratio=1&size=165454&status=done&style=none&taskId=u0158e086-952f-4814-a6d6-e88634317c1&width=644.6666666666666)
分级的属性值也可以单独设置
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703210638-c09ff9cc-d006-4d93-a7a6-0d49dc1d0d74.png#clientId=u022f7a5c-acf4-4&from=paste&height=201&id=ufc096126&margin=%5Bobject%20Object%5D&name=image.png&originHeight=604&originWidth=1930&originalType=binary&ratio=1&size=117626&status=done&style=none&taskId=u8f90849b-74c4-4818-ae89-23da2cfed1e&width=643.3333333333334)
##### 术语分组(Category)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703230292-f17403d3-9823-4634-be6f-74cafaa3f55e.png#clientId=u022f7a5c-acf4-4&from=paste&height=143&id=ubbeae0cf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=430&originWidth=1946&originalType=binary&ratio=1&size=88077&status=done&style=none&taskId=u44bbb734-6758-4602-86f0-f082c29fe4f&width=648.6666666666666)
#### 2.6 分级(Classifications)，术语(Glossary) 跟entitys之间关系
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703246260-4575309a-8d4f-41de-978c-5c278508a007.png#clientId=u022f7a5c-acf4-4&from=paste&height=78&id=u5ea19677&margin=%5Bobject%20Object%5D&name=image.png&originHeight=234&originWidth=1926&originalType=binary&ratio=1&size=36388&status=done&style=none&taskId=u6353d225-69ff-45b0-a9b8-6f13e1202f5&width=642)

-  **Entitys是具体的实体**
-  **Entitys可以设置分级(Classifications)和术语(Terms)**
-  **术语(Terms)可以添加分级(Classifications)和分类(Category)**
-  **Entitys设置对应的术语(Terms)时，术语关联的分级(Classifications) 自动关联到Entitys**
-  **Category，Glossary，Terms 在atlas中也是有对应的type的(AtlasGlossaryCategory,AtlasGlossary,AtlasGlossaryTerm)，并且也是一个个Entity。从Classifications的关联中就可以看出**
   具体的分级(Classifications)，术语(Terms)传递性查看

[http://atlas.apache.org/2.1.0/index#/ClassificationPropagation](http://atlas.apache.org/2.1.0/index#/ClassificationPropagation)
#### 2.7 Atlas数据存储JanuxGraph
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703267408-1fdcf7c2-84a6-4853-88e7-4b29372a0cd5.png#clientId=u022f7a5c-acf4-4&from=paste&height=391&id=udcd24e5c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1174&originWidth=1990&originalType=binary&ratio=1&size=1107545&status=done&style=none&taskId=ub3b3fb9b-b2d1-4a6e-ab22-ba3d165d65f&width=663.3333333333334)
### 3. Atlas元数据导入(以Hive为例)
#### 3.1 批量导入
Atlas在各个插件里提供了元数据导入的脚本。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703286951-be5fdb1e-0ba5-44cd-854f-1e7c68a37846.png#clientId=u022f7a5c-acf4-4&from=paste&height=221&id=u98f1859a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=662&originWidth=1928&originalType=binary&ratio=1&size=211834&status=done&style=none&taskId=u76dbe641-cbde-449a-8b6f-7ffab89ad55&width=642.6666666666666)
```shell
Usage 1: <atlas package>/hook-bin/import-hive.sh
Usage 2: <atlas package>/hook-bin/import-hive.sh [-d <database regex> OR --database <database regex>] [-t <table regex> OR --table <table regex>]
Usage 3: <atlas package>/hook-bin/import-hive.sh [-f <filename>]
           File Format:
             database1:tbl1
             database1:tbl2
             database2:tbl1
```
#### 3.2 实时同步导入
	Atlas支持插件化部署元数据获取，捕捉对应的元数据变更新信息。通过把变更信息写入到 Kafka ATLAS_HOOK消息队列，来同步元数据信息。
支持的操作变更:

- create database
- create table/view, create table as select
- load, import, export
- DMLs (insert)
- alter database
- alter table (skewed table information, stored as, protection is not supported)
- alter view
### 4. Atlas与Ranger打通
#### 4.1 Atlas权限与Ranger
Atlas 支持Ranger 权限控制
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703325947-3d4cd784-6e87-4934-95cd-98c3b670d100.png#clientId=u022f7a5c-acf4-4&from=paste&height=103&id=u112ec25c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=310&originWidth=1612&originalType=binary&ratio=1&size=42627&status=done&style=none&taskId=uaebdb929-ea8d-4cdf-b61c-e9ddc1e233c&width=537.3333333333334)
控制的权限只要是对atlas内 entitys,terms,category等的操作
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703343090-39405ac8-7415-47f7-97c8-6e041f6a7893.png#clientId=u022f7a5c-acf4-4&from=paste&height=75&id=uef57224c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=226&originWidth=1932&originalType=binary&ratio=1&size=77667&status=done&style=none&taskId=ub133826b-6d1f-4833-beee-d087db4b14b&width=644)
#### 4.2 Ranger 基于Tag的策略
Ranger支持基于Tag的策略管理，及基于Tag的权限管理
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703365248-c18f08b2-a491-4115-9c83-f0c65c6d339c.png#clientId=u022f7a5c-acf4-4&from=paste&height=130&id=u44ca09f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=390&originWidth=1924&originalType=binary&ratio=1&size=132536&status=done&style=none&taskId=u9475910f-81af-45c7-858d-b46070f9c25&width=641.3333333333334)
新建tag服务
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703380855-d4a988b5-ea5e-4937-b661-d5a85de5d57a.png#clientId=u022f7a5c-acf4-4&from=paste&height=245&id=u81a120b4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=734&originWidth=1934&originalType=binary&ratio=1&size=90916&status=done&style=none&taskId=u64b43519-9906-4b5c-abda-c9bc363aa0b&width=644.6666666666666)
tag服务支持 Access和 Masking策略。 不支持row filter。
在其他服务里可以通过关联tag服务来启用tag策略
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703404878-6046784d-e835-40f6-8cd2-70e66c0c609f.png#clientId=u022f7a5c-acf4-4&from=paste&height=379&id=u430b09a3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1136&originWidth=1918&originalType=binary&ratio=1&size=165826&status=done&style=none&taskId=ud8482cb4-78ba-4664-a652-b8a0d9c66c8&width=639.3333333333334)
关于tag策略在具体的服务中的使用流程，具体分析各个创建。ranger base plugin 提供统一的策略增强能力。
对应的插件例如ranger hive plugin会在初始化的时候，首先初始化hive服务的策略，然后初始化服务关联的其他增强策略例如 Tag策略。然后做策略合并。
具体的tag策略是选服务组件的
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703429242-5d610eb9-8d89-452f-b2e1-1bc3c712f130.png#clientId=u022f7a5c-acf4-4&from=paste&height=166&id=ue7814eba&margin=%5Bobject%20Object%5D&name=image.png&originHeight=498&originWidth=1924&originalType=binary&ratio=1&size=116671&status=done&style=none&taskId=u62068e10-d28a-4958-8218-cb614166ae1&width=641.3333333333334)
#### 4.3 Atlas作为Ranger的tag来源
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703454446-12104d62-8509-4d0f-884b-c9dc079e209f.png#clientId=u022f7a5c-acf4-4&from=paste&height=187&id=u8eac5781&margin=%5Bobject%20Object%5D&name=image.png&originHeight=560&originWidth=1924&originalType=binary&ratio=1&size=71139&status=done&style=none&taskId=u82670dc7-39c1-4e30-8bf1-aace6ccb127&width=641.3333333333334)
Ranger通过Ranger TagSync服务(独立服务)来
1.数据同步方式

- **Atlas Tag Source** : 通过kafka 接收Entitys队列消息
- **AtlasRest Tag Source** : 通过Atlas rest 接口周期获取

2.有效数据
Ranger只会同步Active的分级及有使用的分级(有关联entitys)
3.数据格式及存储
上传tag到ranger 数据格式,其中"serviceName":"emr_dev_hive" 是拼出来的(因为没有对应的配置)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703484289-1047f8c0-e18a-4766-9981-3c25b764ee57.png#clientId=u022f7a5c-acf4-4&from=paste&height=112&id=u0f0518c1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=336&originWidth=1916&originalType=binary&ratio=1&size=109997&status=done&style=none&taskId=u8b830e65-c3a1-4011-87b0-fb8f1938341&width=638.6666666666666)
```json
{"op":"replace","serviceName":"emr_dev_hive","tagVersion":0,"tagDefinitions":{"0":{"name":"test1","source":"Atlas","attributeDefs":[{"name":"t1","type":"string"},{"name":"t2","type":"string"}],"id":0,"isEnabled":true}},"tags":{"0":{"type":"test1","owner":0,"attributes":{"t1":"11","t2":"12"},"options":{},"validityPeriods":[],"id":0,"isEnabled":true}},"serviceResources":[{"serviceName":"emr_dev_hive","resourceElements":{"database":{"values":["qxd_dev11111"],"isExcludes":false,"isRecursive":false}},"id":0,"guid":"1c3f6d95-365d-402d-9afc-e48a6d90cf0c","isEnabled":true}],"resourceToTagIds":{"0":[0]}}
```
tag数据库存储(ranger 数据库)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703504683-948c1fad-2ff9-4e46-8243-7e25af7f9ab2.png#clientId=u022f7a5c-acf4-4&from=paste&height=165&id=u8774d30d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=496&originWidth=1934&originalType=binary&ratio=1&size=232013&status=done&style=none&taskId=ufa2deb8c-2228-4b68-b846-6fff915d489&width=644.6666666666666)
### 5. 完善的RestFul接口
基本通过rest接口与其他服务对接
### 6. Atlas安全
[http://atlas.apache.org/2.1.0/index#/Security](http://atlas.apache.org/2.1.0/index#/Security)
简单来说 支持simple和kerberos认证。http支持spengo认证。
但是在1.1的版本上我一直没走通spengo的simple认证(服务是kerberos，http单独设置spengo simple)。所以现在整体atlas是simple认证，这也跟ranger tagSync当前的认证代码有关。
**综合考虑认证这块可以放开，但是可以结合ranger 做到用户权限控制。**
### 7.基于Hive的Atlas使用
[http://atlas.apache.org/1.1.0/Hook-Hive.html](http://atlas.apache.org/1.1.0/Hook-Hive.html)
开发环境地址:
[http://192.168.90.122:21000/](http://192.168.90.122:21000/)
用户名:admin      密码:admin
#### 7.0数据准备
CMR库:  用户表，地址表
订单库:  订单记录表，订单商品关联表，用户购买商品统计表
商品库:  商品记录表
账户库:  用户账户表，支付记录表，支付方式表，用户消费金额统计表
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703554060-886c07c1-21d7-4c4e-976c-3ec41890b836.png#clientId=u022f7a5c-acf4-4&from=paste&height=351&id=uec665165&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1052&originWidth=1958&originalType=binary&ratio=1&size=154163&status=done&style=none&taskId=u49f2c081-095d-4483-9fdc-4f08f3b240e&width=652.6666666666666)


```sql
### 订单库
create database atlas_hive_order comment '订单库';

# 订单表
create table IF NOT EXISTS atlas_hive_order.atlas_order (order_id string comment '订单id', user_id  int comment '用户id', order_comment string comment '订单备注', order_status string comment '订单状态:OPEN,FINISH,CANCEL', payment_status string comment '支付状态:WAIT,SUCCESS,FAILED', pay_id string comment '支付id' ) comment '订单表' PARTITIONED BY ( order_date string comment '订单日期') row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 订单商品关联表
create table IF NOT EXISTS atlas_hive_order.atlas_order_products (order_id string comment '订单id', product_id string comment '商品id', product_num int comment '商品数量', product_price decimal(12,2) comment '商品金额') comment '订单商品关联表' row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 用户购买商品统计表(现在没用，通过view去看了血缘)
create table IF NOT EXISTS atlas_hive_order.atlas_user_order_statistic (user_id int comment '用户id', product_id string comment '商品id', product_number int comment '商品数') comment '用户购买商品统计表' PARTITIONED BY (ds string comment '月数') row format delimited fields terminated by ',' STORED AS TEXTFILE;

### 用户库
create database atlas_hive_user comment '用户库';

# 用户表
create table IF NOT EXISTS atlas_hive_user.atlas_user(user_id int comment '用户id', name string comment '用户名', mobile string comment '电话', address_code string comment '用户地址code') comment '用户表' row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 地址表
create table IF NOT EXISTS atlas_hive_user.atlas_address(address_code string comment '用户地址code', detail string comment '用户地址详情') comment '地址表' row format delimited fields terminated by ',' STORED AS TEXTFILE;

### 账户库
create database atlas_hive_account comment '账户库';

# 账户表
create table IF NOT EXISTS atlas_hive_account.atlas_account(account_id int comment '账户id', user_id int comment '用户id', user_money decimal(12,2) comment '用户余额') comment '账户表' row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 支付记录表
create table IF NOT EXISTS atlas_hive_account.atlas_payment(pay_id string comment '支付id', payment_method int comment '支付方式', account_id int comment '账户id', payment_money decimal(12,2)  comment '账单金额') comment '支付记录表' PARTITIONED BY (ds string comment '支付日期') row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 支付方式表
create table IF NOT EXISTS atlas_hive_account.atlas_payment_dict(payment_method int comment '支付方式', payment_name string comment '支付方式') comment '支付方式表' row format delimited fields terminated by ',' STORED AS TEXTFILE;

# 用户消费金额统计表(现在没用，通过view去看了血缘)
create table IF NOT EXISTS atlas_hive_account.atlas_user_payment_statistic(count_money decimal comment '金额统计') comment '用户消费金额统计表' PARTITIONED BY (user_id int comment '用户id', ds string comment '支付日期') row format delimited fields terminated by ',' STORED AS TEXTFILE;

### 商品库
create database atlas_hive_product comment '商品库';

# 商品表
create table IF NOT EXISTS atlas_hive_product.atlas_product (product_id string comment '商品id' , product_number int comment '商品数量' , product_name string comment '商品名称' , product_price decimal(12,2)  comment '商品单价' ) comment '商品表' row format delimited fields terminated by ',' STORED AS TEXTFILE;
```
表数据记录
```
### 订单库
# 订单表
# load data inpath '/atlasdemo/atlas/atlas_order.txt' overwrite into table atlas_hive_order.atlas_order;
'O_00001',1,'下单1','FINISH','SUCCESS','PAY_00001','2021-01-01'
'O_00002',1,'下单2','FINISH','SUCCESS','PAY_00002','2021-01-11''
'O_00003',1,'下单3','FINISH','SUCCESS','PAY_00003','2021-02-11'
'O_00004',1,'下单4','OPEN','WAIT','','2021-01-01'
'O_00005',1,'下单5','OPEN','WAIT','','2021-05-01'
'O_00006',2,'下单6','FINISH','SUCCESS','PAY_00004','2021-01-01'
'O_00007',2,'下单7','FINISH','SUCCESS','PAY_00005','2021-01-11'
'O_00008',3,'下单8','FINISH','SUCCESS','PAY_00006','2021-07-01'
'O_00009',3,'下单9','FINISH','SUCCESS','PAY_00007','2021-01-11'
'O_00010',3,'下单10','FINISH','SUCCESS','PAY_00008','2021-02-11'
'O_00011',3,'下单11','FINISH','SUCCESS','PAY_00009','2021-12-11'
'O_00012',3,'下单12','FINISH','SUCCESS','PAY_00010','2021-06-11'
'O_00013',3,'下单13','CANCEL','WAIT','','2021-01-01'
'O_00014',4,'下单14','FINISH','SUCCESS','PAY_00011','2021-12-11'
'O_00015',4,'下单15','CANCEL','WAIT','','2021-06-11'
'O_00016',4,'下单16','OPEN','WAIT','','2021-10-01'
'O_00017',4,'下单17','OPEN','WAIT','','2021-11-01'
'O_00018',5,'下单18','FINISH','SUCCESS','PAY_00012','2021-01-11'
'O_00019',5,'下单19','FINISH','SUCCESS','PAY_00013','2021-01-11'
'O_00020',5,'下单20','OPEN','WAIT','','2021-01-01'

# 订单商品关联表
# load data inpath '/atlasdemo/atlas/atlas_order_products.txt' overwrite into table atlas_hive_order.atlas_order_products;
'O_00001','P_00001',1,1.01
'O_00001','P_00002',10,2.01
'O_00002','P_00001',1,1.01
'O_00002','P_00003',2,3.01
'O_00003','P_00001',3,1.01
'O_00003','P_00001',6,3.01
'O_00003','P_00002',11,1.01
'O_00004','P_00003',22,3.01
'O_00005','P_00001',3,1.01
'O_00006','P_00001',44,3.01
'O_00007','P_00001',36,1.01
'O_00007','P_00001',37,3.01
'O_00008','P_00001',12,1.01
'O_00009','P_00002',212,3.01
'O_00010','P_00003',122,1.01
'O_00011','P_00003',43,3.01
'O_00011','P_00005',71,1.01
'O_00012','P_00004',23,3.01
'O_00013','P_00003',14,1.01
'O_00014','P_00003',21,3.01
'O_00015','P_00001',12,1.01
'O_00016','P_00003',24,3.01
'O_00017','P_00002',19,1.01
'O_00018','P_00003',21,3.01
'O_00019','P_00003',26,3.01
'O_00019','P_00003',13,1.01
'O_00020','P_00006',24,3.01

### 用户库
# 用户表
# load data inpath '/atlasdemo/atlas/atlas_user.txt' overwrite into table atlas_hive_user.atlas_user;
1,'USER1','13822223145','ADDRESS1'
2,'USER2','13822223222','ADDRESS2'
3,'USER3','13822223122','ADDRESS3'
4,'USER4','13822223112','ADDRESS4'
5,'USER5','13822223555','ADDRESS5'

# 地址表
# load data inpath '/atlasdemo/atlas/atlas_address.txt' overwrite into table atlas_hive_user.atlas_address;
'ADDRESS1','山顶洞'
'ADDRESS2','美国'
'ADDRESS3','日本'
'ADDRESS4','韩国'
'ADDRESS5','英国'

### 账户库
# 账户表
# load data inpath '/atlasdemo/atlas/atlas_account.txt' overwrite into table atlas_hive_account.atlas_account;
1,1,1000.00
2,2,5000.00
3,3,2000.00
4,4,1500.00
5,5,1333.00

# 支付记录表
# load data inpath '/atlasdemo/atlas/atlas_payment.txt' overwrite into table atlas_hive_account.atlas_payment;
'PAY_00001',1,1,20.00,'2021-01-01'
'PAY_00002',2,1,13.23,'2021-01-11'
'PAY_00003',2,1,33.00,'2021-02-11'
'PAY_00004',4,2,20.00,'2021-01-01'
'PAY_00005',2,2,13.23,'2021-01-11'
'PAY_00006',4,3,23.00,'2021-07-11'
'PAY_00007',3,3,3.00,'2021-01-11'
'PAY_00008',2,3,33.30,'2021-02-11'
'PAY_00009',4,3,31.20,'2021-12-11'
'PAY_00010',4,3,39.00,'2021-06-11'
'PAY_00011',3,4,1.00,'2021-02-11'
'PAY_00012',3,5,323.00,'2021-01-11'
'PAY_00013',1,5,32.00,'2021-01-11'

# 支付方式表
# load data inpath '/atlasdemo/atlas/atlas_payment_dict.txt' overwrite into table atlas_hive_account.atlas_payment_dict;
1,'UNION'
2,'AliPay'
3,'ApplePay'
4,'WePay'

### 商品库
# 商品表
# load data inpath '/atlasdemo/atlas/atlas_product.txt' overwrite into table atlas_hive_product.atlas_product;
'P_00001',100333,'apple',1.01
'P_00002',20123,'orange',2.01
'P_00003',30044,'purp',3.01
'P_00004',4033,'banana',4.01
'P_00005',1508,'car',2.02
'P_00006',2203,'water',3.04
```
##### 7.0.1 标签(Classifications)设计
**标签可以建子标签;且父标签属性具有传递性;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703606267-7101e6dc-c6ae-4f1e-afae-d41f8bde448c.png#clientId=u022f7a5c-acf4-4&from=paste&height=655&id=u6f75bd5a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1966&originWidth=1956&originalType=binary&ratio=1&size=573318&status=done&style=none&taskId=ucc638ef1-d2be-4116-9bbe-c3f8f7c136b&width=652)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703629012-de429133-6549-4ab4-b211-aaa14e1ca700.png#clientId=u022f7a5c-acf4-4&from=paste&height=314&id=ud9db027d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=942&originWidth=1922&originalType=binary&ratio=1&size=156401&status=done&style=none&taskId=u92bb045b-0840-4484-9427-a06ff5d744a&width=640.6666666666666)
##### 7.0.2 术语分组(Category)设计
**同术语(Terms)共享目录结构;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703660098-d72ce9bc-0a0b-4b92-9037-a3f3533006f3.png#clientId=u022f7a5c-acf4-4&from=paste&height=190&id=ue7ecf5b6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=570&originWidth=1930&originalType=binary&ratio=1&size=321075&status=done&style=none&taskId=uebb746c6-d5ae-4fc2-9eaa-bdca936705e&width=643.3333333333334)
**可以新建子分类;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703677878-cf2b45f0-f21c-490d-9131-26588f3466b4.png#clientId=u022f7a5c-acf4-4&from=paste&height=355&id=u0bf31696&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1066&originWidth=1932&originalType=binary&ratio=1&size=259854&status=done&style=none&taskId=u64088ba8-92c2-43da-8055-247aa41b936&width=644)
##### 7.0.3 术语(Terms)设计
同术语分组(Category)共享目录结构;
**不能新建子术语;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703707477-c029193e-9cc3-4c92-8e26-bd4bad0d869f.png#clientId=u022f7a5c-acf4-4&from=paste&height=129&id=u0c70c857&margin=%5Bobject%20Object%5D&name=image.png&originHeight=388&originWidth=1940&originalType=binary&ratio=1&size=102825&status=done&style=none&taskId=u0747ce2d-a4b7-4ed3-9714-c67232d104f&width=646.6666666666666)
**术语下也可以添加分类;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703717642-7cb8b61d-1702-45b4-86bb-7c6aec61a4ab.png#clientId=u022f7a5c-acf4-4&from=paste&height=175&id=u578f4ace&margin=%5Bobject%20Object%5D&name=image.png&originHeight=524&originWidth=1928&originalType=binary&ratio=1&size=137656&status=done&style=none&taskId=u5e738469-7985-42be-b864-2eca68913c2&width=642.6666666666666)
**术语之间可以跨分组关联;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703730292-92ba6d6b-2cfd-443b-8f2c-9fa208b313fa.png#clientId=u022f7a5c-acf4-4&from=paste&height=217&id=u5b894b60&margin=%5Bobject%20Object%5D&name=image.png&originHeight=650&originWidth=1924&originalType=binary&ratio=1&size=201534&status=done&style=none&taskId=uddd6a7ef-261d-4cee-a94e-ed4d6dcbd5a&width=641.3333333333334)
**术语可以添加标签并设置标签的属性值;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703743702-395682bb-1c48-43fa-a287-62fa0e9a4b56.png#clientId=u022f7a5c-acf4-4&from=paste&height=196&id=ua468ded9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=588&originWidth=1922&originalType=binary&ratio=1&size=146896&status=done&style=none&taskId=u754c686b-9274-4054-9d3e-2220b463ebd&width=640.6666666666666)
##### 7.0.4 Entitys关联Terms,Classifications
**分区字段也可以设置terms和Classifications**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703772224-d7f1fa09-6b2e-4b2c-99c0-2ba280730825.png#clientId=u022f7a5c-acf4-4&from=paste&height=548&id=u1f423a44&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1644&originWidth=1934&originalType=binary&ratio=1&size=400126&status=done&style=none&taskId=ua85defb3-0bfc-44d9-a905-3a3d6fc1450&width=644.6666666666666)
#### 7.1基于hive的元数据管理
直接导入:
进入atlas 目录
```shell
cd /usr/hdp/3.1.0.0-78/atlas
export ATLASCPPATH=/usr/hdp/3.1.0.0-78/atlas/config
```
认证用户(可选)
```shell
kinit -kt /etc/security/keytabs/hive.service.keytab hive/emr122.dtwave.com@HADOOP.COM
```
执行
```shell
./hook-bin/import-hive.sh
```
输入atlas用户名密码
```shell
admin/admin
```


![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703800881-02d21fe0-b630-4e72-ae1b-bb14c64cad20.png#clientId=u022f7a5c-acf4-4&from=paste&height=67&id=udd247847&margin=%5Bobject%20Object%5D&name=image.png&originHeight=202&originWidth=1948&originalType=binary&ratio=1&size=141137&status=done&style=none&taskId=u0c6f2e7b-3148-409d-84a2-e85f3b8fe11&width=649.3333333333334)
实时同步:
beeline建表
```shell
jdbc:hive2://emr114.dtwave.com:2181,emr128>  create table qxd1 as select id,name from test1;
```
kafka ATLAS_HOOK消费查看
```shell
$ export  KAFKA_OPTS="-Djava.security.auth.login.config=/usr/hdp/3.1.0.0-78/kafka/config/kafka_jaas.conf"
$ ./kafka-console-consumer.sh --bootstrap-server emr114.dtwave.com:6667 --topic ATLAS_HOOK --consumer.config /usr/hdp/3.1.0.0-78/kafka/config/consumer.properties

consumer.properties添加配置:
security.protocol=SASL_PLAINTEXT
sasl.mechanism=GSSAPI
sasl.kerberos.service.name=kafka
```
消费到数据
```json
{"version":{"version":"1.0.0","versionParts":[1]},"msgCompressionKind":"NONE","msgSplitIdx":1,"msgSplitCount":1,"msgSourceIP":"192.168.90.115","msgCreatedBy":"hive","msgCreationTime":1629785739361,"message":{"type":"ENTITY_CREATE_V2","user":"hive","entities":{"referredEntities":{"-10786364997694454":{"typeName":"hive_storagedesc","attributes":{"qualifiedName":"default.qxd1@emr_dev_storage","storedAsSubDirectories":false,"location":"hdfs://mycluster/warehouse/tablespace/managed/hive/qxd1","compressed":false,"inputFormat":"org.apache.hadoop.hive.ql.io.orc.OrcInputFormat","parameters":{},"outputFormat":"org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat","table":{"guid":"-10786364997694453","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev"}},"serdeInfo":{"typeName":"hive_serde","attributes":{"serializationLib":"org.apache.hadoop.hive.ql.io.orc.OrcSerde","name":null,"parameters":{"serialization.format":"1"}}},"numBuckets":-1},"guid":"-10786364997694454","version":0,"proxy":false},"-10786364997694453":{"typeName":"hive_table","attributes":{"owner":"hive","temporary":false,"lastAccessTime":1629785737000,"qualifiedName":"default.qxd1@emr_dev","columns":[{"guid":"-10786364997694455","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.qxd1.id@emr_dev"}},{"guid":"-10786364997694456","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.qxd1.name@emr_dev"}}],"tableType":"MANAGED_TABLE","sd":{"guid":"-10786364997694454","typeName":"hive_storagedesc","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev_storage"}},"createTime":1629785737000,"name":"qxd1","comment":null,"partitionKeys":[],"parameters":{"totalSize":"305","numRows":"2","rawDataSize":"182","COLUMN_STATS_ACCURATE":{"BASIC_STATS":"true"},"numFiles":"1","transient_lastDdlTime":"1629785738","bucketing_version":"2"},"db":{"guid":"-10786364997694447","typeName":"hive_db","uniqueAttributes":{"qualifiedName":"default@emr_dev"}},"retention":0},"guid":"-10786364997694453","version":0,"proxy":false},"-10786364997694456":{"typeName":"hive_column","attributes":{"owner":"hive","qualifiedName":"default.qxd1.name@emr_dev","name":"name","comment":null,"position":1,"type":"string","table":{"guid":"-10786364997694453","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev"}}},"guid":"-10786364997694456","version":0,"proxy":false},"-10786364997694455":{"typeName":"hive_column","attributes":{"owner":"hive","qualifiedName":"default.qxd1.id@emr_dev","name":"id","comment":null,"position":0,"type":"int","table":{"guid":"-10786364997694453","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev"}}},"guid":"-10786364997694455","version":0,"proxy":false},"-10786364997694447":{"typeName":"hive_db","attributes":{"owner":"public","ownerType":"ROLE","qualifiedName":"default@emr_dev","clusterName":"emr_dev","name":"default","description":"Default Hive database","location":"hdfs://mycluster/warehouse/tablespace/managed/hive","parameters":{}},"guid":"-10786364997694447","version":0,"proxy":false},"-10786364997694449":{"typeName":"hive_storagedesc","attributes":{"qualifiedName":"default.test1@emr_dev_storage","storedAsSubDirectories":false,"location":"hdfs://mycluster/warehouse/tablespace/managed/hive/test1","compressed":false,"inputFormat":"org.apache.hadoop.hive.ql.io.orc.OrcInputFormat","parameters":{},"outputFormat":"org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat","table":{"guid":"-10786364997694448","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev"}},"serdeInfo":{"typeName":"hive_serde","attributes":{"serializationLib":"org.apache.hadoop.hive.ql.io.orc.OrcSerde","name":null,"parameters":{"serialization.format":"1"}}},"numBuckets":-1},"guid":"-10786364997694449","version":0,"proxy":false},"-10786364997694448":{"typeName":"hive_table","attributes":{"owner":"hive","temporary":false,"lastAccessTime":1626402002000,"qualifiedName":"default.test1@emr_dev","columns":[{"guid":"-10786364997694450","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.test1.id@emr_dev"}},{"guid":"-10786364997694451","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.test1.name@emr_dev"}},{"guid":"-10786364997694452","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.test1.address@emr_dev"}}],"tableType":"MANAGED_TABLE","sd":{"guid":"-10786364997694449","typeName":"hive_storagedesc","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev_storage"}},"createTime":1626402002000,"name":"test1","comment":null,"partitionKeys":[],"parameters":{"totalSize":"776","numRows":"2","rawDataSize":"368","COLUMN_STATS_ACCURATE":{"BASIC_STATS":"true","COLUMN_STATS":{"address":"true","id":"true","name":"true"}},"numFiles":"2","transient_lastDdlTime":"1626402682","bucketing_version":"2"},"db":{"guid":"-10786364997694447","typeName":"hive_db","uniqueAttributes":{"qualifiedName":"default@emr_dev"}},"retention":0},"guid":"-10786364997694448","version":0,"proxy":false},"-10786364997694450":{"typeName":"hive_column","attributes":{"owner":"hive","qualifiedName":"default.test1.id@emr_dev","name":"id","comment":null,"position":0,"type":"int","table":{"guid":"-10786364997694448","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev"}}},"guid":"-10786364997694450","version":0,"proxy":false},"-10786364997694452":{"typeName":"hive_column","attributes":{"owner":"hive","qualifiedName":"default.test1.address@emr_dev","name":"address","comment":null,"position":2,"type":"string","table":{"guid":"-10786364997694448","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev"}}},"guid":"-10786364997694452","version":0,"proxy":false},"-10786364997694451":{"typeName":"hive_column","attributes":{"owner":"hive","qualifiedName":"default.test1.name@emr_dev","name":"name","comment":null,"position":1,"type":"string","table":{"guid":"-10786364997694448","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev"}}},"guid":"-10786364997694451","version":0,"proxy":false}},"entities":[{"typeName":"hive_process","attributes":{"outputs":[{"guid":"-10786364997694453","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev"}}],"recentQueries":["create table qxd1 as select id,name from test1"],"qualifiedName":"default.qxd1@emr_dev:1629785737000","inputs":[{"guid":"-10786364997694448","typeName":"hive_table","uniqueAttributes":{"qualifiedName":"default.test1@emr_dev"}}],"name":"create table qxd1 as select id,name from test1","queryText":"create table qxd1 as select id,name from test1","operationType":"CREATETABLE_AS_SELECT","startTime":1629785706179,"queryPlan":"Not Supported","endTime":1629785739349,"userName":"hive","queryId":"hive_20210824141506_d558d78f-ea58-4eb3-8edb-2528327aeaff"},"guid":"-10786364997694457","version":0,"proxy":false},{"typeName":"hive_column_lineage","attributes":{"outputs":[{"guid":"-10786364997694455","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.qxd1.id@emr_dev"}}],"expression":null,"qualifiedName":"default.qxd1@emr_dev:1629785737000:id","inputs":[{"guid":"-10786364997694450","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.test1.id@emr_dev"}}],"query":{"guid":"-10786364997694457","typeName":"hive_process","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev:1629785737000"}},"name":"create table qxd1 as select id,name from test1:id","depenendencyType":"SIMPLE"},"guid":"-10786364997694458","version":0,"proxy":false},{"typeName":"hive_column_lineage","attributes":{"outputs":[{"guid":"-10786364997694456","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.qxd1.name@emr_dev"}}],"expression":null,"qualifiedName":"default.qxd1@emr_dev:1629785737000:name","inputs":[{"guid":"-10786364997694451","typeName":"hive_column","uniqueAttributes":{"qualifiedName":"default.test1.name@emr_dev"}}],"query":{"guid":"-10786364997694457","typeName":"hive_process","uniqueAttributes":{"qualifiedName":"default.qxd1@emr_dev:1629785737000"}},"name":"create table qxd1 as select id,name from test1:name","depenendencyType":"SIMPLE"},"guid":"-10786364997694459","version":0,"proxy":false}]}}}
```
#### 7.2基于hive的血缘
##### 7.2.1 表生层血缘
```sql
load data inpath '/atlasdemo/atlas/atlas_order.txt' overwrite into table tmp_atlas_hive_order.atlas_order;
insert OVERWRITE table atlas_hive_order.atlas_order select * from tmp_atlas_hive_order.atlas_order;
```
表血缘
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703839459-f4efafca-a0f2-4aed-8794-a0f144e52bad.png#clientId=u022f7a5c-acf4-4&from=paste&height=303&id=uc78fa5c1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=910&originWidth=1924&originalType=binary&ratio=1&size=443766&status=done&style=none&taskId=u4738c345-69d2-45e6-a975-e637b739f1e&width=641.3333333333334)
字段血缘
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703855437-08ee4ba0-eb53-4259-9692-84244f1d901a.png#clientId=u022f7a5c-acf4-4&from=paste&height=268&id=u35f8c39d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=804&originWidth=1932&originalType=binary&ratio=1&size=470523&status=done&style=none&taskId=ucbe487b2-4259-44b5-84ef-f29cfaa2c9d&width=644)
##### 7.2.2 视图血缘
```sql
create view atlas_hive_order.order_product_view as SELECT o.order_id,op.product_id,p.product_name FROM atlas_hive_order.atlas_order o JOIN atlas_hive_order.atlas_order_products op ON op.order_id = o.order_id LEFT JOIN atlas_hive_product.atlas_product p ON op.product_id=p.product_id order by o.order_id;
```


![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703892692-9b69043d-a7aa-44fd-83c1-2667a3bfb49c.png#clientId=u022f7a5c-acf4-4&from=paste&height=691&id=u92fda697&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2072&originWidth=1916&originalType=binary&ratio=1&size=551090&status=done&style=none&taskId=u6907d0bb-f882-4912-bb61-961122e4b4a&width=638.6666666666666)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703910856-90d6aafe-bb99-42c3-9145-e83521362867.png#clientId=u022f7a5c-acf4-4&from=paste&height=376&id=uaee56cb5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1128&originWidth=1924&originalType=binary&ratio=1&size=271344&status=done&style=none&taskId=ud6f48449-ba19-46b5-8877-280e54404cb&width=641.3333333333334)
#### 7.3atlas与ranger结合
**新建组P_GROUP,  新建三个用户 P1_USER,P2_USER,P3_USER。 并生成keytab。**
```
ktadd -k /etc/security/keytabs/P1_USER.keytab -norandkey P1_USER@HADOOP.COM
ktadd -k /etc/security/keytabs/P2_USER.keytab -norandkey P2_USER@HADOOP.COM
ktadd -k /etc/security/keytabs/P3_USER.keytab -norandkey P3_USER@HADOOP.COM
```
**新建ranger 访问策略:P1用户只能访问P1 TAG数据;P2用户只能访问P2 TAG数据;**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703939969-6ef87d16-5688-46a9-8b7b-aac43bd5c144.png#clientId=u022f7a5c-acf4-4&from=paste&height=679&id=u7e2a9a51&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2036&originWidth=1924&originalType=binary&ratio=1&size=361181&status=done&style=none&taskId=u43ab1183-fff2-4c04-a07b-0e56d70859c&width=641.3333333333334)
**新建ranger masking策略: P3 用户访问P1 tag。做hash**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703957888-4628b1cc-a42b-4af9-a883-b7f60dead3d5.png#clientId=u022f7a5c-acf4-4&from=paste&height=298&id=ub274c2b8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=894&originWidth=1942&originalType=binary&ratio=1&size=129970&status=done&style=none&taskId=ucb0fa007-875d-4607-8cb4-7dac09ac0ad&width=647.3333333333334)
##### 测试1:
P1_USER: 对于hive来说，表的tag优先级高于字段的tag优先级;如下
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703979346-dce9bcde-04ee-4a3a-9b76-567a4a4a65c5.png#clientId=u022f7a5c-acf4-4&from=paste&height=452&id=u4258447d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1356&originWidth=1936&originalType=binary&ratio=1&size=478927&status=done&style=none&taskId=u46ec5aa5-a5cb-442d-9319-8cd06cad195&width=645.3333333333334)
##### 测试2：
P1_USER:  P3级别的表，P1级别的字段
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631703998534-06a5fadf-718d-4173-9889-f20d5c15b19d.png#clientId=u022f7a5c-acf4-4&from=paste&height=476&id=udb6534a9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1428&originWidth=1918&originalType=binary&ratio=1&size=498242&status=done&style=none&taskId=u4af426e2-3417-4616-a1a9-cbd2ef056e2&width=639.3333333333334)
##### 测试3:
P2_USER: P3级别的表，P2级别的字段
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631704016865-8afbe137-1d94-4159-ad93-7ef28dff5ca8.png#clientId=u022f7a5c-acf4-4&from=paste&height=183&id=u6aff1fa9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=550&originWidth=1918&originalType=binary&ratio=1&size=317462&status=done&style=none&taskId=u03f92d8b-f8dc-46c6-801c-62feae20303&width=639.3333333333334)
##### 测试4:
P3_USER: mobile字段hash过滤
![image.png](https://cdn.nlark.com/yuque/0/2021/png/544845/1631704038841-860ab897-ccbf-44a1-a3de-34b20bf79968.png#clientId=u022f7a5c-acf4-4&from=paste&height=481&id=u9f09eaa2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1444&originWidth=1934&originalType=binary&ratio=1&size=541048&status=done&style=none&taskId=u84f3cef7-8c4f-4f36-afb2-6f46890444b&width=644.6666666666666)
**tip: tag设置应该是金字塔型的，最低的表权限到字段最高的权限控制。上面的权限P级建的不好，没有体现出层次性。**
#### 7.4基于hive的ranger 策略合并
会同时抽取hive policy 也会抽取hive服务挂载的tag服务的策略。

[TOC]



## 第1章Ranger概述

### 1.1 什么是Ranger

​		Apache Ranger是一个Hadoop平台上的全方位数据安全管理框架，它可以为整个Hadoop生态系统提供全面的安全管理

​		随着企业业务的扩展，企业可能在多用户环境中运行多个工作任务，这就需要一个可以对安全策略进行集中管理，配置和监控用户访问的框架，Ranger由此产生！

​		Ranger的官网：https://ranger.apache.org/

### 1.2Ranger的目标

- 允许用户使用UI或REST API对所有和安全相关的任务进行集中化的管理
- 允许用户使用一个管理工具对操作Hadoop体系中的组件和工具的行为进行细粒度的授权
- 支持Hadoop体系中各个组件的授权认证标准
- 增强了对不同业务场景需求的授权方式支持，例如基于角色的授权或基于属性的授权

- 支持对Hadoop组件所有涉及安全的审计行为的集中化管理

### 1.3Ranger支持的框架

Apache Hadoop、Apache Hive、Apache HBase、Apache Storm、Apache Knox、Apache Solr、Apache Kafka、YARN、NIFI

### 1.4Ranger的架构

![](https://yingziimage.oss-cn-beijing.aliyuncs.com/img/202206292025329.png)

### 1.5Ranger的工作原理

​		Ranager的核心是Web应用程序，也称为RangerAdmin模块，此模块由管理策略，审计日志和报告等三部分组成。

​		管理角色的用户可以通过RangerAdmin提供的web界面或REST APIS来定制安全策略。这些策略会由Ranger提供的轻量级的针对不同Hadoop体系中组件的插件来执行。插件会在Hadoop的不同组件的核心进程启动后，启动对应的插件进程来进行安全管理！


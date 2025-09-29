---
title: 苍穹外卖技术栈回顾
published: 2025-09-29
description: "简略介绍苍穹外卖中使用到的技术栈"
image: ""
tags: ["sky-take-out"]
category: demo
draft: false
---

# 苍穹外卖项目技术栈解析

本文档将深入分析“苍穹外卖”项目的代码实现，全面且清晰地介绍其所使用的各项技术及其在项目中的具体应用。

### 1. 项目结构

项目采用标准的多模块Maven结构，职责清晰：

- __`sky-pojo`__: 定义数据实体（Entity）、数据传输对象（DTO）和视图对象（VO），是项目的数据模型层。
- __`sky-common`__: 存放通用工具类、常量、自定义异常、枚举等公共代码，供其他模块复用。
- __`sky-server`__: 项目的核心业务模块，包含Controller、Service、Mapper等所有业务逻辑的实现。

### 2. 核心框架与Web配置

- __Spring Boot__: 作为项目的基础框架，整合了Spring MVC、依赖注入等核心功能，并通过内嵌的Tomcat服务器，使得项目可以打包成可执行的JAR文件独立运行。

- __Web配置 (`WebMvcConfiguration`)__:

  - __拦截器__: 配置了两个独立的JWT拦截器（`JwtTokenAdminInterceptor` 和 `JwtTokenUserInterceptor`），分别对管理端 (`/admin/**`) 和用户端 (`/user/**`) 的API请求进行身份认证，实现了清晰的权限控制。
  - __JSON序列化__: 扩展了Spring MVC的消息转换器，使用自定义的`JacksonObjectMapper`来统一处理日期时间格式，确保了前后端数据交互时格式的准确性。
  - __API文档__: 通过集成Knife4j，并分别为管理端和用户端配置了独立的文档分组，生成了结构清晰、便于查阅和测试的RESTful API文档。

### 3. 数据存储与访问

- __MySQL__: 作为主数据库，存储所有业务数据。

- __MyBatis__: 持久层框架，通过XML文件管理SQL语句，将SQL与Java代码解耦，并允许开发者进行精细的SQL优化。

- __Druid__: 高性能数据库连接池，提供连接管理和监控功能，保障数据库访问的稳定高效。

- __Redis__:

  - __作用__: 作为缓存中间件，主要用于缓存用户端频繁访问的数据，如套餐、菜品分类等。
  - __实现__: 在`RedisConfiguration`中配置了RedisTemplate，并自定义了序列化方式，确保Key的可读性。在业务代码中，采用“先查缓存，缓存未命中则查数据库，再将结果写入缓存”的经典缓存策略，有效降低了数据库压力。

- __PageHelper__: MyBatis分页插件，通过简单的配置和调用，无侵入地实现了后台管理系统的分页查询功能。

### 4. 业务功能的亮点实现

- __公共字段自动填充 (AOP)__:

  - __技术__: Spring AOP + 自定义注解。
  - __实现__: 定义了`@AutoFill`注解和`autoFillAspect`切面。在Mapper层的insert或update方法上使用`@AutoFill`注解后，切面会在方法执行前，通&#x8FC7;__&#x53CD;&#x5C04;__&#x81EA;动为实体对象填充`createTime`、`createUser`、`updateTime`、`updateUser`等公共字段。这极大地简化了代码，避免了在每个业务方法中编写重复的赋值逻辑。

- __用户认证 (JWT)__:

  - __技术__: JSON Web Token。
  - __实现__: 用户登录成功后，服务器使用`jjwt`库生成一个包含用户ID的Token。客户端在后续请求的请求头中携带此Token。服务器端的JWT拦截器会捕获请求，验证Token的合法性，并将解析出的用户ID存入基于`ThreadLocal`实现的`BaseContext`中，供当前线程的后续业务逻辑使用。

- __实时消息推送 (WebSocket)__:

  - __技术__: `javax.websocket` API。
  - __实现__: 创建了一个`WebSocketServer`服务端点，通过一个静态Map管理所有客户端连接。当业务系统（如订单服务）需要推送消息时（例如“用户已催单”），会调用`WebSocketServer`的群发方法，将消息实时推送给所有在线的管理端客户端，实现了高效的实时提醒功能。

- __定时任务处理__:

  - __技术__: Spring Task。
  - __实现__: 在`Ordertask`类中使用`@Scheduled`注解定义了定时任务，例如“每小时检查并自动取消超时的未支付订单”。Spring Task会按照注解中定义的cron表达式，周期性地执行这些任务。

### 5. 第三方服务集成

- __阿里云OSS__: 用于存储图片等静态文件。代码中封装了`AliOssUtil`工具类，简化了文件的上传操作。
- __微信支付__: 集成了微信支付的API，实现了用户在线下单支付的功能。
- __Apache POI__: 用于后台报表导出功能，将运营数据生成为Excel文件供下载。

### 总结

“苍穹外卖”项目是一个典型的、技术栈成熟的Java后端应用。它不仅综合运用了Spring Boot生态中的各项主流技术，还在具体业务场景中通过AOP、JWT、WebSocket等技术给出了优雅且高效的解决方案，是学习和实践Java Web开发的优秀案例。

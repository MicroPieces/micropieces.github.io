---
layout:     post
title:      「玩转Apollo」 之 Client篇
subtitle:   Spring boot 整合 Apollo 客户端配置流程
date:       2018-12-05
author:     FF
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Apollo
---

> 正所谓前人栽树，后人乘凉。
> 
> 感谢 自己（原创）

# 前言

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。
Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。
.Net客户端不依赖任何框架，能够运行于所有.Net运行时环境。
如果想要深入了解，可以到github上参见[Apollo配置中心](https://github.com/ctripcorp/apollo#screenshots)，官网的介绍很详细。本章主要讲述Spring Boot 2.0 整合Apollo配置中心。


废话不多说了，开始进入正文。

# 快速开始

### 1.添加Apollo客户端依赖

```
<dependency>
     <groupId>com.ctrip.framework.apollo</groupId>
     <artifactId>apollo-client</artifactId>
     <version>1.1.0</version>
</dependency>
```
1.1.0目前是社区用得最多的，所以我们用1.0.0
  
### 2.添加配置文件

##### 2.1 在项目此路径下创建app.properties文件，classpath:/META-INF/app.properties文件。
[![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/e.3PIPmHtaL6BFVJorAw5seiqhM9yt2lb91VAhu4DT0!/b/dD4BAAAAAAAA&bo=TAJsAQAAAAADBwE!&rf=viewer_4)]

并且其中内容app.id为必填：
```
#服务ID
app.id=realtyOrderService
#apollo服务端-configservice的地址
apollo.meta=http://192.168.0.30:30080

# 指定apollo配置缓冲路径，默认为 linux: /opt/data/{appId}/config-cache Windows: C:\opt\data\{appId}\config-cache
# apollo.cacheDir=/opt/data/some-cache-dir
#设置集群
#apollo.cluster=SomeCluster
```

##### 2.2 application.properties ( 非必须)

[![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/Y*XKDIT7lkFBxUPRtnfaEKERTvNgkhbNIjAPEl1VENI!/b/dFUAAAAAAAAA&bo=QwKSAAAAAAADB*E!&rf=viewer_4)]

-因为考虑到spring.application.name和server.port这类属性配置，在同一机器上部署时，AppId相同，但是配置会不同，所以这类配置，仍然放到配置文件中在打包时指定。 
-需要注意的是若工程中application.properties存在，且其中有配置属性，apollo客户端会默认从配置中心取值，若出现key在配置中心和中application.properties都存在的情况，apollo客户端是不会读取application.properties中的配置。
```
#apollo.bootstrap.enabled = true
#FX.apollo指fx部门编码.
#apollo.bootstrap.namespaces = application,FX.apollo
```
 

### 3. 添加@EnableApolloConfig注解

在SpringBoot的入口类(SpringApplication)上添加注解 @EnableApolloConfig

[![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/37ZJYL0imrUeWrRjINh.WqdX69NHwEhMPqQLIXdZ5.4!/b/dLYAAAAAAAAA&bo=ygPcAAAAAAADBzc!&rf=viewer_4)]

### 4. 添加配置信息监听器

设置监听
```
import com.ctrip.framework.apollo.Config;
import com.ctrip.framework.apollo.model.ConfigChangeEvent;
import com.ctrip.framework.apollo.spring.annotation.ApolloConfig;
import com.ctrip.framework.apollo.spring.annotation.ApolloConfigChangeListener;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * author:傅枫
 * Date:2018/12/5  15:24
 *
 * @desc
 */
@Component
public class SpringBootApolloRefreshConfig {

    @ApolloConfig
    private Config config;

    private static final Logger logger = LoggerFactory.getLogger(SpringBootApolloRefreshConfig.class);

    @ApolloConfigChangeListener
    private void someOnChange(ConfigChangeEvent changeEvent) {
        for (String changedKey : changeEvent.changedKeys()) {
            logger.info(changedKey+"="+config.getProperty(changedKey,"null"));
        }
    }

}
```


### 5. 在服务端创建项目
 
##### 5.1 登录apollo服务 LoginURL：http://192.168.0.30:8070
 
 默认帐号： apollo / admin
 [![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/lcgYu26hnkNJ*JgY6sgCZtS6lgdhQLYu8UktTKfNyCY!/b/dLYAAAAAAAAA&bo=WQP.AQAAAAADB4c!&rf=viewer_4)]

##### 5.2 创建项目

点击创建项目，应用ID与你app.properties里的app.id必须一致
 [![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/Gj3Y6YWDxjtcbuRdsYTIXdSBsUlpgxbqQLzwObaQlKU!/b/dFMBAAAAAAAA&bo=UgYqAgAAAAADB14!&rf=viewer_4)]


最后进入你创建的项目，添加配置再发布，然后你基于apollo管理的配置客户端就可以run起来了。

### 6. 调试

在apollo配置管理平台修改发布配置信息
 [![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/WksfXqvD0Vymqmzga24HDkn9OnUHcfoIJ*lvcZPISWc!/b/dLYAAAAAAAAA&bo=zQPzAQAAAAADBx4!&rf=viewer_4)]
 
在客户端的监听器会输出change的信息
 [![](http://m.qpic.cn/psb?/V11oTtVQ2pcC6W/rGGfLFO2.Lqc*KNOHVm0MlaaZjJktoo6mZnHWsDk.jU!/b/dDUBAAAAAAAA&bo=VgapAgAAAAADB9k!&rf=viewer_4)]




以上所述给大家介绍了spring boot 阿波罗 客户端配置与调试过程

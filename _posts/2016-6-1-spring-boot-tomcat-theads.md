---
layout: post
title: Spring Boot 减少 tomcat 线程开销
---

最近在我的电脑上(mba 11')实验 spring cloud 相关的项目, 发现在起了几个 spring boot 的实例之后, 电脑风扇就开始狂转, 我只是起服务器, 没有做大量的计算啊? 后来 Debug 看到 Tomcat 竟然开启了4, 500个进程, 看来 spring boot 默认是为线上产品配置的. 把 tomcat 线程减小之后, 风扇就不那么转了, :smiley:

解决方法: application.yml 添加

    server:
      tomcat:
        max-threads: 20

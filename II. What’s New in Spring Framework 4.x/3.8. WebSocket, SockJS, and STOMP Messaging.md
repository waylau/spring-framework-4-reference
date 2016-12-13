WebSocket, SockJS, 和 STOMP 消息
=========

一个新的 `spring-websocket` 模块提供了全面的基于 WebSocket 和在Web 应用的客户端和服务器之间双向通信的支持。它和 Java WebSocket API [JSR-356](http://jcp.org/en/jsr/detail?id=356) 兼容，此外还提供了当浏览器不支持 WebSocket 协议时 (如 Internet Explorer < 10)的基于SockJS 的备用选项(如 WebSocket emulation)。

一个新的`spring-messaging`模块添加了支持 STOMP 作为 WebSocket 子协议用于在应用中使用注解编程模型路由和处理从 WebSocket 客户端发送的 STOMP 消息。因此`@Controller`现在可以同时包含`@RequestMapping`和`@MessageMapping`方法用于处理 HTTP 请求和来自 WebSocket 连接客户端发送的消息。新的`spring-messaging`模块还包含了来自以前 [Spring 集成](http://projects.spring.io/spring-integration/)项目的关键抽象，例如`Message`、`MessageChannel`、`MessageHandler`等等, 以此作为基于消息传递的应用程序的基础。

欲知详情以及较全面的介绍，请参见Chapter 20, WebSocket 支持一节。

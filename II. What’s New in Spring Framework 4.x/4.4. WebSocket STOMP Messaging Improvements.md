WebSocket STOMP 消息  改进
========

* SockJS(Java)客户端支持.查看 `SockJsClient` 和在相同包下的其他类.
* 发布新的应用上下文实践 `SessionSubscribeEvent` 和
 `SessionUnsubscribeEvent`, 用于STOMP客户端的订阅和取消订阅.
* 新的"websocket"级别.查看 [Section 25.4.14, “WebSocket Scope”.](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-stomp-websocket-scope)
* `@SendToUser` 仅仅靠一个会话就可以命中, 而不一定需要一个授权的用户.
* `@MessageMapping` 方法使用 **.** 来代替 **/** 作为目录分隔符。查看 [SPR-11660.](https://jira.spring.io/browse/SPR-11660)。
* STOMP/WebSocket 监听消息的收集和记录。查看[25.4.16, “Runtime Monitoring”.](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-stomp-stats)。
* 显著优化和改善在调试模式下依然保持日志的可读性和简洁性。
* 优化消息创建，包含对临时消息可变性的支持和避免自动消息ID和时间戳的创建。查看 `MessageHeaderAccessor` 的java文档。
* 在WebSocket会话建立之后的1分钟没有任何活动,则关闭STOMP/WebSocket连接。

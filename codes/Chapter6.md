﻿# 6.响应初始化请求
------
本章节描述端点在收到初始化请求后应该怎么做。
端点需要检查对端是否支持ICE，自己的ICE角色，以及如果在完全实现下生成检查列表和开始一般检查。

当ICE在SIP环境使用时, 分支(forking)可能会导致1个请求却产生了多个应答。在这种情况下, ICE处理过程是并行的，对于每个应答是独立的。ICE会把每个请求/应答
和它们的候选对，检查列表，状态等组合起来，单独处理。唯一的对一个候选对操作会影响到其他候选对的就是候选的释放，这个在8.3小节中讨论。

# 6.1 验证ICE支持

应答者的逻辑和请求者(5.1小节)是不同的，除非请求者从来不会在SDP中生成 a=ice-mismatch。
某些时候，应答会提交a=candidate在该媒体流SDP中，而代替包含a=ice-mismatch属性对1个或多个媒体流。这会通知请求方应答端点支持ICE，但是ICE处理并不会
在这个会话中使用，因为马上就会通知修改媒体组件的默认目的地而不是修改响应的候选属性。小节18描述这种情况如何发生。本SPEC没有设计指导如何处理这样的错误。

# 6.2 决定角色
与5.2小节描述处理相同

# 6.3 生成检查列表
只有完全实现有该步骤。同小节5.7.

# 6.4 执行一般检查
完全实现下执行。同小节5.8




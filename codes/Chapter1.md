# 1. 介绍
------
`[RFC3264] (An Offer/Answer Model with the Session Description Protocol (SDP))` 定义了一种两个阶段的SDP消息交换模型，
以用来建立多媒体会话。这种 请求/应答 机制也被很多其他协议所采用，例如`Session Initiation Protocol (SIP)[RFC3261]`。   

采用请求/应答机制的协议难以穿透NATs`(Network Address Translators)`。这类协议在建立多媒体传输时，总是在消息里直接带上多媒体的源地址和源端口，
在应用层也没有中间层，追求在通讯端点间直接连接。直连可以有效地减少多媒体延迟，减少丢包和减少应用的部署难度。但这种方式在NAT环境下非常难实现，
具体的解释和原因不在本SPEC的讨论范围了。   

已经有很多的解决方案用于使这些协议穿透NAT,包括: `Application Layer Gateways(ALGs)`, `the Middlebox Control Protocol(RFC3303)`, `the original Simple
Traversal of UDP Through NAT (STUN) [RFC3489]`, `Realm Specific IP [RFC3102] [RFC3103]`,`Session Description Protocol (SDP) [RFC4566]`
配合`Real Time Control Protocol(RTCP) [RFC3605]`等等。不过，这些方案在不同的网络拓扑环境下有着不同的适用性，使得应用的实现,部署和配置都必须针对特定的网络拓扑环境来实施。
这样的系统显得复杂和脆弱，需要找到一个灵活通用的方案来应对所有的情况。

本SPEC定义了一种在请求/应答模型上基于UDP穿越NAT来传输媒体数据的技术:`Interactive Connectivity Establishment(ICE)`（**ICE**也可以工作在其他传输协议上，比如TCP）。
**ICE**扩展了请求/应答模型，包含了多组的地址和端口在SDP请求和应答中，并对这些地址和端口进行点对点直连检测。这些地址和端口对的直连检测，使用了升级版的`STUN[RFC5389]`协议。
新的STUN协议作为**ICE**协议的一个工具存在，而不再像旧版定义为一种NAT穿透的解决方案。为了补充STUN协议，ICE也同时使用了TURN协议(`Traversal Using Relays around NAT (TURN) [RFC5766]`)。
**ICE**为了媒体数据传输会交换传输流的多组地址和端口，可以在多个网卡或多个协议族(v4/v6)上选择地址工作，所以**ICE**废弃了`[RFC4091]`和`[RFC4092]`。
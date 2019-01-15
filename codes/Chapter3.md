# 3. 术语
------

Agent: 端点, 请求/响应模型协议的实现者。在请求/响应交换实现中有两个端点。

Peer: 对端，在一个连接会话中, 一个端点的对端就是另外一个端点。请求者的对端就是应答方，应答者的对端就是请求方。

Transport Address: 通讯地址, IP地址和协议(如TCP/UDP)端口的组合对。

Candidate: 候选, 一个媒体通讯的潜在的可用的通讯地址。候选具有“类型,优先级,基准”等属性。

Component: 组件, 指一个媒体流在一个通讯地址上发起的通讯。媒体流可能需要多个组件配合一起工作。例如RTP媒体流，具有两个组件：RTP 和 RTCP。

Host Candidate: 主机候选地址。绑定端口到本地主机地址上的候选。本地地址包括物理地址和逻辑地址, 例如VPN和RSIP地址。

Server Reflexive Candidate: 服务器反射候选。通过向服务器发送消息穿过NAT，而由NAT分配的IP地址和端口对。STUN服务器可以通过绑定请求获取到
服务器发射候选地址，或同时也提供中继候选地址。

Peer Refexive Candidate: 对端发射候选。端点发送STUN绑定消息到对端时，穿过NAT，而由NAT分配的通讯地址。

Relayed Candidate: 中继候选地址。发送TURN分配请求到TURN服务器时，服务器分配的候选地址。中继候选地址在TURN服务上，TURN会中继此候选地址的消息到对应端点。

Base: 基础地址。服务器反射地址的基础地址是派生它的主机候选地址。主机候选地址的基础就是它本身。类似的, 中继候选地址的基础也是它本身。

Foundation: 基础设定。描述两个候选地址是否具有同样的类型，同样的基准IP地址，同样的协议，和同样的 STUN 或 TURN服务器。如果这些不完全相同，则说这两个
地址的基础是不一样的。具有相同基础设定候选地址认为具有相似的网络特征。基础设定在冰冻算法中有很大作用。

Local Candidate: 本地候选。
Remote Canidate: 对端候选。

Default Destination/Candidate: 默认目的地指在媒体流组件中使用的非ICE框架的通讯地址。例如RTP组件中, 默认IP地址在SDP的c行,端口在SDP的m行。RTCP中,要么在
rtcp属性中，要么就是c行的IP地址和m行出现的另外一个端口。默认候选就是匹配了默认目的地的候选地址。

Candidate Pair: 包含了本地候选和对端候选的一对。

Check, Connectivity Check, STUN Check: 为了检验连接性的一个STUN绑定请求事务。一个事务是由候选地址对中本地候选发往对端候选。

Check List: 一个有序的候选地址列表，端点会依照这个列表来生成检查。

Ordinary Check: 普通检查, 端点的定时器触发后引起的检查。

Triggerd Check: 响应检查，由对端的检查请求响应的检查。

Valid List: 一个有序列表，包含已经又STUN事务检查成功的候选地址对列表。

Full: 完整，详细的ICE实现。由本spec定义的。

Lite: 简化的ICE实现。省略了一些功能的，只做那些最必要的能实现ICE的功能。 精简实现不维护状态机和不做连接性检查。

Controlling Agent: 控制端点。 最终选择候选地址对，并通过STUN通知和更新选择的“决策”端点。在会话中，一般只有一个端点是控制端点，而其他的都是被控制端点。

Controlled Agent: 被控制端点。 通讯会话中等待控制端点决定最终候选地址对的端点。

Regular Nomination: 一般策略。 通过一个STUN请求连检查连接性，然后通过第二个STUN请求携带一个提名标记来完成检查这样的过程叫一般策略。

Aggressive Nomination: 激进策略。发送的第一个STUN请求就携带提名标记来检查连接性，这样只要连接成功就完成了检查，这种的过程叫激进策略。

Nominated: 提名完成。一个候选地址对如果提名标记已有效，则表示ICE已经可以选择这对候选地址来收发消息了。

Seleted Pair, Selected Candidate: 被选来ICE通讯的候选地址对和该对地址中的候选地址。

















﻿# 5. 接收初始化请求
------
当端点收到了初始化请求，它会检查对端是否支持ICE，它自己的通讯角色，收集候选地址，排序候选，选择默认候选，编码和发送应答，然后在完全实现下，
它会开始建立候选检查清单开始做连接性检查。

### 5.1 检查ICE支持

端点会对它收到的SDP中的每个候选属性的媒体流的媒体组件开始ICE处理。例如RTP，则是候选属性中c行和m行中的IP地址和端口，和 rtcp属性中的值。

如果这个条件没有达到，则端点会像RFC 3264定义的过程处理SDP，而不使用ICE机制。普通处理过程包括：
+  端点需要遵循小节10中的规则, 对所有端点实现 keepalive 过程。
+  如果端点不支持ICE，它虽然有a=candidate属性，但是媒体流却没有默认目的地，那么回复需要包含 a=ice-mismatch属性。
+  如果默认候选是从TURN服务器分配的中继候选，则端点需要从TURN服务器得到使用对端SDP提供的地址的许可。如果不成功，则对端媒体流发送的初始化数据包则可能丢失。

### 5.2 决定自己的角色

对于通讯会话，每个端点会有一个角色，控制端点或者被控制端点。控制端点会在最后选择进行通信的候选地址对。对于完全实现，这表示控制端点会决定使用哪一个候选对，
并在有需要时生成ICE选择的更新请求。简化实现时，控制端点会从候选地址对里选择一个，并在需要时生成更新请求来通知这个选择。而被控制节点则是等待候选地址对的
选择通知，不需要生成更新请求。下面的部分，会详细描述控制端点和被控制端点的处理过程。

决定角色的规则和行为效果如下：

双方都是完全实现端点：
请求发起方会成为控制端点，其余的是被控制端点。双方都会建立候选列表，运行ICE状态机和进行连接检查。控制节点会按照 8.1节 的规则来决定通讯使用的候选地址对，
然后双方按照 8.1.2小节的规则来终止ICE。在一些不常见的情况下，如 附件B.11，有可能双方节点都认为自己是控制端点或被控制端点。为了解决这种情况，双方需要生成
一个 0～2^64 - 1间的随机数(64位正整数)，称为tie-breaker。按照7.1.2.2小节, 这个数会在连接检查中使用，用来检查和修复这种情况。

一端是完全实现 + 一端是简化实现：
完全实现端点需要成为控制端点，而简化实现端点为被控制端点。完全端点会生成候选列表，运行ICE状态机，和进行连接性检查。这个端点会决定使用哪个候选地址对(8.1小节),
和结束ICE过程（8.1.2小节). 而简化端点只需要监听连接性检查，做出响应和结束ICE（8.2小节).对于简化实现，每个媒体流的ICE处理都认为在运行，并且整个ICE的状态也
是在正在运行。

双方都是简化实现端点:
生成初始化请求而开始ICE处理的端点是控制端点，其他的则是被控制端点。这个情况下，不会有连接性检查。因此，只要请求/响应消息交换完成，双方端点就开始了小节8定义的处理过程，
只是没有连接检查。有可能双方都认为它们是控制或被控制端点。这个冲突会在后面的请求/响应的消息协议中被监测到和解决。而每个ICE媒体流都认为是在运行状态，
整个ICE也是正在运行状态。

只要角色一被决定下来，整个ICE过程都不会改变，直到ICE重启。ICE重启(9.1小节)会导致角色和tie-breakers的重新生成。


### 5.3 收集候选

对于响应端点，候选地址的收集过程和请求方一样(4.1.1小节和4.2小节)。建议候选收集处理在收到请求时就开始，优先于向上层通知。这样收集处理可能在端点一开始就进行了。

### 5.4 排序候选

排序候选地址的过程和请求方也是一样的(4.1.2小节和4.2小节)。

### 5.5 选择默认候选

同请求方。(4.1.4 和 4.2小节)。

### 5.6 编码SDP

同请求方。(4.3小节)

### 5.7 生成检查列表

检查列表只有完全实现才需要，简化实现需要跳过这一步。

每个媒体流都需要生成一个检查列表。端点需要生成候选地址对，计算候选对优先级，按优先级排序，修剪，以及设定它们的状态。这些步骤在本章节接下来会描述。

### 5.7.1 生成候选地址对

首先，端点把媒体流的每个候选地址(本地候选)和对端消息的每个候选地址(远程候选)配对。端点需要限制每个请求/响应中包含的候选数量，以避免安全攻击(18.5.2小节)。
只有当两个候选具有相同的组件ID和同样的IP地址版本时，一个本地候选和一个远程候选才进行配对。这样有可能有些本地候选或远程候选是不会出现在配对候选地址中的。
比如一个端点为媒体流所有组件获取到候选地址。如果这种情况发生，则这个媒体流下的组件数量就会减少，等于可以进行配对的组件数量。

在RTP流中，这发生一个端点提供了RTCP候选，而另一个没有。另外一个例子，请求方可能把RTP和RTCP分配在同一个端口，并在SDP中通过SDP属性告知(RFC 5761)，
但是由于请求方不知道应答方能不能支持，所以请求方会在不同的端口建立RTP和RTCP，这样请求方就有两个组件。如果应答方支持端口混合，它就会在每个候选只生成一个组件。
ICE在这个候选只有一个组件时就会结束候选配对。

本地候选和远程候选都是默认候选地址的那个配对成为默认候选地址对。这个地址对在双方不支持ICE时将用来传输媒体数据。

图6，提供了一些图例来帮助理解这些名词定义。

![Figure6](https://github.com/dryad60/RFC5254_Translate/blob/master/images/Figure6.png?raw=true) ***图6***  


### 5.7.2 计算候选对优先级和排序

计算优先级时，设定 G 是控制端点的候选地址的优先级，D是被控制端点的候选优先级，那么这组候选对的优先级是:

pair priority = 2^32 * MIN(G,D) + 2 * MAX(G,D) + (G > D ? 1 : 0);

有了优先级后，按从高到低排列。如果两组候选对的优先级相同，那么它们之间的顺序可以任意。

### 5.7.3 修剪候选对

排序后的候选对列表接下来会用来做连接性检查。每个检查是用本地候选发送一个请求到远程候选，但是端点是无法从反射候选发送消息的，只能从反射候选的基础地址。所以
那些本地候选是服务反射候选的，需要替换为反射候选的基础地址。替换之后，就需要对候选对列表进行修剪。修剪的方法就是需要删除那些本地候选和远程候选与比它优先级高的
某个候选对相同的候选地址对。修剪完成就得到了媒体流的排序候选地址对序列。

为了避免攻击(18.5.2小节), 端点需要限制所有检查列表的连接检查数的总和低于一个设定的值，建议这个值是100。这个限制的做法就是抛弃那些低优先级的候选对知道满足
低于100这个限制。建议这个数值默认应该很小，而在配置时可以根据情况调大。对于配置的需求就是在当部署后发现这个数值有问题时，可以有工具修复这个问题。


### 5.7.4 计算处理状态

每个位于检查列表的候选对都有基础设定和状态。候选对的基础设定就是本地和远端候选的基础设定集合。状态值有5种：
+ 等待(waiting): 还未开始检查，当该候选对是列表中最高优先级时立即开始处理
+ 处理中(In-Progress):检查已经发送，正在进行处理
+ 成功(Succeeded):检查已经完成，结果是成功
+ 失败(Failed):检查已经完成，但结果是失败
+ 冻结(Frozen):检查完成并失败，且不会再产生任何处理和响应。

在ICE运行中，候选对的状态进行切换如图7:

![Figure7](https://github.com/dryad60/RFC5254_Translate/blob/master/images/Figure7.png?raw=true) ***图7***  

而候选对的初始状态则如下步骤计算：
1. 首先设置所有的候选对为 Frozen;
2. 端点开始检查第一个媒体流的检查列表(第一个媒体流就是出现请求/响应SDP m行中的第一个)。对于这个媒体流:
   ** 对于所有具有相同基础设定的候选对，设置其中最小组件ID的候选对为Waiting. 如果有多个，则选择优先级最高那个。

这样就会有一个检查列表中的一些候选对处于Waiting状态，而其他检查列表中的候选对都是Frozen。有至少一个是Waiting的列表叫活动检查列表，而所有候选对
都是Frozen的列表叫冻结检查列表。

检查列表也有一个状态值，用来表示媒体流的ICE检查状态。有3种:
+ 运行中(Running):该媒体流的ICE检查正在处理中；
+ 完成(Completed):ICE检查已经为每个媒体流决定了候选地址对。即ICE已经成功，可以发送数据了。
+ 失败(Failed): 此媒体流上的ICE检查已失败。
当一个检查列表开始创建用来做请求/应答消息交换时，设定它为Running状态。
    
而在所有媒体流上进行的ICE处理也有状态。如果ICE检查正在处理，则为 Running 状态；结果成功则是Completed或结果失败则是Failed。状态变化的规则如下描述。

### 5.8 检查的时序安排

检查只发生在ICE完全实现，简化实现中没有这个步骤。
端点的检查有一般的检查和触发的检查。这两种检查都由每个媒体流中的定时器生成。端点会维护一个先入先出队列，触发检查队列，包含了那些在下次发送机会时发送检查的候选对。
当定时器触发时，端点从触发检查队列里取出第一个候选对，开始连接检查，把这个对的状态设置为'In-Progress'。当这个队列为空时，会发送一个一般检查。

当端点生成了检查列表(5.7小节)，就要为每个活动的检查列表设置一个定时器。这个定时器定时间隔为Ta * N秒，N为活动的检查列表的数量(一开始只有1个)。在实现时要考虑
定时器不要触发太频繁，并且不同的媒体流不应同时触发。Ta 和 定时器的重置计算在 16小节 中详细描述。乘以 N 是允许这个总计的检查可以分散到所有的活动检查列表。
定时器会立即触发一次，这样连接检查就会在请求/响应交换后立即发生，并在之后每个 Ta 秒再执行一次（只有1个活动列表情况下）。

当定时器触发但没有触发检查执行时，端点需要选择一个一般检查，规则如下：
+ 找到检查列表最高优先级的为Waiting状态的候选对
+ 如果找到了
    * 发送一个STUN检查，从本地候选到远程候选。这个过程是使用一个STUN请求如小节7.1.2描述。
    * 把这个候选对的状态设置为In-Progress
+ 没有找到
    * 重新寻找最高优先级的为Fronzen状态的候选对
    * 如果能找到
        - 解冻候选对
        - 开始检查,状态变为In-Progress
    * 没有找到
        - 列表的定时器关闭。

在计算检查消息的完整性时，端点会使用对端SDP消息中用户名和密码。而本地用户名端点是直接知晓的。
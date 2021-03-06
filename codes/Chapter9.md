﻿# 9 后续的请求/应答交换
------
每个端点都可以在任何时候生成后续的请求参照RFC3264. 章节8中的规则会导致控制端点在发现提名候选对与默认候选对不同时发送更新请求。
本章节定义了后续的请求/应答的规则。

ICE可以继续处理，而忽略掉后续的任何请求。

# 9.1 请求的生成
# 9.1.1 所有实现的处理流程
# 9.1.1.1 重启ICE
端点可以为一个存在的媒体流重启ICE。重启ICE，会使所有以前的ICE处理被终止并重新开始。ICE重启和一个全新的ICE媒体对话的唯一区别是，在重启时，数据依然可以在
此前的有效的通讯候选对上持续收发。

而端点必须重启ICE：
+ 媒体流的目标发生改变。即端点生成了一个更新请求，含有没有使用过的ICE，会导致媒体组件的目的地变成一个新的值。
+ 端点改变了它的实现层。这个一般发生在第三方控制情况下，信号层和媒体数据层区分开，而信号层改变了数据对话的目标到另外一个实例具有不一样的ICE实现。

这些规则使得把c行中的IP地址设置为0.0.0.0就会导致ICE重启。进一步的，ICE实现不能使用通话保持禁止，而应该使用a=inactive和a=sendonly（RFC3264).

重启ICE，端点需要修改媒体流请求中的ice-pwd和ice-ufrag.要注意，允许在请求中使用session-level属性,但是需要在后续的请求中使用和session-level同样的ice-pwd
和ice-ufrag.这不是密码的改变，只是密码表示方法的改变，这不会导致ICE重启。

而SDP剩下的部分就会和媒体流ICE初始请求时一样的（小节4.3）。接下来，候选地址可能包含以前该媒体流上的一些或者全部候选地址，或者都没有，也可能包含一些新的收集
到的候选(小节4.1.1)。

# 9.1.1.2 删除一个媒体流
如果端点通过设置一个媒体流的端口为0来删除了媒体流，那端点不能再为这个媒体流保留任意候选属性，也不要在保留任何ICE相关属性(章节15).


# 9.1.1.2 增加一个媒体流
当端点想增加一个媒体流时，在SDP中像初始请求一样填充属性值（小节4.3）。这会让ICE在这个媒体流上开始处理。


# 9.1.2 完全实现的流程
本章节描述了完全实现下对于已存在的媒体流的完全实现。

用户名片段，密码和实现层次必须和以前的一致。如果端点需要改变这些属性，就必须在这个媒体流上重启ICE。额外的行为取决于媒体流上ICE处理过程。

# 9.1.2.1 ICE运行状态下的媒体流
如果端点打算给已经连接成功，ICE检查是RUNNING状态的媒体流生成更新请求，则遵循以下的流程定义。

端点必须包含所有的在上次通知中的本地候选在候选属性中。这些候选在SDP中属性，如优先级，基础设定，类型和关联的传输地址，应该和原来一样。候选的自身标识
IP地址，端口和传输协议，应该和原来一样（如果这些发生了改变，则应该是一个新的候选）。组件ID也必须一样。组件可能提供额外的以前没有的候选，在最后一次请求/应答
交换后收集到的，包括对端反射候选。

端点可能改变媒体数据的默认目的地。这样的话在初始化请求中，必须包含匹配这个目的地的一组候选属性。

# 9.1.2.2 ICE完成状态下的媒体流
如果端点打算给已经连接成功，ICE检查是Completed状态的媒体流生成更新请求，则遵循以下的流程定义。

媒体数据的默认目的地(比如媒体流SDP中m行和c行的IP地址和端口)必须是每个组件的有效列表中的最高优先级的提名候选对。这样可以保证媒体数据的默认目的地等于ICE为媒体
数据选择的目的地。

端点必须给每个媒体流上每个组件的匹配默认目的地的候选包含候选属性，而不能包含其他候选的属性。

额外的，如果端点是控制端点，那么它还必须给每个检查列表是Completed状态的媒体流包含a=remote-candidates属性。这个属性包含了每个媒体流的每个组件的有效列表里
最高优先级的远程候选。这是需要的，以避免控制端点在选择候选对时的竞争状态，但是对被控制端点来说，更新请求会越过连接检查，因为被控制端点是不知道这些候选对的。
查看附件B.6有关于这个竞争状态的详细描述。

# 9.1.3 简化实现的过程
# 9.1.3.1 ICE运行状态下的媒体流
本节描述对于处于ICE Running状态下的媒体流的规则。
简化实现必须在后续请求中的a=candidate属性中为每个媒体流的每个组件携带所有的候选。这些候选像初始请求一样独立生成，参照小节4.2。
简化实现不能在后续请求中添加新的额外的本地候选。如果想要添加，则必须重启ICE。
用户名片段，密码和实现层次必须和以前的保持一直。如果需要改变，则必须重启ICE。

# 9.1.3.2 ICE完成状态下的媒体流
如果媒体流的ICE状态是Completed, 那媒体流的默认目的地必须是媒体流组件的有限列表中的候选对的远程候选地址。对于简化实现来说，一个媒体流组件下的有效列表通常只有
一个有效候选对。额外的，端点必须对每个默认目的地赋予一个候选属性。

进一步说，如果端点是控制端点(只会在双方都是简化实现时), 端点必须为每个媒体流包含a=remote-candidates属性。这个属性包含了有限列表里的有效候选对的远程候选。

# 9.2 接收请求和生成响应
# 9.2.1 所有实现下的流程
当收到已经存在的会话的后续请求时，端点必须不依赖任何以前的请求/应答消息下重新进行验证,小节5.1.。事实上，以前的消息交换结果不会再被直接使用，而是作为后续消息的一个
依据来使用。

# 9.2.1.1 侦测ICE重启
如果对端请求中a=ice-ufrage或a=ice-pwd属性比起以前的SDP发生了变化，则表示这个媒体流上要重启ICE。如果所有媒体流都要重启，则ICE整体全部重启。

如果ICE要重启一个媒体流:
+ 端点必须修改响应中a=ice-ufrag和a=ice-pwd属性
+ 端点可能在响应中改变它的实现层级
其余剩下的SDP的属性值像初始响应一样进行设置（小节4.3). 而候选集合可能包括一些或全部，或不包含以前的候选，也可能包含一些新收集的候选地址（小节4.1.1）。

# 9.2.1.2 新建媒体流
如果请求中包含了一个新的媒体流，端点在响应时要设置一个字段表示已经接收到了这个请求(小节4.3)。这要使ICE开始处理这个新的媒体流。

# 9.2.1.3 移除媒体流
如果请求中包含了一个端口为0的媒体流，则端点在响应中一定不能包含这个媒体流的任何候选属性，也不能包含任何ICE-related属性(小节15)。

# 9.2.2 完全实现下的流程
除非监测到ICE重启，端点必须保持用户名片段，密码和实现层级和以前一致。如果要改变这些属性，则端点需要生成一个请求重启ICE；ICE重启是不会出现在响应消息里的。
额外的处理行为依赖媒体流的状态。

# 9.2.2.1 正在运行状态的媒体流且没有远程候选
如果一个媒体流正在运行，并且请求中缺少这个媒体流的远程候选属性，创建响应的规则和小节9.1.2.1一样。

# 9.2.2.1 完成状态的媒体流且没有远程候选
如果媒体流在完成状态，且请求中缺少媒体流的远程候选属性, 响应的创建规则和小节9.1.2.2大体一致，唯有不同是响应里一定不能包含a=remote-candidates属性

# 9.2.2.2 已存在的媒体流并含有远程候选
被控制端点会接受到对端请求含有a=remote-candidates属性，当对端要结束媒体流上的ICE处理时。这个属性在请求中是用来处理接收请求的并发竞争和接受候选选择绑定请求
的并发竞争。附录B.6有这个的详细解释。因此，处理带有这个属性的请求要依赖竞争的胜利者。

端点给媒体流的每个组件生成候选对通过：
+ 设置远程候选为请求的组件默认目的地(例如,RTP中m行和c行的内容，RTCP中a=rtcp属性内容)
+ 设置本地候选为请求中的a=remote-candidates属性的同一组件的传输地址
然后端点检查这些候选对是否出现在有限列表中。如果不存在，则这个检查失去竞争力，这样候选对成为“丢失对”。

端点在检查列表中找出所有的远程候选等丢失对远程候选的候选对:
+ 如果这些候选对没有In-Progress的，并且至少有一个是Failed, 则最有可能是网络失败，比如网络断开或是严重的丢包。端点应该生成一个认为remote-candidates属性没有
存在的响应，然后重启这个媒体流的ICE。
+ 如果至少有一个对是In-Progress，端点应该等待这些检查完成，当完成后，重新按照本节的流程的执行知道没有丢失对。

当没有丢失对时，端点就可以生成响应了。响应必须把默认目的地设置为请求的remote-candidate属性值(每一个值现在应该是有效列表中一个对的本地候选)。响应必须包含为
每个在请求remote-candidates属性的候选的一个属性。

# 9.2.3 简化实现流程
如果接收到的请求含有媒体流的remote-candidates属性，端点应该给该媒体流的每个组件构造一个候选对,方法是:
+ 设置远程候选为请求的组件默认目的地(例如,RTP中m行和c行的内容，RTCP中a=rtcp属性内容)
+ 设置本地候选为请求中的a=remote-candidates属性的同一组件的传输地址
这些候选对会更替到媒体流的有效列表中。这个媒体流的ICE处理状态设置为Completed。

进一步讨论，如果端点认为自己是控制端点，但是对端请求中却包含了remote-candidate属性，则双方都认为自己控制端点。但是通过请求/响应的交换信号协议会解决这个冲突，
总有一个端点会竞争胜利，它的请求会被对方接收而对方还没发出请求。胜利者会保持控制端点，失败的端点（响应方）会转变为被控制角色。因此，控制端点本来将要发送一个更新
请求，参照小节8.2.2，已经不再需要了。

除了可能的角色改变，有效列表改变和状态改变，响应的生成和小节9.1.3中描述的是一致的。

# 9.3 更新检查列表和有效列表
# 9.3.1 所有实现的流程
# 9.3.1.1 ICE重启
在重启之前，端点必须记住每个媒体流下的每个组件的有效列表最高优先级的那个提名候选对，成为被以前选中候选对。端点会继续在这个对上发送媒体数据，小节11.1。
当这些通讯目的地记录后，端点会清空有效和检查列表，开始重新计算列表内容和状态，小节5.7.

# 9.3.1.2 新增媒体流
如果请求/响应中包含了一个新的媒体流，端点必须为这个媒体流创建一个新的检查列表（和一个空的新的有效列表），小节5.7。

# 9.3.1.3 移除媒体流
如果请求/响应中要移除一个媒体流，或是响应拒绝了请求中的媒体流，端点必须把媒体流的有效列表清空。必须终止这个流上在进行的STUN事务。端点必须移除媒体流的检查列表
并放弃所有等待中的一般检查。

# 9.3.1.4 已存在的媒体流的继续处理
有效列表不会在请求/响应的消息交换中被影响，除非是ICE重启。
如果端点的媒体流是在Running状态，检查列表则会更新(在Completed状态就不会)。端点重新计算检查列表参照小节5.7.如果重新计算出的候选对也在以前的检查列表里，
并且状态是Waiting/In-Progress/Succeeded/Failed, 那这个状态也会保持。否则，则设置为Frozen。

如果没有检查列表是活动的(即列表中的候选对都是Frozen),完全实现端点设置第一个媒体流的检查列表的第一个候选对为Waiting,同时设置这个检查列表中其他和这个对同一
组件ID且基础设定相同的候选对也是Wating.

接下来端点开始对每个检查列表的最高优先对进行出来。如果这个对已经是Succeeded状态，并且组件ID是1，则这个检查列表其他所有Frozen对中具有相同基础设定并且组件ID
不是1的候选对状态设置为Waiting.如果对于媒体流上的每个组件都有了Succeeded状态的候选对，端点则把其他媒体流上第一个组件的所有Frozen但是具有相同基础设定的候选
对设置为Waiting.

# 9.3.2 简化实现流程
如果ICE重启一个媒体流，端点必须给媒体流新建一个有效列表。端点必须记住每个媒体流下的每个组件的有效列表最高优先级的那个提名候选对，成为被以前选中候选对。端点会继续
在这个对上发送媒体数据,小节11.1.所有媒体流的ICE处理状态必须变更为Running, 而ICE的总体处理状态也设置为Running。























 





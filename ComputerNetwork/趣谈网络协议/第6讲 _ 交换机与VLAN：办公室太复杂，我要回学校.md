# 第6讲 \| 交换机与VLAN：办公室太复杂，我要回学校



<audio><source src="https://static001.geekbang.org/resource/audio/23/0c/2312a98fcc46cea1d9f5e3f3ed16df0c.mp3" type="audio/mpeg"></audio>

上一次，我们在宿舍里组建了一个本地的局域网LAN，可以愉快地玩游戏了。这是一个非常简单的场景，因为只有一台交换机，电脑数目很少。今天，让我们切换到一个稍微复杂一点的场景，办公室。

## 拓扑结构是怎么形成的？

我们常见到的办公室大多是一排排的桌子，每个桌子都有网口，一排十几个座位就有十几个网口，一个楼层就会有几十个甚至上百个网口。如果算上所有楼层，这个场景自然比你宿舍里的复杂多了。具体哪里复杂呢？我来给你具体讲解。

首先，这个时候，一个交换机肯定不够用，需要多台交换机，交换机之间连接起来，就形成一个稍微复杂的**拓扑结构**。

我们先来看**两台交换机**的情形。两台交换机连接着三个局域网，每个局域网上都有多台机器。如果机器1只知道机器4的IP地址，当它想要访问机器4，把包发出去的时候，它必须要知道机器4的MAC地址。

![](<https://static001.geekbang.org/resource/image/08/29/0867321c36cc52bd3dd4d7622583fa29.jpg?wh=2866*2176>)

于是机器1发起广播，机器2收到这个广播，但是这不是找它的，所以没它什么事。交换机A一开始是不知道任何拓扑信息的，在它收到这个广播后，采取的策略是，除了广播包来的方向外，它还要转发给其他所有的网口。于是机器3也收到广播信息了，但是这和它也没什么关系。

当然，交换机B也是能够收到广播信息的，但是这时候它也是不知道任何拓扑信息的，因而也是进行广播的策略，将包转发到局域网三。这个时候，机器4和机器5都收到了广播信息。机器4主动响应说，这是找我的，这是我的MAC地址。于是一个ARP请求就成功完成了。

<!-- [[[read_end]]] -->

在上面的过程中，交换机A和交换机B都是能够学习到这样的信息：机器1是在左边这个网口的。当了解到这些拓扑信息之后，情况就好转起来。当机器2要访问机器1的时候，机器2并不知道机器1的MAC地址，所以机器2会发起一个ARP请求。这个广播消息会到达机器1，也同时会到达交换机A。这个时候交换机A已经知道机器1是不可能在右边的网口的，所以这个广播信息就不会广播到局域网二和局域网三。

当机器3要访问机器1的时候，也需要发起一个广播的ARP请求。这个时候交换机A和交换机B都能够收到这个广播请求。交换机A当然知道主机A是在左边这个网口的，所以会把广播消息转发到局域网一。同时，交换机B收到这个广播消息之后，由于它知道机器1是不在右边这个网口的，所以不会将消息广播到局域网三。

## 如何解决常见的环路问题？

这样看起来，两台交换机工作得非常好。随着办公室越来越大，交换机数目肯定越来越多。当整个拓扑结构复杂了，这么多网线，绕过来绕过去，不可避免地会出现一些意料不到的情况。其中常见的问题就是**环路问题**。

例如这个图，当两个交换机将两个局域网同时连接起来的时候。你可能会觉得，这样反而有了高可用性。但是却不幸地出现了环路。出现了环路会有什么结果呢？

![](<https://static001.geekbang.org/resource/image/1f/ea/1f909508a8253d4842ffe962883421ea.jpg?wh=3001*1591>)

我们来想象一下机器1访问机器2的过程。一开始，机器1并不知道机器2的MAC地址，所以它需要发起一个ARP的广播。广播到达机器2，机器2会把MAC地址返回来，看起来没有这两个交换机什么事情。

但是问题来了，这两个交换机还是都能够收到广播包的。交换机A一开始是不知道机器2在哪个局域网的，所以它会把广播消息放到局域网二，在局域网二广播的时候，交换机B右边这个网口也是能够收到广播消息的。交换机B会将这个广播信息发送到局域网一。局域网一的这个广播消息，又会到达交换机A左边的这个接口。交换机A这个时候还是不知道机器2在哪个局域网，于是将广播包又转发到局域网二。左转左转左转，好像是个圈哦。

可能有人会说，当两台交换机都能够逐渐学习到拓扑结构之后，是不是就可以了？

别想了，压根儿学不会的。机器1的广播包到达交换机A和交换机B的时候，本来两个交换机都学会了机器1是在局域网一的，但是当交换机A将包广播到局域网二之后，交换机B右边的网口收到了来自交换机A的广播包。根据学习机制，这彻底损坏了交换机B的三观，刚才机器1还在左边的网口呢，怎么又出现在右边的网口呢？哦，那肯定是机器1换位置了，于是就误会了，交换机B就学会了，机器1是从右边这个网口来的，把刚才学习的那一条清理掉。同理，交换机A右边的网口，也能收到交换机B转发过来的广播包，同样也误会了，于是也学会了，机器1从右边的网口来，不是从左边的网口来。

然而当广播包从左边的局域网一广播的时候，两个交换机再次刷新三观，原来机器1是在左边的，过一会儿，又发现不对，是在右边的，过一会，又发现不对，是在左边的。

这还是一个包转来转去，每台机器都会发广播包，交换机转发也会复制广播包，当广播包越来越多的时候，按照上一节讲过一个共享道路的算法，也就是路会越来越堵，最后谁也别想走。所以，必须有一个方法解决环路的问题，怎么破除环路呢？

## STP协议中那些难以理解的概念

在数据结构中，有一个方法叫做**最小生成树**。有环的我们常称为**图**。将图中的环破了，就生成了**树**。在计算机网络中，生成树的算法叫作**STP**，全称**Spanning Tree Protocol**。

STP协议比较复杂，一开始很难看懂，但是其实这是一场血雨腥风的武林比武或者华山论剑，最终决出五岳盟主的方式。

![](<https://static001.geekbang.org/resource/image/47/23/47baa69073b38357e0ae3f88ff74dd23.jpg?wh=3623*2579>)

在STP协议里面有很多概念，译名就非常拗口，但是我一作比喻，你很容易就明白了。

- **Root Bridge**，也就是**根交换机**。这个比较容易理解，可以比喻为“掌门”交换机，是某棵树的老大，是掌门，最大的大哥。

- **Designated Bridges**，有的翻译为**指定交换机**。这个比较难理解，可以想像成一个“小弟”，对于树来说，就是一棵树的树枝。所谓“指定”的意思是，我拜谁做大哥，其他交换机通过这个交换机到达根交换机，也就相当于拜他做了大哥。这里注意是树枝，不是叶子，因为叶子往往是主机。

- **Bridge Protocol Data Units （BPDU）** ，**网桥协议数据单元**。可以比喻为“相互比较实力”的协议。行走江湖，比的就是武功，拼的就是实力。当两个交换机碰见的时候，也就是相连的时候，就需要互相比一比内力了。BPDU只有掌门能发，已经隶属于某个掌门的交换机只能传达掌门的指示。

- **Priority Vector**，**优先级向量**。可以比喻为实力 （值越小越牛）。实力是啥？就是一组ID数目，[Root Bridge ID, Root Path Cost, Bridge ID, and Port ID]。为什么这样设计呢？这是因为要看怎么来比实力。先看Root Bridge ID。拿出老大的ID看看，发现掌门一样，那就是师兄弟；再比Root Path Cost，也即我距离我的老大的距离，也就是拿和掌门关系比，看同一个门派内谁和老大关系铁；最后比Bridge ID，比我自己的ID，拿自己的本事比。


<!-- -->

## STP的工作过程是怎样的？

接下来，我们来看STP的工作过程。

一开始，江湖纷争，异常混乱。大家都觉得自己是掌门，谁也不服谁。于是，所有的交换机都认为自己是掌门，每个网桥都被分配了一个ID。这个ID里有管理员分配的优先级，当然网络管理员知道哪些交换机贵，哪些交换机好，就会给它们分配高的优先级。这种交换机生下来武功就很高，起步就是乔峰。

![](<https://static001.geekbang.org/resource/image/66/2b/66237be156bea81a801dca8d507c1e2b.jpg?wh=1655*1376>)

既然都是掌门，互相都连着网线，就互相发送BPDU来比功夫呗。这一比就发现，有人是岳不群，有人是封不平，赢的接着当掌门，输的就只好做小弟了。当掌门的还会继续发BPDU，而输的人就没有机会了。它们只有在收到掌门发的BPDU的时候，转发一下，表示服从命令。

![](<https://static001.geekbang.org/resource/image/5d/47/5da50b7e328ea3cf8f90430f1deb3f47.jpg?wh=1655*1379>)

数字表示优先级。就像这个图，5和6碰见了，6的优先级低，所以乖乖做小弟。于是一个小门派形成，5是掌门，6是小弟。其他诸如1-7、2-8、3-4这样的小门派，也诞生了。于是江湖出现了很多小的门派，小的门派，接着合并。

合并的过程会出现以下四种情形，我分别来介绍。

### 情形一：掌门遇到掌门

当5碰到了1，掌门碰见掌门，1觉得自己是掌门，5也刚刚跟别人PK完成为掌门。这俩掌门比较功夫，最终1胜出。于是输掉的掌门5就会率领所有的小弟归顺。结果就是1成为大掌门。

![](<https://static001.geekbang.org/resource/image/fb/56/fb0e19a14e00b5825dac11d359ffe056.jpg?wh=1655*1379>)

### 情形二：同门相遇

同门相遇可以是掌门与自己的小弟相遇，这说明存在“环”了。这个小弟已经通过其他门路拜在你门下，结果你还不认识，就PK了一把。结果掌门发现这个小弟功夫不错，不应该级别这么低，就把它招到门下亲自带，那这个小弟就相当于升职了。

我们再来看，假如1和6相遇。6原来就拜在1的门下，只不过6的上司是5，5的上司是1。1发现，6距离我才只有2，比从5这里过来的5（=4+1）近多了，那6就直接汇报给我吧。于是，5和6分别汇报给1。

![](<https://static001.geekbang.org/resource/image/1e/d8/1ef3c9fb5b7d386c519402202233a8d8.jpg?wh=1655*1379>)

同门相遇还可以是小弟相遇。这个时候就要比较谁和掌门的关系近，当然近的当大哥。刚才5和6同时汇报给1了，后来5和6在比较功夫的时候发现，5你直接汇报给1距离是4，如果5汇报给6再汇报给1，距离只有2+1=3，所以5干脆拜6为上司。

### 情形三：掌门与其他帮派小弟相遇

小弟拿本帮掌门和这个掌门比较，赢了，这个掌门拜入门来。输了，会拜入新掌门，并且逐渐拉拢和自己连接的兄弟，一起弃暗投明。

![](<https://static001.geekbang.org/resource/image/8e/da/8e852604ac81ab453115470edb9e70da.jpg?wh=1655*1379>)

例如，2和7相遇，虽然7是小弟，2是掌门。就个人武功而言，2比7强，但是7的掌门是1，比2牛，所以没办法，2要拜入7的门派，并且连同自己的小弟都一起拜入。

### 情形四：不同门小弟相遇

各自拿掌门比较，输了的拜入赢的门派，并且逐渐将与自己连接的兄弟弃暗投明。<br>

![](<https://static001.geekbang.org/resource/image/fd/bf/fdab777fb2f69666e1fd5d838278b1bf.jpg?wh=1655*1379>)

例如，5和4相遇。虽然4的武功好于5，但是5的掌门是1，比4牛，于是4拜入5的门派。后来当3和4相遇的时候，3发现4已经叛变了，4说我现在老大是1，比你牛，要不你也来吧，于是3也拜入1。

最终，生成一棵树，武林一统，天下太平。但是天下大势，分久必合，合久必分，天下统一久了，也会有相应的问题。

### 如何解决广播问题和安全问题？

毕竟机器多了，交换机也多了，就算交换机比Hub智能一些，但是还是难免有广播的问题，一大波机器，相关的部门、不相关的部门，广播一大堆，性能就下来了。就像一家公司，创业的时候，一二十个人，坐在一个会议室，有事情大家讨论一下，非常方便。但是如果变成了50个人，全在一个会议室里面吵吵，就会乱得不得了。

你们公司有不同的部门，有的部门需要保密的，比如人事部门，肯定要讨论升职加薪的事儿。由于在同一个广播域里面，很多包都会在一个局域网里面飘啊飘，碰到了一个会抓包的程序员，就能抓到这些包，如果没有加密，就能看到这些敏感信息了。还是上面的例子，50个人在一个会议室里面七嘴八舌地讨论，其中有两个HR，那他们讨论的问题，肯定被其他人偷偷听走了。

那咋办，分部门，分会议室呗。那我们就来看看怎么分。

有两种分的方法，一个是**物理隔离**。每个部门设一个单独的会议室，对应到网络方面，就是每个部门有单独的交换机，配置单独的子网，这样部门之间的沟通就需要路由器了。路由器咱们还没讲到，以后再说。这样的问题在于，有的部门人多，有的部门人少。人少的部门慢慢人会变多，人多的部门也可能人越变越少。如果每个部门有单独的交换机，口多了浪费，少了又不够用。

另外一种方式是**虚拟隔离**，就是用我们常说的**VLAN**，或者叫**虚拟局域网**。使用VLAN，一个交换机上会连属于多个局域网的机器，那交换机怎么区分哪个机器属于哪个局域网呢？<br>

![](<https://static001.geekbang.org/resource/image/ba/60/ba720f6988558f95c381f4deaab11660.jpg?wh=2066*1583>)

我们只需要在原来的二层的头上加一个TAG，里面有一个VLAN ID，一共12位。为什么是12位呢？因为12位可以划分4096个VLAN。这样是不是还不够啊。现在的情况证明，目前云计算厂商里面绝对不止4096个用户。当然每个用户需要一个VLAN了啊，怎么办呢，这个我们在后面的章节再说。

如果我们买的交换机是支持VLAN的，当这个交换机把二层的头取下来的时候，就能够识别这个VLAN ID。这样只有相同VLAN的包，才会互相转发，不同VLAN的包，是看不到的。这样广播问题和安全问题就都能够解决了。<br>

![](<https://static001.geekbang.org/resource/image/5c/4a/5c207a6e2c1c9881823b04e648f4ba4a.jpg?wh=2593*1873>)

我们可以设置交换机每个口所属的VLAN。如果某个口坐的是程序员，他们属于VLAN 10；如果某个口坐的是人事，他们属于VLAN 20；如果某个口坐的是财务，他们属于VLAN 30。这样，财务发的包，交换机只会转发到VLAN 30的口上。程序员啊，你就监听VLAN 10吧，里面除了代码，啥都没有。

而且对于交换机来讲，每个VLAN的口都是可以重新设置的。一个财务走了，把他所在座位的口从VLAN 30移除掉，来了一个程序员，坐在财务的位置上，就把这个口设置为VLAN 10，十分灵活。

有人会问交换机之间怎么连接呢？将两个交换机连接起来的口应该设置成什么VLAN呢？对于支持VLAN的交换机，有一种口叫作**Trunk口**。它可以转发属于任何VLAN的口。交换机之间可以通过这种口相互连接。

好了，解决这么多交换机连接在一起的问题，办公室的问题似乎搞定了。然而这只是一般复杂的场景，因为你能接触到的网络，到目前为止，不管是你的台式机，还是笔记本所连接的网络，对于带宽、高可用等都要求不高。就算出了问题，一会儿上不了网，也不会有什么大事。

我们在宿舍、学校或者办公室，经常会访问一些网站，这些网站似乎永远不会“挂掉”。那是因为这些网站都生活在一个叫做数据中心的地方，那里的网络世界更加复杂。在后面的章节，我会为你详细讲解。

## 小结

好了，这节就到这里，我们这里来总结一下：

- 当交换机的数目越来越多的时候，会遭遇环路问题，让网络包迷路，这就需要使用STP协议，通过华山论剑比武的方式，将有环路的图变成没有环路的树，从而解决环路问题。
- 交换机数目多会面临隔离问题，可以通过VLAN形成虚拟局域网，从而解决广播问题和安全问题。

<!-- -->

最后，给你留两个思考题。

1. STP协议能够很好地解决环路问题，但是也有它的缺点，你能举几个例子吗？
2. 在一个比较大的网络中，如果两台机器不通，你知道应该用什么方式调试吗？

<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(195)

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  thomas

  置顶

  第一张图中，机器三是如何同时链接两台交换机？

  作者回复: 赞，我以为不会有人问这个问题的，哈哈，老的局域网都是连到线上的，所以延续了这个图，为了准确，这里面中间的局域网可以认为是一个非直连的，例如中间隐藏了交换机等细节的，为了说明这个理论而简化

  2018-05-30

  **15

  **236

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093706-3992.jpeg)

  narry

  stp中如果有掌门死掉了，又得全部重选一次，用的时间比较长，期间网络就会中断的

  2018-05-30

  **2

  **192

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093706-3993.jpeg)

  杨武刚@纷享销客

  老师的这个比喻让我这个门外汉听得很爽，化繁为简，赞一个，希望老师以后多用比喻

  作者回复: 谢谢

  2018-05-30

  **2

  **101

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093706-3994.jpeg)

  硅谷居士

  \1. STP 对于跨地域甚至跨国组织的网络支持，就很难做了，计算量摆着呢。 2. ping 加抓包工具，如 wireshark

  作者回复: 赞

  2018-05-31

  **

  **84

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3995.jpeg)

  iLeGeND

  第一:怎么感觉像培训网管呢 第二:有些东西 不适合做比喻 掌门那块不是到在讲什么 太乱了

  作者回复: 如果是开发，一般可能接触到的是传输层，但是一旦往底层学，这些知识是必须的。对于比喻的事情呢？当前学一门科学，最本质的是去看最原始的文档，表达严谨，论文一样，但是上来门槛比较高，所以做个比喻让人容易理解，建议对着图看一下，这个比喻我其实想了好久，内部培训同事的时候是讲的明白的

  2018-05-30

  **12

  **76

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3996.jpeg)

  李晓东

  请教个问题，讲STP的时候，两个交换机之间连线的数字代表什么？怎么得来的？

  2018-06-13

  **8

  **63

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3997.jpeg)

  颇忒妥

  图一和图二有点看不懂，图里的交换机和PC 是物理设备，这个LAN 是什么？不是应该交换机和PC 直接用一根线相连么？

  作者回复: 这是个虚指的局域网，不一定直连，里面可以隐藏一些设备，例如hub，交换机

  2018-05-30

  **4

  **57

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3998.jpeg)

  magict4

  老师你好，我跟zixuan@有着相同的疑问。 文中提到： 当机器 2 要访问机器 1 的时候，机器 2 并不知道机器 1 的 MAC 地址，所以机器 2 会发起一个 ARP 请求。这个广播消息会到达机器 1，也同时会到达交换机 A。这个时候交换机 A 已经知道机器 1 是不可能在右边的网口的，所以这个广播信息就不会广播到局域网二和局域网三。 根据前一小节的内容，我有以下理解： 1. 交换机是二层设备，不会读取 IP 层的内容。 2. 交换机会缓存 MAC 地址跟转发端口的关系。 3. ARP 协议是广播的，目的地 MAC 地址是广播地址。 如果我的理解是正确的，那机器 2 发起的 ARP 请求中，是不含机器 1 的 MAC 地址的，只有广播地址。交换机 A 中缓存的信息是没法被利用起来的。那么交换机 A 是如何知道不需要把请求转发到其它局域网的呢？

  作者回复: 好像这个说法的确有问题，不是arp过程的，是发包过程的，由全部转发变成有脑子的

  2018-06-19

  **10

  **55

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3999.jpeg)

  A7

  感觉spanning tree和vlan会死一堆人…… stp的缺点就是，当掌门死了或者有新人进入江湖，江湖上就又要经历一场血雨腥风，如果江湖很大的话，就会血雨腥风很久……

  2018-06-04

  **4

  **43

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4000.jpeg)

  戴劼 DAI JIE🤪

  有一次办公室断网，排查时候发现路由器某一个部门的端口的灯在狂闪，拔掉后恢复正常。然后去那个部门排查才发现他们插错了口，形成了环路导致广播风暴。

  作者回复: 是的，赞

  2018-06-07

  **4

  **41

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4001.jpeg)

  奔跑的蜗牛

  从公众号追过来的，头一次听到这么好听的STP，终于明白原理了，再看STP就不那么头大了

  作者回复: 谢谢

  2018-05-31

  **2

  **34

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4002.jpeg)

  埃罗芒阿老师

  1.stp缺点的话，一个是某个交换机状态发生变化的时候，整个树需要重新构建，另一个是被破开的环的链路被浪费了 2.先ping，不通的话traceroute，参数逐渐加一

  2018-05-30

  **1

  **27

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4003.jpeg)

  化雨

  文中的拓扑图确实令我疑惑，好在thomas已经帮我发问了哈哈。能否考虑调整下拓扑图的画法：线条真实反应各个节点(主机，交换机等)的物理连接，同一个局域网的节点用虚线框出来

  作者回复: 看来这个图应该重新画了，谢谢

  2018-06-02

  **

  **20

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4004.jpeg)

  渔夫

  所有交换机都支持STP协议吗？除了STP还有别的什么机制能防止或预防网络环路风暴？谢谢

  作者回复: 有的，现在很少用stp了，后面讲数据中心的时候会提到

  2018-06-04

  **

  **17

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4005.jpeg)

  一叶孤航

  没看懂图一，机器和交换机不是直接网线连接的么？那么LAN1,LAN2是什么?LAN2又是怎么同时连接两台路由器的?求解惑

  2018-05-30

  **

  **16

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-4006.jpeg)

  天空白云

  第二个问题？ping ，traceroute？

  作者回复: 是哒

  2018-05-30

  **

  **12

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4007.jpeg)

  张玮(大圣)

  无网络基础，看起来有点费力啊

  2018-07-08

  **

  **10

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4008.jpeg)

  灰飞灰猪不会灰飞.烟灭

  老师，那假如既要保证vlan之间通讯，又要和其它部门通讯怎么呢？通过设置trunk吗？

  作者回复: 应该用路由器

  2018-05-30

  **2

  **9

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4009.jpeg)

  李博越

  看文章里最后一张图，两个不同的交换机，肯定是2个不同的网段，又为何能使用同一个vlanId进行网络划分呢？一个vlanId只能绑定一个网段才对吧？甚是不解

  作者回复: 交换机可以识别vlan的，交换机本身没有所谓的网段，交换机的口上不配置ip.一个vlan一个网段是对的，在同一个vlan的机器配置相同的网段就可以了，和交换机关系不大

  2018-09-20

  **

  **7

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4010.jpeg)

  Regular

  第一张图中，机器3、交换机A、交换机B三者是怎么连接的？

  2018-05-30

  **

  **7

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4011.jpeg)

  Lsoul

  请问，图一机器三如果是双网卡又是如何通信的呢

  作者回复: 这个比较复杂，看这两个网卡在机器里面是怎么配置的了

  2018-06-07

  **

  **6

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4012.jpeg)

  程序员人生

  原来交换机是机器学习的鼻祖

  作者回复: 那不是，他是基于规则的

  2019-03-08

  **3

  **5

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4013.jpeg)

  Zeal

  比喻很生动，其实就是并查集。

  2020-03-23

  **2

  **4

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4014.jpeg)

  蒋旺Foo

  在讲解STP工作过程的时候，交换机如果能使用字母来命名那应该会更加清晰。因为交换机之间用数字表示距离，交换机命令也用数字，难免使人混淆。谢谢老师。

  作者回复: 好像是

  2019-04-03

  **2

  **4

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4015.jpeg)

  王先统

  无论是hub还是交换机都是为了解决局域网内部机器的互通问题，到现在为止我们还没讲到广域网的访问，是吧？

  作者回复: 是的

  2018-11-28

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  你好，环路那个图，lan1的机器1和机器2怎么同时连两个交换机的，中间是有个集线器吗？

  作者回复: 对的，后面答疑环节纠正了这个问题

  2018-09-11

  **

  **4

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4016.jpeg)

  姬野用菜刀

  刘老师，stp图中交换机之间连线的数字是什么哈？另外为了解决成环问题而增加这个机制，会对性能有多少影响呢

  2018-06-05

  **1

  **4

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4017.jpeg)

  Geek_8a2f3f

  老师有个问题不明白。第二个图中说交换机A和B组成环路，且交换机都学会了，机器1在局域网一中。当机器1的包到达交换机A时，既然交换机A已经知道机器1在左边网络，为什么还会将包广播到局域网二中？

  2018-05-30

  **1

  **4

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4018.jpeg)

  宋桓公

  评论也是个宝库呀，这个栏目的听众质量高啊

  2019-11-07

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_94e115

  什么是广播风暴  === 1. 首先要熟悉 两台主机跨两个交换机 arp请求过程   VM1 | SW1 - SW2 | VM3  	- vm1 arping -b vm3    - sw1 学习到 vm1 是在自己左边的  #关注这个  - sw2 学习到 vm1 是在自己左边的  - vm3 回应 arp 单播  - sw2 根据转发表 给 SW1  - sw1 根据转发表 给 vm1 	 2. 广播风暴   #这个问题不用过分关注，vm1 vm2 所在的网段  VM1 | SW1 | VM3 - sw0   sw3 -  VM2	| SW2 |         - VM1 arping -b vm2  - sw1 学习到vm1是在自己左边的  - sw2 经过 sw3 转发 认为 vm1 在自己的右边  - sw2 通过 sw0 转发 认为 vm1 在自己的左边  - #sw2中对于 vm1 对应哪个网口的转发条目不停的变化这样就会导致 sw2 不能正常工作 - #这个广播报文会不停的转圈，占用带宽  3. 解决方案 stp #面对物理环路，通过软件逻辑破环   vxlan === 1.   VID 2^12 可表示 4096 个 vlan      #后续出现vxlan 解决了该瓶颈，同样vxlan还有其他的优势 2.   #广播域：指 广播报文（eth frame）可传播的范围                                    #是数据链路层的观念 #通过交换机来拓展  # 通过vlan来隔离 #广播  #面向对象 ethf  #冲撞域: 指 原始点对多链路，总线型拓扑，共享链路，发包会有冲撞  后续出现 CSMA/CD #是物理层的概念     #通过hub来拓展     # ？                    #面向对象 数据包  				  3.显然 vlan 是面对 ethf 的协议，实现广播域的隔离  	

  2019-04-29

  **1

  **3

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132.jpeg)

  ferry

  又查找了很多资料，明确了转发表和vlan的区别，转发表作用时是用于客户端之间通信过程中，避免给无关客户端广播，而vlan则是划分广播域，使消息在广播时只在一个小范围内传播，侧重于安全性。

  2019-02-07

  **

  **3

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132.jpeg)

  ferry

  老师，您前面讲到交换机上的转发表是用于找到要接受的MAC地址，避免广播的，vlan也是用于避免全部广播的，那么他们之间的区别是否在于，转发表适用于只发送信息给一个机器，而vlan适合发送信息到一部分（大于等于1个）的机器呢？

  作者回复: 是的，一个是学会了，不广播，一个是不在同一个vlan，不广播

  2019-02-07

  **

  **3

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4019.jpeg)

  zhushenghang

  图一画的不准确。后面的说法也有问题。既然机器1的广播机器4和机器3能收到，那么他们肯定是处于一个广播域内，通俗的讲就是在一个VLAN内，那么就是纯二层的东西。 根据二层转发流程： 提取数据报的源MAC地址，查询MAC转发表（也就是L2FDB），如果找到就直接发送到对应端口。 对于表中不包含的地址，通过广播发送，也就是发送到所有端口。 使用地址自动学习（根据源MAC地址学习）和老化机制（定时机制）来维护MAC转发表的信息，二层转发一般不会更改数据包内容。 在上面的过程中，交换机 A 和交换机 B 都是能够学习到这样的信息： 机器 1 是在左边这个网口的。当了解到这些拓扑信息之后，情况就好转起来。 当机器 2 要访问机器 1 的时候，机器 2 并不知道机器 1 的 MAC 地址，所以机器 2 会发起一个 ARP 请求。 这个广播消息会到达机器 1，也同时会到达交换机 A。这个时候交换机 A 已经知道机器 1 是不可能在右边的网口的，所以这个广播信息就不会广播到局域网二和局域网三。 ->这个广播照样会到达局域网二和局域网三，因为处于同一个广播域，目的MAC地址是全F的。 ->二层交换机不过是提取数据报的源MAC地址，记录到二层的MAC表里方便下次收到的数据报的目的MAC是这个的时候直接转发。 ->二层交换机发现数据报的目的MAC是广播地址，就会在VLAN里进行广播，所有同VLAN的服务器都能收到。 当机器 3 要访问机器 1 的时候，也需要发起一个广播的 ARP 请求。这个时候交换机 A 和交换机 B 都能够收到这个广播请求。交换机 A 当然知道主机 A 是在左边这个网口的，所以会把广播消息转发到局域网一。 同时，交换机 B 收到这个广播消息之后，由于它知道机器 1 是不在右边这个网口的，所以不会将消息广播到局域网三。 ->因此这个说法也有问题，如果有多台交换机A、B上行到核心且处于同一个VLAN，同时接入交换机的集群为K8S集群， ->那么如果其中有容器从A交换机漂移到了B交换机且仅MAC发生了改变IP地址不变，那即使A交换机下的容器再广播也找不到这个容器了？ ->按前面的说法聪明的交换机A并不会把广播包发往交换机B。那岂不是不通了？得等交换机mac表老化了才能通了？ ->很明显交换机不会这么设计，其实上面这个只要A交换机下的容器广播了那么所有处于广播域的服务器都能收到。

  作者回复: 二层交换机是没有ARP表的，而三层交换机是有ARP表的。 二层交换机在转发报文时，使用的是MAC表。根据MAC地址确定目的主机所在端口。 三层交换机在转发报文时，使用ARP表。根据IP地址确定目的MAC和所在端口。 虽然现在三层交换机是主流，但是这里是理论讲解，还在讨论二层交换机的情况。

  2020-05-27

  **2

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4020.jpeg)

  Phoenix

  作为非科班出身的程序员，我居然看懂了，可见老师的功力深厚，感谢老师

  2019-11-26

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4021.jpeg)

  随风

  LAN1/LAN2是什么东西？机器不都是直接连接交换机的吗？

  作者回复: 是的。也可以连hub

  2019-04-15

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4022.jpeg)

  Vicky🐣🐣🐣

  老师好！ 1. STP协议，prioprity vector 中的 PORT ID 指的是什么呢？ 2. 包的 VLAN TAG中的 优先级指的是什么？是属于这个VALN的包的转发优先级吗？

  2018-12-27

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4023.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  HelloBug

  老师，你好~只用交换机连接起来的网络，都是在一个局域网里是吧？

  作者回复: 是的

  2018-12-03

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4024.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  叮咚

  既然stp能解决环路，为什么实际场景中还是有环路发生？

  作者回复: 不一定开启stp

  2018-12-01

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4025.jpeg)

  李金洋

  是说把交换机变成单向广播吗？比如ABC三台交换机，两两互联，最后通过STP，变成A广播只发给B，B广播只发给C，然后反过来c只发给B，b只发给A(不同方向)

  作者回复: 不能单向广播

  2018-09-11

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4026.jpeg)

  Jin

  最前点赞两个问题我也想问的

  2018-05-30

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4027.jpeg)

  第五季

  问一个问题，我们应用服务器读redis缓存每分钟都有一个超时。并发是每分钟四万。三台虚拟redis。看应用服务器和redisc都无压力，red is也无队列阻塞。命令也是get，都是小可以。red is是集群模式，使用cachecloud搜狐产品监控。运维说网络也无问题。请问这种情况我怎么从Tcp协议层监控，怎么样找问题呢 2018-05-29

  2018-05-30

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4028.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  顾骨

  为什么网页没法留言。。。 想请教一下，第一张图能详细一点吗，有点没看懂，一个交换机怎么接2个局域网呢，机器3怎么会被两个交换机接入

  作者回复: 为了描述意思简化了，好多人问这个问题，看来有时候不能简化，但是不简化容易让读者从当时应该关注的点精力分散，比较难权衡

  2018-05-30

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4029.jpeg)

  肖一林

  要是有交换机挂了，特别是掌门挂了，这个江湖会不会大乱

  作者回复: 会的，所以要堆叠

  2018-05-30

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  博白

  可以推荐一下这方面的书籍吗

  2018-05-30

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4030.jpeg)

  sunlight001

  有个疑问，环路问题是交换机解决的还是人工解决的，stp在比较的时候一开始都是形成两两的门派吗，他们怎么比较成为老大的这个没看明白，请老师给解释下

  作者回复: 交换机解决的，通过协议。也不是，为了穷举所有的场景，故意构建的这个图和这种合并顺序

  2018-05-30

  **

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093710-4031.png)

  shihao121

  图中两个交换机相连接的数字1、2、4是不是port cost。根据port cost来选择两个网段内的Designated bridge。

  2020-09-14

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093710-4031.png)

  shihao121

  文章中说生成树的算法叫做STP，但是STP是spanning-tree protocal，它只是个协议不是算法，他使用的算法是STA（spanning-tree algorithm）

  2020-09-14

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093710-4032.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  干饭团

  老师好 请问这么多交换机都要分配id优先级？这个优先级是怎么标记的，保存在什么地方，如果有两个1怎么办？

  作者回复: 不能重复的，是管理员自己配置的，那个时候交换机数目没这么多

  2020-05-29

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093710-4033.jpeg)

  雷刚

  STP 算法感觉和 Raft 协议很像啊。 1. Zab 协议也是先投自己一票（也相当于是先掌门），然后把消息广播出去，然后按照 [epoch, zxid, service_id] 三个维度进行 PK，直到选票半数以上则为老大。 2. STP 算法也是先投自己一票，然后也是把消息广播出去，然后按照 [Root Bridge ID, Root Path Cost, Bridge ID, and Port ID] 进行 PK。不过感觉还是要比 Raft 协议复杂一些。

  2020-04-07

  **1

  **2

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4034.jpeg)

  AMIR

  老师，设置了vlan ，那么程序员想和秘书交流感情的时候，结果在不同的vlan，这时候怎么处理

  作者回复: 就用到下一节的路由了，跨网段访问

  2020-01-10

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4035.jpeg)

  鸠摩·智![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  老师，有个问题请教！支持vlan的交换机之间是通过trunk口通信的，那一台交换机上trunk口有几个呢？如果只有一个，那就只能2台交换机连接在一起了，还是交换机上的所有网口都支持在普通和trunk之间随意切换？

  作者回复: 每个口都可以配置的，不是死的，是可以通过命令进行配置

  2020-01-09

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4036.jpeg)

  花脸猫

  第一张图，机器一和机器三应该不是同一网段的吧... 包怎么过去的

  作者回复: 同一网段的

  2019-08-05

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4037.jpeg)

  焦太郎

  为什么变成STP协议就解决了环路问题，本人愚笨，不明点就不懂

  作者回复: 没有变成STP协议，而是用上STP协议之后，所有的环都破了，变成了树。

  2019-06-14

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4038.jpeg)

  傲娇的小宝

  大网络不通就分段调，看到哪段不通的，不过在有环路的情况下，其实不会不同的，因为相互算备份了。记得大学学的时候除了根桥还有个副根桥，根桥挂掉是副的升级（比较久了可能会记错），问题应该是网络太大的时候路由变动了会比较麻烦吧。

  作者回复: 一旦变动，的确麻烦

  2019-05-13

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4021.jpeg)

  随风

  vlan id不同的机器是不是就没法通信了？

  作者回复: 三层通信

  2019-04-15

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4039.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  春和景明

  老师，是不是小弟不会给大哥转发数据包

  作者回复: 如果小弟认了新的大哥就会，不认就不会

  2019-04-07

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093711-4040.jpeg)

  啦啦啦

  这篇文章对老师讲的是一个补充，大家可以看看https://www.cnblogs.com/weiyikang/p/4999432.html

  2019-03-21

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4041.jpeg)

  zhj

  你好 麻烦问下，知道了对方的具体mac地址后，在同一局域网内发给具体明确的mac地址时，同一lan内的其他机器也能收到该包吗，也就是除了arp寻址阶段的明确广播外，之后寻址后的真正具体mac地址通讯也是会广播给其他lan内的机器吗(只有这样好像vlan才显得有意义，不然vlan只是个例单纯的arp广播好像意义不大)，还望解答

  作者回复: 不是广播，但是都能收到是物理层做的

  2019-02-28

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4042.jpeg)

  阿拉D

  不知道刘老师还能不能看到。我想问，支持VLAN的交换机，是不是交换机转发表也会多VLAN ID这个字段？

  作者回复: 是的

  2018-10-30

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  朝闻道

  数据中心都不用生成树吧，网易的数据中心有用生成树么？

  作者回复: 没有，后面讲数据中心的时候，会讲这个历程，但是不能忽略

  2018-09-16

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  第一个图物理的LAN123都能广播，划分VLan为什么就广播不了了呢？

  作者回复: lan123的说法有误导性，其实是物理网络123

  2018-09-14

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  交换机本身有IP吧？第一个图中，交换机是网关吗？

  作者回复: 交换机不是网关

  2018-09-14

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4043.jpeg)

  jacky

  老师你好，拓扑结构连线中的数字代表什么，还是仅仅是作为标记

  作者回复: cos

  2018-08-21

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4044.jpeg)

  lmtoo

  交换机是怎么计算距离的

  2018-06-25

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4045.jpeg)

  zixuan

  前面这段例子感觉不太恰当：“当机器 2 要访问机器 1 的时候，机器 2 并不知道机器 1 的 MAC 地址，所以机器 2 会发起一个 ARP 请求。这个广播消息会到达机器 1，也同时会到达交换机 A。这个时候交换机 A 已经知道机器 1 是不可能在右边的网口的”。  从前面几节的内容仅能推出交换机是根据记录Mac来做转发，而ARP里面显然没有机器1的Mac地址的，只有机器2的Mac和机器1的IP，难到交换机会去理解并取出二层包payload?

  作者回复: 不会读payload，是上一轮1广播的时候学习的

  2018-06-18

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4046.jpeg)

  zj坚果

  stp不足之处： 1.每次有改变就需要再华山论剑，所以拓扑收敛 2.不能提供负载均衡，浪费网络资源。 各位老师和读者大哥，是不是这样呀，还有其他不足吗？

  2018-06-06

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4047.jpeg)

  Adam

  stp部分有疑问 1、是当一台交换机接入网络时，这台交换机向周围直接相连的发起比武？找出离掌门最近的路。 2、当有一台交换机离开网络时，是不是与它直接相连的都要向周围重新发起比武？ 3、一开始每台交换机的武力值以及同周围连线的距离都要管理员手动指定吗？工作量是不是太大了？

  2018-06-04

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4048.jpeg)

  刘意

  交换机建立了mac地址表后不会转发已知mac地址的arp广播吗

  作者回复: 不在那个口就不转了

  2018-06-01

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4049.jpeg)

  darrylling

  Priority Vector，优先级向量就是一组 ID 数目Root Bridge ID, Root Path Cost, Bridge ID, and Port ID。先看 Root Bridge ID再比 Root Path Cost最后比 Bridge ID。为什么没说到PORTID?

  作者回复: 这个比较容易理解，就是口也有编号，编号小的优先级高。看来是不严谨了，我需要改进。

  2018-05-30

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4050.jpeg)

  文子

  对于二层交换机来说，对于其无法确定的端口arp请求采取是广播的方式，将包分发出去，那如果不同网段的两台主机a，b连接在同一个交换机上，那a发送给b的arp请求，b应该能接收到，且b能回复自己的ip地址，这两台主机是不是可以通信？有点不理解。

  作者回复: 不在同一网段，是不会发arp的

  2018-05-30

  **2

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4051.jpeg)

  CH.en

  老师，那个root  path  cost怎么来的呢？

  作者回复: 见ieee 802.1D

  2018-05-30

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4052.jpeg)

  。。。![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  什么时候使用交换机，什么时候使用路由器？这两个什么区别？

  2022-08-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4053.jpeg)

  戴宇

  回头来看这个spanning tree， 发现这个就是并查集

  2022-04-09

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek6900

  这个视频讲STP讲得不错 https://www.youtube.com/watch?v=Ilpmn-H8UgE

  2022-03-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4054.jpeg)

  Sean

  stp就是通过多个交换机复杂的链接选出最简短的到目标地址的的协议么

  2022-02-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4055.jpeg)

  itzzy

  stp那块有点像图里用迪杰特斯拉/弗洛伊德算法求最短路径

  2022-02-14

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4056.jpeg)

  刘玉琪

  有环路的第一张图，当机器2回复机器1发起的ARP请求的时候，两台交换机能学习到机器2的位置吗？

  2022-01-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4057.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  i_chase

  stp生成过程中，各个交换机是什么时候见面比武的？最终有了生成树以后，只有树上的父子交换机节点才能互相转发报文吗？stp生成树作用是什么呢，没看到

  2021-12-26

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4058.jpeg)

  amber

  vlan可以隔离广播域，那么如果需要不同的VLAN之间通信时，该如何做呢？

  2021-12-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4059.jpeg)

  mickey

  1.STP 协议能够很好地解决环路问题，但是也有它的缺点，你能举几个例子吗？  ==》收敛速度不够快，从而有了 RSTP，MSTP 2.在一个比较大的网络中，如果两台机器不通，你知道应该用什么方式调试吗？  ==》先看线、设备物理连通性，再从物理层往上查，查ARP表，查路由表

  2021-11-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4059.jpeg)

  mickey

  VLANID为12位，即2^12=4096，但只能分4094,0和4095保留

  2021-11-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093713-4060.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  zgrClassTestGroup

  看的不是很明白，有没有简单易懂的书籍推荐

  2021-11-08

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093713-4061.jpeg)

  极客

  看不懂了🤷🤷🤷🤷

  2021-07-31

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093713-4062.jpeg)

  Geek_c6e7a9

  这里的交换机都是三层交换机吗

  2021-07-24

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4063.jpeg)

  鲍勃

  老师，请问下你有研究过工业网络里面常用的光纤环路网吗？你知道这个光纤遇到网络风暴问题是怎么解决的吗？谢谢啊

  2021-07-11

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4064.jpeg)

  一途

  看了评论解决了心中一个大迷惑。VLAN和转发表的区别。一个是对区域一个是对点。STP也真有意思，哈哈哈可能是老师讲的有意思吧。想多了解一些这种（算法？？）知识。

  2021-06-07

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4065.jpeg)

  Gabriel

  就从这个标题开始吧？办公室太复杂，我要回学校，上一章咱们网络是易宿舍为单位，人少，两条交换线，链接就好了。现在呢，到了办公室，办公室几百号人，成千，一栋楼，那得多少人呐。 这样简单的网路机构就不行了，而且网路在传输过程中，都是广播取出，寻址。 面对这么多，网路、就需要单方面去分层了。每个层之间联系。交换机沟通。 公司大了，人多了，每个部门都有自己的工作职责和秘密。当不同部门之间沟通不见得的事时，就不想让其他部门听见，那么就需要将不同部门之间都有一个标识。这样消息在传播的过程中，发现发送出来的tag不是相同的，就不把消息传递给他，达到消息隔离的作用。 ARP：address resolution protocol的简称，中文翻译过来是，地址解析协议。是通过解析网络层地址来找寻数据链路层地址和网络传输协议，他在ipv4中及其重要。

  2021-04-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4066.jpeg)

  布拉姆

  老师，STP原理示意图中，节点和节点之前的路径数值是随意定的吗？比如有的是1，有的是4。仅仅是为了举例而已吗

  2021-04-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093714-4067.jpeg)

  Geek_8866d4

  老师，我想向你你请教一个问题，如果交换机没有确定当前网络是否存在目标MAC就将数据包转发到其他的交换器上去，那这个数据包什么时候停止转发呢

  2021-03-22

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4068.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  Wangyf

  STP 的缺点，我只想到了，毕竟是图和树的转换，计算量可能比较大。文章中只有几个节点，已经用了大段篇幅描述了，如果是实际情况主机数量很多的话，计算量就更大了 别的同学提到的，如果其中有一台机器或者是根节点的交换机没了，需要重新把这个过程来一遍，我是没想到 在学校机房里上课的时候，我看实验室的老师都先 ping 一波百度，看看网线什么的有没有问题。然后再去看角落放着的机柜，再然后，就是重启电脑了.....

  2021-03-18

  **

  **1

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4069.jpeg)

  小东

  VLAN可以格力广播的包，但针对需要广播的包是怎么处理的呢？ 1：DHCP协议需要广播包到DHCP Server, 那需要每个VLAN都配置一个DHCP Server，还是有其他机制 2: VLAN1的A电脑需要和VLAN2的B电脑通讯、A知道B的IP地址，怎么通过ARP获取B电脑的MAC地址，还是说这里已经是跨LAN了，但是他们链接的交换机，也没有路由器的功能，A用什么方式和B通信?

  2021-03-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4070.jpeg)

  Johar

  超哥，自动探测网络拓扑有没有什么推荐的资料？

  2021-02-27

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4071.jpeg)

  我叫不客气

  请问一下，交换机不是有记录MAC的功能吗，比如电脑A第一次发送信息到电脑B，交换机会进行一次广播，成功之后交换机记录了电脑B在那个口，在后面电脑A继续发送信息到电脑B是不是就不用进行广播了？因此就不会产生广播问题和安全问题？

  2021-02-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4072.jpeg)

  www

  妙蛙，把环路结构通过算法变成逻辑上的树型结构

  2020-12-24

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093714-4073.jpeg)

  西电

  交换机中需要广播的内容都是ARP请求啊，具体的数据都不会被广播。为什么还要怕被抓包呢？

  2020-11-23

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4074.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  良凯尔

  如图中的VLAN机器划分图，如果机器1要和机器2通信，机器1先广播ARP协议的包，是不是就不会转发给机器2，所以也知道不了机器2的mac地址，从而通信不了？

  2020-11-23

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4075.jpeg)

  陈德伟

  1.，STP协议的问题：A、太复杂，需要较长时间才能形成层级  B、不能充分利用交换机的性能，可能有的流量大，有的小，但和层级不匹配  2，使用traceroute之类的命令，看是在哪一个转发路径上中断的，然后具体看该路径的路由转发表；或者能拿到网络拓扑查看

  2020-10-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4076.jpeg)

  蚝不鱿鱼

  好好跟大佬学起来，大家加油

  2020-10-19

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093715-4077.png)

  yudh

  老师，环路问题那张图，机器1 发起广播，机器2 是可以收到的，机器2 发起广播，交换机A 和 交换机B 不就都可以收到广播，知道机器2 在哪个局域网了吗？为什么说它们不知道在哪个局域网呢？

  2020-09-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4078.jpeg)

  King-ZJ

  STP收敛通过华山论剑方式来讲解，十分生动形象。这个也不能觉得听懂了，也得自己动手试试。

  2020-09-11

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4079.jpeg)

  浩楠

  老师，VLAN的原理是通过限制ARP请求获取Mac地址么？如果是，说明在不同VLAN口中ARP请求是不能进行的？

  2020-09-10

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4080.jpeg)

  郑祖煌

  1.当这个网络中有机器移除或者宕机的时候就会重新计算，越是 根交换机 或者说是 离根交换机越近的机器 计算的量就越大。 2. 通过ping和抓包工具

  2020-08-19

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4081.jpeg)

  搬瓦工

  我有最近经历了一次断网事件，有很多疑问。事情是这样的，我们学校实验室有根网线连到一个计算机的A1网卡上，通过认证登陆后，又通过这台计算机的另一张A2网卡连到了路由器的wan口，再将路由器的lan口与交换机相连，从而使得我的电脑能够上网。我平时上网很正常，我的电脑和路由器是一个网段，可以通过网关的ip地址登陆路由器。 但是，有一天实验室停电了，来电后就断网了，我去检查的时候发现负责认证的电脑是可以上网的但是，我的电脑没网，也无法登陆路由器。我将路由器lan口插到朋友的笔记本上，因为笔记本的网关和学校的不太一样，所以我设置了笔记本网卡的ip地址和网关地址，随后笔记本就可以上网了并且可以登陆路由器，然后再把网线插回到交换机实验室就来网了。然而出现了一个奇怪的现象，我的电脑虽然可以正常上网，ping路由器也ping得通，可就是无法登陆路由器。 我的问题是： 1. 我的电脑必须和路由器是一个网段才能上网么？ 2. 路由器无线网卡的网段和路由器的有线网卡网段必须不同么？为什么? 3. 能够大致推断一下实验室断网可能的原因么？ 4. 我把笔记本连到路由器lan口后只是设置了笔记本网卡的网关，又把线插回到交换机上为什么就来网了？ 5. 为什么我的电脑无法登陆路由器了？

  2020-08-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4082.jpeg)

  惘 闻

  我有疑问，一开始 1不是已经知道了2的位置了吗，而且包在循环转圈，每次都是广播形式，每次都会知道2的位置吧。为什么老师说转了一圈回来之后1还是不知道，仍然会继续发送广播包呢？

  2020-07-31

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4083.jpeg)

  勇闯天涯

  老师，VLAN通过在二层的头部插入一个TAG后，不是破坏了二层数据包格式了吗？交换机怎么就知道跳过目标MAC和源MAC后，第三个字段就是VLAN ID呢？正常的以太网数据链路层格式中，第三个字段是“类型”，这点没有我没有想通

  2020-07-12

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4084.jpeg)

  Wintersweet

  “程序员啊，你就监听 VLAN 10 吧，里面除了代码，啥都没有”，这句话感觉就像在对着我说。技术岗位做久了，动不动就想抓个包来看看的冲动已经深入骨髓了

  2020-06-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4085.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  湮汐

  不太理解图一，lan1,lan2,lan3应该是同一个网段吧？同一个网段不应该就是一个局域网吗？为啥分三个呢？还是这三个局域网是不同的网段？ 技术太菜了，原谅我提出这么傻逼的问题。

  2020-06-12

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4086.jpeg)

  小侠

  掌门的比喻非常形象

  作者回复: 谢谢

  2020-05-31

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093716-4087.jpeg)

  张舒

  原老大被迫改做不如自己的小弟的小弟，这种是不是会造成原先性能优秀的交换机资源浪费呢？

  2020-05-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4088.jpeg)

  饭粒

  有个疑问：首轮机器1访问机器4时发送ARP广播，它的IP地址和MAC地址会被其他收到广播的机器ARP缓存到吗？如果缓存了，后续机器2访问机器1是不是就不需要ARP请求了？

  2020-05-13

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4089.jpeg)

  意无尽

  总结： 当交换机较少的时候，一开始要传输包的时候，并不知道谁是谁，经过第一次传输后，交换机学习了知道哪个是哪个(拓扑结构)，可以提高传输的效率。 但是，当交换机数量越来越多的时候，会遭遇环路问题，让网络包迷路，这就需要使用 STP 协议，即最小生成树。 STP 的工作过程： 一开始各自为营，每个网桥都被分配了一个 ID，这个 ID 里有管理员分配的优先级。 然后各个节点会逐渐找出树根，慢慢形成树。 如何解决广播问题和安全问题？ 为了解决网络中被抓包导致数据的不安全，有两种方法来解决这样的问题： 1) 物理隔离。每个部门都单独设置一个交换机。单独配置自己的子网，部门之间的沟通靠路由器。 存在的问题：这样的问题在于，有的部门人多，有的部门人少。人少的部门慢慢人会变多，人多的部门也可能人越变越少。如果每个部门有单独的交换机，口多了浪费，少了又不够用。 2) 虚拟隔离，VLAN，或者叫虚拟局域网。在原来的二层的头上加一个 TAG，里面有一个 VLAN ID，一共 12 位。 我们可以设置交换机每个口所属的 VLAN。这样只有相同 VLAN 的包，才会互相转发，不同 VLAN 的包，是看不到的。

  2020-05-06

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4090.jpeg)

  aoe

  比武的例子很容易讲明白了STP协议使用最小生成树的算法解决环路问题的过程。文章很有趣，是看了老师在B站的视频，就毫不犹豫的参加了课程。之前感觉网络协议离我很远，知道一点就行了，也被《TCP/IP详解》这类书的封皮吓到过，觉得这些理论又难又无聊。老师的文章和您在视频中一样风趣幽默。 学习总结： STP 协议 使用最小生成树算法，将有环路的图变成没有环路的树，从而解决环路问题。 缺点：对于跨地域甚至跨国组织的网络支持，很难，因为计算量过大！

  2020-05-01

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4091.jpeg)

  Heaven

  机器1在发送消息给机器2的时候,会发送一个广播包,这时候交换A和机器2都能收到,A会会将广播包转发出去,而机器2收到会返回一个普通数据包,但不是广播的包,这个包只会在第一次交给交换机A的时候去广播出去,后来就学会了,不广播了,是这样吗

  2020-04-22

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4092.jpeg)

  马以

  vlan_id怎么设置呢？

  2020-04-05

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4093.jpeg)

  Captain

  我很想知道，交换机和路由器有什么区别？

  2020-03-27

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4094.jpeg)

  宋健

  学习啦！

  2020-03-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4094.jpeg)

  宋健

  很不错生动形象

  2020-03-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4094.jpeg)

  宋健

  我觉得老师讲的很好啊！

  2020-03-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4094.jpeg)

  宋健

  收获很大

  2020-03-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093715-4078.jpeg)

  King-ZJ

  文中讲STP时以华山论讲，江湖比武为例，是有益于理解STP整个选举和工作过程，但读者能不能先正统学院派，再来接地气地思考和升华更好呢？对于问题1、在STP结合VRRP使用时要二三层统一；2、网络不通两把杀手锏：ping、trancert程序。

  2020-03-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4095.jpeg)

  lakeslove

  所谓的“数字表示优先级”是怎么表示的？感觉从图上看不出来哪个比哪个优先级高啊

  2020-02-28

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4096.jpeg)

  豪曹

  请教一下，第一个图中，这些电脑、交换机组成的不是一个局域网吗，为什么会有3个LAN，还有这个时候这些机器的IP是不是必须在同一个网段下？

  2020-02-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4097.jpeg)

  一个假的程序员

  “使用 VLAN，一个交换机上会连属于多个局域网的机器，那交换机怎么区分哪个机器属于哪个局域网呢”这句话中的局域网指的应该是虚拟局域网吧？

  2020-02-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4098.jpeg)

  肖小强

  老师，我的理解是，stp协议使得在某些交换机下，不如交换机A，在A中包只能从左到右，不能从右到左是吧?那如果右边的机器要向左边的机器发送包，怎么处理的?是不是存在另外一台交换机，它的包的转发方向刚好和A相反?

  2020-02-17

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4099.jpeg)

  kissrain

  第一张图中，2和8中间的连线是1 5和6中间的连线是1，为什么1和7中间的连线是3，这个3代表什么呢？

  2020-02-13

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4100.jpeg)

  吃水不用钱

  在上面的过程中，交换机 A 和交换机 B 都是能够学习到这样的信息：机器 1 是在左边这个网口的。 请问下交互机B如何学习到 机器 1 是在左边这个网口的。 我能理解A是如何学习到 机器 1 是在左边这个网口。但对于B来说，它收到的MAC帧，源mac地址应该是A,B应该只能确定，与自己端口相连的机器里面没有 机器1.

  2020-01-01

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093716-4088.jpeg)

  饭粒

  老师，请问下，文中说的局域网和子网有什么区别？或者说怎么区分？

  2019-11-29

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4101.jpeg)

  美美

  交换机的作用是什么，既然是划分局域网，为什么不用路由器

  2019-11-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4102.jpeg)

  Eleven

  STP的问题我觉得有以下几点： 1.第一“统一整个武林”是需要时间的，在这个期间其实还是会出现环路的问题。 2.第二，这个好像和软件界的一致性算法有点类似，我们这边会出现脑裂的情况，不知道STP是否会有这种情况出现。 3.第三，就像刘超老师说的，天下分久必合，合久必分，是不是间隔多久这个“武林争夺”的过程又会出现，这期间又回再出现环路问题。

  2019-11-18

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4103.jpeg)

  teddytyy

  STP节点优先级变动会导致重新产生最小生成树

  2019-11-13

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4104.jpeg)

  雨后的夜

  我觉得这个比喻很赞也更好理解，总结来说就是，生成树的过程中，节点的级别会发生变化，拓扑也在变化，直到生成最终的一棵树。一开始都是自立山头，PK各自武功收小弟，但后来发现更重要的还要比谁的掌门牛，另外谁离更厉害的掌门近还可以改投其下。靠山 > 距离 > 武功。

  2019-11-06

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4105.jpeg)

  方寸

  有个问题，机器1怎样可以知道机器4的mac地址？

  2019-10-29

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  INFRA_1

  老师，对于环的问题，既然路由器a已经学会了，为什么还会把包发到2中呢？

  2019-10-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4106.jpeg)

  扶幽

  1，可能会消耗的性能比较多，使得延时更大； 2，先ping本地回环地址，看是否可以ping通，判断是否个人主机网卡有问题；再ping同一局域网下其他机子，判断是否局域网下设备出现问题；依次，，，，

  2019-10-23

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093717-4107.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  Geek_4c94d2

  感谢作者。有个疑问，可以使用vlan来划分虚拟局域网。但是这个vlan，是在第二层包头中。那这个vlan的值是由谁来填充的呢？用户的操作系统填充的吗？还是由交换机来填充的

  2019-10-18

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4108.jpeg)

  djfhchdh

  traceroute命令不错

  2019-10-08

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4109.jpeg)

  JoyQueen

  去年买的课，现在才有机会听，很好，受益匪浅，有助于我夯实基础。

  2019-09-19

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093717-4110.png)

  木乃伊

  老师，那个图是真的看不懂，粗线代表什么，细线代表什么，数字代表什么完全找不到对应。还有过多的比喻最后的结果就是忘记了本源吗T_T，感觉到后面我已经搞不懂对应关系了，2333333

  2019-09-19

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4111.jpeg)

  飞飞

  环路问题，当机器1的广播从B左到B右，然后到A右，最后到A左，此时A左发送广播时机器2会主动响应说，这是我的MAC地址，此时不就结束不再循环了吗？ 对此很疑惑，希望老师指点一下 十分感谢

  2019-09-05

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4112.jpeg)

  多襄丸

  大赞 超哥 我从18年5月份买的专栏 当时看不太懂 觉得很吃力 现在才看到第六讲 但是没每讲都理解了 都要做笔记记录超哥的讲解和留言  极客时间的超哥太多了 都喜欢!  等我看完了 凭借这个专栏啃透一半网络协议 我给杭州网易的超哥寄旌旗!

  作者回复: 谢谢，记笔记是个好习惯

  2019-08-28

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093717-4113.jpeg)

  Januarius

  看了几遍终于搞明白了O(∩_∩)O哈哈~

  作者回复: 赞，多看几遍是王道

  2019-08-23

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4114.jpeg)

  -_-

  优先级向量是自己设置的吗，那些ID是怎么来的，交换机本来自带的吗，还是拿来自己设置的

  作者回复: 管理员可以设置

  2019-08-15

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4115.jpeg)

  Roger

  在图一中，当机器2要访问机器1时，机器1会不会收到两个相同的推送？

  作者回复: 不会的

  2019-08-09

  **2

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4116.jpeg)![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,w_14.png)

  糖小宝

  老师您好，看了您讲的STP协议的工作过程，您看我这样理解对不对： 复杂的网络拓扑结构易形成环路 环路会造成广播数据风暴，致使网路阻塞 为解决这一问题，在形成环路的地方应用STP协议打破环路 图4中，1、5、6为环路 应用STP算法1为根交换机 5到1距离为4，6到1距离为2，取最近值 故打破1到5的连接，进而避免环路  但实际运用中，STP协议会对整个网络结构进行计算，无法智能判断局部，效率很低 再加上，网络中若节点改变，根据STP协议就要对整个网络进行重新计算 再再加上，若节点变化频繁，那就需要频繁的重新计算，那效率就低的令人窒息了

  2019-05-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4117.jpeg)

  麦芒小蚱蜢

  我是不是走错片场了，非计算机专业，从事非网络的相关的后端编码工作，本来是抱着学习一下网络相关的内容，虽然老师讲的很有趣，也能听大概明白，但是对于里面出现的名词，大多都很陌生 -_-

  作者回复: 没有，多看几遍就熟了

  2019-05-04

  **2

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f6f02b

  每个网桥都被分配了一个 ID。这个 ID 里有管理员分配，我想问这个优先级有没有可能管理员失误分配错误了，或者机器本身问题性能下降了，能通过BPDU来自动修正优先级吗？

  作者回复: 不能的

  2019-04-19

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4021.jpeg)

  随风

  1向2首次发送消息时，应该先通过arp获得2的mac地址吧？这样交换机A就知道了1和2的mac地址，下次1再向2发消息时，由于消息里有源mac和目的mac，所以交换机就知道了2在左侧，不需要向右边广播了，不知道这样理解对不对。

  作者回复: 是的

  2019-04-15

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093709-4021.jpeg)

  随风

  老师能说详细解答下 @ magict4的问题呢？其实一直想问的是能否提供一个平台（qq群、微信群等）可以让学员相互交流解答问题也好，我看每节课下面都有很多疑问都没有解答。感觉即可平台功能太简陋了，不太适合教学。基础的课堂交流、学生相互交流答疑功能都没有，真说不过去。

  2019-04-15

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  David

  请问在情形二：同门相遇时，下级交换机修改上级关系时，在新建上级关系时是否同时删除旧的关系？

  作者回复: 是的，做了人家的小弟，就不能做人家老大了

  2019-04-07

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4118.jpeg)

  天王

  交换机，用来判断目标mac地址，转到哪台电脑上。第一次不知道转到哪个口，会都转，会记住mac地址来自于哪个口，有回复的话，直接就转出去了。同一个局域网的不同机器之间，用ARP广播，机器1通过ip找机器2，机器2应答，来通过ip找到的mac地址。在一个局域网广播，其他机器和交换机都能收到，如果是在一个局域网，目标机器会应答，否则交换机会广播到其他局域网。2个交换机之间可以形成回环，用华山论剑，将图转成没回环的树。

  2019-03-15

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4119.jpeg)

  传说中的成大大

  我不太明白的是 交换机A B那个图  机器1广播要找机器2  交换机A左边端口收到了广播到局域网2 然后到达 交换机B端口,为啥交换机就发往局域网1了呢 难道交换机B就不会收到这个广播吗？如果交换机B左边端口在最开始也收到了广播又该如何处理呢？

  作者回复: 会啊，所以会广播来，广播去，形成环

  2019-02-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4120.jpeg)

  黄土高坡

  问题一 第一张图中机器1先连接到一个【浅灰色的窄条设备】，这个【浅灰色的窄条设备】才连接到交换机A。想知道这个浅灰色窄条表示的是什么设备？ 问题二 对于图一，有这么一段文字： 当机器 2 要访问机器 1 的时候，机器 2 并不知道机器 1 的 MAC 地址，所以机器 2 会发起一个 ARP 请求。这个广播消息会到达机器 1，也同时会到达交换机 A。 机器2发送的ARP广播会同时到达【机器1和交换机A】，之所以【同时到达】是不是因为这个请求分别通过【窄条设备】到达机器1，和通过【窄条设备】到达交换机A 的？

  2019-02-13

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4121.jpeg)

  陈泰成

  0.比喻清新脱俗，很赞 1.stp缺点主要应该还是网络收敛慢，无负载均衡效果 2.最常用的网络调试当然非属ping工具不可

  2019-02-05

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4122.jpeg)

  古夜

  看这篇之前是不是得去百度找找金庸？岳不群是谁？封不平又是谁？

  2019-01-05

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4122.jpeg)

  古夜

  如何解决常见的环路问题？上面的一段，交换机A怎么知道1在A的局域网内？B怎么就知道没有？

  2019-01-02

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093718-4123.png)

  Oliveryoung

  https://www.youtube.com/watch?v=4jXzuEu-qYE 还不清楚的童鞋建议看这个视频讲解 5分钟大概

  2018-12-29

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4124.jpeg)

  Tyella

  刘老师讲的太好了，一口气学了5节，根本停不下来。

  2018-12-25

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093718-4125.jpeg)

  caohuan

  本篇 所得：1.接入到网络的设备越多，需要更多的交换机，交换机 很多，会形成 环路（闭环），这会不停的消耗资源，纸质耗尽资源，可以通过STP协议解决环路，让环路 变成 没有闭环的树 2.很多的交换机 面临隔离问题，可以通过 VLAN形成虚拟局域网，能解决 广播 和 安全问题。 回答老师第一个问题：STP协议的缺点1.效率不高 2.消耗资源比较多。

  2018-12-22

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4126.jpeg)

  罗格

  一开始的5-6，1-7，2-8，3-4是假设还是有什么依据这样初始化的？？

  2018-12-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4127.jpeg)

  程序员大天地

  精彩！

  2018-12-17

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093708-4015.jpeg)

  王先统

  如果一个交换机所连接两个局域网所配置的ip地址段是一样的，它们之间还能通信吗？我如何指定另一个局域网的ip而不是自己所在局域网的同一个ip?

  2018-12-14

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093719-4128.png)

  乖鱼

  交换机B是如何收到机器1的广播包的，是由机器3转发的吗

  作者回复: 这个的细节参考答疑环节

  2018-11-24

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4129.jpeg)

  Ada

  假如 1 和 6 相遇。6 原来就拜在 1 的门下，只不过 6 的上司是 5，5 的上司是 1。1 发现，6 距离我才只有 2，比从 5 这里过来的 5（=4+1）近多了，那 6 就直接汇报给我吧。 老师，这里的6距离1只有2，5到1需要4+1，这是怎么算出来的啊？还是仅仅只是发个比方，这里听的有点晕。

  作者回复: 路径的距离有个值，相加

  2018-11-22

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4130.jpeg)

  最琴箫

  老师，图二中的环路，用stp的方式是怎么解除环路的呢？

  作者回复: 会有一个边被禁掉

  2018-11-20

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4131.jpeg)

  Sisyphus

  可不可以理解广播风暴只会发生在交换机存在并联的场景

  2018-11-18

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4042.jpeg)

  阿拉D

  刘老师好！ 原文此处，“当机器 2 要访问机器 1 的时候，机器 2 并不知道机器 1 的 MAC 地址，所以机器 2 会发起一个 ARP 请求。这个广播消息会到达机器 1，也同时会到达交换机 A。这个时候交换机 A 已经知道机器 1 是不可能在右边的网口的，所以这个广播信息就不会广播到局域网二和局域网三。” 我有几个疑问： ①机器2发起的ARP请求中，只有机器1的IP地址，交换机是根据IP地址得知机器1是不可能在右边的网口的？ ②如果是根据IP地址，那是在之前机器1发送ARP请求时获得的？ ③如果是，那交换机不仅摘下了帧头，还看了ARP里面的数据？

  2018-11-13

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4132.jpeg)

  金龟

  老师请问下，vlan在交换机上的设置 ，是不是相当于 交换机一个口只能对应一个vlan

  2018-10-26

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4133.jpeg)

  Pakhay

  我记得读书的时候学到vlan是三层的知识了，vlan划分不同网段至少应该使用三层交换机才能够通信，请老师解惑。

  作者回复: 是的

  2018-10-23

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4134.jpeg)

  云上漫步

  机器1广播，广播请求先到交换机，然后交换机会广播，然后机器2才收到。是这样过程吗，如果不是，那机器2收到机器1广播，交换机再广播，那机器2还会再一次收到吗

  作者回复: 一般广播是，除了来的口，其他都发送

  2018-09-18

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093719-4134.jpeg)

  云上漫步

  机器1广播ARP请求，机器2收到，是交换机广播后机器2收到的，还是机器1广播收到的

  作者回复: 机器一广播

  2018-09-18

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4135.jpeg)

  马若飞

  STP选掌门的过程感觉有点像Paxos的选举，当然理念相同，算法完全不同了

  2018-09-18

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  wrzgeek

  情形2，1和6比的时候是怎么比的，6不是不能发送BPDU吗？

  作者回复: 6会转发他们掌门的

  2018-09-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4136.jpeg)

  冷磊

   拓扑结构的地方：当机器 3 要访问机器 1 的时候，也需要发起一个广播的 ARP 请求。这个时候交换机 A 和交换机 B 都能够收到这个广播请求。交换机 A 当然知道主机 A 是在左边这个网口的，所以会把广播消息转发到局域网一。 应该是“交换机 A 当然知道 机器 1（主机 A）A是在左边这个网口的”。。原文表述有歧义

  2018-08-15

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4137.jpeg)

  千年孤独

  说了这么多，回到最开始的交换机A和交换机B图,还没说应用了stp协议后如何破环？

  作者回复: 变成树就没有环了

  2018-07-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4138.jpeg)

  生煎馒头

  老师那个STP 是不是就是最短路径算法吗？

  2018-07-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4137.jpeg)

  千年孤独

  在那江湖纷争图中，如果1和2的位置互换，并且2是7的掌门。 如果这时1掌门和7相遇了，与2掌门决斗胜了。 那么此时2的上司是谁？是7吗？

  2018-07-29

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4139.jpeg)

  yao

  最近正好在看MST算法，看了这篇文章知道算法的切实用处了，再次感谢

  2018-07-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  小咯咯

  请教下，stp协议讲解中，假如1和6距离是3，两边距离是一样的如何标记从哪边走

  2018-07-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4140.jpeg)

  烬默

  什么协议会去检测是否有环路吗？如果没有环路使用stp性能应该会下降很多

  2018-06-21

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4141.jpeg)

  -Nao剧-

  在stp那块看懵了。

  2018-06-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4142.jpeg)

  hunterlodge

  STP中的距离是指通信时间吗？如果是的话，岂不是会频繁更新拓扑？

  2018-06-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4142.jpeg)

  hunterlodge

  请问VLAN是如何配置的？就是说某台主机的VLANID是如何确定的呢？

  2018-06-16

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/132-1662307093720-4143.png)

  宇宙尘埃

  STP协议大学时就没学会，如今从事云计算行业这么多年，面对复杂的网络规划时，出现网络问题时，依然还是没理解透彻！

  2018-06-05

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4144.jpeg)

  楚人

  老师，Priority Vector和Bridge ID需要详细解释一下，枝只能转发根的Bpdu

  2018-06-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4145.jpeg)

  风太息

  一个二层的广播包封装了arp协议，这个广播包到了一个2层的交换机，1.因为是2层交换机不识别arp信息，2. 2层交换机默认根据mac地址转发，默认将这个2层广播转发除接收端口外的所有其他端口，比如这个是主机1请求主机2Mac的2层广播包封装了arp，数据包到是交换机，交换机就算知道主机2的Mac地址，它的行为默认也是广播到除接收端口外的所有接口，因为是2层交换机不识别arp协议，并且2层的mac地址是广播mac，2层交换机只根据mac地址转发，如果是一个3层的交换机有arp表，交换机也不会拆开2层为广播地址而不是自己mac地址的包，它的行为也是丢给除接收端口外的所有接口，这样对吧

  2018-06-04

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4146.jpeg)

  左思

  老师有个问题不明白。第二个图中说交换机A和B组成环路，且交换机都学会了，机器1在局域网一中。当机器1的包到达交换机A时，既然交换机A已经知道机器1在左边网络，为什么还会将包广播到局域网二中？ 就是这样的，源主机需要知道目的mac是谁，所以需要交换机继续转发。一旦目的mac的主机回应后，交换机就学会了目的主机的mac地址。两台主机间的通信之路构建完毕，从而停止广播之后两台主机的通信。但是老化时间到了，继续之前的行为。

  作者回复: 原来的广播包是一直在的，arp没有ttl的

  2018-06-02

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093712-4048.jpeg)

  刘意

  交换机建立了mac地址表后不会转发已知mac地址的arp广播吗？看到前面的介绍感觉就是这个意思，想要确认一下

  2018-06-01

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4147.jpeg)

  痴痴

  环路问题：机器2在返回mac地址的时候，交换机不就知道机器2属于哪个局域网了吗？怎么会像你文章中说的一直左转左转呢？？

  作者回复: 是的，请看下一段，说了这个学会以后的问题

  2018-05-31

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093720-4148.jpeg)

  云学

  请问交换机可以设置某个口为出口吗？发往外部的包都从这个口出去？交换机的每个口ip都是同网段吗？路由器的每个口ip可以不同吗？一台pc用网线直连交换机，那网线两端的网口ip是一样吗？

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093721-4149.jpeg)

  晓迪

  老师，不好意思，刚才写错了，是ARP协议。

  作者回复: 对的，你说nat把我说晕了

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093721-4149.jpeg)

  晓迪

  老师，LAN1,LAN2,LAN3里所有机器的IP应该是属同一个网络号的吧？这里交换机应该是二层交换机吧？NAT协议只是在目标IP地址和源IP地址同属一个网络才起作用吧？若目标IP地址和源IP地址不在同一个网络，源机器系统不会利用NAT找目标MAC地址，而是应该将数据包直接发给源机器所在网络的网关吧？（当然可能会用NAT找网关的MAC地址）。 我的理解准确吗？

  作者回复: 是的，同一个二层里面。

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093721-4150.jpeg)

  fang

  形成环路的那个例子里机器1的广播不只会发给交换机a左边的口，也会发给交换机b左边的口。两个交换机又会分别广播到右边的子网，这样会形成两条环路吧？导致网络中的包越来越多。 stp那个例子有点不太明白，root path cost是两台交换机连线上的值吗？这个是不是也是管理员指定的？还是怎么生成的？

  作者回复: 是的

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093721-4151.jpeg)

  冯选刚

  学到了，跟小说一样的爽文。

  作者回复: 谢谢

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093721-4152.jpeg)

  chino

  请问vlan存在安全问题吗？比如说可以突破vlan的限制访问到其它vlan的设备。

  2018-05-30

  **

  **

- ![img](%E7%AC%AC6%E8%AE%B2%20_%20%E4%BA%A4%E6%8D%A2%E6%9C%BA%E4%B8%8EVLAN%EF%BC%9A%E5%8A%9E%E5%85%AC%E5%AE%A4%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E6%88%91%E8%A6%81%E5%9B%9E%E5%AD%A6%E6%A0%A1.resource/resize,m_fill,h_34,w_34-1662307093707-3997.jpeg)

  颇忒妥

  有些交换机的规格说明中说自己支持802.1Q VLAN 和Port-based VLAN 。一直没搞明白这两有啥区别。

  2018-05-30

  **

  **

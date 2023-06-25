# 第12讲 \| TCP协议（下）：西行必定多妖孽，恒心智慧消磨难

我们前面说到玄奘西行，要出网关。既然出了网关，那就是在公网上传输数据，公网往往是不可靠的，因而需要很多的机制去保证传输的可靠性，这里面需要恒心，也即各种**重传的策略**，还需要有智慧，也就是说，这里面包含着**大量的算法**。

## 一、如何做个靠谱的人？

TCP想成为一个成熟稳重的人，成为一个靠谱的人。那一个人怎么样才算靠谱呢？咱们工作中经常就有这样的场景，比如你交代给下属一个事情以后，下属到底能不能做到，做到什么程度，什么时候能够交付，往往就会有应答，有回复。这样，处理事情的过程中，一旦有异常，你也可以尽快知道，而不是交代完之后就石沉大海，过了一个月再问，他说，啊我不记得了。

**对应到网络协议上，就是客户端每发送的一个包，服务器端都应该有个回复，如果服务器端超过一定的时间没有回复，客户端就会重新发送这个包，直到有回复**。

这个发送应答的过程是什么样呢？可以是**上一个收到了应答，再发送下一个**。这种模式有点像两个人直接打电话，你一句，我一句。但是这种方式的缺点是效率比较低。如果一方在电话那头处理的时间比较长，这一头就要干等着，双方都没办法干其他事情。咱们在日常工作中也不是这样的，不能你交代你的下属办一件事情，就一直打着电话看着他做，而是应该他按照你的安排，先将事情记录下来，办完一件回复一件。在他办事情的过程中，你还可以同时交代新的事情，这样双方就并行了。

如果使⽤这种模式，其实需要你和你的下属就不能靠脑⼦了，⽽是要都准备⼀个本⼦，你每交代下属⼀个事情，双方的本子都要记录⼀下。

当你的下属做完⼀件事情，就回复你，做完了，你就在你的本⼦上将这个事情划去。同时你的本⼦上每件事情都有时限，如果超过了时限下属还没有回复，你就要主动重新交代⼀下：上次那件事情，你还没回复我，咋样啦？

既然多件事情可以一起处理，那就需要给每个事情编个号，防止弄错了。例如，程序员平时看任务的时候，都会看JIRA的ID，而不是每次都要描述一下具体的事情。在大部分情况下，对于事情的处理是按照顺序来的，先来的先处理，这就给应答和汇报工作带来了方便。等开周会的时候，每个程序员都可以将JIRA ID的列表拉出来，说以上的都做完了，⽽不⽤⼀个个说。

## 二、如何实现一个靠谱的协议？

TCP协议使用的也是同样的模式。为了保证顺序性，每一个包都有一个ID。在建立连接的时候，会商定起始的ID是什么，然后按照ID一个个发送。为了保证不丢包，对于发送的包都要进行应答，但是这个应答也不是一个一个来的，而是会应答某个之前的ID，表示都收到了，这种模式称为**累计确认**或者**累计应答**（**cumulative acknowledgment**）。

为了记录所有发送的包和接收的包，TCP也需要发送端和接收端分别都有缓存来保存这些记录。发送端的缓存里是按照包的ID一个个排列，根据处理的情况分成四个部分。

**第一部分：发送了并且已经确认的。这部分就是你交代下属的，并且也做完了的，应该划掉的。**

**第二部分：发送了并且尚未确认的。这部分是你交代下属的，但是还没做完的，需要等待做完的回复之后，才能划掉。**

**第三部分：没有发送，但是已经等待发送的。这部分是你还没有交代给下属，但是马上就要交代的。**

**第四部分：没有发送，并且暂时还不会发送的。这部分是你还没有交代给下属，而且暂时还不会交代给下属的。**

这里面为什么要区分第三部分和第四部分呢？没交代的，一下子全交代了不就完了吗？

这就是我们上一节提到的十个词口诀里的“流量控制，把握分寸”。作为项目管理人员，你应该根据以往的工作情况和这个员工反馈的能力、抗压力等，先在心中估测一下，这个人一天能做多少工作。如果工作布置少了，就会不饱和；如果工作布置多了，他就会做不完；如果你使劲逼迫，人家可能就要辞职了。

到底一个员工能够同时处理多少事情呢？在TCP里，**接收端会给发送端报一个窗口的大小**，叫**Advertised window**。这个窗口的大小应该等于上面的第二部分加上第三部分，就是已经交代了没做完的加上马上要交代的。超过这个窗口的，接收端做不过来，就不能发送了。

于是，发送端需要保持下面的数据结构。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/dd67ba62279a3849c11ffc1deea25d44.jpg>)

- LastByteAcked：第一部分和第二部分的分界线

- LastByteSent：第二部分和第三部分的分界线

- LastByteAcked + AdvertisedWindow：第三部分和第四部分的分界线

对于接收端来讲，它的缓存里记录的内容要简单一些。

第一部分：接受并且确认过的。也就是我领导交代给我，并且我做完的。

第二部分：还没接收，但是马上就能接收的。也即是我自己的能够接受的最大工作量。

第三部分：还没接收，也没法接收的。也即超过工作量的部分，实在做不完。

对应的数据结构就像这样。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/9d597af268016f67caa14178627188be.jpg>)

- MaxRcvBuffer：最大缓存的量；

- LastByteRead之后是已经接收了，但是还没被应用层读取的；

- NextByteExpected是第一部分和第二部分的分界线。

第二部分的窗口有多大呢？

NextByteExpected和LastByteRead的差其实是还没被应用层读取的部分占用掉的MaxRcvBuffer的量，我们定义为A。

AdvertisedWindow其实是MaxRcvBuffer减去A。

也就是：AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)。

那第二部分和第三部分的分界线在哪里呢？NextByteExpected加AdvertisedWindow就是第二部分和第三部分的分界线，其实也就是LastByteRead加上MaxRcvBuffer。

其中第二部分里面，由于受到的包可能不是顺序的，会出现空档，只有和第一部分连续的，可以马上进行回复，中间空着的部分需要等待，哪怕后面的已经来了。

## 三、顺序问题与丢包问题

示例：还是刚才的图，在发送端来看，1、2、3已经发送并确认；4、5、6、7、8、9都是发送了还没确认；10、11、12是还没发出的；13、14、15是接收方没有空间，不准备发的。

在接收端来看，1、2、3、4、5是已经完成ACK，但是没读取的；6、7是等待接收的；8、9是已经接收，但是没有ACK的。

发送端和接收端当前的状态如下：

- 1、2、3没有问题，双方达成了一致。

- 4、5接收方说ACK了，但是发送方还没收到，有可能丢了，有可能在路上。

- 6、7、8、9肯定都发了，但是8、9已经到了，但是6、7没到，出现了乱序，缓存着但是没办法ACK。

根据这个例子，我们可以知道，顺序问题和丢包问题都有可能发生，所以我们先来看**确认与重发的机制**。

假设4的确认到了，不幸的是，5的ACK丢了，6、7的数据包丢了，这该怎么办呢？

一种方法就是**超时重试**，也即对每一个发送了，但是没有ACK的包，都有设一个定时器，超过了一定的时间，就重新尝试。但是这个超时的时间如何评估呢？这个时间不宜过短，时间必须大于往返时间RTT，否则会引起不必要的重传。也不宜过长，这样超时时间变长，访问就变慢了。

估计往返时间，需要TCP通过采样RTT的时间，然后进行加权平均，算出一个值，而且这个值还是要不断变化的，因为网络状况不断地变化。除了采样RTT，还要采样RTT的波动范围，计算出一个估计的超时时间。由于重传时间是不断变化的，我们称为**自适应重传算法**（**Adaptive Retransmission Algorithm**）。

如果过一段时间，5、6、7都超时了，就会重新发送。接收方发现5原来接收过，于是丢弃5；6收到了，发送ACK，要求下一个是7，7不幸又丢了。当7再次超时的时候，有需要重传的时候，TCP的策略是**超时间隔加倍**。**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍**。**两次超时，就说明网络环境差，不宜频繁反复发送。**

超时触发重传存在的问题是，超时周期可能相对较长。那是不是可以有更快的方式呢？

有一个可以快速重传的机制，当接收方收到一个序号大于下一个所期望的报文段时，就会检测到数据流中的一个间隔，于是它就会发送冗余的ACK，仍然ACK的是期望接收的报文段。而当客户端收到三个冗余的ACK后，就会在定时器过期之前，重传丢失的报文段。

例如，接收方发现6收到了，8也收到了，但是7还没来，那肯定是丢了，于是发送6的ACK，要求下一个是7。接下来，收到后续的包，仍然发送6的ACK，要求下一个是7。当客户端收到3个重复ACK，就会发现7的确丢了，不等超时，马上重发。

还有一种方式称为**Selective Acknowledgment** （**SACK**）。这种方式需要在TCP头里加一个SACK的东西，可以将缓存的地图发送给发送方。例如可以发送ACK6、SACK8、SACK9，有了地图，发送方一下子就能看出来是7丢了。

## 四、流量控制问题

我们再来看流量控制机制，在对于包的确认中，同时会携带一个窗口的大小。

我们先假设窗口不变的情况，窗口始终为9。4的确认来的时候，会右移一个，这个时候第13个包也可以发送了。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/af16ecdfabf97f696d8133a20818fd87.jpg>)

这个时候，假设发送端发送过猛，会将第三部分的10、11、12、13全部发送完毕，之后就停止发送了，未发送可发送部分为0。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/e011cb0e56f43bae942f0b7ab7407b35.jpg>)

当对于包5的确认到达的时候，在客户端相当于窗口再滑动了一格，这个时候，才可以有更多的包可以发送了，例如第14个包才可以发送。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/f5a4fcc035d1bb2d7e11c38391d768c2.jpg>)

如果接收方实在处理的太慢，导致缓存中没有空间了，可以通过确认信息修改窗口的大小，甚至可以设置为0，则发送方将暂时停止发送。

我们假设一个极端情况，接收端的应用一直不读取缓存中的数据，当数据包6确认后，窗口大小就不能再是9了，就要缩小一个变为8。

![](<https://static001.geekbang.org/resource/image/95/9d/953e6706cfb5083e1f25b267505f5c9d.jpg?wh=2400*771>)

这个新的窗口8通过6的确认消息到达发送端的时候，你会发现窗口没有平行右移，而是仅仅左面的边右移了，窗口的大小从9改成了8。

![](<https://static001.geekbang.org/resource/image/0a/1f/0a9265c63d5e0fb08c442ea0a7cffa1f.jpg?wh=2481*933>)

如果接收端还是一直不处理数据，则随着确认的包越来越多，窗口越来越小，直到为0。

![](<https://static001.geekbang.org/resource/image/c2/4a/c24c414c31bd5deb346f98417ecdb74a.jpg?wh=2304*897>)

当这个窗口通过包14的确认到达发送端的时候，发送端的窗口也调整为0，停止发送。

![](<https://static001.geekbang.org/resource/image/89/cb/89fe7b73e40363182b13e3d9c9aa2acb.jpg?wh=2724*1053>)

如果这样的话，发送方会定时发送窗口探测数据包，看是否有机会调整窗口的大小。当接收方比较慢的时候，要防止低能窗口综合征，别空出一个字节来就赶快告诉发送方，然后马上又填满了，可以当窗口太小的时候，不更新窗口，直到达到一定大小，或者缓冲区一半为空，才更新窗口。

这就是我们常说的流量控制。

## 五、拥塞控制问题

最后，我们看一下拥塞控制的问题，也是通过窗口的大小来控制的，前面的滑动窗口rwnd是怕发送方把接收方缓存塞满，而拥塞窗口cwnd，是怕把网络塞满。

这里有一个公式 LastByteSent - LastByteAcked <= min {cwnd, rwnd} ，是拥塞窗口和滑动窗口共同控制发送的速度。

那发送方怎么判断网络是不是慢呢？这其实是个挺难的事情，因为对于TCP协议来讲，他压根不知道整个网络路径都会经历什么，对他来讲就是一个黑盒。TCP发送包常被比喻为往一个水管里面灌水，而TCP的拥塞控制就是在不堵塞，不丢包的情况下，尽量发挥带宽。

水管有粗细，网络有带宽，也即每秒钟能够发送多少数据；水管有长度，端到端有时延。在理想状态下，水管里面水的量=水管粗细 x 水管长度。对于到网络上，通道的容量 = 带宽 × 往返延迟。

如果我们设置发送窗口，使得发送但未确认的包为为通道的容量，就能够撑满整个管道。

![](<%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/c467d450d8000472e690ed378b8019c6.jpeg>)

 如图所示，假设往返时间为8s，去4s，回4s，每秒发送一个包，每个包1024byte。已经过去了8s，则8个包都发出去了，其中前4个包已经到达接收端，但是ACK还没有返回，不能算发送成功。5-8后四个包还在路上，还没被接收。这个时候，整个管道正好撑满，在发送端，已发送未确认的为8个包，正好等于带宽，也即每秒发送1个包，乘以来回时间8s。

如果我们在这个基础上再调大窗口，使得单位时间内更多的包可以发送，会出现什么现象呢？

我们来想，原来发送一个包，从一端到达另一端，假设一共经过四个设备，每个设备处理一个包时间耗费1s，所以到达另一端需要耗费4s，如果发送的更加快速，则单位时间内，会有更多的包到达这些中间设备，这些设备还是只能每秒处理一个包的话，多出来的包就会被丢弃，这是我们不想看到的。

这个时候，我们可以想其他的办法，例如这个四个设备本来每秒处理一个包，但是我们在这些设备上加缓存，处理不过来的在队列里面排着，这样包就不会丢失，但是缺点是会增加时延，这个缓存的包，4s肯定到达不了接收端了，如果时延达到一定程度，就会超时重传，也是我们不想看到的。

于是TCP的拥塞控制主要来避免两种现象，**包丢失**和**超时重传**。一旦出现了这些现象就说明，发送速度太快了，要慢一点。但是一开始我怎么知道速度多快呢，我怎么知道应该把窗口调整到多大呢？

如果我们通过漏斗往瓶子里灌水，我们就知道，不能一桶水一下子倒进去，肯定会溅出来，要一开始慢慢的倒，然后发现总能够倒进去，就可以越倒越快。这叫作慢启动。

一条TCP连接开始，cwnd设置为一个报文段，一次只能发送一个；当收到这一个确认的时候，cwnd加一，于是一次能够发送两个；当这两个的确认到来的时候，每个确认cwnd加一，两个确认cwnd加二，于是一次能够发送四个；当这四个的确认到来的时候，每个确认cwnd加一，四个确认cwnd加四，于是一次能够发送八个。可以看出这是**指数性的增长**。

涨到什么时候是个头呢？有一个值ssthresh为65535个字节，当超过这个值的时候，就要小心一点了，不能倒这么快了，可能快满了，再慢下来。

每收到一个确认后，cwnd增加1/cwnd，我们接着上面的过程来，一次发送八个，当八个确认到来的时候，每个确认增加1/8，八个确认一共cwnd增加1，于是一次能够发送九个，变成了线性增长。

但是线性增长还是增长，还是越来越多，直到有一天，水满则溢，出现了拥塞，这时候一般就会一下子降低倒水的速度，等待溢出的水慢慢渗下去。

拥塞的一种表现形式是丢包，需要超时重传，这个时候，将sshresh设为cwnd/2，将cwnd设为1，重新开始慢启动。这真是一旦超时重传，马上回到解放前。但是这种方式太激进了，将一个高速的传输速度一下子停了下来，会造成网络卡顿。

前面我们讲过**快速重传算法**。当接收端发现丢了一个中间包的时候，发送三次前一个包的ACK，于是发送端就会快速地重传，不必等待超时再重传。TCP认为这种情况不严重，因为大部分没丢，只丢了一小部分，cwnd减半为cwnd/2，然后sshthresh = cwnd，当三个包返回的时候，cwnd = sshthresh + 3，也就是没有一夜回到解放前，而是还在比较高的值，呈线性增长。

![](<https://static001.geekbang.org/resource/image/19/d2/1910bc1a0048d4de7b2128eb0f5dbcd2.jpg?wh=923*613>)

就像前面说的一样，正是这种知进退，使得时延很重要的情况下，反而降低了速度。但是如果你仔细想一下，TCP的拥塞控制主要来避免的两个现象都是有问题的。

**第一个问题**是丢包并不代表着通道满了，也可能是管子本来就漏水。例如公网上带宽不满也会丢包，这个时候就认为拥塞了，退缩了，其实是不对的。

**第二个问题**是TCP的拥塞控制要等到将中间设备都填充满了，才发生丢包，从而降低速度，这时候已经晚了。其实TCP只要填满管道就可以了，不应该接着填，直到连缓存也填满。

为了优化这两个问题，后来有了**TCP BBR拥塞算法**。它企图找到一个平衡点，就是通过不断地加快发送速度，将管道填满，但是不要填满中间设备的缓存，因为这样时延会增加，在这个平衡点可以很好的达到高带宽和低时延的平衡。

![](<https://static001.geekbang.org/resource/image/a2/4c/a2b3a5df5eca52e302b75824e4bbbd4c.jpg?wh=762*508>)

## 小结

好了，这一节我们就到这里，总结一下：

- 顺序问题、丢包问题、流量控制都是通过滑动窗口来解决的，这其实就相当于你领导和你的工作备忘录，布置过的工作要有编号，干完了有反馈，活不能派太多，也不能太少；

- 拥塞控制是通过拥塞窗口来解决的，相当于往管道里面倒水，快了容易溢出，慢了浪费带宽，要摸着石头过河，找到最优值。

最后留两个思考题：

1. TCP的BBR听起来很牛，你知道他是如何达到这个最优点的嘛？

2. 学会了UDP和TCP，你知道如何基于这两种协议写程序吗？这样的程序会有什么坑呢？

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(97)

- ![img](%E7%AC%AC12%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8B%EF%BC%89%EF%BC%9A%E8%A5%BF%E8%A1%8C%E5%BF%85%E5%AE%9A%E5%A4%9A%E5%A6%96%E5%AD%BD%EF%BC%8C%E6%81%92%E5%BF%83%E6%99%BA%E6%85%A7%E6%B6%88%E7%A3%A8%E9%9A%BE.resource/resize,m_fill,h_34,w_34.jpeg)

  进阶的码农

  置顶

  AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)。 我根据图中例子计算 14-((5-1)-0) 算出来是10 ，括号里边的-1是减的什么，为啥和图例算出来的结果不一样，还是我计算的有问题，麻烦详细说一下 谢谢

  作者回复: 这里我写的的确有问题，nextbyteexpected其实是6，就是目前接收到五，下一个期望的是六，这样就对了

  2018-06-13

  **6

  **46

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/a5/e4c1c2d4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小文同学

  1 设备缓存会导致延时？ 假如经过设备的包都不需要进入缓存，那么得到的速度是最快的。进入缓存且等待，等待的时间就是额外的延时。BBR就是为了避免这些问题： 充分利用带宽；降低buffer占用率。 2 降低发送packet的速度，为何反而提速了？ 标准TCP拥塞算法是遇到丢包的数据时快速下降发送速度，因为算法假设丢包都是因为过程设备缓存满了。快速下降后重新慢启动，整个过程对于带宽来说是浪费的。通过packet速度-时间的图来看，从积分上看，BBR充分利用带宽时发送效率才是最高的。可以说BBR比标准TCP拥塞算法更正确地处理了数据丢包。对于网络上有一定丢包率的公网，BBR会更加智慧一点。 回顾网络发展过程，带宽的是极大地改进的，而最小延迟会受限与介质传播速度，不会明显减少。BBR可以说是应运而生。 3 BBR如何解决延时？ S1：慢启动开始时，以前期的延迟时间为延迟最小值Tmin。然后监控延迟值是否达到Tmin的n倍，达到这个阀值后，判断带宽已经消耗尽且使用了一定的缓存，进入排空阶段。 S2：指数降低发送速率，直至延迟不再降低。这个过程的原理同S1 S3：协议进入稳定运行状态。交替探测带宽和延迟，且大多数时间下都处于带宽探测阶段。 深夜读了BBR的论文和网上大牛的讲解得出的小结，分享给大家，过程比较匆忙，不足之处也望老师能指出指正。

  2018-06-14

  **5

  **213

- ![img](https://static001.geekbang.org/account/avatar/00/11/3f/c5/58787480.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大唐江山

  作者辛苦啊，这一章读起来开始有点累了😂

  2018-06-13

  **1

  **82

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  BBR 论文原文：https://queue.acm.org/detail.cfm?id=3022184

  作者回复: 就应该这样，一言不合就读论文，是最好的方式

  2019-08-27

  **4

  **57

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  华林

  这么说起来，我们经常发现下载速度是慢慢的增加到顶峰，然后就在那个值的附近徘徊，原因就是tcp的流量控制喽？

  2018-06-13

  **9

  **33

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c2/1e/ec02941d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  食用淡水鱼

  快速重传那一段有问题。接收端接收6，7，8，9，10时漏掉了7，不是连续发送3个6ack，而是收到6，发送6的确认ack，收到8，9，10各发送一个6的重复ack，发送端检测到3个重复ack时（加上确认ack，总共有4个ack），进入重传机制。包括下面的拥塞控制，也是类似的逻辑

  2019-01-21

  **3

  **24

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/8f/51f044dc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谛听

  不太清楚累积应答，比如接收端收到了包1、2、3、4，它的应答应该是5吗？也就是说中间的包就不用应答了吗？

  作者回复: 是的

  2018-09-22

  **4

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/15/18/bb/9299fab1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Null

  BBR 不填满缓存还是不填缓存？不填缓存那么缓存干啥用，如果填了了，即使不满，但是不是还有延迟。。。

  作者回复: 填的少，延迟就少了，当然做不到完全避免，毕竟缓存是路径上每一个设备自己的事情。缓存是每个设备自己的设计选择，BBR算法是两端的算法。 就像买火车票，我建议网上购买，身份证刷进去，这样速度快，但是对于火车站还是要设置人工窗口，因为不是每个人都会选择速度快的方式的。

  2019-03-27

  **

  **17

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTL4mxgSAKHpHI2aNmG6EicicJekPRzBMXUC7TPxKYvEYhyA2RnpyT6ELicmwaiaTic7ibnWFAAOBaxDN0dg/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  卜

  好奇什么级别的程序开发需要了解怎么细，开发了好多网络程序都没用到，重传这些都是应用层在做

  2018-06-24

  **6

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/11/58/4b/34e7ceca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  秋去冬来

  快速重传那块6.8.9   7丢了为什么会发送3个冗余的6的3个ack

  作者回复: 六的ack里面强调下一个是七

  2018-06-22

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/b3/9e8f4b4f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MoFanDon

  5的ACK丢包了，出发发送端重发。重发过去后，接收端发现接收过了，丢弃。……如果接收端丢弃5后，没有继续发送ACK,那这样不是发送端永远也也没法接受到5的ACK？

  2018-07-24

  **3

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/6f/63/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扬～

  2个问题： 1. TCP可靠的连接会不会影响到业务层，比如超时重传导致了服务端函数调用2次，那岂不是业务都要考虑幂等性了，我很懵逼，果然是懂得越多越白痴。 2. 拥塞控制的窗口跟流量控制的窗口一回事吗，还是流量控制的窗口的出来后就会进入拥塞控制窗口？

  作者回复: 不会，重传的包不会交给应用层。是一个窗口

  2018-11-01

  **9

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  老师，TCP协议栈，保证包一定到吗，，哪几种情况下会丢失，，，能不能总结下

  作者回复: 不能保证，只是尽力重试，再重试

  2018-06-13

  **2

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/12/68/5d/1ccee378.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  茫农

  有一个值 ssthresh 为 65535 个字节，，这个是什么意思？

  作者回复: slow start threshold

  2018-10-23

  **2

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/10/3b/36/2d61e080.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  行者

  问题1，想到BBR可以根据ACK时间来判断，比如同一时刻发送了A、B、C三个包，A、B两个包10ms收到ACK，而C包20ms后收到ACK，那么就认为网络拥堵或被中间设备缓存，降低发送速度。 问题2，TCP优点在于准确到达，可靠性高，但是速度慢；UDP优点在于简单，但是不确认可达；像后端接口一般使用TCP协议，因为客户端和服务器之间会有多次交互，且请求数据要确认可达；但是如果是直播的话使用UDP会更好，管你网络咋样，反正我赶紧发，让用户等急了就不好了。 刘老师，有一个问题，接口有时候会受到2次相同时间相同内容的请求，但是客户端仅仅调用了接口一次，会是什么原因导致这个问题呢？TCP重复发送包导致的嘛？

  2018-06-13

  **1

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/13/68/ef/6264ca3d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Magic

  祝刘超老师教师节快乐，专栏很棒，受益良多

  作者回复: 谢谢

  2019-09-10

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/0f/88/cd/2c3808ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Yangjing

  对于发送端，为什么会保存着“已发送并已确认”的数据呢？已确认的不是已经没用了吗？

  作者回复: 是的，可以回收了，这里是分一下类

  2019-06-22

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/fc/75/af67e5cc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑猫紧张

  内核是将从网络接收的tcp数据 都接收完成再一次发给应用层呢 还是在tcp接收的过程中就已经开始发给应用层了 求回复

  作者回复: 接收的过程中就发给应用层了

  2019-05-29

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得： 1.数据在不同客户端传输 需要经过网关，在TCP传输层，用有不同任务的编号JIRA ID 标注 那些 包 已发送，哪些包 有反馈，灵活的做到 4个分类（1）发送 完成 且有反馈 （2）发送完成 没有反馈（3）将发送（4）不会发送，这可能是没有 带宽和空间，也可以知道 那些包 的ack丢了、数据丢了，这个可以使用 滑动窗口 的缓存机制 来处理，然后 通过调节 滑动窗口的大小 灵活应对 顺序问题 和丢包 问题 以及流量控制，最后还可以 灵活给TCP 指派 任务量，多少可以完成，多少可以待处理，多少不能被处理。 2.拥塞控制 处理 网络带宽塞满，防止 包丢失，超时重传 带来的 低延时 和 带宽利用率低的情况，所以 采用 TCP BBR拥塞算法 提高带宽利用率 和 低延时，并且不占缓存 的 高效方法。 回答老师的问题，第一个问题1.TCP的BBR很牛，如何达到最优点，老师 提到过 （1）高效率利用 带宽，让带宽 处于 满贯状态 （2）不占用缓存空间（3）低延时， 一旦空间被 占满，会流入 缓冲空间时，马上降速，降低传输速度，一直到有空余空间 再慢启动 提高 带宽利用率。 第二个问题：1.UDP和TCP应用在不同的应用场景下， UDP性善论，网络 一片和谐，有任务只管发送 和 接受，不太理会 网络环境 和 资源，不会主动调整速度;TCP 性恶论，世界很黑暗，得制定规则，如果 网络资源好，多传输数据，不好的环境，降低 传输速度，还得注重 顺序问题、丢包问题、控制情况、带宽流量等现实情况。 2.TCP的坑;TCP需要 考虑 带宽资源、阻塞、延时等各种问题，里面设计很多算法和数据结构，都没完全捋清 里面的原理，还不知道 坑在哪里，期待继续听 刘超老师的课程，都遇问题，然后解决问题 ，来填补TCP的坑。

  2019-01-16

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8f/ad/6e3e9e15.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  产品助理

  问题： 1、如下公式的 -1  到底是为什么？ 	AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)    图例中的LastByteRead是0还是1？NextByteExpected是6还是5？MaxRcvBuffer是14吗？ 2、如果按照上述公式，那下面又是为了什么？ 	NextByteExpected 加 AdvertisedWindow 就是第二部分和第三部分的分界线，其实也就是 LastByteRead 加上 MaxRcvBuffer。 	按照第一条的公式，NextByteExpected + AdvertisedWindow = NextByteExpected + （MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)) = MaxRcvBuffer + 1 + LastByteRead 	应该有个+1啊。。 多谢老师！	

  作者回复: 这里我写的的确有问题，nextbyteexpected其实是6，就是目前接收到五，下一个期望的是六，这样就对了 2018-06-13

  2018-11-22

  **2

  **4

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/4AyMqvLia5xW0d1CxlPYoHJk2LQhaDzAialczCIuRKibiczWHkJIra0DpmtxREwibtPiajk3NhQVSicXMKxB0Oyb5GEsg/132)

  旭风

  在传统算法和快速算法的对比描述中，对于快速算法中提到 cwnd减半为cwnd/2，sshthresh=cwnd ，后面cwnd=sshthresh+3，转折点是20变为13，这里的cwnd为10吗？ 2018-06-15  作者回复 cwnd先是为10，后面变为13 继上一个问题，传统算法，cwnd 从1,2,4,8,16指数增长，这时cwnd为16，收发是不是各占8个？然后由16变成17，满8个确认时cwnd＋1，就是17，那么增加到20这里，cwnd为20，这时产生拥塞，如果是传统的话，立马降下来，如果是快速，那么减半，也就是10，等待3个确认包＋3，，后续每一个确认cwnd就＋1吗？还是？这样子理解吗？

  作者回复: 是的

  2018-06-19

  **

  **4

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/pTD8nS0SsORKiaRD3wB0NK9Bpd0wFnPWtYLPfBRBhvZ68iaJErMlM2NNSeEibwQfY7GReILSIYZXfT9o8iaicibcyw3g/132)

  雷刚

  \1. rwnd：滑动窗口。上一讲提到的 TCP 报文中的 windown 字段，表示接收方将能接收的数据大小告知发送发方。 2. cwnd：拥塞窗口。TCP 发送方维护的一个变量，表示发送方一次要发送多少字节的数据。 总而言之，rwnd 用来控制能发送多少数据，cwnd 控制发送速度。这就和我们吃饭一样，虽然能吃两碗饭，但你不能一口就全部吃完，还是要一口口吃。

  2020-04-09

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  这一章感觉比前面好一些, 比喻比较形象.也挺详细. 谢谢, 就是有一点音频和文字不完全一样, 有时候边看变听有点跟不上

  作者回复: 音频更能表达我的思想，哈哈哈

  2019-05-21

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/14/78/ac/e5e6e7f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古夜

  最后一个图是画的最清楚的了🤷‍♂️

  2019-02-23

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/37/1b/82310e20.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  拿笔小星

  怎么办，越到后面越看不懂了，没有网络基础，老湿有什么建议吗？

  2018-12-08

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/37/ce/65bf8dc9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lyz

  jira太真实了

  2018-07-24

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  https://medium.com/google-cloud/tcp-bbr-magic-dust-for-network-performance-57a5f1ccf437

  2019-08-27

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/fa/ab/0d39e745.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李小四

  网络_12 看了一些资料，还是没有最终弄明白，只知道了Cubic是基于丢包的，BBR是基于测量的。 关于UDP与TCP的程序，它们的选型以及“坑”都与特点密切相关。 - UDP特点是快，不可靠。所以在需要无法容忍高时延的场景要选择它，当然，这个时候，一些必要校验和重发逻辑就留给了应用层，这里应该是最大的“坑”。 - TCP特点是可靠，慢。在我开发过的程序中，“坑”比较多地存在于长连接的keepalive阶段，需要在资源消耗与稳定性之间取得平衡。

  作者回复: 加油，多看几遍

  2019-08-07

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/27/2c/919e5773.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Abirdcfly

  问个问题。最刚开始ssthres 是65535字节.然后一个mss是1460字节。那么指数增长应该是到44才变慢啊。为啥是16呢？

  作者回复: 举个例子而已，重点是那个曲线，而非数值

  2019-06-13

  **

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKficy9MkomP4658kicbqNwoAibL71byWowtjnrMqS3rdqTicTl41hCGu7Jjf0Dp6J7YZcd43yQevuaJw/132)

  Geek_hustnw

  （1）文章提到下面这段话：    每收到一个确认后，cwnd 增加 1/cwnd，我们接着上面我们接着上面的过程来，一次发送八个，当八个确认到来的时候，每... （2）然后快速重传算法里面提到拥塞时：    cwnd 减半为 cwnd/2，然后 sshthresh =... 问题：快速重传拥塞处理里面，为什么遇到3个包返回的时候，是“sshthresh + 3”，而不是“sshthresh + 3/cwnd”

  作者回复: 先慢启动一把

  2019-04-14

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/63/f4/6fdc1508.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈栋

  第三部分和第四部分分界线写错了，应该是lastByteSent+advertisedWindow

  作者回复: 没错的，advertisewindow是那个黑框，不是接下来能发的部分

  2018-06-19

  **

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/4AyMqvLia5xW0d1CxlPYoHJk2LQhaDzAialczCIuRKibiczWHkJIra0DpmtxREwibtPiajk3NhQVSicXMKxB0Oyb5GEsg/132)

  旭风

  在传统算法和快速算法的对比描述中，对于快速算法中提到 cwnd减半为cwnd/2，sshthresh=cwnd ，后面cwnd=sshthresh+3，转折点是20变为13，这里的cwnd为10吗？

  作者回复: cwnd先是为10，后面变为13

  2018-06-15

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/20/1d/0c1a184c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  罗辑思维

  接受方的缓存数据分为三部分：接受并确认，等待接受未确认，不能接受。 有个不理解的地方 如果对应发送方的几部分，接受方不是应该有个「接受未确认」。

  2020-03-08

  **2

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/7c/99/4dac6ce6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lakeslove

  解释一下ssthresh： 慢启动阈值「ssthresh」的概念，如果「cwnd」小于「ssthresh」，那么表示在慢启动阶段；如果「cwnd」大于「ssthresh」，那么表示在拥塞避免阶段，此时「cwnd」不再像慢启动阶段那样呈指数级整整，而是趋向于线性增长，以期避免网络拥塞，此阶段有多种算法实现。

  2020-03-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/05/d3/3ca27b1b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我D

  发送端和接收端的角色是固定的吗？一直都是单向的？还有，能讲一下思考题吗，自己找的不一定对啊，又不是老师可以修改作业，那不是白学了

  2020-02-06

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/08/92f42622.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  尔冬橙

  为什么填缓存还会加大时延，有缓存不是更快了么

  作者回复: 这是指网络设备的缓存

  2019-07-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/52/d0/b7cbde62.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zjw

  所以我每次用uplay这种服务器很烂的应用下游戏时速度有时候会一下子掉下来，然后又慢慢回上去。是因为tcp在传输过程中丢包了，然后自动降低了传输速度了吗

  作者回复: 很可能是的

  2019-07-14

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLTPmSgD6QSgicqsbzibiau9xWSYgdsvYlnVWBg91ibHQBYg39MT4O3AV5CHlJlVUvw9Ks9TZEmRvicfTw/132)

  InfoQ_0ef441e6756e

  请问老师，比如发送方 发送了 【1，2，3，4】，接收方回了3的ack，这种情况窗口怎么动啊？   

  作者回复: 累积ack，前三个都算结束了

  2019-07-07

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/14/06eff9a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jerry银银

  请教老师一个概念性的问题：packet   chunk   segment  block   这几个词在“网络”文章或者书籍中经常初出现，感觉它们很相似，又感觉它们又有些细微的区别，但是具体的也说不清！

  作者回复: 不同层次的叫法不一样

  2019-05-17

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/txBsdmdDUEzC356b5sICOJdw8Ls7blql0vWeXbeWQQw6NEQtk85JBIvGGdwLTXxJYVTmicFbITfnQwUofEhGQqA/132)

  wildmask

  如果接收方发现7丢了，为什么不能直接要求重发7，发送3个ack 有必要吗？

  作者回复: 3个ack就表示要求重发7呀，要不然咋要求。

  2019-05-07

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/b5/98/ffaf2aca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ronnyz

  接收端是采用累计确认机制的吗

  作者回复: 是的

  2019-04-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/99/d3/5434e6e0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  德道

  当三个包返回的时候，cwnd = sshthresh + 3，当三个包返回是什么意思，为什么要加3

  作者回复: 每返回一个加一

  2019-04-09

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/18/bb/9299fab1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Null

  而且，老师行文过程，哪些是现象，问题，算法解决方案，目前最优，或者广泛采用的是哪个没有清晰的脉络，很容易看完一遍一头雾水。

  作者回复: 哪有最优，都是根据各自的场景做选择而已。

  2019-03-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/0b/90/f161b09a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵加兴

  不是很理解第三部分和第四部分的分界线，老师能说明下么

  作者回复: 超过这个不能发送了

  2019-01-14

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/74/f4/fb729388.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  (-.-)

  "快速重传，接收方发现 6、8、9 都已经接收了，就是 7 没来，发送三个6的ack，要求下一个是7.客户端收到 3 个，就会发现 7 的确又丢了，不等超时.." 这个不应该是服务端收到三个6的ack包吗?

  作者回复: 客户端收到6的ack，就说明7不在，因为6的ack里面要求下一个是7

  2019-01-07

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/51/33/56f8bcbe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yinhua_upc

  "6、7 是等待接收的；8、9 是已经接收，但是没有 ACK 的" 从接收端的图看6-14都是等待接收未确认，为什么8、9是已经接收？

  作者回复: 假设，所以画了实线

  2018-11-09

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/8b/b3/c340227c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  GL

  网络层分片和传输层分片是怎么进行的？

  2018-06-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e9/91/4219d305.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  初学者

  fack 应该不是收到一个大于期望的包发送三个ack，收到一个包只会回复这个ack吧

  2018-06-14

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/c5/3467cf94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  正是那朵玫瑰

  为大牛点赞，有点复杂，先慢慢消化下；先问个问题，tcp建立连接后，每发送一个数据包都要像三次握手一样来回三次么，发送数据-发送ack-回复ack，才算数据发送成功？

  2018-06-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1e/42/3a/bca96a7f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  人什么人

  老师, 慢启动门限到底是sshthresh还是sshresh

  2022-07-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  GMA029

  接收端的数据结构不清晰。语音说6和7是虚线，但是看图片还是实线。这对初学者很容易造成困扰

  2022-06-26

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/23/6c/82ba5e1f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LIKE

  还请各位大哥大姐们帮忙解答，问题： 1.接收端的LastByteRead 在哪个位置？ 2.6、7、8、9 肯定都发了，但是 8、9 已经到了，但是 6、7 没到，出现了乱序，缓存着但是没办法 ACK。   从哪里可以看到 6和7没有到？

  2022-05-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/d4/a1/8bc8e7e1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  赌神很低调

  请问下老师，如何写好高性能的网络程序，尽量占用带宽，比如大数据传输，想尽量把带宽打满。如万兆网卡的两个节点传数据，最大带宽可以达到1GB/s,但我们写的程序测出吞吐量最大约为350MB/s，请问可以通过哪些手段来提高吞吐量，逼近带宽呢？

  2022-04-27

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIKoEicqUZTJly55qoUXRmK4wia7YbnibsMncJaO6tKgKAQNJRfpMsibvfeiaukIibsCsuaic8QjQ3gOoTGA/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  张可夫斯基

  数学不好，这个点+点=范围？

  2022-03-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/65/b7/058276dc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  i_chase

  文章里提到的也是教科书里的拥塞控制算法，是Reno算法。Linux默认的应该是cubic拥塞控制算法，三次函数，初始增速较快。BBR是基于带宽时延积计算判断网络拥塞情况的，reno和cubic是根据丢包判断网络是否拥堵。文章里也说了，丢包无法准确反映出网络拥塞（管子漏水，中间设备缓冲区的存在）

  2021-12-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0c/46/dfe32cf4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多选参数

  “cwnd 减半为 cwnd/2，然后 sshthresh = cwnd，当三个包返回的时候，cwnd = sshthresh + 3，也就是没有一夜回到解放前，而是还在比较高的值，呈线性增长。” +3 是怎么来？

  2021-10-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/6f/70/01f50897.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大年

  强烈建议作者重写下这节，既然有错误为什么不修改呢？我们作为消费者是花钱买的课想要更准确的知识 输出，这里不是论坛的免费分享

  2021-10-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/c0/58/6908f7d1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  dbant

  参考一起看：https://queue.acm.org/detail.cfm?id=3022184

  2021-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  老师，我看网上对于低能窗口综合症，提到了Nagle算法和延迟ACK机制，能讲一下两者的关系吗，就比如说网络中默认启用的机制是什么，什么情况下会启用

  2021-02-03

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  K菌无惨

  感觉把滑动窗口改成接收窗口就比较好理解了

  2021-01-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ff/08/7c18d8a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  团

  老师好，对发送速度和窗口大小的算法关系还不是十分清楚，文中说通过控制窗口大小来控制发送速度，具体是怎么一个算法？比如窗口是14时，对应的是每秒发送多少个包或者多少字节？窗口大小为8时又是对应每秒发送多少包或多少字节。比如文中拥塞控制时的举例，是每秒发送一个包，窗口大小和带宽容量有关，那这样看来窗口大小和‘每秒发送一个包’这个速度没有什么直接联系？文中有一处讲如果发送方发的过猛，把四个包都发送了，这个’猛’是怎么控制的？

  2020-09-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/42/b0/50b8faa2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lixf

  老师，这节课逻辑很清楚啊，点个赞

  2020-08-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/57/d8/5816eb6b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  啦啦啦

  哈哈，正好之前系统查redis慢，sar发现redis有丢包的情况，明天研究研究

  2020-08-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/3c/84/608f679b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  连边

  开始吃力，记录一下。

  2020-07-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/16/b7/b69ded05.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ai

  拥塞控制和流量有什么区别, 流量控制窗口不是已经控制了发送者频率了吗?

  2020-07-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/21/00600713.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小侠

  图非常形象

  作者回复: 谢谢

  2020-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/13/e7/6e75012c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谭倩倩_巴扎黑

  不是说累计确认累计应答么，为什么我看到的例子都是发送一个包返回一个ack?不是累计确认？

  2020-05-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/1d/de/62bfa83f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aoe

  看完很激动，滑动窗口是限流神器啊

  2020-05-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  这一节听得的确懵啦。接触了一些未曾接触过的概念。看来越往深海，越是寒冷，哈哈哈哈

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/c5/2e/0326eefc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Larry

  流量控制的对象是发送端和接收端，目的是使得接收方处理的过来。 拥塞控制的对象是带宽，目的是让找出传输量最优。

  2020-03-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  TCP中的顺序、丢包和拥塞问题，都涉及到一个窗口大小的事情。文中以工作中上司安排下属任务为例来说明（一个是发送端、一个是接收端），一是上司安排任务时，为了提升效率，以ID号标识一件事情（以防乱序），二是下属领会到上司传达的任务后，要有响应，别石沉大海让领导干着急（解决丢包），三是领导通过和下属长期相处过程中，知道下属的能力和潜力，在安排任务时要量力而为，别拔苗助长。回到TCP的知识体系中，其中很多细节是需要去琢磨的，比喻是为了让读者更靠近指示器，而不是忽略知识。

  2020-03-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  这个tcp协议讲的，牛！服了！

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  看到这个TCP的包丢失和包重传，其中涉及的窗口大小还是有一些晕的，但是不会放过它的，就如何曾别人放过你一样，加油钱。多画画、多看看、多想想，就多收获。

  2020-03-21

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTI4akcIyIOXB2OqibTe7FF90hwsBicxkjdicUNTMorGeIictdr3OoMxhc20yznmZWwAvQVThKPFWgOyMw/132)

  Chuan

  “当三个包返回的时候，cwnd = sshthresh + 3，当三个包返回是什么意思，为什么要加3” 老师，请问下： 1. 这里的三是有什么特指吗，为什么是3不是2？ 2. 另外您回复说这三个包返回一个加一，那是不是是从10到11、12，再到13的。 3. 最后，这三个包应该就是指之前发过的包吧？还是特定的什么包？

  2020-02-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/90/8f/9c691a5f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  奔跑的码仔

  下面是对于滑动窗口和拥塞窗口的一点个人的理解： 1.滑动窗口是关于TCP发送和接收端来说的，用于控制顺序问题、丢包问题以及流量控制等问题，这些问题也都是端-端的问题。 2.拥塞窗口是关于TCP发送和接收端之间传输链路容量的问题。

  2019-12-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  这节确实对照音频看要顺畅，好理解些。

  2019-12-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/bd/62/283e24ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雨后的夜

  这一课有点烧脑啊

  2019-11-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b1/52/40540f9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sky

  cwnd为20时产生拥塞，是快速重传的话减半，也就是cwnd和ssthresh都是10。但为什么“等待3个确认包＋3”？cwnd到达门限后，不是应该每个确认包 + 1/cwnd么？

  2019-11-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/62/f873cd8f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tongmin_tsai

  老师，如果发送方发了1 2 3 4 5，接收方目前接收了1 2 4 5，3丢了，那么接收方会把1 2发给应用层吗？那如果这样的话，4 5怎么办？存储在哪里？如果很多包都丢了中间的，那所有接收到的包都存储，存储不会爆掉吗？

  2019-10-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/7a/03/51399175.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mr_Tree

  前几章看起来还有点轻松，这一张看起来有点难懂，晦涩，看着看着就想睡觉，不知道为什么，老师能给些能上手的练习吗？纯理论容易反馈

  2019-09-19

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEIaTvOKvUt4WnuSjkBp0tjd6O6vvVyw5fcib3UgZibE8tz2ICbTfkwbzs8MHNMJjV6W2mLjywLsvBibg/132)

  火力全开

  目的端口通常我们都比较清楚，想请教一下源端口是怎么来的？

  作者回复: 随机分配

  2019-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/30/2e/81cb76ad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吴林峰 🏀

  AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead) 这个计算感觉有问题啊，不应该是：AdvertisedWindow=MaxRcvBuffer-(NextByteExpected-LastByteRead+1) 么？

  作者回复: AdvertisedWindow=MaxRcvBuffer-[(NextByteExpected-1)-LastByteRead]这个是对的。NextByteExpected - 1是第一部分和第二部分的分界线

  2019-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/74/ab/82de1d38.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨帆

  老师，能结合最近爆出的TCP SACK漏洞详细讲解一下SACK吗

  2019-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/65/c3/5324b326.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半清醒

  感觉老师的课，确实是要科班出身读起来要轻松许多！

  2019-05-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/e9/98b6ea61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员大天地

  老师这个拥塞控制用水管倒水比喻相当好，很容易理解，👍

  2019-04-17

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEL5xnfuicbtRz4F87AAjZX6oCEjMtYiaIu4iaQichQmy0vEBA6Sumic1RDvUCeuBEqj6iatnt2kENbKYmuw/132)

  dexter

  这章还是 tcp握手建立链接过程吧，不包括具体数据的传输。

  作者回复: 是数据的传输呀

  2019-02-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/46/b8/407b1494.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  媛

  接受方的缓存数据分为三部分：接受并确认，等待接受未确认，不能接受。那后面的例子，接受方8、9收到了没有ask，是啥意思，8、9属于上面哪一部分数据呢？上面三部分并没有接受未确认数据类型啊？

  作者回复: 第二部分。因为没有ack，客户端就会认为没有发成功，还会再发的。

  2019-02-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  克里斯

  talk is cheap,show me the code.这些内容教材上也讲了，能不能看看内核代码如何实现流量控制，拥塞控制呢？

  作者回复: 内核是个大事情，不是一两句话说的清楚的。可以参考相关书籍

  2019-01-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/31/1a/fd82b2d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘士涛

  请教下像tcpcopy这类请求复制工具的原理是啥，怎么处理丢包问题？

  2019-01-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/64/a9/27d63f2e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  佳楠

  不仅学会了网络知识，还学会了工作方式，真正的触类旁通吧！

  2019-01-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fc/a8/db8095c5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  航仔

  第三部分的和第四部分的分界线应该是lastByteAcked + lastByteSent吧，3 + 9 = 12，如果是 lastByteAcked + AdvertisedWindow的话就是 3 + 12 = 15，如果老师说的没有问题，麻烦老师解释下

  作者回复: advertise window指向的是黑框，是9

  2018-11-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/a2/da/a8a32113.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  太子长琴

  发送和接受的图为什么不能对应起来呢？

  2018-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/21/20/1299e137.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  秋天

  滑动窗口移动计算没明白

  2018-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/34/26/abcb00fd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Roy

  真正深入浅出，这专栏物超所值

  2018-08-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/b3/9e8f4b4f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MoFanDon

  顺序问题和丢包问题中的例子，接收端6789，从图上看，都是等待接收未确认，是如何判断6789乱序的呢？

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  老师能不能问个深得的问题，mqtt为什么有的嵌入TCP协议栈，，TCP的重传与mqtt  qos保证，重复了qos1或者2应用层重复，这个感觉不对劲啊，冲突啊

  2018-06-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  根据ack划掉，不是根据ttl划掉，要是极端情况，，client发送端重传包出去，这时接到server ack，清除，，，server端收到包给上层，发送ack清除buffer区，这时重发包在ttl内到达，不是会重复吗

  2018-06-13



# cookie详解

**背景**

在HTTP协议的定义中，采用了一种机制来记录客户端和服务器端交互的信息，这种机制被称为cookie，cookie规范定义了服务器和客户端交互信息的格式、生存期、使用范围、安全性。

在JavaScript中可以通过 document.cookie 来读取或设置这些信息。由于 cookie 多用在客户端和服务端之间进行通信，所以除了JavaScript以外，服务端的语言（如PHP）也可以存取 cookie。

**Cookie详解**

Cookie在远程浏览器端存储数据并以此跟踪和识别用户的机制。从实现上说，Cookie是存储在客户端上的一小段数据，浏览器（即客户端）通过HTTP协议和服务器端进行Cookie交互。

Cooke独立于语言存在，严格地说，Cookie并不是由PHP、Java等语言实现的，而是由这些语言对Cookie进行间接操作，即发送HTTP指令，浏览器收到指令便操作Cookie并返回给服务器。因此，Cookie是由浏览器实现和管理的。举例说，PHP并没有真正设置过Cookie，只是发出指令让浏览器来做这件事。PHP中可以使用setcookie() 或 setrawcookie() 函数设置Cookie。setcookie()最后一个参数HttpOnly设置了后，JavaScript就无法读取到这个Cookie。



**设置Cookie时需注意：**

①函数有返回值，false失败，true成功，成功仅供参考，不代表客户端一定能接收到；

②PHP设置的Cookie不能立即生效，要等下一个页面才能看到（Cookie从服务器传给浏览器，下个页面浏览器才能把设置的Cookie传回给服务器）；如果是JavaScript设置的，是立即生效的；

③Cookie没有显示的删除函数，可以设置expire过期时间，自动触发浏览器的删除机制。

Cookie是HTTP头的一部分，即现发送或请求Cookie，才是data域；setcookie()等函数必须在数据之前调用，这和header() 函数是相同的。不过也可以使用输出缓冲函数延迟脚本的输出，知道设置好所有Cookie和其他HTTP标头。

Cookie通常用来存储一些不是很敏感的信息，或者进行登录控制，也可用来记住用户名、记住免密码登录、防止刷票等。每个域名下允许的Cookie是有限制的，根据浏览器这个限制也不同。Cookie不是越多越好，它会增加宽带，增加流量消耗，所以不要滥用Cookie；不要把Cookie当作客户端的存储器来用。一个域名的每个Cookie限制以4千字节（KB）键值对的形式存储。

还有一种Cookie是Flash创建的，成为Flash Shard Object，又称Flash Cookie，即使清空浏览器所有隐私数据，这类顽固的Cookie还会存在硬盘上，因为它只受Flash管理，很多网站采用这种技术识别用户。

Cookie跨域，主要是为了统一应用平台，实现单点登录；需使用P3P协议（Platform for Privacy Preferences），通过P3P使用户自己可以指定浏览器的隐私策略，达到存储第三方Cookie的目的，只需要在响应用户请求时，在HTTP的头信息中增加关于P3P的配置信息就可以了。Cookie跨域涉及两个不同的应用，习惯上称为第一方和第三方。第三方通常是来自别人的广告、或Iframe别的网站的URL，这些第三方网站可能使用的Cookie。



**Cookie格式**

Cookie中保存的信息都是文本信息，在客户端和服务器端交互过程中，cookie信息被附加在HTTP消息头中传递，cookie的信息由键/值对组成。下面是一个HTTP头中cookie的例子：
`Set-Cookie: key = value; Path=/`
Cookie中存放的信息包含cookie本身属性和用户自定义属性，一个cookie只能包含一个自定义键/值对。Cookie本身属性有”Comment” 、”Domain”、”Max-Age”、”Path”、”Secure”、”Version”。

- Comment 属性是cookie的产生着对该cookie的描述；

- Domain 属性定义可访问该cookie的域名，对一些大的网站，如果希望cookie可以在子网站中共享，可以使用该属性。例如设置Domain为 .bigsite.com ,则sub1.bigsite.com和sub2.bigsite.com都可以访问已保存在客户端的cookie，这时还需要将Path设置为/。

* Max-Age 属性定义cookie的有效时间，用秒计数，当超过有效期后，cookie的信息不会从客户端附加在HTTP消息头中发送到服务端。

* Path 属性定义网站上可以访问cookie的页面的路径，缺省状态下Path为产生cookie时的路径，此时cookie可以被该路径以及其子路径下的页面访问；可以将Path设置为/，使cookie可以被网站下所有页面访问。

* Secure 属性值定义cookie的安全性，当该值为true时必须是HTTPS状态下cookie才从客户端附加在HTTP消息中发送到服务端，在HTTP时cookie是不发送的；Secure为false时则可在HTTP状态下传递cookie，Secure缺省为false。

* Version 属性定义cookie的版本，由cookie的创建者定义。



**Cookie的创建**

Cookie可以在服务器端创建，然后cookie信息附加在HTTP消息头中传到客户端，如果cookie定义了有效期，则本保存在客户端本地磁盘。保存cookie的文件是一个文本文件，因此不用担心此文件中的内容会被执行而破坏客户的机器。支持Web端开发的语言都有创建cookie的方法或函数，以及设置cookie属性和添加自定义属性的方法或函数，最后是将cookie附加到返回客户端的HTTP消息头中。

创建cookie时如果不指定生存有效时间，则cookie只在浏览器关闭前有效，cookie会在服务器端和客户端传输，但是不会保存在客户机的磁盘上，打开新的浏览器将不能获得原先创建的cookie信息。

Cookie信息保存在本地时会保存到当前登录用户专门目录下，保存的cookie文件名中会包含创建cookie所在页面网站的域名，当浏览器再次连接该网站时，会从本机cookie存放目录下选出该网站的有效cookie，将保存在其中的信息附加在HTTP消息头中发送到服务器端，服务器端程序就可根据上次保存在cookie的信息为访问客户提供“记忆”或个性化服务。

Cookie除了可以在服务器端创建外，也可以在客户端的浏览器中用客户端脚本(如javascript)创建。客户端创建的cookie的性质和服务器端创建的cookie一样，可以保存在本地，也可以被传送到服务器端被服务器程序读取。



**Cookie 基础知识**

1.  cookie 是有大小限制的，大多数浏览器支持最大为 4096 字节的 Cookie(具体会有所差异，可以使用这个好用的工具:http://browsercookielimits.squawky.net/ 进行测试);如果 cookie 字符串的长度超过最大限制，则该属性将返回空字符串。

2.  由于 cookie 最终都是以文件形式存放在客户端计算机中，所以查看和修改 cookie 都是很方便的，这就是为什么常说 cookie 不能存放重要信息的原因。

3.  每个 cookie 的格式都是这样的：cookieName = Vaue；名称和值都必须是合法的标示符。

4.  cookie 是存在 有效期的。在默认情况下，一个 cookie 的生命周期就是在浏览器关闭的时候结束。如果想要 cookie 能在浏览器关掉之后还可以使用，就必须要为该 cookie 设置有效期，也就是 cookie 的失效日期。

5.  alert(typeof document.cookie)结果是 string.

6.  cookie 有域和路径这个概念。域就是domain的概念，因为浏览器是个注意安全的环境，所以不同的域之间是不能互相访问 cookie 的(当然可以通过特殊设置的达到 cookie 跨域访问)。路径就是routing的概念，一个网页所创建的 cookie 只能被与这个网页在同一目录或子目录下得所有网页访问，而不能被其他目录下得网页访问（这句话有点绕，一会看个例子就好理解了）。

7.  其实创建cookie的方式和定义变量的方式有些相似，都需要使用 cookie 名称和 cookie 值。同个网站可以创建多个 cookie ，而多个 cookie 可以存放在同一个cookie 文件中。

8.  cookie 存在两种类型：①:你浏览的当前网站本身设置的 cookie ②来自在网页上嵌入广告或图片等其他域来源的 第三方 cookie (网站可通过使用这些 cookie 跟踪你的使用信息)

9.  cookie 有两种清除方式：①:通过浏览器工具清除 cookie (有第三方的工具，浏览器自身也有这种功能) ②通过设置 cookie 的有效期来清除 cookie.注：删除 cookie 有时可能导致某些网页无法正常运行。

10.  浏览器可以通过设置来接受和拒绝访问 cookie。出于功能和性能的原因考虑，建议尽量降低 cookie 的使用数量，并且要尽量使用小 cookie。



**Cookie的使用**



从cookie的定义可以看到，cookie一般用于采用HTTP作为进行信息交换协议的客户端和服务器端用于记录需要持久化的信息。一般是由服务器端创建要记录的信息，然后传递到客户端，由客户端从HTTP消息中取出信息，保存在本机磁盘上。当客户端再次访问服务器端时，从本机磁盘上读出原来保存的信息，附加到HTTP消息中发送给服务器端，服务器端从HTTP消息中读取信息，根据实际应用的需求进行进一步的处理。



服务器端cookie的创建和再次读取功能通常由服务器端编程语言实现，客户端cookie的保存、读取一般由浏览器来提供，并且对cookie的安全性方面可以进行设置，如是否可以在本机保存cookie。



由于cookie信息以明文方式保存在文本文件中，对一些敏感信息如口令、银行帐号如果要保存在本地cookie文件中，最好采用加密形式。



与cookie类似的另一个概念是会话（Session），会话一般是记录客户端和服务器端从客户端浏览器连接上服务器端到关闭浏览器期间的持久信息。会话一般保存在内存中，不保存到磁盘上。会话可以通过cookie机制来实现，对于不支持cookie的客户端，会话可以采用URL重写方式来实现。可以将会话理解为内存中的cookie。



使用会话会对系统伸缩性造成负面影响，当服务器端要在很多台服务器上同步复制会话对象时，系统性能会受到较大伤害，尤其会话对象较大时。这种情况下可以采用cookie，将需要记录的信息保存在客户端，每次请求时发送到服务器端，服务器端不保留状态信息，避免在服务器端多台机器上复制会话而造成的性能下降。



**Cookie 基本操作**



对于 Cookie 得常用操作有，存取，读取，以及设置有效期；具体可以参照 JavaScript 操作 Cookie 一文；但，近期在前端编码方面，皆以Vue为冲锋利器，所以就有用到一款插件 vue-cookie,其代码仅30行，堪称精妙，读取操作如下：


```language
set: function (name, value, days) {
var d = new Date;
d.setTime(d.getTime() + 24*60*60*1000*days);
window.document.cookie = name + "=" + value + ";path=/;expires=" + d.toGMTString();
},
get: function (name) {
var v = window.document.cookie.match('(^|;) ?' + name + '=([^;]*)(;|$)');
return v ? v[2] : null;
},
delete: function (name) {
this.set(name, '', -1);
}

```



**cookie 域概念**

路径能解决在同一个域下访问 cookie 的问题，咱们接着说 cookie 实现同域之间访问的问题。语法如下：
`document.cookie = “name=value;path=path;domain=domain“`

红色的domain就是设置的 cookie 域的值。例如 “www.qq.com” 与 “sports.qq.com” 公用一个关联的域名”qq.com”，我们如果想让”sports.qq.com” 下的cookie被 “www.qq.com” 访问，我们就需要用到cookie 的domain属性，并且需要把path属性设置为 “/“。例：
`document.cookie = “username=Darren;path=/;domain=qq.com“`

注：一定的是同域之间的访问，不能把domain的值设置成非主域的域名。



**cookie 安全性**

通常 cookie 信息都是使用HTTP连接传递数据，这种传递方式很容易被查看，在控制台下运行document.cookie,一目了然；所以 cookie 存储的信息容易被窃取。假如 cookie 中所传递的内容比较重要，那么就要求使用加密的数据传输。所以 cookie 的这个属性的名称是“secure”，默认的值为空。如果一个 cookie 的属性为secure，那么它与服务器之间就通过HTTPS或者其它安全协议传递数据。语法如下：
`document.cookie = “username=Darren;secure”`

把cookie设置为secure，只保证 cookie 与服务器之间的数据传输过程加密，而保存在本地的 cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。

注： 就算设置了secure 属性也并不代表他人不能看到你机器本地保存的 cookie 信息，所以说到底，别把重要信息放cookie就对了。



**Session详解**

Session即回话，指一种持续性的、双向的连接。Session与Cookie在本质上没有区别，都是针对HTTP协议的局限性而提出的一种保持客户端和服务器间保持会话连接状态的机制。Session也是一个通用的标准，但在不同的语言中实现有所不同。针对Web网站来说，Session指用户在浏览某个网站时，从进入网站到浏览器关闭这段时间内的会话。由此可知，Session实际上是一个特定的时间概念。

使用Session可以在网站的上下文不同页面间传递变量、用户身份认证、程序状态记录等。常见的形式就是配合Cookie使用，实现保存用户登录状态功能。和Cookie一样，session_start() 必须在程序最开始执行，前面不能有任何输出内容，否则会出现警告。PHP的Session默认通过文件的方式实现，即存储在服务器端的Session文件，每个Session一个文件。

Session通过一个称为PHPSESSID的Cookie和服务器联系。Session是通过sessionID判断客户端用户的，即Session文件的文件名。sessionID实际上是在客户端和服务端之间通过HTTP Request 和 HTTP Response传来传去。sessionID按照一定的算法生成，必须包含在 HTTP Request 里面，保证唯一性和随机性，以确保Session的安全。如果没有设置 Session 的生成周期， sessionID存储在内存中，关闭浏览器后该ID自动注销；重新请求该页面，会重新注册一个sessionID。如果客户端没有禁用Cookie，Cookie在启动Session回话的时候扮演的是存储sessionID 和 Session 生存期的角色。Session过期后，PHP会对其进行回收。

假设客户端禁用Cookie，可以通过URL或者隐藏表单传递sessionID；php.ini中把session.use_trans_sid 设成1，那么连接后就会自己加Session的ID。

Session以文件的形式存放在本地硬盘的一个目录中，当比较多时，磁盘读取文件就会比较慢，因此把Session分目录存放。

对于访问量大的站点，用默认的Session存储方式并不适合，较优的方法是用Data Base存取Session。在大流量的网站中，Session入库存在效率不高、占据数据库connection资源等问题。针对这种情况，可以使用Memcached、Redis等Key-Value数据存储方案实现高并发、大流量的Session存储。

**session与cookie的区别：**
1，session 在服务器端，cookie 在客户端（浏览器）
2，session 存在在服务器的一个文件里（默认），不是内存
3，session 的运行依赖 session id，而 session id 是存在 cookie 中的，也就是说，如果 浏览器禁用了 cookie ，同时 session 也会失效（当然也可以在 url 中传递）
4，session 可以放在 文件，数据库，或内存中都可以。
5，用户验证这种场合一般会用 session
因此，维持一个会话的核心就是客户端的唯一标识，即 session id

更为详尽的说法：
1.  由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.典型的场景比如购物车，当你点击下单按钮时，由于HTTP协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个Session是保存在服务端的，有一个唯一标识。在服务端保存Session的方法很多，内存、数据库、文件都有。集群的时候也要考虑Session的转移，在大型的网站，一般会有专门的Session服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如Memcached之类的来放 Session。

2.  思考一下服务端如何识别特定的客户？这个时候Cookie就登场了。每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用 Cookie 来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。有人问，如果客户端的浏览器禁用了 Cookie 怎么办？一般这种情况下，会使用一种叫做URL重写的技术来进行会话跟踪，即每次HTTP交互，URL后面都会被附加上一个诸如 sid=xxxxx 这样的参数，服务端据此来识别用户。

3.  Cookie其实还可以用在一些方便用户的场景下，设想你某次登陆过一个网站，下次登录的时候不想再次输入账号了，怎么办？这个信息可以写到Cookie里面，访问网站的时候，网站页面的脚本可以读取这个信息，就自动帮你把用户名给填了，能够方便一下用户。这也是Cookie名称的由来，给用户的一点甜头。

所以，总结一下：
Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；

Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。


**Cookie与Session问答**



1.  Cookie运行在客户端，Session运行在服务端，对吗？

   A：不完全正确。Cookie是运行在客户端，有客户端进行管理；Session虽然是运行在服务器端，但是sessionID作为一个Cookie是存储在客户端的。

2.  浏览器禁止Cookie，Cookie就不能用了，但Session不会受浏览器影响，对吗？

   A：错。浏览器禁止Cookie，Cookie确实不能用了，Session会受浏览器端的影响。很简单的实验，在登录一个网站后，清空浏览器的Cookie和隐私数据，单机后台的连接，就会因为丢失Cookie而退出。当然，有办法通过URL传递Session。

3.  浏览器关闭后，Cookie和Session都消失了，对吗？

  A：错。存储在内存中额Cookie确实会随着浏览器的关闭而消失，但存储在硬盘上的不会。更顽固的是Flash Cookie，不过现在很多系统优化软件和新版浏览器都已经支持删除Flash Cookie。百度采用了这样的技术记忆用户：Session在浏览器关闭后也不会消失，除非正常退出，代码中使用了显示的unset删除Session。否则Session可能被回收，也有可能永远残留在系统中。

4.  Session 比 Cookie 更安全吗？ 不应该大量使用Cookie吗？

   A：错误。Cookie确实可能存在一些不安全因素，但和JavaScript一样，即使突破前端验证，还有后端保障安全。一切都还要看设计，尤其是涉及提权的时候，特别需要注意。通常情况下，Cookie和Session是绑定的，获得Cookie就相当于获得了Session，客户端把劫持的Cookie原封不动地传给服务器，服务器收到后，原封不动地验证Session，若Session存在，就实现了Cookie和Session的绑定过程。因此，不存在Session比Cookie更安全这种说法。如果说不安全，也是由于代码不安全，错误地把用作身份验证的Cookie作为权限验证来使用。

5.  Session是创建在服务器上的，应该少用Session而多用Cookie，对吗？

   A：错。Cookie可以提高用户体验，但会加大网络之间的数据传输量，应尽量在Cookie中仅保存必要的数据。

6.  如果把别人机器上的Cookie文件复制到我的电脑上（假设使用相同的浏览器），是不是能够登录别人的帐号呢？如何防范？

  A：是的。这属于Cookie劫持的一种做法。要避免这种情况，需要在Cookie中针对IP、UA等加上特殊的校验信息，然后和服务器端进行比对。


7.  在IE浏览器下登录某网站，换成Firefox浏览器是否仍然是未登录状态？使用IE登录了腾讯网站后，为什么使用Firefox能保持登录状态？

   A：不同浏览器使用不同的Cookie管理机制，无法实现公用Cookie。如果使用IE登录腾讯网站，使用Firefox也能登录，这是由于在安装腾讯QQ软件时，你的电脑上同时安装了针对这两个浏览器的插件，可以识别本地已登录QQ号码进而自动登录。本质上，不属于共用Cookie的范畴。
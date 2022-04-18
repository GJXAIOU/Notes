# [Java Web(二) Servlet中response、request乱码问题解决](https://www.cnblogs.com/whgk/p/6412475.html)

三月不减肥，五月徒伤悲，这就是我现在的状态，哈哈~ 健身、博客坚持。

--WZY

一、request 请求参数出现的乱码问题　　

get 请求：

get 请求的参数是在 url 后面提交过来的，也就是在请求行中，

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218092036988-519329509.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218092050675-710906232.png)

MyServlet 是一个普通的 Servlet，浏览器访问它时，使用 get 请求方式提交了一个 name=小明的参数值，在 doGet 中获取该参数值，并且打印到控制台，发现出现乱码

出现乱码的原因：

前提知识：需要了解码表，编码，解码这三个名词的意思。我简单说一下常规的，

码表：是一种规则，用来让我们看得懂的语言转换为电脑能够认识的语言的一种规则，有很多中码表，IS0-8859-1,GBK,UTF-8,UTF-16 等一系列码表，比如 GBK,UTF-8,UTF-16 都可以标识一个汉字，而如果要标识英文，就可以用 IS0-8859-1 等别的码表。

编码：将我们看得懂的语言转换为电脑能够认识的语言。这个过程就是编码的作用

解码：将电脑认识的语言转换为我们能看得懂得语言。这个过程就是解码的作用

[详细请参考这篇博文。](http://blog.csdn.net/u010627840/article/details/50407575 "http://blog.csdn.net/u010627840/article/details/50407575")

这里只能够代表经过一次编码例子，有些程序中，会将一个汉字或者一个字母用不同的码表连续编码几次，那么第一次编码还是上面所说的作用，第二次编码的话，就是将电脑能够认识的语言转换为电脑能够认识的语言(转换规则不同)，那么该解码过程，就必须要经过两次解码，也就是编码的逆过程，下面这个例子就很好的说明了这个问题。

浏览器使用的是 UTF-8 码表，通过 http 协议传输，http 协议只支持 IS0-8859-1，到了服务器，默认也是使用的是 IS0-8859-1 的码表，看图

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218102853191-1043718309.png)

也就是三个过程，经历了两次编码，所以就需要进行两次解码，

1、浏览器将"小明"使用 UTF-8 码表进行编码(因为小明这个是汉字，所以使用能标识中文的码表，这也是我们可以在浏览器上可以手动设置的，如果使用了不能标识中文的码表，那么就将会出现乱码，因为码表中找不到中文对应的计算机符号，就可能会用？？等其他符号表示)，编码后得到的为 1234 ，将其通过 http 协议传输。

2、在 http 协议传输，只能用 ISO-8859-1 码表中所代表的符号，所以会将我们原先的 1234 再次进行一次编码，这次使用的是 ISO-8859-1，得到的为 ???? ，然后传输到服务器

3、服务器获取到该数据是经过了两次编码后得到的数据，所以必须跟原先编码的过程逆过来解码，先是 UTF-8 编码，然后在 ISO-8859-1 编码，那么解码的过程，就必须是先 ISO-8859-1 解码，然后在用 UTF-8 解码，这样就能够得到正确的数据。????.getBytes("ISO-8859-1");//第一次解码，转换为电脑能够识别的语言， new String(1234,"UTF-8");//第二次解码，转换为我们认识的语言

解决代码

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218110909457-1939137580.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218110919004-1402636202.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218111028550-1038940540.png)

Post 请求：

post 请求方式的参数是在请求体中，相对于 get 请求简单很多，没有经过 http 协议这一步的编码过程，所以只需要在服务器端，设置服务器解码的码表跟浏览器编码的码表是一样的就行了，在这里浏览器使用的是 UTF-8 码表编码，那么服务器端就设置解码所用码表也为 UTF-8 就 OK 了

设置服务器端使用 UTF-8 码表解码

request.setCharacterEncoding("UTF-8");　　//命令 Tomcat 使用 UTF-8 码表解码，而不用默认的 ISO-8859-1 了。

所以在很多时候，在 doPost 方法的第一句，就是这句代码，防止获取请求参数时乱码。

总结请求参数乱码问题

get 请求和 post 请求方式的中文乱码问题处理方式不同

get:请求参数在请求行中，涉及了 http 协议，手动解决乱码问题，知道出现乱码的根本原因，对症下药，其原理就是进行两次编码，两次解码的过程

new String(xxx.getBytes("ISO-8859-1"),"UTF-8");

post：请求参数在请求体中，使用 servlet API 解决乱码问题，其原理就是一次编码一次解码，命令 tomcat 使用特定的码表解码。

request.setCharaterEncoding("UTF-8");

二、response 响应回浏览器出现的中文乱码。　　　　　　　　　　

首先介绍一下，response 对象是如何向浏览器发送数据的。两种方法，一种 getOutputStream，一种 getWrite。

ServletOutputStream getOutputStream();　　//获取输出字节流。提供 write() 和 print() 两个输出方法

PrintWriter getWrite();　　//获取输出字符流　　提供 write() 和 print()两个输出方法

print()方法底层都是使用 write()方法的，相当于 print()方法就是将 write()方法进行了封装，使开发者更方便快捷的使用，想输出什么，就直接选择合适的 print()方法，而不用考虑如何转换字节。

1、ServeltOutputStream getOutputStream();

不能直接输出中文，直接输出中文会报异常，

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218141536332-1657727152.png)　　　　

报异常的源代码

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218145046988-1340244663.png)

解决：

resp.getoutputStream().write("哈哈哈，我要输出到浏览器".getBytes("UTF-8"));

将要输出的汉字先用 UTF-8 进行编码，而不用让 tomcat 来进行编码，这样如果浏览器用的是 UTF-8 码表进行解码的话，那么就会正确输出，如果浏览器用的不是 UTF-8，那么还是会出现乱码，所以说这个关键要看浏览器用的什么码表，这个就不太好，这里还要注意一点，就是使用的是 write(byte)方法，因为 print()方法没有输出 byte 类型的方法。

2、PrintWriter getWrite();

直接输出中文，不会报异常，但是肯定会报异常，因为用 ISO-8859-1 的码表不能标识中文，一开始就是错的，怎么解码编码读没用了

有三种方法来让其正确输出中文

1、使用 Servlet API  response.setCharacterEncoding()

response.setCharacterEncoding("UTF-8");　　//让 tomcat 将我们要响应到浏览器的中文用 UTF-8 进行编码，而不使用默认的 ISO-8859-1 了，这个还是要取决于浏览器是不是用的 UTF-8 的码表，跟上面的一样有缺陷

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170218150934550-1632885660.png)

2、通知 tomcat 和浏览器都使用同一张码表

response.setHeader("content-type","text/html;charset=uft-8");　　//手动设置响应内容，通知 tomcat 和浏览器使用 utf-8 来进行编码和解码。

charset=uft-8 就相当于 response.setCharacterEncoding("UTF-8");//通知 tomcat 使用 utf-8 进行编码

response.setHeader("content-type","text/html;charset=uft-8");//合起来，就是既通知 tomcat 用 utf-8 编码，又通知浏览器用 UTF-8 进行解码。

response.setContentType("text/html;charset=uft-8");　　//使用 Servlet API 来通知 tomcaat 和强制浏览器使用 UTF-8 来进行编码解码，这个的底层代码就是上一行的代码，进行了简单的封装而已。　　　　　　　　　　　　　　　　　　　　　　　　　　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170219131715379-1609567969.png)

3、通知 tomcat，在使用 html<meta>通知浏览器 (html 源码)，注意：<meta>建议浏览器应该使用编码，不能强制要求

进行两步

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170219131751316-975932797.png)　　　　

所以 response 在响应时，只要通知 tomcat 和浏览器使用同一张码表，一般使用第二种方法，那么就可以解决响应的乱码问题了

三、总结

在上面讲解的时候总是看起来很繁琐，其实知道了其中的原理，很简单，现在来总结一下，

请求乱码

get 请求：

经过了两次编码，所以就要两次解码

第一次解码：xxx.getBytes("ISO-8859-1");得到 yyy

第二次解码：new String(yyy,"utf-8");

连续写：new String(xxx.getBytes("ISO-8859-1"),"UTF-8");

post 请求：

只经过一次编码，所以也就只要一次解码,使用 Servlet API　request.setCharacterEncoding();

request.setCharacterEncoding("UTF-8");　　//不一定解决，取决于浏览器是用什么码表来编码，浏览器用 UTF-8，那么这里就写 UTF-8。

响应乱码

getOutputStream();

使用该字节输出流，不能直接输出中文，会出异常，要想输出中文，解决方法如下

解决：getOutputStream().write(xxx.getBytes("UTF-8"));　　//手动将中文用 UTF-8 码表编码，变成字节传输，变成字节后，就不会报异常，并且 tomcat 也不会在编码，因为已经编码过了，所以到浏览器后，如果浏览器使用的是 UTF-8 码表解码，那么就不会出现中文乱码，反之则出现中文乱码，所以这个方法，不能完全保证中文不乱码

getWrite();

使用字符输出流，能直接输出中文，不会出异常，但是会出现乱码。能用三种方法解决，一直使用第二种方法

解决：通知 tomcat 和浏览器使用同一张码表。

response.setContentType("text/html;charset=utf-8");　　//通知浏览器使用 UTF-8 解码 

通知 tomcat 和浏览器使用 UTF-8 编码和解码。这个方法的底层原理是这句话：response.setHeader("contentType","text/html;charset=utf-8"); 

注意：getOutputStream()和 getWrite() 这两个方法不能够同时使用，一次只能使用一个，否则报异常
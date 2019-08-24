# JavaDay25网络

@toc

网络编程：网络编程是用来解决计算机和计算机之间的通讯问题；
网页编程：基于 HDML 页面的基础上进行数据的交互；

## 一、获取 IP 地址

- IP 类获取 IP 对象的方式
使用类：InetAddress
常用方法：

方法名| 含义
---|---- 
getLocalHost();            | 获取本机的主机名和 IP 地址
getByName(String address); | 可以通过计算机名或者 IP 地址，得到对应的 IP 对象
getHostAddress();          | 返回一个 IP 地址的字符串表示方式
getHostName();             |返回主机名

以上方法使用示例：
```java
package Demo;

import java.net.InetAddress;
import java.net.UnknownHostException;

public class Puzzle4{
    public static void main(String[] args) throws UnknownHostException {
        //获取本机的IP地址对象
        InetAddress address = InetAddress.getLocalHost();
        System.out.println(address);
        
        //通过主机名获取IP地址
        InetAddress address1 = InetAddress.getByName("GJXAIOU");
        System.out.println(address1);

        InetAddress address2 = InetAddress.getByName("192.168.1.1");
        System.out.println(address2);

        System.out.println(address.getHostAddress());
        System.out.println(address.getHostName());

        InetAddress[] addresses = InetAddress.getAllByName("www.taobao.com");
        for (InetAddress inetAddress : addresses) {
            System.out.println(inetAddress);
        }
    }
}
```
程序运行结果：
```java
GJXAIOU/192.168.137.1
GJXAIOU/192.168.137.1
/192.168.1.1
192.168.137.1
GJXAIOU
www.taobao.com/61.155.222.102
www.taobao.com/61.155.222.103
```


## 二、UDP 中 socket

在网络编程中所有的数据传递都是依赖于 Socket 来完成的，要求进行通信的两台计算机都要安装有 Socket；
不同的传输协议有不同的 Socket；

- UDP 下面的 socket：
1.把数据封装成一个数据包，面向无连接；
2.UDP 数据报大小限制在 64kB 以内；
3.无连接传输速度快但是不可靠；
4.UDP 不区分服务端和客户端，只有发送端和接收端；

- UDP 下面的 Socket 使用：
  - DatagramSocket(); //获取 UDP 的 Socket
  - DatagramPacket(byte[] buf, int length, InetAddress address, int port); //UDP 传输的数据包
    - buf:要打包的数据，要求数据类型是 byte 类型数组；
    - length:要打包数据的字节个数；
    - address:发送目标地址的 IP 对象；
    - port：端口号


端口号：是系统中每一个执行的程序唯一的编号，接收到的数据根据不同的端口号将数据发送给对应的程序；
序号从 0-65535，其中 0-1023 是为系统服务的端口号，已经绑定；

- 发送端流程：
  - 1.建立 UDP 服务，打开 UDP 协议下的 Socket；
  - 2.准备数据；
  - 3.将数据打包；
  - 4.通过 Socket 发送数据；
  - 5.Socket 关闭资源

代码示例：
- 发送端代码：
```java
package Demo;
import java.io.IOException;
import java.net.*;

public class Puzzle4{
    public static void main(String[] args) throws IOException {
        //1.建立UDP服务，打开UDP协议下的Socket，发送端Socket创建不需要任何参数
        DatagramSocket socket = new DatagramSocket();

        //2.准备数据
        String data = "hello";

        //3.数据打包
        DatagramPacket datagramPacket = new DatagramPacket (data.getBytes(), 
        data.getBytes().length, 
        InetAddress.getLocalHost(), 
        8848);

        //4.通过Socket发送数据
        socket.send(datagramPacket);

        //5.关闭资源
        socket.close();

    }
}
```


- 接收端流程：
  1.建立UDP服务，监听端口
  2.准备空数据包，用于接收数据
  3.调用UDP服务接收数据
  4.获取数据
  5.释放资源
 
接收端代码：
```java
package Demo;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;


public class Puzzle4Receive {
    public static void main(String[] args) throws IOException {
        //1.建立UDP服务，监听端口
        DatagramSocket socket = new DatagramSocket(8848);

        //2.准备空数据包，接收数据
        byte[] buf = new byte[1024];
        //利用byte数据创建空数据包
        DatagramPacket datagramPacket = new DatagramPacket(buf, buf.length);

        //3.调用UDP服务，使用Socket接收数据
        socket.receive(datagramPacket);

        //4.从数据包中获取socket接收到的数据
        //所有的数据都会被保存在byte数组中，通过调用UDP数据包的getlength方法，获取到接收到的数据字节长度
        System.out.println(new String(buf, 0, datagramPacket.getLength()));

        //5.关闭资源
        socket.close();
    }
}
```


### （一）搭建局域网聊天工具
chatSender.java 文件
```java
package Chat;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

/**
 * @author GJXAIOU
 * @create 2019-07-24-16:02
 */

//这里使用多线程进行操作
public class chatSender extends Thread {
    @Override
    public void run() {
        try {
            DatagramSocket socket = new DatagramSocket();

            //system.in是键盘
            //这里是将一个输入字节流对象做成一个输入字符流对象，提供给缓冲字符流作为读写的能力
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

            String line = null;
            DatagramPacket packet = null;

            while ((line = bufferedReader.readLine()) != null){
                packet = new DatagramPacket(line.getBytes(), line.getBytes().length, 
                InetAddress.getByName("218.2.216.255"),8888); //里面的IP地址为广播地址

                socket.send(packet);
            }

            socket.close();

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


```

chatReceive 文件
```java
package Chat;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

/**
 * @author GJXAIOU
 * @create 2019-07-24-16:02
 */
public class chatReceive extends Thread{
    @Override
    public void run() {
        try {
            DatagramSocket socket = new DatagramSocket(8888);

            byte[] buf = new byte[1024];
            DatagramPacket packet = new DatagramPacket(buf, buf.length);

            boolean flag = true;
            while (flag){
                socket.receive(packet);
                System.out.println(packet.getAddress().getHostAddress() + ":" + new String(buf, 0,packet.getLength()));
            }

            socket.close();

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

chatMain.java 文件
```java
package Chat;

/**
 * @author GJXAIOU
 * @create 2019-07-24-16:02
 */
public class chatMain {
    public static void main(String[] args) {
        chatSender chatSender = new chatSender();
        chatReceive chatReceive = new chatReceive();
        chatSender.start();
        chatReceive.start();

    }
}

```

### （二）模拟飞秋发送数据
因此这里只有发送端程序
```java
package Chat;

import java.io.IOException;
import java.net.*;

/**
 * @author GJXAIOU
 * @create 2019-07-24-16:45
 */
/*FeiQ的数据格式：
  version:time:sender:ip:flag:content
  版本号   时间  发送人  IP地址
 */
public class FeiQ {
    public static void main(String[] args) throws IOException {
        DatagramSocket socket = new DatagramSocket();
        String data = getData("hello");

        DatagramPacket packet = new DatagramPacket(data.getBytes(), data.getBytes().length, InetAddress.getByName("192.168.1.1"), 2425);
        socket.send(packet);
        socket.close();
    }


    private static String getData(String content){
        StringBuilder data = new StringBuilder();

        data.append("1.0:");
        data.append(System.currentTimeMillis() + ":");
        data.append("匿名君");
        data.append("10.1.1.1");
        data.append("32:");
        data.append(content);

        return data.toString();
    }

}


```



## 三、TCP 中 Socket
- TCP 特征：
1.TCP 是完全基于 IO 流进行数据传输的，面向连接；
2.TCP 进行数据传递时候没有显示数据报的大；
3.TCP 面向连接，必须通过三次握手之后才能保证数据的传输通道是完整的；
4.TCP 面向连接，速度相对较慢；
5.TCP 是区分客户端和服务器；

- TCP 协议下的 Socket
Socket(服务器 IP 地址对象， 服务器软件对应的端口号); 创建 TCP 协议下的端口号，并且。。。。
ServerSocket(); 服务器的 Socket，开始服务器服务，准备捕获 Socket

TCP 客户端流程：
1.建立 TCP 客户端连接，申请连接服务器，需要服务器 IP 地址对象和对应的程序端口号；
2.获取对应的流对象；
3.写入或者读取数据；
4.关闭资源；


发送端：
```java
package IP;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.Socket;


/**
 * @author GJXAIOU
 * @create 2019-07-24-17:11
 */
public class IpSocket {
    public static void main(String[] args) throws IOException {
        //1.建立客户端Socket，申请连接服务器，需要服务器的IP地址和对应程序的端口号
        Socket socket = new Socket(InetAddress.getLocalHost(), 8000);

        //2.发送数据给服务器，需要获取Socket的输出流对象
        OutputStream os = socket.getOutputStream();

        //使用OutputStream方法发送数据到服务器，也就是输出数据
        os.write("你好，服务器".getBytes());

        //3.获取Socket的InputStream
        InputStream inputStream = socket.getInputStream();
        byte[] buf = new byte[1024];
        int length = inputStream.read(buf);

        System.out.println("服务器说:" + new String(buf, 0, length));

        socket.close();
    }
}

```

服务器接收端：
```java
package IP;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @author GJXAIOU
 * @create 2019-07-24-17:23
 */
public class IpService {
    public static void main(String[] args) throws IOException {
        //1.使用ServiceSocket开始TCP服务器，监听指定端口，准备捕获从客户端申请Socket连接
        ServerSocket serverSocket = new ServerSocket(8000);

        //2.接受客户端连接，得到客户端Socket对象
        Socket accept = serverSocket.accept();

        //3.获取从客户端得到的Socket对象的输入流
        InputStream inputStream = accept.getInputStream();

        byte[] buf = new byte[1024];
        int length = inputStream.read(buf);
        System.out.println("客户端说：" + new String(buf, 0, length));

        //4.获取Socket的输出流对象，给客户端发送数据
        OutputStream outputStream = accept.getOutputStream();

        outputStream.write("你好，客户端：".getBytes());

        //关闭ServerSocket，即关闭TCP协议下的服务器程序
        serverSocket.close();
    }

}

```




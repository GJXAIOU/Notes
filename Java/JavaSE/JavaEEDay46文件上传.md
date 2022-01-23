# JavaEEDay46文件上传

## 一、基本知识

### HTTP 请求组成
- 请求行：由请求方法字段、URL 字段、HTTP 协议版本字段组成，中间使用空格分隔；例如：`GET /index.html HTTP/1.1`
- 请求头部：由关键字/键值对组成，每行一对，关键字和值使用英文`:`隔开；请求头部作用是通知服务器有关客户端请求的信息；例如：Host，请求的主机名；cookie，客户端的缓存；
- 请求数据：请求数据不在 get 方法中使用，在 POST 方法中使用，POST 方法适用于需要客户填写表单的场合，与请求数据相关的最常使用的请求头是 `Content-Type` 和 `Content-Length`。

### GET 和 POST 区别
|操作方式|数据位置|明文密文|数据安全|长度限制|应用场景|
|----|----|----|---|---|---
|GET|HTTP 包头|明文|不安全|长度较小|查询数据
|POST|HTTP 正文|可明可密    |安全|支持较大数据传输|修改数据


## 二、上传文件

使用 Servlet 却一般不使用 JSP 处理文件上传的原因：
因为 JSP 一般在 MVC 模式中是一个 view，偏向于视图展示。处理数据比如处理上传的文件进行保存，主要功能在于后台获取文件和文件保存，在 mvc 中是一个 control，或者可以简单的理解为不需要视图展示；jsp 可以做文件上传处理，但是一般不这样使用。

### 上传依赖的 jar 包

Commons-fileupload.jar 和 commons-io.jar

### fileupload 包中重要的类和方法

#### DiskFileltemFactory 类，用于配置上传参数

- setSizeThreshold 设置缓存的大小，当上传文件的容量超过该缓存时，再放到临时存储室
- setRepository() 设置临时存储路径，存放 tem 临时文件，可以和最终存储文件相同。
如果没上面两个方法设置的话,上传大的文件会占用很多内存

可以存放到项目根目录下，根路径的获取
`request.getSession().getServletContext(). getRealPath(“/upload“)`
返回一个字符串,包含一个给定虚拟路径的真实路径

#### ServletFileUpload 类，处理文件和文件上传

- 初始化:`new ServletFileUpload( factory);` factory就是 DiskFileltemFactory类方法:
- `isMultipartContent( request)`判断请求form是否是文件类型。

- `setFilesizemax()`设置最大文件上传值(即单个文件大小的上限)
- `setSizemax()`设置最大请求值(包含文件和表单数据)
- `parseRequest( request);`解析请求中的文件返回List< Filelten>//获取数据


- Fillter从表单数据中获取到的文件。
  - `ServletFileUpload.parseRequest()`返回的是个泛型为 Filleter的集合,因为可以多文件上传。
  - 方法  
  - `getName()`获取到上传文件的名字
  - `isFormField()`是否是普通表单
  - `item.write(File)`把文件写入
  //一下两个方法用于tem是普通inpu时
  - getFieldName
  - getString()


### 上传的表单指定属性
form 中添加 `enctype=" multipart/form-data"` 设置表单的 MME 编码。

enctype 共有 3 个值可选
- `application/x-www-form-urlencode`(默认值)，作用是设置表单传输的编码。
- `multipart/form-dat`，主要就是我们上传二进制文件的非文本的内容,比如图片或是 mp3 等。
- `text/pain`，纯文本传输的意思,在发邮件的时候要设置这种编码类型(不常用)

### 代码示例

UploadServlet.java 用来处理文件上传并且保存文件
upload.html 是上传页面
error.html 是上传失败跳转到的页面
success.html 是上传成功跳转到的页面

UploadServlet.java
```java
package upload;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;
import java.util.List;
import java.util.UUID;

/**
 * @author GJXAIOU
 * @create 2019-08-13-19:26
 */
@WebServlet("upload")
public class UploadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        // 添加：判断前端页面中的form表单中是否有属性：enctype= " XXXX";因为没有会报异常
        if (ServletFileUpload.isMultipartContent(req)) {
            // 带有该属性
            // 1.对请求进行解析，读取文件

            // 用于设置缓冲区大小（单位为字节），和临时存储目录
            DiskFileItemFactory diskFileItemFactory = new DiskFileItemFactory();
            // 设置缓冲区大小
            diskFileItemFactory.setSizeThreshold(2 * 1024);

            // 设置临时目录，一般将临时目录和最终保存文件目录放在一起
            File file = new File("E:");
            if (!file.exists()){
                file.mkdir();
            }
            diskFileItemFactory.setRepository(file);

            // 这里读取文件不再是：req.getParamter，而是使用fileupload(),因为getparamter的返回值为string
            ServletFileUpload servletFileUpload = new ServletFileUpload(diskFileItemFactory);

            // 设置上传文件最大大小
            // 设置上传的单文件最大值
            servletFileUpload.setFileSizeMax(1024*1024*5);
            // 设置整个表单能够上传的文件最大值（可能有多个文件）,这里以10兆为例
            servletFileUpload.setSizeMax(1024 *1024 *10);

            // 参数为请求，作用是解析请求，并且将请求的数据封装成一个集合返回（因为可能有多文件上传），泛型是FileItem
            try {
                List<FileItem> fileItems = servletFileUpload.parseRequest(req);

                // 遍历集合，获取到数据
                for (FileItem item : fileItems) {

                    // 当表单中既有文本又有文件时候，要分开处理
                    if (item.isFormField()) {
                        // 表示当前数据是一个普通文本

                        // 获取文本内容即可(同时防止中文乱码)
                        String string = item.getString("UTF-8");
                        System.out.println(string);
                    }else {
                        // 当前Item是一个非文本的文件

                        // 2.把读取的文件对象保存到服务器本地磁盘中
                        // 在本地文件夹转给你新建一个文件，以上传文件名保存:item.getName()
                        // 为了保证存储的文件名唯一，防止被覆盖，使用UUID，这是用于生成随机的唯一字符的api
                        String newFileName = UUID.randomUUID().toString().replace("-", "");
                        // 还要获取原来文件的后缀名（取.之后的字符）
                        String name = item.getName();
                        String substring = name.substring(name.lastIndexOf("."));

                        // 为了让其他用户可以访问（下载），将文件保存在web项目资源目录下面：
                        // 这里获取到的不是代码的工作区间路径，是项目部署到Tomcat之后在Tomcat里的路径
                        String realPath = req.getServletContext().getRealPath("");

                        File file1 = new File(realPath+ newFileName + substring);
                        item.write(file1);
                    }

                }
                // 跳转到成功页面
                resp.sendRedirect("success.html");

            } catch (FileUploadException e) {
                // 跳转到错误页面
                resp.sendRedirect("error.html");
                e.printStackTrace();
            } catch (Exception e) {
                // 跳转到错误页面
                resp.sendRedirect("error.html");
                e.printStackTrace();
            }
        } else {
            // 跳转到错误页面
            resp.sendRedirect("error.html");
        }
    }
}


```

Upload.html
```html
<body>
    <!--    action路径是一个处理文件的Servlet-->
    <!--method这里为POST，因为可能上传的文件较大，而POST没有文件大小限制-->
<!--    上传文件表单中，要加上enctype属性，具体见图片-->
    <form action="upload" method="post" enctype="multipart/form-data">

        <input type="text" name="test">
        <!--这里点击input就打开文件系统-->
        选择一个文件：<input type="file" name="file"> <br>
        提交：<input type="submit" value="提交">

    </form>
</body>
```

其他如：success.html 和 error.html

## 三、下载文件

==WEB-INF 下面的资源只能通过转发进行访问==

### 响应的组成部分

HTTP 响应与 HTTP 请求相似，HTTP 响应也由 3 个部分构成
- 1)状态行
请求版本响应码(404,200,500)

- 2)响应头
例如：
```http
Response Headers
Content-Length: 463
Content-Type: text/html; charset=UTF
Date: Tue, 22 Aug 2017 06: 58: 37 GMT
Server: Apache-Coyote/1.1
```
- 3)响应正文

### 响应头
响应头中是多个键值对组成，我们在实现一个文件下载的时候需要设置响应头中以下的参数
- Content-Disposition:设置响应的HTTP头信息中的 Content- Disposition参数的值就是当用户想把请求所得的内容存为一个文件的时候提供一个默认的文件名.设置文件下载的时候文件的名字)

- ContentType:设置发送到浏览器的MME类型，也就是告诉浏览器返回给你的数据类型

**ContentType 文件类型及其对应设置**
文件类型 |类型设置
|---|---
Word |application/msword
Execl  |application/vnd.ms-excel
PPT |application/vnd.ms-powerpoint
图片  |image/gif, image/bmp, image/jpeg
文本文件  |text/plain
html 网页  |text/html
可执行文件 | application/octet-stream


### 文件下载关键的类
- response,负责向客户端进行相应,需要设置contenttype和Content-Disposition
- ServletOutputStream，通过 response. getOutputStream获取,负责向客户端进行响应二进制类型数据
- lOUtils，Commons-io包提供的文件操作工具类,负责把文件流转成byte[]传递 给ServletOutputStream 进行响应


DownloadServlet.java
```java
package download;

import org.apache.commons.io.IOUtils;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URLEncoder;

/**
 * @author GJXAIOU
 * @create 2019-08-13-21:26
 */
@WebServlet("/download")
public class DownloadServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {



        // 1.首先将服务器上的某一个文件转换为File对象
        File file = new File("E:\\File\\HHIT\\照片\\照片\\壁纸/1.jpg");

        // 文件下载需要对响应头进行设置
        // 设置返回给客户端的数据类型,这是设置是可执行文件
        resp.setContentType("application/octet-stream");

        // 告诉浏览器需要下载的文件的名称
        // 编码：将中文转成Unicode字节码
        // 解码：将Unicode码转换成中文 ：URLDecoder.encode(file.getName(), "UTF-8"),这里浏览器会自动解码
        resp.addHeader("Content-Dispositon", "attachment;filename = " + URLEncoder.encode(file.getName(), "UTF-8"));

        // 用于向客户端发送文件；向客户端输出文本或者HTML源码使用：resp.getWriter().print()
        ServletOutputStream outputStream = resp.getOutputStream();

        // 将文件转换为流
        InputStream inputStream = new FileInputStream(file);
        outputStream.write(IOUtils.toByteArray(inputStream));

        inputStream.close();
        outputStream.close();


        // 2.把File通过响应传递给客户端
    }
}

```

download.html
```html
<body>
<!--点击下载链接，开始调用浏览器的下载器下载路径对应的文件-->

<!--href中直接使用链接会打开该文件，需要将a标签的href指向一个Servlet，
    最重要的是在Servlet中设置头部，头部中有contenttype
-->
<a href="download">下载</a>

</body>
```

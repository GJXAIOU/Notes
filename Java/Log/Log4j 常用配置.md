## Log4J

### （一）基本配置

- 导入 log4j-xxx.jar
- 在 src 下新建 log4j.properties(路径和名称都不允许改变)

```java
// log4j 五大输出级别：fatal > error > warn > info > debug 只有大于配置的级别的日志才输出
// 遇到什么级别才输出以及输出位置：调试信息输出， 输出到控制台，输出到 log 文件
log4j.rootCategory=DEBUG, CONSOLE, LOGFILE
// 向控制台输出相关的配置
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout 
// 控制台日志输出格式    
log4j.appender.CONSOLE.layout.ConversionPattern=%C %d{YYYY-MM-dd hh:mm:ss}%m  %n

// 向日志文件输出的相关的配置
log4j.appender.LOGFILE=org.apache.log4j.FileAppender 
// 日志文件位置和名称    
log4j.appender.LOGFILE.File=E:/my.log 
// true 表示往后追加，false 表示清空再追加    
log4j.appender.LOGFILE.Append=true log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout 
// 日志文件输出格式    
log4j.appender.LOGFILE.layout.ConversionPattern=%C	%m %L %n
```

### （二） 基本使用

- 使用方法一：

    ```java
    // 省略 Test 类的内容
    Logger  logger = Logger.getLogger(Test.class);
    // 这里以 debug 为例，具体的方法看上面输出级别调用不同的方法
    logger.debug("这是调试信息"); 
    ```

- 使用方法二：
    在 catch 中 ： `logger.error(e.getMessage());` 即可，注意在 log4j 配置文件中加上相关的表达式；

### （三）表达式配置

pattern 中常用几个表达式（区分大小写，多个共同使用时中间添加空格）

- `%C`：输出包名+类名；
- `%d{YYYY-MM-dd  HH:mm:ss}`：时间；
- `%L`：行号
- `%m`：信息
- `%n`：换行
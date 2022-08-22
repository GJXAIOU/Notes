## Jar 包和 war 包区别

### 一、概念

- jar包：**JAR 包是类的归档文件**，JAR 文件格式以流行的 ZIP 文件格式为基础。与 ZIP 文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。
- war包:war包是 JavaWeb 程序打的包，war 包里面包括写的代码编译成的 class 文件，依赖的包，配置文件，所有的网站页面，包括html，jsp等等。一个war包可以理解为是一个web项目，里面是项目的所有东西。

### 二、目录结构

jar包里的com里放的就是class文件，配置文件，但是没有静态资源的文件,大多数 JAR 文件包含一个 META-INF 目录，它用于存储包和扩展的配置数据，如安全性和版本信息。
而war包里的WEB-INF里放的class文件和配置文件，META-INF和jar包作用一样，war包里还包含静态资源的文件

### 三、总结起来就是有两点不同：

war包和项目的文件结构保持一致，jar包则不一样。
jar包里没有静态资源的文件（index.jsp）

### 四、部署项目的区别

部署普通的spring项目用war包就可以，部署springboot项目用jar包就可以，因为springboot内置tomcat。



# **jar包的一些事儿**

实际开发中，maven等项目管理工具为我们自动地管理jar包以及相关的依赖，让jar包的调用看起来如黑盒一般"密不透风"。今天，让我们借这个难得的机会来解开这一黑盒，去探索其中的奥秘，正因为这样，咱们接下来主要还是以原生的指令为主，只有通过对原生的理解才能在后续实际工作中贪快图方便使用idea等IDE直接打包jar时不会感到有很多"奇怪"的知识点。希望大家认真学习，不要因为是枯燥的知识而犯困。

文章主要分为以下六大模块。

1. 什么是jar包
2. 为什么要打jar包
3. jar包和war包的区别
4. 如何打jar包
5. jar包的依赖如何解决
6. 如何调用jar包

## [什么是jar包](https://coding.imooc.com/class/303.html?mc_marking=08b2d5f5d99349a1b253ed98a754f110&mc_channel=shouji)

这里要啰嗦一遍概念，jar包就是 Java Archive File，顾名思义，它的应用是与 Java 息息相关的，是 Java 的一种文档格式，是一种与平台无关的文件格式，可将多个文件合成一个文件。jar 包与 zip 包非常相似——准确地说，它就是 zip 包，所以叫它文件包。jar 与 zip 唯一的区别就是在 jar 文件的内容中，包含了一个 META-INF/MANIFEST.MF 文件，该文件是在生成 jar 文件的时候自动创建的，作为jar里面的"详情单"，包含了该Jar包的版本、创建人和类搜索路径Class-Path等信息，当然如果是可执行Jar包，会包含Main-Class属性，表明Main方法入口，尤其是较为重要的Class-Path和Main-Class，咱们一会在后续的内容里面会进行详细地讲解。

此外，值得注意的是，因为jar包主要是对class文件进行打包，而java编译生成的class文件是平台无关的，这就意味着jar包是跨平台的，所以不必关心涉及具体平台的问题。说到jar里面的文件，咱们来看看最普通的一个带有静态页面的springboot项目jar里面的内容，就会发现解压出来的jar并不简单，为了贴近实际咱们未做任何删减，可以看到有很多东西
只需要运行如下指令，就能看到jar里面的内容（调用jar指令的前提是已经配置了jdk的环境变量）

```shell
jar -tf springbootdemo-0.0.1-SNAPSHOT.jar
```

其中-tf 后接的jar就是我们要查看的jar

```xml
META-INF/
META-INF/MANIFEST.MF
org/
org/springframework/
org/springframework/boot/
org/springframework/boot/loader/
org/springframework/boot/loader/util/
org/springframework/boot/loader/util/SystemPropertyUtils.class
org/springframework/boot/loader/archive/
org/springframework/boot/loader/archive/ExplodedArchive$FileEntryIterator.class
org/springframework/boot/loader/PropertiesLauncher$ArchiveEntryFilter.class
org/springframework/boot/loader/LaunchedURLClassLoader$UseFastConnectionExceptionsEnumeration.class
org/springframework/boot/loader/PropertiesLauncher$PrefixMatchingArchiveFilter.class
org/springframework/boot/loader/archive/Archive.class
org/springframework/boot/loader/data/
org/springframework/boot/loader/data/RandomAccessDataFile.class
org/springframework/boot/loader/WarLauncher.class
org/springframework/boot/loader/archive/ExplodedArchive.class
org/springframework/boot/loader/archive/JarFileArchive.class
org/springframework/boot/loader/Launcher.class
org/springframework/boot/loader/archive/ExplodedArchive$FileEntry.class
org/springframework/boot/loader/archive/ExplodedArchive$1.class
org/springframework/boot/loader/archive/JarFileArchive$JarFileEntry.class
org/springframework/boot/loader/jar/
org/springframework/boot/loader/jar/JarFile$1.class
org/springframework/boot/loader/jar/JarFile.class
org/springframework/boot/loader/archive/ExplodedArchive$FileEntryIterator$EntryComparator.class
org/springframework/boot/loader/jar/JarEntry.class
org/springframework/boot/loader/jar/CentralDirectoryEndRecord.class
org/springframework/boot/loader/jar/ZipInflaterInputStream.class
org/springframework/boot/loader/archive/Archive$Entry.class
org/springframework/boot/loader/PropertiesLauncher$1.class
org/springframework/boot/loader/LaunchedURLClassLoader.class
org/springframework/boot/loader/PropertiesLauncher.class
org/springframework/boot/loader/jar/CentralDirectoryVisitor.class
org/springframework/boot/loader/jar/Handler.class
org/springframework/boot/loader/jar/JarURLConnection.class
org/springframework/boot/loader/jar/JarURLConnection$JarEntryName.class
org/springframework/boot/loader/jar/CentralDirectoryParser.class
org/springframework/boot/loader/archive/Archive$EntryFilter.class
org/springframework/boot/loader/ExecutableArchiveLauncher.class
org/springframework/boot/loader/data/RandomAccessDataFile$1.class
org/springframework/boot/loader/data/RandomAccessDataFile$FileAccess.class
org/springframework/boot/loader/jar/CentralDirectoryFileHeader.class
org/springframework/boot/loader/archive/JarFileArchive$EntryIterator.class
org/springframework/boot/loader/JarLauncher.class
org/springframework/boot/loader/data/RandomAccessDataFile$DataInputStream.class
org/springframework/boot/loader/data/RandomAccessData.class
org/springframework/boot/loader/MainMethodRunner.class
org/springframework/boot/loader/jar/Bytes.class
org/springframework/boot/loader/jar/StringSequence.class
org/springframework/boot/loader/jar/JarEntryFilter.class
org/springframework/boot/loader/jar/JarFileEntries.class
org/springframework/boot/loader/jar/FileHeader.class
org/springframework/boot/loader/jar/JarURLConnection$1.class
org/springframework/boot/loader/jar/JarFile$JarFileType.class
org/springframework/boot/loader/jar/JarFile$2.class
org/springframework/boot/loader/jar/JarFileEntries$EntryIterator.class
org/springframework/boot/loader/jar/AsciiBytes.class
org/springframework/boot/loader/jar/JarFileEntries$1.class
BOOT-INF/
BOOT-INF/classes/
BOOT-INF/classes/static/
BOOT-INF/classes/static/css/
BOOT-INF/classes/static/js/
BOOT-INF/classes/static/img/
BOOT-INF/classes/static/pages/
BOOT-INF/classes/com/
BOOT-INF/classes/com/imooc/
BOOT-INF/classes/com/imooc/springbootdemo/
BOOT-INF/classes/com/imooc/springbootdemo/web/
META-INF/maven/
META-INF/maven/com.imooc/
META-INF/maven/com.imooc/springbootdemo/
BOOT-INF/classes/static/js/index.js
BOOT-INF/classes/com/imooc/springbootdemo/web/DemoController.class
BOOT-INF/classes/com/imooc/springbootdemo/SpringbootdemoApplication.class
BOOT-INF/classes/application.properties
META-INF/maven/com.imooc/springbootdemo/pom.xml
BOOT-INF/classes/static/css/index.css
META-INF/maven/com.imooc/springbootdemo/pom.properties
BOOT-INF/classes/static/pages/index.html
BOOT-INF/classes/static/img/cert.png
BOOT-INF/lib/
BOOT-INF/lib/spring-boot-starter-web-2.1.8.RELEASE.jar
BOOT-INF/lib/spring-boot-starter-2.1.8.RELEASE.jar
BOOT-INF/lib/spring-boot-2.1.8.RELEASE.jar
BOOT-INF/lib/spring-boot-autoconfigure-2.1.8.RELEASE.jar
BOOT-INF/lib/spring-boot-starter-logging-2.1.8.RELEASE.jar
BOOT-INF/lib/logback-classic-1.2.3.jar
BOOT-INF/lib/logback-core-1.2.3.jar
BOOT-INF/lib/log4j-to-slf4j-2.11.2.jar
BOOT-INF/lib/log4j-api-2.11.2.jar
BOOT-INF/lib/jul-to-slf4j-1.7.28.jar
BOOT-INF/lib/javax.annotation-api-1.3.2.jar
BOOT-INF/lib/snakeyaml-1.23.jar
BOOT-INF/lib/spring-boot-starter-json-2.1.8.RELEASE.jar
BOOT-INF/lib/jackson-databind-2.9.9.3.jar
BOOT-INF/lib/jackson-annotations-2.9.0.jar
BOOT-INF/lib/jackson-core-2.9.9.jar
BOOT-INF/lib/jackson-datatype-jdk8-2.9.9.jar
BOOT-INF/lib/jackson-datatype-jsr310-2.9.9.jar
BOOT-INF/lib/jackson-module-parameter-names-2.9.9.jar
BOOT-INF/lib/spring-boot-starter-tomcat-2.1.8.RELEASE.jar
BOOT-INF/lib/tomcat-embed-core-9.0.24.jar
BOOT-INF/lib/tomcat-embed-el-9.0.24.jar
BOOT-INF/lib/tomcat-embed-websocket-9.0.24.jar
BOOT-INF/lib/hibernate-validator-6.0.17.Final.jar
BOOT-INF/lib/validation-api-2.0.1.Final.jar
BOOT-INF/lib/jboss-logging-3.3.3.Final.jar
BOOT-INF/lib/classmate-1.4.0.jar
BOOT-INF/lib/spring-web-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-beans-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-webmvc-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-aop-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-context-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-expression-5.1.9.RELEASE.jar
BOOT-INF/lib/slf4j-api-1.7.28.jar
BOOT-INF/lib/spring-core-5.1.9.RELEASE.jar
BOOT-INF/lib/spring-jcl-5.1.9.RELEASE.jar
```

大致看看里面的东西我们可以发现，除了.MF以及.class文件之外，jar还能打包静态资源文件如.html、.css以及.js等项目所需的一切，这也就意味着咱们能将自己的项目打成jar，即不管是web应用还是底层框架，都能打成jar包。
有的jar包是可以直接通过 java -jar 指令来执行的。我们都知道，有的类之所以能够执行，是因为它用你有main函数，该函数是程序的入口，同理，可执行的jar包中肯定是有某个.class文件提供了main函数才使得其可执行。那么问题来了，一个jar里面可能存在多个.class文件都有main函数的情况，我怎么知道该执行哪个？其实答案非常简单，就是看前面说的MANIFEST.MF里面的Main-Class属性，它会指定函数入口，相关知识咱们会在执行jar的时候进行讲解。

## 为什么要打jar包

在大致了解了什么是jar包了之后，咱们来讲讲为什么要打jar包。主要从我们自身的徐需求出发，不难发现，当我们开发了一个程序以后，程序中有很多的类，如果需要提供给别人使用,发给对方一大堆源文件是非常不好的，因此通常需要把这些类以及相关的资源文件打包成一个 jar 包,把这个 jar 包提供给别人使用,同时提供给使用者清晰的文档。这样他人在拿到我们提供的jar之后，就能方便地进行调用，具体如何调用后面会进行讲解。
因此，建议大家在平时写代码搬砖的时候，注意把自己代码的通用部分抽离出来，主键积累一些通用的util类，将其逐渐模块化，最后打成jar包供自己在别的项目或者模块中使用，同时不断打磨jar里面的内容，将其做得越来越容易理解和通用，这样的好处是除了会对你的代码重构能力以及模块抽象能力有很好的帮助之外，更是一种从长期解放你的重复工作量，让你有更多的精力去做其他事情的方式，甚至当你抽象出业内足够通用的jar之后，jar包还能为你带来意想不到的利润（当然公司里该保密的东西还是得保密的）。这也是java发展得如此之好的原因，无论出于盈利或者非盈利的目的，将自己的通用工具或者框架抽取出来，打成jar包供他人调用，使得整个java生态圈变得越来越强大–几乎很多业务场景都能找到对应的jar包。

## jar包和war包的区别

war包想必大家也都接触过，war是一个可以直接运行的web模块，通常应用于web项目中，将其打成war包部署到Tomcat等容器中。以大家熟悉的Tomcat举例，将war包放置在tomcat根目录的webapps目录下，如果Tomcat成功启动，这个包就会自动解压，就相当于发布了。我将我第一门实战课里面的war包解压，大家看看里面的结构，内容没有做任何删减

```xml
META-INF/MANIFEST.MF
META-INF/
META-INF/maven/
META-INF/maven/com.imooc/
META-INF/maven/com.imooc/myo2o/
META-INF/maven/com.imooc/myo2o/pom.properties
META-INF/maven/com.imooc/myo2o/pom.xml
META-INF/maven/com.imooc/o2o/
META-INF/maven/com.imooc/o2o/pom.properties
META-INF/maven/com.imooc/o2o/pom.xml
WEB-INF/
WEB-INF/classes/
WEB-INF/classes/com/
WEB-INF/classes/com/imooc/
WEB-INF/classes/com/imooc/o2o/
WEB-INF/classes/com/imooc/o2o/cache/
WEB-INF/classes/com/imooc/o2o/cache/JedisPoolWriper.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil$Hash.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil$Keys.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil$Lists.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil$Sets.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil$Strings.class
WEB-INF/classes/com/imooc/o2o/cache/JedisUtil.class
WEB-INF/classes/com/imooc/o2o/dao/
WEB-INF/classes/com/imooc/o2o/dao/split/
WEB-INF/classes/com/imooc/o2o/dao/split/DynamicDataSource.class
WEB-INF/classes/com/imooc/o2o/dao/split/DynamicDataSourceHolder.class
WEB-INF/classes/com/imooc/o2o/dao/split/DynamicDataSourceInterceptor.class
WEB-INF/classes/com/imooc/o2o/dao/AreaDao.class
WEB-INF/classes/com/imooc/o2o/dao/HeadLineDao.class
WEB-INF/classes/com/imooc/o2o/dao/LocalAuthDao.class
WEB-INF/classes/com/imooc/o2o/dao/PersonInfoDao.class
WEB-INF/classes/com/imooc/o2o/dao/ProductCategoryDao.class
WEB-INF/classes/com/imooc/o2o/dao/ProductDao.class
WEB-INF/classes/com/imooc/o2o/dao/ProductImgDao.class
WEB-INF/classes/com/imooc/o2o/dao/ShopCategoryDao.class
WEB-INF/classes/com/imooc/o2o/dao/ShopDao.class
WEB-INF/classes/com/imooc/o2o/dao/WechatAuthDao.class
WEB-INF/classes/com/imooc/o2o/dto/
WEB-INF/classes/com/imooc/o2o/dto/AreaExecution.class
WEB-INF/classes/com/imooc/o2o/dto/HeadLineExecution.class
WEB-INF/classes/com/imooc/o2o/dto/ImageHolder.class
WEB-INF/classes/com/imooc/o2o/dto/LocalAuthExecution.class
WEB-INF/classes/com/imooc/o2o/dto/ProductCategoryExecution.class
WEB-INF/classes/com/imooc/o2o/dto/ProductExecution.class
WEB-INF/classes/com/imooc/o2o/dto/Result.class
WEB-INF/classes/com/imooc/o2o/dto/ShopCategoryExecution.class
WEB-INF/classes/com/imooc/o2o/dto/ShopExecution.class
WEB-INF/classes/com/imooc/o2o/dto/UserAccessToken.class
WEB-INF/classes/com/imooc/o2o/dto/WechatAuthExecution.class
WEB-INF/classes/com/imooc/o2o/dto/WechatUser.class
WEB-INF/classes/com/imooc/o2o/entity/
WEB-INF/classes/com/imooc/o2o/entity/Area.class
WEB-INF/classes/com/imooc/o2o/entity/Award.class
WEB-INF/classes/com/imooc/o2o/entity/HeadLine.class
WEB-INF/classes/com/imooc/o2o/entity/LocalAuth.class
WEB-INF/classes/com/imooc/o2o/entity/PersonInfo.class
WEB-INF/classes/com/imooc/o2o/entity/Product.class
WEB-INF/classes/com/imooc/o2o/entity/ProductCategory.class
WEB-INF/classes/com/imooc/o2o/entity/ProductImg.class
WEB-INF/classes/com/imooc/o2o/entity/ProductSellDaily.class
WEB-INF/classes/com/imooc/o2o/entity/Shop.class
WEB-INF/classes/com/imooc/o2o/entity/ShopAuthMap.class
WEB-INF/classes/com/imooc/o2o/entity/ShopCategory.class
WEB-INF/classes/com/imooc/o2o/entity/UserAwardMap.class
WEB-INF/classes/com/imooc/o2o/entity/UserProductMap.class
WEB-INF/classes/com/imooc/o2o/entity/UserShopMap.class
WEB-INF/classes/com/imooc/o2o/entity/WechatAuth.class
WEB-INF/classes/com/imooc/o2o/enums/
WEB-INF/classes/com/imooc/o2o/enums/AreaStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/HeadLineStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/LocalAuthStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/ProductCategoryStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/ProductStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/ShopCategoryStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/ShopStateEnum.class
WEB-INF/classes/com/imooc/o2o/enums/WechatAuthStateEnum.class
WEB-INF/classes/com/imooc/o2o/exceptions/
WEB-INF/classes/com/imooc/o2o/exceptions/AreaOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/HeadLineOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/LocalAuthOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/ProductCategoryOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/ProductOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/ShopCategoryOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/ShopOperationException.class
WEB-INF/classes/com/imooc/o2o/exceptions/WechatAuthOperationException.class
WEB-INF/classes/com/imooc/o2o/interceptor/
WEB-INF/classes/com/imooc/o2o/interceptor/shopadmin/
WEB-INF/classes/com/imooc/o2o/interceptor/shopadmin/ShopLoginInterceptor.class
WEB-INF/classes/com/imooc/o2o/interceptor/shopadmin/ShopPermissionInterceptor.class
WEB-INF/classes/com/imooc/o2o/service/
WEB-INF/classes/com/imooc/o2o/service/impl/
WEB-INF/classes/com/imooc/o2o/service/impl/AreaServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/CacheServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/HeadLineServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/LocalAuthServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/PersonInfoServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/ProductCategoryServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/ProductServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/ShopCategoryServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/ShopServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/impl/WechatAuthServiceImpl.class
WEB-INF/classes/com/imooc/o2o/service/AreaService.class
WEB-INF/classes/com/imooc/o2o/service/CacheService.class
WEB-INF/classes/com/imooc/o2o/service/HeadLineService.class
WEB-INF/classes/com/imooc/o2o/service/LocalAuthService.class
WEB-INF/classes/com/imooc/o2o/service/PersonInfoService.class
WEB-INF/classes/com/imooc/o2o/service/ProductCategoryService.class
WEB-INF/classes/com/imooc/o2o/service/ProductService.class
WEB-INF/classes/com/imooc/o2o/service/ShopCategoryService.class
WEB-INF/classes/com/imooc/o2o/service/ShopService.class
WEB-INF/classes/com/imooc/o2o/service/WechatAuthService.class
WEB-INF/classes/com/imooc/o2o/util/
WEB-INF/classes/com/imooc/o2o/util/wechat/
WEB-INF/classes/com/imooc/o2o/util/wechat/MyX509TrustManager.class
WEB-INF/classes/com/imooc/o2o/util/wechat/SignUtil.class
WEB-INF/classes/com/imooc/o2o/util/wechat/WechatUtil.class
WEB-INF/classes/com/imooc/o2o/util/CodeUtil.class
WEB-INF/classes/com/imooc/o2o/util/DESUtil.class
WEB-INF/classes/com/imooc/o2o/util/EncryptPropertyPlaceholderConfigurer.class
WEB-INF/classes/com/imooc/o2o/util/HttpServletRequestUtil.class
WEB-INF/classes/com/imooc/o2o/util/ImageUtil.class
WEB-INF/classes/com/imooc/o2o/util/MD5.class
WEB-INF/classes/com/imooc/o2o/util/PageCalculator.class
WEB-INF/classes/com/imooc/o2o/util/PathUtil.class
WEB-INF/classes/com/imooc/o2o/web/
WEB-INF/classes/com/imooc/o2o/web/frontend/
WEB-INF/classes/com/imooc/o2o/web/frontend/FrontendController.class
WEB-INF/classes/com/imooc/o2o/web/frontend/MainPageController.class
WEB-INF/classes/com/imooc/o2o/web/frontend/ProductDetailController.class
WEB-INF/classes/com/imooc/o2o/web/frontend/ShopDetailController.class
WEB-INF/classes/com/imooc/o2o/web/frontend/ShopListController.class
WEB-INF/classes/com/imooc/o2o/web/local/
WEB-INF/classes/com/imooc/o2o/web/local/LocalAuthController.class
WEB-INF/classes/com/imooc/o2o/web/local/LocalController.class
WEB-INF/classes/com/imooc/o2o/web/shopadmin/
WEB-INF/classes/com/imooc/o2o/web/shopadmin/ProductCategoryManagementController.class
WEB-INF/classes/com/imooc/o2o/web/shopadmin/ProductManagementController.class
WEB-INF/classes/com/imooc/o2o/web/shopadmin/ShopAdminController.class
WEB-INF/classes/com/imooc/o2o/web/shopadmin/ShopController.class
WEB-INF/classes/com/imooc/o2o/web/shopadmin/ShopManagementController.class
WEB-INF/classes/com/imooc/o2o/web/superadmin/
WEB-INF/classes/com/imooc/o2o/web/superadmin/AreaController.class
WEB-INF/classes/com/imooc/o2o/web/wechat/
WEB-INF/classes/com/imooc/o2o/web/wechat/WechatController.class
WEB-INF/classes/com/imooc/o2o/web/wechat/WechatLoginController.class
WEB-INF/classes/mapper/
WEB-INF/classes/mapper/AreaDao.xml
WEB-INF/classes/mapper/HeadLineDao.xml
WEB-INF/classes/mapper/LocalAuthDao.xml
WEB-INF/classes/mapper/PersonInfoDao.xml
WEB-INF/classes/mapper/ProductCategoryDao.xml
WEB-INF/classes/mapper/ProductDao.xml
WEB-INF/classes/mapper/ProductImgDao.xml
WEB-INF/classes/mapper/ShopCategoryDao.xml
WEB-INF/classes/mapper/ShopDao.xml
WEB-INF/classes/mapper/WechatAuthDao.xml
WEB-INF/classes/spring/
WEB-INF/classes/spring/spring-dao.xml
WEB-INF/classes/spring/spring-redis.xml
WEB-INF/classes/spring/spring-service.xml
WEB-INF/classes/spring/spring-web.xml
WEB-INF/classes/1.jpg
WEB-INF/classes/AreaDao.xml
WEB-INF/classes/HeadLineDao.xml
WEB-INF/classes/LocalAuthDao.xml
WEB-INF/classes/PersonInfoDao.xml
WEB-INF/classes/ProductCategoryDao.xml
WEB-INF/classes/ProductDao.xml
WEB-INF/classes/ProductImgDao.xml
WEB-INF/classes/ShopCategoryDao.xml
WEB-INF/classes/ShopDao.xml
WEB-INF/classes/WechatAuthDao.xml
WEB-INF/classes/jdbc.properties
WEB-INF/classes/logback.xml
WEB-INF/classes/mybatis-config.xml
WEB-INF/classes/redis.properties
WEB-INF/classes/spring-dao.xml
WEB-INF/classes/spring-redis.xml
WEB-INF/classes/spring-service.xml
WEB-INF/classes/spring-web.xml
WEB-INF/classes/watermark.jpg
WEB-INF/classes/weixin.properties
WEB-INF/html/
WEB-INF/html/frontend/
WEB-INF/html/frontend/index.html
WEB-INF/html/frontend/productdetail.html
WEB-INF/html/frontend/shopdetail.html
WEB-INF/html/frontend/shoplist.html
WEB-INF/html/local/
WEB-INF/html/local/accountbind.html
WEB-INF/html/local/changepsw.html
WEB-INF/html/local/login.html
WEB-INF/html/shop/
WEB-INF/html/shop/productcategorymanagement.html
WEB-INF/html/shop/productmanagement.html
WEB-INF/html/shop/productoperation.html
WEB-INF/html/shop/shoplist.html
WEB-INF/html/shop/shopmanagement.html
WEB-INF/html/shop/shopoperation.html
WEB-INF/index.jsp
WEB-INF/web.xml
WEB-INF/lib/
WEB-INF/lib/logback-classic-1.2.3.jar
WEB-INF/lib/logback-core-1.2.3.jar
WEB-INF/lib/slf4j-api-1.7.25.jar
WEB-INF/lib/spring-core-4.3.7.RELEASE.jar
WEB-INF/lib/commons-logging-1.2.jar
WEB-INF/lib/spring-beans-4.3.7.RELEASE.jar
WEB-INF/lib/spring-context-4.3.7.RELEASE.jar
WEB-INF/lib/spring-aop-4.3.7.RELEASE.jar
WEB-INF/lib/spring-expression-4.3.7.RELEASE.jar
WEB-INF/lib/spring-jdbc-4.3.7.RELEASE.jar
WEB-INF/lib/spring-tx-4.3.7.RELEASE.jar
WEB-INF/lib/spring-web-4.3.7.RELEASE.jar
WEB-INF/lib/spring-webmvc-4.3.7.RELEASE.jar
WEB-INF/lib/javax.servlet-api-3.1.0.jar
WEB-INF/lib/jackson-databind-2.8.7.jar
WEB-INF/lib/jackson-annotations-2.8.0.jar
WEB-INF/lib/jackson-core-2.8.7.jar
WEB-INF/lib/commons-collections-3.2.jar
WEB-INF/lib/mybatis-3.4.2.jar
WEB-INF/lib/mybatis-spring-1.3.1.jar
WEB-INF/lib/mysql-connector-java-5.1.37.jar
WEB-INF/lib/c3p0-0.9.1.2.jar
WEB-INF/lib/thumbnailator-0.4.8.jar
WEB-INF/lib/kaptcha-2.3.2.jar
WEB-INF/lib/filters-2.0.235-1.jar
WEB-INF/lib/commons-fileupload-1.3.2.jar
WEB-INF/lib/commons-io-2.2.jar
WEB-INF/lib/jedis-2.9.0.jar
WEB-INF/lib/commons-pool2-2.4.2.jar
index.jsp
resources/
resources/css/
resources/css/frontend/
resources/css/frontend/index.css
resources/css/frontend/productdetail.css
resources/css/frontend/shopdetail.css
resources/css/frontend/shoplist.css
resources/css/shop/
resources/css/shop/productcategorymanagement.css
resources/css/shop/productmanagement.css
resources/css/shop/shoplist.css
resources/css/shop/shopmanagement.css
resources/js/
resources/js/common/
resources/js/common/common.js
resources/js/frontend/
resources/js/frontend/index.js
resources/js/frontend/productdetail.js
resources/js/frontend/shopdetail.js
resources/js/frontend/shoplist.js
resources/js/local/
resources/js/local/accountbind.js
resources/js/local/changepsw.js
resources/js/local/login.js
resources/js/local/logout.js
resources/js/shop/
resources/js/shop/productcategorymanagement.js
resources/js/shop/productmanagement.js
resources/js/shop/productoperation.js
resources/js/shop/shoplist.js
resources/js/shop/shopmanagement.js
resources/js/shop/shopoperation.js
resources/watermark.jpg
```

就会发现除了目录结构外，jar里有的war里也都有。war包是Sun提出的一种web应用程序格式，与jar类似，是很多文件的压缩包。war包中的文件按照一定目录结构来组织。根据其根目录下包含有html和jsp文件，或者包含有这两种文件的目录，另外还有WEB-INF目录。通常在WEB-INF目录下含有一个web.xml文件和一个classes目录，web.xml是这个应用的配置文件，而classes目录下则包含编译好的servlet类和jsp，或者servlet所依赖的其他类（如JavaBean）。通常这些所依赖的类也可以打包成jar包放在WEB-INF下的lib目录下。这也就意味着，war能打包的内容，jar也都可以。有的同学会问了，那既然是这样，直接用jar来替代war不就可以了？诚然，对于现今的应用来讲，主流都是用jar来替代war了。因为war仅服务于Web应用，而jar的涵盖范围更广。目前，war相较于jar的唯一优势在于，就拿tomcat来讲，当tomcat的进程启动之后，将符合规范的war包放在tomcat的webapps目录下的时候，tomcat会自动将war包解压并对外提供web服务，而jar包则不行。
![war能自动被解压](%E7%90%86%E8%A7%A3%20Jar.resource/5d7713900001e63212960170.png)
过去由于并未通过微服务将机器资源进行隔离，因此提倡的是一个tomcat实例管理多个java web项目，因此对于java web项目，都提倡将其打成war包然后放置于同一个tomcat的webapps下进行管理，便于资源的统一利用。而随着微服务成为主流，同一台机器上的多个web服务可以通过docker等容器进行隔离，因此我们可以让每个容器都单独运行一个tomcat实例，每个tomcat实例独立运行一个web服务，换句话说，我们可以像springboot一样，将tomcat和web项目打成jar放在一起，以内嵌的方式来启动web服务，使得所有服务的启动方式更优雅和统一，不管是Web服务还是后台服务，均使用java -jar指令来启动。

## 如何打jar包&&jar的依赖如何处理&如何调用

关于如何打jar包，应对不同的情况，可以有很多种方式，这里通过讲解应对主流需求的打包方式来让大家明白如何打jar包，其他的方式大家可以在课下触类旁通进行学习。

1. ### 含有多个类的jar，类之间存在调用关系

咱们先创建一个[java项目](https://coding.imooc.com/class/303.html?mc_marking=08b2d5f5d99349a1b253ed98a754f110&mc_channel=shouji)，编写两个非常简单的类，Welcome.java和Teacher.jar，其中Welcome类在main函数里调用了Teacher类的静态方法greeting

```java
Welcome.java
package com.imooc.jardemo1;

import com.imooc.jardemo1.impl.Teacher;

public class Welcome {
    public static void main(String[] args) {
        Teacher.greeting();
    }
}
Teacher.java
package com.imooc.jardemo1.impl;

public class Teacher {
    public static void greeting(){
        System.out.printf("Welcome!");
    }
}
```

整个项目结构如下图：
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d771dab0001394602770318.png)
编写完成后，在命令行里，去到项目的src路径下，执行javac指令

```shell
javac com/imooc/jardemo1/Welcome.java
```

此时就会生成与这两个类相对应的.class字节码文件
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d771e99000102d504700362.png)
由于jvm实际解析的是.class字节码文件而非.java文件，且jar中最好不要包含代码源文件，我们来将.class文件打个jar包，在src根目录下执行如下指令

```shell
jar -cvf welcome.jar com/imooc/jardemo1/Welcome.class com/imooc/jardemo1/impl/Teacher.class
```

c表示要创建一个新的jar包，v表示创建的过程中在控制台输出创建过程的一些信息，f表示给生成的jar包命名
执行结果如下
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d7720650001924511960211.png)
接下来便是如何执行上面的welcome.jar了，非常简单，只需要执行如下指令

```shell
java -jar welcome.jar
```

就会发现，事情果然没有这么简单，报错了：）
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d7721270001d06104300125.png)
通过异常信息我们不难看出，这是和先前咱们说过的MANIFEST.Mf这个主清单属性相关的，此时我们查看一下welcome.jar里面的内容，通过指令

```shell
jar -tf welcome.jar
```

就会发现我们之前打jar的时候，会生成一个META-INF的目录，里面有MANIFEST.MF这个清单列表
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77222000014c0705060156.png)
将welcome.jar拷贝到新建的文件夹myjar下面，用解压缩软件或者unzip指令解压，并打开MANIFEST.MF，就会发现文件里有如下内容

```shell
Manifest-Version: 1.0
Created-By: 11 (Oracle Corporation)
```

我们发现，缺少咱们先前说的Main-Class属性，导致jar被执行的时候，不知道执行哪个main函数。因此我们需要加上Main-Class，后接main函数所在类的全路径名（注意冒号之后一定要跟英文的空格，整个文件最后有一行空行）

```shell
 Manifest-Version: 1.0
 Created-By: 11 (Oracle Corporation)
 Main-Class: com.imooc.jardemo1.Welcome
 
```

添加完成后，重新执行指令打包，这次咱们在打包指令里多加一个参数，即多传入修改完成后的MANIFEST.MF文件

```shell
jar -cvfm welcome.jar META-INF/MANIFEST.MF  com/imooc/jardemo1/Welcome.class com/imooc/jardemo1/impl/Teacher.class
```

其中多了一个参数m，表示要定义MANIFEST文件。之后再重新执行

```shell
java -jar welcome.jar
```

就会发现jar已成功执行
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77a1c8000192f304190062.png)
有的同学会说有没有更简便点的方式，因为每次打jar的时候都要指定所有class文件。确实是有的，大家可以尝试一下，直接在src根目录下执行

```shell
javac com/imooc/jardemo1/Welcome.java -d target
```

该命令表示，将所有编译后的.class文件，都放到src/target文件夹下
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77b46600014c6017460358.png)
再将先前修改好的META-INF文件夹整体复制或者移动到target/下，去到target目录，直接执行

```
jar -cvfm welcome.jar META-INF/MANIFEST.MF * 
```

即可完成打包，注意最后一个位置变成了*，表示把当前目录下所有文件都打在jar包里。
此外，还有一种更简单的也更灵活的方式，不需要修改META-INF/MANIFEST.MF，即不需要指定main函数，而通过如下指令来动态指定

```
java -cp welcome.jar com.imooc.jardemo1.Welcome
```

其中cp表示classpath，后面接上全限的main函数所在的类即可
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77d9650001e65b13600270.png)
此种方式虽然灵活，但是由于不需要在MANIFEST.MF里面标注执行函数以及后面要将的Class-Path，需要调用方充分熟悉jar及其内部构造，否则需要在MANIFEST.MF以及相关的使用说明文档里描述清楚。

1. ### 含有多个jar，jar之间存在调用关系

咱们还是跟刚刚的项目一样，只是这次将Teacher.class和Welcome.class打成两个jar，并且为了避免IDE报错，咱们直接手写java类，创建一个jardemo2目录，进到目录里，再创建一个src目录，再进入到src目录里，创建按照package的形式创建深层的文件夹com/imooc/jardemo2，之后在jardemo2里面创建Welcome.java，跟先前类似

```java
package com.imooc.jardemo2;

import com.imooc.jardemo1.impl.Teacher;

public class Welcome {
    public static void main(String[] args) {
        Teacher.greeting();
    }
}
```

唯一的区别是package从com.imooc.jardemo1变成了com.imooc.jardemo2，但是仍然引用的是jardemo1里面的Teacher，com.imooc.jardemo1.impl.Teacher。
如果此时直接在src目录下执行

```shell
javac  com/imooc/jardemo2/Welcome.java -d target/ 
```

必然会报错，提示Teacher找不到
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77b89500019fad13680660.png)
为了保证编译通过，除了把Teacher连同其文件目录结构一同复制过来之外，咱们还可以在原先的jardemo1项目里给Teacher.class打个jar，即在jardemo1/src目录下执行

```
javac com/imooc/jardemo1/impl/Teacher.java -d target2/
```

随后去到target2文件夹里将里面的信息打个jar包

```
jar -cvf teacher.jar *
```

![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77b9b600017efc20180556.png)
将生成好的jar复制粘贴到jardemo2项目的lib目录底下（需要先创建好lib目录，其位于jardemo2根目录下，与src同级)
随后来到src目录下，执行

```
javac -cp ../lib/teacher.jar com/imooc/jardemo2/Welcome.java -d target
```

便会发现Teacher.java会被成功编译，相关的class文件以及package对应的目录均被放置在src/target目录底下，这里的 -cp 表示 -classpath，指的是把teacher.jar加入classpath路径下
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77bb85000170d513580216.png)
此时咱们在target目录底下创建好META-INF/MANIFEST.MF，并写入以下内容

```shell
Manifest-Version: 1.0
Created-By: 11 (Oracle Corporation)
Main-Class: com.imooc.jardemo2.Welcome
```

然后在target里打jar包

```
jar -cvfm welcome.jar META-INF/MANIFEST.MF * 
```

![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77be6f0001d02613720660.png)
就会发现相关的jar已经被打包成功，咱们是无法直接调用jar的
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77bee70001694513540814.png)
还是报找不到Teacher这个类的错误，原因是因为咱们打包的jar里面并为包含Teacher.class，大家不信可以解压出来看看就知道了。
那该如何解决呢，其实思路非常简单，我们只需要跟javac -cp一样，将teacher.jar加入到classpath里即可，如何实现呢？此时我们便需要了解MANIFEST.MF的另外一个属性Class-Path了，这个属性看名字就知道是用来指定CLASSPATH的，CLASSPATH是指定程序中所使用的类文件所在的位置，相信咱们在编写项目的时候，经常也会接触到这个词，此时我们需要修改target里面的META-INF/MANIFEST.MF文件，添加Class-Path属性，指定相对当前的执行路径，即target，teacher在什么路径下（如果是多个jar，则用英文空格隔开)。

```
Manifest-Version: 1.0
Created-By: 11 (Oracle Corporation)
Main-Class: com.imooc.jardemo2.Welcome
Class-Path: ../../lib/teacher.jar
```

修改好之后，重新打jar

```
jar -cvfm welcome.jar META-INF/MANIFEST.MF * 
```

之后再执行jar包，就会发现执行成功了
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77c25d0001a45710120218.png)
当然，更快捷的方式还是通过现在的java -cp指令来执行
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77dad0000136dc13460372.png)

1. ### 读取jar内的资源文件

这种情况就是在普通的java项目内部创建一个资源文件并读取，由于实际和资源文件都打包在了一块，可以直接调用。像这里，如果在根目录下执行jar包的main函数时，main函数有如下指令

```java
InputStream is = Welcome.getClass().getResourceAsStream("static/text.txt");
```

则便能获取到项目根目录static/下面的text.txt的信息。

1. ### 读取jar外的资源文件

这种情况更简单，指明需要去读取的路径即可。像这里，如果在根目录下执行jar包的main函数时，main函数有如下指令

```java
InputStream is = new FileInputStream("/home/work/outside/text.txt");
```

则便能获取到/home/work/outside/text.txt绝对路径下的text.txt内容。

1. ### 读取外部jar包里的资源文件

结合前面的第2，3种情况，咱们可以先指定MANIFEST.MF里的Class-Path为所要读取的jar包所在的路径，之后和第3种情况一样访问目标jar中的资源文件即可。

1. ### 访问Jar包内部的Jar包资源

接着jardemo2，咱们把teacher.jar从src/lib复制一份，粘贴到target里，同时将target里的welcome.jar删除，如图
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77c4d70001843212180380.png)
随后将META-INF/MANIFEST.MF里的Class-Path给去了，因为现在我们想把teacher.jar打包进welcome.jar里面，并希望teacher.jar能够自动被调用。

```
Manifest-Version: 1.0
Created-By: 11 (Oracle Corporation)
Main-Class: com.imooc.jardemo2.Welcome
```

重新打包成welcome.jar
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77c5a90001061413580720.png)
随后执行welcome.jar，便发现会报错![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77c6c20001113e13680766.png)
当项目中咱们把所需要的第三方jar包也打进自己的jar包中时，如果仍然按照上述操作方式，会报找不到Class异常，原因就是jar引用不到放在自己内部的jar包。咱们尝试重新加入Class-Path

```
Manifest-Version: 1.0
Created-By: 11 (Oracle Corporation)
Main-Class: com.imooc.jardemo2.Welcome
Class-Path: teacher.jar
```

再次打包后，将target目录下的teacher.jar删除并执行，会发现，依旧不能执行成功
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77cef90001820b13500754.png)
这下坏了，无论怎么尝试都访问不到jar里的jar，该如何是好？此时就需要适当地深入去分析其中的奥秘。我们都知道，执行jar其实也就是执行里面的class，而class之所以能够被执行，前提提交是被classloader加载到内存当中，而目前如何加载内部jar的问题也就简化到了如何让classloader加载这些存在于内部jar里的class。
咱们首先来了解下classloader的加载机制，这个在[我的面试课程中](https://coding.imooc.com/class/303.html?mc_marking=1f1eb391b59b3e4139718a46d8673049&mc_channel=syb10)提到过。
主要是研究其双亲委派机制，这里限于篇幅，大致讲解一下jar的运行过程。jar 运行过程和类加载机制有关，而类加载机制又和我们自定义的类加载器有关，现在我们先来了解一下双亲委派模式。

java 中类加载器分为三个：

1. BootstrapClassLoader 负责加载 ${JAVA_HOME}/jre/lib 部分 jar 包
2. ExtClassLoader 加载 ${JAVA_HOME}/jre/lib/ext 下面的 jar 包
3. AppClassLoader 加载用户自定义 -classpath 或者 Jar 包的 Class-Path 定义的第三方包

类的生命周期为：加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using) 和 卸载(Unloading)七个阶段。

当我们执行 java -jar 的时候 jar 文件以二进制流的形式被读取到内存，但不会加载到 jvm 中，类会在一个合适的时机加载到虚拟机中。类加载的时机：

1. 遇到 new、getstatic、putstatic 或 invokestatic 这四条字节码指令时，如果类没有进行过初始化，则需要先对其进行初始化。这四条指令的最常见的 Java 代码场景是使用 new 关键字实例化对象的时候，读取或设置一个类的静态字段调用一个类的静态方法的时候。
2. 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类。

当触发类加载的时候，类加载器也不是直接加载这个类。首先交给 AppClassLoader ，它会查看自己有没有加载过这个类，如果有直接拿出来，无须再次加载，如果没有就将加载任务传递给 ExtClassLoader ，而 ExtClassLoader 也会先检查自己有没有加载过，没有又会将任务传递给 BootstrapClassLoader ，最后 BootstrapClassLoader 会检查自己有没有加载过这个类，如果没有就会去自己要寻找的区域去寻找这个类，如果找不到又将任务传递给 ExtClassLoader ，以此类推最后才是 AppClassLoader 加载我们的类。这样做是确保类只会被加载一次。通常我们的类加载器只识别 classpath （这里的 classpath 指项目根路径，也就是 jar 包内的位置）下 .class 文件。jar 中其他的文件包括 jar 包被当做了资源文件，而不会去读取里面的 .class 文件。但实际上我们可以通过自定义类加载器来实现一些特别的操作。

学到这里，我们便大致明白，之前咱们这样的做法是使用AppClassloader来加载相关jar里面的class的，而在加了-jar参数之后，AppClassloader就只关注welcome.jar范围内的class了，注意这里说的是class，并不包含内部的jar，其内部的jar此时相当于是前面说的内部资源文件，是以二进制流的形式存在的，因此，此时是访问不到内部jar文件的。那该如何是好？其实，这里的线索已经很充足了，我们其实就是用自定义的classloader来发现并获取其内部jar里的class即可。自定义ClassLoader需要继承ClassLoader抽象类，重写findClass方法，这个方法定义了ClassLoader查找class的方式。前面提到，Java 本身支持访问Jar包里面的资源， 他们以 Stream 的形式存在（他们本就处于Jar包之中），而Jar文件被描述为JarFile， 里面的资源文件被描述为JarEntry，可以通过判断JarEntry的Jar属性使得直接访问Jar包内部的Jar包，这里给出一些关键的程序语句以及思路。
首先我们可以以前面静态访问jar内部资源文件的方式访问jar

```java
　　　InputStream stream = 
　　　　 ClassLoader.getSystemResourceAsStream(name); 
```

其中name可以通过遍历jar里面的内容获取到，并且能够过滤出以.jar结尾的文件名并读取
获取到InputStream之后，可以将其转换为File（网上很多教程），而后转换成JarFile

```java
　　　 JarFile jarFile = new JarFile(file); 
```

获取到了jarFile之后，便能获取到jar里面的信息并进行后续的操作了，

```java
 Enumeration enum = jarFile.entries(); 
　　 while (enum.hasMoreElements()) { 
　　　 process(enum.nextElement()); 
　　 } 
```

后续的操作无非就是获取到class二进制流并传递给classloader defineClass去定义并做后续加载（能实现的前提是你了解自定义类加载器的工作原理）
上述过程比较复杂，如果希望直接在自己的类里面访问引用在 Jar包中的Jar包， 可以使用Spring Boot打包插件。强烈推荐该方案， 适用于所有的jar项目，主要通过maven打包
pom.xml

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

具体过程就靠大家去研究了。这里大致讲解下Springboot的Jar包内的类加载原理。咱们可以尝试打包一个最简单的springboot jar包，查看META-INF/MANIFEST.MF中的Main-Class，就会发现是Spring Boot的自定义类org.springframework.boot.loader.JarLauncher。
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77eb8900012ebc21880804.png)
其加载原理则是通过自定义类加载器LaunchedURLClassLoader实现类加载。 流程图如下：
![图片描述](%E7%90%86%E8%A7%A3%20Jar.resource/5d77ecd7000114cb12941252.png)
大致了解即可：)

## [总结与展望](https://coding.imooc.com/class/303.html?mc_marking=08b2d5f5d99349a1b253ed98a754f110&mc_channel=shouji)

这几天熬夜通宵，码了这么多字总算完成分享，也辛苦大家跟着看完了，又不懂的地方没有关系，可以多看几次，相信每次都会有不一样的收获。[课程](https://coding.imooc.com/class/303.html?mc_marking=08b2d5f5d99349a1b253ed98a754f110&mc_channel=shouji)里面我们首先介绍了jar包，方便大家对jar有一个比较直观的认知（zip包+主清单），之后讲解了为什么要打jar以及war和jar的区别，后面用重笔墨讲解了打jar包的方式以及jar内调用jar的思路，希望大家能够喜欢。

总结了过去咱们来展望下未来，在计划今年十二月底上线的课程中，我们将会对spring源码进行深入讲解，我会用通俗易懂，深入浅出的方式进行讲解，而且这种方式能够让大家学到额外的知识，敬请期待：）
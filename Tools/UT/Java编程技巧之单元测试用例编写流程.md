## Java 编程技巧之单元测试用例编写流程

## 前言

清代杰出思想家章学诚有一句名言：“学必求其心得，业必贵其专精。”

意思是：学习上一定要追求心得体会，事业上一定要贵以专注精深。做技术就是这样，一件事如果做到了极致，就必然会有所心得体会。作者最近在一个项目上，追求单元测试覆盖率到极致，所以才有了这篇心得体会。

上一篇文章[《Java单元测试技巧之PowerMock》](http://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247502647&idx=1&sn=eb1600c04511243d5a2c4c3e6ea43bfa&chksm=e92af638de5d7f2e2b0078bb4cee5b67a7b6d0f8ee3f7e666db9a753fede8ca530884e6ae03b&scene=21#wechat_redirect)除了介绍单元测试基础知识外，主要介绍了“为什么要编写单元测试”。很多同学读完后，还是不能快速地编写单元测试用例。而这篇文章，立足于“如何来编写单元测试用例”，能够让同学们“有章可循”，能快速地编写出单元测试用例。

## 一、编写单元测试用例

### （一）测试框架简介

Mockito 是一个单元测试模拟框架，可以让你写出优雅、简洁的单元测试代码。Mockito 采用了模拟技术，模拟了一些在应用中依赖的复杂对象，从而把测试对象和依赖对象隔离开来。

PowerMock 是一个单元测试模拟框架，是在其它单元测试模拟框架的基础上做出扩展。通过提供定制的类加载器以及一些字节码篡改技术的应用，PowerMock 实现了对静态方法、构造方法、私有方法以及 final 方法的模拟支持等强大的功能。但是，正因为 PowerMock 进行了字节码篡改，导致部分单元测试用例并不被JaCoco 统计覆盖率。

优先推荐使用 Mockito 提供的功能；只有在 Mockito 提供的功能不能满足需求时，才会采用 PowerMock 提供的功能；但是，不推荐使用影响 JaCoco 统计覆盖率的 PowerMock 功能。在本文中，我们也不会对影响 JaCoco 统计覆盖率的 PowerMock 功能进行介绍。

下面，将以 Mockito 为主、以 PowerMock 为辅，介绍一下如何编写单元测试用例。

### （二）测试框架引入

为了引入 Mockito 和 PowerMock 包，需要在 maven 项目的 pom.xml 文件中加入以下包依赖：

```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-core</artifactId>
    <version>${powermock.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>${powermock.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>${powermock.version}</version>
    <scope>test</scope>
</dependency>
```

其中，powermock.version 为 2.0.9，为当前的最新版本，可根据实际情况修改。在 PowerMock 包中，已经包含了对应的 Mockito 和 JUnit 包，所以无需单独引入 Mockito 和 JUnit 包。



### （三）典型代码案例

一个典型的服务代码案例如下：

```java
/**
 * 用户服务类
 */
@Service
public class UserService {
    /** 服务相关 */
    /** 用户DAO */
    @Autowired
    private UserDAO userDAO;
    /** 标识生成器 */
    @Autowired
    private IdGenerator idGenerator;

    /** 参数相关 */
    /** 可以修改 */
    @Value("${userService.canModify}")
    private Boolean canModify;

    /**
     * 创建用户
     * 
     * @param userCreate 用户创建
     * @return 用户标识
     */
    public Long createUser(UserVO userCreate) {
        // 获取用户标识
        Long userId = userDAO.getIdByName(userCreate.getName());

        // 根据存在处理
        // 根据存在处理: 不存在则创建
        if (Objects.isNull(userId)) {
            userId = idGenerator.next();
            UserDO create = new UserDO();
            create.setId(userId);
            create.setName(userCreate.getName());
            userDAO.create(create);
        }
        // 根据存在处理: 已存在可修改
        else if (Boolean.TRUE.equals(canModify)) {
            UserDO modify = new UserDO();
            modify.setId(userId);
            modify.setName(userCreate.getName());
            userDAO.modify(modify);
        }
        // 根据存在处理: 已存在禁修改
        else {
            throw new UnsupportedOperationException("不支持修改");
        }

        // 返回用户标识
        return userId;
    }
}
```



### （四）测试用例编写

采用 Mockito 和 PowerMock 单元测试模拟框架，编写的单元测试用例如下：

**UserServiceTest.java**

```java

/**
 * 用户服务测试类
 */
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    /** 模拟依赖对象 */
    /** 用户DAO */
    @Mock
    private UserDAO userDAO;
    /** 标识生成器 */
    @Mock
    private IdGenerator idGenerator;

    /** 定义被测对象 */
    /** 用户服务 */
    @InjectMocks
    private UserService userService;

    /**
     * 在测试之前
     */
    @Before
    public void beforeTest() {
        // 注入依赖对象
        Whitebox.setInternalState(userService, "canModify", Boolean.TRUE);
    }

    /**
     * 测试: 创建用户-新
     */
    @Test
    public void testCreateUserWithNew() {
        // 模拟依赖方法
        // 模拟依赖方法: userDAO.getByName
        Mockito.doReturn(null).when(userDAO).getIdByName(Mockito.anyString());
        // 模拟依赖方法: idGenerator.next
        Long userId = 1L;
        Mockito.doReturn(userId).when(idGenerator).next();

        // 调用被测方法
        String text = ResourceHelper.getResourceAsString(getClass(), "userCreateVO.json");
        UserVO userCreate = JSON.parseObject(text, UserVO.class);
        Assert.assertEquals("用户标识不一致", userId, userService.createUser(userCreate));

        // 验证依赖方法
        // 验证依赖方法: userDAO.getByName
        Mockito.verify(userDAO).getIdByName(userCreate.getName());
        // 验证依赖方法: idGenerator.next
        Mockito.verify(idGenerator).next();
        // 验证依赖方法: userDAO.create
        ArgumentCaptor<UserDO> userCreateCaptor = ArgumentCaptor.forClass(UserDO.class);
        Mockito.verify(userDAO).create(userCreateCaptor.capture());
        text = ResourceHelper.getResourceAsString(getClass(), "userCreateDO.json");
        Assert.assertEquals("用户创建不一致", text, JSON.toJSONString(userCreateCaptor.getValue()));

        // 验证依赖对象
        Mockito.verifyNoMoreInteractions(idGenerator, userDAO);
    }

    /**
     * 测试: 创建用户-旧
     */
    @Test
    public void testCreateUserWithOld() {
        // 模拟依赖方法
        // 模拟依赖方法: userDAO.getByName
        Long userId = 1L;
        Mockito.doReturn(userId).when(userDAO).getIdByName(Mockito.anyString());

        // 调用被测方法
        String text = ResourceHelper.getResourceAsString(getClass(), "userCreateVO.json");
        UserVO userCreate = JSON.parseObject(text, UserVO.class);
        Assert.assertEquals("用户标识不一致", userId, userService.createUser(userCreate));

        // 验证依赖方法
        // 验证依赖方法: userDAO.getByName
        Mockito.verify(userDAO).getIdByName(userCreate.getName());
        // 验证依赖方法: userDAO.modify
        ArgumentCaptor<UserDO> userModifyCaptor = ArgumentCaptor.forClass(UserDO.class);
        Mockito.verify(userDAO).modify(userModifyCaptor.capture());
        text = ResourceHelper.getResourceAsString(getClass(), "userModifyDO.json");
        Assert.assertEquals("用户修改不一致", text, JSON.toJSONString(userModifyCaptor.getValue()));

        // 验证依赖对象
        Mockito.verifyNoInteractions(idGenerator);
        Mockito.verifyNoMoreInteractions(userDAO);
    }

    /**
     * 测试: 创建用户-异常
     */
    @Test
    public void testCreateUserWithException() {
        // 注入依赖对象
        Whitebox.setInternalState(userService, "canModify", Boolean.FALSE);

        // 模拟依赖方法
        // 模拟依赖方法: userDAO.getByName
        Long userId = 1L;
        Mockito.doReturn(userId).when(userDAO).getIdByName(Mockito.anyString());

        // 调用被测方法
        String text = ResourceHelper.getResourceAsString(getClass(), "userCreateVO.json");
        UserVO userCreate = JSON.parseObject(text, UserVO.class);
        UnsupportedOperationException exception = Assert.assertThrows("返回异常不一致",
            UnsupportedOperationException.class, () -> userService.createUser(userCreate));
        Assert.assertEquals("异常消息不一致", "不支持修改", exception.getMessage());
    }
}
```

**userCreateVO.json**

`{"name":"test"}`

**userCreateDO.json**

`{"id":1,"name":"test"}`

**userModifyDO.json**

`{"id":1,"name":"test"}`

通过执行以上测试用例，可以看到对源代码进行了100%的行覆盖。



## 二、测试用例编写流程

通过上一章编写Java类单元测试用例的实践，可以总结出以下Java类单元测试用例的编写流程：



![图片](Java编程技巧之单元测试用例编写流程.resource/640)

上面一共有 3 个测试用例，这里仅以测试用例 testCreateUserWithNew(测试：创建用户-新)为例说明。

### （一）定义对象阶段

第 1 步是定义对象阶段，主要包括定义被测对象、模拟依赖对象（类成员）、注入依赖对象（类成员）3大部分。

**定义被测对象**

在编写单元测试时，首先需要定义被测对象，或直接初始化、或通过 Spy 包装……其实，就是把被测试服务类进行实例化。

```java
/** 定义被测对象 */
/** 用户服务 */
@InjectMocks
private UserService userService;
```

**模拟依赖对象（类成员）**

在一个服务类中，我们定义了一些类成员对象——服务（Service）、数据访问对象（DAO）、参数（Value）等。在Spring框架中，这些类成员对象通过`@Autowired`、`@Value` 等方式注入，它们可能涉及复杂的环境配置、依赖第三方接口服务……但是，在单元测试中，为了解除对这些类成员对象的依赖，我们需要对这些类成员对象进行模拟。

```java
/** 模拟依赖对象 */
/** 用户DAO */
@Mock
private UserDAO userDAO;
/** 标识生成器 */
@Mock
private IdGenerator idGenerator;
```

**注入依赖对象（类成员）**

当模拟完这些类成员对象后，我们需要把这些类成员对象注入到被测试类的实例中。以便在调用被测试方法时，可能使用这些类成员对象，而不至于抛出空指针异常。

```java
/** 定义被测对象 */
/** 用户服务 */
@InjectMocks
private UserService userService;
/**
 * 在测试之前
 */
@Before
public void beforeTest() {
    // 注入依赖对象
    Whitebox.setInternalState(userService, "canModify", Boolean.TRUE);
}
```

### （二）模拟方法阶段

第 2 步是模拟方法阶段，主要包括模拟依赖对象（参数或返回值）、模拟依赖方法2大部分。

**模拟依赖对象（参数或返回值）**

通常，在调用一个方法时，需要先指定方法的参数，然后获取到方法的返回值。所以，在模拟方法之前，需要先模拟该方法的参数和返回值。

```
Long userId = 1L;
```

**模拟依赖方法**

在模拟完依赖的参数和返回值后，就可以利用Mockito和PowerMock的功能，进行依赖方法的模拟。如果依赖对象还有方法调用，还需要模拟这些依赖对象的方法。

```java
// 模拟依赖方法
// 模拟依赖方法: userDAO.getByName
Mockito.doReturn(null).when(userDAO).getIdByName(Mockito.anyString());
// 模拟依赖方法: idGenerator.next
Mockito.doReturn(userId).when(idGenerator).next();
```

### （三）调用方法 阶段

第 3 步是调用方法阶段，主要包括模拟依赖对象（参数）、调用被测方法、验证参数对象（返回值）3步。

**模拟依赖对象（参数）**

在调用被测方法之前，需要模拟被测方法的参数。如果这些参数还有方法调用，还需要模拟这些参数的方法。

```java
String text = ResourceHelper.getResourceAsString(getClass(), "userCreateVO.json");
UserVO userCreate = JSON.parseObject(text, UserVO.class);
```

**调用被测方法**

在准备好参数对象后，就可以调用被测试方法了。如果被测试方法有返回值，需要定义变量接收返回值；如果被测试方法要抛出异常，需要指定期望的异常。

```java
userService.createUser(userCreate)
```

**验证数据对象（返回值）**

在调用被测试方法后，如果被测试方法有返回值，需要验证这个返回值是否符合预期；如果被测试方法要抛出异常，需要验证这个异常是否满足要求。

```java
Assert.assertEquals("用户标识不一致", userId, userService.createUser(userCreate));
```



### （四） 验证方法阶段

第 4 步是验证方法阶段，主要包括验证依赖方法、验证数据对象（参数）、验证依赖对象3步。

**验证依赖方法**

作为一个完整的测试用例，需要对每一个模拟的依赖方法调用进行验证。

```java
// 验证依赖方法
// 验证依赖方法: userDAO.getByName
Mockito.verify(userDAO).getIdByName(userCreate.getName());
// 验证依赖方法: idGenerator.next
Mockito.verify(idGenerator).next();
// 验证依赖方法: userDAO.create
ArgumentCaptor userCreateCaptor = ArgumentCaptor.forClass(UserDO.class);
Mockito.verify(userDAO).create(userCreateCaptor.capture());
```

**验证数据对象（参数）**

对应一些模拟的依赖方法，有些参数对象是被测试方法内部生成的。为了验证代码逻辑的正确性，就需要对这些参数对象进行验证，看这些参数对象值是否符合预期。

```java
text = ResourceHelper.getResourceAsString(getClass(), "userCreateDO.json");
Assert.assertEquals("用户创建不一致", text, JSON.toJSONString(userCreateCaptor.getValue()));
```

**验证依赖对象**

作为一个完整的测试用例，应该保证每一个模拟的依赖方法调用都进行了验证。正好，Mockito提供了一套方法，用于验证模拟对象所有方法调用都得到了验证。

```java
// 验证依赖对象
Mockito.verifyNoMoreInteractions(idGenerator, userDAO);
```

## 三、定义被测对象

在编写单元测试时，首先需要定义被测对象，或直接初始化、或通过Spy包装……其实，就是把被测试服务类进行实例化。

### （一）直接构建对象

直接构建一个对象，总是简单又直接。

```java
UserService userService = new UserService();
```

### （二）利用 Mockito.spy 方法

Mockito 提供一个 spy 功能，用于拦截那些尚未实现或不期望被真实调用的方法，默认所有方法都是真实方法，除非主动去模拟对应方法。所以，利用spy功能来定义被测对象，适合于需要模拟被测类自身方法的情况，适用于普通类、接口和虚基类。

```java
UserService userService = Mockito.spy(new UserService());
UserService userService = Mockito.spy(UserService.class);
AbstractOssService ossService = Mockito.spy(AbstractOssService.class);
```

### （三）利用 @Spy 注解

@Spy 注解跟 Mockito.spy 方法一样，可以用来定义被测对象，适合于需要模拟被测类自身方法的情况，适用于普通类、接口和虚基类。@Spy 注解需要配合 @RunWith 注解使用。

```java
@RunWith(PowerMockRunner.class)
public class CompanyServiceTest {
    @Spy
    private UserService userService = new UserService();

    ...
}
```

> 注意：@Spy注解对象需要初始化。如果是虚基类或接口，可以用 Mockito.mock 方法实例化。

### （四）利用 @InjectMocks 注解

@InjectMocks 注解用来创建一个实例，并将其它对象（@Mock、@Spy或直接定义的对象）注入到该实例中。所以，@InjectMocks 注解本身就可以用来定义被测对象。@InjectMocks 注解需要配合 @RunWith 注解使用。

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @InjectMocks
    private UserService userService;
    ...

}
```

## 四、模拟依赖对象

在编写单元测试用例时，需要模拟各种依赖对象——类成员、方法参数和方法返回值。

### （一）直接构建对象

如果需要构建一个对象，最简单直接的方法就是——定义对象并赋值。

```java
Long userId = 1L;
String userName = "admin";
UserDO user = new User();
user.setId(userId);
user.setName(userName);
List userIdList = Arrays.asList(1L, 2L, 3L);
```

### （二）反序列化对象

如果对象字段或层级非常庞大，采用直接构建对象方法，可能会编写大量构建程序代码。这种情况，可以考虑反序列化对象，将会大大减少程序代码。由于JSON字符串可读性高，这里就以JSON为例，介绍反序列化对象。

**反序列化模型对象**

```java
String text = ResourceHelper.getResourceAsString(getClass(), "user.json");
UserDO user = JSON.parseObject(text, UserDO.class);
```

**反序列化集合对象**

```java
String text = ResourceHelper.getResourceAsString(getClass(), "userList.json");
List userList = JSON.parseArray(text, UserDO.class);
```

**反序列化映射对象**

```java
String text = ResourceHelper.getResourceAsString(getClass(), "userMap.json");
Map userMap = JSON.parseObject(text, new TypeReference>() {});
```

### （三）利用 Mockito.mock 方法

Mockito 提供一个 mock 功能，用于拦截那些尚未实现或不期望被真实调用的方法，默认所有方法都已被模拟——方法为空并返回默认值（null或0），除非主动执行 doCallRealMethod 或 thenCallRealMethod 操作，才能够调用真实的方法。

利用 Mockito.mock 方法模拟依赖对象，主要用于以下几种情形：

1. 只使用类实例，不使用类属性；
2. 类属性太多，但使用其中少量属性（可以mock属性返回值）；
3. 类是接口或虚基类，并不关心其具体实现类。

```java
MockClass mockClass = Mockito.mock(MockClass.class);
List userIdList = (List)Mockito.mock(List.class);
```

### （四） 利用 @Mock 注解

@Mock 注解跟 Mockito.mock 方法一样，可以用来模拟依赖对象，适用于普通类、接口和虚基类。@Mock注解需要配合 @RunWith 注解使用。

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @Mock
    private UserDAO userDAO;
    ...

}
```

### （五）利用 Mockito.spy 方法

Mockito.spy 方法跟 Mockito.mock 方法功能相似，只是Mockito.spy方法默认所有方法都是真实方法，除非主动去模拟对应方法。

```java
UserService userService = Mockito.spy(new UserService());
UserService userService = Mockito.spy(UserService.class);
AbstractOssService ossService = Mockito.spy(AbstractOssService.class);
```

### （六）利用@Spy注解

@Spy注解跟Mockito.spy方法一样，可以用来模拟依赖对象，适用于普通类、接口和虚基类。@Spy注解需要配合@RunWith注解使用。

```java
@RunWith(PowerMockRunner.class)
public class CompanyServiceTest {
    @Spy
    private UserService userService = new UserService();

    ...
}
```

> 注意：@Spy注解对象需要初始化。如果是虚基类或接口，可以用Mockito.mock方法实例化。



## 五、注入依赖对象

当模拟完这些类成员对象后，我们需要把这些类成员对象注入到被测试类的实例中。以便在调用被测试方法时，可能使用这些类成员对象，而不至于抛出空指针异常。

### （一）利用Setter方法注入

如果类定义了Setter方法，可以直接调用方法设置字段值。

```java
userService.setMaxCount(100);
userService.setUserDAO(userDAO);
```

### （二）利用ReflectionTestUtils.setField方法注入

JUnit 提供 ReflectionTestUtils.setField 方法设置属性字段值。

```java
ReflectionTestUtils.setField(userService, "maxCount", 100);
ReflectionTestUtils.setField(userService, "userDAO", userDAO);
```

### （三）利用 Whitebox.setInternalState 方法注入

PowerMock 提供 Whitebox.setInternalState 方法设置属性字段值。

```java
Whitebox.setInternalState(userService, "maxCount", 100);
Whitebox.setInternalState(userService, "userDAO", userDAO);
```

### （四）利用@InjectMocks注解注入

@InjectMocks注解用来创建一个实例，并将其它对象（@Mock、@Spy或直接定义的对象）注入到该实例中。@InjectMocks注解需要配合@RunWith注解使用。

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @Mock
    private UserDAO userDAO;
    private Boolean canModify;

    @InjectMocks
    private UserService userService;
    ...
}
```

### （五）设置静态常量字段值

有时候，我们需要对静态常量对象进行模拟，然后去验证是否执行了对应分支下的方法。比如：需要模拟Lombok的@Slf4j生成的log静态常量。但是，Whitebox.setInternalState方法和@InjectMocks注解并不支持设置静态常量，需要自己实现一个设置静态常量的方法：

```java
public final class FieldHelper {
    public static void setStaticFinalField(Class clazz, String fieldName, Object fieldValue) throws NoSuchFieldException, IllegalAccessException {
        Field field = clazz.getDeclaredField(fieldName);
        FieldUtils.removeFinalModifier(field);
        FieldUtils.writeStaticField(field, fieldValue, true);
    }
}
```

具体使用方法如下：

```java
FieldHelper.setStaticFinalField(UserService.class, "log", log);
```

> 注意：经过测试，该方法对于int、Integer等基础类型并不生效，应该是编译器常量优化导致。



## 六、模拟依赖方法

在模拟完依赖的参数和返回值后，就可以利用 Mockito 和 PowerMock 的功能，进行依赖方法的模拟。如果依赖对象还有方法调用，还需要模拟这些依赖对象的方法。

### （一） 根据返回模拟方法

**模拟无返回值方法**

```java
Mockito.doNothing().when(userDAO).delete(userId);
```

**模拟方法单个返回值**

```java
Mockito.doReturn(user).when(userDAO).get(userId);
Mockito.when(userDAO.get(userId)).thenReturn(user);
```

**模拟方法多个返回值**

直接列举出多个返回值：

```java
Mockito.doReturn(record0, record1, record2, null).when(recordReader).read();
Mockito.when(recordReader.read()).thenReturn(record0, record1, record2, null);
```

转化列表为多个返回值：

```java
List recordList = ...;
Mockito.doReturn(recordList.get(0), recordList.subList(1, recordList.size()).toArray()).when(recordReader).read();
Mockito.when(recordReader.read()).thenReturn(recordList.get(0), recordList.subList(1, recordList.size()).toArray());
```

**模拟方法定制返回值**

可利用 Answer 定制方法返回值：

```java
Map userMap = ...;
Mockito.doAnswer(invocation -> userMap.get(invocation.getArgument(0)))
    .when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong()))
    .thenReturn(invocation -> userMap.get(invocation.getArgument(0)));
Mockito.when(userDAO.get(Mockito.anyLong()))
    .then(invocation -> userMap.get(invocation.getArgument(0)));
```

**模拟方法抛出单个异常**

指定单个异常类型：

```java
Mockito.doThrow(PersistenceException.class).when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong())).thenThrow(PersistenceException.class);
```

指定单个异常对象：

```java
Mockito.doThrow(exception).when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong())).thenThrow(exception);
```

**模拟方法抛出多个异常**

指定多个异常类型：

```java
Mockito.doThrow(PersistenceException.class, RuntimeException.class).when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong())).thenThrow(PersistenceException.class, RuntimeException.class);
```

指定多个异常对象：

```java
Mockito.doThrow(exception1, exception2).when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong())).thenThrow(exception1, exception2);
```

**直接调用真实方法**

```java
Mockito.doCallRealMethod().when(userService).getUser(userId);
Mockito.when(userService.getUser(userId)).thenCallRealMethod();
```

### （二）根据参数模拟方法

Mockito 提供 do-when 语句和 when-then 语句模拟方法。

**模拟无参数方法**

对于无参数的方法模拟：

```java
Mockito.doReturn(deleteCount).when(userDAO).deleteAll();
Mockito.when(userDAO.deleteAll()).thenReturn(deleteCount);
```

**模拟指定参数方法**

对于指定参数的方法模拟：

```java
Mockito.doReturn(user).when(userDAO).get(userId);
Mockito.when(userDAO.get(userId)).thenReturn(user);
```

**模拟任意参数方法**

在编写单元测试用例时，有时候并不关心传入参数的具体值，可以使用Mockito参数匹配器的any方法。Mockito提供了anyInt、anyLong、anyString、anyList、anySet、anyMap、any(Class clazz)等方法来表示任意值。

```java
Mockito.doReturn(user).when(userDAO).get(Mockito.anyLong());
Mockito.when(userDAO.get(Mockito.anyLong())).thenReturn(user);
```

**模拟可空参数方法**

Mockito参数匹配器的any具体方法，并不能够匹配null对象。而Mockito提供一个nullable方法，可以匹配包含null对象的任意对象。此外，Mockito.any()方法也可以用来匹配可空参数。

```java
Mockito.doReturn(user).when(userDAO)
    .queryCompany(Mockito.anyLong(), Mockito.nullable(Long.class));
Mockito.when(userDAO.queryCompany(Mockito.anyLong(), Mockito.<Long>any()))
    .thenReturn(user);
```

**模拟必空参数方法**

同样，如果要匹配null对象，可以使用isNull方法，或使用eq(null)。

```java
Mockito.doReturn(user).when(userDAO).queryCompany(Mockito.anyLong(), Mockito.isNull());
Mockito.when(userDAO.queryCompany(Mockito.anyLong(), Mockito.eq(null))).thenReturn(user);
```

**模拟不同参数方法**

Mockito支持按不同的参数分别模拟同一方法。

```java
Mockito.doReturn(user1).when(userDAO).get(1L);
Mockito.doReturn(user2).when(userDAO).get(2L);
...
```

> 注意：如果一个参数满足多个模拟方法条件，会以最后一个模拟方法为准。

**模拟可变参数方法**

对于一些变长度参数方法，可以按实际参数个数进行模拟：

```java
Mockito.when(userService.delete(Mockito.anyLong()).thenReturn(true);
Mockito.when(userService.delete(1L, 2L, 3L).thenReturn(true);
```

也可以用Mockito.any()模拟一个通用匹配方法：

```java
Mockito.when(userService.delete(Mockito.<Long>any()).thenReturn(true);
```

> 注意：Mockito.any()并不等于Mockito.any(Class type),前者可以匹配null和类型T的可变参数，后者只能匹配T必填参数。



### （三）模拟其它特殊方法

**模拟 final 方法**

PowerMock 提供对 final 方法的模拟，方法跟模拟普通方法一样。但是，需要把对应的模拟类添加到@PrepareForTest 注解中。

```java
// 添加@PrepareForTest注解
@PrepareForTest({UserService.class})

// 跟模拟普通方法完全一致
Mockito.doReturn(userId).when(idGenerator).next();
Mockito.when(idGenerator.next()).thenReturn(userId);
```

**模拟私有方法**

PowerMock 提供提对私有方法的模拟，但是需要把私有方法所在的类放在 @PrepareForTest 注解中。

```java
PowerMockito.doReturn(true).when(UserService.class, "isSuper", userId);
PowerMockito.when(UserService.class, "isSuper", userId).thenReturn(true);
```

**模拟构造方法**

PowerMock提供PowerMockito.whenNew方法来模拟构造方法，但是需要把使用构造方法的类放在@PrepareForTest注解中。

```java
PowerMockito.whenNew(UserDO.class).withNoArguments().thenReturn(userDO);
PowerMockito.whenNew(UserDO.class).withArguments(userId, userName).thenReturn(userDO);
```

**模拟静态方法**

PowerMock提供PowerMockito.mockStatic和PowerMockito.spy来模拟静态方法类，然后就可以模拟静态方法了。同样，需要把对应的模拟类添加到@PrepareForTest注解中。

```java
// 模拟对应的类
PowerMockito.mockStatic(HttpHelper.class);
PowerMockito.spy(HttpHelper.class);

// 模拟对应的方法
PowerMockito.when(HttpHelper.httpPost(SERVER_URL)).thenReturn(response);
PowerMockito.doReturn(response).when(HttpHelper.class, "httpPost", SERVER_URL);
PowerMockito.when(HttpHelper.class, "httpPost", SERVER_URL).thenReturn(response);
```

> 注意：第一种方式不适用于PowerMockito.spy模拟的静态方法类。



## 七、调用被测方法

在准备好参数对象后，就可以调用被测试方法了。

如果把方法按访问权限分类，可以简单地分为有访问权限和无访问权限两种。但实际上，Java语言中提供了public、protected、private和缺失共4种权限修饰符，在不同的环境下又对应不同的访问权限。具体映射关系如下：

| 修饰符    | 本类 | 本包 | 子类 | 其它 |
| --------- | ---- | ---- | ---- | ---- |
| public    | 有   | 有   | 有   | 有   |
| protected | 有   | 有   | 有   | 无   |
| 缺省      | 有   | 有   | 无   | 无   |
| private   | 有   | 无   | 无   | 无   |

下面，将根据有访问权限和无访问权限两种情况，来介绍如何调用被测方法。

### （一）调用构造方法

**调用有访问权限的构造方法**

可以直接调用有访问权限的构造方法。

```java
UserDO user = new User();
UserDO user = new User(1L, "admin");
```

**调用无访问权限的构造方法**

调用无访问权限的构造方法，可以使用 PowerMock 提供的 Whitebox.invokeConstructor 方法。

```java
Whitebox.invokeConstructor(NumberHelper.class);
Whitebox.invokeConstructor(User.class, 1L, "admin");
```

> 备注：该方法也可以调用有访问权限的构造方法，但是不建议使用。

### （二）调用普通方法

**调用有访问权限的普通方法**

可以直接调用有访问权限的普通方法。

```java
userService.deleteUser(userId);
User user = userService.getUser(userId);
```

**调用无权限访问的普通方法**

调用无访问权限的普通方法，可以使用PowerMock提供的Whitebox.invokeMethod方法。

```java
User user = (User)Whitebox.invokeMethod(userService, "isSuper", userId);
```

也可以使用 PowerMock 提供 Whitebox.getMethod 方法和 PowerMockito.method 方法，可以直接获取对应类方法对象。然后，通过 Method 的 invoke 方法，可以调用没有访问权限的方法。

```java
Method method = Whitebox.getMethod(UserService.class, "isSuper", Long.class);
Method method = PowerMockito.method(UserService.class, "isSuper", Long.class);
User user = (User)method.invoke(userService, userId);
```

> 备注：该方法也可以调用有访问权限的普通方法，但是不建议使用。



### （三）调用静态方法

**调用有权限访问的静态方法**

可以直接调用有访问权限的静态方法。

```java
boolean isPositive = NumberHelper.isPositive(-1);
```

**调用无权限访问的静态方法**

调用无权限访问的静态方法，可以使用PowerMock提供的Whitebox.invokeMethod方法。

```java
String value = (String)Whitebox.invokeMethod(JSON.class, "toJSONString", object);
```

> 备注：该方法也可以调用有访问权限的静态方法，但是不建议使用。



## 八、验证依赖方法

在单元测试中，验证是确认模拟的依赖方法是否按照预期被调用或未调用的过程。

Mockito提供了许多方法来验证依赖方法调用，给我们编写单元测试用例带来了很大的帮助。

（一）根据参数验证方法调用

**验证无参数方法调用**

```java
Mockito.verify(userDAO).deleteAll();
```

**验证指定参数方法调用**

```java
Mockito.verify(userDAO).delete(userId);
Mockito.verify(userDAO).delete(Mockito.eq(userId));
```

**验证任意参数方法调用**

```java
Mockito.verify(userDAO).delete(Mockito.anyLong());
```

**验证可空参数方法调用**

```java
Mockito.verify(userDAO).queryCompany(Mockito.anyLong(), Mockito.nullable(Long.class));
```

**验证必空参数方法调用**

```java
Mockito.verify(userDAO).queryCompany(Mockito.anyLong(), Mockito.isNull());
```

**验证可变参数方法调用**

对于一些变长度参数方法，可以按实际参数个数进行验证：

```java
Mockito.verify(userService).delete(Mockito.any(Long.class));
Mockito.verify(userService).delete(1L, 2L, 3L);
```

也可以用Mockito.any()进行通用验证：

```java
Mockito.verify(userService).delete(Mockito.<Long>any());
```

### （二）验证方法调用次数

**验证方法默认调用1次**

```java
Mockito.verify(userDAO).delete(userId);
```

**验证方法从不调用**

```java
Mockito.verify(userDAO, Mockito.never()).delete(userId);
```

**验证方法调用n次**

```java
Mockito.verify(userDAO, Mockito.times(n)).delete(userId);
```

**验证方法调用至少1次**

```java
Mockito.verify(userDAO, Mockito.atLeastOnce()).delete(userId);
```

**验证方法调用至少n次**

```java
Mockito.verify(userDAO, Mockito.atLeast(n)).delete(userId);
```

**验证方法调用最多1次**

```java
Mockito.verify(userDAO, Mockito.atMostOnce()).delete(userId);
```

**验证方法调用最多n次**

```java
Mockito.verify(userDAO, Mockito.atMost(n)).delete(userId);
```

**验证方法调用指定n次**

Mockito允许按顺序进行验证方法调用，未被验证到的方法调用将不会被标记为已验证。

```java
Mockito.verify(userDAO, Mockito.call(n)).delete(userId);
```

**验证对象及其方法调用1次**

用于验证对象及其方法调用1次，如果该对象还有别的方法被调用或者该方法调用了多次，都将导致验证方法调用失败。

```java
Mockito.verify(userDAO, Mockito.only()).delete(userId);
```

相当于：

```java
Mockito.verify(userDAO).delete(userId);
Mockito.verifyNoMoreInteractions(userDAO);
```

### （三）验证方法调用并捕获参数值

Mockito提供 ArgumentCaptor 类来捕获参数值，通过调用 `forClass(Class clazz)` 方法来构建一个 ArgumentCaptor 对象，然后在验证方法调用时来捕获参数，最后获取到捕获的参数值并验证。如果一个方法有多个参数都要捕获并验证，那就需要创建多个 ArgumentCaptor 对象。

ArgumentCaptor 的主要接口方法：

- capture 方法，用于捕获方法参数；

- getValue 方法，用于获取捕获的参数值，如果捕获了多个参数值，该方法只返回最后一个参数值；

- getAllValues 方法，用户获取捕获的所有参数值。

**使用 ArgumentCaptor.forClass 方法定义参数捕获器**

在测试用例方法中，直接使用ArgumentCaptor.forClass方法定义参数捕获器。

```java
ArgumentCaptor<UserDO> userCaptor = ArgumentCaptor.forClass(UserDO.class);
Mockito.verify(userDAO).modify(userCaptor.capture());
UserDO user = userCaptor.getValue();
```

> 注意：定义泛型类的参数捕获器时，存在强制类型转换，会引起编译器警告。

**使用@Captor注解定义参数捕获器**

也可以用Mockito提供的@Captor注解，在测试用例类中定义参数捕获器。

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @Captor
    private ArgumentCaptor<UserDO> userCaptor;
    @Test
    public void testModifyUser() {
        ...
        Mockito.verify(userDAO).modify(userCaptor.capture());
        UserDO user = userCaptor.getValue();
    }
}
```

> 注意：定义泛型类的参数捕获器时，由于是Mockito自行初始化，不会引起编译器警告。

**捕获多次方法调用的参数值列表**

```java
ArgumentCaptor<UserDO> userCaptor = ArgumentCaptor.forClass(UserDO.class);
Mockito.verify(userDAO, Mockito.atLeastOnce()).modify(userCaptor.capture());
List<UserDO> userList = userCaptor.getAllValues();
```



### （四） 验证其它特殊方法

**验证final方法调用**

final方法的验证跟普通方法类似，这里不再累述。

**验证私有方法调用**

PowerMockito提供verifyPrivate方法验证私有方法调用。

```java
PowerMockito.verifyPrivate(myClass, times(1)).invoke("unload", any(List.class));
```

**验证构造方法调用**

PowerMockito提供verifyNew方法验证构造方法调用。

```java
PowerMockito.verifyNew(MockClass.class).withNoArguments();
PowerMockito.verifyNew(MockClass.class).withArguments(someArgs);
```

**验证静态方法调用**

PowerMockito提供verifyStatic方法验证静态方法调用。

```java
PowerMockito.verifyStatic(StringUtils.class);
StringUtils.isEmpty(string);
```

## 九、验证数据对象

JUnit 测试框架中 Assert 类就是断言工具类，主要验证单元测试中实际数据对象与期望数据对象一致。在调用被测方法时，需要对返回值和异常进行验证；在验证方法调用时，也需要对捕获的参数值进行验证。

### （一）验证数据对象空值

**验证数据对象为空**

通过JUnit提供的Assert.assertNull方法验证数据对象为空。

```java
Assert.assertNull("用户标识必须为空", userId);
```

**验证数据对象非空**

通过JUnit提供的Assert.assertNotNull方法验证数据对象非空。

```java
Assert.assertNotNull("用户标识不能为空", userId);
```

### （二）验证数据对象布尔值

**验证数据对象为真**

通过JUnit提供的Assert.assertTrue方法验证数据对象为真。

```java
Assert.assertTrue("返回值必须为真", NumberHelper.isPositive(1));
```

**验证数据对象为假**

通过JUnit提供的Assert.assertFalse方法验证数据对象为假。

```java
Assert.assertFalse("返回值必须为假", NumberHelper.isPositive(-1));
```

### （三）验证数据对象引用

在单元测试用例中，对于一些参数或返回值对象，不需要验证对象具体取值，只需要验证对象引用是否一致。

**验证数据对象一致**

JUnit提供的Assert.assertSame方法验证数据对象一致。

```java
UserDO expectedUser = ...;
Mockito.doReturn(expectedUser).when(userDAO).get(userId);
UserDO actualUser = userService.getUser(userId);
Assert.assertSame("用户必须一致", expectedUser, actualUser);
```

**验证数据对象不一致**

JUnit提供的Assert.assertNotSame方法验证数据对象一致。

```java
UserDO expectedUser = ...;
Mockito.doReturn(expectedUser).when(userDAO).get(userId);
UserDO actualUser = userService.getUser(otherUserId);
Assert.assertNotSame("用户不能一致", expectedUser, actualUser);
```

### （四）验证数据对象值

JUnit提供Assert.assertEquals、Assert.assertNotEquals、Assert.assertArrayEquals方法组，可以用来验证数据对象值是否相等。

**验证简单数据对象**

对于简单数据对象（比如：基础类型、包装类型、实现了equals的数据类型……），可以直接通过JUnit的Assert.assertEquals和Assert.assertNotEquals方法组进行验证。

```java
Assert.assertNotEquals("用户名称不一致", "admin", userName);
Assert.assertEquals("账户金额不一致", 10000.0D, accountAmount, 1E-6D);
```

**验证简单数组或集合对象**

对于简单数组对象（比如：基础类型、包装类型、实现了equals的数据类型……），可以直接通过JUnit的Assert.assertArrayEquals 方法组进行验证。对于简单集合对象，也可以通过Assert.assertEquals方法验证。

```java
Long[] userIds = ...;
Assert.assertArrayEquals("用户标识列表不一致", new Long[] {1L, 2L, 3L}, userIds);

List<Long> userIdList = ...;
Assert.assertEquals("用户标识列表不一致", Arrays.asList(1L, 2L, 3L), userIdList);
```

**验证复杂数据对象**

对于复杂的JavaBean数据对象，需要验证JavaBean数据对象的每一个属性字段。

```java
UserDO user = ...;
Assert.assertEquals("用户标识不一致", Long.valueOf(1L), user.getId());
Assert.assertEquals("用户名称不一致", "admin", user.getName());
Assert.assertEquals("用户公司标识不一致", Long.valueOf(1L), user.getCompany().getId());
...
```

**验证复杂数组或集合对象**

对于复杂的JavaBean数组和集合对象，需要先展开数组和集合对象中每一个JavaBean数据对象，然后验证JavaBean数据对象的每一个属性字段。

```java
List<UserDO> expectedUserList = ...;
List<UserDO> actualUserList = ...;
Assert.assertEquals("用户列表长度不一致", expectedUserList.size(), actualUserList.size());
UserDO[] expectedUsers = expectedUserList.toArray(new UserDO[0]);
UserDO[] actualUsers = actualUserList.toArray(new UserDO[0]);
for (int i = 0; i < actualUsers.length; i++) {
    Assert.assertEquals(String.format("用户(%s)标识不一致", i), expectedUsers[i].getId(), actualUsers[i].getId());
    Assert.assertEquals(String.format("用户(%s)名称不一致", i), expectedUsers[i].getName(), actualUsers[i].getName());
Assert.assertEquals("用户公司标识不一致", expectedUsers[i].getCompany().getId(),  actualUsers[i].getCompany().getId());
    ...
}
```

**通过序列化验证数据对象**

如上一节例子所示，当数据对象过于复杂时，如果采用Assert.assertEquals依次验证每个JavaBean对象、验证每一个属性字段，测试用例的代码量将会非常庞大。这里，推荐使用序列化手段简化数据对象的验证，比如利用JSON.toJSONString方法把复杂的数据对象转化为字符串，然后再使用Assert.assertEquals方法进行验证字符串。但是，序列化值必须具备有序性、一致性和可读性。

```java
List<UserDO> userList = ...;
String text = ResourceHelper.getResourceAsString(getClass(), "userList.json");
Assert.assertEquals("用户列表不一致", text, JSON.toJSONString(userList))
```



通常使用JSON.toJSONString方法把Map对象转化为字符串，其中key-value的顺序具有不确定性，无法用于验证两个对象是否一致。这里，JSON提供序列化选项SerializerFeature.MapSortField(映射排序字段)，可以用于保证序列化后的key-value的有序性。

```java
Map<Long, Map<String, Object>> userMap = ...;
String text = ResourceHelper.getResourceAsString(getClass(), "userMap.json");
Assert.assertEquals("用户映射不一致", text, JSON.toJSONString(userMap, SerializerFeature.MapSortField));
```

**验证数据对象私有属性字段**

有时候，单元测试用例需要对复杂对象的私有属性字段进行验证。而PowerMockito提供的Whitebox.getInternalState方法，获取轻松地获取到私有属性字段值。

```java
MapperScannerConfigurer configurer = myBatisConfiguration.buildMapperScannerConfigurer();
Assert.assertEquals("基础包不一致", "com.alibaba.example", Whitebox.getInternalState(configurer, "basePackage"));
```

### （五） 验证异常对象内容

异常作为 Java 语言的重要特性，是 Java 语言健壮性的重要体现。捕获并验证异常数据内容，也是测试用例的一种。

**通过@Test注解验证异常对象**

JUnit的注解@Test提供了一个expected属性，可以指定一个期望的异常类型，用来捕获并验证异常。但是，这种方式只能验证异常类型，并不能验证异常原因和消息。

```java
@Test(expected = ExampleException.class)
public void testGetUser() {
    // 模拟依赖方法
    Mockito.doReturn(null).when(userDAO).get(userId);

    // 调用被测方法
    userService.getUser(userId);
}
```

**通过@Rule注解验证异常对象**

如果想要验证异常原因和消息，就需求采用@Rule注解定义ExpectedException对象，然后在测试方法的前面声明要捕获的异常类型、原因和消息。

```java
@Rule
private ExpectedException exception = ExpectedException.none();
@Test
public void testGetUser() {
    // 模拟依赖方法
    Long userId = 123L;
    Mockito.doReturn(null).when(userDAO).get(userId);
    // 调用被测方法
    exception.expect(ExampleException.class);
    exception.expectMessage(String.format("用户(%s)不存在", userId));
    userService.getUser(userId);
}
```

**通过Assert.assertThrows验证异常对象**

在最新版的JUnit中，提供了一个更为简洁的异常验证方式——Assert.assertThrows方法。

```java
@Test
public void testGetUser() {
    // 模拟依赖方法
    Long userId = 123L;
    Mockito.doReturn(null).when(userDAO).get(userId);
    // 调用被测方法
    ExampleException exception = Assert.assertThrows("异常类型不一致", ExampleException.class, () -> userService.getUser(userId));
    Assert.assertEquals("异常消息不一致", "处理异常", exception.getMessage());
}
```

## 十、验证依赖对象

### （一）验证模拟对象没有任何方法调用

Mockito提供了verifyNoInteractions方法，可以验证模拟对象在被测试方法中没有任何调用。

```java
Mockito.verifyNoInteractions(idGenerator, userDAO);
```

### （二）验证模拟对象没有更多方法调用

Mockito提供了verifyNoMoreInteractions方法，在验证模拟对象所有方法调用后使用，可以验证模拟对象所有方法调用是否都得到验证。如果模拟对象存在任何未验证的方法调用，就会抛出NoInteractionsWanted异常。

```
Mockito.verifyNoMoreInteractions(idGenerator, userDAO);
```

> 备注：Mockito的verifyZeroInteractions方法与verifyNoMoreInteractions方法功能相同，但是目前前者已经被废弃。

### （三）清除模拟对象所有方法调用标记



在编写单元测试用例时，为了减少单元测试用例数和代码量，可以把多组参数定义在同一个单元测试用例中，然后用for循环依次执行每一组参数的被测方法调用。为了避免上一次测试的方法调用影响下一次测试的方法调用验证，最好使用Mockito提供clearInvocations方法清除上一次的方法调用。

```java
// 清除所有对象调用
Mockito.clearInvocations();
// 清除指定对象调用
Mockito.clearInvocations(idGenerator, userDAO);
```



## 十一、典型案例及解决方案

这里，只收集了几个经典案例，解决了特定环境下的特定问题。

### （一）测试框架特性导致问题

在编写单元测试用例时，或多或少会遇到一些问题，大多数是由于对测试框架特性不熟悉导致，比如：

- Mockito不支持对静态方法、构造方法、final方法、私有方法的模拟；

- Mockito的any相关的参数匹配方法并不支持可空参数和空参数；

- 采用Mockito的参数匹配方法时，其它参数不能直接用常量或变量，必须使用Mockito的eq方法；

- 使用when-then语句模拟Spy对象方法会先执行真实方法，应该使用do-when语句；

- PowerMock对静态方法、构造方法、final方法、私有方法的模拟需要把对应的类添加到@PrepareForTest注解中；

- PowerMock模拟JDK的静态方法、构造方法、final方法、私有方法时，需要把使用这些方法的类加入到@PrepareForTest注解中，从而导致单元测试覆盖率不被统计；

- PowerMock使用自定义的类加载器来加载类，可能导致系统类加载器认为有类型转换问题；需要加上@PowerMockIgnore({“javax.crypto.*”})注解，来告诉PowerMock这个包不要用PowerMock的类加载器加载，需要采用系统类加载器来加载。

- ……



对于这些问题，可以根据提示信息查阅相关资料解决，这里就不再累述了。



### （二）捕获参数值已变更问题

在编写单元测试用例时，通常采用ArgumentCaptor进行参数捕获，然后对参数对象值进行验证。如果参数对象值没有变更，这个步骤就没有任何问题。但是，如果参数对象值在后续流程中发生变更，就会导致验证参数值失败。



**原始代码**

```java
public <T> void readData(RecordReader recordReader, int batchSize, Function<Record, T> dataParser, Predicate<List<T>> dataStorage) {
    try {
        // 依次读取数据
        Record record;
        boolean isContinue = true;
        List<T> dataList = new ArrayList<>(batchSize);
        while (Objects.nonNull(record = recordReader.read()) && isContinue) {
            // 解析添加数据
            T data = dataParser.apply(record);
            if (Objects.nonNull(data)) {
                dataList.add(data);
            }

            // 批量存储数据
            if (dataList.size() == batchSize) {
                isContinue = dataStorage.test(dataList);
                dataList.clear();
            }
        }

        // 存储剩余数据
        if (CollectionUtils.isNotEmpty(dataList)) {
            dataStorage.test(dataList);
            dataList.clear();
        }
    } catch (IOException e) {
        String message = READ_DATA_EXCEPTION;
        log.warn(message, e);
        throw new ExampleException(message, e);
    }
}
```

**测试用例**

```java
@Test
public void testReadData() throws Exception {
    // 模拟依赖方法
    // 模拟依赖方法: recordReader.read
    Record record0 = Mockito.mock(Record.class);
    Record record1 = Mockito.mock(Record.class);
    Record record2 = Mockito.mock(Record.class);
    TunnelRecordReader recordReader = Mockito.mock(TunnelRecordReader.class);
    Mockito.doReturn(record0, record1, record2, null).when(recordReader).read();
    // 模拟依赖方法: dataParser.apply
    Object object0 = new Object();
    Object object1 = new Object();
    Object object2 = new Object();
    Function<Record, Object> dataParser = Mockito.mock(Function.class);
    Mockito.doReturn(object0).when(dataParser).apply(record0);
    Mockito.doReturn(object1).when(dataParser).apply(record1);
    Mockito.doReturn(object2).when(dataParser).apply(record2);
    // 模拟依赖方法: dataStorage.test
    Predicate<List<Object>> dataStorage = Mockito.mock(Predicate.class);
    Mockito.doReturn(true).when(dataStorage).test(Mockito.anyList());

    // 调用测试方法
    odpsService.readData(recordReader, 2, dataParser, dataStorage);

    // 验证依赖方法
    // 模拟依赖方法: recordReader.read
    Mockito.verify(recordReader, Mockito.times(4)).read();
    // 模拟依赖方法: dataParser.apply
    Mockito.verify(dataParser, Mockito.times(3)).apply(Mockito.any(Record.class));
    // 验证依赖方法: dataStorage.test
    ArgumentCaptor<List<Object>> recordListCaptor = ArgumentCaptor.forClass(List.class);
    Mockito.verify(dataStorage, Mockito.times(2)).test(recordListCaptor.capture());
    Assert.assertEquals("数据列表不一致", Arrays.asList(Arrays.asList(object0, object1), Arrays.asList(object2)), recordListCaptor.getAllValues());
}
```

**问题现象**

执行单元测试用例失败，抛出以下异常信息：

```java
java.lang.AssertionError: 数据列表不一致 expected:<[[java.lang.Object@1e3469df, java.lang.Object@79499fa], [java.lang.Object@48531d5]]> but was:<[[], []]>
```

**问题原因**

由于参数dataList在调用dataStorage.test方法后，都被主动调用dataList.clear方法进行清空。由于ArgumentCaptor捕获的是对象引用，所以最后捕获到了同一个空列表。

**解决方案**

可以在模拟依赖方法dataStorage.test时，保存传入参数的当前值进行验证。代码如下：

```java
@Test
public void testReadData() throws Exception {
    // 模拟依赖方法
    ...
    // 模拟依赖方法: dataStorage.test
    List<Object> dataList = new ArrayList<>();
    Predicate<List<Object>> dataStorage = Mockito.mock(Predicate.class);
    Mockito.doAnswer(invocation -> dataList.addAll((List<Object>)invocation.getArgument(0)))
        .when(dataStorage).test(Mockito.anyList());
    // 调用测试方法
    odpsService.readData(recordReader, 2, dataParser, dataStorage);
    // 验证依赖方法
    ...
    // 验证依赖方法: dataStorage.test
    Mockito.verify(dataStorage, Mockito.times(2)).test(Mockito.anyList());
    Assert.assertEquals("数据列表不一致", Arrays.asList(object0, object1, object2), dataList);
}
```

### （三）模拟Lombok的log对象问题

Lombok的@Slf4j注解，广泛地应用于Java项目中。在某些代码分支里，可能只有log记录日志的操作，为了验证这个分支逻辑被正确执行，需要在单元测试用例中对log记录日志的操作进行验证。

**原始方法**

```java
@Slf4j
@Service
public class ExampleService {
    public void recordLog(int code) {
        if (code == 1) {
            log.info("执行分支1");
            return;
        }
        if (code == 2) {
            log.info("执行分支2");
            return;
        }
        log.info("执行默认分支");
    }
    ...
}
```

**测试用例**

```java
@RunWith(PowerMockRunner.class)
public class ExampleServiceTest {
    @Mock
    private Logger log;
    @InjectMocks
    private ExampleService exampleService;
    @Test
    public void testRecordLog1() {
        exampleService.recordLog(1);
        Mockito.verify(log).info("执行分支1");
    }
}
```

**问题现象**

执行单元测试用例失败，抛出以下异常信息：

```
Wanted but not invoked:logger.info("执行分支1");
```

**原因分析**

经过调式跟踪，发现ExampleService中的log对象并没有被注入。通过编译发现，Lombok的@Slf4j注解在ExampleService类中生成了一个静态常量log，而@InjectMocks注解并不支持静态常量的注入。

**解决方案**

采用作者实现的FieldHelper.setStaticFinalField方法，可以实现对静态常量的注入模拟对象。

```java

@RunWith(PowerMockRunner.class)
public class ExampleServiceTest {
    @Mock
    private Logger log;
    @InjectMocks
    private ExampleService exampleService;
    @Before
    public void beforeTest() throws Exception {
        FieldHelper.setStaticFinalField(ExampleService.class, "log", log);
    }
    @Test
    public void testRecordLog1() {
        exampleService.recordLog(1);
        Mockito.verify(log).info("执行分支1");
    }
}
```



### （四）兼容Pandora等容器问题

阿里巴巴的很多中间件，都是基于Pandora容器的，在编写单元测试用例时，可能会遇到一些坑。

**原始方法**

```java

@Slf4j
public class MetaqMessageSender {
    @Autowired
    private MetaProducer metaProducer;
    public String sendMetaqMessage(String topicName, String tagName, String messageKey, String messageBody) {
        try {
            // 组装消息内容
            Message message = new Message();
            message.setTopic(topicName);
            message.setTags(tagName);
            message.setKeys(messageKey);
            message.setBody(messageBody.getBytes(StandardCharsets.UTF_8));
          
            // 发送消息请求
            SendResult sendResult = metaProducer.send(message);
            if (sendResult.getSendStatus() != SendStatus.SEND_OK) {
                String msg = String.format("发送标签(%s)消息(%s)状态错误(%s)", tagName, messageKey, sendResult.getSendStatus());
                log.warn(msg);
                throw new ReconsException(msg);
            }
            log.info(String.format("发送标签(%s)消息(%s)状态成功:%s", tagName, messageKey, sendResult.getMsgId()));
            
            // 返回消息标识
            return sendResult.getMsgId();
        } catch (MQClientException | RemotingException | MQBrokerException | InterruptedException e) {
            // 记录消息异常
            Thread.currentThread().interrupt();
            String message = String.format("发送标签(%s)消息(%s)状态异常:%s", tagName, messageKey, e.getMessage());
            log.warn(message, e);
            throw new ReconsException(message, e);
        }
    }
}
```

**测试用例**

```java
@RunWith(PowerMockRunner.class)
public class MetaqMessageSenderTest {
    @Mock
    private MetaProducer metaProducer;
    @InjectMocks
    private MetaqMessageSender metaqMessageSender;
    @Test
    public void testSendMetaqMessage() throws Exception {
        // 模拟依赖方法
        SendResult sendResult = new SendResult();
        sendResult.setMsgId("msgId");
        sendResult.setSendStatus(SendStatus.SEND_OK);
        Mockito.doReturn(sendResult).when(metaProducer).send(Mockito.any(Message.class));

        // 调用测试方法
        String topicName = "topicName";
        String tagName = "tagName";
        String messageKey = "messageKey";
        String messageBody = "messageBody";
        String messageId = metaqMessageSender.sendMetaqMessage(topicName, tagName, messageKey, messageBody);
        Assert.assertEquals("messageId不一致", sendResult.getMsgId(), messageId);

        // 验证依赖方法
        ArgumentCaptor<Message> messageCaptor = ArgumentCaptor.forClass(Message.class);
        Mockito.verify(metaProducer).send(messageCaptor.capture());
        Message message = messageCaptor.getValue();
        Assert.assertEquals("topicName不一致", topicName, message.getTopic());
        Assert.assertEquals("tagName不一致", tagName, message.getTags());
        Assert.assertEquals("messageKey不一致", messageKey, message.getKeys());
        Assert.assertEquals("messageBody不一致", messageBody, new String(message.getBody()));
    }
}
```

**问题现象**

执行单元测试用例失败，抛出以下异常信息：

```
java.lang.RuntimeException: com.alibaba.rocketmq.client.producer.SendResult was loaded by org.powermock.core.classloader.javassist.JavassistMockClassLoader@5d43661b, it should be loaded by Pandora Container. Can not load this fake sdk class.
```

**原因分析**

基于Pandora容器的中间件，需要使用Pandora容器加载。在上面测试用例中，使用了PowerMock容器加载，从而导致抛出类加载异常。

**解决方案**

首先，把PowerMockRunner替换为PandoraBootRunner。其次，为了使@Mock、@InjectMocks等Mockito注解生效，需要调用MockitoAnnotations.initMocks(this)方法进行初始化。

```java
@RunWith(PandoraBootRunner.class)
public class MetaqMessageSenderTest {
    ...
    @Before
    public void beforeTest() {
        MockitoAnnotations.initMocks(this);
    }
    ...
}
```



## 十二、消除类型转换警告

在编写测试用例时，特别是泛型类型转换时，很容易产生类型转换警告。常见类型转换警告如下：

```java
Type safety: Unchecked cast from Object to List<Object>
Type safety: Unchecked invocation forClass(Class<Map>) of the generic method forClass(Class<S>) of type ArgumentCaptor
Type safety: The expression of type ArgumentCaptor needs unchecked conversion to conform to ArgumentCaptor<Map<String,Object>>
```

作为一个有代码洁癖的轻微强迫症程序员，是绝对不容许这些类型转换警告产生的。于是，总结了以下方法来解决这些类型转换警告。



### （一） 利用注解初始化

Mockito提供@Mock注解来模拟类实例，提供@Captor注解来初始化参数捕获器。由于这些注解实例是通过测试框架进行初始化的，所以不会产生类型转换警告。

**问题代码**

```java
Map<Long, String> resultMap = Mockito.mock(Map.class);
ArgumentCaptor<Map<String, Object>> parameterMapCaptor = ArgumentCaptor.forClass(Map.class);
```

**建议代码**

```java
@Mock
private Map<Long, String> resultMap;
@Captor
private ArgumentCaptor<Map<String, Object>> parameterMapCaptor;
```

### （二） 利用临时类或接口

我们无法获取泛型类或接口的class实例，但是很容易获取具体类的class实例。这个解决方案的思路是——先定义继承泛型类的具体子类，然后mock、spy、forClass以及any出这个具体子类的实例，然后把具体子类实例转换为父类泛型实例。

**问题代码**

```java
Function<Record, Object> dataParser = Mockito.mock(Function.class);
AbstractDynamicValue<Long, Integer> dynamicValue = Mockito.spy(AbstractDynamicValue.class);
ArgumentCaptor<ActionRequest<Void>> requestCaptor = ArgumentCaptor.forClass(ActionRequest.class);
```

**建议代码**

```java
/** 定义临时类或接口 */
private interface DataParser extends Function<Record, Object> {};
private static abstract class AbstractTemporaryDynamicValue extends AbstractDynamicValue<Long, Integer> {};
private static class VoidActionRequest extends ActionRequest<Void> {};

/** 使用临时类或接口 */
Function<Record, Object> dataParser = Mockito.mock(DataParser.class);
AbstractDynamicValue<Long, Integer> dynamicValue = Mockito.spy(AbstractTemporaryDynamicValue.class);
ArgumentCaptor<ActionRequest<Void>> requestCaptor = ArgumentCaptor.forClass(VoidActionRequest.class);
```

### （三） 利用CastUtils.cast方法

SpringData包中提供一个CastUtils.cast方法，可以用于类型的强制转换。这个解决方案的思路是——利用CastUtils.cast方法屏蔽类型转换警告。

**问题代码**

```java
Function<Record, Object> dataParser = Mockito.mock(Function.class);
ArgumentCaptor<ActionRequest<Void>> requestCaptor = ArgumentCaptor.forClass(ActionRequest.class);
Map<Long, Double> scoreMap = (Map<Long, Double>)method.invoke(userService);
```

**建议代码**

```java
Function<Record, Object> dataParser = CastUtils.cast(Mockito.mock(Function.class));
ArgumentCaptor<ActionRequest<Void>> requestCaptor = CastUtils.cast(ArgumentCaptor.forClass(ActionRequest.class));
Map<Long, Double> scoreMap = CastUtils.cast(method.invoke(userService));
```

这个解决方案，不需要定义注解，也不需要定义临时类或接口，能够让测试用例代码更为精简，所以作者重点推荐使用。如果不愿意引入SpringData包，也可以自己参考实现该方法，只是该方法会产生类型转换警告。

> 注意：CastUtils.cast方法本质是——先转换为Object类型，再强制转换对应类型，本身不会对类型进行校验。所以，CastUtils.cast方法好用，但是不要乱用，否则就是大坑（只有执行时才能发现问题）。



### （四）利用类型自动转换方法

在Mockito中，提供形式如下的方法——泛型类型只跟返回值有关，而跟输入参数无关。这样的方法，可以根据调用方法的参数类型自动转换，而无需手动强制类型转换。如果手动强制类型转换，反而会产生类型转换警告。

```java
<T> T getArgument(int index);
public static <T> T any();
public static synchronized <T> T invokeMethod(Object instance, String methodToExecute, Object... arguments) throws Exception;
```

**问题代码**

```java
Mockito.doAnswer(invocation -> dataList.addAll((List<Object>)invocation.getArgument(0)))
    .when(dataStorage).test(Mockito.anyList());
Mockito.doThrow(e).when(workflow).beginToPrepare((ActionRequest<Void>)Mockito.any());
Map<Long, Double> scoreMap = (Map<Long, Double>)Whitebox.invokeMethod(userService, "getScoreMap");
```

**建议代码**

```java
Mockito.doAnswer(invocation -> dataList.addAll(invocation.getArgument(0)))
    .when(dataStorage).test(Mockito.anyList());
Mockito.doThrow(e).when(workflow).beginToPrepare(Mockito.any());
Map<Long, Double> scoreMap = Whitebox.invokeMethod(userService, "getScoreMap");
```

其实，SpringData的CastUtils.cast方法之所以这么强悍，也是采用了类型自动转化方法。

### （五）利用doReturn-when语句代替when-thenReturn语句

Mockito的when-thenReturn语句需要对返回类型强制校验，而doReturn-when语句不会对返回类型强制校验。利用这个特性，可以利用doReturn-when语句代替when-thenReturn语句解决类型转换警告。

**问题代码**

```java
List<String> valueList = Mockito.mock(List.class);
Mockito.when(listOperations.range(KEY, start, end)).thenReturn(valueList);
```

**建议代码**

```java
List<?> valueList = Mockito.mock(List.class);
Mockito.doReturn(valueList).when(listOperations).range(KEY, start, end);
```



### （六） 利用Whitebox.invokeMethod方法代替Method.invoke方法

JDK提供的Method.invoke方法返回的是Object类型，转化为具体类型时需要强制转换，会产生类型转换警告。而PowerMock提供的Whitebox.invokeMethod方法返回类型可以自动转化，不会产生类型转换警告

**问题代码**

```java
Method method = PowerMockito.method(UserService.class, "getScoreMap");
Map<Long, Double> scoreMap = (Map<Long, Double>)method.invokeMethod(userService);
```

**建议代码**

```java
Map<Long, Double> scoreMap = Whitebox.invokeMethod(userService, "getScoreMap");
```



### （七） 利用instanceof关键字

在具体类型强制转换时，建议利用instanceof关键字先判断类型，否则会产生类型转换警告。

**问题代码**

```java
JSONArray jsonArray = (JSONArray)object;
...
```

**建议代码**

```java
if (object instanceof JSONArray) {
    JSONArray jsonArray = (JSONArray)object;
    ...
}
```



### （八）利用Class.cast方法

在泛型类型强制转换时，会产生类型转换警告。可以采用泛型类的cast方法转换，从而避免产生类型转换警告。

**问题代码**

```java
public static <V> V parseValue(String text, Class<V> clazz) {
    if (Objects.equals(clazz, String.class)) {
        return (V)text;
    }
    return JSON.parseObject(text, clazz);
}
```

**建议代码**

```java
public static <V> V parseValue(String text, Class<V> clazz) {
    if (Objects.equals(clazz, String.class)) {
        return clazz.cast(text);
    }
    return JSON.parseObject(text, clazz);
}
```



### （九） 避免不必要的类型转换

有时候，没有必要进行类型转换，就尽量避免类型转换。比如：把Object类型转换为具体类型，但又把具体类型当Object类型使用，就没有必要进行类型转换。像这种情况，可以合并表达式或定义基类变量，从而避免不必要的类型转化。

**问题代码**

```java
Boolean isSupper = (Boolean)method.invokeMethod(userService, userId);
Assert.assertEquals("期望值不为真", Boolean.TRUE, isSupper);

List<UserVO> userList = (Map<Long, Double>)method.invokeMethod(userService, companyId);
Assert.assertEquals("期望值不一致", expectedJson, JSON.toJSONString(userList));
```

**建议代码**

```java
Assert.assertEquals("期望值不为真", Boolean.TRUE, method.invokeMethod(userService, userId));

Object userList = method.invokeMethod(userService, companyId);
Assert.assertEquals("期望值不一致", expectedJson, JSON.toJSONString(userList));
```

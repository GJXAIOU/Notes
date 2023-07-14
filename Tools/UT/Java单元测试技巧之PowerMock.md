## Java单元测试技巧之PowerMock

**前言**

方法论：“复杂的问题要简单化，简单的问题要深入化。”

这句话让我感触颇深，这何尝不是一套编写代码的方法——把一个复杂逻辑拆分为许多简单逻辑，然后把每一个简单逻辑进行深入实现，最后把这些简单逻辑整合为复杂逻辑，总结为八字真言即是“化繁为简，由简入繁”。

编写 Java 单元测试用例，其实就是把“复杂的问题要简单化”——即把一段复杂的代码拆解成一系列简单的单元测试用例；写好Java单元测试用例，其实就是把“简单的问题要深入化”——即学习一套方法、总结一套模式并应用到实践中。

## 一、准备环境

PowerMock 是一个扩展了其它如 EasyMock 等 mock 框架的、功能更强大的框架。PowerMock 使用一个自定义类加载器和字节码操作来模拟静态方法、构造方法、final 类和方法、私有方法、去除静态初始化器等。

### （一）引入 PowerMock 包

为了引入 PowerMock 包，需要在 pom.xml 文件中加入下列 maven 依赖：

```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>2.0.9</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>2.0.9</version>
    <scope>test</scope>
</dependency>

<!--如果集成 SpringMVC 项目，加入 JUnit 依赖-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

<!--如果集成 SpringBoot 项目，加入 JUnit 依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



### （二）一个简单的测试用例

用 List 举例，模拟一个不存在的列表，但是返回的列表大小为 100。

```java
public class ListTest {
    @Test
    public void testSize() {
        Integer expected = 100;
        List list = PowerMockito.mock(List.class);
        PowerMockito.when(list.size()).thenReturn(expected);
        Integer actual = list.size();
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

## 二、mock 语句

### （一）mock 方法

声明：`T PowerMockito.mock(Class clazz);`

用途：可以用于模拟指定类的对象实例。

当模拟非 final 类（接口、普通类、虚基类）的非 final 方法时，不必使用 @RunWith 和 @PrepareForTest 注解。当模拟 final 类或 final 方法时，必须使用 @RunWith 和 @PrepareForTest 注解。注解形如：

> @RunWith(PowerMockRunner.class)
>
> @PrepareForTest({TargetClass.class})

**模拟非final类普通方法**

```java
@Getter
@Setter
@ToString
public class Rectangle implements Sharp {
    private double width;
    private double height;
    @Override
    public double getArea() {
        return width * height;
    }
}

public class RectangleTest {
    @Test
    public void testGetArea() {
        double expectArea = 100.0D;
        Rectangle rectangle = PowerMockito.mock(Rectangle.class);
        PowerMockito.when(rectangle.getArea()).thenReturn(expectArea);
        double actualArea = rectangle.getArea();
        Assert.assertEquals("返回值不相等", expectArea, actualArea, 1E-6D);
    }
}
```

**模拟final类或final方法**

```java
@Getter
@Setter
@ToString
public final class Circle {
    private double radius;
    public double getArea() {
        return Math.PI * Math.pow(radius, 2);
    }
}

@RunWith(PowerMockRunner.class)
@PrepareForTest({Circle.class})
public class CircleTest {
    @Test
    public void testGetArea() {
        double expectArea = 3.14D;
        Circle circle = PowerMockito.mock(Circle.class);
        PowerMockito.when(circle.getArea()).thenReturn(expectArea);
        double actualArea = circle.getArea();
        Assert.assertEquals("返回值不相等", expectArea, actualArea, 1E-6D);
    }
}
```

### （二）mockStatic 方法

声明：

> PowerMockito.mockStatic(Class clazz);

用途：可以用于模拟类的静态方法，必须使用 `@RunWith` 和 `@PrepareForTest` 注解。

```java
package com.gjxaiou.ut;

import org.apache.commons.lang.StringUtils;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class StringUtilsTest {
	@Test
	public void testEmpty() {
		String abc = "abc";
		boolean expected = true;

		PowerMockito.mockStatic(StringUtils.class);
		PowerMockito.when(StringUtils.isEmpty(abc)).thenReturn(expected);
		boolean actual = StringUtils.isEmpty(abc);
		Assert.assertEquals("返回值不相等", expected, actual);
	}
}
```

### （三）spy 语句

如果一个对象，我们只希望模拟它的部分方法，而希望其它方法跟原来一样，可以使用 `PowerMockito.spy` 方法代替 `PowerMockito.mock` 方法。于是，通过 when 语句设置过的方法，调用的是模拟方法；而没有通过 when 语句设置的方法，调用的是原有方法。

#### 1.spy 类

声明：`PowerMockito.spy(Class clazz)；`

用途：用于模拟类的部分方法。

案例：

```java
public class StringUtils {
    public static boolean isNotEmpty(final CharSequence cs) {
        return !isEmpty(cs);
    }
    public static boolean isEmpty(final CharSequence cs) {
        return cs == null || cs.length() == 0;
    }
}

@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class StringUtilsTest {
    @Test
    public void testIsNotEmpty() {
        String string = null;
        boolean expected = true;
        PowerMockito.spy(StringUtils.class);
        PowerMockito.when(StringUtils.isEmpty(string)).thenReturn(!expected);
        boolean actual = StringUtils.isNotEmpty(string);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

#### 2.spy 对象

声明：`T PowerMockito.spy(T object)；`

用途：用于模拟对象的部分方法。

案例：

```java
public class StringUtils {
    public static boolean isNotEmpty(final CharSequence cs) {
        return !isEmpty(cs);
    }
    public static boolean isEmpty(final CharSequence cs) {
        return cs == null || cs.length() == 0;
    }
}

@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class StringUtilsTest {
    @Test
    public void testIsNotEmpty() {
        String string = null;
        boolean expected = true;
        PowerMockito.spy(StringUtils.class);
        PowerMockito.when(StringUtils.isEmpty(string)).thenReturn(!expected);
        boolean actual = StringUtils.isNotEmpty(string);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```



## 四、when 语句

### （一）when().thenReturn() 模式

声明：

> PowerMockito.when(mockObject.someMethod(someArgs)).thenReturn(expectedValue);
>
> PowerMockito.when(mockObject.someMethod(someArgs)).thenThrow(expectedThrowable);
>
> PowerMockito.when(mockObject.someMethod(someArgs)).thenAnswer(expectedAnswer);
>
> PowerMockito.when(mockObject.someMethod(someArgs)).thenCallRealMethod();

用途：用于模拟对象方法，先执行原始方法，再返回期望的值、异常、应答，或调用真实的方法。

**返回期望值**

```java
package com.gjxaiou.ut;

import org.junit.Assert;
import org.junit.Test;
import org.powermock.api.mockito.PowerMockito;

import java.util.List;

public class ListTest {
	@Test
	public void testGet() {
		int index = 0;
		Integer expected = 100;
		List<Integer> mockList = PowerMockito.mock(List.class);
		PowerMockito.when(mockList.get(index)).thenReturn(expected);
		Integer actual = mockList.get(index);
		Assert.assertEquals("返回值不相等", expected, actual);
	}
}
```

**返回期望异常**

```java

public class ListTest {
    @Test(expected = IndexOutOfBoundsException.class)
    public void testGet() {
        int index = -1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.when(mockList.get(index)).thenThrow(new IndexOutOfBoundsException());
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**返回期望应答**

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.when(mockList.get(index)).thenAnswer(invocation -> {
            Integer value = invocation.getArgument(0);
            return value * 100;
        });
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**调用真实方法**

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 0;
        Integer expected = 100;
        List<Integer> oldList = new ArrayList<>();
        oldList.add(expected);
        List<Integer> spylist = PowerMockito.spy(oldList);
        PowerMockito.when(spylist.get(index)).thenCallRealMethod();
        Integer actual = spylist.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

### （二）doReturn().when()模式

声明：

> PowerMockito.doReturn(expectedValue).when(mockObject).someMethod(someArgs);
>
> PowerMockito.doThrow(expectedThrowable).when(mockObject).someMethod(someArgs);
>
> PowerMockito.doAnswer(expectedAnswer).when(mockObject).someMethod(someArgs);
>
> PowerMockito.doNothing().when(mockObject).someMethod(someArgs);
>
> PowerMockito.doCallRealMethod().when(mockObject).someMethod(someArgs);

用途：用于模拟对象方法，直接返回期望的值、异常、应答，或调用真实的方法，无需执行原始方法。

注意，千万不要使用以下语法：

不要将 `when(mockObject).someMethod(someArgs)` 写成 `when(mockObject.someMethod(someArgs))`

虽然不会出现编译错误，但是在执行时会抛出 UnfinishedStubbingException 异常。

**返回期望值**

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 0;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doReturn(expected).when(mockList).get(index);
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**返回期望异常**

```java
public class ListTest {
    @Test(expected = IndexOutOfBoundsException.class)
    public void testGet() {
        int index = -1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doThrow(new IndexOutOfBoundsException()).when(mockList).get(index);
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**返回期望应答**

```java
public class ListTest {
    @Test(expected = IndexOutOfBoundsException.class)
    public void testGet() {
        int index = -1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doThrow(new IndexOutOfBoundsException()).when(mockList).get(index);
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**模拟无返回值**

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doAnswer(invocation -> {
            Integer value = invocation.getArgument(0);
            return value * 100;
        }).when(mockList).get(index);
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**调用真实方法**

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 0;
        Integer expected = 100;
        List<Integer> oldList = new ArrayList<>();
        oldList.add(expected);
        List<Integer> spylist = PowerMockito.spy(oldList);
        PowerMockito.doCallRealMethod().when(spylist).get(index);
        Integer actual = spylist.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

### （三）两种模式的主要区别

两种模式都用于模拟对象方法，在 mock 实例下使用时，基本上是没有差别的。但是，在 spy 实例下使用时，when().thenReturn() 模式会执行原方法，而 doReturn().when() 模式不会执行原方法。

**测试服务类**

```java
@Slf4j
@Service
public class UserService {
    public long getUserCount() {
        log.info("调用获取用户数量方法");
        return 0L;
    }
}
```

**使用when().thenReturn()模式**

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @Test
    public void testGetUserCount() {
        Long expected = 1000L;
        UserService userService = PowerMockito.spy(new UserService());
        PowerMockito.when(userService.getUserCount()).thenReturn(expected);
        Long actual = userService.getUserCount();
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

在测试过程中，将会打印出"调用获取用户数量方法"日志。

**使用doReturn().when()模式**

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @Test
    public void testGetUserCount() {
        Long expected = 1000L;
        UserService userService = PowerMockito.spy(new UserService());
        PowerMockito.doReturn(expected).when(userService).getUserCount();
        Long actual = userService.getUserCount();
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

在测试过程中，不会打印出"调用获取用户数量方法"日志。



### （四）whenNew 模拟构造方法

声明：

> PowerMockito.whenNew(MockClass.class).withNoArguments().thenReturn(expectedObject);
>
> PowerMockito.whenNew(MockClass.class).withArguments(someArgs).thenReturn(expectedObject);

用途：用于模拟构造方法。

案例：

```java
public final class FileUtils {
    public static boolean isFile(String fileName) {
        return new File(fileName).isFile();
    }
}

@RunWith(PowerMockRunner.class)
@PrepareForTest({FileUtils.class})
public class FileUtilsTest {
    @Test
    public void testIsFile() throws Exception {
        String fileName = "test.txt";
        File file = PowerMockito.mock(File.class);
        PowerMockito.whenNew(File.class).withArguments(fileName).thenReturn(file);
        PowerMockito.when(file.isFile()).thenReturn(true);
        Assert.assertTrue("返回值为假", FileUtils.isFile(fileName));
    }
}
```

注意：需要加上注解 `@PrepareForTest({FileUtils.class})`，否则模拟方法不生效。



## 五、参数匹配器

在执行单元测试时，有时候并不关心传入的参数的值，可以使用参数匹配器。

### （一）参数匹配器(any)

Mockito 提供 `Mockito.anyInt()`、`Mockito.anyString`、`Mockito.any(Class<T> clazz)` 等来表示任意值。

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.when(mockList.get(Mockito.anyInt())).thenReturn(expected);
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

### （二）参数匹配器(eq)

当我们使用参数匹配器时，所有参数都应使用匹配器。如果要为某一参数指定特定值时，就需要使用Mockito.eq() 方法。

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class StringUtilsTest {
    @Test
    public void testStartWith() {
        String string = "abc";
        String prefix = "b";
        boolean expected = true;
        PowerMockito.spy(StringUtils.class);
        // 第一个参数任意，第二个参数指定
        PowerMockito.when(StringUtils.startsWith(Mockito.anyString(), Mockito.eq(prefix))).thenReturn(expected);
        boolean actual = StringUtils.startsWith(string, prefix);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

### （三）附加匹配器

Mockito 的 AdditionalMatchers 类提供了一些很少使用的参数匹配器，我们可以进行参数大于(gt)、小于(lt)、大于等于(geq)、小于等于(leq)等比较操作，也可以进行参数与(and)、或(or)、非(not)等逻辑计算等。

```java
public class ListTest {
    @Test
    public void testGet() {
        int index = 1;
        Integer expected = 100;
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.when(mockList.get(AdditionalMatchers.geq(0))).thenReturn(expected);
        PowerMockito.when(mockList.get(AdditionalMatchers.lt(0))).thenThrow(new IndexOutOfBoundsException());
        Integer actual = mockList.get(index);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```



## 六、verify 语句

验证是确认在模拟过程中，被测试方法是否已按预期方式与其任何依赖方法进行了交互。

格式：

> Mockito.verify(mockObject[,times(int)]).someMethod(somgArgs);

用途：用于模拟对象方法，直接返回期望的值、异常、应答，或调用真实的方法，无需执行原始方法。

案例：

### （一）验证调用方法

```java
public class ListTest {
    @Test
    public void testGet() {
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doNothing().when(mockList).clear();
        mockList.clear();
        Mockito.verify(mockList).clear();
    }
}
```

### （二）验证调用次数

```java
public class ListTest {
    @Test
    public void testGet() {
        List<Integer> mockList = PowerMockito.mock(List.class);
        PowerMockito.doNothing().when(mockList).clear();
        mockList.clear();
        Mockito.verify(mockList, Mockito.times(1)).clear();
    }
}
```

除 times 外，Mockito 还支持 atLeastOnce、atLeast、only、atMostOnce、atMost 等次数验证器。

### （三）验证调用顺序

```java
public class ListTest {
    @Test
    public void testAdd() {
           List<Integer> mockedList = PowerMockito.mock(List.class);
        PowerMockito.doReturn(true).when(mockedList).add(Mockito.anyInt());
        mockedList.add(1);
        mockedList.add(2);
        mockedList.add(3);
        InOrder inOrder = Mockito.inOrder(mockedList);
        inOrder.verify(mockedList).add(1);
        inOrder.verify(mockedList).add(2);
        inOrder.verify(mockedList).add(3);
    }
}
```

### （四）验证调用参数

```java
public class ListTest {
    @Test
    public void testArgumentCaptor() {
        Integer[] expecteds = new Integer[] {1, 2, 3};
        List<Integer> mockedList = PowerMockito.mock(List.class);
        PowerMockito.doReturn(true).when(mockedList).add(Mockito.anyInt());
        for (Integer expected : expecteds) {
            mockedList.add(expected);
        }
        ArgumentCaptor<Integer> argumentCaptor = ArgumentCaptor.forClass(Integer.class);
        Mockito.verify(mockedList, Mockito.times(3)).add(argumentCaptor.capture());
        Integer[] actuals = argumentCaptor.getAllValues().toArray(new Integer[0]);
        Assert.assertArrayEquals("返回值不相等", expecteds, actuals);
    }
}
```

### （五）确保验证完毕

Mockito 提供 Mockito.verifyNoMoreInteractions 方法，在所有验证方法之后可以使用此方法，以确保所有调用都得到验证。如果模拟对象上存在任何未验证的调用，将会抛出 NoInteractionsWanted 异常。

```java
public class ListTest {
    @Test
    public void testVerifyNoMoreInteractions() {
        List<Integer> mockedList = PowerMockito.mock(List.class);
        Mockito.verifyNoMoreInteractions(mockedList); // 执行正常
        mockedList.isEmpty();
        Mockito.verifyNoMoreInteractions(mockedList); // 抛出异常
    }
}
```

备注：Mockito.verifyZeroInteractions 方法与 Mockito.verifyNoMoreInteractions 方法相同，但是目前已经被废弃。

### （六）验证静态方法

Mockito没有静态方法的验证方法，但是PowerMock提供这方面的支持。

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class StringUtilsTest {
    @Test
    public void testVerifyStatic() {
        PowerMockito.mockStatic(StringUtils.class);
        String expected = "abc";
        StringUtils.isEmpty(expected);
        PowerMockito.verifyStatic(StringUtils.class);
        ArgumentCaptor<String> argumentCaptor = ArgumentCaptor.forClass(String.class);
        StringUtils.isEmpty(argumentCaptor.capture());
        Assert.assertEquals("参数不相等", argumentCaptor.getValue(), expected);
    }
}
```

## 七、私有属性

### （一）ReflectionTestUtils.setField 方法

在用原生 JUnit 进行单元测试时，我们一般采用 ReflectionTestUtils.setField 方法设置私有属性值。

```java
@Service
public class UserService {
    @Value("${system.userLimit}")
    private Long userLimit;
    public Long getUserLimit() {
        return userLimit;
    }
}

public class UserServiceTest {
    @Autowired
    private UserService userService;
    @Test
    public void testGetUserLimit() {
        Long expected = 1000L;
        ReflectionTestUtils.setField(userService, "userLimit", expected);
        Long actual = userService.getUserLimit();
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

注意：在测试类中，UserService 实例是通过 @Autowired 注解加载的，如果该实例已经被动态代理，ReflectionTestUtils.setField 方法设置的是代理实例，从而导致设置不生效。

### （二）Whitebox.setInternalState 方法

使用 PowerMock 进行单元测试时，可以采用 `Whitebox.setInternalState` 方法设置私有属性值。

```java
@Service
public class UserService {
    @Value("${system.userLimit}")
    private Long userLimit;
    public Long getUserLimit() {
        return userLimit;
    }
}

@RunWith(PowerMockRunner.class)
public class UserServiceTest {
    @InjectMocks
    private UserService userService;
    @Test
    public void testGetUserLimit() {
        Long expected = 1000L;
        Whitebox.setInternalState(userService, "userLimit", expected);
        Long actual = userService.getUserLimit();
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

注意：需要加上注解 `@RunWith(PowerMockRunner.class)`。

## 八、私有方法

### （一）模拟私有方法

**通过when实现**

```java
public class UserService {
    private Long superUserId;
    public boolean isNotSuperUser(Long userId) {
        return !isSuperUser(userId);
    }
    private boolean isSuperUser(Long userId) {
        return Objects.equals(userId, superUserId);
    }
}

@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest {
    @Test
    public void testIsNotSuperUser() throws Exception {
        Long userId = 1L;
        boolean expected = false;
        UserService userService = PowerMockito.spy(new UserService());
        PowerMockito.when(userService, "isSuperUser", userId).thenReturn(!expected);
        boolean actual = userService.isNotSuperUser(userId);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

**通过 stub 实现**

通过模拟方法 stub(存根)，也可以实现模拟私有方法。但是，只能模拟整个方法的返回值，而不能模拟指定参数的返回值。

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest {
    @Test
    public void testIsNotSuperUser() throws Exception {
        Long userId = 1L;
        boolean expected = false;
        UserService userService = PowerMockito.spy(new UserService());
        PowerMockito.stub(PowerMockito.method(UserService.class, "isSuperUser", Long.class)).toReturn(!expected);
        boolean actual = userService.isNotSuperUser(userId);
        Assert.assertEquals("返回值不相等", expected, actual;
    }
}
```

### （二）测试私有方法

```java
@RunWith(PowerMockRunner.class)
public class UserServiceTest9 {
    @Test
    public void testIsSuperUser() throws Exception {
        Long userId = 1L;
        boolean expected = false;
        UserService userService = new UserService();
        Method method = PowerMockito.method(UserService.class, "isSuperUser", Long.class);
        Object actual = method.invoke(userService, userId);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

### （三）验证私有方法

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest10 {
    @Test
    public void testIsNotSuperUser() throws Exception {
        Long userId = 1L;
        boolean expected = false;
        UserService userService = PowerMockito.spy(new UserService());
        PowerMockito.when(userService, "isSuperUser", userId).thenReturn(!expected);
        boolean actual = userService.isNotSuperUser(userId);
        PowerMockito.verifyPrivate(userService).invoke("isSuperUser", userId);
        Assert.assertEquals("返回值不相等", expected, actual);
    }
}
```

这里，也可以用 Method 那套方法进行模拟和验证方法。

## 九、主要注解

PowerMock为了更好地支持SpringMVC/SpringBoot项目，提供了一系列的注解，大大地简化了测试代码。

- @RunWith注解：`@RunWith(PowerMockRunner.class)`

    指定 JUnit 使用 PowerMock 框架中的单元测试运行器。

- @PrepareForTest 注解：`@PrepareForTest({ TargetClass.class })`

    当需要模拟 final 类、final 方法或静态方法时，需要添加 @PrepareForTest 注解，并指定方法所在的类。如果需要指定多个类，在{}中添加多个类并用逗号隔开即可。

- @Mock 注解

    @Mock 注解创建了一个全部 Mock 的实例，所有属性和方法全被置空（0或者null）。

- @Spy 注解

    @Spy 注解创建了一个没有 Mock 的实例，所有成员方法都会按照原方法的逻辑执行，直到被 Mock 返回某个具体的值为止。

    注意：@Spy 注解的变量需要被初始化，否则执行时会抛出异常。

-  @InjectMocks 注解

    @InjectMocks 注解创建一个实例，这个实例可以调用真实代码的方法，其余用 @Mock 或 @Spy 注解创建的实例将被注入到用该实例中。

    ```java
    @Service
    public class UserService {
        @Autowired
        private UserDAO userDAO;
        public void modifyUser(UserVO userVO) {
            UserDO userDO = new UserDO();
            BeanUtils.copyProperties(userVO, userDO);
            userDAO.modify(userDO);
        }
    }
    
    @RunWith(PowerMockRunner.class)
    public class UserServiceTest {
        @Mock
        private UserDAO userDAO;
        @InjectMocks
        private UserService userService;
        @Test
        public void testCreateUser() {
            UserVO userVO = new UserVO();
            userVO.setId(1L);
            userVO.setName("changyi");
            userVO.setDesc("test user");
            userService.modifyUser(userVO);
            ArgumentCaptor<UserDO> argumentCaptor = ArgumentCaptor.forClass(UserDO.class);
            Mockito.verify(userDAO).modify(argumentCaptor.capture());
            UserDO userDO = argumentCaptor.getValue();
            Assert.assertNotNull("用户实例为空", userDO);
            Assert.assertEquals("用户标识不相等", userVO.getId(), userDO.getId());
            Assert.assertEquals("用户名称不相等", userVO.getName(), userDO.getName());
            Assert.assertEquals("用户描述不相等", userVO.getDesc(), userDO.getDesc());
        }
    }
    ```

- @Captor 注解

    @Captor 注解在字段级别创建参数捕获器。但是，在测试方法启动前，必须调用	`MockitoAnnotations.openMocks(this)` 进行初始化。

    ```java
    @Service
    public class UserService {
        @Autowired
        private UserDAO userDAO;
        public void modifyUser(UserVO userVO) {
            UserDO userDO = new UserDO();
            BeanUtils.copyProperties(userVO, userDO);
            userDAO.modify(userDO);
        }
    }
    
    @RunWith(PowerMockRunner.class)
    public class UserServiceTest {
        @Mock
        private UserDAO userDAO;
        @InjectMocks
        private UserService userService;
        @Captor
        private ArgumentCaptor<UserDO> argumentCaptor;
        @Before
        public void beforeTest() {
            MockitoAnnotations.openMocks(this);
        }
        @Test
        public void testCreateUser() {
            UserVO userVO = new UserVO();
            userVO.setId(1L);
            userVO.setName("changyi");
            userVO.setDesc("test user");
            userService.modifyUser(userVO);
            Mockito.verify(userDAO).modify(argumentCaptor.capture());
            UserDO userDO = argumentCaptor.getValue();
            Assert.assertNotNull("用户实例为空", userDO);
            Assert.assertEquals("用户标识不相等", userVO.getId(), userDO.getId());
            Assert.assertEquals("用户名称不相等", userVO.getName(), userDO.getName());
            Assert.assertEquals("用户描述不相等", userVO.getDesc(), userDO.getDesc());
        }
    }
    ```

- @PowerMockIgnore 注解

    为了解决使用 PowerMock 后，提示 ClassLoader 错误。



### 十、相关观点

### （一）《Java开发手册》规范

**【强制】**好的单元测试必须遵守 AIR 原则。说明：单元测试在线上运行时，感觉像空气（AIR）一样感觉不到，但在测试质量的保障上，却是非常关键的。好的单元测试宏观上来说，具有自动化、独立性、可重复执行的特点。

- A：Automatic（自动化）
- I：Independent（独立性）
- R：Repeatable（可重复）

【强制】单元测试应该是全自动执行的，并且非交互式的。测试用例通常是被定期执行的，执行过程必须完全自动化才有意义。输出结果需要人工检查的测试不是一个好的单元测试。单元测试中不准使用System.out来进行人肉验证，必须使用assert来验证。

【强制】单元测试是可以重复执行的，不能受到外界环境的影响。

说明：单元测试通常会被放到持续集成中，每次有代码check in时单元测试都会被执行。如果单测对外部环境（网络、服务、中间件等）有依赖，容易导致持续集成机制的不可用。

正例：为了不受外界环境影响，要求设计代码时就把SUT的依赖改成注入，在测试时用spring 这样的DI框架注入一个本地（内存）实现或者Mock实现。



【推荐】编写单元测试代码遵守BCDE原则，以保证被测试模块的交付质量。

- B：Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。
- C：Correct，正确的输入，并得到预期的结果。
- D：Design，与设计文档相结合，来编写单元测试。
- E：Error，强制错误信息输入（如：非法数据、异常流程、业务允许外等），并得到预期的结果。



 （二）为什么要使用 Mock？

根据网络相关资料，总结观点如下：

**Mock可以用来解除外部服务依赖，从而保证了测试用例的独立性**

现在的互联网软件系统，通常采用了分布式部署的微服务，为了单元测试某一服务而准备其它服务，存在极大的依耐性和不可行性。

**Mock可以减少全链路测试数据准备，从而提高了编写测试用例的速度**

传统的集成测试，需要准备全链路的测试数据，可能某些环节并不是你所熟悉的。最后，耗费了大量的时间和经历，并不一定得到你想要的结果。现在的单元测试，只需要模拟上游的输入数据，并验证给下游的输出数据，编写测试用例并进行测试的速度可以提高很多倍。

**Mock可以模拟一些非正常的流程，从而保证了测试用例的代码覆盖率**

根据单元测试的BCDE原则，需要进行边界值测试（Border）和强制错误信息输入（Error），这样有助于覆盖整个代码逻辑。在实际系统中，很难去构造这些边界值，也能难去触发这些错误信息。而Mock从根本上解决了这个问题：想要什么样的边界值，只需要进行Mock；想要什么样的错误信息，也只需要进行Mock。

**Mock可以不用加载项目环境配置，从而保证了测试用例的执行速度**

在进行集成测试时，我们需要加载项目的所有环境配置，启动项目依赖的所有服务接口。往往执行一个测试用例，需要几分钟乃至几十分钟。采用Mock实现的测试用例，不用加载项目环境配置，也不依赖其它服务接口，执行速度往往在几秒之内，大大地提高了单元测试的执行速度。



### （三）单元测试与集成测试的区别

在实际工作中，不少同学用集成测试代替了单元测试，或者认为集成测试就是单元测试。这里，总结为了单元测试与集成测试的区别：

**测试对象不同**

单元测试对象是实现了具体功能的程序单元，集成测试对象是概要设计规划中的模块及模块间的组合。

**测试方法不同**

单元测试中的主要方法是基于代码的白盒测试，集成测试中主要使用基于功能的黑盒测试。

**测试时间不同**

集成测试要晚于单元测试。

**测试内容不同**

单元测试主要是模块内程序的逻辑、功能、参数传递、变量引用、出错处理及需求和设计中具体要求方面的测试；而集成测试主要验证各个接口、接口之间的数据传递关系，及模块组合后能否达到预期效果。

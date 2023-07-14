# Controller 层规范

## 前言

本篇主要要介绍的就是 controller 层的处理，一个完整的后端请求由 4 部分组成：

- 接口地址（也就是 URL 地址）
- 请求方式（一般就是 get、set，当然还有 put、delete）
- 请求数据（request，有 head 跟 body）
- 响应数据（response）

本篇将解决以下 3 个问题：

- 当接收到请求时，如何优雅的校验参数
- 返回响应数据该如何统一的进行处理
- 接收到请求，处理业务逻辑时抛出了异常又该如何处理

**Controller 层参数接收（太基础了，可以跳过）**

常见的请求就分为 get 跟 post 两种：

```java
@RestController
@RequestMapping("/product/product-info")
public class ProductInfoController {

    @Autowired
    ProductInfoService productInfoService;

    @GetMapping("/findById")
    public ProductInfoQueryVo findById(Integer id) {
        ...
    }

    @PostMapping("/page")
    public IPage findPage(Page page, ProductInfoQueryVo vo) {
        ...
    }
}
```

**RestController：** 之前解释过，@RestController=@Controller+ResponseBody。

加上这个注解，springboot 就会吧这个类当成 controller 进行处理，然后把所有返回的参数放到 ResponseBody 中。

**@RequestMapping：** 请求的前缀，也就是所有该 Controller 下的请求都需要加上 /product/product-info 的前缀。

**@GetMapping("/findById")：** 标志这是一个 get 请求，并且需要通过 /findById 地址才可以访问到。

**@PostMapping("/page")：** 同理，表示是个 post 请求。
参数：至于参数部分，只需要写上 ProductInfoQueryVo，前端过来的 json 请求便会通过映射赋值到对应的对象中，例如请求这么写，productId 就会自动被映射到 vo 对应的属性当中。

```
size : 1
current : 1

productId : 1
productName : 泡脚
```

### 统一状态码

#### 返回格式

为了跟前端妹妹打好关系，我们通常需要对后端返回的数据进行包装一下，增加一下状态码，状态信息，这样前端妹妹接收到数据就可以根据不同的状态码，判断响应数据状态，是否成功是否异常进行不同的显示。

当然这让你拥有了更多跟前端妹妹的交流机会，假设我们约定了 1000 就是成功的意思。

如果你不封装，那么返回的数据是这样子的：

```
{
  "productId": 1,
  "productName": "泡脚",
  "productPrice": 100.00,
  "productDescription": "中药泡脚加按摩",
  "productStatus": 0,
}
```

经过封装以后时这样子的：

```
{
  "code": 1000,
  "msg": "请求成功",
  "data": {
    "productId": 1,
    "productName": "泡脚",
    "productPrice": 100.00,
    "productDescription": "中药泡脚加按摩",
    "productStatus": 0,
  }
}
```

#### 封装 ResultVo

这些状态码肯定都是要预先编好的，怎么编呢？写个常量 1000？还是直接写死 1000？

要这么写就真的书白读的了，写状态码当然是用枚举拉：

**①**首先先定义一个状态码的接口，所有状态码都需要实现它，有了标准才好做事：

```
public interface StatusCode {
    public int getCode();
    public String getMsg();
}
```

**②**然后去找前端妹妹，跟他约定好状态码（这可能是你们唯一的约定了）枚举类嘛，当然不能有 setter 方法了，因此我们不能在用 @Data 注解了，我们要用 @Getter。

```
@Getter
public enum ResultCode implements StatusCode{
    SUCCESS(1000, "请求成功"),
    FAILED(1001, "请求失败"),
    VALIDATE_ERROR(1002, "参数校验失败"),
    RESPONSE_PACK_ERROR(1003, "response返回包装失败");

    private int code;
    private String msg;

    ResultCode(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}
```

**③**写好枚举类，就开始写 ResultVo 包装类了，我们预设了几种默认的方法，比如成功的话就默认传入 object 就可以了，我们自动包装成 success。

```
@Data
public class ResultVo {
    // 状态码
    private int code;

    // 状态信息
    private String msg;

    // 返回对象
    private Object data;

    // 手动设置返回vo
    public ResultVo(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    // 默认返回成功状态码，数据对象
    public ResultVo(Object data) {
        this.code = ResultCode.SUCCESS.getCode();
        this.msg = ResultCode.SUCCESS.getMsg();
        this.data = data;
    }

    // 返回指定状态码，数据对象
    public ResultVo(StatusCode statusCode, Object data) {
        this.code = statusCode.getCode();
        this.msg = statusCode.getMsg();
        this.data = data;
    }

    // 只返回状态码
    public ResultVo(StatusCode statusCode) {
        this.code = statusCode.getCode();
        this.msg = statusCode.getMsg();
        this.data = null;
    }
}
```

使用，现在的返回肯定就不是 return data；这么简单了，而是需要 new ResultVo(data)；

```
@PostMapping("/findByVo")
    public ResultVo findByVo(@Validated ProductInfoVo vo) {
        ProductInfo productInfo = new ProductInfo();
        BeanUtils.copyProperties(vo, productInfo);
        return new ResultVo(productInfoService.getOne(new QueryWrapper(productInfo)));
    }
```

最后返回就会是上面带了状态码的数据了。

### 统一校验

#### 原始做法

假设有一个添加 ProductInfo 的接口，在没有统一校验时，我们需要这么做。

```
@Data
public class ProductInfoVo {
    // 商品名称
    private String productName;
    // 商品价格
    private BigDecimal productPrice;
    // 上架状态
    private Integer productStatus;
}
@PostMapping("/findByVo")
    public ProductInfo findByVo(ProductInfoVo vo) {
        if (StringUtils.isNotBlank(vo.getProductName())) {
            throw new APIException("商品名称不能为空");
        }
        if (null != vo.getProductPrice() && vo.getProductPrice().compareTo(new BigDecimal(0)) < 0) {
            throw new APIException("商品价格不能为负数");
        }
        ...

        ProductInfo productInfo = new ProductInfo();
        BeanUtils.copyProperties(vo, productInfo);
        return new ResultVo(productInfoService.getOne(new QueryWrapper(productInfo)));
    }

```

这 if 写的人都傻了，能忍吗？肯定不能忍啊。

#### @Validated 参数校验

好在有 @Validated，又是一个校验参数必备良药了。有了 @Validated 我们只需要再 vo 上面加一点小小的注解，便可以完成校验功能。

```
@Data
public class ProductInfoVo {
    @NotNull(message = "商品名称不允许为空")
    private String productName;

    @Min(value = 0, message = "商品价格不允许为负数")
    private BigDecimal productPrice;

    private Integer productStatus;
}
@PostMapping("/findByVo")
    public ProductInfo findByVo(@Validated ProductInfoVo vo) {
        ProductInfo productInfo = new ProductInfo();
        BeanUtils.copyProperties(vo, productInfo);
        return new ResultVo(productInfoService.getOne(new QueryWrapper(productInfo)));
    }
```

运行看看，如果参数不对会发生什么？

我们故意传一个价格为 -1 的参数过去：

```json
productName : 泡脚
productPrice : -1
productStatus : 1
```

```
{
  "timestamp": "2020-04-19T03:06:37.268+0000",
  "status": 400,
  "error": "Bad Request",
  "errors": [
    {
      "codes": [
        "Min.productInfoVo.productPrice",
        "Min.productPrice",
        "Min.java.math.BigDecimal",
        "Min"
      ],
      "arguments": [
        {
          "codes": [
            "productInfoVo.productPrice",
            "productPrice"
          ],
          "defaultMessage": "productPrice",
          "code": "productPrice"
        },
        0
      ],
      "defaultMessage": "商品价格不允许为负数",
      "objectName": "productInfoVo",
      "field": "productPrice",
      "rejectedValue": -1,
      "bindingFailure": false,
      "code": "Min"
    }
  ],
  "message": "Validation failed for object\u003d\u0027productInfoVo\u0027. Error count: 1",
  "trace": "org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object \u0027productInfoVo\u0027 on field \u0027productPrice\u0027: rejected value [-1]; codes [Min.productInfoVo.productPrice,Min.productPrice,Min.java.math.BigDecimal,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [productInfoVo.productPrice,productPrice]; arguments []; default message [productPrice],0]; default message [商品价格不允许为负数]\n\tat org.springframework.web.method.annotation.ModelAttributeMethodProcessor.resolveArgument(ModelAttributeMethodProcessor.java:164)\n\tat org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:121)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:167)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:134)\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:879)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:793)\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:660)\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:741)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat com.alibaba.druid.support.http.WebStatFilter.doFilter(WebStatFilter.java:124)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:373)\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1594)\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\n\tat java.base/java.lang.Thread.run(Thread.java:830)\n",
  "path": "/leilema/product/product-info/findByVo"
}
```

大功告成了吗？虽然成功校验了参数，也返回了异常，并且带上"商品价格不允许为负数"的信息。

但是你要是这样返回给前端，前端妹妹就提刀过来了，当年约定好的状态码，你个负心人说忘就忘？

用户体验小于等于 0 啊！所以我们要进行优化一下，每次出现异常的时候，自动把状态码写好，不负妹妹之约！

#### 优化异常处理

首先我们先看看校验参数抛出了什么异常：

```
Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors
```

我们看到代码抛出了 org.springframework.validation.BindException 的绑定异常，因此我们的思路就是 AOP 拦截所有 controller，然后异常的时候统一拦截起来，进行封装！完美！

玩你个头啊完美，这么呆瓜的操作 springboot 不知道吗？spring mvc 当然知道拉，所以给我们提供了一个 @RestControllerAdvice 来增强所有 @RestController，然后使用 @ExceptionHandler 注解，就可以拦截到对应的异常。

这里我们就拦截 BindException.class 就好了。最后在返回之前，我们对异常信息进行包装一下，包装成 ResultVo，当然要跟上 ResultCode.VALIDATE_ERROR 的异常状态码。

这样前端妹妹看到 VALIDATE_ERROR 的状态码，就会调用数据校验异常的弹窗提示用户哪里没填好。

```
@RestControllerAdvice
public class ControllerExceptionAdvice {

    @ExceptionHandler({BindException.class})
    public ResultVo MethodArgumentNotValidExceptionHandler(BindException e) {
        // 从异常对象中拿到ObjectError对象
        ObjectError objectError = e.getBindingResult().getAllErrors().get(0);
        return new ResultVo(ResultCode.VALIDATE_ERROR, objectError.getDefaultMessage());
    }
}
```

来看看效果，完美。1002 与前端妹妹约定好的状态码：

```
{
  "code": 1002,
  "msg": "参数校验失败",
  "data": "商品价格不允许为负数"
}
```

### 统一响应

#### 统一包装响应

再回头看一下 controller 层的返回：

```
return new ResultVo(productInfoService.getOne(new QueryWrapper(productInfo)));
```

开发小哥肯定不乐意了，谁有空天天写 new ResultVo(data) 啊，我就想返回一个实体！怎么实现我不管！

好把，那就是 AOP 拦截所有 Controller，再 @After 的时候统一帮你封装一下咯。

怕是上一次脸打的不够疼，springboot 能不知道这么个操作吗？

```
@RestControllerAdvice(basePackages = {"com.bugpool.leilema"})
public class ControllerResponseAdvice implements ResponseBodyAdvice<Object> {
    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        // response是ResultVo类型，或者注释了NotControllerResponseAdvice都不进行包装
        return !methodParameter.getParameterType().isAssignableFrom(ResultVo.class);
    }

    @Override
    public Object beforeBodyWrite(Object data, MethodParameter returnType, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest request, ServerHttpResponse response) {
        // String类型不能直接包装
        if (returnType.getGenericParameterType().equals(String.class)) {
            ObjectMapper objectMapper = new ObjectMapper();
            try {
                // 将数据包装在ResultVo里后转换为json串进行返回
                return objectMapper.writeValueAsString(new ResultVo(data));
            } catch (JsonProcessingException e) {
                throw new APIException(ResultCode.RESPONSE_PACK_ERROR, e.getMessage());
            }
        }
        // 否则直接包装成ResultVo返回
        return new ResultVo(data);
    }
}
```

①@RestControllerAdvice(basePackages = {"com.bugpool.leilema"}) 自动扫描了所有指定包下的 controller，在 Response 时进行统一处理。

②重写 supports 方法，也就是说，当返回类型已经是 ResultVo 了，那就不需要封装了，当不等与 ResultVo 时才进行调用 beforeBodyWrite 方法，跟过滤器的效果是一样的。

③最后重写我们的封装方法 beforeBodyWrite，注意除了 String 的返回值有点特殊，无法直接封装成 json，我们需要进行特殊处理，其他的直接 new ResultVo(data); 就 ok 了。

打完收工，看看效果：

```
@PostMapping("/findByVo")
    public ProductInfo findByVo(@Validated ProductInfoVo vo) {
        ProductInfo productInfo = new ProductInfo();
        BeanUtils.copyProperties(vo, productInfo);
        return productInfoService.getOne(new QueryWrapper(productInfo));
    }
```

此时就算我们返回的是 po，接收到的返回就是标准格式了，开发小哥露出了欣慰的笑容。

```
{
  "code": 1000,
  "msg": "请求成功",
  "data": {
    "productId": 1,
    "productName": "泡脚",
    "productPrice": 100.00,
    "productDescription": "中药泡脚加按摩",
    "productStatus": 0,
    ...
  }
}
```

#### NOT 统一响应

**不开启统一响应原因：** 开发小哥是开心了，可是其他系统就不开心了。举个例子：我们项目中集成了一个健康检测的功能，也就是这货。

```
@RestController
public class HealthController {
    @GetMapping("/health")
    public String health() {
        return "success";
    }
}
```

公司部署了一套校验所有系统存活状态的工具，这工具就定时发送 get 请求给我们系统：

> “兄弟，你死了吗？”
> “我没死，滚”
> “兄弟，你死了吗？”
> “我没死，滚”

是的，web 项目的本质就是复读机。一旦发送的请求没响应，就会给负责人发信息（企业微信或者短信之类的），你的系统死啦！赶紧回来排查 bug 吧！

让大家感受一下。每次看到我都射射发抖，早上 6 点！我 tm！！！！！![图片](Controller%E5%B1%82%E8%A7%84%E8%8C%83.resource/640-1672671759910-1.png)

好吧，没办法，人家是老大，人家要的返回不是：

```
{
  "code": 1000,
  "msg": "请求成功",
  "data": "success"
}
```

人家要的返回只要一个 success，人家定的标准不可能因为你一个系统改。俗话说的好，如果你改变不了环境，那你就只能我****

### **新增不进行封装注解：** 因为百分之 99 的请求还是需要包装的，只有个别不需要，写在包装的过滤器吧？又不是很好维护，那就加个注解好了。所有不需要包装的就加上这个注解。

```

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface NotControllerResponseAdvice {
}
```

然后在我们的增强过滤方法上过滤包含这个注解的方法：

```
@RestControllerAdvice(basePackages = {"com.bugpool.leilema"})
public class ControllerResponseAdvice implements ResponseBodyAdvice<Object> {
    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        // response是ResultVo类型，或者注释了NotControllerResponseAdvice都不进行包装
        return !(methodParameter.getParameterType().isAssignableFrom(ResultVo.class)
                || methodParameter.hasMethodAnnotation(NotControllerResponseAdvice.class));
    }
    ...
```

最后就在不需要包装的方法上加上注解：

```
@RestController
public class HealthController {

    @GetMapping("/health")
    @NotControllerResponseAdvice
    public String health() {
        return "success";
    }
}
```

这时候就不会自动封装了，而其他没加注解的则依旧自动包装：

![图片](Controller%E5%B1%82%E8%A7%84%E8%8C%83.resource/640-1672671759911-2.png)

### 统一异常

每个系统都会有自己的业务异常，比如库存不能小于 0 子类的，这种异常并非程序异常，而是业务操作引发的异常，我们也需要进行规范的编排业务异常状态码，并且写一个专门处理的异常类，最后通过刚刚学习过的异常拦截统一进行处理，以及打日志.

①异常状态码枚举，既然是状态码，那就肯定要实现我们的标准接口 StatusCode。

```
@Getter
public enum  AppCode implements StatusCode {

    APP_ERROR(2000, "业务异常"),
    PRICE_ERROR(2001, "价格异常");

    private int code;
    private String msg;

    AppCode(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}
```

②异常类，这里需要强调一下，code 代表 AppCode 的异常状态码，也就是 2000；msg 代表业务异常，这只是一个大类，一般前端会放到弹窗 title 上；最后 super(message); 这才是抛出的详细信息，在前端显示在弹窗体中，在 ResultVo 则保存在 data 中。

```
@Getter
public class APIException extends RuntimeException {
    private int code;
    private String msg;

    // 手动设置异常
    public APIException(StatusCode statusCode, String message) {
        // message用于用户设置抛出错误详情，例如：当前价格-5，小于0
        super(message);
        // 状态码
        this.code = statusCode.getCode();
        // 状态码配套的msg
        this.msg = statusCode.getMsg();
    }

    // 默认异常使用APP_ERROR状态码
    public APIException(String message) {
        super(message);
        this.code = AppCode.APP_ERROR.getCode();
        this.msg = AppCode.APP_ERROR.getMsg();
    }

}
```

**③**最后进行统一异常的拦截，这样无论在 service 层还是 controller 层，开发人员只管抛出 API 异常，不需要关系怎么返回给前端，更不需要关心日志的打印。

```
@RestControllerAdvice
public class ControllerExceptionAdvice {

    @ExceptionHandler({BindException.class})
    public ResultVo MethodArgumentNotValidExceptionHandler(BindException e) {
        // 从异常对象中拿到ObjectError对象
        ObjectError objectError = e.getBindingResult().getAllErrors().get(0);
        return new ResultVo(ResultCode.VALIDATE_ERROR, objectError.getDefaultMessage());
    }

    @ExceptionHandler(APIException.class)
    public ResultVo APIExceptionHandler(APIException e) {
        // log.error(e.getMessage(), e); 由于还没集成日志框架，暂且放着，写上TODO
        return new ResultVo(e.getCode(), e.getMsg(), e.getMessage());
    }
}
```

**④**最后使用，我们的代码只需要这么写。

```
if (null == orderMaster) {
            throw new APIException(AppCode.ORDER_NOT_EXIST, "订单号不存在：" + orderId);
        }
{
  "code": 2003,
  "msg": "订单不存在",
  "data": "订单号不存在：1998"
}
```

就会自动抛出 AppCode.ORDER_NOT_EXIST 状态码的响应，并且带上异常详细信息订单号不存在：xxxx。
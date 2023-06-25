# 前言

上个月我负责的系统SSO升级，对接京东ERP系统，这也让我想起了之前我做过一个单点登录的项目。想来单点登录有很多实现方案，不过最主流的还是基于CAS的方案，所以我也就分享一下我的CAS实践之路。

## 什么是单点登录

单点登录的英文名叫做：Single Sign On（简称SSO）。SSO的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。之前我做的系统，需要需要设计一套支持单点登录的鉴权认证系统，所有系统都基于一套鉴权系统进行登录，并且可以实现各个系统之间的互信和跳转。所以就采用了CAS架构。

## 什么是CAS

CAS架构的核心是需要搭建一个CAS Server，该服务独立部署，拥有独立三级域名，主要负责对用户的认证工作。他主要组成包括WEB前端提供登录页面，票据模块，认证模块。

核心票据：

a. **TGT(Ticket Grangting Ticket)**：TGT是CAS为用户签发的登录票据，有TGT就表明用户在CAS上成功登录过。用户在CAS认证成功后，会生成一个TGT对象，放入自己的缓存中(Session)，同时生成TGC以cookie的形式写入浏览器。当再次访问CAS时，会先看cookie中是否存在TGC，如果存在则通过TGC获取TGT，如果获取到了TGT则代表用户之前登录过，通过TGT及访问来源生成针对来源的ST，用户就不用再次登录，以此来实现单点登录。

 b. **TGC(Ticket-granting cookie)**：TGC就是TGT的唯一标识，以cookie的形式存在在CAS Server三级域名下，是CAS Server 用来明确用户身份的凭证。

c. **ST(Service Ticket)**：ST是CAS为用户签发的访问某一客户端的服务票据。用户访问service时，service发现用户没有ST，就会重定向到 CAS Server 去获取ST。CAS Server 接收到请求后，会先看cookie中是否存在TGC，如果存在则通过TGC获取TGT，如果获取到了TGT则代表用户之前登录过，通过TGT及访问来源生成针对来源的ST。用户凭借ST去访问service，service拿ST 去CAS Server 上进行验证，验证通过service生成用户session，并返回资源。

﻿

## 基于CAS的系统实践方案

### 1. 业务背景

在我负责的项目系统中，后台业务采用的是微服务架构，有统一的业务网关，所以基于统一的业务网关，整合客户其他系统登录鉴权流程。具体业务架构图如下：

![img](jingdong_%E5%9F%BA%E4%BA%8ECAS%E7%9A%84%E5%8D%95%E7%82%B9%E7%99%BB%E5%BD%95%E5%AE%9E%E8%B7%B5%E4%B9%8B%E8%B7%AF.resource/link.png)

﻿﻿在此说明一下，因为登录系统的用户体系在不同的系统中，所以我在设计SSO统一登录认证的时候，把SSO系统与业务系统结构出来。而用户体系有两套，一套叫做采方用户体系，一套叫做供方用户体系。所以才会有如图所示的SSO Server服务，他本身不负责用户管理，但会通过统一标准接口的方式实现控制反转，实现对用户服务的调用。

### 2. 单点登录时序图

时序图如下：

﻿

![img](jingdong_%E5%9F%BA%E4%BA%8ECAS%E7%9A%84%E5%8D%95%E7%82%B9%E7%99%BB%E5%BD%95%E5%AE%9E%E8%B7%B5%E4%B9%8B%E8%B7%AF.resource/2023-04-06-15-52FJbrilA7Pv06V7y.png)

﻿如图所示，时序图标识的是两个系统通过SSO服务，实现了单点登录。

### 3. 单点登录核心接口说明

#### 3.1 sso认证跳转接口

调用说明：

由应用侧发起调用认证中心的接口。

URL地址：

```
https:// sso.com?appId=***&tenantType=1&redirectUri=***
```

请求方式：302重定向

参数说明：

appId: 对接SSO认证中心的应用唯一标识,由SSO认证中心通过线下的方式颁发给各个应用系统。

tenantType: 标记是供方登录还是采方登录。采方为1，供方为2.

RedirectUri: 应用回调地址。

#### 3.2 重定向获取临时令牌code接口

调用说明：

有认证中心发起，应用侧需实现的接口。认证中心通过302重定向，将code传给应用侧，应用侧自行发起通过临时令牌code换取accessTokenInfo。

URL地址：

```
https://应用域名?code=***
```

请求方式：GET

参数说明：

Code: 临时令牌，有效时间5min

#### 3.3 获取accessTokenInfo接口

调用说明

由应用侧发起调用认证中心的接口。通过该接口可以获取accessTokenInfo信息，然后系统自行生成本系统session信息。

URL地址:

```
https://sso.com/api/token/create?grantType=authorization_code&appId=yuncai&code=***
```

请求方式：GET

参数说明：

appId: 对接SSO认证中心的应用唯一标识,由SSO认证中心通过线下的方式颁发给各个应用系统。

code: 临时令牌,需加密

加密规则如下:*

1.Code先进行base64加密

2.用认证中心给的privateKey进行加密（RSA加密）。

3.加密后进行URLCode转码。

返回参数：

```
{
  “accessToken”:  “****”,  //token令牌
  “expiresIn”: 7200,        //过期时间
  “user”: {
    “username”: “zhangsan”,
       “fullName”: “张三”,
      “userId”: “1212”,
      “phone”: “13100000000”,
      “email”: zhangsan@test.com,
      “tenantId”: “S2131123”,
      “tenantType”: 1
  }
}
```

#### 3.4 刷新Token接口

调用说明：

由应用侧发起调用认证中心的接口。当token快到失效期时，通过该接口可以刷新accessTokenInfo信息，然后系统自行生成本系统session信息。

URL地址:

```
https://sso.com/api/token/refresh?appId=yuncai&accessToken=***
```

请求方式：GET

参数说明：

appId: 对接SSO认证中心的应用唯一标识,由SSO认证中心通过线下的方式颁发给各个应用系统。

accessToken: 需要刷新的token值。

### 4. 单点登出逻辑

有单点登录，也会有单点登出，这样才会形成业务闭环，对于单点登出逻辑，基本类似登录的逆操作，时序图如下：

﻿

![img](jingdong_%E5%9F%BA%E4%BA%8ECAS%E7%9A%84%E5%8D%95%E7%82%B9%E7%99%BB%E5%BD%95%E5%AE%9E%E8%B7%B5%E4%B9%8B%E8%B7%AF.resource/2023-04-06-15-538jpxQsySVdDJWE9.png)

## 5. 单点登出核心接口说明

### 5.1 登出sso认证中心跳转接口

调用说明：

由应用侧发起调用认证中心的接口。

URL地址：

```
https://sso.com/logout?redirectUri=***
```

请求方式：GET

参数说明

RedirectUri: 应用回调地址。

### 5.2 应用系统退出接口

调用说明

有认证中心发起，应用侧需实现的接口。通过该接口触发个应用系统清除缓存和session相关信息，实现系统登出。

URL地址：

```
https://应用系统域名/ssoLogout
```

请求方式：GET

```
 header: logoutRequest:=accessToken
```

## 总结

对于CAS这种单点登录的架构，他是非常依赖于cookie的安全性的。所以CAS的安全性也在一定程度上取决于cookie的安全性，所有增强cookie安全性的措施，对于增强CAS都是有效的。

最后提一句，一定要使用HTTPS协议哦。
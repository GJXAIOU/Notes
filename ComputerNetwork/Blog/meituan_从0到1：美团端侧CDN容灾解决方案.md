# 从0到1：美团端侧CDN容灾解决方案

原创 魏磊 心澎 陈彤 [美团技术团队](javascript:void(0);) *2022-01-13 19:58*

收录于合集

\#前端22个

\#美团外卖20个

\#CDN容灾1个

\#CDN1个

\#SRE1个

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640.png)

**总第485****篇**

**2022年 第002篇**

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453681-1.png)

CDN已经成为互联网重要的基建之一，越来越多的网络服务离不开CDN，它的稳定性也直接影响到业务的可用性。CDN的容灾一直由美团的SRE团队在负责，在端侧鲜有方案和实践。

本文结合美团外卖业务中的具体实践，介绍了一种在端侧感知CDN可用性状况并进行自动容灾切换的方案，通过该方案可有效降低业务对CDN异常的敏感，提高业务的可用性，同时降低CDN运维压力。希望本方案能够对被CDN问题所困扰的同学有所帮助或者启发。

- \1. 前言

- \2. 背景

- \3. 目标与场景

- - 3.1 核心目标
  - 3.2 适用场景

- \4. Phoenix 方案

- - 4.1 总体设计
  - 4.2 容灾流程设计
  - 4.3 实现原理

- \5. 总结与展望

## 1. 前言

作为业务研发，你是否遇到过因为 CDN 问题导致的业务图片加载失败，页面打开缓慢，页面布局错乱或者页面白屏？你是否又遇到过某些区域 CDN 域名异常导致业务停摆，客诉不断，此时的你一脸茫然，不知所措？作为 CDN 运维，你是否常常被业务方反馈的各种 CDN 问题搞得焦头烂额，一边顶着各种催促和压力寻求解决方案，一边抱怨着服务商的不靠谱？今天，我们主要介绍一下美团外卖技术团队端侧 CDN 的容灾方案，经过实践，我们发现该产品能有效减少运维及业务开发同学的焦虑，希望我们的这些经验也能够帮助到更多的技术团队。

## 2. 背景

CDN 因能够有效解决因分布、带宽、服务器性能带来的网络访问延迟等问题，已经成为互联网不可或缺的一部分，也是前端业务严重依赖的服务之一。在实际业务生产中，我们通常会将大量的静态资源如 JS 脚本、CSS 资源、图片、视频、音频等托管至 CDN 服务，以享受其边缘节点缓存对静态资源的加速。但是在享用 CDN 服务带来更好体验的同时，也经常会被 CDN 故障所影响。比如因 CDN 边缘节点异常，CDN 域名封禁等导致页面白屏、排版错乱、图片加载失败。

每一次的 CDN 故障，业务方往往束手无策，只能寄希望于 CDN 团队。而 CDN 的监控与问题排查，对 SRE 也是巨大的难题和挑战。一方面，由于 CDN 节点的分布广泛，边缘节点的监控就异常困难。另一方面，各业务汇聚得到的 CDN 监控大盘，极大程度上隐匿了细节。小流量业务、定点区域的 CDN 异常往往会被淹没。SRE 团队也做了很多努力，设计了多种方案来降低 CDN 异常对业务的影响，也取得了一定的效果，但始终有几个问题无法很好解决：

- **时效性**：当 CDN 出现问题时，SRE 会手动进行 CDN 切换，因为需要人为操作，响应时长就很难保证。另外，切换后故障恢复时间也无法准确保障。
- **有效性**：切换至备份 CDN 后，备份 CDN 的可用性无法验证，另外因为 Local DNS 缓存，无法解决域名劫持和跨网访问等问题。
- **精准性**：CDN 的切换都是大范围的变更，无法针对某一区域或者某一项目单独进行。
- **风险性**：切换至备份 CDN 之后可能会导致回源，流量剧增拖垮源站，从而引发更大的风险。

当前，美团外卖业务每天服务上亿人次，即使再小的问题在巨大的流量面前，也会被放大成大问题。外卖的动态化架构，70%的业务资源都依赖于 CDN，所以 CDN 的可用性严重影响着外卖业务。如何更有效的进行 CDN 容灾，降低 CDN 异常对业务的影响，是我们不断思考的问题。

既然以上问题 SRE 侧无法完美地解决，端侧是不是可以进行一些尝试呢？比如将 CDN 容灾前置到终端侧。不死鸟（Phoenix） 就是在这样的设想下，通过前端能力建设，不断实践和完善的一套端侧 CDN 容灾方案。该方案不仅能够有效降低 CDN 异常对业务的影响，还能提高 CDN 资源加载成功率，现已服务整个美团多个业务和 App。

## 3. 目标与场景

### 3.1 核心目标

为降低 CDN 异常对业务的影响，提高业务可用性，同时降低 SRE 同学在 CDN 运维方面的压力，在方案设计之初，我们确定了以下目标：

- **端侧 CDN 域名自动切换**：在 CDN 异常时，端侧第一时间感知并自动切换 CDN 域名进行加载重试，减少对人为操作的依赖。
- **CDN 域名隔离**：CDN 域名与服务厂商在区域维度实现服务隔离且服务等效，保证 CDN 切换重试的有效性。
- **更精准有效的 CDN 监控**：建设更细粒度的 CDN 监控，能够按照项目维度实时监控 CDN 可用性，解决 SRE CDN 监控粒度不足，告警滞后等问题。并根据容灾监控对 CDN 容灾策略实施动态调整，减少 SRE 切换 CDN 的频率。
- **域名持续热备**：保证每个 CDN 域名的持续预热，避免流量切换时导致回源。

### 3.2 适用场景

适用所有依赖 CDN ，希望降低 CDN 异常对业务影响的端侧场景，包括 Web、SSR Web、Native 等技术场景。

## 4. Phoenix 方案

一直以来，CDN 的稳定性是由 SRE 来保障，容灾措施也一直在 SRE 侧进行，但仅仅依靠链路层面上的保障，很难处理局部问题和实现快速止损。用户终端作为业务的最终投放载体，对资源加载有着天然的独立性和敏感性。如果将 CDN 容灾前置到终端侧，无论从时效性，精准性，都是 SRE 侧无法比拟的。在端侧进行容灾，就需要感知 CDN 的可用性，然后实现端侧自动切换的能力。我们调研整个前端领域，并未发现业内在端侧 CDN 容灾方面有所实践和输出，所以整个方案的实现是从无到有的一个过程。

### 4.1 总体设计

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453681-2.png)

图 1

Phoenix 端侧 CDN 容灾方案主要由五部分组成：

- **端侧容灾 SDK**：负责端侧资源加载感知，CDN 切换重试，监控上报。
- **动态计算服务**：根据端侧 SDK 上报数据，对多组等效域名按照城市、项目、时段等维度定时轮询计算域名可用性，动态调整流量至最优 CDN。同时也是对 CDN 可用性的日常巡检。
- **容灾监控平台**：从项目维度和大盘维度提供 CDN 可用性监控和告警，为问题排查提供详细信息。
- **CDN 服务**：提供完善的 CDN 链路服务，在架构上实现域名隔离，并为业务方提供等效域名服务，保证端侧容灾的有效性。等效域名，就是能够通过相同路径访问到同一资源的域名，比如：cdn1.meituan.net/src/js/test.js 和 cdn2.meituan.net/src/js/test.js 能够返回相同内容，则 cdn1.meituan.net 和 cdn2.meituan.net 互为等效域名。
- **容灾配置平台**：对项目容灾域名进行配置管理，监控上报策略管理，并提供 CDN 流量人工干预等措施。

### 4.2 容灾流程设计

为保证各个端侧容灾效果和监控指标的一致性，我们设计了统一的容灾流程，整体流程如下：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453681-3.png)

图 2

### 4.3 实现原理

#### 4.3.1 端侧容灾 SDK

##### Web 端实现

Web 端的 CDN 资源主要是 JS、CSS 和图片，所以我们的容灾目标也聚焦于这些。在 Web 侧的容灾，我们主要实现了对静态资源，异步资源和图片资源的容灾。

**实现思路**

要实现资源的容灾，最主要的问题是感知资源加载结果。通常我们是在资源标签上面添加错误回调来捕获，图片容灾可以这样实现，但这并不适合 JS，因为它有严格的执行顺序。为了解决这一问题，我们将传统的标签加载资源的方式，换成**XHR**来实现。通过**Webpack**在工程构建阶段把同步资源进行抽离，然后通过**PhoenixLoader**来加载资源。这样就能通过网络请求返回的状态码，来感知资源加载结果。

在方案的实现上，我们将 SDK 设计成了 Webpack Plugin，主要基于以下四点考虑：

1. **通用性**：美团前端技术栈相对较多，要保证容灾 SDK 能够覆盖大部分的技术框架。
2. **易用性**：过高的接入成本会增加开发人员的工作量，不能做到对业务的有效覆盖，方案价值也就无从谈起。
3. **稳定性**：方案要保持稳定可靠，不受 CDN 可用性干扰。
4. **侵入性**：不能侵入到正常业务，要做到即插即用，保证业务的稳定性。

通过调研发现，前端有 70%的工程构建都离不开 Webpack，而 Webpack Plugin 独立配置，即插即用的特性，是实现方案的最好选择。整体方案设计如下：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453681-4.png)

图 3

当然，很多团队在做性能优化时，会采取代码分割，按需引入的方式。这部分资源在同步资源生成的过程中无法感知，但这部分资源的加载结果，也关系到业务的可用性。在对异步资源的容灾方面，我们主要是通过对 Webpack 的异步资源处理方式进行重写，使用**Phoenix Loader**接管资源加载，从而实现异步资源的容灾。整体分析过程如下图所示：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-5.png)

图 4

CSS 资源的处理与 JS 有所差别，但原理相似，只需要重写 **mini-css-extract-plugin** 的异步加载实现即可。

Web 端方案资源加载示意：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-6.png)

图 5

**容灾效果**

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-7.png)

图6 容灾大盘

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-8.jpeg)

图7 容灾案例

##### Native 端容灾

客户端的 CDN 资源主要是图片，音视频以及各种动态化方案的 bundle 资源。Native 端的容灾建设也主要围绕上述资源展开。

**实现思路**

重新请求是 Native 端 CDN 容灾方案的基本原理，根据互备 CDN 域名，由 Native 容灾基建容灾域名重新进行请求资源，整个过程发生在原始请求失败后。Native 容灾基建不会在原始请求过程中进行任何操作，避免对原始请求产生影响。原始请求失败后，Native 容灾基建代理处理失败返回，业务方仍处于等待结果状态，重请新求结束后向业务方返回最终结果。整个过程中从业务方角度来看仍只发出一次请求，收到一次结果，从而达到业务方不感知的目的。为将重新请求效率提升至最佳，必须尽可能的保证重新请求次数趋向于最小。

调研业务的关注点和技术层面使用的网络框架，结合 Phoenix 容灾方案的基本流程，在方案设计方面，我们主要考虑以下几点：

- **便捷性**：接入的便捷性是 SDK 设计时首先考虑的内容，即业务方可以用最简单的方式接入，实现资源容灾，同时也可以简单无残留拆除 SDK。
- **兼容性**：Android 侧的特殊性在于多样的网络框架，集团内包括 Retrofit 框架，okHttp 框架，okHttp3 框架及已经很少使用的 URLConnection 框架。提供的 SDK 应当与各种网络框架兼容，同时业务方在即使变更网络框架也能够以最小的成本实现容灾功能。而 iOS 侧则考虑复用一个 NSURLProtocol 去实现对请求的拦截，降低代码的冗余度，同时实现对初始化项进行统一适配。
- **扩展性**：需要在基础功能之上提供可选的高级配置来满足特殊需求，包括监控方面也要提供特殊的监控数据上报能力。

基于以上设计要点，我们将 Phoenix 划分为以下结构图，图中将整体的容灾 SDK 拆分为两部分 Phoenix-Adaptor 部分与 Phoenix-Base 部分。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-9.png)

图 8

**Phoenix-Base**

Phoenix-Base 是整个 Phoenix 容灾的核心部分，其包括容灾数据缓存，域名更换组件，容灾请求执行器（区别于原始请求执行器），监控器四个对外不可见的内部功能模块，并包含外部接入模块，提供外部接入功能。

1. **容灾数据缓存**：定期获取及更新容灾数据，其产生的数据只会被域名更换组件使用。
2. **域名更换组件**：连接容灾数据缓存，容灾请求执行器，监控器的中心节点，负责匹配原始失败 Host，过滤错误码，并向容灾请求执行器提供容灾域名，向监控器提供整个容灾过程的详细数据副本。
3. **容灾执行器**：容灾请求的真正请求者，目前采用内部 OkHttp3Client，业务方也可以自主切换至自身的执行器。
4. **监控器**：分发容灾过程的详细数据，内置数据大盘的上报，若有外部自定义的监控器，也会向自定义监控器分发数据。

**Phoenix-Adaptor**

Phoenix-Adaptor 是 Phoenix 容灾的扩展适配部分，用于兼容各种网络框架。

- **绑定器**：生成适合各个网络框架的拦截器并绑定至原始请求执行者。
- **解析器**：将网络框架的 Request 转换为 Phoenix 内部执行器的 Request，并将 Phoenix 内部执行器的 Response 解析为外部网络框架 Response，以此达到适配目的。

**容灾效果**

① **业务成功率**

以外卖图片业务为例，Android 业务成功率对比（同版本 7512，2021.01.17 未开启 Phoenix 容灾，2021.01.19 晚开启 Phoenix 容灾）。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-10.png)

图 9

iOS 业务成功率对比（同版本 7511，2021.01.17 未开启 Phoenix 容灾，2021.01.19 晚开启 Phoenix 容灾）。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-11.png)

图 10

② **风险应对**

以外卖与美团图片做为对比 ，在 CDN 服务出现异常时，接入 Phoenix 的外卖 App 和未接入的美团 App 在图片成功率方面的对比。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453682-12.png)

图 11

#### 4.3.2 动态计算服务

端侧的域名重试，会在某一域名加载资源失败后，根据容灾列表依次进行重试，直至成功或者失败。如下图所示：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-13.png)

图 12

如果域名 A 大范围异常，端侧依然会首先进行域名 A 的重试加载，这样就导致不必要的重试成本。如何让资源的首次加载更加稳定有效，如何为不同业务和地区动态提供最优的 CDN 域名列表，这就是动态计算服务的要解决的问题。

**计算原理**

动态计算服务通过域名池和项目的 Appkey 进行关联，按照不同省份、不同地级市、不同项目、不同资源等维度进行策略管理。通过获取 5 分钟内对应项目上报的资源加载结果进行**定时轮询计算**，对域名池中的域名按照地区（城市&&省份）的可用性监控。计算服务会根据域名可用性动态调整域名顺序并对结果进行输出。下图是一次完整的计算过程：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-14.png)

图 13

假设有 A、B、C 三个域名，成功率分别是 99%、98%、97.8%，流量占比分别是 90%、6%、4%。基于转移基准，进行流量转移，比如，A 和 B 成功率差值是 1，B 需要把自己 1/2 的流量转移给 A，同时 A 和 C 的成功率差值大于 1，C 也需要把自己 1/2 的流量转移给 A，同时 B 和 C 的差值是 0.2，所以 C 还需要把自己 1/4 的流量转移给 B。最终，经过计算，A 的流量占比是 95%，B 是 3.5%，C 是 1.5%。最后，经过排序和随机计算后将最终结果输出。

因为 A 的占比最大，所以 A 优先被选择；通过随机，B 和 C 也会有一定的流量；基于转移基准，可以实现流量的平稳切换。

**异常唤起**

当某个 CDN 无法正常访问的时候，该 CDN 访问流量会由计算过程切换至等效的 CDN B。如果 SRE 发现切换过慢可以进行手动干预分配流量。当少量的 A 域名成功率上升后，会重复计算过程将 A 的流量加大。直至恢复初始态。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-15.png)

图 14

**服务效果**

动态计算服务使得资源的首次加载成功率由原来的**99.7%提升至99.9%**。下图为接入动态计算后资源加载成功率与未接入加载成功率对比。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-16.png)

图 15

### 4.3.3 容灾监控

在监控层面，SRE 团队往往只关注域名、大区域、运营商等复合维度的监控指标，监控流量巨大，对于小流量业务或者小范围区域的 CDN 波动，可能就无法被监控分析识别，进而也就无法感知 CDN 边缘节点异常。容灾监控建设，主要是为了解决 SRE 团队的 CDN 监控告警滞后和监控粒度问题。监控整体设计如下：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-17.png)

图 16

**流程设计**

端侧容灾数据的上报，分别按照**项目、App、资源、域名**等维度建立监控指标，将 CDN 可用性变成项目可用性的一部分。通过计算平台对数据进行分析聚合，形成 CDN 可用性大盘，按照域名、区域、项目、时间等维度进行输出，与天网监控互通，建立分钟级别的监控告警机制，大大提升了 CDN 异常感知的灵敏性。同时，SRE 侧的天网监控，也会对动态计算服务结果产生干预。监控整体流程如下：

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-18.png)

图 17

**监控效果**

CDN 监控不仅从项目维度更加细粒度的监测 CDN 可用性，还为 CDN 异常排查提供了区域、运营商、网络状况、返回码等更丰富的信息。在监控告警方面，实现了分钟级异常告警，灵敏度也高于美团内部的监控系统。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-19.png)

图 18

### 4.3.4 CDN 服务

端侧域名切换的有效性，离不开 CDN 服务的支持。在 CDN 服务方面，在原有 SRE 侧容灾的基础上，对 CDN 服务整体做了升级，实现域名隔离，解决了单域名对应多 CDN 和多域名对应单 CDN 重试无效的弊端。

![图片](meituan_%E4%BB%8E0%E5%88%B01%EF%BC%9A%E7%BE%8E%E5%9B%A2%E7%AB%AF%E4%BE%A7CDN%E5%AE%B9%E7%81%BE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.resource/640-1672585453683-20.png)

图 19

## 5. 总结与展望

经过一年的建设与发展，Phoenix CDN 容灾方案日趋成熟，现已成为美团在 CDN 容灾方面唯一的公共服务，在多次 CDN 异常中发挥了巨大的作用。在端侧，当前该方案日均容灾资源**3000万+**，挽回用户**35万+**，覆盖外卖，酒旅，餐饮，优选，买菜等业务部门，服务200+个工程，**外卖 App**、**美团 App**、**大众点评 App**均已接入。

在 SRE 侧，实现了项目维度的分钟级精准告警，同时丰富了异常信息，大大提高了 SRE 问题排查效率。自从方案大规模落地以来，CDN 异常时鲜有手动切换操作，极大减轻了 SRE 同学的运维压力。

由于前端技术的多样性和复杂性，我们的 SDK 无法覆盖所有的技术方案，所以在接下来的建设中，我们会积极推广我们的容灾原理，公开动态计算服务，希望更多的框架和服务在我们的容灾思想上，贴合自身业务实现端侧的 CDN 容灾。另外，针对方案本身，我们会不断优化资源加载性能，完善资源验签，智能切换等能力，也欢迎对 Phoenix CDN 容灾方案有兴趣的同学，跟我们一起探讨交流。同时更欢迎加入我们，文末附招聘信息，期待你的邮件。

## 6. 作者简介

魏磊、陈彤、张群、粤俊等，均来自美团外卖平台-大前端团队，丁磊、心澎，来自美团餐饮 SaaS 团队。

---------- END ----------

**招聘信息**

美团外卖平台-大前端团队是一个开放、创新、无边界的团队，鼓励每一位同学追求自己的技术梦想。团队长期招聘Android、iOS、FE 高级/资深工程师和技术专家。欢迎感兴趣的同学投递简历至：wangxiaofei03@meituan.com（邮件标题请注明：美团外卖大前端）。

**也许你还想看**

 **|** [云端的SRE发展与实践](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651746627&idx=1&sn=53ca6e5ebbb9700abeca2c0156bef780&chksm=bd12a80e8a65211843902a75ed48f7c471fed784e50920f515af17e56001703650e70f4d6f78&scene=21#wechat_redirect)

 **|** [数据库智能运维探索与实践](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749742&idx=1&sn=b6bcf56ab7f11e765bdd4384e462bd0e&chksm=bd12a4238a652d35668eb3b77023244fe17e2fd0759337723653eb3a38bdd0888039a6c5174d&scene=21#wechat_redirect)

 **|** [美团外卖自动化业务运维系统——Alfred](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747050&idx=1&sn=08f0e9ebca4454c6a5cc4cf9794389aa&chksm=bd12aba78a6522b17c8fccdca15ebfc10ab3398a5e16d1f3f0b17cf58affebbc450df0f91f9c&scene=21#wechat_redirect)
# **导读**本文为读者介绍一种易于学习、规范明确、对开发人员友好的软件架构可视化模型——C4模型，并手把手教大家使用代码绘制出精美的C4架构图。 

> 原文:https://mp.weixin.qq.com/s/hriZBjbPsRLqWRppOuyE4A





在今年的敏捷团队建设中，我通过Suite执行器实现了一键自动化单元测试。Juint除了Suite执行器还有哪些执行器呢？由此我的Runner探索之旅开始了！

良好的软件架构图能够清晰地表达架构设计意图，帮助开发团队直观地理解系统设计方案，提高团队沟通、协作效率，识别潜在问题和瓶颈，规划和实现系统的功能扩展和优化，从而提高开发效率和质量，确保系统设计能成功实施并符合期望。



本文将给大家介绍一种易于学习、规范明确、对开发人员友好的软件架构可视化模型——C4模型，并手把手教大家使用代码绘制出精美的C4架构图。



阅读本文之后，读者画的架构图将会是这样的：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640.jpeg)

*注：该图例仅作绘图示例使用，不确保其完整性、可行性。*





**02** 

 **C4模型** 





理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。   

**2.1 C4模型整体介绍**



  

C4是软件架构可视化的一种方案。架构可视化，指的是用图例的方式，把软件架构设计准确、清晰、美观地表示出来。架构可视化不是指导开发者如何进行架构设计，而是指导开发者将架构设计表达出来，产出简洁直观的架构图。

架构可视化的方法有很多，主流的有“4+1”视图模型、C4模型。视图模型描述的是架构本身，架构确定之后，不管用什么模型去表达，本质上都应该是一样的，并没有优劣之分。

C4 模型是一种易于学习、对开发人员友好的软件架构图示方法，C4模型没有规定使用特定的图形、特定的建模语言来画图，因而使用者可以非常灵活地产出架构图。

C4模型将系统从上往下分为System Context, Containers, Components, Code四层视图，每一层都是对上一层的完善和展开，层层递进地对系统进行描述，如下图。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246105.png)

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245783.png)

图2 XXX

**2.2** **System Context diagram**



  

System Context（系统上下文）视图位于顶层，是软件系统架构图的起点，表达的是系统的全貌。System Context视图重点展示的是系统边界、系统相关的用户、其他支撑系统以及与本系统的交互。本层不涉及到具体细节（例如技术选型、协议、部署方案和其他低级细节），因此System Context可以很好地向非技术人员介绍系统。

作用：清晰地展示待构建的系统、用户以及现有的IT基础设施。

范围：待描述的核心系统以及其相关用户、支撑系统，不应该出现与核心系统无关的其他系统。例如我们要描述一个打车系统，不应该把无关联的药店系统绘制进去，并且要确保一个System Context只有一个待描述的软件系统。

主要元素：Context内待描述的软件系统。

支持元素：在范围内直接与主要元素中的软件系统有关联的人员（例如用户、参与者、角色或角色）和外部依赖系统。通常，这些外部依赖系统位于我们自己的软件系统边界之外。

目标受众：软件开发团队内外的所有人，包括技术人员和非技术人员。

推荐给大多数团队：是的。

示例：这是该网上银行系统的系统上下文图。它显示了使用它的人，以及与该系统有关系的其他软件系统。网上银行系统是将要建设的系统，银行的个人客户使用网上银行系统查看其银行账户的信息并进行支付。网上银行系统本身使用银行现有的大型机银行系统来执行此操作，并使用银行现有的电子邮件系统向客户发送电子邮件。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245895.png)

图例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246116.png)

**2.3** **Container diagram**



  

Container（容器）视图是对System Context的放大，是对System Context细节的补充。

注意这里的容器，指的不是Docker等容器中间件。Container的描述范围是一个可单独运行/可部署的单元。Container一般指的是应用以及依赖的中间件，例如服务器端 Web 应用程序、单页应用程序、桌面应用程序、移动应用程序、数据库架构、文件系统、Redis、ElasticSeach、MQ等。

Container显示了软件架构的高级形状以及系统内各容器之间的职责分工。

在Container这一层，还显示了系统的主要的技术选型以及容器间的通信和交互。

作用：展示系统整体的开发边界，体现高层次的技术选型，暴露系统内容器之间的分工交互。

范围：单个软件系统，关注的系统内部的应用构成。

主要元素：软件系统范围内的容器，例如Spring Boot打包后的应用，MySQL数据库、Redis、MQ等。

支持元素：直接使用容器的人员和外部依赖系统。

目标受众：软件开发团队内外的技术人员，包括软件架构师、开发人员和运营/支持人员。

推荐给大多数团队：是的。

注意：Container视图没有说明部署方案、集群、复制、故障转移等。部署相关的视图，会通过Deployment视图进行展示。

示例：网上银行系统（此时System Contenxt中的系统已经被展开，所以用虚线框表示）由五个容器组成：服务器端 Web 应用程序、单页应用程序、移动应用程序、服务器端 API 应用程序和数据库。

- Web 应用程序是一个 Java/Spring MVC Web 应用程序，它只提供构成单页应用程序的静态内容（HTML、CSS 和 JS）。
- 单页应用程序是在客户的网络浏览器中运行的 Angular 应用程序，是网上银行功能的前端。
- 客户也可以使用跨平台 Xamarin 移动应用程序来访问网上银行。
- 单页应用程序和移动应用程序都使用 JSON+HTTPS API，该 API 由运行在服务器上的另一个 Java/Spring MVC 应用程序提供。
- API 应用程序从关系数据库中获取用户信息。
- API 应用程序还使用专有的 XML/HTTPS 接口与现有的大型机银行系统进行通信，以获取有关银行账户的信息或进行交易。
- 如果 API 应用程序需要向客户发送电子邮件，它也会使用现有的电子邮件系统。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245877.png)

该容器图的图例如下，主要是引入了数据库、APP、浏览器的图例。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245815.png)

**2.4** **Component diagram**



  

将单个容器放大，则显示了该容器内部的组件。Component（组件）视图显示了一个容器是如何由许多“组件”组成的，每个组件是什么，它们的职责以及技术实现细节。

作用：展示了可执行的容器内部构成与分工，可直接指导开发。

范围：单个容器。

主要元素：范围内容器内的组件，通常可以是Dubbo接口、REST接口、Service、Dao等。

支持元素：直接连接到容器的人员和外部依赖系统。

目标受众：软件架构师和开发人员。

推荐给大多数团队：Component用于指导开发，当有需要时创建。

示例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246120.png)

图例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245869.png)

**2.5** **Code diagram**



  

放大组件视图，则得到出组件的Code视图（代码视图）。

Code视图一般采用 UML 类图、ER图等。Code视图是一个可选的详细级别，通常可以通过 IDE 等工具按需生成。除了最重要或最复杂的组件外，不建议将这种详细程度用于其他任何内容。

在注重敏捷开发的今天，一般不建议产出Code视图。

范围：单个组件。

主要元素：范围内组件内的代码元素（例如类、接口、对象、函数、数据库表等）。

目标受众：软件架构师和开发人员。

推荐给大多数团队：不，大多数 IDE 可以按需生成这种级别的详细信息。

**2.6** **System Landscape diagram**



  

C4 模型提供了单个软件系统的静态视图，不管是 System Context、Container、Component都是针对单个软件系统的进行描述的，但在实际中软件系统不会孤立存在。为描述所有这些软件系统如何在给定的企业、组织、部门等中与其他系统组合在一起，C4采用扩展视图System Landscape （系统景观图）。

System Landscape diagram和System Context diagram是一个级别的，都不涉及到技术实现，也不会涉及到系统内部的细节。系统景观图实际上只是一个没有特定关注的软件系统的系统上下文图（System Context diagram），系统景观图内的软件系统都可以采用C4进行深入分析。

适用范围：企业/组织/部门/等。

主要元素：与所选范围相关的人员和软件系统。

目标受众：软件开发团队内外的技术人员和非技术人员。

示例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245893.png)

图例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246151.png)

为什么有了System Context diagram还需要System Landscape diagram呢？因为Context是针对单个应用的，没有办法把完整链路表达出来，要表达完整链路，就需要使用Landscape了。

举个例子，A系统调用B，B系统调用C，作为A系统的开发团队，在绘制A系统的Context图时，只能包含A以及直接相关的B系统，没有办法把C系统画进去，因为他们不知道B与C是如何交互的；在组织架构的更高层面，管理者需要了解所有系统的完整链路，于是A、B、C各个系统团队的开发者们把自己相关的系统补充到同一个Context里，共同完成了Landscape。

A系统的Context图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245900.png)

全部系统的Landscape图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245875.png)

**2.7** **Dynamic diagram**



  

Dynamic diagram（动态图）用于展示静态模型中的元素如何在运行时协作。动态图允许图表元素自由排列，并通过带有编号的箭头以指示执行顺序。

范围：特定功能、故事、用例等。

主要元素和支持元素：按照实际需要，可以是软件系统、容器或组件。

目标受众：软件开发团队内外的技术人员和非技术人员。

示例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246188.png)

图例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245919.png)

**2.8** **Deployment diagram**



  

Deployment diagram（部署图）用于说明静态模型中的软件系统（或容器）的实例在给定环境（例如生产、测试、预发、开发等）中的部署方案。

C4的部署图基于UML 部署图，但为了突出显示容器和部署节点之间的映射会做略微的简化。

部署节点表示表示软件系统/容器实例运行的位置，类似于物理基础架构（例如物理服务器或设备）、虚拟化基础架构（例如 IaaS、PaaS、虚拟机）、容器化基础架构（例如 Docker 容器）、执行环境（例如数据库服务器、Java EE web/应用服务器、Microsoft IIS）等。部署节点可以嵌套，也可以将基础设施节点包括进去，例如 DNS 服务、负载平衡器、防火墙等。

可以在部署图中随意使用 Amazon Web Services、Azure 等提供的图标，只需确保被使用的任何图标都包含在图例中，不产生歧义。

范围：单个部署环境中的一个或多个软件系统（例如生产、暂存、开发等）。

主要元素：部署节点、软件系统实例和容器实例。
支持元素：用于部署软件系统的基础设施节点。

目标受众：软件开发团队内外的技术人员；包括软件架构师、开发人员、基础架构架构师和运营/支持人员。

示例：网上银行系统的开发环境部署图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245910.png)

图例：



![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246019.png)

网上银行的生产环境部署图

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246080.png)

图例：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245939.png)

**2.9** **C4模型规范以及Review CheckList**



  

为了确保C4模型的架构图的可读性，C4模型提供了作图规范，并且提供了CheckList供自查。

**2.9.1 C4模型规范**

- 图表
    每个图都应该有一个描述图类型和范围的标题（例如“我的软件系统的系统环境图”）。
    每个图表都应该有一个关键/图例来解释所使用的符号（例如形状、颜色、边框样式、线型、箭头等）。
    首字母缩略词和缩写词（业务/领域或技术）应为所有受众所理解，或在图表键/图例中进行解释。
- 元素
    应明确指定每个元素的类型（例如，人员、软件系统、容器或组件）。
    每个元素都应该有一个简短的描述，以提供关键职责的“一目了然”的视图。
    每个容器和组件都应该有明确指定的技术。
- 关系
    每条线都应该代表一个单向关系。
    每一行都应该被标记，标记与关系的方向和意图一致（例如依赖或数据流）。尝试尽可能具体地使用标签，最好避免使用“使用”等单个词。
    容器之间的关系（通常代表进程间通信）应该有明确标记的技术/协议。



**2.9.2 Review Checklist**

C4模型图表绘制完成后，可以通过Review Checklist 进行自查，检查是否有不规范之处。Review Checklist被制成网页，可以通过 https://c4model.com/review/ 进行访问。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246167.png)





**03** 

 

# **C4模型架构图代码绘制实战**

 





理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

**3.1 文本绘图工具选型**



​     

关于C4模型的架构图的绘制，一般有两种方式：

第一种是采用**绘图工具**，这类工具直接拖拽元素、调整样式，即可产出图片，例如draw.io、PPT等工具。绘图工具的优点是非常灵活，可以满足很多细节需求；缺点是通常调整元素的样式会比较繁琐。

第二种是采用**基于文本的绘图工具**，根据一定的语法去描述图片元素，最后根据文本自动渲染成图片，例如PlantUML。基于文本的绘图工具的优点是绘图快捷，只要根据语法写出描述文件，即可渲染出来，元素的样式已经默认调试好；缺点是样式不一定符合我们的审美，调整不方便。

本文着重讲解第二种，即基于文本的绘图工具。

基于文本的绘图工具有很多，例如：structurizr、PlantUML、mermaid，分别有自己的语法。

| 工具        | 语法     | 使用方法                                                    | 地址                                                         |
| ----------- | -------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| structurizr | DSL      | 提供Web界面渲染图片，并且可以生成C4-PlantUML和mermaid的代码 | https://structurizr.com/                                     |
| C4-PlantUML | PlantUML | VS Code插件、IntelliJ Idea插件                              | https://github.com/plantuml-stdlib/C4-PlantUML               |
| mermaid     | mermaid  | Markdown插件，提供Live Editor                               | https://mermaid.js.org/syntax/c4c.html , Mermaid Live Editor |

由于IntelliJ Idea、VS Code目前在开发者中非常普及，我们选择使用C4-PlantUML，结合VS Code和IntelliJ Idea分别进行C4模型的绘制。

VS Code环境的安装，见3.2。

IntelliJ Idea环境的安装，见3.3

**3.2** **V****S Code 下C4-PlantUML安装**



  

### **3.2.1 安装VS Code**

直接官网下载安装即可，过程略去。

### **3.2.2 安装PlantUML插件**

在VS Code的Extensions窗口中搜索PlantUML，安装PlantUML插件。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246079.png)

### **3.2.3 配置VS Code代码片段**

安装完PlantUML之后，为了提高效率，最好安装PlantUML相关的代码片段。

打开VS Code菜单，层级为Code→Preferences→User Snippets，如下图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246178.png)

在选择Snippets File Or Create Snippets弹窗中，选择New Global Snippets file，如下图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246044.png)

在接下来的弹窗中，输入Snippets file的文件名，如下图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245851.png)

使用浏览器打开以下链接，并将浏览器返回的文本内容粘贴到VS Code编辑区

> https://github.com/plantuml-stdlib/C4-PlantUML/blob/master/.vscode/C4.code-snippets

如图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246025.png)

### **3.2.4 安装Graphviz**

如果图形渲染出现问题，提示安装graphviz库，直接到graphviz官网安装即可。官网链接如下：

> https://graphviz.gitlab.io/download/

Mac系统推荐采用MacPorts安装。

**3.3** **IntelliJ Idea下C4-PlantUML安装**



  

### **3.3.1 安装Idea**

### **3.3.2 安装PlantUML Integration插件**

![图片](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1Ufukjmhq0ZwscVzJfaQia4YxjfSmia32t88jxmXBNFuAUgUxzH5Uco4chbBicxpc3htMrsf5UROtNeA/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

### **3.3.3 安装代码模版**

通过以下链接，下载IntelliJ live template。

> https://github.com/plantuml-stdlib/C4-PlantUML/blob/master/intellij/c4_live_template.zip

通过菜单路径 `File | Manage IDE Settings | Import Settings` ，选择下载的 ZIP文件， `c4_live_template.zip`，导入并重启Idea即可。

**3.4** **案例实战及C4-PlantUML语法介绍**



  

C4-PlantUML的详细语法可以到官网github项目主页（ https://github.com/plantuml-stdlib/C4-PlantUML ）去了解，在此只做简单介绍。

### **3.4.1 案例**

以某招聘APP服务端架构图（Container级）为例子进行讲解，以下是渲染出来的效果图。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640.jpeg)

以下是完整plantuml代码：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@startuml  
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml  !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml  !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
!define SPRITESURL https://raw.githubusercontent.com/rabelenda/cicon-plantuml-sprites/master/sprites    !define DEVICONS https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons  !define DEVICONS2 https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons2  !define FONTAWESOME https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/font-awesome-5  !include DEVICONS/java.puml  !include DEVICONS/mysql.puml  !include DEVICONS2/spring.puml  !include DEVICONS2/redis.puml  !include DEVICONS2/android.puml  !include DEVICONS2/apple_original.puml  
title 招聘APP架构图（Container）  
Person(P_User, "找工作的APP用户（应聘者）")  
System_Boundary(Boundary_APP, "招聘APP系统边界"){    Container(C_ANDROID, "安卓移动端", "android", "移动APP安卓端",$sprite="android")    Container(C_IOS, "iOS移动端", "iOS", "移动APP iOS端",$sprite="apple_original")    Container(C_GATEWAY, "HTTP网关", "Netty", "鉴权、协议转换",$sprite="java")    Container(C_GATEWAY_CACHE, "网关缓存", "Redis", "缓存认证凭据",$sprite="redis")    Container(C_BFF, "BFF网关", "Spring Boot","整合后端接口",$sprite="spring")    Container(C_CERT, "实名认证服务", "Spring Boot", "内部实名认证服务",$sprite="spring")    Container(C_BIZ_1, "职位服务", "Spring Boot", "发布、搜索职位",$sprite="spring")    Container(C_PAYMENT, "支付服务", "Spring Boot", "内部支付服务",$sprite="spring")    ContainerDb(CDB_MYSQL, "职位信息数据库", "MySQL", "持久化职位信息",$sprite="mysql")  }  
System_Ext(OUT_S_CERT, "实名认证服务","对用户进行姓名身份证号实名认证")  System_Ext(OUT_S_PAYMENT, "第三方支付服务","支持用户使用多种支付方式完成支付")  
Rel(P_User, C_ANDROID, "注册登陆投递简历")  Rel(P_User, C_IOS, "注册登陆投递简历")  
Rel(C_ANDROID, C_GATEWAY, "请求服务端","HTTPS")  Rel(C_IOS, C_GATEWAY, "请求服务端","HTTPS")  
Rel_L(C_GATEWAY, C_GATEWAY_CACHE, "读写缓存","jedis")  Rel(C_GATEWAY, C_BFF, "将HTTP协议转为RPC协议","RPC")  
Rel(C_GATEWAY, C_BIZ_1, "将HTTP协议转为RPC协议","RPC")  Rel(C_GATEWAY, C_PAYMENT, "将HTTP协议转为RPC协议","RPC")  
Rel(C_BFF, C_CERT, "通过BFF处理之后，对外暴露接口服务","RPC")  
Rel(C_BIZ_1, CDB_MYSQL, "读写数据","JDBC")  
Rel(C_CERT, OUT_S_CERT, "对接外部查询实名信息接口","HTTPS")  Rel(C_PAYMENT, OUT_S_PAYMENT, "对接外部支付系统","HTTPS")  
left to right direction  
SHOW_LEGEND()  
@enduml

```

### **3.4.2 PlantUML文件**

PlantUML文件以puml作为文件扩展名。

### **3.4.3 @startuml和@enduml**

整个文档由`@startuml`和`@enduml`包裹，是固定语法。

- 
- 
- 

```
@startuml
@enduml

```

### **3.4.4 注释**

PlantUML中使用单引号（即`'`）作为注释标识。

### **3.4.5 include语句**

首先是C4各个视图的include语句，以下语句代表引入了C4的Context、Container、Component视图。

- 
- 
- 

```
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml  !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml  !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

```

其次是图标库：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
!define SPRITESURL https://raw.githubusercontent.com/rabelenda/cicon-plantuml-sprites/master/sprites    !define DEVICONS https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons  !define DEVICONS2 https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons2  !define FONTAWESOME https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/font-awesome-5  !include DEVICONS/java.puml  !include DEVICONS/mysql.puml  !include DEVICONS2/spring.puml  !include DEVICONS2/redis.puml  !include DEVICONS2/android.puml  !include DEVICONS2/apple_original.puml

```

注意这里有一个define语法，先通过 !define 定义一个标识，之后使用该标识的地方都会被替换

- 
- 
- 

```
!define DEVICONS2 https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons2!include DEVICONS2/spring.puml‘ 等价于 !include https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons2/spring.puml
```

使用图标时，只需要在元素的声明语句中加入`$sprite="xxx"`即可。

- 

```
ContainerDb(CDB_MYSQL, "职位信息数据库", "MySQL", "持久化职位信息",$sprite="mysql")

```

### **3.4.6 C4模型静态元素**

Person：系统的用户，可能是人或者其他系统

System：代表即将建设的系统，通常渲染为蓝色方块。
System_Ext：代表已存在的系统，通常渲染为灰色方块。
System_Boundary：某系统展开为容器时，则将System改为System_Boundary，代表系统的边界，内部放置容器元素，通常渲染为虚线框。

Container：待建设的容器，通常渲染为蓝色方块。
Container_Ext：已建设容器，通常渲染为灰色方块。
Container_Boundary：某容器展开为组件之后，则将Container改为Container_Boundary，代表容器的边界，内部放置组件元素，通常渲染为虚线框。

ContainerDb：待建设数据库，通常渲染为蓝色圆柱。
ContainerQueue：待建设消息队列，通常渲染为水平放置的蓝色圆柱。

Component：待建设组件，通常渲染为蓝色方块。
Component_Ext：已建设组件，通常渲染为灰色方块。

静态元素的语法为：

- 

```
Container(alias, "label", "technology", "description")

```

alias：是图内元素的唯一ID，其他地方可以通过alias进行引用，比如在`Rel`中引用
label：代表元素的显示名称
technology：代表元素采用的核心技术，包括但不限于开发语言、框架、通信协议等
description：代表元素的简单描述

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

对于System_Boundary和Container_Boundary，则只需要alias和label，大括号内是该元素边界内的子元素。

- 
- 
- 

```
Container_Boundary(alias, "label"){
}

```

### 3.4.7 C4模型的关系元素

Rel代表两个元素之间的关系，其语法为：

- 

```
Rel(from_alias, to_alias, "label", "technology")

```

from_alias是起点元素的别名，to_alias是终点元素的别名，label则用来说明这个关联关系，technology代表采用的技术、通信协议。例如：

- 

```
Rel(C_IOS, C_GATEWAY, "请求服务端","HTTPS")

```

代表iOS客户端通过请求网关接口访问服务端资源，采用HTTPS的通信方式。

建议在绘制`Rel`时标注出`technology`。

### **3.4.8 C4-PlantUML布局**

C4-PlantUML提供了多种自动布局方案，我们可以根据实际需要进行选择。

- LAYOUT_TOP_DOWN()：从上往下布局，默认采用该布局。如下图：

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205246044.jpeg)

- LAYOUT_LEFT_RIGHT()：从左到右，即横向放置元素。`left to right direction`是PlantUML的语法，也可以直接用。

###  

### **3.4.9 图例**

通过`SHOW_LEGEND()`添加图例。

![图片](jingdong_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.resource/640-20230619205245903.png)





**04** 

 **总****结** 





理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

本文介绍了如何使用C4模型进行架构可视化，并展示了如何使用代码绘制架构图，限于篇幅，读者可到以下官网了解更多C4相关的知识。

C4模型：https://c4model.com/

C4-PlantUML：https://github.com/plantuml-stdlib/C4-PlantUML
# JavaEEDay47


MVC：
- model: 数据模型 ：就是数据，例如：实体类；
- view : 视图展示 ： 例如 HTML、jsp(jsp 虽然可以像 Servlet 一样操作数据，但是一般用于视图展示)
- control：业务逻辑控制：负责组装数据，并且把数据传递给前端页面，Servlet 就是做这个；

一个动态页面无非就是查询数据,把数据渲染(绑定)到前端页面

一个完成的动态页面是要由一个 servlet+jsp共同实现。
因为 servlet自己本身适合数据读取,逻辑判断,但是展示页面很复杂,
只用jsp,展示页面简单,但是我们需要在jsp中书写大量的java代码。

一般使用 Servlet 组装数据，使用 jsp 负责展示数据
**整体流程**：一个请求，请求到 Servlet，在 Servlet 中读取数据库，通过转发挑转到就 jsp 页面，我们可以通过 request 作用域，把数据存放。然后转发到 jsp，jsp 既可以直接从 request 作用域中读取数据，然后通过 jstl +el 进行数据展示。



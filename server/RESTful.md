# RESTful API理论



#### REST是什么？

* 万维网软件架构风格、设计风格
* Representational 数据表现形式(JSON、XML ...)
* State 当前状态或者数据
* Transfer 数据传输



#### REST模式的6个限制

* **客户端 - 服务端 (Client - Server)**，前后端关注点分离。
* **服务端无状态(Stateless)**，所有用户会话都保存在客户端，每次请求都由客户端提供当前的状态。
* **缓存(Cache)**，减少前后端交互，提升性能。
* **统一接口(Uniform Interface)**，所有接口统一通用，提升了简单性、可见性；接口实现节藕，前端实现独立开发。
* **分层系统(Layered System)**，隐藏服务端关系，中间层、安全层、负载均衡、缓存层。
* **按需代码(Code-On-Demand 可选)**，按需加载代码，提升客户端的简单性。



#### 统一接口

* **资源**是任何可以命名的事物，比如用户、评论等。
* 每一个资源都可以通过URI被**唯一标识**。
* **表达资源**，客户端不能直接操作服务端资源，需要通过表述来操作资源，通过JSON进行表述。
* **自描述消息**，媒体类型(application/json、application/xml)、HTTP方法(GET、POST、DELETE)、是否缓存(Cache-Control)。
* **超媒体**(带文字的链接)，作为应用状态引擎，即点击一个链接跳转到另一个页面；在接口返回的数据中，可以通过链接引导到其他的接口，用链接的形式将不同接口串联起来，就像一个网站那样。



#### RESTful API 简介

* 符合REST架构风格的API，包含基本的URI、标准的HTTP方法、传输的数据媒体类型(JSON、XML)。
* 使用GET(查询)、POST(创建)、PUT(替换)、PATCH(部分修改)、DELETE(删除)等动词，实现增删改查的操作。
* [Guthub API 示例](https://developer.github.com/v4/)



#### RESTful API 设计最佳实践

* **请求设计规范**
  * URI使用名词，尽量用复数，如/users；
  * URI使用前套表达关联关系，如/users/12/repos/5
  * 使用正确的HTTP方法，如GET、POST、PUT、DELETE
  * 不符合CRUD的情况：POST + 动词、使用查询字符串、设计成子资源
* **相应设计规范**
  * 查询
  * 分页
  * 字段过滤，排除多余的字段
  * 状态码，2xx，3xx，4xx，5xx
  * 错误处理，通用错误提示
* 安全
  * HTTPS，防止网页被篡改
  * 用户鉴权，[Jwt](Jwt.md)用户登录验证
  * 限流，防止恶意攻击
* 开发者友好
  * 文档，完善API文档
  * 超媒体，链接式导航API



#### RESTful API 常用状态码 <a id="restful_status"></a>

* 2xx (成功类别)
  * 201 Created：在创建新实例时，应返回此状态代码。例如，使用 POST 方法创建一个新的实例，应该始终返回 201 状态码。
  * 204 内容不存在：表示请求已被成功处理，但并未返回任何内容。
* 3xx (重定向类别)
  * 304 未修改：表示客户端的响应已经在其缓存中。 因此，不需要再次传送相同的数据。
* 4xx (客户端错误类别)
  * 400 错误请求：表示客户端的请求没有被处理，因为服务器不能理解客户端请求的是什么。
  * 401 未授权：表示客户端不被允许访问该资源，需要使用指定凭证重新请求。
  * 403 禁止访问：表示请求是有效的并且客户端已通过身份验证，但客户端不被允许以任何理由访问对应页面或资源。 例如有时授权的客户端不被允许访问服务器上的目录。
  * 404 未找到：表示所请求的资源现在不可用。
  * 410 资源不可用：表示所请求的资源后续不再可用，该资源已被移动。
* 5xx (服务器错误类别)
  * 500是服务器内部错误，表示请求已经被接收到了，但服务器被要求处理某些未预设的请求而完全混乱。
  * 503服务不可用表示服务器已关闭或无法接收和处理请求。大多数情况是服务器正在进行维护。



#### 参考文章

* [如何使用koa2+es6/7打造高质量Restful API](https://www.jianshu.com/p/f59594b90500)
* 
# RESTful API理论



#### REST是什么？

* 万维网软件架构风格、设计风格
* Representational 数据表现形式(Json、Xml...)
* State 当前状态或者数据
* Transfer 数据传输



#### 6个限制

* **客户端 - 服务端 (Client - Server)**，前后端关注点分离。
* **服务端无状态(Stateless)**，所有用户会话都保存在客户端，每次请求都由客户端提供当前的状态。
* **缓存(Cache)**，减少前后端交互，提升性能。
* **统一接口(Uniform Interface)**，所有接口统一通用，提升了简单性、可见性；接口实现节藕，前端实现独立开发。
* **分层系统(Layered System)**，隐藏服务端关系，中间层、安全层、负载均衡、缓存层。
* **按需代码(Code-On-Demand 可选)**，按需加载代码，提升客户端的简单性。



#### 参考文章

* [如何使用koa2+es6/7打造高质量Restful API](https://www.jianshu.com/p/f59594b90500)
* 
# Koa



#### 简介

* 基于Nodejs的下一代Web框架
* 是Node.js的一个模块
* 蚕食上一代Web框架Express的市场
* Web框架不是命令行工具、不是算法



#### 特点

* 利用async函数，丢弃回调函数
* 增强错误处理：try catch
* 没有捆绑任何中间件
* 快速而愉快的编写程序



#### Async Await

* **fetch**：回调前套

```js
fetch('https://api.github.com/users').then(res => res.json()).then(json => { 
	console.log(json)
	fetch('https://api.github.com/users/zhaobobin').then(res => res.json()).then(json2 => {
		console.log(json2)
	})
})
```

* **Async**：顺序执行

```js
(async () => {
	const res = await fetch('https://api.github.com/users')
	const json = await res.json()
	console.log(json)
	const res2 = await fetch('https://api.github.com/users/zhaobobin')
	const json2 = await res2.json()
	console.log(json2)
})()
```



#### [洋葱模型](https://baike.baidu.com/item/%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B/7675375?fr=aladdin)



#### 获取查询参数

* 获取url问号参数：**ctx.query**
* 获取路由参数：**ctx.params**
* 获取请求的body参数：**ctx.request.body**
* 获取header参数：**ctx.header**




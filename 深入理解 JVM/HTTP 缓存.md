[HTTP 缓存（MDN web docs）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ)

[可能是最被误用的 HTTP 响应头之一 Cache-Control: must-revalidate ](https://zhuanlan.zhihu.com/p/60357719)



## 缓存控制

Cache-control 头，用来区分缓存机制的支持情况，请求头和响应头都支持这个属性。

### 禁止进行缓存

> Cache-Control: no-store

就是说缓存中不得存储任何关于客户端请求和服务器响应的内容。每次由客户端发起的请求都会下载完整的响应内容。

### 强制确认缓存

> Cache-Control: no-cache

在这种方式下，每次有请求发出的时候，缓存会将这个请求发到服务器（该请求应该会带有与本地缓存相关的验证字段），服务器端会验证请求中所描述的缓存是否过期，若未过期（实际上就是返回 304），则缓存才使用本地缓存副本。

### 私有缓存和公共缓存

> Cache-Control: private
>
> Cache-Control: public

默认是 private，表示该响应是专用于单个用户的，中间人（比如中间代理、CDN 等）不能缓存此响应，该响应只能应用于浏览器私有缓存中。

### 缓存过期机制

> Cache-Control: max-age=31536000

指令“``` max-age=<seconds>```”，表示资源能够被缓存的最大时间。max-age 是距离请求发起的时间的秒数。

### 缓存验证确认

> Cache-Control: must-revalidate

`must-revalidate`指令是用来表示在一个缓存过期之后，不能直接使用这个过期的缓存，必须校验之后才能使用。

>  这里要提一下，revalidate 是缓存过期后自然而然的表现，为什么还需要专门的指令呢？这是因为 `must-revalidate`生效的场景有一个大前提，就是说 HTTP 规范是允许客户端在某些特殊的情况下直接使用过期缓存的，比如校验请求发送失败的时候，还比如有配置一些特殊指令（`stale-while-revalidate`、`stale-if-error`等）的时候。
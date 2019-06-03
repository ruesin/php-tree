# 跨域

## 同源策略

同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互，是一个用于隔离潜在恶意文件的重要安全机制。

如果两个页面的协议，端口（如果有指定）和主机（域名）都相同，则两个页面具有相同的源。

浏览器对于javascript的同源策略限制，如`a.com`下面的js不能调用`b.com`或`b.a.com`中的js、对象或数据。

请求的url地址，必须与浏览器上的url地址处于同域上，也就是域名、端口、协议相同。

## 解决

### JSONP

在HTML标签里，一些标签比如script、img等拥有”src”这个属性的获取资源的标签是没有跨域限制的，比如<\script>、<\img>、<\iframe>。

向远程接口发起get请求，API接收callback的参数，封装json到callback中，返回给客户端（浏览器），浏览器中有自定义的callback函数。

等同于，本地定义了callback()函数，然后引用远程js文件，远程js文件调用了callback()。

### Iframe

JSONP只能发GET请求，因为本质上script加载资源就是GET，如果要发POST请求可以使用iframe，基本原理差不多，需要注意数据传输问题。

### CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）跨域资源共享，允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制，是处理跨域问题的标准做法。

浏览器发现AJAX请求跨源，会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

#### 简单请求

请求方法是HEAD、GET、POST三种方法之一；HTTP的头信息不超出以下几种字段：Accept、Accept-Language、Content-Language、Last-Event-ID、Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain。

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。
如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```
- Access-Control-Allow-Origin：必须。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。
- Access-Control-Allow-Credentials：可选。值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
- Access-Control-Expose-Headers：可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

要向服务器发送cookie，前端必须在AJAX请求中打开withCredentials属性，否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。

如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。

#### 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

如果浏览器发起一个“非简单”请求，一个叫做预检查的机制会在正式通信之前，发送一个OPTIONS请求到服务器。

如果服务器没有返回带有特殊头部的数据，简单请求GET或者POST请求仍然会发送，服务器的数据也会返回，但是浏览器会阻止Javascript获取这次请求。

"预检"请求头信息里面，关键字段是Origin，表示请求来自哪个源，以及两个特殊字段：
- Access-Control-Request-Method：必须。用来列出浏览器的CORS请求会用到哪些HTTP方法。
- Access-Control-Request-Headers：一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段r。


服务器回应的其他CORS相关字段：
- Access-Control-Allow-Methods: 必须。值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法，如：GET, POST, PUT。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
- Access-Control-Allow-Headers: 如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
- Access-Control-Allow-Credentials: 与简单请求时的含义相同。
- Access-Control-Max-Age: 可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

### 代理
通过Nginx配置代理转发，请求同域地址后，进行转发。
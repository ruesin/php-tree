# 路由

路由的基础功能就是通过URL访问时，可以快速的根据URL解析出应执行的代码，基本思路就是从URL中按规则提取字符串，然后进行匹配。


## 简单
传统URL如：`https://localhost/index.php?action=user_info&uid=1`，定义路由配置文件`route.php`，解析url中的`action`的值，然后匹配相应控制器，将参数传入。
```php
# route.php
return [
    'user_info' => '\App\Controllers\User::info'
];
```

restful地址通常是解析资源定位符，截取`PATH_INFO`及请求方式，按照指定规则进行访问。

## Laravel
在Laravel中，路由定义在配置文件`routes.php`中，路由通常是URI、处理函数、请求方法来定义的。

在服务启动过程中，通过路由服务提供者`RouteServiceProvider`将已定义的信息注册到路由表。 

之后通过`Kernel`将请求信息`$request`传递给路由处理实例进行路由分发。

分发请求中根据请求方法、URL来查找对应的路由实例，查找到之后将请求传递给对应的路由去处理。

监测是否为常规的控制分发器，通过服务容器自动生成，将请求及路由中处理函数信息交给控制分发器处理。

在控制分发器中，先判断是否有中间件需要处理，之后根据路由提供的响应函数信息通过服务容器来实例化控制器类，并调用对应的响应函数来生成响应内容。

关于[服务容器](./container.md)

//TODO  补充代码
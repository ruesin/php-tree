# RESTful

RESTful API 是API的设计规范或者是一套设计理论，REpresentational State Transfer，中译为“表属性状态传递”。

可以理解为：`URL`用来唯一标示一个互联网资源，作用是定位资源；HTTP动词`Method`（GET,POST,DELETE,PUT）用来描述当前请求对该资源进行什么操作。

如同一URL：http://www.xx.com/user/123，DELETE请求标识要删除用户，GET请求标识获取用户信息。

## 特点
- URL是基于资源的，只能使用名词来指定资源，不能有动词，名词推荐使用复数，“资源”是REST规范的核心
- 用HTTP协议的动词来实现资源的增、删、改、查操作。
- RESTful只要维护资源的状态，而不需要维护客户端的状态。对于它来说，每次请求都是全新的，它只需要针对本次请求作相应的操作，不需要将本次请求的相关信息记录下来以便用于后续来自相同客户端请求的处理。
- 标准的`HTTP Status Code`在REST中都有特定的意义：200，201,202,204,400,401,403,500。比如401表示用户身份认证失败，403表示你验证身份通过了，但这个资源你不能操作。
- API要有版本的概念，如：v1。可以放URL或header中
- 支持多种资源表示方式：JSON,XML，推荐JSON
- 通过API提供的参数，过滤返回结果。如：?page=2&per_page=100&sortby=name&order=asc
- 使用Token令牌来做用户身份的校验与权限分级，而不是Cookie
- url中大小写不敏感，不要出现大写字母
- 使用 - 而不是使用 _ 做URL路径中字符串连接。

## HTTP方法
- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源；URI一般是标识添加资源存放容器的URI
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）；最终标识添加资源的URI是可以由请求者控制的，一般直接将标识添加资源的URI作为请求的URI；如果PUT提供的资源不存在，则做添加操作，否则做修改
- PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
- DELETE（DELETE）：删除一个已经存在的资源
- HEAD：一般不是为了获取目标资源本身的内容，而是得到描述目标资源的元数据信息
- OPTIONS：发送一种“探测”请求以确定针对资源的请求必须具有怎样的约束（比如应该采用怎样的HTTP方法以及自定义的请求报头），资源的哪些属性是客户端可以改变的，然后根据其约束发送真正的请求。比如针对“跨域资源”的预检（Preflight）请求采用的HTTP方法就是OPTIONS。

## 状态码（Status Codes）
- 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
- 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
- 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
- 204 NO CONTENT - [DELETE]：用户删除数据成功。
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
- 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
- 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
- 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
- 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
- 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
- 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
- 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

## 安全性与幂等性
HTTP请求的方法，具有两个基本的特性，即“安全性”和“幂等性”。

GET、HEAD和OPTIONS均被认为是安全的方法，因为它们旨在实现对数据的获取，并不具有“边界效应”。其他4个HTTP方法，由于会导致服务端资源的变化，所以被认为是不安全的方法。

幂等性（Idempotent）是一个数学上的概念，在这里表示发送一次和多次请求引起的边界效应是一致的。在网速不够快的情况下，客户端发送一个请求后不能立即得到响应，由于不能确定是否请求是否被成功提交，所以它有可能会再次发送另一个相同的请求，幂等性决定了第二个请求是否有效。

上述3种安全的HTTP方法（GET、HEAD和OPTIONS）均是幂等方法。由于DELETE和PATCH请求操作的是现有的某个资源，所以它们是幂等方法。对于PUT请求，只有在对应资源不存在的情况下服务器才会进行添加操作，否则只作修改操作，所以它也是幂等方法。至于最后一种POST，由于它总是进行添加操作，如果服务器接收到两次相同的POST操作，将导致两个相同的资源被创建，所以这是一个非幂等的方法。

在设计Web API的时候，应该尽量根据请求HTTP方法的幂等型来决定处理的逻辑。由于PUT是一个幂等方法，所以携带相同资源的PUT请求不应该引起资源的状态变化，如果我们在资源上附加一个自增长的计数器表示被修改的次数，这实际上就破坏了幂等型。

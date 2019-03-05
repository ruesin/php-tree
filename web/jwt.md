# Jwt

`JWT`是（JSON Web Token]）的缩写，主要用来做用户身份验证的。

随着当前分布式应用、前后端分离的技术广泛使用，早年通过session管理用户状态的成本越来越高，session共享问题以及之后出现的token认证基本都是通过如Redis之类的中间件实现的。

JWT通过将数据保存在客户端，每次请求时将token发送至服务端校验，服务端无需存储token，实现完全无状态化。

## 流程
- 客户端登录请求认证
- 服务端认证通过后，生成包含数据的`JSON`对象，并将此对象进行签名生成`token`
- 服务端将`token`返回客户端，客户端存储在本地，如cookie或localStorage
- 客户端下次请求时携带`token`到服务端，常用的是放在 HTTP 请求的头的Authorization字段中，`Authorization: Bearer <token>`
- 服务端验证`token`有效性

## 结构
`Token`是一个使用`.`分割的三部分组成的长字符串，`Header.Payload.Signature`

### Header
Header是一个Base64URL之后的json对象，`{"typ":"JWT","alg":"HS256"}`，`alg`表示签名的算法（algorithm），默认是 `HMAC SHA256`（HS256），`typ`表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT。

### Payload
Payload 也是一个Base64URL之后的JSON对象，用来存放传递的数据。JWT 规定了7个官方字段可用：
- iss (issuer)：签发人
- iat (Issued At)：签发时间
- exp (expiration time)：过期时间
- nbf (Not Before)：生效时间
- jti (JWT ID)：编号
- sub (subject)：主题
- aud (audience)：受众

除了官方字段，还可以在这个部分定义私有字段，比如
```json
{
  "sub": "101",
  "name": "ruesin",
  "LoginToken":"abcd123"
}
```
因为默认是Base64URL编码不加密的，所以客户端是可以解码读取这些数据，不要把秘密信息放在这个部分。

### Signature
Signature 是对前两部分的签名，校验tonken的有效性，防止数据篡改。

签名是通过服务端指定的密钥（secret）及Header中指定的签名算法产生的。
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```


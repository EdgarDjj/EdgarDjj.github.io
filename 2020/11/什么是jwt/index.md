# 什么是JWT


# 什么是JWT

> 本质上它是一段签名的 JSON 格式的数据。由于它是带有签名的，因此接收者便可以验证它的真实性。同时由于它是 JSON 格式的因此它的体积也很小。如果你想了解有关它的正式定义，可以在 [RFC 7519](https://tools.ietf.org/html/rfc7519) 中找到。

## Session、Cookie、Token

Cookie和Session的一个本质的区别就是，Session存在于服务端，而Cookie存在于客户端。Session的主要作用就是通过服务端记录用户的状态，Cookie数据保存在客户端（浏览器端）。

> cookie是一个非常具体的东西，指的就是浏览器中能永久存储的一种数据，仅仅是浏览器实现的一种数据存储功能。
>
> cookie由服务器生成，发送给浏览器，浏览器吧cookie以key-value形式保留。

### Session 和 Cookie 的使用过程

每当用户登陆的时候，服务器验证通过后，会创建一个Session并将Session信息存储起来，同时服务器会向用户返回一个SessionId并写入用户Cookie，当用户登陆的时候，Cookie将与每个后续请求一起被发送出去，服务端可以将存储在Cookie上的SessionId与存储在内存或者数据库中的Session信息进行校对，以验证用户身份，返回给用户客户端响应信息的时候回附带用户当前对的状态。

但由于Cookie无法防止CSRF（Cross Site Request Forgery）攻击，因为访问的信息存储在Cookie中，可以被窃取，被窃取的SessionId可以被用来登录，因此产生危险。

### Token

同时为了避免用户数极大的情况下，数据库需要存储大量的SessionId进行校对，效率低下同时影响性能。Token诞生，所有数据都存在于客户端，每次用户请求服务的时候都带上JWT到服务端，服务端用同样的算法进行解密与请求的Token中的签名进行比较来进行认证。用token替换sessionid用CPU计算时间来换取存取sessionid的代价。

## JWT

### 实现思路

* 用户登录校验，校验成功返回token
* 客户端接收数据保存到客户端
* 客户端每次访问API是携带Token到服务端的
* 服务端采用filter过滤器校验，校验成功返回请求数据，校验失败返回错误码

### JWT数据结构

* JWT token 的格式：xxxx.yyyy.zzzz
  * 头信息（header）
  * 负载（payload）
  * 签名（signature）
* header中用于存放签名的生成算法

```json
{
  "alg": "HS512",
  "typ": "JWT"
}
```

上面代码中，`alg`属性表示签名的算法（algorithm））；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

* payload中用于存放用户名、token的生成时间和过期时间

  ```json
  {
    "sub":"admin",
    "created":1489079981393,
    "exp":1489684781
  }
  ```

  

  * Registered claims
    * iss（Issuer）：JWT签发者
    * created：（Create Time）：JWT创建时间
    * exp（Expiration Time）：JWT的过期时间（必须大于签发时间）
    * sub（Subject）：JWT所面向的用户
    * aud（Audience）：接收JWT的一方
    * jti（JWT id）：JWT的身份标识，每个JWT的id都应该是不重复的，避免重复发放
  * Public claims
  * Private claims

* signature为以header和payload生成的签名，一旦header和payload被篡改，验证将失败。三部分组成
  * base64UrlEncode（header）
  * base64UrlEncode（payload）
  * secret（服务器秘密字串）

```java
//secret为加密算法的密钥
String signature = HMACSHA512(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)

```

### JWT实现认证和授权的原理

基于Token的身份验证是无状态的，不将用户信息存储在服务器中或者Session中。

* 用户调用登录接口，登录成功后获取到token
* 之后用户每次调用接口都在http的header中添加一个叫Authorization的头，值为JWT的token
* 后台程序通过对Authorization头中信息的解码及数字签名校验来获取其中用户信息，从而实现认证和授权

### 实现

```lua
mall-security
├── component
|    ├── JwtAuthenticationTokenFilter -- JWT登录授权过滤器
|    ├── RestAuthenticationEntryPoint -- 自定义返回结果：未登录或登录过期
|    └── RestfulAccessDeniedHandler -- 自定义返回结果：没有权限访问时
├── config
|    ├── IgnoreUrlsConfig -- 用于配置不需要安全保护的资源路径
|    └── SecurityConfig -- SpringSecurity通用配置
└── util
     └── JwtTokenUtil -- JWT的token处理工具类
```

### JWT的几个特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。



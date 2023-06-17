# 维护登录状态，自动验证身份

对于具备用户登录功能的应用程序，为提高用户操作的友好性，应该在用户第一次登录后，持续维护用户的登录状态，已登录用户在后面访问某些资源时便不再需要重新登录。

而且，如果每次用户访问资源都需要登录，这会造成大量的数据库访问，极其影响性能。

# 传统的 Session 身份验证

对于登录的客户，在服务端创建对应的 Session，其中保存用户的各种信息，比如购物车。之后用户再次请求，服务端会根据在 Cookie，或请求头，或请求参数的 SessionId，获取匹配的 Session，从而验证用户身份。

Session 身份验证的缺点：

* 随着登录用户的增多，服务端需要维护的 Session 也愈来愈多，服务器压力线性增加。
* 分布式系统下拓展性低，因为 Session 在服务端之间并不共享。如果强行使用这种方式，那就必须对 Session 进行持久化或缓存。
* 不安全，携带 SessionId 的 Cookie 如果被拦截，黑客便能伪造请求进行攻击。

# JSON Web Token（JWT）

JWT 是一种跨域认证解决方案，用于在空间受限的环境下安全传递声明。

简单地理解，JWT 就是一种使用 Json 来传输令牌 Token 的规范。它的特点是签名 Signature，Token 的签发者可以使用加密算法对信息进行签名，之后可以根据 Signature 验证 Token 信息是否被修改。

## JWT 的结构

JWT 的 Token 是一条很长的字符串，以两个 `.` 分为 3 个子串，分别表示 JWT 的 3 个部分。

### 头部 Header

第一个子串是一个 Json 对象的 ，这就是 Header。Hader 包含 Token 的元数据，比如 Token 类型，以及签名使用的算法。

这个子串其实是 Header 对象的 Base64URL 字符串。

### 负载 Payload

第二个子串也表示一个 Json 对象，这就是 Payload。Payload 包含真正需要传递的信息，JWT 规定有 7 个官方字段：签发者、主题、接收者、签发时间、生效时间、过期时间、唯一标识。

当然，还能再 Payload 中定义其它的字段，这也是 JWT 具有拓展性的原因。

这个子串其实是 Payload 对象的 Base64URL 字符串。

### 签名 Signature

最后一个子串是 Token 签名，它通过指定的加密算法，对 Header 和 Payload 进行计算而获得。公式如下：

```
HMACSHA256(
    base64UrlEncode(header) + "." + 
    base64UrlEncode(payload),
    secret)
```

先获得 Header 和 Payload 的 Base64URL 字符串，然后使用密钥加密，密钥只有服务端知道。

服务端收到请求，获取 Token，根据 Header 和 Payload 计算签名，然后和 Token 携带的签名进行比较，从而验证 Token 是否被修改。

## JWT 不安全

从 JWT 的结构可知，Token 的 Header 和 Payload 是公开的，任何人都能获取 Token 中携带的信息。当然，也可以对 Token 整体进行二次加密，这样就能保证 Token 所有部分的安全。

服务端不保存 Token 的信息，一旦签发就失去对它的控制。只要请求携带正确的 Token，服务端就会通过验证。

Token 包含过期时间，服务端不能主动使其失效，只能等待它过期。

## JWT 客户端

客户端收到服务端返回的 Token，可以把它放在 Cookie，也能放在 localStorage。客户端之后每次发送请求，都要携带这个 Token。

把 Token 放在 Cookie 可以实现自动发送，但这不能跨域，更好的做法是放在 HTTP 请求头的 `Authorization` 字段：

```
Authorization: Bearer <token>
```

## Spring Boot 集成 JWT

引入依赖：

```
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.5.0</version>
</dependency>
```

生成 Token：

```
public static String createToken(long id) throws Exception {
    // 构造签名的加密算法，这里使用RSA，需要设置密钥对
    Algorithm algorithm = Algorithm.RSA256(
    		RSAUtil.getPublicKey(), RSAUtil.getPrivateKey());
    // 构造时间
    Date start = new Date();
    Calendar end = Calendar.getInstance();
    end.setTime(new Date());
    end.add(Calendar.SECOND, 60 * 30);
    // 构造Token并返回
    return JWT.create()
            .withAudience("Lv0") // 接收者
            .withIssuer(ISSUER) // 签发者
            .withKeyId(String.valueOf(id)) // 唯一标识，这是使用入参
            .withIssuedAt(start) // 开始时间
            .withExpiresAt(end.getTime()) // 过期时间
            .sign(algorithm); // 根据指定的加密算法生成Token
}
```

验证 Token：

```
public static boolean verifyToken(String token, long id) throws Exception {
    // 构造签名的加密算法
    Algorithm algorithm = Algorithm.RSA256(
            RSAUtil.getPublicKey(), RSAUtil.getPrivateKey());
    // 构造Token解析器
    JWTVerifier verifier = JWT.require(algorithm).build();
    // 使用解析器获得Token的原始内容
    DecodedJWT jwt = verifier.verify(token);
    // 现在，可以使用DecodedJWT对象获取Token的各种字段
    // 获取唯一标识并和入参对比
    return id == Long.parseLong(jwt.getKeyId());
}
```

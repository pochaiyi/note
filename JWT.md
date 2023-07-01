# JSON Web Token

JWT 是一种数据交换格式，它把 JSON 数据进行包装，使用独立紧凑的方式进行传输。

# 内容结构

JWT 以长字符串的形式展现，字符串由以 `.` 连接的 3 个子串组成：`Header`、`Deploy` 和 `Signature`。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**头部 Header**

保存元数据，通常只有两个字段：

* `typ`：Token 类型，默认 `jwt`。
* `alg`：签名的加密算法，默认 HS256，可选 RSA、ECDSA。

示例，这是 Header 原文，以及经过 Base64 加密的结果。加密算法固定，不可修改。

```
{
  "alg": "HS256",
  "typ": "jwt"
}
```

```
eyJhbGciOiJIUzI1NiIsInR5cCI6Imp3dCJ9
```

**负载 Deploy**

保存用户数据，官方字段：签发者、接收者、签发时间、生效时间、过期时间、唯一标识。

示例，这是 Deploy 原文，以及经过 Base64 加密的结果。加密算法固定，不可修改。

```
{
  "sub": "Fans",
  "name": "Tom",
  "iat": "85590911"
}
```

```
eyJzdWIiOiIxMjM0NDMyMSIsIm5hbWUiOiJNYXJz6YWxIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

**签名 Signature**

使用 Header 指定的算法计算 JWT 的校验码，这个 `secret` 就是前面 Base64 所用的盐值。

```
HMACSHA256(
        base64UrlEncode(header) + "." + base64UrlEncode(payload),
        secret
	)
```

# 登录授权

JWT 常见用途是用户授权，免除反复登录，以下是可能的处理逻辑：

* 用户登录账号，服务端验证通过，创建并返回 JWT；
* JWT 的 Deploy 保存能够标识用户身份的信息；
* 客户端把 JWT 放在 Cookie 或 LocalStorage，用于后续请求；
* 服务端收到新请求，获取和解析 JWT，得到用户身份，检查权限，返回响应。

Token 自带用户信息，服务端不用保存数据，相比 Session+Cookie，节省大量内存；JWT 可拓展性高，天然支持单点登录，适合分布式服务；但是，服务端无法影响 JWT，最多设置较短的有效期。

# Spring 集成

引入依赖

```
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.5.0</version>
</dependency>
```

创建 JWT

```
public static String createToken(long id) throws Exception {
    // 签名使用RSA算法
    Algorithm algorithm = Algorithm.RSA256(PublicKey, PrivateKey);
    // 签发时间
    Date start = new Date();
    Calendar end = Calendar.getInstance();
    end.setTime(new Date());
    end.add(Calendar.SECOND, 60 * 30);
    // 创建Token
    return JWT.create()
            .withAudience("Lv0") // 接收者
            .withIssuer(ISSUER) // 签发者
            .withIssuedAt(start) // 开始时间
            .withExpiresAt(end.getTime()) // 过期时间
            .withKeyId(String.valueOf(id)) // 唯一标识
            .sign(algorithm); // 签名算法
}
```

解析 JWT

```
public static boolean verifyToken(String token, long id) throws Exception {
    Algorithm algorithm = Algorithm.RSA256(PublicKey, PrivateKey); // 签名算法
    JWTVerifier verifier = JWT.require(algorithm).build(); // 解析器
    DecodedJWT jwt = verifier.verify(token); // 得到原始内容
    return id == Long.parseLong(jwt.getKeyId()); // 检查唯一标识
}
```

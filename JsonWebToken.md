# JWT认证机制

**本小节目标**：

- 理解回顾session认证机制的过程和优缺点
- 理解JWT认证机制的过程及和session认证机制的区别
- 理解JWT token的3个组成部分及使用注意点

## JWT 认证机制

> JSON Web Token（JWT）是目前前后端分离开发中用户身份认证最流行的一种解决方案。

在用户登录后，我们想记录用户的登录状态，这里不再使用Session认证机制，而使用Json Web Token认证机制。

### 1. session认证机制

我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证。

![session认证机制](./images/session认证机制.png)

但是这种基于session的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于session认证应用的问题就会暴露出来.

##### 基于session认证所显露的问题：

- **Session**: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。
- **CSRF**: session是依赖于cookie的, 如果cookie被截获，用户就会很容易受到跨站请求伪造的攻击。
- **扩展性**: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

### 2. JWT token认证机制

JWT token认证机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

![jwt](./images/jwt.png)

流程上是这样的：

- 用户使用用户名和密码来请求服务器进行登录
- 服务器验证用户的登录信息
- 服务器通过验证，生成一个token并返回给用户
- 客户端存储token，并在每次验证请求携带上这个token值
- 服务端验证token值，并返回数据

### 3. JWT token数据格式

JWT token是一个字符串，由3部分组成，用`.`隔开。如下:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

1）头部(header)

jwt的头部承载两部分信息：

- 声明类型，这里是jwt
- 声明加密的算法 通常直接使用 HMAC SHA256

完整的头部就像下面这样的JSON：

```json
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

然后将头部进行base64加密（该加密是可以对称解密的)，构成了第一部分.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

2）载荷(payload)

载荷就是存放有效数据的地方。这些有效数据主要保存存储的数据和token的有效时间。

定义一个payload:

```json
{
  "username": "smart",
  "mobile": "13155667788",
  "email": "smart@163.com",
  "exp": "<token有效时间>"
}
```

然后将其进行base64加密，得到JWT的第二部分。

```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

3）签名(signature)

JWT的第三部分是一个签名信息，用来防止JWT token被伪造。

```
1. 签名生成过程？
答：服务器在生成jwt token时，会将header和payload字符串进行拼接，用.隔开，然后使用一个只有服务
器知道的密钥对拼接后的内容进行加密，加密之后生成的字符串就是signature内容。

2. 签名验证过程？
答：当客户端将jwt token传递给服务器之后，服务器首先需要进行签名验证：
  1）将客户端传递的jwt token中的header和payload字符串进行拼接，用.隔开
  2）使用服务器自己的密钥对拼接之后的字符串进行加密
  3）将加密之后的内容和将客户端传递的jwt token中signature进行对比，如果不一致，就说明
  jwt token是被伪造的
```

### 4. 使用注意点

- 不要在jwt的payload部分存放敏感信息，该部分是客户端可解密的部分
- 保护好服务器端的签名加密secret密钥
- 如果可以，请使用https协议

**总结**：

- session认证机制：
  - 缺点：登录用户过多，会过多占用服务器的存储空间
  - 优点：session中可以存储一些和业务相关的敏感数据
- JWT认证机制：
  - 特点：用户的身份信息保存在jwt token中，jwt token由客户端进行保存，之后客户端将jwt token传递给服务器，服务器对jwt token进行校验
  - 优点：jwt token存储在客户端，不会占用服务器的存储空间
- JWT token的组成部分：
  - header(头部)：存储token类型和签名加密算法
  - payload(载荷)：存储有效数据和token的有效时间
  - signature(签名)：防止jwt token被伪造
- JWT认证使用注意点：
  - payload中不要存过于敏感的数据
  - 服务器的签名加密密钥需要保存好
  - 使用https安全网络协议



# JWT扩展使用

**本小节目标**：

- 了解JWT扩展的功能和使用配置

## JWT扩展

**需求**：

1）服务器在验证完用户的身份后（检验用户名和密码)，需要生成jwt token并返回给客户端。

2）在客户端将jwt token传递给服务器时，服务器需要对jwt token数据进行校验。

**关于jwt token的生成和校验，都可以使用DRF JWT扩展来完成。**

[JWT扩展官网文档](http://getblimp.github.io/django-rest-framework-jwt/)

### 1. 使用

1）安装

```shell
pip install djangorestframework-jwt
```

2）配置

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # 引入JWT认证机制，当客户端将jwt token传递给服务器之后
        # 此认证机制会自动校验jwt token的有效性，无效会直接返回401(未认证错误)
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}

# JWT扩展配置
JWT_AUTH = {
    # 设置生成jwt token的有效时间
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
}
```

3）手动生成jwt token

JWT扩展的说明文档中提供了手动生成jwt token的方法。

```python
from rest_framework_jwt.settings import api_settings

jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

payload = jwt_payload_handler(user)
token = jwt_encode_handler(payload)
```

**总结**：

- JWT扩展
  - 功能：生成jwt token和校验jwt token
  - 安装：pip install djangorestframework-jwt
  - 配置：DRF框架引入jwt认证机制并设置生成jwt token的有效期
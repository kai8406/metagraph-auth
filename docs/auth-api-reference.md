# Metagraph Oauth2 API 设计规范

> 参考自[Github API V3](https://developer.github.com/v3/)

## 1. Overview

> Metagraph Oauth2 API 定义Authorization Code、Implicit、Password、Client Credentials四种授权模式的API

### 1.1 Root Endpoint

- http://domain:8080
- https://domain:8080

其中`domain`是具体的域名或者`ip:port`的形式,可以在`root endpoint`中加上`api version`标识调用的api版本

### 1.2 MediaType

form-data+json

### 1.3 Charset

UTF-8

### 1.4 Errors

错误码格式范例(JSON)

```json
{
  "error_code":40001
  "message":"invalid credential"
}
```

错误码含义表

| error_code | message                 | description                 |
| ---------- | ----------------------- | --------------------------- |
| 40001      | invalid credential      | 不合法的调用凭证                    |
| 40002      | invalid grant_type      | 不合法的grant_type              |
| 40003      | invalid access_token    | 不合法的access_token            |
| 40004      | invalid user            | 用户名或者密码错误                   |
| 40005      | invalid scope           | 不合法的scope                   |
| 40006      | invalid request         | 请求不合法                       |
| 40007      | invalid client          | client_id或client_secret参数无效 |
| 40008      | invalid redirect_uri    | 重定向地址不匹配                    |
| 40009      | invalid response_type   | 不合法的response_type           |
| 40010      | invalid code            | 不合法或已过期的code                |
| 40011      | invalid refresh_token   | 不合法的refresh_token           |
| 40012      | invalid token_type      | 不合法的token_type              |
| 41000      | missing access_token    | 缺失access_token参数            |
| 41001      | missing user_info       | 缺失用户信息                      |
| 41002      | missing grant_type      | 缺失missing参数                 |
| 41003      | missing response_type   | 缺失response_type参数           |
| 41004      | missing client_id       | 缺失client_id参数               |
| 41005      | missing refresh_token   | 缺失refresh_token参数           |
| 41006      | missing client_secret   | 缺失client_secret参数           |
| 41007      | missing code            | 缺失code参数                    |
| 41008      | missing redirect_uri    | 缺失redirect_uri参数            |
| 42001      | expired access_token    | access_token超时              |
| 42002      | expired refresh_token   | refresh_token超时             |
| 42003      | expired code            | code超时                      |
| 50001      | api unauthorized        | 接口未授权                       |
| 50002      | access_denied           | 用户或授权服务器拒绝授予数据访问权限          |
| 50001      | temporarily_unavailable | 服务暂时无法访问                    |



## 2. Authorization Code API

> Authorization Code API(授权码模式)，常用于服务端

### 2.1 authorize

> 用户访问客户端，客户端将用户导向认证服务器，返回授权界面，用户同意授权客户端后，认证服务器将用户导向客户端指定的"重定向URI"，同时附上一个授权码

**Endpoint:** 

POST: /oauth/authorize

**Request：**

```json
http://localhost:8080/oauth/authorize?client_id=myClientId&redirect_uri=http://example.com&response_type=code&scope=read write
```

**Parameter:**

| 参数            | 说明                          |
| ------------- | --------------------------- |
| response_type | 表示响应类型，此处的值固定为"code"，必选项。   |
| client_id     | 表示客户端的ID，必选项。               |
| redirect_uri  | 表示重定向的URI，可选项。              |
| scope         | 表示权限范围，如果与客户端申请的范围一致，此项可省略。 |

**Response:**

```json
Headers：

Location:http://example.com?code=8wWZTJ
```

| 结果       | 说明     |
| -------- | ------ |
| Location | 重定向uri |
| code     | 授权码    |

### 2.2 token

> 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）

**Endpoint:**

POST: /oauth/token

**Request：**

```json
http://localhost:8080/oauth/token?grant_type=authorization_code&redirect_uri=http://example.com&code=8wWZTJ
```

**Parameter:**

| 参数           | 说明                                      |
| ------------ | --------------------------------------- |
| grant_type   | 表示授权类型，此处的值固定为"authorization_code"，必选项。 |
| redirect_uri | 表示重定向的URI，可选项。                          |
| code         | 授权码                                     |

**Response:**

```json
{
  "access_token": "3a94d35d-e3fe-4426-ba4e-8774d8cdf0fe",
  "token_type": "bearer",
  "refresh_token": "4c8f7938-7ecb-45ac-b649-7c87f9a01dca",
  "expires_in": 1799,
  "scope": "read write"
}
```

| 结果            | 说明                                       |
| ------------- | ---------------------------------------- |
| access_token  | 表示访问令牌，必选项。                              |
| token_type    | 表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型，MAC类型相对于BEARER类型对于用户资源请求的区别在于，BEARER类型只需要携带授权服务器下发的token即可，而对于MAC类型来说，除了携带授权服务器下发的token，客户端还要携带时间戳，nonce，以及在客户端计算得到的mac值等信息，并通过这些额外的信息来保证传输的可靠性。 |
| refresh_token | 表示更新令牌，用来获取下一次的访问令牌，可选项。                 |
| expires_in    | 表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。        |
| scope         | 表示权限范围，如果与客户端申请的范围一致，此项可省略。              |

## 3. Implicit API

> Implicit API(简化模式)：客户端将用户导向认证服务器，返回授权界面，用户授权客户端后，认证服务器将用户导向客户端指定的"重定向URI"，并在URI中包含了访问令牌。常用于移动设备上。

**Endpoint:** 

POST: /oauth/authorize

**Request：**

```json
http://localhost:8080/oauth/authorize?response_type=token&scope=read write&client_id=myClientId&redirect_uri=http://example.com
```

**Parameter:**

| 参数            | 说明                         |
| ------------- | -------------------------- |
| response_type | 表示响应类型，此处的值固定为"token"，必选项。 |
| client_id     | 表示客户端的ID，必选项。              |
| redirect_uri  | 表示重定向的URI，可选项。             |
| scope         | 表示权限范围，如果与客户端申请的范围一致，此项可省略 |

**Response:**

```json
Headers：

Location:http://example.com#access_token=7c48f026-37d5-47e8-b45d-0ec7a4e30abc&token_type=bearer&expires_in=1028
```

| 结果           | 说明                                    |
| ------------ | ------------------------------------- |
| access_token | 表示访问令牌，必选项。                           |
| token_type   | 表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型 |
| expires_in   | 表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。     |



## 4. Password API

> Password API（密码模式）：客户端通过用户名和密码访问授权服务器获取token。常用于那些比较信任的服务，比如团队内的项目。

**Endpoint:** 

POST: /oauth/token

**Request：**

```json
http://localhost:8080/oauth/token?grant_type=password&username=admin&password=admin
```

**Parameter:**

| 参数         | 说明                           |
| ---------- | ---------------------------- |
| username   | 用户名，必填项                      |
| password   | 密码，必填项                       |
| grant_type | 表示授权类型，必选项，此处的值固定为"password" |
| scope      | 表示权限范围，可选项。                  |

**Response:**

```json
{
  "access_token": "3a94d35d-e3fe-4426-ba4e-8774d8cdf0fe",
  "token_type": "bearer",
  "refresh_token": "4c8f7938-7ecb-45ac-b649-7c87f9a01dca",
  "expires_in": 1455,
  "scope": "read write"
}
```

| 结果            | 说明                                    |
| ------------- | ------------------------------------- |
| access_token  | 表示访问令牌，必选项。                           |
| token_type    | 表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型 |
| refresh_token | 表示更新令牌，用来获取下一次的访问令牌，可选项。              |
| expires_in    | 表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。     |
| scope         | 表示权限范围，如果与客户端申请的范围一致，此项可省略。           |

## 5. Client Credentials API

> Client Credentials API（客户端模式）：客户端通过client_id和client_secret访问授权服务器获取token，主要用于应用api访问

**Endpoint:** 

POST: /oauth/token

**Request：**

```json
http://localhost:8080/oauth/token?client_id=myClientId&client_secret=myClientSecret&grant_type=client_credentials&&scope=read write
```

**Parameter:**

| 参数            | 说明                                     |
| ------------- | -------------------------------------- |
| client_id     | 客户端ID，必填项                              |
| client_secret | 客户端密钥，必填项                              |
| grant_type    | 表示授权类型，必选项，此处的值固定为"client_credentials" |
| scope         | 表示权限范围，可选项。                            |

**Response:**

```json
{
  "access_token": "d525bbd8-2aa8-402c-a4dc-b23992128801",
  "token_type": "bearer",
  "refresh_token": "ec64084b-11c1-4888-83d0-1c6e8c2f9e51",
  "expires_in": 1799,
  "scope": "read write"
}
```
| 结果            | 说明                                       |
| ------------- | ---------------------------------------- |
| access_token  | 表示访问令牌，必选项。                              |
| token_type    | 表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型，MAC类型相对于BEARER类型对于用户资源请求的区别在于，BEARER类型只需要携带授权服务器下发的token即可，而对于MAC类型来说，除了携带授权服务器下发的token，客户端还要携带时间戳，nonce，以及在客户端计算得到的mac值等信息，并通过这些额外的信息来保证传输的可靠性。 |
| refresh_token | 表示更新令牌，用来获取下一次的访问令牌，可选项。                 |
| expires_in    | 表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。        |
| scope         | 表示权限范围，如果与客户端申请的范围一致，此项可省略。              |


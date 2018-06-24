
# 基本认证(basic authentication)

1. --> 普通 GET 请求
2. <-- 401 响应码拒绝请求，携带响应头 `WWW-Authenticate` 描述保护区域 和 认证算法
3. --> 请求携带 `Authorization` 头，指明认证算法和用户名密码
4. <-- 200 相应

## Read More
- [通用的 HTTP 认证框架](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication)
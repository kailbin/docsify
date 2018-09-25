# Spring Cloud Zuul 内置过滤器

下表为 Spring Cloud Zuul 内置过滤器 ，类名前面的数字代表了 Zuul 过滤器的执行顺序，**数字越小，越先执行**，`...zuul.filters.support.FilterConstants` 常量类 中定义了 过滤器的类型 和 执行顺序。

| pre                        | route                       | post                       | error              |
| -------------------------- | --------------------------- | -------------------------- | ------------------ |
| -3  ServletDetectionFilter | 10   RibbonRoutingFilter    | 900  LocationRewriteFilter | 0  SendErrorFilter |
| -2  Servlet30WrapperFilter | 100 SimpleHostRoutingFilter | 1000  SendResponseFilter   |                    |
| -1  FormBodyWrapperFilter  | 500 SendForwardFilter       |                            |                    |
| 1   DebugFilter            |                             |                            |                    |
| 5   PreDecorationFilter    |                             |                            | []()               |

## 简要说明

## pre

- **`ServletDetectionFilter`**：检查是否是 Web MVC，并设置`isDispatcherServletRequest`到上下文
- **`Servlet30WrapperFilter`**：Zuul 默认的包装器`HttpServletRequestWrapper`，对请求参数的获取进行了统一化包装，该过滤器获取其原始Request对象，并用 `Servlet30RequestWrapper`进行重新包装，去除`HttpServletRequestWrapper`中多余的操作，`ctx.getRequest()` 返回的 `HttpServletRequest` 的实现是 `Servlet30RequestWrapper`
- **`FormBodyWrapperFilter`**：如果是表单提交，用`FormBodyRequestWrapper`包装Request，目的是解析表单数据重新编码，这时 `ctx.getRequest()` 返回的 `HttpServletRequest` 的实现是 `FormBodyRequestWrapper`
- **`DebugFilter`**：如果设置了`zuul.debug.request=true` ，或在请求时 加上`debug=true`的参数，过滤器会进行如下操作 `ctx.setDebugRouting(true);`、`ctx.setDebugRequest(true);`
- _**`PreDecorationFilter`**_：填充了大量的上下文 和 请求头 信息，在 `@EnableZuulProxy` 注解下生效

## route

- _**`RibbonRoutingFilter`**_：该过滤器使用Ribbon，Hystrix 和可插拔的HTTP客户端发送请求，`routeHost` 为空&& `serviceId` 不为空的时候执行
- _**`SimpleHostRoutingFilter`**_：真正实现地址转发，其实质是调用 `HttpClient` 进行请求，`routeHost` 不为空的时候执行 
- `SendForwardFilter`：使用`RequestDispatcher`转发请求，转发位置存储在上下文`forward.to` 中

## post

- **`LocationRewriteFilter`**：对 状态是 `301` ，相应头中有 `Location` 的相应进行处理
- **`SendResponseFilter`**：将服务的响应数据写入当前响应

## error

- **`SendErrorFilter`**：默认转发到`/error`，可以设置`error.path`属性修改默认的转发路径



## 过滤器类型

下面是几种标准过滤器类型，与请求的生命周期相对应 

- `pre` 在路由到 Origin 之前执行。常见用法如：权限校验、记录日志等
- `route` 将请求路由到 Origin 。使用 Apache HttpClient 或 Netflix Ribbon 构建和发送原始 HTTP 请求
- `post` 在请求被路由到 Origin 执行执行。常见用法如：设置 HTTP 响应头，统计指标、返回相应数据等
- `error` 当其中一个阶段发生错误时执行

除了标准的过滤器类型外，我们还可以创建自定义的过滤器类型。例如，有个自定义的 `static` 类型，直接生成相应数据，而不转发请求到 Origin。

![Zuul Request Lifecycle](https://camo.githubusercontent.com/4eb7754152028cdebd5c09d1c6f5acc7683f0094/687474703a2f2f6e6574666c69782e6769746875622e696f2f7a75756c2f696d616765732f7a75756c2d726571756573742d6c6966656379636c652e706e67)



## Zuul 原生过滤器扩展

| Class                | filterType | filterOrder | 说明                                                |
| -------------------- | ---------- | ----------- | --------------------------------------------------- |
| SurgicalDebugFilter  | pre        | 99          | 允许将特定请求路由到单独的调试集群或主机            |
| StaticResponseFilter | static     | 0           | 允许从 Zuul 本身生成响应，而不是将请求转发到 Origin |



## 参考资料

- [Api网关 Zuul源码解读 Filters](https://www.slahser.com/2016/10/26/API%E7%BD%91%E5%85%B3-Zuul%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB-filters/)
- [Spring Cloud内置的Zuul过滤器详解](http://www.itmuch.com/spring-cloud/zuul/zuul-filter-in-spring-cloud/)
- [How it Works](https://github.com/Netflix/zuul/wiki/How-it-Works)
- [zuul simple webapp](https://github.com/Netflix/zuul/wiki/zuul-simple-webapp)
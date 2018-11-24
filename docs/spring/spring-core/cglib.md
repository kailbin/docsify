# cglib

cglib 已纳入 spring-core 模块，包 `org.springframework.cglib.proxy`。

## 入门示例



## Callback

- `Callback` (..cglib.proxy) ：空接口
    - `MethodInterceptor` (..cglib.proxy)：
    - `InvocationHandler` (..cglib.proxy)：
    - `NoOp` (..cglib.proxy)：什么也不做，类似于直接调用目标对象
    - `FixedValue` (..cglib.proxy)：不调用 目标方法，而是返回一个固定的值
    - `LazyLoader` (..cglib.proxy)：
    - `Dispatcher` (..cglib.proxy)：
    - `ProxyRefDispatcher` (..cglib.proxy)：
    



## Read More
- [CGLib动态代理的介绍及用法（单回调、多回调、不处理、固定值、懒加载）](https://blog.csdn.net/difffate/article/details/70552056)
- [cglib How To](https://github.com/cglib/cglib/wiki/How-To)
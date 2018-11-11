# aopalliance

```xml
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
```

这个包是 **AOP联盟** 的API包，里面包含了针对面向切面的接口。

目前 Spring 已把改包 纳入到了 `spring-aop` 模块里面，所以无需依赖。

## 包结构

- org.aopalliance
  - aop
    - `Advice`： 空接口，标识 接口 是 一个 advice 接口（通知、增强，如：前置通知、后置增强... 直译过来是通知，按照代理的意思是增强）
    - AspectException
  - intercept
    - `Interceptor` ： 空接口，标识 接口 是 一个 通用的拦截器。拦截器可以拦截运行时程序中的事件(方法调用、字段访问、异常...) ，事件的具体操作由 `Joinpoint` 实现

    - ConstructorInterceptor：构造器 拦截器

    - MethodInterceptor：方法 拦截器

    - `Joinpoint`：代码的执行，方法的执行，构造器的执行等，代表被切入的位置。功能：调用被切入对象、获取被切入对象、获取被调用对象（方法、构造器、字段..）

    - Invocation：代表程序的调用，功能：获取调用参数

    - ConstructorInvocation：构造器的调用，功能：获取 Constructor 构造器对象

    - MethodInvocation：方法的调用：获取 Method 对象


## 继承结构

- **Advice** org.aopalliance.aop
  - **Interceptor** org.aopalliance.intercept
    - **ConstructorInterceptor** org.aopalliance.intercept
    - **MethodInterceptor** org.aopalliance.intercept
  - *BeforeAdvice* org.springframework.aop
  - *AfterAdvice* org.springframework.aop
    - *AfterReturningAdvice* org.springframework.aop
    - *ThrowsAdvice* org.springframework.aop
- **Joinpoint** org.aopalliance.intercept
  - **Invocation** org.aopalliance.intercept
    - **ConstructorInvocation** org.aopalliance.intercept
    - **MethodInvocation** org.aopalliance.intercept
      - *ProxyMethodInvocation* org.springframework.aop
        - *ReflectiveMethodInvocation* org.springframework.aop.framework
          - *CglibMethodInvocation* org.springframework.aop.framework
# 基于注解进行 AOP 编程

## 打开开关

-  `<aop:aspectj-autoproxy/>`
- 或 `<bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator" />`
- 或 `@EnableAspectJAutoProxy`

## 编写切面

```java
package xyz.kail.demo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class HelloAspectj {

    /**
     * 所有的 say 方法
     */
    @Pointcut("execution( * say(..) )")// 切入点表达式
    public void sayMethod() { // 切入点签名，返回类型必须是 void
        // 切入点代表一些列位置，没有具体执行逻辑
    }

    /**
     * 所有的带有 @Transactional 或者 方法名是 say 的方法
     */
    @Pointcut("sayMethod() || @annotation(org.springframework.transaction.annotation.Transactional)")
    public void tx() {
    }

    /**
     * 前置通知
     */
    @Before("tx()")
    public void before() {
        System.out.println("before() 有个事务");
    }

    /**
     * 环绕通知
     */
    @Around("tx()")
    private Object around(ProceedingJoinPoint pjp) throws Throwable {
        try {
            System.out.println("around() 开始事务");
            Object result = pjp.proceed();
            System.out.println("around() 提交事务");
            return result;
        } catch (Throwable ex) {
            System.out.println("around() 回滚事务");
            throw ex;
        } finally {
            System.out.println("around() 释放资源");
        }
    }

    /**
     * 后置通知(参数类型与目标方法 返回值一致)
     */
    @AfterReturning(pointcut = "tx()", returning = "result")
    public void afterReturning(Long result) {
        System.out.println("afterReturning() 事务正常结束，返回结果：" + result);
    }

    /**
     * 异常通知
     */
    @AfterThrowing(pointcut = "tx()", throwing = "ex")
    public void afterThrowing(Throwable ex) {
        System.out.println("afterThrowing() 事务错误，回滚； 异常信息：" + ex.getMessage());
    }

    /**
     * 最终通知
     */
    @After("tx()")
    public void after() {
        System.out.println("after() 释放资源");
    }

}

```

## 目标类

```java
package xyz.kail.demo;

import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

@Component
public class Hello {

    public void say(String something) {
        System.out.println("Hello " + something);
    }

    @Transactional
    public void doit(String something) {
        System.out.println("doit " + something);
        // 异常
        System.out.println(1 / 0);
    }

}
```

## 启动程序

```java
package xyz.kail.demo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class App {
    
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext("xyz.kail.demo");
        Hello hello = context.getBean(Hello.class);
        System.out.println(hello.getClass());

        hello.say("World");
        System.out.println(" ============ ");
        try {
            hello.doit("World");
        } catch (Exception ex) {
            // 吃掉异常信息
        }
    }
}

```

## 输出

```bash
# 使用的 cglib 代理
class xyz.kail.demo.Hello$$EnhancerBySpringCGLIB$$2d590916
# 
# 调用 say 方法
around() 开始事务
before() 有个事务
Hello World
around() 提交事务
around() 释放资源
after() 释放资源
# afterReturning 在 after 后执行
afterReturning() 事务正常结束，返回结果：1542448402933
 ============ 
# 
# 调用 doit 方法 
around() 开始事务
before() 有个事务
doit World
around() 回滚事务
around() 释放资源
after() 释放资源
# 抛出异常，被执行
afterThrowing() 事务错误，回滚； 异常信息：/ by zero
# 抛出异常后 不执行 afterReturning
```



## Spring AOP 支持的 AspectJ 语法

Spring AOP 并没有支持 AspectJ 全部语法，具体语法详见以下文档：

> - [使用Spring进行面向切面编程（AOP）6.2.3.4. 示例](http://shouce.jb51.net/spring/aop.html#aop-pointcuts-examples)
> - [Spring AOP中JoinPoint的表达式定义描述](http://chiyx.iteye.com/blog/1568600)
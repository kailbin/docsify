# 基于 schema 进行 AOP 编程



## 定义切面

```java
package xyz.kail.demo;

import org.aspectj.lang.ProceedingJoinPoint;

public class HelloAspectj {

    /**
     * 前置通知
     */
    public void before() {
        System.out.println("before() 有个事务");
    }

    /**
     * 环绕通知
     */
    private Object around(ProceedingJoinPoint pjp) throws Throwable {
        try {
            System.out.println("around() 开始事务");
            Object result = pjp.proceed(pjp.getArgs());
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
     * 后置通知
     */
    public void afterReturning(Long result) {
        System.out.println("afterReturning() 事务正常结束，返回结果：" + result);
    }

    /**
     * 异常通知
     */
    public void afterThrowing(Throwable ex) {
        System.out.println("afterThrowing() 事务错误，回滚； 异常信息：" + ex.getMessage());
    }

    /**
     * 最终通知
     */
    public void after() {
        System.out.println("after() 释放资源");
    }
}
```

## 目标类

```java
package xyz.kail.demo;

import org.springframework.transaction.annotation.Transactional;

public class Hello {

    public long say(String something) {
        System.out.println("Hello " + something);
        return System.currentTimeMillis();
    }

    @Transactional
    public long doit(String something) {
        System.out.println("doit " + something);
        // 异常
        System.out.println(1 / 0);
        return System.currentTimeMillis();
    }

}

```

## schema 配置文件

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="hello" class="xyz.kail.demo.Hello"/>
    <bean id="helloAspectj" class="xyz.kail.demo.HelloAspectj"/>

    <aop:config>
        <aop:pointcut id="sayMethod" expression="execution( * say(..) ) or @annotation(org.springframework.transaction.annotation.Transactional)"/>

        <aop:aspect ref="helloAspectj">
            <aop:before pointcut-ref="sayMethod" method="before"/>
            <aop:around pointcut-ref="sayMethod" method="around"/>
            <aop:after pointcut-ref="sayMethod" method="after"/>
            <aop:after-throwing pointcut-ref="sayMethod" method="afterThrowing" throwing="ex"/>
            <aop:after-returning pointcut-ref="sayMethod" method="afterReturning" returning="result"/>
        </aop:aspect>
    </aop:config>

</beans>
```



## 启动程序

```java
package xyz.kail.demo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:/applicationContext.xml");
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
before() 有个事务
around() 开始事务
Hello World
around() 提交事务
around() 释放资源
after() 释放资源
# afterReturning 在 after 后执行
afterReturning() 事务正常结束，返回结果：1542448402933
 ============ 
# 
# 调用 doit 方法 
before() 有个事务
around() 开始事务
doit World
around() 回滚事务
around() 释放资源
after() 释放资源
# 抛出异常，被执行
afterThrowing() 事务错误，回滚； 异常信息：/ by zero
# 抛出异常后 不执行 afterReturning
```

## Read More

- [使用Spring进行面向切面编程（AOP） 6.3. 基于Schema的AOP支持](http://shouce.jb51.net/spring/aop.html#aop-schema)
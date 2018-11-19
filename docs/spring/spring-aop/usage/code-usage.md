# 基于代码进行 AOP 编程

基于代码进行 AOP 编程是 [**基于AspectJ注解**](spring-aop/usage/annotation-usage.md) 和 [**基于 Schema**](spring-aop/usage/schema-xml-usage.md) 的底层实现。



##  ProxyFactory 硬编码

ProxyFactory 可以脱离 Spring 容器创建代理，根据目标类是否实现接口 或 相关设置 来返回 cglib 或 JDK 代理类。具体逻辑可查看 `DefaultAopProxyFactory` 类。

```java
ProxyFactory proxyFactory = new ProxyFactory();
// 设置目标类
proxyFactory.setTarget(new Hello());
// 设置通知
proxyFactory.addAdvice(new HelloAdvice());
Hello hello = (Hello) proxyFactory.getProxy();
```



## ProxyFactoryBean

`ProxyFactoryBean` 是一个 `FactoryBean`， 最佳使用场景是与 Spring 容器结合，与 `ProxyFactory` 的区别是 `ProxyFactoryBean` 实现了 Spring 生命周期的一些接口：BeanClassLoaderAware、BeanFactoryAware，可以从 Spring 容器中获取 Advice。

下面也是一个硬编码的示例，每次只能代理一个目标类

```xml
    <bean id="helloTarget" class="xyz.kail.demo.Hello"/>

    <bean id="helloAdvice" class="xyz.kail.demo.HelloAdvice"/>

    <bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="helloTarget"/>
        <property name="interceptorNames">
            <list>
                <!-- 通过 bean 的名称获取 Advice -->
                <value>helloAdvice</value>
            </list>
        </property>
    </bean>
```



## AutoProxy 自动扫描创建 AOP 代理流程

1. Spring 容器在 Bean 扫描完成后会把 **上下文信息** 和 **Bean**（目标类） 传入 `AbstractAutoProxyCreator`
2. `AbstractAutoProxyCreator` 从 Spring 上下文中扫描 匹配 目标类 的 Advice，根据子类实现方式的不同 检索方式
3. 拿到Advice之后，根据目标类的属性不同，通过 `ProxyFactory` 创建 JDK 或 cglib 代理

### AbstractAutoProxyCreator 继承结构

- **ProxyConfig** (..aop.framework) implements Advised
  - ...
  - ProxyProcessorSupport (..aop.framework)
    - **AbstractAutoProxyCreator** (..aop.framework.autoproxy) implements **..BeanPostProcessor**, **BeanFactoryAware**
      - BeanNameAutoProxyCreator (..aop.framework.autoproxy)
      - **AbstractAdvisorAutoProxyCreator** (..aop.framework.autoproxy)
        - DefaultAdvisorAutoProxyCreator (..aop.framework.autoproxy)
        - **AspectJAwareAdvisorAutoProxyCreator** (..aop.aspectj.autoproxy)
          - **AnnotationAwareAspectJAutoProxyCreator** (..aop.aspectj.annotation) 
        - InfrastructureAdvisorAutoProxyCreator (..aop.framework.autoproxy)
    - AbstractAdvisingBeanPostProcessor (..aop.framework) implements **BeanPostProcessor**
      - AbstractBeanFactoryAwareAdvisingPostProcessor (..aop.framework.autoproxy)
        - ...
  - **AbstractSingletonProxyFactoryBean** (..aop.framework)
    - **TransactionProxyFactoryBean** (..transaction.interceptor)
    - **CacheProxyFactoryBean** (..cache.interceptor)
  - **AdvisedSupport** (..aop.framework)
    - **ProxyCreatorSupport** (..aop.framework)
      - **ProxyFactoryBean** (..aop.framework)
      - **ProxyFactory** (..aop.framework)
      - **AspectJProxyFactory** (..aop.aspectj.annotation)



## Pointcut

`Pointcut` 表示匹配规则，通过不同的语法表示一个位置，主要继承体系如下：

- Pointcut (..aop)：定义了 类与方法 匹配接口
  - **ExpressionPointcut** (..aop.support)
    - **AbstractExpressionPointcut** (..aop.support)
      - **AspectJExpressionPointcut** (..aop.aspectj)：AspectJ 表达式语法
  - AnnotationMatchingPointcut (..aop.support.annotation)：注解匹配
  - **StaticMethodMatcherPointcut** (..aop.support)：只匹配方法，类默认匹配成功
    - **TransactionAttributeSourcePointcut** (..transaction.interceptor) ：
    - CacheOperationSourcePointcut (..cache.interceptor)
    - AbstractRegexpMethodPointcut (..aop.support)
      - **JdkRegexpMethodPointcut** (..aop.support)：通过 JDK 正则表达式匹配
    - NameMatchMethodPointcut (..aop.support)

## Advisor

`Advisor`  用来组装 `Pointcut` 和 `Advice`。创建代理类的时候，通过 `Pointcut` 配置匹配目标类，通过 `Advice` 增强目标类创建代理。

- Advisor (..aop)
  - **PointcutAdvisor** (..aop)
    - StaticMethodMatcherPointcutAdvisor (..aop.support)
    - AspectJPointcutAdvisor (..aop.aspectj)
    - **AbstractPointcutAdvisor** (..aop.support)
      - AsyncAnnotationAdvisor (..scheduling.annotation)
      - PersistenceExceptionTranslationAdvisor (..dao.annotation)
      - **AbstractBeanFactoryPointcutAdvisor** (..aop.support)
        - **BeanFactoryTransactionAttributeSourceAdvisor** (..transaction.interceptor)
        - **DefaultBeanFactoryPointcutAdvisor** (..aop.support)
        - BeanFactoryCacheOperationSourceAdvisor (..cache.interceptor)
      - AbstractGenericPointcutAdvisor (..aop.support)
        - RegexpMethodPointcutAdvisor (..aop.support)
        - **AspectJExpressionPointcutAdvisor** (..aop.aspectj)
        - **NameMatchMethodPointcutAdvisor** (..aop.support)
        - **DefaultPointcutAdvisor** (..aop.support)
      - **TransactionAttributeSourceAdvisor** (..transaction.interceptor)
  - IntroductionAdvisor (..aop)
    - DefaultIntroductionAdvisor (..aop.support)
    - DeclareParentsAdvisor (..aop.aspectj)

## 总结各个组件之间的关系

- Pointcut：用来描述匹配规则，匹配 类 和 方法
- Advice：代表具体要增强的功能，如：前置通知(拦截器)要做什么、后置通知(拦截器)要做什么 等
- Advisor：等于 Pointcut + Advice
- Creator：会扫描容器中 Advisor，对 Spring 容器中 Bean 进行匹配，匹配成功通过 Advice 进行增强

## Read More

- [【中文】Spring AOP APIs](http://shouce.jb51.net/spring/aop-api.html)
- [6. Spring AOP APIs](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-api)
- [Spring中AOP的配置从1.0到5.0的演进](http://cxis.me/2017/04/10/Spring%E4%B8%ADAOP%E7%9A%84%E9%85%8D%E7%BD%AE%E4%BB%8E1.0%E5%88%B05.0%E7%9A%84%E6%BC%94%E8%BF%9B/)
- [Spring AOP拦截-使用切点:AspectJExpressionPointcut-切点语言](https://blog.csdn.net/qq_26525215/article/details/52422395/)
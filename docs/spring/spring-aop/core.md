# Spring AOP 核心概念



## 术语

- **切面（Aspect）**：可理解为AOP切入的单元
- **连接点（Joinpoint）**：表示一个方法的执行，常作为 **通知（Advice）** 方法的参数
- **切入点（Pointcut）**：匹配连接点的**语法**
- **通知（Advice）**：在切面的某个特定的连接点上**执行的动作**
  - **前置通知**：连接点之前执行，`@Before`
  - **后置通知**：连接点正常完成后执行，`@AfterReturning`
  - **异常通知**：抛出异常时执行，`@AfterThrowing`
  - **最终通知**：当某连接点退出的时候执行的通知，`After`，不论是正常返回还是异常退出
  - **环绕通知**：以上四种通知都可以通过环绕通知实现，`@Around`。但是，用最合适的通知类型可以使得编程模型变得简单，并且能够避免很多潜在的错误，这是其他类型通知存在的价值。
- Advisor：用于把 Advice 和 Pointcut 进行绑定，就像是只有一个通知的切面。在AspectJ中没有等价的概念
- **引入（Introduction）**：用来给一个类型**声明额外的方法或属性**
- **目标对象（Target Object）**： 被代理对象、被切入对象
- **织入（Weaving）**：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时、类加载时、运行时完成。**Spring在运行时完成织入**
- AspectJ：AOP 的一种实现方式，可采用 编译时 或 类加载时的代码织入（Spring AOP 则使用 运行时织入）。控制力度 比 Spring AOP 更细，可对私有方法、变量等进行控制。Spring 可无缝集成 AspectJ。

## Spring AOP 如何选择 JDK 或 cglib 代理?

具体逻辑如下：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		/*
		 *    optimize 被设置为 true
		 * || proxyTargetClass 被设置为 true，<aop:aspectj-autoproxy proxy-target-class="true"/>
		 * || 目标类 没有接口 或 接口是org.springframework.aop.SpringProxy
		 */
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			 // 目标类是一个接口 || 或者本身就是一个代理类
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				// JDK 代理
				return new JdkDynamicAopProxy(config);
			}
			// cglib 代理
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			// JDK 代理
			return new JdkDynamicAopProxy(config);
		}
	}

	/**
	 * 目标类 没有接口 或 接口是org.springframework.aop.SpringProxy
	 */
	private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
		Class<?>[] ifcs = config.getProxiedInterfaces();
		return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
	}

}
```







## 参考资料

- [使用Spring进行面向切面编程（AOP）](http://shouce.jb51.net/spring/aop.html)
# 动态代理



## 定义接口(同静态代理)

```java
public interface HelloI {
    void say(String say);
}
```

## 实现类(同静态代理)

```java
public class HelloImpl implements HelloI {

    @Override
    public void say(String say) {
        System.out.println("Hello " + say);
    }
}
```

## 动态代理类

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class HelloInvocationHandler implements InvocationHandler {

    private HelloI hello;

    public HelloInvocationHandler(HelloI hello) {
        this.hello = hello;
    }
    
    /**
     * @param proxy  生成的代理对象
     * 				com.sun.proxy.$Proxy0 extends java.lang.reflect.Proxy implements HelloI
     * @param method 调用的方法
     * @param args   方法参数
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            System.out.println("before!!!");
            // 调用被代理的方法
            return method.invoke(hello, args);
        } finally {
            System.out.println("after!!!");
        }
    }
}

```

## 使用

```java
HelloImpl helloImpl = new HelloImpl();

HelloI helloI = (HelloI) Proxy.newProxyInstance(
        ClassLoader.getSystemClassLoader(),
    	// 需要实现的接口
        new Class[]{HelloI.class},
    	// 处理器
        new HelloInvocationHandler(helloImpl)
);

helloI.say("World");
```

`Proxy.newProxyInstance(classLoader, interfaces, invocationHandler)` 也可以拆成以下三步

```java
// 生成代理类 com.sun.proxy.$Proxy0 ex java.lang.reflect.Proxy impl HelloI
Class<?> proxyClass = Proxy.getProxyClass(ClassLoader.getSystemClassLoader(), HelloI.class);
// 获取代理类的构造方法 public $Proxy0(InvocationHandler ih)
Constructor<?> constructor = proxyClass.getConstructor(InvocationHandler.class);
// 实例化代理类
HelloI helloI = (HelloI) constructor.newInstance(new HelloInvocationHandler(helloImpl));
```


# 常见使用场景

## 通过注解获取行为

常见场景有： 基于注解的 ORM 框架（paoding-rose-jade、MyBatis ...）

### 接口

```java
public interface HelloI {

    @Resource(name = "Hello ")
    void say(String say);
}
```



### 代理处理器

```java
import javax.annotation.Resource;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.Arrays;

public class HelloInvocationHandler implements InvocationHandler {

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        try {
            System.out.println("before!!!");
            // 通过方法注解，获取行为
            Resource resource = method.getAnnotation(Resource.class);
            String action = resource.name();
            System.out.println(action + Arrays.toString(args));
        } finally {
            System.out.println("after!!!");
        }
        return null;
    }
}
```



## 重写方法行为

以下摘录了部分 tomcat-jdbc 里面 数据库连接池 的部分实现，`ProxyConnection` 修改了 部分 `Connection` 的行为。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    ...
        
    // 判断是否是调用 Connection接口 的 close 方法
    if (compare(CLOSE_VAL,method)) {
        if (connection==null) return null; //noop for already closed.
        // 原始的 connection 实例
        PooledConnection poolc = this.connection;
        this.connection = null;
        // 不是真的关闭 Connection，而是把 Connection 返回到池中
        pool.returnConnection(poolc);
        return null;
    } 
    
    ...
    
    try {
        PooledConnection poolc = connection;
        if (poolc!=null) {
            // 不需要改变默认行为的，就调用原始 connection 实例的方法
            return method.invoke(poolc.getConnection(),args);
        } else {
            throw new SQLException("Connection has already been closed.");
        }
    }catch (Throwable t) {
        if (t instanceof InvocationTargetException) {
            throw t.getCause() != null ? t.getCause() : t;
        } else {
            throw t;
        }
    }
}
```



# 优缺点

## 优点

- 接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理

## 缺点

- 无法摆脱接口

## 注意

根类 java.lang.Object 中有三个方法也同样会被分派到调用处理器的 invoke 方法执行，它们是 hashCode，equals 和 toString 



# Read More

- sun.misc.ProxyGenerator
  - System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true") 输出生成的代理类到本地文件，反编译后，便于查看 生成的代理类 到底生成了哪些东西
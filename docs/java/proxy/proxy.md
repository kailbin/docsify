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
        new Class[]{HelloI.class},
        new HelloInvocationHandler(helloImpl)
);

helloI.say("World");
```


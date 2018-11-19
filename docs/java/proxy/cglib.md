# cglib

cglib 是针对类来实现代理的，原理是对指定的业务类**生成一个子类**，并覆盖父类(被代理类)方法实现代理。因为采用的是继承，所以不能对 final 修饰的类进行代理。  

- `cglib-nodep-x.x.x.jar`：使用no dep包不需要关联asm的jar包，jar包内部包含asm的类
- `cglib-x.x.x.jar`：使用此jar包需要关联asm的jar包，否则运行时报错
- `spring-core` 从 `3.2.0` 开始，已经将 asm 和 cglib 纳入到自己的包中，所以已无需再依赖 cglib 的包

# cglib 的使用方式

## 被代理类

不需要实现一个接口

```java
public class Hello {
    public void say(String say) {
        System.out.println("Hello " + say);
    }
}
```

## 方法拦截器

```java


import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class HelloInvocationHandler implements MethodInterceptor {

    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        try {
            System.out.println("before!!!");
            // 调用被代理的方法
            return methodProxy.invokeSuper(proxy, args);
        } finally {
            System.out.println("after!!!");
        }
    }
}
```

## 如何使用

```java
// 增强器
Enhancer enhancer = new Enhancer();
// 设置父类
enhancer.setSuperclass(Hello.class);
// 设置处理器
enhancer.setCallback(new HelloInvocationHandler(new Hello()));
// 创建代理对象 class Hello$$EnhancerByCGLIB$$xxx ex Hello impl org.springframework.cglib.proxy.Factory
Hello hello = (Hello) enhancer.create();

//调用代理对象方法
hello.say("World");
```

# 与动态代理 行为上的区别

## 动态代理 效果测试 代码

```java
public class MainProxyTest {

    public interface HelloI {
        void method1();
        void method2();
    }

    public static class HelloImpl implements HelloI {

        @Override
        public void method1() {
            System.out.println("method 111");
			// 在 method1 种调用 method2
            method2();
        }

        @Override
        public void method2() {
            System.out.println("method 222");
        }
    }

    public static class HelloProxy implements HelloI {

        private HelloI hello;

        public HelloProxy(HelloI hello) {
            this.hello = hello;
        }

        @Override
        public void method1() {
            System.out.println("start method 1");
            hello.method1();
            System.out.println("end method 1");
        }

        @Override
        public void method2() {
            System.out.println("start method 2");
            hello.method2();
            System.out.println("end method 2");
        }
    }

    public static void main(String[] args) {
        HelloProxy helloProxy = new HelloProxy(new HelloImpl());
        helloProxy.method1();
    }
}
```

## cglib 效果测试 代码

```java
public class MainCglibTest {

    public static class HelloImpl {

        public void method1() {
            System.out.println("method 111");
			// 在 method1 种调用 method2
            method2();
        }

        public void method2() {
            System.out.println("method 222");
        }
    }

    public static class HelloCglib extends HelloImpl {

        @Override
        public void method1() {
            System.out.println("<start method 1>");
            super.method1();
            System.out.println("</end method 1>");
        }

        @Override
        public void method2() {
            System.out.println("    <start method 2>");
            super.method2();
            System.out.println("    </end method 2>");
        }
    }

    public static void main(String[] args) {
        HelloCglib helloProxy = new HelloCglib();
        helloProxy.method1();
    }
}
```

## 输出结果对比

### MainProxyTest

```
<start method 1>
method 111
method 222
</end method 1>
```

### MainCglibTest

```html
<start method 1>
method 111
    <start method 2>
method 222
    </end method 2>
</end method 1>
```

## 区别说明

- 从输出结果可以看出，在嵌套调用的时候：
  - JDK动态代理 只对 被调用方法进行了增强（因为 method2 在被调用的时候，并没有通过代理对象去调用）
  - cglib代理，


 TODO 详见 Spring Core cglib

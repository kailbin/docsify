# cglib

cglib 是针对类来实现代理的，原理是对指定的业务类**生成一个子类**，并覆盖父类(被代理类)方法实现代理。因为采用的是继承，所以不能对 final 修饰的类进行代理。  

- `cglib-nodep-x.x.x.jar`：使用no dep包不需要关联asm的jar包,jar包内部包含asm的类.
- `cglib-x.x.x.jar`：使用此jar包需要关联asm的jar包,否则运行时报错.



# cglib 的使用方式

## 被代理类

不需要接口

```java
public class Hello {
    public void say(String say) {
        System.out.println("Hello " + say);
    }
}
```

## 处理器

与 动态代理 的 处理器类似，接口名都一样，只是包名不一样

```java

import net.sf.cglib.proxy.InvocationHandler;

import java.lang.reflect.Method;

public class HelloInvocationHandler implements InvocationHandler {

    private Hello hello;

    public HelloInvocationHandler(Hello hello) {
        this.hello = hello;
    }

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

## 如何使用

```java
// 增强器
Enhancer enhancer = new Enhancer();
// 设置父类
enhancer.setSuperclass(Hello.class);
// 设置处理器
enhancer.setCallback(new HelloInvocationHandler(new Hello()));
// 创建代理对象 class Hello$$EnhancerByCGLIB$$512b9276 ex Hello impl net.sf.cglib.proxy.Factory
Hello hello = (Hello) enhancer.create();

//调用代理对象方法
hello.say("World");
```

# 与动态代理 行为上的区别

```java
package test.ex;

public class MainProxyTest {

    public interface HelloI {

        void method1();

        void method2();
    }

    public static class HelloImpl implements HelloI {

        @Override
        public void method1() {
            System.out.println("method 111");

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

```java
package test.impl;

public class MainCglibTest {


    public static class HelloImpl {

        public void method1() {
            System.out.println("method 111");

            method2();
        }

        public void method2() {
            System.out.println("method 222");
        }
    }

    public static class HelloCglib extends HelloImpl {

        @Override
        public void method1() {
            System.out.println("start method 1");
            super.method1();
            System.out.println("end method 1");
        }

        @Override
        public void method2() {
            System.out.println("start method 2");
            super.method2();
            System.out.println("end method 2");
        }
    }


    public static void main(String[] args) {
        HelloCglib helloProxy = new HelloCglib();
        helloProxy.method1();
    }


}
```

# cglib 其他用法


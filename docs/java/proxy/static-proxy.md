# 静态代理



## 定义接口

```java
public interface HelloI {
    void say(String say);
}
```
## 实现类

```java
public class HelloImpl implements HelloI {

    @Override
    public void say(String say) {
        System.out.println("Hello " + say);
    }
}
```

## 代理类

```java
public class HelloProxy implements HelloI {

    private HelloI hello;

    public HelloProxy(HelloI hello) {
        this.hello = hello;
    }

    /**
     * 代理类 与 被代理类，实现相同的接口
     */
    @Override
    public void say(String say) {
        try {
            System.out.println("before!!!");
		   // 调用被代理的方法
            hello.say(say);
        } finally {
            System.out.println("after!!!");
        }
    }
}
```

## 使用

```java
// 创建 被代理类
HelloI hello = new HelloImpl();
// 被代理对象 传入 代理类
HelloProxy helloProxy = new HelloProxy(hello);
// 调用代理类的方法
helloProxy.say("World");
```



## 代理工厂

```java
public class HelloProxyFactory {

    public static HelloI getInstance() {
        return new HelloProxy(new HelloImpl());
    }
    
}
```



## 缺点

- 要代理的方法很多，要为每一种方法都进行代理，**在程序规模稍大时，静态代理无法胜任**
- 接口增加一个方法，所有代理类也需要实现此方法，**增加了代码维护的复杂度**
- **被代理类(委托类)必须是事先已经存在的，并将其作为代理对象的内部属性，**如果事先并没有被代理类（委托类），则无法进行处理
- 以上这些问题可以通过 **Java的动态代理** 来解决


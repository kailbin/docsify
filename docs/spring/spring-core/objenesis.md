
# Objenesis

Objenesis 是一个小型Java库，作用只有一个：实例化一个特定类的对象。

Java 已经支持使用 `Class.newInstance()` 动态实例化类的实例。但是类必须拥有一个合适的构造器。有很多场景下不能使用这种方式实例化类，比如：

- 构造器需要参数
- 构造器有副作用
- 构造器会抛异常

因此，在类库中经常会有类必须拥有一个默认构造器的限制。Objenesis 通过绕开对象实例构造器来克服这个限制。


实例化一个对象而不调用构造器是一个特殊的行为，然而在一些特定的场合是有用的：

- 序列化，远程调用和持久化 -对象需要实例化并存储为到一个特殊的状态，而没有调用代码
- 代理，AOP库和Mock对象 -类可以被子类继承而子类不用担心父类的构造器
- 容器框架 -对象可以以非标准的方式被动态实例化

Spring 已经将 Objenesis 纳入到了 `spring-core` 模块中，所以使用的时候不同单独引用。 Spring 在 采用 cglib 代理的时候，会默认先采用 Objenesis 实例化代理类( @see: `ObjenesisCglibAopProxy`)

## 示例

### Hello 类

``` java
public class Hello {

    private static String prefix = "Hello ";

    {
        System.out.println("do <init> Hello");
    }

    public Hello() {
        System.out.println("do init Hello");
    }

    public void say(String something) {
        System.out.println(prefix + something);
        System.out.println();
    }

}
```

### 调用方式

``` java
Hello hello = new Hello();
hello.say("World");

hello = Hello.class.newInstance();
hello.say("World");

// org.springframework.objenesis 包
SpringObjenesis objenesis = new SpringObjenesis();
ObjectInstantiator<Hello> instantiator = objenesis.getInstantiatorOf(Hello.class);
hello = instantiator.newInstance();
hello.say("World");

// sun.reflect 包
ReflectionFactory reflectionFactory = ReflectionFactory.getReflectionFactory();
Constructor<?> constructor = reflectionFactory.newConstructorForSerialization(Hello.class, Object.class.getConstructor((Class[]) null));
hello = (Hello) constructor.newInstance();
hello.say("World");
```

### 输出
``` bash
do <init> Hello
do init Hello
Hello World

do <init> Hello
do init Hello
Hello World

Hello World

Hello World
```


## Read More

- [官网](http://objenesis.org/)
- [不使用构造方法创建Java对象： objenesis的基本使用方法](https://blog.csdn.net/codershamo/article/details/52015206)
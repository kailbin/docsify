# 动态代理

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class HelloInvocationHandler implements InvocationHandler {

    /**
     * @param proxy  生成的代理对象
     * @param method 调用方法
     * @param args   方法参数
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ProxyUtil.printExtend(proxy);
        System.out.println(args[0]);
        return null;
    }
}


package proxy;

import java.lang.reflect.Proxy;
import java.util.Arrays;

public class Main {

    public static void main(String[] args) {

        /*
         * 被代理类 并不存在
         * 调用接口的方法产生的行为 在 HelloInvocationHandler 在处理器里面进行自定义
         */
        HelloI helloI = (HelloI) Proxy.newProxyInstance(
                ClassLoader.getSystemClassLoader(),
                new Class[]{HelloI.class},
                new HelloInvocationHandler()
        );

        ProxyUtil.printExtend(helloI);

        helloI.say("World");


        Class<?> proxyClass = Proxy.getProxyClass(ClassLoader.getSystemClassLoader(), HelloI.class);
    }


}


package proxy;

import java.util.Arrays;

public class ProxyUtil {

    public static void printExtend(Object obj) {
        String sj = "";
        Class<?> parentClass = obj.getClass();
        for (; null != parentClass; ) {
            System.out.println(sj + parentClass);
            parentClass = parentClass.getSuperclass();
            sj = sj + "  ";
        }

        Class<?>[] interfaces = obj.getClass().getInterfaces();
        System.out.println("接口：" + Arrays.toString(interfaces));
        System.out.println();
    }

}


# HandlerInterceptor

```java
package xyz.kail.demo.mvc.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SpringMVCInterceptor implements HandlerInterceptor {
    
      /** 
       * 该方法将在Controller处理之前进行调用，SpringMVC中的Interceptor拦截器是链式的，可以同时存在 多个Interceptor，
       * 然后SpringMVC会根据声明的前后顺序一个接一个的执行，而且所有的Interceptor中的preHandle方法都会在 Controller方法调用之前调用。
       * SpringMVC的这种Interceptor链式结构也是可以进行中断的，这种中断方式是令preHandle的返回值为false，当preHandle的返回值为false的时候整个请求就结束了。 
       */  
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        return true;
    }

    /**
     * 在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，
     * 所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。
     * postHandle 方法被调用的方向跟preHandle 是相反的，也就是说先声明的Interceptor 的postHandle 方法反而会后执行，
     */
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    /**
     * 该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。
     * 这个方法的主要作用是用于进行资源清理工作的。
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}

```

## 拦截器生效

### XML

``` xml

<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:mvc="http://www.springframework.org/schema/mvc"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
                        http://www.springframework.org/schema/beans/spring-beans.xsd  
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context.xsd  
                        http://www.springframework.org/schema/mvc  
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
     
    <mvc:interceptors>
        <!-- 直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
        <bean class="xyz.kail.demo.mvc.interceptor.AllInterceptor"/>  
        
        <mvc:interceptor>  
            <!-- 拦截login 开头的请求 -->
            <mvc:mapping path="/login/**"/>  
            <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->  
            <bean class="xyz.kail.demo.mvc.interceptor.SpringMVCInterceptor"/>  
        </mvc:interceptor>  
    </mvc:interceptors>  
</beans>
```

### Bean Config

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Component
public class AllInterceptor extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截所有
        // 排除 /login/**
        registry.addInterceptor(new SpringInterceptorAll()).addPathPatterns("/**").excludePathPatterns("/login/**");
    }
}   
```

## Read More

- [Spring中使用Interceptor拦截器](https://www.cnblogs.com/hy928302776/articles/6956747.html)
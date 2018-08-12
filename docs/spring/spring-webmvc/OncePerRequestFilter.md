
# OncePerRequestFilter

OncePerRequestFilter继承自GenericFilterBean，所以我们知道，genericFilterBean是任何类型的过滤器的一个比较方便的超类，
这个类主要实现的就是从web.xml文件中取得init-param中设定的值，然后对Filter进行初始化（当然，其子类可以覆盖init方法）。

OncePerRequestFilter继承自GenericFilterBean，那么它自然知道怎么去获取配置文件中的属性及其值，
所以其重点不在于取值，而在于确保在接收到一个request后，每个filter只执行一次，它的子类只需要关注Filter的具体实现即doFilterInternal。

`AbstractRequestLoggingFilter`是对`OncePerRequestFilter`的扩展，它除了遗传了其父类及祖先类的所有功能外，还在doFilterInternal中决定了在过滤之前和之后执行的事件，它的子类关注的是beforeRequest和afterRequest。
        
总体来说，这三个类分别执行了Filter的某部分功能，当然，具体如何执行由它们的子类规定，若你需要实现自己的过滤器，也可以根据上文所述继承你所需要的类。

``` java
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
@WebFilter(urlPatterns = "/**", filterName = "allFilter")
public class MyFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, FilterChain filterChain) throws ServletException, IOException {

        System.out.println("before");
        filterChain.doFilter(httpServletRequest, httpServletResponse);
        System.out.println("after");

    }

}

```

## Read More

- [Spring MVC过滤器-超类](http://blog.csdn.net/geloin/article/details/7441330)
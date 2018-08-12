
# WebRequestInterceptor

与`HandlerInterceptor`接口类似，区别是`WebRequestInterceptor`的`preHandle`没有返回值。
还有就是`WebRequestInterceptor`是针对请求的，接口方法参数中没有`response`。

``` java
import org.springframework.ui.ModelMap;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.context.request.WebRequestInterceptor;

public class RequestInterceptor implements WebRequestInterceptor {

    @Override
    public void preHandle(WebRequest webRequest) throws Exception {
    }

    @Override
    public void postHandle(WebRequest webRequest, ModelMap modelMap) throws Exception {

    }

    @Override
    public void afterCompletion(WebRequest webRequest, Exception e) throws Exception {

    }
}
```

## 配置生效

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
        registry.addWebRequestInterceptor(new RequestInterceptor()).addPathPatterns("/**").excludePathPatterns("/login/**");
    }
}   
```

## Read More

- [SpringMVC拦截器详解[附带源码分析]](https://www.cnblogs.com/fangjian0423/p/springMVC-interceptor.html)
# AspectJ

## jar 包

- `aspectjrt.jar` 包主要是提供运行时的一些注解，静态方法等，通常我们要使用 AspectJ 的时候都要使用这个包。
- `aspectjtools.jar` 包主要是提供赫赫有名的 `ajc` 编译器，可以在 编译期 将将java文件或者class文件或者aspect文件定义的切面织入到业务代码中。通常这个东西会被封装进各种IDE插件或者自动化插件中
- `aspectjweaver.jar` 包主要是提供了一个 `java agent` 用于在类加载期间织入切面(Load time weaving)。并且提供了对切面语法的相关处理等基础方法，供`ajc`使用或者供第三方开发使用。这个包一般我们不需要显式引用，除非需要使用 `LTW`(Load Time Weaving)

## 三种使用方式
- 编译期织入，
  - **编译时织入**，利用 ajc编译器 替代 javac编译器，直接将源文件(java或者aspect文件)编译成class文件并将切面织入进代码
  - **编译后织入**，利用 ajc编译器 向javac编译期编译后的class文件或jar文件织入切面代码
- **加载期织入**，不使用 ajc编译器，利用 aspectjweaver.jar 工具，使用 `java agent代理` 或 `ClassLoader` 在类加载期将切面织入进代码



## Spring AspectJ





























































```java
if (isAspectJWeavingEnabled(element.getAttribute("aspectj-weaving"), parserContext)) {
    if (!parserContext.getRegistry().containsBeanDefinition("context.config.internalAspectJWeavingEnabler")) {
        RootBeanDefinition def = new RootBeanDefinition("context.weaving.AspectJWeavingEnabler");
        parserContext.registerBeanComponent(
                new BeanComponentDefinition(def, "context.config.internalAspectJWeavingEnabler"));
    }

    if (isBeanConfigurerAspectEnabled(parserContext.getReaderContext().getBeanClassLoader())) {
        new SpringConfiguredBeanDefinitionParser().parse(element, parserContext);
    }
}
```








AOP (Aspect Oriented Programming 面向切面编程) 常见的实现方式有一下几种：

- **静态代理**：重用性差
- **JDK 动态代理**：运行时，面向接口
- **cglib动态代理**：运行时
- **AspectJ**：编译时（aspectjtools） 、类加载时（aspectjweaver）、运行时（aspectjrt） ，切面语法 + 织入工具
  - 基于注解，兼容 Java 语法 
  - 基于aspect文件，需要IDE的插件支持


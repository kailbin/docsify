# Selector

Selector 能够检测一个或多个 Channel，知晓通道是否为事件做好准备。这样一个单独的线程可以管理多个Channel，从而管理多个网络连接。

与 Selector 一起使用时，Channel 必须处于**非阻塞模式**下。这意味着**不能将 FileChannel 与 Selector 一起使用**，因为FileChannel 不能切换到非阻塞模式，而套接字通道都可以。

# 核心概念

## SelectionKey

SelectionKey 中四种事件常量，表示了在通过 Selector 监听 Channel 时，对什么事件感兴趣。通道触发了一个事件意思是该事件已经就绪。

- `OP_CONNECT` 某个 Channel 成功连接到另一个服务器称为**“连接就绪”**
- `OP_ACCEPT` ServerSocketChannel 准备好接收新进入的连接称为**“接收就绪”**
- `OP_READ` 有数据可读的通道是**“读就绪”**
- `OP_WRITE` 等待写数据的通道是**“写就绪”**

如果你对不止一种事件感兴趣，那么可以用“或”操作符将常量连接起来，如下：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```





# 示例

```java

```



# 参考资料

- [Java NIO 之 Selector 练习](http://blog.51cto.com/xingej/1969782)


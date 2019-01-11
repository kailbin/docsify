
# SimpleChannelInboundHandler`<I>`

``` java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    // 默认释放资源
    boolean release = true;
    try {
        // 判断 msg 的类型是否与指定的泛型类型匹配
        if (acceptInboundMessage(msg)) {
            // 匹配的话强转
            I imsg = (I) msg;
            // 强转之后由子类处理，如果子类处理完仍需要后续的 Handler 处理，
            // 需要 调用 ReferenceCountUtil.retain(imsg) 对消息的引用计数 +1（release的时候是 -1）
            channelRead0(ctx, imsg);
        } else {
            // 不匹配的话不处理、不释放
            release = false;
            ctx.fireChannelRead(msg);
        }
    } finally {
        // 默认释放资源
        if (autoRelease && release) {
            ReferenceCountUtil.release(msg);
        }
    }
}

protected abstract void channelRead0(ChannelHandlerContext ctx, I msg) throws Exception;
```
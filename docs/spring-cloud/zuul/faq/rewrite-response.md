# 如何重写响应数据



## ctx.setResponseDataStream(inputStream)

```java
InputStream stream = ctx.getResponseDataStream();

// 无论被转发的页面是什么内容，直接重写以下些字符串
String body = "Hello Zuul";

InputStream inputStream = new ByteArrayInputStream(body.getBytes(StandardCharsets.UTF_8));
ctx.setResponseDataStream(inputStream);

```

## ctx.setResponseBody(string);

```java
InputStream stream = ctx.getResponseDataStream();

// commons-io
String body = IOUtils.toString(stream, StandardCharsets.UTF_8);
body = doSomeThing(body);

ctx.setResponseBody(body);
```



## ResponseDataStream 和 ResponseBody 的区别

```java
public class SendResponseFilter extends ZuulFilter {
	
    ...
    
	@Override
	public String filterType() {
		return POST_TYPE;
	}

	@Override
	public int filterOrder() {
        // 1000
		return SEND_RESPONSE_FILTER_ORDER;
	}

	@Override
	public boolean shouldFilter() {
		RequestContext context = RequestContext.getCurrentContext();
        // 没有异常
		return context.getThrowable() == null
            	// 并且 ZuulResponseHeaders 不为空
            	// 或 ResponseDataStream || ResponseBody 不为空
				&& (!context.getZuulResponseHeaders().isEmpty()
					|| context.getResponseDataStream() != null
					|| context.getResponseBody() != null);
	}

	@Override
	public Object run() {
		...
        writeResponse();
		...
	}

	private void writeResponse() throws Exception {
		...
		OutputStream outStream = servletResponse.getOutputStream();
		InputStream is = null;
		try {
            // 如果 ResponseBody 有数据，直接把流写入 Response OutputStream
			if (RequestContext.getCurrentContext().getResponseBody() != null) {
				String body = RequestContext.getCurrentContext().getResponseBody();
				writeResponse(
						new ByteArrayInputStream(
								body.getBytes(servletResponse.getCharacterEncoding())),
						outStream);
				return;
			}
            
            // 判断是否是 gzip 压缩
			...
                
			is = context.getResponseDataStream();
			InputStream inputStream = is;
			if (is != null) {
				if (context.sendZuulResponse()) {
					if (context.getResponseGZipped() && !isGzipRequested) {
						// 对流进行 gzip 解压缩
                        ... 
					}
					else if (context.getResponseGZipped() && isGzipRequested) {
						servletResponse.setHeader(ZuulHeaders.CONTENT_ENCODING, "gzip");
					}
                    // 把流写入 Response OutputStream
					writeResponse(inputStream, outStream);
				}
			}
		} finally {
			// 关闭资源
		}
	}

    ...

}
```

从 源码 可以看出 ResponseDataStream 支持 gzip 压缩，ResponseBody 直接返回，没有进行特殊处理
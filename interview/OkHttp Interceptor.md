### OkHttp Interceptor

1.拦截器

可以监视、重写和重试call.

用来打印出去的请求和收到的响应。

```
class LoggingInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request request = chain.request();

    long t1 = System.nanoTime();
    logger.info(String.format("Sending request %s on %s%n%s",
        request.url(), chain.connection(), request.headers()));

    Response response = chain.proceed(request);

    long t2 = System.nanoTime();
    logger.info(String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));

    return response;
  }
}
```

调用chain.proceed(request)是每个拦截器实现的一个主要部分。这个简单的方法是HTTP工作发生，产出满足请求的响应之处.

2.应用拦截器     `.addInterceptor()`

* 日志拦截器

3.网络拦截器    `addNetworkInterceptor()`

4.两者的区别：

- 应用拦截器
  - 不需要关心像重定向和重试这样的中间响应。
  - 总是调用一次，即使HTTP响应从缓存中获取服务。
  - 监视应用原始意图。不关心OkHttp注入的像If-None-Match头。
  - 允许短路并不调用Chain.proceed()。
  - 允许重试并执行多个Chain.proceed()调用。
- 网络拦截器
  - 可以操作像重定向和重试这样的中间响应。
  - 对于短路网络的缓存响应不会调用。
  - 监视即将要通过网络传输的数据。
  - 访问运输请求的Connection。

5.重写请求

拦截器可以添加，移除或替换请求头，如果有请求主体，它们也可以改变。

例如，如果你连接一个已知支持请求主体压缩的网络服务器，你可以使用一个应用拦截器来添加请求主体压缩。

```
/** This interceptor compresses the HTTP request body. Many webservers can't handle this! */
final class GzipRequestInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request originalRequest = chain.request();
    if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
      return chain.proceed(originalRequest);
    }

    Request compressedRequest = originalRequest.newBuilder()
        .header("Content-Encoding", "gzip")
        .method(originalRequest.method(), gzip(originalRequest.body()))
        .build();
    return chain.proceed(compressedRequest);
  }

  private RequestBody gzip(final RequestBody body) {
    return new RequestBody() {
      @Override public MediaType contentType() {
        return body.contentType();
      }

      @Override public long contentLength() {
        return -1; // We don't know the compressed length in advance!
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
        body.writeTo(gzipSink);
        gzipSink.close();
      }
    };
  }
}
```

6.重写响应

拦截器可以重写响应头并且改变响应主体。这个通常是比重写请求头更危险的，因为它可能违背网络服务器的期望！

如果你在一个棘手的环境下并准备处理结果，重写响应头是一个强大的方式来解决问题。例如，你可以修复一个服务器未配置的Cache-Control响应头来启用响应缓存：

```
/** Dangerous interceptor that rewrites the server's cache-control header. */
private static final Interceptor REWRITE_CACHE_CONTROL_INTERCEPTOR = new Interceptor() {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Response originalResponse = chain.proceed(chain.request());
    return originalResponse.newBuilder()
        .header("Cache-Control", "max-age=60")
        .build();
  }
};
```

7.自定义拦截器

![interceptor](C:\Users\Administrator\Desktop\interceptor.png)

上图的Interceptors里传递的Request, Response可以在每一个Interceptor进行预处理，替换等。
从而可以自定义Interceptor实现Rewriting Requests（重写Request）， Rewriting Responses(重写Response)， Retrying Requests(重试Request)， Follow-up Requests等。

发现okhttp是通过执行一个List来完成整个网络请求的过程，

```
// RealCall里的getResponseWithInterceptorChain()方法，
private Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!retryAndFollowUpInterceptor.isForWebSocket()) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(
        retryAndFollowUpInterceptor.isForWebSocket()));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }

```

从上面的代码可以看出List里Interceptor的顺序，
client.interceptors() (自定义的拦截器, 在call api的前的拦截) - > retryAndFollowUpInterceptor （实现请求重试）

- > BridgeInterceptor - > CacheInterceptor - > ConnectInterceptor(真正网络连接的实现) - > client.networkInterceptors() (自定义拦截器，请求完全的拦截)

- > CallServerInterceptor

Request在Interceptor里是怎么传递的？把interceptors传递到RealInterceptorChain里，执行了chain.proceed(originalRequest).

```
//RealInterceptorChain的proceed方法
public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
     Connection connection) throws IOException {
   if (index >= interceptors.size()) throw new AssertionError();

   calls++;

   // If we already have a stream, confirm that the incoming request will use it.
   if (this.httpCodec != null && !sameConnection(request.url())) {
     throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
         + " must retain the same host and port");
   }

   // If we already have a stream, confirm that this is the only call to chain.proceed().
   if (this.httpCodec != null && calls > 1) {
     throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
         + " must call proceed() exactly once");
   }

   // Call the next interceptor in the chain.
   RealInterceptorChain next = new RealInterceptorChain(
       interceptors, streamAllocation, httpCodec, connection, index + 1, request);
   Interceptor interceptor = interceptors.get(index);
   Response response = interceptor.intercept(next);

   // Confirm that the next interceptor made its required call to chain.proceed().
   if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
     throw new IllegalStateException("network interceptor " + interceptor
         + " must call proceed() exactly once");
   }

   // Confirm that the intercepted response isn't null.
   if (response == null) {
     throw new NullPointerException("interceptor " + interceptor + " returned null");
   }

   return response;
 }

```

proceed方法里这段代码由几处的this.httpCodec != null这样的判断，httpCodec什么时候不为null? 当ConnectInterceptor执行完后就不为null了,
也就是请求已经执行了才不为null。这里对在ConnectInterceptor之后的拦截器进行了检查，request的url要一致，interceptor必须执行一次proceed()方法，否则会抛异常。
所有Interceptor的intercept()返回的Response必须不能为null。

在RealCall里的getResponseWithInterceptorChain方法里，创建了一个RealInterceptorChain对象，调用proceed(),
在proceed方法里又创建了RealInterceptorChain对象，interceptors.get(index)获取interceptor，接着调用interceptor.intercept(next)把realInterceptorChain传递下，
在interceptor的intercept()方法里又调用proceed()，明显形成了一个递归。
通过RealInterceptorChain的构造器，传递interceptors，streamAllocation，httpCodec，connection，index, request。而index每次new一个RealInterceptorChain都会加1，
是为了保证interceptors遍历。

```
RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
      HttpCodec httpCodec, Connection connection, int index, Request request)

```

这样的设计可以在Interceptor的intercept()方法里调用proceed(request,streamAllocation,httpCodec, connection)
这个方法之前替换request等参数，也可以在返回Response前替换response。灵活性就大大增加。

在ConnectInterceptor之后的拦截器必须满足：request的url要一致，interceptor必须执行一次proceed()。这样子做是为了保证递推的正常运作。
而对与client.interceptors是在ConnectInterceptor之前的拦截器，可以不用必须执行一次proceed()。可以实现直接返回虚拟的response用于是测试等功能。
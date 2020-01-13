---
title: Retrofit 几个疑问解答
date: 2019-11-25 00:01:08
tags: Android
categories: Android
---

这篇文章是关于 Retrofit 使用的几个疑问解答！

<!-- more -->

## 问题一：Retrofit的 baseUrl 必须以 / (斜线)结尾吗？

baseUrl 如果为 `https://api.github.com` 则会自动转换为 `https://api.github.com/`；如果为 `https://api.github.com/repos` 没有 / 结尾，会抛异常。

路径生成规则表格如下：

BaseUrl  | Path对应的值 |最后Url |
---|---|---
http://host:port/a/b/ | /apath | http://host:port/apath
http://host:port/a/b/ | apath | http://host:port/a/b/apath
http://host:port/a/b/ | 	http://host:port/aa/apath | http://host:port/aa/apath

## 问题二：如何在请求时使用动态URL？

可以使用拦截器和在单个请求上使用注解 `@Url` 来定义结点。例子如下：

* 案例一：

```java
Retrofit retrofit = Retrofit.Builder()  
    .baseUrl("https://your.api.url/");
    .build();

UserService service = retrofit.create(UserService.class);  
service.profilePicture("https://s3.amazon.com/profile-picture/path");

// 最后请求地址
// request url results in:
// https://s3.amazon.com/profile-picture/path
```

由于通过 `@Url` 设置了完全不同的包含 scheme 的 host （https://s3.amazon.com），Okhttp 的 HttpUrl 会解析它为我们设置的 url，所以直接忽略 base url，使用我们传入带 scheme 的地址。

* 案例二：

```java
Retrofit retrofit = Retrofit.Builder()  
    .baseUrl("https://your.api.url/");
    .build();

UserService service = retrofit.create(UserService.class);  
service.profilePicture("profile-picture/path");

// request url results in:
// https://your.api.url/profile-picture/path
```

我们请求中传入的 `@Url` 没有指定 scheme 和 host，所以最后的请求会使用我们定义的 base url 和动态结点 url 联系起来，生成一个请求地址。*具体拼接规则，请看问题一图示表格！*

* 案例三：

```java
Retrofit retrofit = Retrofit.Builder()  
    .baseUrl("https://your.api.url/v2/");
    .build();

UserService service = retrofit.create(UserService.class);  
service.profilePicture("/profile-picture/path");

// request url results in:
// https://your.api.url/profile-picture/path
```

这个例子和第二个案例不同的是，我们在 base url 的 host 后面添加了 `v2/` 和结点 url 的前面多了一个 `/`。实际上这将导致添加到 base url 后面的 `v2/`被舍弃，host 直接连接结点的 url。*具体拼接规则，请看问题一图示表格！*

## 问题三：如何在给 Retrofit 添加 header 参数？

* 方法一：方法使用注解

```java
public interface UserService {  
    // @Headers("Cache-Control: max-age=640000")
    // @Headers({"Accept: application/json", "User-Agent: Your-App-Name"})
    @GET("/tasks")
    Call<List<Task>> getTasks();
}
```

在单个请求上添加多个 header，`@Headers`注解支持传递一个或者多个 header。

* 方法二：使用拦截器

```java

OkHttpClient.Builder httpClient = new OkHttpClient.Builder();  
httpClient.addInterceptor(new Interceptor() {  
    @Override
    public Response intercept(Interceptor.Chain chain) throws IOException {
        Request original = chain.request();
        Request request = original.newBuilder()
            .header("User-Agent", "Your-App-Name")
            .header("Accept", "application/vnd.yourapi.v1.full+json")
            .method(original.method(), original.body())
            .build();
 
        return chain.proceed(request);
    }
}
 
OkHttpClient client = httpClient.build();  
Retrofit retrofit = new Retrofit.Builder()  
    .baseUrl(API_BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .client(client)
    .build();
```

使用 OkHttp 的拦截器拦截请求，给请求添加 headers。

* 方法三：函数参数使用注解，动态 Header

```java

public interface UserService {  
    @GET("/tasks")
    Call<List<Task>> getTasks(@Header("Content-Range") String contentRange);
}
```

给单个请求动态添加 header。

> 其中方法一和方法三很相似，为什么还要提供方法三呢？举个例子，例如我只想给某个请求的 header 添加一个 key 为 `sso`，value 为用户登录 token 时，方法一就没有办法做到，在函数注解上无法使用表达式，而用方法三就很方便，通过参数传递进去就可以了！


## 参考

* [Retrofit2 的baseUrl 真的必须以 /（斜线） 结尾吗？](https://www.jianshu.com/p/d6b8b6bc6209)
* [Retrofit2-如何在请求时使用动态URL](https://www.jianshu.com/p/4268e434150a)
* [Retrofit 2 — How to Use Dynamic Urls for Requests](https://futurestud.io/tutorials/retrofit-2-how-to-use-dynamic-urls-for-requests)
* [Github/以最简洁的 Api 让 Retrofit 同时支持多个 BaseUrl 以及动态改变 BaseUrl](https://github.com/JessYanCoding/RetrofitUrlManager)
* [Retrofit添加header参数的几种方法](https://blog.csdn.net/zhuhai__yizhi/article/details/52956719)


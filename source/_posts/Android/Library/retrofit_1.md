---
layout: post
title: Retrofit开发-1-基础
categories:
  - Android
  - Library
tags:
  - Android
  - Library
  - Retrofit
keywords:
  - Android
  - Retrofit
  - OkHttp
comments: true
abbrlink: e57450c6
date: 2017-07-01 14:31:11
password:
---

`Retrofit`

本文介绍 `OkHttp` 和 `Retrofit` 的基本使用，包括：

> 创建和配置 `OkHttpClient`

> 创建和配置 `Retrofit`

> 如何使用 `Retrofit` 定义接口发起请求

> 关于 `Retrofit` 中 `method`，`path`，`query param`，`body`，`Header` 注解声明的介绍。

<!--more-->


## 添加依赖

```gradle
// 网络请求
okhttp3                 : 'com.squareup.okhttp3:okhttp:3.4.2',
// 单元测试，未用
okhttp3Mockwebserver    : 'com.squareup.okhttp3:mockwebserver:3.4.2',
// 调试工具类库
stetho                  : 'com.facebook.stetho:stetho:1.4.2',
stetho_okhttp3          : 'com.facebook.stetho:stetho-okhttp3:1.4.2',
// json解析
gson                    : 'com.google.code.gson:gson:2.6.1',
// retrofit 2.0
retrofit                : 'com.squareup.retrofit2:retrofit:2.3.0',
converter_gson          : 'com.squareup.retrofit2:converter-gson:2.1.0',
// 日志打印
logging_interceptor     : 'com.squareup.okhttp3:logging-interceptor:3.8.0',
```

## 创建 OkHttp

`Retrofit` 是对 `OkHttp` 的封装，提供了使用注解更简单的构建各种请求，配置各种参数的方式。本质发起网络请求的还是 `OkHttp`，但 `Retrofit` 让这一操作更加的简单优雅。

创建一个 `OkHttpClent` 并进行简单配置


```java
/**
 * 创建 OkHttpClient
 *
 * @return OkHttpClient
 */
private OkHttpClient buildOkHttpClient() {
    OkHttpClient.Builder okHttpBuilder = new OkHttpClient.Builder();
    // 连接超时
    okHttpBuilder.connectTimeout(5 * 1000, TimeUnit.MILLISECONDS);
    // 读超时
    okHttpBuilder.readTimeout(5 * 1000, TimeUnit.MILLISECONDS);
    // 写超时
    okHttpBuilder.writeTimeout(5 * 1000, TimeUnit.MILLISECONDS);
    // 清除 interceptors
    okHttpBuilder.interceptors().clear();
    okHttpBuilder.networkInterceptors().clear();
    // 自定义 Interceptor，用来添加全局 Header
    okHttpBuilder.addNetworkInterceptor(new HeaderInterceptor());
    // 自定义 Interceptor，进行日志打印，扩展自 HttpLoggingInterceptor
    okHttpBuilder.addNetworkInterceptor(new LogInterceptor(LogInterceptor.Level.BOTH));
    // 失败后重试
    okHttpBuilder.retryOnConnectionFailure(true);
    // face book 调试框架
    okHttpBuilder.addNetworkInterceptor(new StethoInterceptor());
    // token校验，返回 403 时
    okHttpBuilder.authenticator(new TokenAuthenticator());
    return okHttpBuilder.build();
}
```


## 创建 Retrofit
`OkHttp` 只负责发起网络请求，维护网络连接等操作，而 `Retrofit` 帮我们将网络传输的数据转换为可用的 `model` 对象，并且提供简单的数据处理方式。

配置 `CallAdapterFactory` 为 `RxJava2CallAdapterFactory.createWithScheduler(Schedulers.io())` 用来将返回的数据转换为 `RxJava` 的 `Observable` 对象，但是本文还没涉及到 `RxJava` 的部分。

配置 `ConverterFactory`  为 `GsonConverterFactory.create(new Gson())` 用来使用 `GSON` 将网络数据序列化为可用对象。

ps：`baseUrl` 要求以 `/` 结尾，而请求的 `path` 要求不以 `/` 开头，也就是说我们需要类似 `https://www.baidu.com/` 的形式。

```java
/**
 * 创建 retrofit
 *
 * @param okHttpClient client
 * @return retrofit
 */
private Retrofit buildRetrofit(OkHttpClient okHttpClient) {
    final Retrofit.Builder retrofitBuilder = new Retrofit.Builder();
    // client
    retrofitBuilder.client(okHttpClient);
    // baseUrl
    retrofitBuilder.baseUrl(BASE_URL);
    // rxJava 调用 adapter
    retrofitBuilder.addCallAdapterFactory(
            RxJava2CallAdapterFactory.createWithScheduler(Schedulers.io()));
    // 数据转换 adapter
    retrofitBuilder.addConverterFactory(GsonConverterFactory.create(new Gson()));
    return retrofitBuilder.build();
}
```

## 创建 api 服务接口
创建接口类，使用注解声明网络请求的方法和相关参数，这里只是一个简单的例子，更多参数和配置将在后面说明，如：

```java
public interface ApiService {

    String GET_USER_INFO = "api/v1/user/info";
	 
    @GET(GET_USER_INFO)
    Call<UserInfoDTOResponse> getUser(@Header("Authorization") String auth);
}
```
使用 `Retrofit` 初始化 `ApiService`，获取到 `ApiService` 的实例之后就可以使用该实例发起请求。

我使用 `ApiRequest` 持有 `ApiService` 的实例，同时 `ApiRequest` 是一个单例，作为发起网络请求的管理者。

```java
private void createService() {
    Retrofit retrofit = buildRetrofit(buildOkHttpClient());
    mApiService = retrofit.create(ApiService.class);
}
```
发起请求

```java
public static void getUserInfo() {
    // 从单例中获取 ApiService
    ApiService service = ApiRequest.getService();
    // 创建请求
    Call<UserInfoDTOResponse> userCall = service.getUser();
    // 发起请求
    userCall.enqueue(new Callback<UserInfoDTOResponse>() {
        @Override
        public void onResponse(@NonNull Call<UserInfoDTOResponse> call,@NonNull Response<UserInfoDTOResponse> response) {
            UserInfoDTOResponse body = response.body();
            if (body != null && body.getData() != null) {
                L.e(TAG, body.getData().toString());
            }
        }
        @Override
        public void onFailure(@NonNull Call<UserInfoDTOResponse> call,@NonNull Throwable t) {
        }
    });
}
```

---

下面对 `Retrofit` 中 `method`,`path`,`qurey`,`body`,`header`几个部分分别说明。


## Method

`Retrofit` 支持 `GET`，`POST`，`PUT`，`DELETE`，`HEAD`，`OPTION` 等请求方式，并有对应的注解。


```java
@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser();

@DELETE(DELETE_RM_BABY)
Call<BaseResponse> deleteBaby(@Path(PATH_BABY_ID) Long babyId);
```

## PATH

请求路径，请求路径中可以包含参数，并在参数中使用 `@PATH` 注解来动态改变路径，
比如路径 `api/v1/user/{id}`，使用注解 `@PATH("id") Long id` 即可改变路径中 `{id}` 请求时的值。

```java
//  path 中不包含参数
@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser();


String PATH_BABY_ID = "babyId";
String PUT_EDIT_BABY = "api/v2/baby/doEdit/{" + PATH_BABY_ID + "}";

@PUT(PUT_EDIT_BABY)
Call<BaseResponse> putEditBaby(@Path(PATH_BABY_ID) Long babyId);
```

## Body

发送 `POST` 、`PUT` 请求时通常需要携带 `body` 数据，使用 `@Body` 注解，如下：

```java
@PUT(PUT_EDIT_BABY)
Call<BaseResponse> putEditBaby(@Body BabyParam babyParam);
```

## Query

配置 `UrlQuery` 参数，比如 `api/v1/user/list?limit=100&offset=10` 这种类型。有两种配置方式：

第一种使用 `@Query` 注解，如 `@Query("userId") Long userId` 的形式。这种形式可以传递 `null` 值，如果某个参数为 `null`，将不会拼接在 `url` 后面。

第二种使用 `@QueryMap` 注解，如 `@QureyMap Map<String,String> params`  的形式。这种形式传递一个 `map` 作为参数，但是 `map` 中 `value` 不能为 `null`，否则会抛出异常。

```java
@GET(GET_BABY_RELATION_LIST)
Call<UserBabyRelationResp> getBabyRelationList(@Query("babyId") Long babyId, @Query("limit") int limit, @Query("offset") Long offset);

@GET(GET_BABY_RELATION_LIST)
Call<UserBabyRelationResp> getBabyRelationList(@QueryMap Map<String,String> map);
```

## Header

静态 `Header`，使用 `@Headers` 注解添加在请求接口上，用法如下：

```java
@Headers({"key:value","key:value"})
@Headers("key:value")

eg:
@Headers({"Authorization: authValue",
                 "User-Agent: Retrofit-Sample-App"})
@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser();

@Headers("Authorization: authValue")
@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser();
```

局部动态 `Header`，使用 `@Header("key")` 和 `@HeaderMap` 注解添加在请求参数上，动态配置，用法如下：

```java
@Header("Authorization") String auth
// retrofit:2.1.0 添加
@HeaderMap Map<String, String> headers

eg:
@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser(@Header("Authorization") String auth);

@GET(GET_USER_INFO)
Call<UserInfoDTOResponse> getUser(@HeaderMap Map<String, String> headers);
```

全局配置 `Header`，为 `OkHttp` 添加 `Interceptor`，需要注意的是 `addNetWorkIntercepror()` 和 `addInterceptor()` 的区别。当进入 `netWorkInterceptor` 时已经添加了 `User-Agent` 等 `Header`，此时要把自己的 `Header` 一个一个 `addHeader()` 进去，防止把原来的冲掉。

```java
public final class HeaderInterceptor implements Interceptor {

    public static final String TAG = HeaderInterceptor.class.getSimpleName();

    // 到达 netWorkInterceptor 时，默认 header 已经添加，不能使用替换的方式，要使用 addHeader 的方式
    // 到达 interceptor 时，还没有添加默认 header ,可以直接替换原来的 header ，后面会追加默认 header
    
    @Override
    public Response intercept(@NonNull Chain chain) throws IOException {
        Request request = chain.request();
        L.e(TAG, "添加全局 header ");
        Request.Builder builder = request.newBuilder();
        Map<String, String> headersMap = ApiRequest.getHeadersMap();
        for (String key : headersMap.keySet()) {
            builder.addHeader(key, headersMap.get(key));
        }
        Response response;
        try {
            response = chain.proceed(builder.build());
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        }
        return response;
    }
}

OkHttpClient.Builder okHttpBuilder = new OkHttpClient.Builder();
okHttpBuilder.addNetworkInterceptor(new HeaderInterceptor());
```


## Todo

更多内容还没有涉及...

```java
// 表单提交
@FormUrlEncoded
@POST("user/edit")
@HEAD
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);


// 文件上传
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```

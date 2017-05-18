---
title: QuickAF网络连接及数据解析简介
date: 2017-05-12 10:16:41
tags: [Android, QuickAF]
category: [Android]
toc: true
---

[QuickAF]中使用Volley进行网络连接，使用Gson来解析响应数据。为了更方便地执行REST API网络请求，[QuickAF]对Volley+Gson进行了简单的封装。

<!-- more -->

## 接口请求与响应设计

### 接口
REST接口是基于HTTP协议的，一个接口的定义包含请求地址，请求方法，请求参数，响应信息。请求地址为一个URL，由基地址和接口路径和查询字符串组成。比如http://127.0.0.1:8080/meituan/api/1.0/user/login?token=xxxxxxx; http://127.0.0.1:8080/meituan/api/1.0/ 为基地址，一套api，其基地址是相同的。1.0为接口版本，user/login为接口路径，token=xxxxxx为查询字符串。请求方法有GET/POST/PUT/DELETE等。[QuickAF]将接口地址抽象为IUrl接口
```java
public interface IUrl {
    /**
     * return http method, see {@link com.android.volley.Request.Method}
     *
     * @return http method
     * @see com.android.volley.Request.Method
     */
    int getMethod();

    /**
     * return the full url
     *
     * @return
     */
    String getUrl();

    /**
     * set query string to url.
     *
     * @param query query string, it a request parameter of string format usually
     */
    void setQuery(String query);
}
```

### 请求
一般来说接口的请求比较简单，如果请求是application/json，将请求对象转为json字符串即可。但是实际当中，仍然有许多接口使用的还是application/x-www-form-urlencoded，这种方式简单，而且适用于网页。[QuickAF]默认以后者来提交http请求，并且支持以下两种请求格式
1. 键值对，这也是早期使用最多的，通过Map来存储请求参数。
2. 对象，通过反射机制将对象的属性及属性值转化对键值对，具有很高的可扩展性，一旦接口有变更，比如接口要求添加uuid参数，可以非常方便的修改请求基类来满足业务需求，[QuickAF]建议使用这种方式来封装请求。

通常在REST API中包含appKey, secret, uuid等全局请求参数，[QuickAF]的sample app中定义的请求基类如下：
```java
public class BaseRequest implements java.io.Serializable {
    public String appKey;
    public String secret;
    public String version;
    public String uuid;
}
```
具体的业务API只需要继承BaseRequest，然后添加具体的业务请求参数，比如注册的请求
```java
public class RegisterRequest extends BaseRequest {
    public String phone;
    public String code;
    public String password;
}
```
对于GET请求，将请求对象转为查询字符串附在url中，对于POST请求，则将请求对象写入body中。

### 响应
REST API接口响应一般包含状态码(status)，提示信息(message)及业务对象(data)，需要经过json工具将其转为对象，这个对象我们姑且称之为接口对象。伪代码如下：
```java
class MyResponse {
  int code;//业务响应状态码
  String message;//业务响应信息，比如投票失败
  Object data;//业务响应对象，比如登录，返回的是一个User对象
}
```

其实业务模块往往关心的只有业务对象(data)，因为对于业务操作不成功的处理，可以在基类中统一处理。在[QuickAF]中，将接口对象抽象为IBaseResponse接口。
```java
public interface IBaseResponse<Output> extends java.io.Serializable{
    /**
     * Return the business data object
     *
     * @return concrete business data
     */
    Output getData();
}
```

### 请求任务
如果我们将请求视为输入，响应视为输出，那么对于一次网络请求，使用代码实现的话，就是：
```java
abstract class MyTask<Input,Output> {
  void onSuccess(Output output); // 业务请求成功
  void onError(RestError error);//业务请求失败
  Url getUrl();//业务请求地址
}
```
对于每个业务接口，[QuickAF]已经为您做好了网络连接与数据解析，对于开发者来说，只需关心接口地址，请求对象，响应对象。业务请求成功，在相关的界面填充数据，请求失败，给出相应的错误提示。
[QuickAF]有两个执行任务的方法
1. 如果输出为对象(Output)是一个对象，则需调用load方法，将Output的class传进去。
2. 如果输出为集合(List)，则需调用load2List方法，将集合中的元素class传进去。
```java
class MyTask<Object, User> {
  ...
}
//
new MyTask().load(null, User.class, false);
// for List
class MyListTask<Object, List<User>> {
  ...
}
new MyListTask().load2List(null, User.class, false);
```


## 进阶操作
### 配置接口对象
接口对象，一个app，一般只有一个。定义如下
```java BaseResponse.java
public class BaseResponse<Output> implements IBaseResponse {
    private static final long serialVersionUID = -3440061414071692254L;

    /**
     * 状态码
     */
    public int code;

    /**
     * 消息
     */
    public String message;

    /**
     * 数据，业务对象
     */
    public Output data;
    public Output getData() {
      return data;
    }
}
```
然后可以在Application.onCreate()中配置。
```java
// typically, you just config volley in Application.onCreate
VolleyConfig config = new VolleyConfig.Builder()
    .setBaseResponseClass(WeatherBaseResponse.class)
    .build();
VolleyManager.init(getApplicationContext(), config);
```
如果有喜欢使用OkHttp的同学，还可配置网络连接使用OkHttp。

### 接口统一处理
主要是根据接口业务状态码进行处理。比如定义业务操作成功，响应码为0，那么不为0的时候，就不应该解析业务对象，转入错误分支。
```java
protected abstract class AppBaseTask<Input, Output> extends RequestObjectTask<Input, Output> {

    @Override
    public boolean onInterceptor(IBaseResponse response) throws Exception {
        if (response instanceof BaseResponse) {
            BaseResponse resp = (BaseResponse) response;
            if (0 != resp.code) {
                onLogicError(new LogicError(null, resp.code, resp.message));
                throw new InterceptorError();
            }
        }
        return false;
    }

    public void onLogicError(LogicError error) {
        if (404 == error.getCode() || 104 == error.getCode()) { {
            // LoginActivity.go(MyApplication.instance);
            return;
        }
        onError(new RestError(error));
    }
}
```
### 数据模拟
接口对象或业务对象类需在mock()方法中给对象填充模拟值。

## 最佳实践
- 所有的请求继承一个BaseRequest，接口定义的全局请求参数在BaseRequest中定义
- 一套接口API，定义一个全局的AppController及AppBaseTask来处理公共的业务，比如业务拦截。
- 所有的业务模型继承一个BaseInfo
- 一个Controller对应一个界面，应继承AppController，包含若干网络请求Task
- 网络请求Task回调作为内部interface定义在Controller中。

更多请参考demo app工程

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF

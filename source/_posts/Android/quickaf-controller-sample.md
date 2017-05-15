---
title: QuickAF解析REST API综合示例
date: 2016-08-12 10:16:41
tags: [Android, QuickAF]
category: [Android]
toc: true
---

{% asset_img weather.png %}

在[QuickAF]中使用基于Volley的网络数据连接框架。能够方便地执行REST API网络请求，并与界面进行交互。在本文中通过一个天气查询的综合示例来讲解[QuickAF]中如何进行网络请求。

<!-- more -->

## 框架设置
在使用网络数据框架之前，须先配置，一般在Application.onCreate()中配置，当然，也可以使用前配置，比如Activity.onCreate()

```java
// typically, you just config volley in Application.onCreate
VolleyConfig config = new VolleyConfig.Builder()
    .setBaseResponseClass(WeatherBaseResponse.class)
    .build();
VolleyManager.init(getApplicationContext(), config);
```
如果有喜欢使用OkHttp的同学，还可配置网络连接使用OkHttp。

### 接口响应对象
`WeatherBaseResponse.class`为接口返回对象class，一般一套接口标准中只有唯一的一个。接口返回的对象一般包含状态码(status)，提示信息(message)及业务对象（data）。对于业务模块，只需要关心业务对象(data)即可，框架中还可以定义拦截器，如果接口请求成功，业务操作不成功，则不会进入业务逻辑层。

## 接口API
在这里，使用的是[百度api store中的天气api](http://apistore.baidu.com/apiworks/servicedetail/112.html)，其中了其中的两个API
1. 查询城市可用列表，通过输入的城市名称，查询该城市下的城市（或区）列表。
2. 查询指定城市（或区）的天气详情

### 模型
根据接口API，先把模型定义出来

- 查询城市列表请求模型`BaseRequest`
非常简单，只有一个cityname的请求参数
```java
public class BaseRequest implements Serializable {
    public String cityname;
}
```
- 查询城市列表响应 `List<CityInfo>`
```java
public class CityInfo implements Serializable {
    public String province_cn;
    public String district_cn;
    public String name_cn;
    public String name_en;
    public String area_id;
}
```
- 查询城市天气详情请求 `CityInfo`，与查询城市列表响应返回的列表项一样
- 查询城市天气详情响应 `WeatherInfo`
```java
public class WeatherInfo implements Serializable {
    public String city;
    public String pinyin;
    public String citycode;
    public String date;
    public String time;
    public String postCode;
    public double longitude;
    public double latitude;
    public String altitude;
    public String weather;
    public String temp;
    public String l_tmp;
    public String h_tmp;
    public String WD;
    public String WS;
    public String sunrise;
    public String sunset;
}
```

## Controller
[QuickAF]中通过Controller来执行API请求，一般来说，一个界面对应一个Controller，在Controller中提供了页面操作需要调用的接口API。在Controller内部，每个网络请求都是一个Task。所有的Controller须继承自`cn.ieclipse.af.volley.Controller`。通常，一个App只有一套接口API，所以，可以定义一个`AppController`继承自`cn.ieclipse.af.volley.Controller`。然后各业务模块的Controller继承`AppController`并向外提供一个回调Listener。

### Task
Task是Controller中的内部类，用于执行网络请求。可以`AppController`中定义一个`AppTask`并继承自父Controller中的`RequestObjectTask`，来执行一些常用的操作，比如API调用错误处理，给接口API统一添加appKey等http头等。这样子各个业务模块只需专注模块业务即可。
Task的调用有两种方式
- `load(Input input, Class<Output> clazz, boolean needCache)`，接口返回为对象，如果模型不匹配，编译时将会报错。
- `load2List(Input input, Class<?> itemClass, boolean needCache)`，接口返回为数组，此种方式，需要指定List集合中的模型Class，如果错了，将导致json无法解析为正确的模型，在运行时会报错

### Listener
回调Listener必须在Controller中定义，通常回调Listener需提供两个抽象方法

- onXXXSuccess(response model)
当接口API调用成功时的回调，接口API的响应在参数中
- onXXXFailure(error object)
当接口API调用失败时的回调，参数为错误对象，比如网络错误等。

***注：回调接口中的方法可以在UI线程中调用***

### 缓存
在Task中可以设置是否允许缓存，相应地，在接口回调中，会标识此次接口调用是来自于缓存还是实时的接口响应。在[QuickAF]中，有自己的API缓存，与Volley不太一样，主要是在国内我们的大后端，一般不会在HTTP中控制缓存（反正我工作快10年了，没有见过一个后端这么做）。

### 实例
下面来看看`WeatherController`。
```java
public class WeatherController extends AppController<WeatherController.WeatherListener> {

    public WeatherController(WeatherListener l) {
        super(l);
    }

    public void loadCityList(BaseRequest req, boolean needCache) {
        CityListTask task = new CityListTask();
        task.load2List(req, CityInfo.class, needCache);
    }

    public void loadWeather(BaseRequest req) {
        CityWeatherTask task = new CityWeatherTask();
        task.load(req, WeatherInfo.class, false);
    }

    private class CityListTask extends AppBaseTask<BaseRequest, List<CityInfo>> {

        @Override
        public IUrl getUrl() {
            return new URLConst.AbsoluteUrl("http://apis.baidu.com/apistore/weatherservice/citylist").get();
        }

        @Override
        public void onSuccess(List<CityInfo> out, boolean fromCache) {
            mListener.onLoadCityListSuccess(out, fromCache);
        }

        @Override
        public void onError(RestError error) {
            mListener.onLoadCityListFailure(error);
        }

        @Override
        public boolean onInterceptor(IBaseResponse response) throws Exception {
            if (response instanceof WeatherBaseResponse) {
                WeatherBaseResponse resp = (WeatherBaseResponse) response;
                if (resp.errNum != 0) {
                    throw new LogicError(null, String.valueOf(resp.errNum), resp.errMsg);
                }
            }
            return false;
        }

        @Override
        protected GsonRequest buildRequest(IUrl url, String body) {
            GsonRequest request = super.buildRequest(url, body);
            request.addHeader("apikey", "e8c043231152d9cbcf30a648382ca4c5");
            return  request;
        }
    }

    private class CityWeatherTask extends AppBaseTask<BaseRequest, WeatherInfo> {

        @Override
        public IUrl getUrl() {
            return new URLConst.AbsoluteUrl("http://apis.baidu.com/apistore/weatherservice/cityname").get();
        }

        @Override
        public void onSuccess(WeatherInfo out, boolean fromCache) {
            mListener.onLoadWeatherSuccess(out, fromCache);
        }

        @Override
        public void onError(RestError error) {
            mListener.onLoadWeatherError(error);
        }

        @Override
        public boolean onInterceptor(IBaseResponse response) throws Exception {
            if (response instanceof WeatherBaseResponse) {
                WeatherBaseResponse resp = (WeatherBaseResponse) response;
                if (resp.errNum != 0) {
                    throw new LogicError(null, String.valueOf(resp.errNum), resp.errMsg);
                }
            }
            return false;
        }

        @Override
        protected GsonRequest buildRequest(IUrl url, String body) {
            GsonRequest request = super.buildRequest(url, body);
            request.addHeader("apikey", "e8c043231152d9cbcf30a648382ca4c5");
            return  request;
        }
    }

    public interface WeatherListener {
        void onLoadCityListSuccess(List<CityInfo> out, boolean fromCache);

        void onLoadCityListFailure(RestError error);

        void onLoadWeatherSuccess(WeatherInfo out, boolean fromCache);

        void onLoadWeatherError(RestError error);
    }
```

## 界面
终于到界面了，先看看`WeatherActivity`的代码
```java
public class WeatherActivity extends BaseActivity implements WeatherController.WeatherListener {

    TextView tv;
    EditText et;
    Spinner spn;
    CityAdapter adapter;
    WeatherController controller;

    @Override
    protected int getContentLayout() {
        return R.layout.sample_activity_volley_weather;
    }

    @Override
    protected void initHeaderView() {
        super.initHeaderView();
        setTitle("Weather Sample");
    }

    @Override
    protected void initContentView(View view) {
        super.initContentView(view);
        // typically, you just config volley in Application.onCreate
        VolleyConfig config = new VolleyConfig.Builder().setBaseResponseClass(WeatherBaseResponse.class).build();
        VolleyManager.init(getApplicationContext(), config);
        spn = (Spinner) findViewById(R.id.spn1);
        adapter = new CityAdapter();
        spn.setAdapter(adapter);
        et = (EditText) findViewById(R.id.et_text);
        tv = (TextView) findViewById(R.id.tv);
    }

    @Override
    protected void initData() {
        super.initData();
        controller = new WeatherController(this);
        String name = et.getText().toString();
        if (TextUtils.isEmpty(name)) {
            name = et.getHint().toString();
        }
        loadCityList(name, true);
    }
```

`WeatherActivity`实现了`WeatherController.WeatherListener`回调接口，在初始化时，调用了`loadCityList`来获取城市列表。

下面再看4个跟接口API相关的方法。

```java
    /**
     * 获取城市列表
     * @param name
     * @param needCache
     */
    public void loadCityList(String name, boolean needCache) {
        BaseRequest req = new BaseRequest();
        req.cityname = name;
        controller.loadCityList(req, needCache);
    }

    /**
     * 获取城市天气详情
     */
    public void loadWeather() {
        BaseRequest req = new BaseRequest();
        CityInfo city = (CityInfo)spn.getSelectedItem();
        if (city != null) {
            req.cityname = city.name_cn;
            controller.loadWeather(req);
        }
    }

    @Override
    public void onLoadCityListSuccess(List<CityInfo> out, boolean fromCache) {
        adapter.setDataList(out);
        adapter.notifyDataSetChanged();
    }

    @Override
    public void onLoadCityListFailure(RestError error) {
        VolleyUtils.showError(tv, error);
    }

    @Override
    public void onLoadWeatherSuccess(WeatherInfo out, boolean fromCache) {
        String msg = String.format("city:%s\ntemp:%sC (%s - %s)\nwind:%s(%s)", out.city, out.temp, out.l_tmp, out.h_tmp,
            out.WD, out.WS);
        tv.setText(msg);
    }

    @Override
    public void onLoadWeatherError(RestError error) {
        VolleyUtils.showError(tv, error);
    }
```

在回调onXXXSuccess方法中，将模型设置到界面控件中以显示在UI中。

## 总结
使用[QuickAF]请求REST API非常的方便，开发相当快，重点步骤如下

1. 设置，设置好接口返回大对象IBaseResponse, AppController，这个一般在工程初始化中做。
2. 根据接口API生成模型，可以通过GsonFormat之类的工具来生成
3. 编写Controller，根据模块业务编写对应的Controller，一个API对应一个Task。
4. 界面中实现Controller，在适当的地方比如点击按钮调用Controller中的方法，在回调方法中处理业务逻辑。

本示例中的所有代码，可以访问：https://github.com/Jamling/QuickAF/tree/master/sample/src/main/java/cn/ieclipse/af/demo/sample/volley/weather 查看。

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF

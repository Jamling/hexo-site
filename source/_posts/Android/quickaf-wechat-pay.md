---
title: 谈谈那些年微信支付踩过的坑
date: 2017-05-24 10:00:00
tags: [Android, QuickAF]
category: [Android]
toc: true
description: Android微信支付大全,微信支付踩的坑,微信支付常见问题
---

很早的时候就想写这篇文章了，作为BAT中的一员，还真不想吐槽它，免得被人身攻击。有人说，微信支付很简单嘛，官网有例子，网上也有现成的例子，不过谁用谁知道，本人也是在深入了解之后，真心觉得微信支付里的坑太多，BAT的开发们太敷衍了事，结果给不少的其他开发者带来诸多麻烦。我在这里做个稍全一点的介绍，尽量减少其他同学们掉坑里的概率。

<!-- more -->

## 在微信上创建你的应用
这里特别强调一下，这一步很重要，不然微信支付集成调试会出现莫名的错误。

1，在注册之前对于Android客户端，需要提供app应用的包名和应用签名（md5值），这两个东西问开发或产品同学要。尽量在注册前提供。另外还需准备一个28x28和108x108的logo图片，问设计或产品同学要。
2，在[微信开放平台]上注册，注意了，是微信开放平台，不是公众号平台，公众号平台账号不能在开放平台登录。如果已经注册过，请直接登录，登录成功后，点击创建移动应用，创建Android应用时，`应用包名`和`应用签名`，尽量一次性填对了。应用创建成功后，将得到`appid`(以wx开头的一串数字)。然后再去申请微信支付（需做开发者认证并缴费）
3，在[微信支付商户平台]注册或登录，申请app支付，申请通过后，将得到商户id`mch_id`(一串10位数字)，然后在*账户设置-->API安全-->密钥设置*中设置API密钥`key`(32位的字串)

更多的申请帮助，请参考：https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317780&token=6c7b59a75b08969c15fa41141ee8d88c236f01ab&lang=zh_CN

注：关于微信支付申请，请在开放平台申请（同公众号支付申请流程，公众号支付不在本文讨论范围之内）微信的工作人员审核通过后，会发审核通过的邮件。里面包含*微信支付商户号*`mch_id`(一串10位数字)和APPID等信息。然后根据邮件提示或者直接在*微信商户平台中-->账户设置-->API安全-->密钥设置*中下载api证书并设置API密钥`key`(32位的字串)

在微信开放平台创建应用成功后，APP支付也申请通过了。请提供给开发同学以下东西：

- APPID `appid`(以wx开头的一串数字)
- 商户id `mch_id`(一串10位数字)
- API密钥 `key` (32位的字串)
对于Android应用，还需要保证**应用包名**与**应用签名**正确

只要上面的信息正确无误，下面就交给开发同学了。如果是直接使用微信支付sdk的同学，请准备好踩坑吧。

## 支付SDK和demo
### 微信支付SDK
SDK从4.0.2开始，已改为使用gradle方式。
```
dependencies {
   compile 'com.tencent.mm.opensdk:wechat-sdk-android-without-mta:+'
}
```
以前是直接使用libammsdk.jar，相比起来，算是进步了。

### demo
我建议直接忽略官方的demo，不信我的可以直接去[Android资源下载](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167&token=&lang=zh_CN)下载相关demo

以下是我踩过的坑：

- 支付Demo在另外一个单独的project中，尽管sdk demo与pay demo非常类似，搞不懂为啥不合为一个demo project。我昨天在微信网站下载的支付demo，今天在微信网站上就找不着了。昨天下载的支付demo，看版本是v3(sdk demo已经是5.0.2了)，还是eclipse工程，不过导入到eclipse之后，编译不通过。sdk中原来的com.tencent.mm.sdk包换成了com.tencent.mm.opensdk，demo，导致src代码一片红。
- 支付Demo中的服务端下单接口地址(http://wxpay.weixin.qq.com/pub_v2/app/app_pay.php?plat=android) 不能访问，就算修复了编译错误，结果仍然运行不了。
然后我看了下其它的文档，sdk换了，但是相关的文档并没有更新，尤其是混淆配置，直接影响到联调，这也是一个隐藏的坑。

## 统一下单
[统一下单]是商户系统（客户端或服务端）向微信支付后台发送请求，以拿到预支付交易会话标识(`prepayid`)等信息。关于接口的请求与响应信息，请参考[统一下单]接口描述。文档写得还是蛮全的，不过，实际上如何，我就只能呵呵了。请看下面

- 接口的请求与响应都是xml格式，xml的解析不太方便，微信仅支持这个格式。算是小坑。
- 接口文档中定义了一个`NOT_UTF8`的错误码，似乎微信要求的请求编码为utf-8，不过使用官方demo的Util.httpPost时，如果参数中有中文，还需将请求的xml转为ISO-8859-1编码的字符串才行。不然，微信直接返回给你一个空字符串，保证让你找不着北。
- 接口文档中定义了一个`LACK_PARAMS`的错误码，指的是如果必填的参数为空，会返回此错误码，可事实上呢，有一次我把total_fee参数搞混了，写成了支付宝的参数名，结果接口返回一个空串，结果让我一顿好找。
- Android从6.0开始，删除了apache的http组件，于是乎我把Util中的apache http组件换成了HttpUrlConnection，关于请求头，解析都和原来保持一致，结果返回个签名错误。要不是我换http组件之前可以成功下单，我几乎就信了它，真的去查签名了(事实上根本不是签名错误好吧)。后来我修改http的请求头，尝试各种请求方式与编码，结果还是返回签名错误，这让我很是折腾，一度曾想改回apache的http client。最后，我把请求的xml转为utf-8编码，然后HttpUrlConnection全都默认设置，这才下单成功。（PS，这里顺带讲一个HttpUrlConnection的坑，设置请求方法为post，可是debug中看到的HttpUrlConnection对象，请求方法仍为get，当初还以为是这个导致的，后来才知道HttpUrlConnection真正的实现是在delegate中）

总结一下，[统一下单]的响应、错误码和错误描述比较混乱，成功响应还好，一旦出现问题，准让你摸不清东南西北。

签名这块，请参考[签名算法](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)，注意按字母升序组织请求参数。我用的是有序的TreeMap保存参数并做签名。没掉坑里。这里附上[微信提供的签名检验工具](https://pay.weixin.qq.com/wiki/tools/signverify/)

## 调用支付接口
这一步是客户端拿到`prepayid`之后，向微信发送PayReq。请求参数有7个(详细参考微信sdk中的PayReq类，其中`partnerId`填商户ID`mch_id`；`packageValue`固定填`Sign=WXPay`；`timestamp`为北京时间戳，单位为秒。其它的顾名思义，对号入座即可)，缺一不可。这里呢，也有坑

- PayResp（响应）定义了errCode和errStr，只要支付失败，errCode都为-1，errStr永远为null，所以，具体的错误原因：签名错误、未注册APPID、项目设置APPID不正确、注册的APPID与设置的不匹配、其他异常等，一个一个查吧。
- [统一下单]接口不返回timestamp，如果是我们自己设置一个，那么需要对请求重新签名，[统一下单]返回的sign就不能用了。不然调用支付接口肯定返回-1。
- 如果接口调用失败，返回-1，是应用签名（apk的签名）不正确（比如debug和release版本使用的不同签名）导致的，那么需要清空微信缓存，才能支付成功。不过，我可不敢随便清微信缓存，我建议可以将手机重启，如果仍然不行，再清微信的缓存。

## libammsdk.jar冲突
因微信支付和分享是在同一个jar中，所以如果使用了第三方的分享sdk，极有可能会出现jar冲突。即使是相同模块中的jar完全一致也不行。解决的办法是只保留一个模块中的libammsdk.jar然后clen build。

## 建议

- apk的包名，签名在工程初始化的时候就确定好，debug和release使用同一个签名。
- 微信开放平台在创建应用前，确认好apk包名和签名。
- 如果是服务端下单，确保sign值是对的（需要再次签名的再签一次）。客户端拿到七个参数后，可以直接支付。
- 建议在服务端下单，这样更安全。
- 使用第三方的支付sdk，避免入坑。

## af-pay
最后推荐一个Android上的支付sdk：[af-pay]，github地址为：https://github.com/Jamling/af-pay
[af-pay]是一个Android平台上支付库，支持支付宝和微信，使用[af-pay]可以让支付变得更简单，并且支持服务端与服务单下单。示例代码如下：
```java
private void doWxpay(String orderInfo) {
    final Activity activity = this;
    // 获取支付类
    Wxpay wxpay = Wxpay.getInstance(activity);
    // 设置支付回调监听
    wxpay.setPayListener(new Wxpay.PayListener() {
        @Override
        public void onPaySuccess(BaseResp resp) {
            showToast(activity, "支付成功");
        }

        @Override
        public void onPayCanceled(BaseResp resp) {
            showToast(activity, "支付取消");
        }

        @Override
        public void onPayFailure(BaseResp resp) {
            showToast(activity, "支付失败");
        }
    });
    // 这里是服务端下单，内容是统一下单返回的xml
    if (!TextUtils.isEmpty(orderInfo)) {
        PayReq req = OrderInfoUtil.getPayReq(orderInfo);
        wxpay.pay(req);
    }
    else { // 客户端下单
        Wxpay.DEBUG = true; // 开启日志
        // API密钥，在微信商户平台设置
        Wxpay.Config.api_key = "32位的字串";
        // APPID，在微信开放平台创建应用后生成
        Wxpay.Config.app_id = "wx...";
        // 商户ID，注册商户平台后生成
        Wxpay.Config.mch_id = "14...";
        // 支付结果异步通知接口，由后台开发提供
        Wxpay.Config.notify_url = "http://www.ieclipse.cn/app/pay/wxpay_notify.do";
        // 创建统一下单异步任务
        Wxpay.DefaultOrderTask task = new Wxpay.DefaultOrderTask(wxpay);
        // 这个商户订单号，由后台返回，在这里随便生成一个
        String outTradeNo = OrderInfoUtil2_0.genOutTradeNo();
        // 设置统一下单的请求参数
        task.setParams(OrderInfoUtil.buildOrderParamMap(outTradeNo, "测试支付", "", "1", null, null, null));
        task.execute();
    }
}
```
附上完整的支付过程中的日志
![完整日志](http://dl.ieclipse.cn/screenshots/af-pay-wechat.png)

如果是服务端下单，客户端只需生成PayReq对象，然后调用`wxpay.pay(PayReq)`即可。[af-pay]已经配置好回调的WXPayActivity和Receiver。详细信息请访问Github。

## 关于
[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF
微信支付开发文档：https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_1

[QuickAF]: https://github.com/Jamling/QuickAF
[微信开放平台]: https://open.weixin.qq.com
[微信商户平台]: https://mch.weixin.qq.com
[统一下单]: https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1
[af-pay]: https://github.com/Jamling/af-pay

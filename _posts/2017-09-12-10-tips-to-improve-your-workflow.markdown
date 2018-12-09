---
layout: post
title: Android app 微信支付爬坑
date: 2018-12-09 19:00:00 +0300
description: 具体所记录是从懵逼状态到支付成功
img: software.jpg
tags: [Android app, 微信支付，爬坑] # add tag
---

> 本来开开心心的上班，安安静静的撸代码，偶尔幻想一下美好的人生多好，让我做微信支付，说没有支付的购物流程是没有灵魂的。。。当然，这是实话|||--

## 前序
先说说在做这个功能前我的状态（对微信支付的认知），四个字：一脸懵逼。
完全不知道思路，不知道专用名词的意思，不知道调试，不知道该有的步骤和思路是什么，有的只是百度（后面有了翻墙的本领，也用了谷歌）。
所以先总结一下需要用到的东西和思路。
## 准备
所搭建的app是基于ionic 和 cordova框架下的。app开发的时候可以在本地网页上f12调试就可以了。但要调试apk，可不会那么方便。我选择的是谷歌：***chrome://inspect/#devices***来进行调试。
调试需要：
- 安卓手机一部
- 能传数据的安卓数据线一根
- 翻墙
- debug.apk（调试问题的时候需要使用未签名的debug.apk，最后要调用支付的时候需要正式签名的apk）

微信支付需要：
- AppID
- 应用签名
- 开发的包名
- 商户id
- key

这里需要详细讲一下这四个东西是咋搞来的，还有他们之间的关系是啥，有啥用。需要访问微信开放平台：https://open.weixin.qq.com

#### AppID
访问微信开放平台，申请app应用，申请的时候，里面所需要填的几个关键的东西是：包名，签名。包名就是我们打包的时候，项目根目录下的config.xml文件中的顶部，那里所需要填的包名，要写一致。关于这个签名，见下一步。申请完了审核过了后会有一个对应的AppId和AppSecret
#### 应用签名
我们可以访问[应用签名生成工具](https://res.wx.qq.com/open/zh_CN/htmledition/res/dev/download/sdk/Gen_Signature_Android2.apk)，下载后安装到手机上。然后打开输入正式版的apk应用的包名。可以获取到签名。包名见下一步
#### 开发的包名
包名是和在微信开放平台申请的app所填的包名一致。也可以使用已经存在的app的包名，到时候生成签名后将该签名替换成改的签名就可以了。主要签名是用来验证这个app是不是合法的。
#### 商户id
商户平台：https://pay.weixin.qq.com/index.php/core/home/login?return_url=%2F，这个和下面的key，是需要在商户平台下申请的，要大洋的。在api安全那里，获取权限后可以获取到。

#### key
这个key很重要，如果不对会得不到正确的sign签名的。在我们后台代码的处理中是必须字段。

具体关键术语和支付流程已经思路，在官方文档这里有写：https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_1。

以上的准备做好后，下面就开始安装插件了。

##安装微信支付的插件

为了环境干净不会有其他的问题出现，我们可以先rm掉之前的android工程。下面是相关的命令

```
npm清除缓存：
npm cache clean -f

ionic移除android项目工程：
ionic platform remove android

ionic添加android项目工程：
ionic platform add android

移除微信支付插件：
cordova plugin remove cordova-plugin-wechat

添加2.0.1版本的微信支付插件：
cordova plugin add cordova-plugin-wechat@2.1.0 --variable wechatappid=wx3ee4998829defa64

cordova-android@6.4.0以下：
cordova plugin add cordova-plugin-wechat@2.1.0 --variable wechatappid=YOUR_WECHAT_APPID

cordova-android@7.0.0以上：
cordova plugin add cordova-plugin-wechatv2 --variable wechatappid=YOUR_WECHAT_APPID

index.html中添加：
<script src="cordova.js"></script>

config.xml中添加微信支付插件的说明：
<plugin name="cordova-plugin-wechat" spec="~2.1.0">
    <variable name="WECHATAPPID" value="" />
</plugin>

打包debug包：
ionic build android
```

如上可以看到安装完了插件后需要在config.xml中声明一下插件。然后还要在index.html中引用该插件。
安装成功后可以在plugins文件夹中看到cordova-plugin-wechat。
下面的一个注意的地方很重要，也是我花了不少时间谷歌找到的，就是在项目中需要引入cordova.js,且其路径是直接引用就可以了，因为在项目build后，在assets/www下面就有该js。注意这里的apk代码是项目工程里打包好的代码，在assers/www下的，而不是我们前面的根目录下的www。所以引入的路径要注意。
以上的步骤成功后，获得的debug.apk，可以在调试的网页上可以看到引入了wechat.js。有看到这个引入了，就说明插件安装没有问题了。

## 代码的实现

#### 前端调用接口的代码

```js
$scope.pay = function () {
                if ($scope.type == 1) {
                    Wechat.isInstalled(function (installed) {
                        if (installed) {
                            ClientOpt.opt({
                                act: 'wxpay',
                                op: 'pay',
                                orderIds: $scope.orderIds,
                                order_amount: $scope.order_amount
                            }, function (json) {
                                if (!StringUtil.isEmpty(json.datas.error)) {
                                    MyDialog.error(json.datas.error);
                                }
                                else {
                                    var params = {
                                        partnerid: json.datas.partnerid,
                                        prepayid: json.datas.prepayid,
                                        noncestr: json.datas.noncestr,
                                        timestamp: json.datas.timestamp,
                                        sign: json.datas.sign
                                    };
                                    Wechat.sendPaymentRequest(params, function () {
                                        MyDialog.tip("支付成功");
                                        $rootScope.gotoPage('/mine_order/0');
                                    }, function (reason) {
                                        MyDialog.tip("Failed: " + reason);
                                        $rootScope.gotoPage('/mine_order/0');
                                    });
                                }
                            });
                        }
                        else {
                            MyDialog.tip("尚未安装微信");
                        }
                    }, function (reason) {

                    });
                }

```
上面的代码中我们传的参数有订单的id号，和需要支付的金额。也可以在前端这里传appid，但建议在后端用三元处理。

#### 后端微信支付相关代码

```php
public function payOp(){
        $app_id=empty($_REQUEST['appID'])?"自己的appid":$_REQUEST['appID'];
        $mch_id='商户id';
        $app_key='商户的那个key';
        $openid=$_REQUEST['openid'];
        // order_info
        $orderIds=$_REQUEST['orderIds'];
        if(empty($orderIds)){
            output_error("网络繁忙");
        }
        $orderIds=getRandChar(5)."_".$orderIds;
        $order_amount=(float)$_REQUEST['order_amount'];
        // get prepay id
        $prepay_id = $this->generatePrepayId($app_id, $mch_id, $app_key,$orderIds,$order_amount,$openid);
        // re-sign it
        if($_REQUEST['platform'] == "wxmini"){ //小程序的支付接口
            $response = array(
                'appId' => $app_id,
                'timeStamp' => ''.time(),
                'nonceStr' => $this->generateNonce(),
                'package' => 'prepay_id='.$prepay_id,
                'signType' => 'MD5'
            );
            $response['paySign'] = $this->calculateSign($response, $app_key);
        }
        else{
            $response = array(
                'appid'     => $app_id,
                'partnerid' => $mch_id,
                'prepayid'  => $prepay_id,
                'package'   => 'Sign=WXPay',
                'noncestr'  => $this->generateNonce(),
                'timestamp' => time(),
            );
            $response['sign'] = $this->calculateSign($response, $app_key);
        }
        output_data($response);
    }
```
具体代码已保存在百度云（wxpay.php和wxpaynotify.php回调验证数据是否正确文件，用于检验数据库的数据是否是正确的，从而避免只使用了前端js来判断支付的成败）

在接下来调用微信支付

## 调用微信支付
需要使用我们填在开放平台那个app下面的签名的那个keystore来进行签名。得到签名后的apk安装运行，才可以调起支付接口。

完整做好上面的步骤就可以完成支付的全流程了。当然，在后台代码的实现上还是需要注意很多地方的，有时间的话会再单独写一个里面的坑（主要是官方文档中的巨坑）。然后再后面做微信分享和登陆等，再一一写到来。

相关参考链接：
- https://blog.csdn.net/m00123456789/article/details/56481656
- https://pay.weixin.qq.com/wiki/tools/signverify/
- https://blog.csdn.net/ws1836300/article/details/53893102
- https://blog.csdn.net/xinluqishi123/article/details/74596357
- https://www.cnblogs.com/porter/p/6364633.html
- https://blog.csdn.net/qq_34815528/article/details/78264958
- https://blog.csdn.net/nnmmbb/article/details/50533138
- https://segmentfault.com/q/1010000012787760
---
layout: post
title: ionic2 实现多图片上传
date: 2019-01-10 10:21:00 +0900
description: 插件的使用
img: 2019/01.jpg
tags: [图片上传] # add tag
---

插件的使用，主要有以下几个重要步骤：

- 首先安装插件，对应的版本号要明确
- 在config.xml中加上对应的插件名称和版本号
- 然后删除之前的项目工程，ionic platform rm android
- 再新建项目工程，ionic platform add android
- 打包debug包，ionic build android
- 打包正式包，ionic build android --prod --release
- 进入项目工程目录下，cd platforms  cd android
- 生成签名文件，keytool -genkey -v -keystore myApp.keystore -alias myAppKey -keyalg RSA -keysize 2048 -validity 20000，其中 myApp.keystore为签名文件，myApp.apk是签名成功后的apk名称，都是可以改成自己想要的名称。这里插播个题外话，就是在微信支付这块，这里生成的签名，需要使用微信支付官方的小应用来根据自己的包名获取对应的签名，然后要更新填入到开放平台下对应的app应用中。小工具下载链接在[这里]("https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167&token=62bfbbd065e17a3393ccd34f7ca5fb7e3c0cba92&lang=zh_CN")
- 把生成的未签名的apk复制粘贴到和签名同目录下（项目工程目录下）
- 对apk签名，jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore myApp.keystore -signedjar myApp.apk android-release-unsigned.apk myAppKey

以上是较为完整的使用插件和打包项目的步骤。其中调试用的是debug包，在前面的文章中有介绍，这里就不重复。
下面来进入正题。

## 下载安装相关插件
```text
ionic plugin add cordova-plugin-image-picker@1.1.1

ionic plugin add cordova-plugin-camera@2.4.1

ionic plugin add cordova-plugin-file-transfer@1.7.1

ionic plugin add cordova-plugin-file@4.3.3
```
添加到config.xml中
```xml
  <plugin name="cordova-plugin-camera" spec="~2.4.1"/>
  <plugin name="cordova-plugin-image-picker" spec="~1.1.1"/>
  <plugin name="cordova-plugin-file" spec="~4.3.3"/>
  <plugin name="cordova-plugin-file-transfer" spec="~1.7.1"/>
```
## js和html中代码的实现
**JS**
```js

.controller('Publish_tshirtCtrl', ['$rootScope', '$scope', '$state', '$stateParams', '$location', '$ionicPopup', '$cookies', '$ionicHistory', '$ionicModal', '$ionicScrollDelegate', '$timeout', '$ionicLoading', '$ionicNativeTransitions','$filter', 'Util', '$ionicPopover', 'StringUtil', 'ClientOpt', 'MyDialog', 'YorderUtil', 'AppUpdateService', 'JpushService', '$ionicActionSheet', 'Constant', '$cordovaCamera', '$cordovaImagePicker', '$cordovaFileTransfer',
        function ($rootScope, $scope, $state, $stateParams, $location, $ionicPopup, $cookies, $ionicHistory, $ionicModal, $ionicScrollDelegate, $timeout, $ionicLoading, $ionicNativeTransitions, $filter, Util, $ionicPopover, StringUtil, ClientOpt, MyDialog, YorderUtil, AppUpdateService, JpushService, $ionicActionSheet, Constant, $cordovaCamera, $cordovaImagePicker, $cordovaFileTransfer) {
            $scope.image_list = [];
            $scope.selectSize = '';
            $scope.$on('$ionicView.enter', function (scopes, states) {
                $scope.rootEle = $("#publish_tshirt[nav-view='active']");
                if ($scope.loaded) {
                    return;
                }
                $scope.loaded = true;
                $rootScope.shijiReload = false;
                $scope.init();
            });
            $scope.init = function () {
                
            };
            $scope.submit = function () {
                var rootEle = $scope.rootEle;
                var ex_name = $('#ex_name').val();
                ClientOpt.opt({
                    act: 'goods',
                    op: 'exchange',
                    ex_name: ex_name
                }, function (json) {
                    if (!StringUtil.isEmpty(json.datas.error)) {
                        MyDialog.tip(json.datas.error);
                    }
                    else {
                        $scope.ex_id = json.datas.ex_id;
                        $scope.upImage($scope.image_list);
                        //MyDialog.tip("发布成功");
                    }
                });
            }

            $scope.selectImg = function () {
                $ionicActionSheet.show({
                    buttons: [{
                        text: '相册'
                    }, {
                        text: '拍照'
                    }
                    ],
                    titleText: '选择图片',
                    cancelText: '取消',
                    cancel: function () {
                    },
                    buttonClicked: function (index) {
                        switch (index) {
                            case 0:
                                $scope.pickImage();
                                break;
                            case 1:
                                $scope.takePhoto();
                                break;
                            default:
                                break;
                        }
                        return true;
                    }
                });
            };
            $scope.pickImage = function () {
                var options = {
                    maximumImagesCount: 9, //这里是选择几个图片
                    width: 800,
                    height: 800,
                    quality: 80
                };
                $cordovaImagePicker.getPictures(options)
                    .then(function (results) {
                        $scope.image_list = results;
                    }, function (error) {
                        MyDialog.tip(error);
                    });
            };
            $scope.takePhoto = function () {
                var options = {
                    //这些参数可能要配合着使用，比如选择了sourcetype是0，destinationtype要相应的设置
                    quality: 100,                                            //相片质量0-100
                    destinationType: Camera.DestinationType.FILE_URI,        //返回类型：DATA_URL= 0，返回作为 base64 編碼字串。 FILE_URI=1，返回影像档的 URI。NATIVE_URI=2，返回图像本机URI (例如，資產庫)
                    sourceType: Camera.PictureSourceType.CAMERA,             //从哪里选择图片：PHOTOLIBRARY=0，相机拍照=1，SAVEDPHOTOALBUM=2。0和1其实都是本地图库
                    allowEdit: false,                                        //在选择之前允许修改截图
                    encodingType: Camera.EncodingType.JPEG,                   //保存的图片格式： JPEG = 0, PNG = 1
                    targetWidth: 800,                                        //照片宽度
                    targetHeight: 800,                                       //照片高度
                    mediaType: 0,                                             //可选媒体类型：圖片=0，只允许选择图片將返回指定DestinationType的参数。 視頻格式=1，允许选择视频，最终返回 FILE_URI。ALLMEDIA= 2，允许所有媒体类型的选择。
                    cameraDirection: 0,                                       //枪后摄像头类型：Back= 0,Front-facing = 1
                    popoverOptions: CameraPopoverOptions,
                    saveToPhotoAlbum: true                                   //保存进手机相册
                };
                $cordovaCamera.getPicture(options).then(function (imageData) {
                    $scope.upImage(imageData);
                }, function (err) {
                    MyDialog.tip(err);
                });
            };
            $scope.upImage = function (imageUrl) {
                document.addEventListener('deviceready', function () {
                    var url = Constant.WebPath + Constant.BackendModule + '/index.php?act=goods&op=upload_image&ex_id=' + $scope.ex_id + '&platform=' + $rootScope.platform_name + '&token=' + localStorage.getItem('token') + '&store_id=' + 1714;
                    var options = { chunkedMode: false };
                    //var ft = new FileTransfer();
                    //上传进度
                    $cordovaFileTransfer.onprogress = function(progressEvent) {
                        MyDialog.showProgress(progressEvent);
                        if (progressEvent.lengthComputable) {
                            loadingStatus.setPercentage(progressEvent.loaded / progressEvent.total);
                        } else {
                            loadingStatus.increment();
                        }
                    };
                    //这里是实现多个图片上传的步骤
                    for (var i = 0; i < $scope.image_list.length; i++) {
                        if ($scope.image_list[i] != "") {
                            var imageUrl = $scope.image_list[i];
                            $cordovaFileTransfer.upload(url, imageUrl, options)
                                .then(function (result) {
                                    var data = JSON.parse(result.response);
                                    if (data.error !== undefined) {
                                        MyDialog.error("网络繁忙");
                                    }
                                    else {
                                        MyDialog.hideProgress();
                                        MyDialog.tip("发布成功", 2500);
                                        $rootScope.gotoPage('/home');
                                    }
                                }, function (err) {
                                    MyDialog.error(err);
                                }, function (progress) {
                                });
                        }
                    }

                }, false);
            };
        }])
```
**HTML**
```html
<div class="pt_wrapper">
            <input class="pt_mz" id="ex_name" type="text" placeholder="标题 T恤的名称">
            <div class="pt_bbox">
                <div class="pt_bimg" ng-repeat="image in image_list" ng-if="image_list">
                    <img src={{image}} />
                </div>
                <div class="pt_bimg">
                    <img name="file" ng-click="selectImg()" src="./img/ac.png" /> 
                </div>
            </div>
</div>
```
**代码分析**
这里需要弄清楚几个逻辑，因为这里是多个图片上传，然后还是需要和其他信息一起提交到服务端的，所以和头像上传会有所不同，头像上传的逻辑是上传一个图片，点击头像选择后，会直接上传到服务器端，然后使用ajax刷新一下头像数据即可。但如果这里也这样做的话有点不好，因为如果每次选择图片后都要先直接传到服务器的话，那删除或重新添加等操作又要对服务器进行操作，这里其实可以分开两步，第一步先使用插件cordova-plugin-image-picker选择图片，将选择的图片存到一个数组中，双向绑定输出到页面中遍历，这里的图片并没有走服务器端，仅仅是显示选择了哪些图片，速度更快，使用这个插件注意点击的html中不要使用input标签，不然会调起键盘或其他的东西。第二步，在点击submit提交的时候，触发使用cordova-plugin-file-transfe插件来对图片的上传，上面代码中的for循环进行了多个图片异步上传。
下面是后端对图片的处理：
**PHP**
```php
public function upload_imageOp(){
        $file = $_FILES["file"];
        $name = $file['name'];
        $dir=BASE_UPLOAD_PATH . DS.'tplus'.DS;
        if(!file_exists($dir))
            mkdir($dir,0777,true);  //如果没有该文件夹则创建一个
        $filename=TIMESTAMP . "_" . $name;
        $path = BASE_UPLOAD_PATH . DS.'tplus'.DS . $filename;
        if(move_uploaded_file($file['tmp_name'],$path)){
            $r=Model("ex_goods")->addData(array("pic"=>$filename,"ex_id"=>$_REQUEST['ex_id']));  //这里写入数据库的图片是处理过的名称，在读图片的时候需要进行地址的拼接处理
            if(!$r){
                echo json_encode(array("error"=>"网络繁忙"));
                exit;
            }
            echo json_encode(array("result"=>"上传成功","filename"=>$filename));
            exit;
        }else{
            echo json_encode(array("error"=>"网络繁忙"));
            exit;
        }
    }
```
至此，多图片上传就完成了，后面会接着写下载图片和视频播放的文章总结

![]({{site.baseurl}}/assets/img/2019/02.jpg)

最后附上一个大神总结提供的关于ionic2安装插件的版本问题，复制下来作为备用，链接在[这里]("https://github.com/yanxiaojun617/ionic2_tabs/blob/master/config.xml")
```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.kit.ionic2tabs" version="1.1.5" xmlns="http://www.w3.org/ns/widgets">
    <name>ionic2_tabs</name>
    <description>An awesome Ionic/Cordova app.</description>
    <author email="yanxiaojun617@163.com" href="http://www.gzkit.com.cn/">广州科腾信息集成部技术分部</author>
    <content src="index.html" />
    <access origin="*" />
    <allow-navigation href="http://ionic.local/*" />
    <allow-intent href="http://*/*" />
    <allow-intent href="https://*/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="geo:*" />
    <platform name="android">
        <allow-intent href="market:*" />
        <icon density="ldpi" src="resources\android\icon\drawable-ldpi-icon.png" />
        <icon density="mdpi" src="resources\android\icon\drawable-mdpi-icon.png" />
        <icon density="hdpi" src="resources\android\icon\drawable-hdpi-icon.png" />
        <icon density="xhdpi" src="resources\android\icon\drawable-xhdpi-icon.png" />
        <icon density="xxhdpi" src="resources\android\icon\drawable-xxhdpi-icon.png" />
        <icon density="xxxhdpi" src="resources\android\icon\drawable-xxxhdpi-icon.png" />
        <splash density="land-ldpi" src="resources\android\splash\drawable-land-ldpi-screen.png" />
        <splash density="land-mdpi" src="resources\android\splash\drawable-land-mdpi-screen.png" />
        <splash density="land-hdpi" src="resources\android\splash\drawable-land-hdpi-screen.png" />
        <splash density="land-xhdpi" src="resources\android\splash\drawable-land-xhdpi-screen.png" />
        <splash density="land-xxhdpi" src="resources\android\splash\drawable-land-xxhdpi-screen.png" />
        <splash density="land-xxxhdpi" src="resources\android\splash\drawable-land-xxxhdpi-screen.png" />
        <splash density="port-ldpi" src="resources\android\splash\drawable-port-ldpi-screen.png" />
        <splash density="port-mdpi" src="resources\android\splash\drawable-port-mdpi-screen.png" />
        <splash density="port-hdpi" src="resources\android\splash\drawable-port-hdpi-screen.png" />
        <splash density="port-xhdpi" src="resources\android\splash\drawable-port-xhdpi-screen.png" />
        <splash density="port-xxhdpi" src="resources\android\splash\drawable-port-xxhdpi-screen.png" />
        <splash density="port-xxxhdpi" src="resources\android\splash\drawable-port-xxxhdpi-screen.png" />
    </platform>
    <platform name="ios">
        <allow-intent href="itms:*" />
        <allow-intent href="itms-apps:*" />
        <icon height="57" src="resources/ios/icon/icon.png" width="57" />
        <icon height="114" src="resources/ios/icon/icon@2x.png" width="114" />
        <icon height="40" src="resources/ios/icon/icon-40.png" width="40" />
        <icon height="80" src="resources/ios/icon/icon-40@2x.png" width="80" />
        <icon height="120" src="resources/ios/icon/icon-40@3x.png" width="120" />
        <icon height="50" src="resources/ios/icon/icon-50.png" width="50" />
        <icon height="100" src="resources/ios/icon/icon-50@2x.png" width="100" />
        <icon height="60" src="resources/ios/icon/icon-60.png" width="60" />
        <icon height="120" src="resources/ios/icon/icon-60@2x.png" width="120" />
        <icon height="180" src="resources/ios/icon/icon-60@3x.png" width="180" />
        <icon height="72" src="resources/ios/icon/icon-72.png" width="72" />
        <icon height="144" src="resources/ios/icon/icon-72@2x.png" width="144" />
        <icon height="76" src="resources/ios/icon/icon-76.png" width="76" />
        <icon height="152" src="resources/ios/icon/icon-76@2x.png" width="152" />
        <icon height="167" src="resources/ios/icon/icon-83.5@2x.png" width="167" />
        <icon height="29" src="resources/ios/icon/icon-small.png" width="29" />
        <icon height="58" src="resources/ios/icon/icon-small@2x.png" width="58" />
        <icon height="87" src="resources/ios/icon/icon-small@3x.png" width="87" />
        <icon height="1024" src="resources/ios/icon/icon-1024.png" width="1024" />
        <splash height="1136" src="resources/ios/splash/Default-568h@2x~iphone.png" width="640" />
        <splash height="1334" src="resources/ios/splash/Default-667h.png" width="750" />
        <splash height="2208" src="resources/ios/splash/Default-736h.png" width="1242" />
        <splash height="1242" src="resources/ios/splash/Default-Landscape-736h.png" width="2208" />
        <splash height="1536" src="resources/ios/splash/Default-Landscape@2x~ipad.png" width="2048" />
        <splash height="2048" src="resources/ios/splash/Default-Landscape@~ipadpro.png" width="2732" />
        <splash height="768" src="resources/ios/splash/Default-Landscape~ipad.png" width="1024" />
        <splash height="2048" src="resources/ios/splash/Default-Portrait@2x~ipad.png" width="1536" />
        <splash height="2732" src="resources/ios/splash/Default-Portrait@~ipadpro.png" width="2048" />
        <splash height="1024" src="resources/ios/splash/Default-Portrait~ipad.png" width="768" />
        <splash height="960" src="resources/ios/splash/Default@2x~iphone.png" width="640" />
        <splash height="480" src="resources/ios/splash/Default~iphone.png" width="320" />
        <splash height="2732" src="resources/ios/splash/Default@2x~universal~anyany.png" width="2732" />
    </platform>
    <preference name="webviewbounce" value="false" />
    <preference name="UIWebViewBounce" value="false" />
    <preference name="DisallowOverscroll" value="true" />
    <preference name="android-minSdkVersion" value="16" />
    <preference name="android-maxSdkVersion" value="22" />
    <preference name="android-targetSdkVersion" value="22" />
    <preference name="BackupWebStorage" value="none" />
    <preference name="ShowSplashScreen" value="true" />
    <preference name="SplashScreen" value="screen" />
    <preference name="SplashScreenDelay" value="3000" />
    <preference name="loadUrlTimeoutValue" value="700000" />
    <preference name="AutoHideSplashScreen" value="false" />
    <preference name="SplashShowOnlyFirstTime" value="false" />
    <preference name="FadeSplashScreen" value="false" />
    <feature name="SplashScreen">
        <param name="android-package" value="org.apache.cordova.splashscreen.SplashScreen" />
    </feature>
    <feature name="StatusBar">
        <param name="ios-package" onload="true" value="CDVStatusBar" />
    </feature>
    <icon src="resources\android\icon\drawable-xhdpi-icon.png" />
    <allow-navigation href="*" />
    <allow-navigation href="http://88.128.18.144:8100" />
    <allow-navigation href="http://localhost:8080/*" />
    <feature name="CDVWKWebViewEngine">
        <param name="ios-package" value="CDVWKWebViewEngine" />
    </feature>
    <preference name="CordovaWebViewEngine" value="CDVWKWebViewEngine" />
    <plugin name="ionic-plugin-keyboard" spec="~2.2.1" />
    <plugin name="cordova-plugin-whitelist" spec="1.3.1" />
    <plugin name="cordova-plugin-console" spec="1.0.5" />
    <plugin name="cordova-plugin-statusbar" spec="~2.3.0" />
    <plugin name="cordova-plugin-device" spec="1.1.4" />
    <plugin name="cordova-plugin-splashscreen" spec="~4.0.1" />
    <plugin name="cordova-plugin-ionic-webview" spec="^1.1.11" />
    <plugin name="cordova-plugin-camera" spec="~2.4.1" />
    <plugin name="cordova-plugin-inappbrowser" spec="~1.7.2" />
    <plugin name="cordova-plugin-x-toast" spec="~2.6.0" />
    <plugin name="com.synconset.imagepicker" spec="~2.1.8">
        <variable name="PHOTO_LIBRARY_USAGE_DESCRIPTION" value="请允许使用图库" />
    </plugin>
    <plugin name="cordova-plugin-appminimize" spec="~1.0" />
    <plugin name="cordova-plugin-app-version" spec="~0.1.9" />
    <plugin name="cordova-plugin-network-information" spec="~1.3.4" />
    <plugin name="cordova-plugin-file" spec="~4.3.3" />
    <plugin name="cordova-plugin-file-transfer" spec="~1.7.0" />
    <plugin name="mx.ferreyra.callnumber" spec="~0.0.2" />
    <plugin name="cordova-sqlite-storage" spec="~2.1.2" />
    <plugin name="cordova-plugin-advanced-http" spec="~1.9.1" />
    <plugin name="cordova-plugin-file-opener2" spec="~2.0.19" />
    <engine name="ios" spec="~4.3.1" />
    <plugin name="cordova-plugin-qrscanner" spec="~2.6.0" />
    <plugin name="cordova-plugin-x-socialsharing" spec="~5.4.4">
        <variable name="ANDROID_SUPPORT_V4_VERSION" value="24.1.1+" />
    </plugin>
</widget>
```
---
layout: post
title: ionic2实现图片下载
date: 2019-01-10 12:00:00 +10000
img: 2019/02.jpg # Add image post (optional)
tags: [图片下载,ionic,app ] # add tag
---

**cordova-plugin-file-transfer**

这个插件，前面的一篇中介绍了多图片上传，也用到了这个插件，下载图片也会用到这个插件，官方链接[戳这里]("https://cordova.apache.org/docs/en/8.x/reference/cordova-plugin-file-transfer/index.html#binaryFile")

![download]({{site.baseurl}}/assets/img/2019/1-01.png)

前面有一个文章说的是另一个插件保存图片到相册的，但那种保存图片的速度很慢，目前还没有找到是什么原因，但现在这个插件保存图片很快，下面来说具体怎么用这个插件。

同样的，需要安装插件：
- ionic plugin add cordova-plugin-file-transfer@1.7.1
- ionic plugin add cordova-plugin-file@4.3.3

添加到config.xml中：

```text
<plugin name="cordova-plugin-file" spec="~4.3.3"/>
<plugin name="cordova-plugin-file-transfer" spec="~1.7.1"/>
```

**HTML**
```html
<div class="ps_icon ps_dl_ico" ng-click="download(series_picture)"></div>
```

**JS**
```js
.controller('Photo_showCtrl', ['$rootScope', '$scope', '$state', '$location', '$timeout', '$ionicHistory', '$ionicModal', '$ionicPopover', '$cordovaBarcodeScanner', 'Util', 'StringUtil', 'ClientOpt', 'MyDialog', '$ionicNativeTransitions','$filter','$q',
        function ($rootScope, $scope, $state, $location, $timeout, $ionicHistory, $ionicModal, $ionicPopover, $cordovaBarcodeScanner, Util, StringUtil, ClientOpt, MyDialog, $ionicNativeTransitions, $filter, $q) {
            //这里我删除了其他的代码，只保留了下载的相关代码，实际项目中，有其他的代码，如数据加载等，因为后面我们传来的图片地址也是在加载后才有的；上面的函数中需加上：'$ionicNativeTransitions','$filter','$q',$ionicNativeTransitions, $filter, $q，且顺序只能在其他的后面
            $scope.download=function(series_picture){
                MyDialog.confirm("保存提示","确定要保存图片到本地吗？","是的","按错了",function () {
                    $scope.fullPath="file:///tplus";
                    $scope.downloadFile(series_picture);
                });
            }
            $scope.downloadFile=function(fileUrl) {
                $scope.onDeviceReady().then(function(){
                    var ft = new FileTransfer();
                    var uri = encodeURI(fileUrl);
                    var fileURL = $scope.fullPath + uri.substr(uri.lastIndexOf('/') + 1);
                    ft.onprogress = function(progressEvent) {
                        if (progressEvent.lengthComputable) {
                            console.log("当前进度："+progressEvent.loaded / progressEvent.total);
                        }
                        MyDialog.showProgress(progressEvent);
                    };
                    ft.download(
                        uri,
                        fileURL,
                        function() {
                            MyDialog.hideProgress();
                            MyDialog.tip("保存成功,保存在目录tplus下",2000);
                        },
                        function(error) {
                            MyDialog.tip("download error source " + error.source);
                            console.log("download error source " + error.source);
                            console.log("download error target " + error.target);
                            console.log("upload error code" + error.code);
                        },
                        false,
                        {
                            headers: {
                                "Authorization": "Basic dGVzdHVzZXJuYW1lOnRlc3RwYXNzd29yZA=="
                            }
                        }
                    );
                });
            }
            //下面是检查和创建相册文件夹的操作
            $scope.onDeviceReady=function() {
                var q = $q.defer();
                try {
                    window.resolveLocalFileSystemURL(cordova.file.externalRootDirectory, function (fileSystem) {
                        fileSystem.getDirectory("tplus",  {
                            create: true,
                            exclusive: false
                        }, function (result) {
                            $scope.fullPath = result.toURL();
                            q.resolve(result);
                        }, function (error) {
                            MyDialog.tip(error.message);
                            q.reject(error);
                        });
                    }, function (err) {
                        MyDialog.tip(err.message);
                        q.reject(err.message);
                    });
                }
                catch (e) {
                    MyDialog.tip(e.message);
                    q.reject(e);
                }
                return q.promise;
            }
        }])
```
关于进度和弹出确认框的相关代码：
```js
showProgress:function(downloadProgress){
                if(!$rootScope.isProgressShow){
                    $ionicBackdrop.retain();
                    $rootScope.isProgressShow=true;
                }
                $rootScope.progress_v.showProgress=true;
                $rootScope.progress_v.progress=downloadProgress;
}

hideProgress:function(){
    $ionicBackdrop.release();
    $rootScope.progress_v.showProgress=false;
    $rootScope.isProgressShow=false;
}

confirm:function(title,msg,okText,cancelText,okFunc,cancelFunc){
                $ionicPopup.confirm({
                    title: '<a class="picon ion-help-circled"></a><span>'+title+'</span>',
                    template: msg,
                    cancelText:StringUtil.isEmpty(cancelText)?"关闭":cancelText,
                    okText:StringUtil.isEmpty(okText)?"确定":okText
                }).then(function(res) {
                    if(res) {
                        if(!StringUtil.isEmpty(okFunc))
                            okFunc();
                    } else {
                        if(!StringUtil.isEmpty(cancelFunc))
                            cancelFunc();
                    }
                });
}
```
到此，下载图片的功能就做好了。

---
layout: post
title: cordova实现保存图片到手机
date: 2018-12-13 12:00:00 +10000
description: cordova实现保存图片到手机
img: 2018-12-13-06.jpg # Add image post (optional)
tags: [保存图片,CORDOVA,手机端 ] # add tag
---

手机端保存图片到相册中会分好以下两种情况：
- 长按保存
- 点击下载保存

实现的方法也有好几种。由于项目中使用的是cordova框架搭建的移动端app。所以下面只说一种比较好的方法，当然，其中还需优化，比如下载图片时的用户体验，这里并没有做。然后，后面有时间会再总结一篇长按图片保存的方法。这里只介绍点击下载保存图片。

## 正文

### 步骤一
```php
下载插件命令：
cordova plugin add https://github.com/devgeeks/Canvas2ImagePlugin.git
```
下载完成后显示如下图，说明下载成功,插件***org.devgeeks.Canvas2ImagePlugin***

![图一]({{site.baseurl}}/assets/img/2018-12/down-picture-plugin.png)

### 步骤二
项目配置文件引入该插件js
```php
config.xml

<plugin name="org.devgeeks.Canvas2ImagePlugin" spec="~0.6.0"/>
```
### 步骤三
相应的html文件需要点击下载的位置写上点击事件

```html
xxx.html

<div class="ps_icon ps_dl_ico" ng-click="downPhoto(series_picture)"></div>
```

### 步骤四
在controllers.js中的相应控制器位置添加js代码

```angularjs
$scope.downPhoto = function (photoPath) {
                var pictrueUrl = encodeURI(photoPath);
                function saveImageToPhone(url, success, error) {
                    var canvas, context, imageDataUrl, imageData;
                    var img = new Image();
                    img.onload = function () {
                        canvas = document.createElement('canvas');
                        canvas.width = img.width;
                        canvas.height = img.height;
                        context = canvas.getContext('2d');
                        context.drawImage(img, 0, 0);
                        try {
                            imageDataUrl = canvas.toDataURL('image/jpeg', 1.0);
                            imageData = imageDataUrl.replace(/data:image\/jpeg;base64,/, '');
                            cordova.exec(
                                success,
                                error,
                                'Canvas2ImagePlugin',
                                'saveImageDataToLibrary',
                                [imageData]
                            );
                        }
                        catch (e) {
                            error(e.message);
                        }
                    };
                    try {
                        img.src = url;
                    }
                    catch (e) {
                        error(e.message);
                    }
                }
                var success = function (msg) {
                    MyDialog.tip("下载成功");
                    //下载成功
                };
                var error = function (err) {
                    MyDialog.tip("下载失败");
                    //下载失败
                };
                saveImageToPhone(photoPath, success, error);
            };
```
### 步骤五
可在调试中查看是否成功，也可以直接看手机相册中是否有图片了。

![图一]({{site.baseurl}}/assets/img/2018-12/down-pic.png)

结束。

参考链接：https://blog.csdn.net/u010730897/article/details/54923250
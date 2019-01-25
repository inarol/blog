# 微信前端开发知识库

## 目录

* [前言](#前言)
* [问题](#问题)
    * [根据微信抓取规则，自定义分享标题和分享的icon](#根据微信抓取规则自定义分享标题和分享的icon)
    * [动态更改页面title](#动态更改页面title)
    * [微信环境下下载apk、ipa安装包](#微信环境下下载apkipa安装包)

## 前言

本文总结在微信里开发前端页面时遇到的问题，同时也给出了相对比较靠谱的解决方案，或是一些hack技术。

## 问题

### 根据微信抓取规则，自定义分享标题和分享的icon

我们知道，微信出了新的JSSDK后，是允许用户自定义分享的title和图片的，但必须引用js文件（10k），并且也必须申请公众号，用`appId`,`timestamp`,`nonceStr`,`signature`去认证权限，否则无法使用JSSDK的功能。但是，一个很小的微信页面，在没有公众号又不想去搞JSSDK的情况，能不能去设置分享的title和icon呢。答案是可以的，微信最新版亲测可靠。

* 标题：微信会抓取当前页面的title。
* 图片：微信自己会去抓取当前页面body内最前面的一张符合条件（290px   × 290px）的图片，只有符合两个条件的图片才能成为分享的icon。如果这两个条件都不符合，则分享出去的消息是不带icon的。  

反编译微信的源代码，获得wxjs.js，可以看到微信抓取分享图片规则：

```javascript
// 在页面中找到第一个最小边大于290的图片，如果1秒内找不到，则返回空（不带图分享）。
var getSharePreviewImage = function (cb) {
  var isCalled = false;
  var callCB = function (_img) {
    if (isCalled) {
      return;
    };
    isCalled = true;

    cb(_img);
  }
  var _allImgs = _WXJS('img');
  if (_allImgs.length == 0) {
    return callCB();
  }
  // 过滤掉重复的图片
  var _srcs = {};
  var allImgs = [];
  for (var i = 0; i < _allImgs.length; i++) {
    var _img = _allImgs[i];

    // 过滤掉不可以见的图片
    if (_WXJS(_img).css('display') == 'none' || _WXJS(_img).css('visibility') == 'hidden') {
      // _log('ivisable image !! ' + _img.src);
      continue;
    }
    if (_srcs[_img.src]) {
      // added
    } else {
      _srcs[_img.src] = 1; // mark added
      allImgs.push(_img);
    }
  };
  var results = [];
  var img;
  for (var i = 0; i < allImgs.length && i < 100; i++) {
    img = allImgs[i];
    var newImg = new Image();
    newImg.onload = function () {
      this.isLoaded = true;
      var loadedCount = 0;
      for (var j = 0; j < results.length; j++) {
        var res = results[j];
        if (!res.isLoaded) {
          break;
        }
        loadedCount++;
        if (res.width > 290 && res.height > 290) {
          callCB(res);
          break;
        }
      }
      if (loadedCount == results.length) {
        // 全部都已经加载完了，但还是没有找到。
        callCB();
      };
    }
    newImg.src = img.src;
    results.push(newImg);
  }
  setTimeout(function () {
    for (var j = 0; j < results.length; j++) {
      var res = results[j];
      if (!res.isLoaded) {
        continue;
      }
      if (res.width > 290 && res.height > 290) {
        callCB(res);
        return;
      }
    }
    callCB();
  }, 1000);
}
```

注意：如果要分享出去的icon不想在页面出现，可以用`position:absolute;top:-99999px;`这样就不会在页面出现了。不要把图片隐藏了，如果是用`display:none，`但是微信抓取图片的规则会把它忽略了，分享无效。

### 动态更改页面title

当页面缓慢时，再用js代码去更改页面的title时，会发现已经无法修改了，用`setTimeout`延迟500ms去模拟这种情况时，会发现在pc浏览器和手机其他浏览器（非微信）测下，还是能修改title的。这时候就要用到一种hack技术了,亲测有效。
```javascript
document.title = "前端小栈";
var $iframe = $('<iframe src="/favicon.ico"></iframe>').on('load', function () {
  setTimeout(function () {
    $iframe.off('load').remove()
  }, 0)
}).appendTo($("body")); 
```

### 微信环境下下载apk、ipa安装包

微信开始封闭扫码或点击链接下载apk、ipa安装包，甚至现在连水果的appstore都不能直接打开了，不管微信是耍流氓还是出于提升用户体验，毕竟那是TX的地盘。好了，既然无法改变世界，那么就去适应它吧。
网上一种通用的解决方案是：腾讯应用宝（这diao东西我从来没用过）。非常简单，只需要到腾讯开发平台的微下载申请自己的移动应用后，就会得到应用宝的下载地址了，将你原先的下载地址替换成应用宝下载地址即可。其实整个体验也是挺不错的，相对于“打开右上角-其他浏览器”这样的多余交互。
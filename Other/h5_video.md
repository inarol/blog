## H5页面视频播放

### 自动播放

#### 微信

- video元素设置`autoplay`
- 引入微信JSSDK的脚本：http://res.wx.qq.com/open/js/jweixin-1.4.0.js
- 监听`WeixinJSBridgeReady`事件
- 调用play函数

```html
<video src="foo.mp4" autoplay />
<script>
document.addEventListener('WeixinJSBridgeReady', () => {
  this.video.play();
}, false);
</script>
```

#### QQ

- 设置`autoplay`

```html
<video src="foo.mp4" autoplay />
```

#### QQ浏览器

ios：

- 设置`autoplay`
- 调用play函数，对调用时机有要求，最好`setTimeout`延时调用

Android

- 设置`autoplay`

#### Safari

ios 9 之前（包含ios9），播放策略就比较简单：

- 必须是用户触发了交互， 才允许播放。

ios 10+：

对于设置了`autoplay`的视频，播放策略如下：

- 对于没有音轨的视频源，允许自动播放
- 静音模式下，设置`autoplay`和`muted`，允许自动播放，但在播放过程中，如果设置了开启了声音，则会视频会被暂停。
- 自动播放的视频必须要是在用户的可视范围内，否则视频也会被暂停。

对于普通`video`元素调用`play()`方法，播放策略如下：

- 如果没有音轨的视频源，或者设置了静音模式，无需用户交互也可以播放视频，同样的，如果在播放过程中，如果设置了开启了声音，则会视频也会被暂停。
- 无需video元素可见，`play()`也可播放视频
- `play()`会返回一个`Promise`，如果播放出现问题，会触发`reject`。

参考资料：

- https://webkit.org/blog/6784/new-video-policies-for-ios/

#### Chrome

Chrome浏览器（PC）播放策略，符合以下情况允许自动播放：

- 视频处于静音的状态

- 跟页面发生交互（例如点击或触摸等交互行为）

- 在**桌面端**，根据用户端的媒体参与指数（Media Engagement Index，MEI）评分，包含访问次数，播放次数，上次播放时间，最终得出的分数如果是高频率播放的话，浏览器则允许自动播放

- 在移动端，用户添加网站到主屏中

- 对于iframe里面的媒体，如果是同域下，可以设置是否允许自动播放：

  ```html
  <!-- Autoplay is allowed. -->
  <iframe src="https://cross-origin.com/myvideo.html" allow="autoplay">
  
  <!-- Autoplay and Fullscreen are allowed. -->
  <iframe src="https://cross-origin.com/myvideo.html" allow="autoplay; fullscreen">
  ```

 MEI 评分是评估用户参与当前域名站点的媒体指数，主要取决于用户的访问播放次数，但同时对视频源也有要求：

- 视频长度必须大于**7** 秒
- 不能是静音
- 视频处于激活态
- 视频video必须大于`200x140`px

打开`chrome://media-engagement/`可以看到各个域名的媒体参与评分指数：

![Screenshot of the chrome://media-engagement page](https://developers.google.com/web/updates/images/2017/09/media-engagement.png)

注：

打开`chrome://flags/#autoplay-policy`可以设置Chrome 自动播放的策略，如果要重置分数，选择浏览器的`清除浏览数据`即可。

最佳实践：

**永远不要信任浏览器支持播放视频**，`play()`方法会返回一个promise，我们可以处理视频不能播放的情况，比如在不能自动播放时，显示播放按钮：

```js
var promise = document.querySelector('video').play();

if (promise !== undefined) {
  promise.then(_ => {
    // Autoplay started!
  }).catch(error => {
    // Autoplay was prevented.
    // Show a "Play" button so that user can start playback.
  });
}
```

这里有个小技巧，先让视频静音自动播放，再引导用户点击**开启声音**。

```html
<video id="video" muted autoplay />
<button id="unmuteButton"></button>

<script>
  unmuteButton.addEventListener('click', function() {
    video.muted = false;
  });
</script>

```

参考资料：

- https://developers.google.com/web/updates/2017/09/autoplay-policy-changes

#### 其他浏览器

安卓设备的浏览器比较杂乱，对自动播放的限制策略不尽相同，研究起来需要花费大量的时间。总结，无论是否支持自动播放，我们都需要处理视频播放异常的情况。

### 内联播放

在早期的浏览器，video播放器是在页面顶层的，无法视频上面添加设计元素。现在，各大移动端浏览器逐渐支持了内联播放：

- ios 4.0开始支持内联播放，在video元素的`webkit-playsinline`属性就可以了（ios 10+已移除前缀`webkit-`）。
- 安卓内置浏览器也逐渐支持设置`playsinline`内联播放（安卓浏览器碎片化比较严重，存在各种问题）。
- 需要特别注意的是，微信，QQ内置浏览器，QQ浏览器都是用X5内核，并有一套自己的H5同层播放器接入规范。详情查看TBS文档： https://x5.tencent.com/tbs/guide/video.html



**一些问题和建议**： 

- ios端的QQ浏览器（最新版9.6.1）对内联播放支持有问题，video还是处于顶层。
- 各大浏览器对内联播放的支持程度不一样，建议根据产品的运行环境比例，再单独做HACK。
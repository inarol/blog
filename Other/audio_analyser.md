# 基于puppeteer离线解析音频频率信息

## 目录

* [背景](#背景)
* [Puppeteer](#puppeteer)
* [加载音频](#加载音频)
* [采集音频频率数据](#采集音频频率数据)
* [总结](#总结)

## 背景

最近在做微信小游戏时，需要实现一个讲音乐图形化的需求，网上很多用`Web Audio API`就可以轻松实现把音频转换为图形的例子，比如[Web Audio API绘制可视化音乐](https://www.chunqiuyiyu.com/2017/04/draw-visual-music-with-web-audio-api.html)这个，但微信小游戏环境目前还不支持`Web Audio API`，所以想到了办法只能是做离线分析音频的频率了。  

关离线要怎么做？找个真实的浏览器环境再写段JavasScript脚本去采集？不不不，感觉实现起来就很麻烦，但思路基本是正确的，只是我们不需要用真实的浏览器去解析，大google开源了一个`headless Chrome`的工具[Puppeteer](https://github.com/GoogleChrome/puppeteer)，这样我们就可以集成到nodejs中，写出基于 `Nodejs + Puppeteer`的离线解析音频频率的工具。

## Puppeteer

首先创建一个能提供`AudioContext`浏览器环境，并打开一个空白页面：

```js
const browser = await puppeteer.launch({
  ignoreDefaultArgs: ['--mute-audio'], // 默认是静止声音的
});
const page = await browser.newPage();
```

解析音频频率信息的代码是在浏览器中执行的，所以需要用到`page.evaluate`：

```js
// nodejs环境
await page.evaluate(() => {
  // 浏览器环境
  const audioContext = new AudioContext();
});
```

在这里我们的已经成功创建`audioContext`上下文了，剩下的就是怎样解析音频的事情了。

## 加载音频

首先作为一个工具来说，我们的音频基本是放在本地的，所以需要用nodejs的`readFile`来读取本地文件，这里又抛出了另外一个问题，在Nodejs环境下，怎么样把读取的音频文件给到浏览器环境呢？别担心，Puppeteer已经提供了`exposeFunction`来处理这种特殊的交互了，首先我们在nodejs环境下，定义一个用来加载音频文件函数`loadFile`，的，注意文件格式选择`base64`文本格式，`buffer`格式是无法传递给浏览器环境的：

```js
await page.exposeFunction('loadFile', (filePath) => {
  return new Promise((resolve, reject) => {
    fs.readFile(path.resolve(filePath), 'base64', (err, data) => {
      err ? reject(err) : resolve(data)
    });
  });
});
```

然后我们在浏览器新建一个函数`decode`，用来解码nodejs传递过来的音频文本信息，最后返回一个`AudioBuffer`：

```js
const decode = () => {
  // base64转buffer
  const base64ToBuffer = (base64) => {
    var binaryString = atob(base64);
    var len = binaryString.length;
    var bytes = new Uint8Array(len);
    for (var i = 0; i < len; i++) {
      bytes[i] = binaryString.charCodeAt(i);
    }
    return bytes. buffer;
  };
  return new Promise((resolve, reject) => {
    audioContext.decodeAudioData(base64ToBuffer(audioFile), (buffer) => {
      resolve(buffer)
    }, (err) => {
      reject(err);
    });
  });
};
```

然后再创建一个音频`bufferSource`，并把解码后的`AudioBuffer`添加到bufferSource：

```js
const audioFile = await loadFile('./audio/bgm.mp3');
const audioBuffer = await decode(audioFile);
const bufferSource = audioContext.createBufferSource();
bufferSource.buffer = audioBuffer;
```

## 采集音频频率数据

首先我们需要创建一个`analyserNode`，用来处理获取音频频率数据：

![](./img/audio_analyser-1.png)

```js
const analyserNode = audioContext.createAnalyser();
bufferSource.connect(analyserNode);
analyserNode.connect(audioContext.destination);
bufferSource.start();
```
然后跑一下代码，这时候已经可以正常听到bgm响起了。等等，好像有点不对，难道我要等一首bgm全部播完才算采集完吗？没错，`AudioContext`确实只能是实时采集音频数据。但是好在`Web Audio`提供了`OfflineContext`，我们可以在设备不播放音乐的时候，同样采集到数据。我们只有稍微修改一下代码，首先创建一个`offlineContext`上下文：

```js
const offlineContext = new OfflineAudioContext(
  audioBuffer.numberOfChannels, // 声道数
  audioBuffer.length, // buffer大小
  audioBuffer.sampleRate, // 采样率
);
```

然后新增一个`scriptProcessorNode`，用来实时分析音频信息：

```js
const scriptProcessorNode = offlineContext.createScriptProcessor();
```

最后我们的采集流程是这样的：

![](./img/audio_analyser-2.png)

也就是analyserNode与输出音频直接新增了一个`scriptProcessorNode`模块，同时该模块注册一个回调方法去采集过程中的音频数据：

```js
const analyseData = {};
scriptProcessorNode.onaudioprocess = () => {
  const currentTime = Math.floor(offlineContext.currentTime * 100);
  if (currentTime % 5 === 0) {
    // 采集16位
    const dataArray = new Uint8Array(16);
    analyserNode.getByteFrequencyData(dataArray);
    analyseData[(currentTime / 100).toFixed(2)] = Array.from(dataArray);
  }
};
```

然后再注册一个`oncomplete`方法，把采集到的数据通过exposeFunction暴露到浏览器环境的方法`writeFile`输出到json文件。

最后再播放并渲染音频：

```js
bufferSource.start();
offlineContext.startRendering();
```

## 总结

很多程序语言都能解析音频的频率信息，包括纯nodejs、python都可以，本文只是用Puppeteerr这个无界面浏览器去做音频解析，最后我把以上的处理方案封装成一个命令工具：`audio-analyser-cli`，感兴趣可以去我的github了解：[https://github.com/inarol/audio-analyser-cli](https://github.com/inarol/audio-analyser-cli)
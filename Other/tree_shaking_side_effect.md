# tree-shaking处理副作用代码

## 目录

* [什么是副作用代码](#什么是副作用代码)
* [如何解决](#如何解决)
  * [package.json设置](#packagejson设置)
  * [module.rules指定模块](#modulerules指定模块)

## 什么是副作用代码
webpack 2.x开始就支持了`Tree-Shaking`特性，该特性利用了ES6的模块依赖，如果严格按照ES6的模块化开发，一般情况下它是很好用的。但是如果遇到副作用代码的话，情况就不一样了：

```js
// a.js
export const a = () => {
  console.log('This is a module.');
}
```

```js
// b.js
export const b = () => {
  console.log('This is b module.');
}
console.log(b);
```

```js
// index.js
import { a } from './a.js';
import { b } from './b.js' ;
a();
```

`b.js`模块虽然没被`index.js`模块引用，但**b.js **模块采用被打包进来，这是因为**b.js**模块包含了副作用代码`console.log(b);`。出于打包安全考虑，而`webpack`默认是不会自己删除带有副作用代码的模块，只有开发者才知道需不需要这些副作用的代码。

## 如何解决

- `package.json`指定模块
- `module.rules`指定模块

### package.json设置

`package.json`的`sideEffects`属性有两种设置方式:

1. 直接设置为`false`：
```json
"sideEffects": "false"
```
这种方式是告诉webpack，所有代码模块里没有副作用代码，可以安全地删除未用到的模块，所以上面的**b.js**模块不会被包含进来，副作用代码`console.log(b);`也不会被执行。

2. 显式指定那些模块有副作用代码：

```
"sideEffects": [
  "!b.js"
]
```

`sideEffects`是一个数组列表，而且遵循`相对路径`、`绝对路径`和`glob 模式`。这种方式则是告诉webpack，不需要**b.js**模块，即使它有副作用代码。

### `module.rules`指定模块

在webpack的`rules`配置，也可配置带有副作用代码的模块：

```js
module: {
  rules: [
    {
      include: path.resolve('./', 'b.js'),
      sideEffects: false,
    },
  ],
},
```
上面这种方式则是告诉webpack，我不需要这个模块了，即使它有副作用代码，另外这种方式的设置优先级比`package.json`高。
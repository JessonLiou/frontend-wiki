## Webpack 原理解析 - 模块化

### 基础
`a.js`
```js
function getName () {
  console.log('This is a.');
}

module.exports = {
  getName: getName
}
```

`index.js`：入口文件
```js
var a = require('./a')
a.getName()
```

webpack打包后只会生成一个文件，该文件内容类似如下：

`注意`：该文件内容只是根据原理自己写的代码，并非真实打包后的代码，打包后的代码过于复杂，不清晰，这里只是想阐述原理。

```js
(function(modules) {
  var installedModules = {};

  // 实现了require方法加载模块
  function require(moduleId) {
    // 检查是否已经加载过
    if(installedModules[moduleId]) {
    	return installedModules[moduleId].exports;
    }

    // 创建模块并且加入到缓存中
    var module = installedModules[moduleId] = {
    	exports: {}
    };

    // 将脚本的上下文指定为该模块的本身，并执行该脚本
    modules[moduleId].call(module.exports, module, module.exports, require);

    // 所有放在module.exports中的对象都将被返回
    return module.exports;
  }

  // 默认执行moduleId为0的脚本
  return require(0);
})([
  (function(module, exports, require) {
    /* index.js 文件内容被复制到了这里 */
    var a = require(1)
    a.getName()
  }),
  (function(module, exports) {
    /* a.js 文件内容被复制到了这里 */
    function getName () {
      console.log('This is a.');
    }

    module.exports = {
      getName: getName
    }
  })
]);
```

### 懒加载
`a.js`
```js
function getName () {
  console.log('This is a.');
}

module.exports = {
  getName: getName
}
```

`b.js`
```js
function getName () {
  console.log('This is b.');
}

module.exports = {
  getName: getName
}
```

`index.js`：入口文件
```js
var a = require.ensure(['./b'], function() {
  var a = require('./a')
  a.getName()
})
```

webpack打包后只会生成2个文件，将 `a.js` 和 `b.js` 单独提取出来合并成了一个文件：

入口文件
```js
(function (modules) {
  var installedModules = {}, // 模块缓存
      installedChunks = {}; // chunk 缓存

  // jsonp回调函数
  function webpackJsonpCallback(data) {
    var chunkIds = data[0];
    var moreModules = data[1];
    var moduleId, i = 0, resolves = [];

    // 标记chunk为加载完成
    for (;i < chunkIds.length; i++) {
      chunkId = chunkIds[i];
      resolves.push(installedChunks[chunkId][0]);

      installedChunks[chunkId] = 0;
    }

    // 将该模块加入到modules中
    for (moduleId in moreModules) {
      modules[moduleId] = moreModules[moduleId];
    }

    oldJsonpFunction(data)

    // resolve promise
    while (resolves.length) {
      resolves.shift()()
    }
  }

  // 实现了require方法加载模块
  function require(moduleId) {

    // 检查是否已经加载过
    if(installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }

    // 创建模块并且加入到缓存中
    var module = installedModules[moduleId] = {
      exports: {}
    };

    // 将脚本的上下文指定为该模块的本身，并执行该脚本
    modules[moduleId].call(module.exports, module, module.exports, require);

    // 所有放在module.exports中的对象都将被返回
    return module.exports;
  }

  require.ensure = function (chunkId) {
    var promises = [];
    var installedChunkData = installedChunks[chunkId];

    // 判断缓存中是否存在
    if (installedChunkData !== 0) { // 为 0 代表已经加载完成
      if (installedChunkData) {
        // 正在加载中
        promises.push(installedChunkData[2]);
      } else {
        // 创建promise并加入到缓存中
        var promise = new Promise(function(resolve, reject) {
          installedChunkData = installedChunks[chunkId] = [resolve, reject];
        });
        promises.push(installedChunkData[2] = promise);

        // 准备加载chunk
        var script = document.createElement('script');
        var onScriptComplete;
        script.charset = 'utf-8';
        script.timeout = 120;
        script.src = "js/" + chunkId + ".js";
        var error = new Error();

        onScriptComplete = function (event) {
          // 释放资源
          script.onerror = script.onload = null;
          // 清楚定时器
          clearTimeout(timeout);

          var chunk = installedChunks[chunkId];
          if (chunk !== 0) {
            if (chunk) {
              var errorType = event && (event.type === 'load' ? 'missing' : event.type);
              var realSrc = event && event.target && event.target.src;
              error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
              error.name = 'ChunkLoadError';
              error.type = errorType;
              error.request = realSrc;
              chunk[1](error);
            }
          }
          installedChunks[chunkId] = undefined;
        }

        // 设置超时定时器
        var timeout = setTimeout(function () {
          onScriptComplete({ type: 'timeout', target: script });
        }, 120000);
        // 绑定出错、完成的处理函数
        script.onerror = script.onload = onScriptComplete;
        // 开始加载
        document.head.appendChild(script);
      }
    }
    return Promise.all(promises);
  }

  var jsonpArray = window['webpackJsonp'] = window["webpackJsonp"] || [];
  var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
  jsonpArray.push = webpackJsonpCallback;

  // 默认执行moduleId为0的脚本
  return require(0);
})([
  (function(module, exports, require) {
    /* index.js 文件内容被复制到了这里 */
    require.ensure(1).then(function () {
      var a = require(2)
      a.getName()
    })
  })
]);
```

`a.js` 和 `b.js` 合并后的文件
```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
  [1],
  [
    ,
    // a.js
    (function(module, exports) {

      function getName () {
        console.log('This is a.');
      }

      module.exports = {
        getName: getName
      }

    }),
    // b.js
    (function(module, exports) {

      function getName () {
        console.log('This is a.');
      }

      module.exports = {
        getName: getName
      }

    })
  ]
]);
```

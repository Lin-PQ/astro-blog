---
title: '纯前端实现发版通知'
description: '手写plugin实现发版通知'
pubDate: 'July 6 2024'
heroImage: '../../assets/images/te.jpg'
category: 'Project'
tags: ['Webpack']
---

## 背景介绍

> monorepo项目发版是轮询接口返回的hash值，如果hash值变化则通知发版，实际情况下，只是更新了A模块，用户仅使用B模块也会收到发版通知。
>
> 期望调整：前端每次模块打包生成hash值，当前模块hash值变化则提示发版。​

### 注意事项

1. hash文件生成要在打包构建之后
2. hash文件存储路径要跟轮询时请求的路径一致

```javascript
 webpackConfig.output = {
    ...webpackConfig.output,
    path: path.resolve(__dirname, 'dist'), // 修改输出文件目录
    // 注意线上资源访问路径
    publicPath: process.env.NODE_ENV === 'production' ? '/mfs/asset/' : '/',
    library: `${name}-[name]`,
    libraryTarget: 'umd',
    jsonpFunction: `webpackJsonp_${name}`,
    globalObject: 'window',
};
```

### 过程描述

线上代码资源请求路径在域名/mfs/模块名下,该路径下以static/js，static/css区分不同类型的静态资源

打包后生成version.json在static目录下

之后在模块顶层layout.tsx轮询version.json，检测到变化则提示用户刷新

## 实现

### 自定义Plugin：GenerateFileHashPlugin

```javascript
const path = require('path');
const fs = require('fs');

class GenerateFileHashPlugin {
  filePath;
  constructor(filePath) {
    this.filePath = filePath;
  }
  apply(compiler) {
    // 文件生成时机在构建之后,否则缺少static目录会报错
    compiler.hooks.afterEmit.tap('GenerateFileHashPlugin', (compilation) => {
      if (process.env.NODE_ENV !== 'production') return;
      // 该路径同线上资源请求路径一致
      const configPath = path.join(
        compiler.options.output.path,
        'static',
        this.filePath
      );
      fs.writeFileSync(
        configPath,
        JSON.stringify({ hash: compilation.hash }, null, 2)
      );
    });
  }
}
module.exports = GenerateFileHashPlugin;
```

### 载入Plugin

```javascript
const GenerateFileHashPlugin = require('./src/utils/generateFileHashPlugins');
...
module.exports = {
    webpack:{
        plugins:[new GenerateFileHashPlugin('version.json'),]
    }
}
```

### 轮询updater

```typescript
interface Options {
  timer?: number;
}

export class Updater {
  oldHash: string; // 存储第一次值也就是script 的hash 信息

  newHash: string; // 获取新的值 也就是新的script 的hash信息

  dispatch: Record<string, Function[]>; // 小型发布订阅通知用户更新了

  constructor(options?: Options) {
    this.oldHash = '';

    this.newHash = '';

    this.dispatch = {};

    this.init(); // 初始化

    this.timing(options?.timer); // 轮询
  }

  async init() {
    const html: string = await this.getHtml();
    this.oldHash = this.parserScript(html);
  }

  async getHtml() {
    if (process.env.NODE_ENV !== 'production') return undefined;
    const json = await fetch(
      process.env.NODE_ENV !== 'production'
        ? '/version.json'
        : '/mfs/asset/static/version.json',
      {
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
        },
      }
    ).then((res) => {
      return res.json();
    }); // 读取index json

    return json;
  }

  parserScript(json: any) {
    return json?.hash || '';
  }

  // 发布订阅通知
  on(key: 'no-update' | 'update', fn: Function) {
    (this.dispatch[key] || (this.dispatch[key] = [])).push(fn);
    return this;
  }

  compare(oldString: string, newString: string) {
    // 如果新旧length 一样无更新
    if (oldString === newString || (!oldString && !newString)) {
      this.dispatch['no-update']?.forEach((fn) => {
        fn();
      });
    } else {
      // 否则通知更新
      this.dispatch?.update?.forEach((fn) => {
        fn();
      });
    }
  }

  refresh() {
    this.oldHash = this.newHash;
  }

  timing(time = 60000) {
    console.log(time, 'time');
    // 轮询
    setInterval(async () => {
      const newHtml = await this.getHtml();
      this.newHash = this.parserScript(newHtml);
      this.compare(this.oldHash, this.newHash);
    }, time);
  }
}
```

### 通知提示

```typescript
import { Updater } from 'src/utils/updater';
const updater = new Updater({ timer: 60000 });
updater.on('update', () => {
  console.warn('更新');
  setIsRefresh(true);
  ... 
  // 自定义提示方式
});
```

## 小结

做的事情很简单，发版记录，轮询比较，通知发版。

但是该功能几乎在每一个项目上都能够使用，无论是monorepo，单模块，小程序等项目都能基于该思路实现纯前端的发版通知。

另外，看到一些文章里是直接比较构建后文件的hash指纹，后续可以再做调整。

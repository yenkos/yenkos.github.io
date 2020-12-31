---
title: rollup初体验
categories: ['工程化']
tags: ['工程化']
---


进入一个目录，npm init 进行初始化，可以一路回车<br />
<br />安装ts<br />npm install -D typescript      <br />
<br />生存配置文件<br />./node_modules/.bin/tsc --init

![image.png](https://cdn.nlark.com/yuque/0/2020/png/203222/1609398709224-7e2a4465-2939-43ac-94c8-18ea8d71a132.png#align=left&display=inline&height=152&margin=%5Bobject%20Object%5D&name=image.png&originHeight=152&originWidth=754&size=14915&status=done&style=none&width=754)<br />
<br />根目录下新建一个rollup配置文件<br />rollup.config.js

添加以下内容
```typescript
import clear from 'rollup-plugin-clear'; // 转换cjs
import commonjs from 'rollup-plugin-commonjs'; // 转换cjs
import { terser } from 'rollup-plugin-terser'; // 压缩，可以判断模式，开发模式不加入到plugins
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import typescript from 'rollup-plugin-typescript';

export default {
  input: 'src/index.ts', // 源文件入口
  output: [
    {
      file: 'dist/browser-version.esm.js', // package.json 中 "module": "dist/browser-version.esm.js"
      format: 'esm', // es module 形式的包， 用来import 导入， 可以tree shaking
      sourcemap: false
    }, {
      file: 'dist/browser-version.cjs.js', // package.json 中 "main": "dist/browser-version.cjs.js",
      format: 'cjs', // commonjs 形式的包， require 导入
      sourcemap: false
    }, {
      file: 'dist/browser-version.umd.js',
      name: 'GLWidget',
      format: 'umd', // umd 兼容形式的包， 可以直接应用于网页 script
      sourcemap: false,
    }
  ],
  plugins: [
    clear({
      targets: ['dist']
    }),
    resolve(),
    babel({
      exclude: 'node_modules/**'
    }),
    typescript(),
    commonjs(),
    terser(),
  ]
}
```

<br />修改packagejson
```json
{
  "name": "browser-version-tool-html",
  "version": "1.0.0",
  "description": "Print tips on outdate browser. ",
  "main": "dist/browser-version.cjs.js",
  "module": "dist/browser-version.esm.js",
  "browser": "dist/browser-version.umd.js",
  "scripts": {
    "build": "rollup -c",
    "dev": "rollup -c -w",
    "test": "ts-node test/test.ts",
    "pretest": "npm run build"
  },
  "author": "Yenkos",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.12.10",
    "@rollup/plugin-html": "^0.2.0",
    "@types/ms": "^0.7.31",
    "babel-plugin-external-helpers": "^6.22.0",
    "babel-preset-latest": "^6.24.1",
    "rollup": "^2.35.1",
    "rollup-plugin-babel": "^4.4.0",
    "rollup-plugin-clear": "^2.0.7",
    "rollup-plugin-commonjs": "^10.1.0",
    "rollup-plugin-hash": "^1.3.0",
    "rollup-plugin-node-resolve": "^5.2.0",
    "rollup-plugin-terser": "^7.0.2",
    "rollup-plugin-typescript": "^1.0.1",
    "rollup-plugin-uglify": "^6.0.4",
    "ts-node": "^9.1.1",
    "tslib": "^2.0.3",
    "typescript": "^4.1.3"
  },
  "types": "dist/index.d.ts"
}

```

<br />npm i 安装依赖<br />
<br />进入src/index.ts 编写代码<br />
<br />rollup 打包<br />npm run build<br />
<br />

<a name="CPAFy"></a>
### 发布包到npm
首先需要注册npm账号

npm adduser<br />npm login<br />npm publish<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/203222/1609410556289-98a91d3e-a6be-45db-b790-a053c908d720.png#align=left&display=inline&height=375&margin=%5Bobject%20Object%5D&name=image.png&originHeight=750&originWidth=1152&size=173447&status=done&style=none&width=576)<br />


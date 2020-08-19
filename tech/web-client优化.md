# web-client优化

2019/07/05

## 优化成果

build time: 120s --> 60s

rebuild time: 8s --> 2s

首屏体积：10mb --> 7mb / 4mb --> 2.5mb

首屏时间：DOMContentDownload: 1.32s --> 1s，loaded: 1.71s --> 1.35s



## 优化清单

### 升级环境

webpack@4.32.2

webpack-cli@3.3.2

node@v10.16.0

npm@6.9.0

#### loaders

image-webpack-loader@5.0.0

html-webpack-plugin@4.0.0-beta.5

others@latest

#### plugins

circular-dependency-plugin

add-asset-html-webpack-plugin

mini-css-extract-plugin



### 打包脚本优化

去掉build的config，简化打包脚本为base, dev, build三个脚本，使用webpack-dev-server代替webpack-dev-middleware

- `start`  开发使用的脚本，去掉不必要的输出

- `start:analyze` 分析优化开发环境时使用的脚本，更详细的log/process/profile
- `build`  构建脚本
- `build:dll`  构建webpack.dll文件
- `build:analyze` 分析优化构建的脚本，bundle分析工具，stats.json输出，更详细的log/process/profile，http-server预览输出，检查循环依赖



### 预编译第三方包

webpackdll, vendor + echarts，add-asset-html-webpack-plugin自动添加到html-webpack-plugin生成的html中



### 使用构建缓存

cache-loader 



### 资源处理

#### CSS处理

`extract-text-webpack-plugin`替换为`mini-css-extract-plugin`

#### 图片处理

使用`image-webpack-loader`压缩文件，eg: bg.png 1.2m --> 470k

#### 字体图标

`icon-font`使用cdn资源，去掉font-awesome引用，使用icon-font代替



### 拆包

runtime
common: (assets|common|api|router|vuex)
styles
【未应用】：相关模块打包到一起二级菜单打到一个包中, /* webpackChunkName: "" */ 
【未应用】：启用预加载：home, order, credential /* webpackPrefetch: "" */ 
启用：.babelrc comments:true。通过babel去掉注释关闭相关功能



### 减少entry的体积

element-variables.scss使用插件从entry中抽离出来减少app.js的体积



### 删除不再需要的工程依赖

npm prune --dry-run --json



### 环境模拟

[http://vhsupply.dev.viewchain.net/?env=staging#/invoice/invoiceBill](http://vhsupply.dev.viewchain.net/?env=staging#/invoice/invoiceBill)



### 代码相关优化

**去除死循环：circular-dependency-plugin**

**去掉表达式import**

去掉vue警告：scope==> slot-scope，keys
去掉无用代码：bg.png没有用到，重置密码组件没有用到
启用预加载：首页，供采，证件模块



## 过程

1. 升级`webpack`

   ```shell
   npm install webpack webpack-cli --save-dev
   ```

   安装不成功，发现依赖的worker-farm@1.7.0在 nexus仓库里没有，手动install一下这个版本解决了

2. 更新所有的loader

3. 发现循环依赖报错
  `circular-dependency-plugin`

4. html-webpack-plugin报错

  ```shell
  npm i -D html-webpack-plugin@next
  # html-webpack-plugin@4.0.0-beta.5
  # chunksSortMode: 'none'
  ```

5. dev-server 
  warning通过stats.warning = false 关闭
  ERR_TLS_CERT_ALTNAME_INVALID："changeOrigin": true

  ```shell
  Invalid Host header
  # disableHostCheck: true,
  ```






babel升级：

babel改成require设为false



## 继续优化

首屏优化，build，rebuild时间优化
使用浏览器工具: pagespeed

减少entry体积：使用import()异步加载依赖
去掉require
使用lodashes代替lodash + 全局import
去掉无用的代码，去掉不必要的包
去掉对第三方包src资源的引入，element-ui
精确的import--
分析包结构提取公共部分

/* webpackPrefetch: true */

element-variables.scss packages体积优化

去掉babel-polyfill



## 参考+工具

https://webpack.js.org/guides/build-performance/
https://webpack.jakoblind.no/optimize/
http://webpack.github.io/analyse/


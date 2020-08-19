### Webpack 2.0

[TOC]

Webpack是JS的打包工具，从entry开始递归的把依赖的资源module( js, css等)生成dependency graph，除去不在依赖中的，生成打包好的文件bundle/trunck。支持开发环境的dev_server。webpack 使用ES6的import/export依赖管理，但不支持其他的ES6的特性，需要使用babel loader来支持其他ES6的特性。



#### 安装Webpack

```bash
#局部安装 *推荐
npm i webpack -D

#全局安装
npm i webpack -g
```



#### 配置文件

webpack会使用`webpack.config.js`作为默认的配置文件

```bash
#使用cli
webpack entry.js dist/bundle.js

# 使用config
webpack --config xxxx.js

# 在npm中使用webpack, tips: npm中可以直接调用模块工具 '.bin'被放到环境变量中
scripts:{
	build: 'webpack'
}

```



#### Entry

Entry是webpack开始解析依赖关系的入口

```javascript
// 单一入口
entry: './path/to/my/entry/file.js'

//单页程序，依赖包单独打包。需要CommonsChunkPlugin
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }

//多页程序，每个html相关js打一个包
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }

```



#### output

output是关于打包后的文件的设置，文件的位置，文件名，是否每个entry打一个包等。

```javascript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  /*entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }*/
  
  output: {
    path: path.resolve(__dirname, 'dist'),	//打包文件生成的目录
    // [id] [name] [hash] [chunkHash]
    filename: 'my-first-webpack.bundle.js' 
    //chunkFilename
    publicPath:	// 被插件使用，用来再生产环境替换资源的URL。url_loader
  }
};
```



#### loader

webpack只认识javascript，如果想将其他资源打包就需要将其实现翻译成javascript代码。loader就是处理module的预处理器(per file)。

```javascript
//...
// loader需要提前安装
module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  }
//...
```



#### plugin

loader是按文件执行的，而plugin是按bundle、chunk执行的。使用前需要require

```javascript
  const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
  const webpack = require('webpack'); //to access built-in plugins
  const path = require('path');
...
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
```



#### Config reference

```javascript
const path = require('path');

module.exports = {
  // click on the name of the option to get to the detailed documentation
  // click on the items with arrows to show more examples / advanced options

  entry: "./app/entry", // string | object | array
  // Here the application starts executing
  // and webpack starts bundling

  output: {
    // options related to how webpack emits results

    path: path.resolve(__dirname, "dist"), // string
    // the target directory for all output files
    // must be an absolute path (use the Node.js path module)

    filename: "bundle.js", // string
    // the filename template for entry chunks

    publicPath: "/assets/", // string
    // the url to the output directory resolved relative to the HTML page

    library: "MyLibrary", // string,
    // the name of the exported library

    libraryTarget: "umd", // universal module definition
    // the type of the exported library

    /* Advanced output configuration (click to show) */
  },

  module: {
    // configuration regarding modules

    rules: [
      // rules for modules (configure loaders, parser options, etc.)

      {
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],
        // these are matching conditions, each accepting a regular expression or string
        // test and include have the same behavior, both must be matched
        // exclude must not be matched (takes preferrence over test and include)
        // Best practices:
        // - Use RegExp only in test and for filename matching
        // - Use arrays of absolute paths in include and exclude
        // - Try to avoid exclude and prefer include

        issuer: { test, include, exclude },
        // conditions for the issuer (the origin of the import)

        enforce: "pre",
        enforce: "post",
        // flags to apply these rules, even if they are overridden (advanced option)

        loader: "babel-loader",
        // the loader which should be applied, it'll be resolved relative to the context
        // -loader suffix is no longer optional in webpack2 for clarity reasons
        // see webpack 1 upgrade guide

        options: {
          presets: ["es2015"]
        },
        // options for the loader
      },

      {
        test: "\.html$",

        use: [
          // apply multiple loaders and options
          "htmllint-loader",
          {
            loader: "html-loader",
            options: {
              /* ... */
            }
          }
        ]
      },

      { oneOf: [ /* rules */ ] },
      // only use one of these nested rules

      { rules: [ /* rules */ ] },
      // use all of these nested rules (combine with conditions to be useful)

      { resource: { and: [ /* conditions */ ] } },
      // matches only if all conditions are matched

      { resource: { or: [ /* conditions */ ] } },
      { resource: [ /* conditions */ ] },
      // matches if any condition is matched (default for arrays)

      { resource: { not: /* condition */ } }
      // matches if the condition is not matched
    ],

    /* Advanced module configuration (click to show) */
  },

  resolve: {
    // options for resolving module requests
    // (does not apply to resolving to loaders)

    modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    // directories where to look for modules

    extensions: [".js", ".json", ".jsx", ".css"],
    // extensions that are used

    alias: {
      // a list of module name aliases

      "module": "new-module",
      // alias "module" -> "new-module" and "module/path/file" -> "new-module/path/file"

      "only-module$": "new-module",
      // alias "only-module" -> "new-module", but not "module/path/file" -> "new-module/path/file"

      "module": path.resolve(__dirname, "app/third/module.js"),
      // alias "module" -> "./app/third/module.js" and "module/file" results in error
      // modules aliases are imported relative to the current context
    },
    /* alternative alias syntax (click to show) */

    /* Advanced resolve configuration (click to show) */
  },

  performance: {
    hints: "warning", // enum
    maxAssetSize: 200000, // int (in bytes),
    maxEntrypointSize: 400000, // int (in bytes)
    assetFilter: function(assetFilename) { 
      // Function predicate that provides asset filenames
      return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
  },

  devtool: "source-map", // enum
  // enhance debugging by adding meta info for the browser devtools
  // source-map most detailed at the expense of build speed.

  context: __dirname, // string (absolute path!)
  // the home directory for webpack
  // the entry and module.rules.loader option
  //   is resolved relative to this directory

  target: "web", // enum
  // the environment in which the bundle should run
  // changes chunk loading behavior and available modules

  externals: ["react", /^@angular\//],
  // Don't follow/bundle these modules, but request them at runtime from the environment

  stats: "errors-only",
  // lets you precisely control what bundle information gets displayed

  devServer: {
    proxy: { // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
    compress: true, // enable gzip compression
    historyApiFallback: true, // true for index.html upon 404, object for multiple paths
    hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
    https: false, // true for self-signed, object for cert authority
    noInfo: true, // only errors & warns on hot reload
    // ...
  },

  plugins: [
    // ...
  ],
  // list of additional plugins


  /* Advanced configuration (click to show) */
}
```



#### 常用插件

```javascript
//抽离css
let ExtractTextPlugin = require('extract-text-webpack-plugin');
//多入口
new webpack.optimize.CommonsChunkPlugin(options)
//压缩
new webpack.optimize.UglifyJsPlugin([options])
//去掉重复依赖
new webpack.optimize.DedupePlugin()
//定义全局变量
new webpack.DefinePlugin()
//拷贝
new CopyWebpackPlugin()

// 开发 - 自动打开浏览器
const OpenBrowserPlugin = require('open-browser-webpack-plugin');

// HMR插件
new webpack.HotModuleReplacementPlugin()

```



通过definePlugin定义的变量可以在代码中使用。条件分支中else里的代码不会被生成到dist文件中。可以运用这个技术使用同一套代码为不同环境打包特定的dist文件。



#### Examples

// 阮一峰
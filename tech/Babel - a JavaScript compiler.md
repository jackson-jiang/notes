# Babel - a JavaScript compiler

> Use next generation JavaScript, today.

新的javascript标准提供了更高效的语法(syntax)和特性(features)，但是由于标准发布和各个目标环境（浏览器，node...）的实现存在延迟和差异性，导致前端工程师无法立刻使用新的标准来书写代码。使用babel允许按照最新的标准（甚至是提案阶段的标准）来编写代码，在代码运行前编译成目标环境可以支持的代码。使用特定的presets(一组语法规则)，还可以支持react, flow, typescript等语言，甚至可以编写自己的transformer实现一种全新的语言。

## Hello babel!!

### 创建目录

```shell
babel-practice
|-src 
|-dist
|-babel.config.js
|-package.json
|-index.html
```

### 安装工具

```shell
# 需要提前安装node@10^ TLS版本
npm init -y
npm install --save-dev @babel/core @babel/cli
npm install --save-dev @babel/plugin-transform-template-literals
```

### 代码和配置

#### ./babel.config.js

```js
// 配置babel.config.js
const plugins = [
  '@babel/plugin-transform-template-literals'
];

module.exports = { plugins };
```



#### ./src/index.js

```js
const hello = ({
    text = 'babel'
} = {}) => console.log(`hello ${text}`);

hello();
```



#### ./index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./dist/index.js"></script>
</head>
<body>
    
</body>
</html>
```



##### ./package.json

```json
{
  // ...
  "scripts": {
    "compile": "babel ./src -d ./dist",
    "watch": "babel ./src -d ./dist -w",
  },
  // ...
}
```



### 编译运行

```shell
# npx babel ./src -d ./lib -w
npm run watch
```



## babel的工作原理

> 为了实现`Use next generation JavaScript, today.` babel主要做了以下工作：
>
> 1. `转换语法(syntax transform)` 将ES2015+, react, typescript等语法转换为目标环境所支持的语法(AST)，如ES5
> eg： ()=>{}, class xx{}, a.b, a['b']
> 2. `源代码转换` 将原代码转换为目标环境支持的代码
> 3. `添加垫片库(polyfill)` 为目标环境添加所需的特性features，如内置对象，静态方法，实例方法，generator函数等。
> eg:  built-ins(Date, Promise，Symbol)，static methods(Array.from)，instance methods(Array.prototype.includes)
>
> PS: 
>
> 语法转换和源码转换是通过plugin实现，垫片库通过引入polyfill实现
>


$$
源码→ @babel/parser →AST→transformer[s]→AST→@babel/generator→目标代码
$$

我是谁-->**parser**--> 我(主)是(谓)谁(宾) --> **transformer[s]**--> who表 am系 I主? -->**generator**-->who am I?

### Plugins

> babel本身并没有编译的功能，编译功能是通过plugin实现的，plugin（transform plugin）其实就是编译器，和其他编译器一样编译分为三个阶段：解析，转换和输出。
>
> *ps: Syntax plugin只解析语法，使用transform plugin会自动启用对应的Syntax plugin。*每一个Plugin处理自己的一种AST Type语法。

#### 配置

```json
// node_modules中的包
{
  "plugins": ["babel-plugin-myPlugin"]
}
// 相对路径
{
  "plugins": ["./node_modules/asdf/plugin"]
}
// 简写，去掉"babel-plugin-"
{
  "plugins": ["myPlugin"]
}
// 插件配置选项
{
  "plugins": [
  	["my-plugin", {
  		// opts...
  	}, name]
  ]
}

// eg: "@babel/plugin-proposal-optional-chaining"
// npm i -D @babel/plugin-proposal-optional-chaining
// const baz = obj?.foo?.bar?.baz;
{
  "plugins": [
  		"@babel/plugin-proposal-optional-chaining"
  	]
  ]
}

```




#### 执行顺序

如果有多个plugin或者preset都匹配到了某一语法树节点“program”，plugins和presets按照以下顺序执行

- Plugins在Presets前执行
- Plugin从前到后执行
- Preset从后到前执行



#### @babel/plugin-transform-runtime插件

> 插件将正在编译文件所需的依赖指向@babel/runtime/@babel/runtime-corejs3（而不是直接插入到文件中）减少总体体积，并以别名的方式引用以避免污染全局空间
>
> 1. regenerator别名：从@babel/runtime中引用generator/async函数所依赖的帮助函数
>
> 2. core-js别名：垫片库从@babel/runtime-corejs3中以别名的方式引用
>
>    需要配置corejs和安装@babel/runtime-corejs3
>
> 3. helpers别名：从@babel/runtime中引用帮助函数



##### @babel/plugin-transform-runtime插件例子

######安装插件

```shell
# 编译时使用
npm install --save-dev @babel/plugin-transform-runtime
# 编译后代码的依赖
npm install --save @babel/runtime
# classes插件
npm install --save-dev @babel/plugin-transform-classes
# 如果配置了corejs，corejs：3
# npm install --save @babel/runtime-corejs3
```



###### ./babel.config.js

```js
// 配置babel.config.js
const plugins = [
  '@babel/plugin-transform-template-literals', '@babel/plugin-transform-classes', '@babel/plugin-transform-runtime'
];

module.exports = { plugins };
```



###### ./src/index.js

```js
class Message {
    constructor() {
        this.message = 'babel'
    }
}

const hello = (text = (new Message).message) => document.write(`<h2>hello ${text}!</h2>`);
hello();

```



###### 未使用@babel/plugin-transform-runtime插件生成的代码

> 直接添加代码到文件中

```js
function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

let Message = function Message() {
  _classCallCheck(this, Message);

  this.message = 'babel';
};

const hello = (text = new Message().message) => document.write("<h2>hello ".concat(text, "!</h2>"));

hello();
```



###### 使用@babel/plugin-transform-runtime插件生成的代码

> 以别名的方式从@babel/runtime中引用

```js
import _classCallCheck from "@babel/runtime/helpers/classCallCheck";

let Message = function Message() {
  _classCallCheck(this, Message);

  this.message = 'babel';
};

const hello = (text = new Message().message) => document.write("<h2>hello ".concat(text, "!</h2>"));

hello();
```



##### transform-runtime配置说明

```json
// transform-runtime插件配置
{
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        // 解决要编译的文件和node_modules不在同一项目，或者一个项目存在多个@babel/runtime时，@babel/runtime的解析问题
        "absoluteRuntime": false, 
        "corejs": false, // 默认假设用户已经添加了polyfill/core-js, 
        /* 
        	如果使用corejs，那么需要对应的runtime，helper,corejs,regenerator指向
        	
        	"corejs": 3
       		@babel/runtime-corejs3 instead of @babel/runtime.
       		(不污染环境)Set, Symbol, Promise等，"".includes无效
       		
        */
        // classCallCheck, extends, etc
        "helpers": true,
        // 是否为generator函数创造一个regenerator上下文

        "regenerator": true, 
        // 如果启用js模块将不会被转为cjs形式（@babel/plugin-transform-modules-commonjs）
        "useESModules": false  
      }
    ],
  ]
}
// TODO example
```



#### 插件开发

开发自己的插件

https://github.com/jamiebuilds/babel-handbook



### Presets

> presets就是一组plugins和相关配置的集合



#### 官方preset

- [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
- [@babel/preset-flow](https://babeljs.io/docs/en/babel-preset-flow)
- [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
- [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)



#### preset配置

```json
// babel-preset-可以省略
{
  "presets": [
    "@org/babel-preset-name",
    "@org/name" // equivalent
  ]
}
```



#### @babel/preset-env

preset-env是个智能的preset，它会根据项目支持的`目标环境`(targets声明)结合 [`browserslist`](https://github.com/browserslist/browserslist), [`compat-table`](https://github.com/kangax/compat-table), and [`electron-to-chromium`](https://github.com/Kilian/electron-to-chromium) 的数据，动态计算出所需要的语法和特性的集合（**基于已发布的标准**），以及所对应的plugins和polyfills

##### @babel/preset-env例子

###### 安装

```shell
npm install --save-dev @babel/preset-env core-js@3
```

###### ./babel.config.js

```js
// 配置babel.config.js
const presets = [
  ['@babel/env', {
    "useBuiltIns": "usage",
    "corejs": {version: 3, proposals: true,},
    modules: false,
    "targets": {
        "chrome": "58",
        "browsers":[
            "last 2 versions",
            "ie >= 10"
        ]
    }
  }]
];
  
module.exports = { presets };
```

###### ./src/index.js

```js
class Message {
    constructor() {
        this.message = 'babel'
    }
}

const hello = (text = (new Message).message) => document.write(`<h2>hello ${text}!</h2>`);
hello();

console.log(Array.from(new Set([1, 2, 3, 2, 1])))

async ()=>{};

"foobar".includes("foo")
```



##### 目标环境配置

###### preset-env选项

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry",
        "targets": {
            "chrome": "58", // 最低版本, ie, safari...
            "node": true, // 当前node版本
            "browsers":[
                "last 2 versions",
                "ie >= 10"
            ]
        }
      }
    ]
  ]
}
```

###### 或者.browserslistrc

```shell
# Browsers that we support

last 1 version
> 1%
maintained node versions
not dead
```

###### 或者package.json

```js
 "browserslist": [
    "last 1 version",
    "> 1%",
    "maintained node versions",
    "not dead"
  ]

```



##### 其他配置

###### useBuiltIns

```shell
# 需要安装corejs，指定corejs对应的版本
npm install --save core-js@3 regenerator-runtime
```

`useBuiltIns: false` （默认）

不会自动引入polyfills，需要手动引入@babel/polyfill/dist/polyfill.js, 也不会按照目标环境进行缩减

`useBuiltIns: entry` 

需要手动引入polyfills，会将手动引入的polyfills按照目标环境进行缩减

`useBuiltIns: usage`（推荐）

根据core-js设定的版本自动引入目标环境所需要的core-js

> 参见，[core-js](#使用)的使用
>

###### core-js

指定core-js的版本，默认为2需要安装。

```js
// 自动引入所需要proposal所对应的plugin和polyfill
useBuiltIns: usage, 
// 包括proposals
corejs: { version: 3, proposals: true }
// or
// 只包括stable
corejs: 3,
```



###### modules

`"amd" | "umd" | "systemjs" | "commonjs" | "cjs" | "auto" | false`, defaults to `"auto"`.

将ES6的module转换成其他模式，webpack中设置为false不转换来启用`tree-shake`功能



###### loose

`boolean`, defaults to `false`.

loose模式下编译的代码实现会更加简单，但是可能会和ES的标准有很大偏差，导致无法再由ES5转为ES6+



#### 创建preset

```javascript
module.exports = () => ({
  presets: [
    require("@babel/preset-env"),
  ],
  plugins: [
    [require("@babel/plugin-proposal-class-properties"), { loose: true }],
    require("@babel/plugin-proposal-object-rest-spread"),
  ],
});
```



### @babel/polyfill or corejs

> polyfill可以为不同浏览器提供包括ECMAScript，ECMAScript proposals和web standard的javascript运行环境，意味着引入polyfill你可以使用builtins `Promise`, `Symbol`, `Set`..., static methods`object.assign`和instance methods `Array.prototype.includes`，以及web标准：URL等
>
> babel7.4.0+ `@babel/polyfill` = `core-js/stable` + `regenerator-runtime/runtime` 推荐直接使用corejs
>
> corejs2只包含stable的特性，corejs3包含proposal的特性
>
> babel使用三种方式引入垫片库
>
> 1.  @babel/preset-env，详见：[@babel/preset-env](#@babel/preset-env) useBuiltIns和corejs部分的说明
> 2. @babel/transform-runtime，使用alias的方式引用runtime中的特性实现，不会污染全局环境
> 3. 直接在工程代码前引入@babel/polyfill, 比如：在webpack的entry中添加@babel/polyfill
>
> 以上三种方式使用一种即可，推荐配置：
>
> @babel/preset-env+corejs3，transform-runtime+@babel/runtime（提供regenerator等）

#### 安装

```shell
# 安装 *polyfill是运行时依赖
npm install --save @babel/polyfill
# or
npm install --save core-js regenerator-runtime
```

#### 使用

polyfill需要在项目文件前引入，可以完整引入也可以只引入某些需要的特性(推荐)

##### 浏览器

可以使用script标签直接引用`@babel/polyfill/dist/polyfill.min.js`

##### commonjs

```js
 require('core-js/features/promise'); // 精确引入
 require('@babel/polyfill'); // 完整引入
```

##### esm

```js
import 'core-js/features/promise'; // 精确引入
import '@babel/polyfill'; // 完整引入
```

##### webpack结合preset-env

```js
// 1. useBuiltIns: false或者不定义, 在entry中直接引入
module.exports = {
  entry: ["@babel/polyfill", "./app/js"],
};
// 2. useBuiltIns: 'entry'在入口的js文件中使用import引入
// 3 useBuiltIns: 'usage'
// babel会根据targets中配置的目标环境，结合browserlist和caniuse判断出目标环境缺少的最小集合并添加
```



## 配置

> babel的配置选项可以来源于很多地方可以共存：
>
> 1. programmatic options: 程序的选项@babel/core, babel-loader, babel-register...
> 2. “包级别”：`.babelrc`，`.babelrc.js`，`package.json`的`babel`设置
> 3. 项目级别：只有一个默认为当前文件夹下的`babel.config.js`文件可以通过`root`和`configFile`配置
>
> 覆盖规则：
>
> 1. 以上三种配置选项可以共存，程序的选项 > 包级 > 项目级
> 2. 按key值覆盖
> 3. plugins/presets按照标识符`唯一标识/name`替换
> 4. `parserOpts` /`generatorOpts`选项合并



### 完整的配置例子

> babel是个编译器，在没有模块管理的环境中运行要结合打包工具webpack，browserfy等使用，需要将require/import的文件导入进来



#### ./babel.config.js

```js
const presets = [
  [
    "@babel/env",
    {
      targets: {
        "browsers": [ // 浏览器
          "last 4 versions",
          "ie >= 10"
        ],
      },
      modules: false, // 不转换esm
      useBuiltIns: "usage",
      corejs: { version: "3", proposals: true },
    },
  ],
];

const plugins = [
  "@vue/transform-vue-jsx",
  "@babel/syntax-dynamic-import",
  // 不转换helpers的esm为cjs
  ["@babel/plugin-transform-runtime",{useESModules: true}],
  "@babel/plugin-proposal-class-properties",
  ["@babel/plugin-proposal-decorators", { decoratorsBeforeExport: false }],
  "@babel/plugin-proposal-do-expressions",
  "@babel/plugin-proposal-export-default-from",
  "@babel/plugin-proposal-export-namespace-from",
  "@babel/plugin-proposal-function-bind",
  "@babel/plugin-proposal-function-sent",
  "@babel/plugin-proposal-logical-assignment-operators",
  "@babel/plugin-proposal-nullish-coalescing-operator",
  "@babel/plugin-proposal-numeric-separator",
  "@babel/plugin-proposal-partial-application",
  ["@babel/plugin-proposal-pipeline-operator", { proposal: 'fsharp'}],
  "@babel/plugin-proposal-private-methods",
  "@babel/plugin-proposal-throw-expressions",
  "@babel/plugin-proposal-optional-chaining",
]

module.exports = { presets, plugins, comments: true };

```

#### ./webpack.config.js

```js
module.exports = {
    entry: {
        app: './src/index.js'
    },
    mode: 'development',
    output: {
        filename: '[name].bundle.js',
    },
    module: {
        rules: [
            {
                test: /js$/,
                use: ['babel-loader'],
                // 不编译node_modules下的js文件
                exclude: /node_modules/,
            }
        ]
    }
}
```



### 配置文件的解析

1. 在当前正在解析文件的包中的（第一个package.json）路径，文件的cwd向上寻找`包级别`的配置文件，如果该文件夹在`babelrcRoots`所定义的范围内则应用该文件
2. 使用项目级别的配置文件，应用当前路径（可通过root配置）下的`babel.config.js`（可通过configFile配置）文件。使用`console.log()`检查配置文件是否被应用。



### 配置文件的格式

`.babelrc`使用json格式

`.babelrc.js`, `babel.config.js`返回json/json5，`配置函数`



### 配置项

programmatic options：@babel/core, babel-loader, babel-register...

- Primary options：`pro-options`
  - [`cwd`](https://babeljs.io/docs/en/options#cwd) 工作目录
  - [`caller`](https://babeljs.io/docs/en/options#caller) 配置文件中可以知道调用配置文件的插件
  - [`filename`](https://babeljs.io/docs/en/options#filename) 解析的文件名
  - [`filenameRelative`](https://babeljs.io/docs/en/options#filenamerelative) 解析的文件名+全路径
  - [`code`](https://babeljs.io/docs/en/options#code)：是否输出code+map
  - [`ast`](https://babeljs.io/docs/en/options#ast): 是否输出AST
- Config Loading options
  - [`root`](https://babeljs.io/docs/en/options#root)`pro-options`项目级配置文件的根目录
  - [`rootMode`](https://babeljs.io/docs/en/options#rootmode)`pro-options`"root" | "upward" | "upward-optional"
  - [`envName`](https://babeljs.io/docs/en/options#envname)`pro-options` presets, plugins, 配置函数中api.env()引用
  - [`configFile`](https://babeljs.io/docs/en/options#configfile) `pro-options`项目级配置文件的文件名
  - [`babelrc`](https://babeljs.io/docs/en/options#babelrc) `pro-options`“包级别”配置文件的文件名
  - [`babelrcRoots`](https://babeljs.io/docs/en/options#babelrcroots)包级别配置文件的root范围 ["."，"./packages/*"]
- Plugin and Preset options
  - [`plugins`](https://babeljs.io/docs/en/options#plugins)
  - [`presets`](https://babeljs.io/docs/en/options#presets)
  - [`passPerPreset`](https://babeljs.io/docs/en/options#passperpreset) babel将preset中的每个plugin独立执行
- Config Merging options
  - [`extends`](https://babeljs.io/docs/en/options#extends) 继承其他的配置文件
  - [`env`](https://babeljs.io/docs/en/options#env) 当env的key和`envName`相同时应用，适用于生产/开发使用不同配置的情况
  - [`overrides`](https://babeljs.io/docs/en/options#overrides) 根据test的值应用配置项
  - [`test`](https://babeljs.io/docs/en/options#test) 文件名的匹配规则，匹配则应用`配置选项`
  - [`include`](https://babeljs.io/docs/en/options#include) 和test作用相同
  - [`exclude`](https://babeljs.io/docs/en/options#exclude) 和test相反，通常用到`overrides`中，一旦匹配整个配置不生效
  - [`ignore`](https://babeljs.io/docs/en/options#ignore) 如果匹配则停止编译
  - [`only`](https://babeljs.io/docs/en/options#only) 只编译
- Source Map options
  - [`inputSourceMap`](https://babeljs.io/docs/en/options#inputsourcemap) 从.map文件加载源码映射
  - [`sourceMaps`](https://babeljs.io/docs/en/options#sourcemaps) 源码映射的存在方式，inline：生成结果（对象或者文件）中添加dataUrl，true：生成独立的map。
  - [`sourceMap`](https://babeljs.io/docs/en/options#sourcemap) 同sourceMaps
  - [`sourceFileName`](https://babeljs.io/docs/en/options#sourcefilename) 
  - [`sourceRoot`](https://babeljs.io/docs/en/options#sourceroot)
- Misc options
  - [`sourceType`](https://babeljs.io/docs/en/options#sourcetype) "module" | "script"默认使用module，使用自动插入import/export
  - [` highlightCode`](https://babeljs.io/docs/en/options#highlightcode) 高亮错误消息
  - [`wrapPluginVisitorMethod`](https://babeljs.io/docs/en/options#wrappluginvisitormethod) 插件执行的包装函数，可以用作logging，analyze的切面
  - [`parserOpts`](https://babeljs.io/docs/en/options#parseropts) 
  - [`generatorOpts`](https://babeljs.io/docs/en/options#generatoropts)
- Code Generator options
  - [`retainLines`](https://babeljs.io/docs/en/options#retainlines) 尽力保证编译后的文件和源码在行数上对应
  - [`compact`](https://babeljs.io/docs/en/options#compact) `code.length > 500_000`时压缩空白
  - [`minified`](https://babeljs.io/docs/en/options#minified) 最小化
  - [`auxiliaryCommentBefore`](https://babeljs.io/docs/en/options#auxiliarycommentbefore) babel生成的代码前加注释
  - [`auxiliaryCommentAfter`](https://babeljs.io/docs/en/options#auxiliarycommentafter) babel生成的代码后加注释
  - [`comments`](https://babeljs.io/docs/en/options#comments) 是否保留注释
  - [`shouldPrintComment`](https://babeljs.io/docs/en/options#shouldprintcomment) 是否保留注释
- AMD / UMD / SystemJS module options
  - [`moduleIds`](https://babeljs.io/docs/en/options#moduleids)
  - [`moduleId`](https://babeljs.io/docs/en/options#moduleid)
  - [`getModuleId`](https://babeljs.io/docs/en/options#getmoduleid)
  - [`moduleRoot`](https://babeljs.io/docs/en/options#moduleroot)
- Options Concepts
  - [`MatchPattern`](https://babeljs.io/docs/en/options#matchpattern)
  - [Merging](https://babeljs.io/docs/en/options#merging)
  - [Plugin/Preset entries](https://babeljs.io/docs/en/options#plugin-preset-entries)
  - [Name Normalization](https://babeljs.io/docs/en/options#name-normalization)



## 工具

### 内置工具

@babel/core 包括了整个babel工作流，也就是说在@babel/core里面我们会使用到@babel/parser、transformer[s]、以及@babel/generator。

@babel/parser 将源代码解析成 AST。

@babel/generator 将AST解码生 js代码。

@babel/code-frame 用于生成错误信息并且打印出错误原因和错误行数。（其实就是个console工具类）

@babel/helpers 也是工具类，提供了一些内置的函数实现，主要用于babel插件的开发。

@babel/runtime 也是工具类，但是是为了babel编译时提供一些基础工具库。作用于transformer[s]阶段，当然这是一个工具库，如果要使用这个工具库，还需要引入@babel/plugin-transform-runtime，它才是transformer[s]阶段里面的主角。

@babel/template 也是工具类，主要用途是为parser提供模板引擎，更加快速的转化成AST

@babel/traverse 也是工具类，主要用途是来便利AST树，也就是在@babel/generator过程中生效。

@babel/types 也是工具类，主要用途是在创建AST的过程中判断各种语法的类型。

```
@babel/code-frame 为全局错误捕获工具类

@babel/core
├── 输入字符串
├── @babel/parser
│   └── @babel/template
│       └── @babel/types
├── AST
├── transformer[s]
│   └── @babel/helpers
├── AST
├── @babel/generator
│   └── @babel/traverse
```

### 其他工具

#### @babel/cli

命令行工具

```shell
# install
npm install --save-dev @babel/core @babel/cli
# run
npx babel [options]
# options
-d 输出目录，pipe
-o 输出文件，合并成一个， (< script.js) pipe
-s sourceMaps: true|inline|both
-w watch
-D 拷贝不编译的文件
--plugins 逗号分隔的list
--presets 逗号分隔的list 
--ignore 忽略编译
--minified 最小化
--delete-dir-on-start 编译前清空目标文件夹
```



#### @babel/register

在nodejs中使用，在`require("@babel/register")`之后引入的文件都会被babel编译后在引入。

```js
// npm install @babel/core @babel/register --save-dev
require("@babel/register")({
  // opts
  // ignore: [],
});;
```



#### babel-loader

与webpack集成

```shell
npm install --save-dev babel-loader
```



```js
module.exports = {
    entry: {app: './src/main.js'},
    module: {
      rules: [
          {
            test: /\.js|.mjs$/,
            use: 'babel-loader',
            exclude: /node_modules/
          }
      ]
    }
}
```



#### @babel/node

node+babel功能的REPL(Read Eval Print Loop:交互式解释器)



## babel7做的更新

1. babel7的npm包正式更名为@babel/core、@babel/cli等。
2. babel7不在支持Node.js 0.10, 0.12, 4 和 5版本。
3. babel7移除了@babel/polyfill内的polyfills支持，现在@babel/polyfill几乎只是core-js v2的映射。
   [https://github.com/babel/babel/blob/master/packages/babel-polyfill/src/index.js](https://github.com/babel/babel/blob/master/packages/babel-polyfill/src/index.js)
4. babylon现在被重命名为@babel/parser
5. 去除包名上的年份。
6. 将react和flow预设分离。

```shell
# babel-upgrade工具会更新babel相关的依赖包和配置
npx babel-upgrade --write --install
```



## ES2015+syntax

ES2015+
=>

function a(x=123, {a, b}, ...rest) {}

{[a+1]: 3}

2**3

async function* 

/.../s

 /(?<year>\d{4})/

{...obj}

try{}catch{}finally{}

class {
   x = 1
   y = ()=>{}
  static Z = 3
}

@decorator: class, attr, accessor, obj literal

do {}

obj::func
obj::func()
::obj.func // obj.func.bind(obj)
::obj.func()

function* generator() {
    console.log("Sent", function.sent);
}
const iterator = generator();
iterator.next(1); // function.sent == 1

a ||= b; // a || (a=b); a = a||b;
a &&=b;// a && (a=b); a = a&&b

a = a ?? "default";

1_000_000, 0b0000_0000 0xa0_b0_c0

obj?.['foo']?.bar?.baz

o.f(?, x, ?) // partial function

promise
  |> await
  |> x => doubleSay(x, ', ')
  |> capitalize
  |> x => x + '!'
  |> x => new User.Message(x)
  |> x => stream.write(x)
  |> await
  |> console.log;

\#private()

true && throw new Error



## core-js proposals

stage-2/3以前的垫片需要手动导入：

  import "core-js/stage/0";
  import "core-js/web";







## 参考

### 参考连接

[babel官网](https://babeljs.io/docs/en/)

[babel插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)

[ECMAScript 6 入门](http://es6.ruanyifeng.com/)

[TC39流程文档](https://tc39.es/process-document/)

[TC39](https://github.com/tc39)

[babel入门到精通](https://www.jianshu.com/p/9aaa99762a52)

[npm babel插件](https://www.npmjs.com/search?q=babel-plugin)

[the-super-tiny-compiler](https://www.npmjs.com/search?q=babel-plugin)

[browserlist](https://github.com/browserslist/browserslist)

[core-js](https://github.com/zloirock/core-js)

[polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill)

[json5](https://json5.org/)



### 技术名词

#### AST

> 抽象语法树（abstract syntax tree或者缩写为AST），或者语法树（syntax tree），是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码。
> 和抽象语法树相对的是具体语法树（concrete syntaxtree），通常称作分析树（parse tree）。
> 一般的，在源代码的翻译和编译过程中，语法分析器创建出分析树。一旦AST被创建出来，在后续的处理过程中，比如语义分析阶段，会添加一些信息。

[AST parser lab](https://esprima.org/demo/parse.html#)



#### Monorepo项目的配置

> 相对于将子项目拆分成独立的repo，Monorepo项目将子项目集合到一起使用一个repo，每个子项目有自己的配置文件

```yml
.babelrc
packages/
  mod1/
    package.json
    src/index.js
  mod2/
    package.json
    src/index.js
```

##### 方式一

使用项目级别的配置文件，通过配置overrides为不同的模块做不同的设置，为了方便在子模块执行命令`rootMode`应设置为`upword`

##### 方式二

每个子模块配置`.babelrc`, 设置每个模块的路径为babelrcRoot

```js
babelrcRoots: [
  ".",
  "packages/*",
],
```

[精读《Monorepo 的优势》](https://www.cnblogs.com/ascoders/p/10854872.html)



#### Ecmascript标准发布流程

The [TC39](https://github.com/tc39) categorizes proposals into the following stages:

- [Stage 0](https://babeljs.io/docs/en/babel-preset-stage-0) - Strawman: just an idea, possible Babel plugin.
- [Stage 1](https://babeljs.io/docs/en/babel-preset-stage-1) - Proposal: this is worth working on.
- [Stage 2](https://babeljs.io/docs/en/babel-preset-stage-2) - Draft: initial spec.
- [Stage 3](https://babeljs.io/docs/en/babel-preset-stage-3) - Candidate: complete spec and initial browser implementations.
- Stage 4 - Finished: will be added to the next yearly release.
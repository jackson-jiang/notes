#一个小栗子带你玩转前端开源

> 

[TOC]

## 写在前面的话

>:stars:
>
>使用开源项目能避免重造轮子提高编码效率
>
>研究开源项目，提高技能的广度和深度
>
>我热爱开源--主要因为不花钱 :joy:



## 搭建前端环境

> 项目开始前需要搭建前端的开发环境

[Nodejs](https://nodejs.org/en/)

[git](https://git-scm.com/)

```bash
node --version
npm --version
git --version
```



## 托管代码

> 在github，码云等公共代码托管网站上托管你的代码，方便用户随时访问

* 浏览器打开[github](http://github.com) 并登陆

* 创建项目`jks-opensource`

* 将项目克隆到本地，推荐使用[SSH](https://help.github.com/articles/connecting-to-github-with-ssh/)的方式链接github

  ```bash
  echo "# jksopensource" >> README.md
  git init
  git add README.md
  git commit -m "first commit"
  git remote add origin git@github.com:jackson-jiang/jksopensource.git
  git push -u origin master
  ```




## 使用包管理工具

> 使用包管理工具管理前端项目，在项目/库发布后用户可以很容易的通过包管理工具安装使用

### 配置npm

打开[npm](https://docs.npmjs.com/)配置文档，查看npm的配置项


```bash
#项目下，打开命令行
#配置npm（若果没配置过）
npm set init-author-name jackson.jiang
npm set init-author-email jackson.jiang@outlook.com
npm set init-license MIT

#通过npm安装的依赖包会有固定的版本号 eg: 1.0.0 vs ^1.0.0
# npm shrinkwrap
npm set save-exact true

# .npmrc // ~/.npmrc

#初始化项目
npm init
# 输入description
# entry point src/index.js
```



## 开发项目代码

#### 开发代码

```javascript
// 打开编辑器(vscode)
// code .
// ./teams.json
[
  "巴西",
  "法国",
]
// ./src/index.js
var teams = require('./teams.json');
var _ = require('lodash');
module.exports = {
  all: _.clone(teams),
  champion: function () {
    const i = Math.ceil(Math.random() * teams.length)
    return teams[i];
  }
}
```



#### 安装依赖

```bash
# CMD
npm install --save lodash
```



#### 测试项目代码

```bash
# 使用node REPL测试代码
# CMD
node
var champion = require('./src/index.js');
champion.all;
champion.champion();
```



## 发布代码

### 发布到github

```bash
#CMD
git status

# 配置git ignore
# /.gitignore
# node_modules

git add .
git push
```



### 发布到npm仓库

```bash
# CMD
npm login

# 登陆不成功可能是之前配置过 "~/.npmrc", 可删除后重试
#–-registry=https://registry.npm.org

npm publish

# npm官网查看是否发布成功
# npm.im/jks-opensource

# npm命令行查看是否成功
npm info 'jks-opensource'

# 测试
npm i 'jks-opensource'

# 参考"测试项目代码"章节
```



### 发布的版本

> 项目应该使用版本发布，这样就不会对使用现有版本的用户造成影响
>
> 使用semver语义版本定义版本号: [major.minor.patch] *例子*：1.0.0， 2.1.0-beta.0 测试版本.



```bash
#github 发版
git tag 1.0.0
git push --tags
# github上可以查看release和编写release note

#npm 发版
#npm的版本会使用package.json文件的version
npm publish [--tag <tag>]

# 发布/安装beta版本和正式版本
# 1. 添加中国队，发布beta版
# 2. 去掉中国队，发布正式版

# 安装测试
# npm i <pakage@tag> "tag:latest(默认)/beta(tag名字)"
```



## 优雅的开发

### 规范化git提交信息

```bash
#CMD
#安装依赖 - angular风格的提交信息格式，提交信息校验工具commitlint和git钩子工具husky
npm install -save-dev --save-exact husky commitizen cz-conventional-changelog  @commitlint/config-conventional @commitlint/cli

# package.json 配置
#scripts
"commit": "git-cz",
"cm": "git-cz", # 应该使用cm,如scirpt为'commit', npm会试图调用'precommit'的程序，这会出发precommit钩子

# config
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
    "pre-commit": "pretty-quick --staged --pattern \"src/**/*.*(js|vue)\""
  }
},
# github创建issue

# 更新代码
# ./src/teams.json
[
  "巴西",
  "法国",
  "德国",
]

#使用npm run commit代替git commit
git add .
npm run commit

# commit消息中fix/close #1 会自动关闭github上关联的issue

npm tag v1.1.0
npm push --tags
```



### 自动化发布版本

> 通过commitizen， semantic-release，travis-ci实现版本的自动发布

```bash
# CMD
# 安装依赖
npm install -g semantic-release-cli
semantic-release-cli setup
#=bjjtTJ3

# package.json
# scripts:
# 去掉test

# 查看发布状态
# https://travis-ci.org/
```



### 规范代码风格

#### 代码风格-ESLint

```bash
# CMD
npm i -D eslint

# 初始化eslint配置
./node_modules/.bin/eslint --init

# 使用airbnb规则: eslint-config-airbnb-base@latest
# .eslintrc

# package.json
# scripts:
"lint": "eslint src --ignore-pattern *.test.js"

# vscode 安装eslint插件可以实时查看并修复可以自动修复的问题（多数为格式方面）

# .eslintignore
*.test.js

# vscode用户配置:
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
        "language": "html",
        "autoFix": true
        },
        {
        "language": "vue",
        "autoFix": true
        }
    ],
    "eslint.autoFixOnSave": true,
    
```



#### 代码风格

非eslint项目基于prettier和prettier-quick格式化

```bash
# npm install --save-dev --save-exact prettier prettier-quick onchange
# package.json
"scripts": {
  "format-all": "prettier --write \"src/**/*.{vue,js}\"",
  "format": "onchange \"src/**/*.js\" \"src/**/*.vue\" -- prettier --write {{changed}}",
}
"husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
      "pre-commit": "pretty-quick --staged --pattern \"src/**/*.*(js|vue)\""
    }
  },

# /.prettierrc.js
module.exports = {
  singleQuote: true,
  trailingComma: 'es5',
  insertPragma: true,
  endOfLine: 'lf',
  htmlWhitespaceSensitivity: 'ignore',
};
```





##### 编辑器风格

```bash
# .editorconfig
root = true

[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
```





## 发布前测试你的代码

> 手工测试/测试框架 mocha/chai，**TDD**

### 单元测试

### 安装测试框架

```bash
# CMD
npm i -D mocha chai
# package.json
# scripts
"test": "mocha ./src/index.test.js",
"watch:test": "npm t -- -w",
```



### 编写测试代码

```javascript
// TDD测试驱动开发，先写添加 champion的参数（n）然后实现功能
var fifa = require('./index.js')
var expect =require('chai').expect;

describe('jks-opensource', function() {
  describe('all', function() {
    it('一共4只球队', function() {
      expect(fifa.all.length).to.equal(4);
    });
    it('必须有`巴西`队', function() {
      expect(fifa.all).to.include('巴西');
    });
  });
  describe('champion', function() {
    it('冠军球队在所有队伍中', function() {
      expect(fifa.all).to.include(fifa.champion());
    });
  });
});
```

```bash
# CMD
# 运行测试
# ==> npm run test
npm t
# ==> npm t -- -w
npm run watch:test
```



### 代码覆盖测试

```bash
# CMD
npm i -D nyc

# package.json
# scripts
"cover": "nyc --reporter=lcov npm t"

# package.json
# scripts
"check-coverage": "nyc check-coverage --statements 100 --branches 100 --functions 100 --lines 100"
```



### 提交前自动检验测试结果

> 为了避免忘记测试，提交代码前强制检查测试结果

```bash
# CMD
npm i -D ghooks

#配置ghooks
# package.json
# config
    "ghooks": {
      "pre-commit": "npm t && npm run check-coverage"
    }

# 测试
git commit -am 'test'
```



### ~~发送测试报告到第三方机构~~

```bash
# codecov.io
# CMD
npm i -D codecov.io

# package.json
# scripts
"cover": "nyc  --reporter=lcov npm t",
"report-coverage": "cat ./coverage/lcov.info | codecov",

# codecov.io无法登陆
```



## 添加ES6支持

> 使用ES6提高效率

### 为代码添加ES6支持

```bash
   # CMD
   npm i -D babel-cli babel-core babel-register babel-preset-env
   
   # package.json
   # scripts:
   "build:main": "babel --copy-files --out-dir dist --ignore *.test.js src",
  
  # 使用编译过的版本
    "main": "dist/index.js",
    
   #package.json
   "babel": {
    "presets": [
      "env"
    ]
  },
  
  # 在travis CI中配置运行build 
  #.travis.yml
  script:
    - npm run build
  
  # ----------------------------------------------
  #npm pack -- 打包前检查
  
  #不好用
  #配置要发布的文件
  #package.json
    "files": [
    "dist",
    "readme.md"
  ],
```

### 为测试框架添加ES6支持

```bash
  # package.json
  # scripts:   
  "test": "mocha src/index.test.js --compilers js:babel-register",
```



## 打包不同的版本

> 项目中代码使用commonjs编写，浏览器是不支持require语法，如果想直接在浏览器使用，需要build umd版本的软件包



```bash
# 安装webpack依赖
npm i -D webpack-cli

# webpack.config.js -- 支持ES6
import {join} from 'path'

const include = join(__dirname, 'src')

export default {
  entry: './src/index',
  output: {
    path: join(__dirname, 'dist'),
    libraryTarget: 'umd',
    library: 'open-source',
  },
  devtool: 'source-map',
  module: {
    loaders: [
       {test: /\.js$/, loader: 'babel', include},
       {test: /\.json$/, 'loader': 'json', include},
     ]
  }
}

npm i -D babel-loader json-loader npm-run-all
npm i -D rimraf
npm i -D babel-preset-env

# package.json
# scripts:
    "prebuild": "rimraf dist",
    "build": "npm-run-all --parallel build:*",
    "build:umd.min": "webpack --output-filename index.umd.min.js -p",
    "build:umd": "webpack --output-filename index.umd.js"

git push
#unpkg.com/:package@:version/:file
```



## README: 为项目添加标志

> readme文件通常为markdown格式，内容为项目的使用说明，添加badge可以清晰的看到项目的状态信息

[Shields.io](http://shields.io)



```markdown
# readme.md
[![travis build](https://img.shields.io/travis/jackson-jiang/jksopensource.svg)](https://travis-ci.org/jackson-jiang/jksopensource)
[![MIT License](https://img.shields.io/npm/l/jksopensource.svg?style=flat-square)](http://opensource.org/licenses/MIT)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg?style=flat-square)](https://github.com/semantic-release/semantic-release)
[![GitHub package version](https://img.shields.io/github/release/jackson-jiang/jksopensource.svg)](https://github.com/jackson-jiang/jksopensource)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
```



## 完整的开发一个新功能（TDD）

* github创建issue
* 编写test case
* TDD
* 实现代码
* 提交 close #issue
* 发布版本



## 参考

### 网站

- [github](https://github.com/)
- [nodejs](https://nodejs.org/en/)
- [npm](https://www.npmjs.com/)
- [eslint](https://eslint.org/)
- [editorconfig](https://editorconfig.org/)
- [commitizen](https://github.com/commitizen/cz-cli)
- [travis](https://travis-ci.org)
- [babel](http://babeljs.io/)

- [shields](http://shields.io)

- [webpack](http://webpack.github.io/)

- [codecov](https://codecov.io)

- [mocha](https://mochajs.org/)
- [chai](http://www.chaijs.com/)
- [markdown](http://www.markdown.cn/)
- [nyc](https://www.npmjs.com/package/nyc)



### 参考文章

- [egghead: how to write an open source javascript library](https://egghead.io/courses/how-to-write-an-open-source-javascript-library)

- [starwars-names](https://codecov.io)

- [语义化版本](https://semver.org/)

- [使用SSH的方式链接github](https://help.github.com/articles/connecting-to-github-with-ssh/)

- [阮一峰ES6](http://es6.ruanyifeng.com/)

- [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

  ​

### 工具

- 在线repl工具：https://repl.it
- 文本编辑器：[vscode](https://code.visualstudio.com/)
- 浏览器插件：[vimium](http://vimium.github.io/)
- 语义版本管理工具：[semantic-release](https://www.npmjs.com/package/semantic-release)
- git钩子： [ghooks](https://www.npmjs.com/package/ghooks)
- js工具库： [lodash](https://www.npmjs.com/package/ghooks)



## 代办

mocha watch json?
eslint --compilers要弃用了
webpack.config.js -- 支持es6？

在望海应用

Vimnium++??

Npm readme 乱码?
Webpack配置使用import编写
完善文档并发布
测试



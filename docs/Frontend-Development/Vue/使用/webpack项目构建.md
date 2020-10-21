vue-cli 是vue.js的脚手架，用于自动生成vue.js模板工程的。

 1、安装vue-cli

使用npm全局安装vue-cli（前提是已经安装了nodejs，否则连npm都用不了）

```
$ npm install -g vue-cli
```

2、创建自己的工作空间

```
$ vue init webpack vuetest
```

3、项目信息

命令输入后，会进入安装阶段，需要用户输入一些信息

```
Project name (vuetest)
```

项目名称，可以自己指定，也可直接回车，按照括号中默认名字（注意这里的名字不能有大写字母，如果有会报错Sorry, name can no longer contain capital letters）

```
Project description (A Vue.js project) //项目描述
```

项目描述，也可直接点击回车，使用默认名字

Author (........)    //作者

4、用户选择

```
Install vue-router? (Y/n)
```

是否安装vue-router，这是官方的路由。这里就输入“y”后回车即可。

```
Use ESLint to lint your code? (Y/n)
```

是否使用ESLint管理代码，ESLint是个代码风格管理工具，是用来统一代码风格的，并不会影响整体的运行，这也是为了多人协作。用则选Y

Standard (https://github.com/feross/standard)

标准，什么标准呢，去给提示的standardgithub地址看一下， 原来时js的标准风格

AirBNB (https://github.com/airbnb/JavaScript)

JavaScript最合理的方法，这个github地址说的是JavaScript最合理的方法

```
Setup unit tests with Karma + Mocha? (Y/n)
```

是否安装单元测试

```
Setup e2e tests with Nightwatch(Y/n)?
```

是否安装e2e测试

5、切换到项目目录

```
cd vuetest
```

6、安装依赖

```json
{
  "name": "alanmall-front",
  "version": "1.0.0",
  "description": "A Vue.js project",
  "author": "xiaomi <546493589@qq.com>",
  "private": true,
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "node build/build.js"
  },
  "dependencies": {
    "axios": "^0.19.0",
    "axios-mock-adapter": "^1.17.0",
    "element-ui": "^1.4.13",
    "geetest3": "^1.0.0",
    "gitment": "0.0.3",
    "mockjs": "^1.0.1-beta3",
    "vue": "^2.6.10",
    "vue-area-linkage": "^5.1.0",
    "vue-content-placeholders": "^0.2.1",
    "vue-cookie": "^1.1.4",
    "vue-cropper": "^0.1.7",
    "vue-infinite-scroll": "^2.0.2",
    "vue-lazyload": "^1.3.1",
    "vue-router": "^2.8.1",
    "vuex": "^2.5.0"
  },
  "devDependencies": {
    "autoprefixer": "^7.1.2",
    "babel-core": "^6.22.1",
    "babel-helper-vue-jsx-merge-props": "^2.0.3",
    "babel-loader": "^7.1.1",
    "babel-plugin-syntax-jsx": "^6.18.0",
    "babel-plugin-transform-runtime": "^6.22.0",
    "babel-plugin-transform-vue-jsx": "^3.5.0",
    "babel-preset-env": "^1.3.2",
    "babel-preset-stage-2": "^6.22.0",
    "chalk": "^2.0.1",
    "copy-webpack-plugin": "^4.0.1",
    "css-loader": "^0.28.0",
    "extract-text-webpack-plugin": "^3.0.0",
    "file-loader": "^1.1.4",
    "friendly-errors-webpack-plugin": "^1.6.1",
    "html-webpack-plugin": "^2.30.1",
    "node-notifier": "^5.1.2",
    "optimize-css-assets-webpack-plugin": "^3.2.0",
    "ora": "^1.2.0",
    "portfinder": "^1.0.13",
    "postcss-import": "^11.0.0",
    "postcss-loader": "^2.0.8",
    "postcss-url": "^7.2.1",
    "rimraf": "^2.6.0",
    "semver": "^5.3.0",
    "shelljs": "^0.7.6",
    "uglifyjs-webpack-plugin": "^1.1.1",
    "url-loader": "^0.5.8",
    "vue-loader": "^13.3.0",
    "vue-style-loader": "^3.0.1",
    "vue-template-compiler": "^2.5.2",
    "webpack": "^3.6.0",
    "webpack-bundle-analyzer": "^2.9.0",
    "webpack-dev-server": "^2.9.1",
    "webpack-merge": "^4.1.0"
  },
  "engines": {
    "node": ">= 6.0.0",
    "npm": ">= 3.0.0"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ]
}

```



```
npm install
```

7、运行

```
npm run dev
```

8、自动打开默认浏览器显示页面

![img](https://img.jbzj.com/file_images/article/201704/201742693321715.png?201732693410)

9、目录简要说明

```xml
├── build
 
// 项目构建(webpack)相关代码
 
│ ├── build.js
 
// 生产环境构建代码
 
│ ├── check-versions.js
 
// 检查node&npm等版本
 
│ ├── dev-client.js
 
// 热重载相关
 
│ ├── dev-server.js
 
// 构建本地服务器
 
│ ├── utils.js
 
// 构建配置公用工具
 
│ ├── vue-loader.conf.js
 
// vue加载器
 
│ ├── webpack.base.conf.js
 
// webpack基础配置
 
│ ├── webpack.dev.conf.js
 
// webpack开发环境配置
 
│ └── webpack.prod.conf.js
 
// webpack生产环境配置
 
├── config
 
// 项目开发环境配置
 
│ ├── dev.env.js
 
// 开发环境变量
 
│ ├── index.js
 
// 项目一些配置变量
 
│ └── prod.env.js
 
// 生产环境变量
 
├──node_modules
 
// 项目依赖的模块
 
├── src
 
// 源码目录
 
│ │ ├── assets
 
// 资源目录
 
│ │ └── logo.png
 
 
 
│ ├── components
 
// vue公共组件
 
│ │ └── Hello.vue
 
 
 
│ ├──router
 
// 前端路由
 
│ │ └── index.js
 
// 路由配置文件
 
│ ├── App.vue
 
// 页面入口文件，预留路由占位符给组件预留展示空间，导出根组件以便main.js使用
 
│ └── main.js
 
// 程序入口文件，引入公共文件，做一些全局的配置，实例化vue
 
└── static
 
// 静态文件，比如一些图片，json数据等
 
│ ├── .gitkeep
 
 
 
├── .babelrc
 
// ES6语法编译配置
 
├── .editorconfig
 
// 定义代码格式
 
├── .gitignore
 
// git上传需要忽略的文件格式
 
├── index.html
 
// 入口页面
 
├── package.json
 
// 项目基本信息
 
├── README.md
 
// 项目说明
```

以上目录选择了独立构建模式，安装vue-router，但没有安装ESLint、单元测试、e2e测试。
## Typescript and Webpack
### 叨逼叨
JS生态在近几年经历了不可阻挡的爆发式增长, 一大堆新技术名词的出现让前端程序员有点不胜其烦. 每当你觉得自己差不多跟上潮流的时候, 总有一波大牛出来带节奏, 因为他们发现了比所谓的最佳实践更好的解决方案. 这种不稳定性让大部分前端程序员陷入困惑, 甚至有点无所适从.
> The main problem is that things are moving so fast — best practices change every month as someone comes out with a new approach to a problem.

推荐一篇[2016年里做前端是怎样一种体验](https://segmentfault.com/a/1190000007083024), 看完我默默的说了一声**.

面对这个处境, 我个人认为只能是先挑一个系列的技术栈弄精通, 其他的后面再说, 毕竟谁都没有那么多精力来折腾. 什么叫一个系列的技术栈? 比如以Angular为核心, 衍生出来的生态圈子, Webpack(构建工具), Typescript(开发语言), RxJs(数据流), NgRx(状态管理), Ionic(移动端)等等. 讲真, 这一套大保健下来不是开玩笑的.

到位前为止, 大部分框架随着版本趋于稳定都提供了官方的命令行工具([angular-cli](https://github.com/angular/angular-cli), [ionic-cli](https://github.com/ionic-team/ionic-cli), [vue-cli](https://github.com/vuejs/vue-cli), [create-react-app](https://github.com/facebookincubator/create-react-app/)), 一个命令可以帮助我们快速生成项目结构, 完成很多工具的配置与兼容. 甚至涵盖到模块, 编译, 打包, 测试等等环节. 虽然命令行工具简化了我们的工作, 但是抱着折腾的精神, 我还是要尝试自己搭建[Typescript](http://www.typescriptlang.org/docs/tutorial.html)和[Webpack](https://webpack.js.org/)的开发环境.

### 挽起袖子
全局安装依赖包, webpack命令没有提供类似 -init这样的参数, 所以安装了一个[webpack-init](https://github.com/agirton/webpack-init)的包辅助生成webpack.config.js, 自己手动写文件的话不需要安装这个包.
```
npm install -g typescript tslint webpack webpack-init webpack-dev-server

```

### 开始干

#### 新建项目
```
// 创建目录tw
mkdir tw

// 进入目录tw
cd tw

// 新建文件index.ts inc.ts index.html
touch index.ts inc.ts index.html
```

#### 生成配置文件
```
// 生成tsconfig.json
tsc --init

// 生成tslint.json
tslint --init

// 生成package.json
npm init

// 生成webpack.config.js
webpack-init
```
执行上边的命令, 有需要用户反馈的按情况选择或者直接回车就好了, 具体的配置我们会再修改. 到这一步我们该有的文件就都齐活了. What? So Easy?
```
ls

//目录结构如下
.
├── inc.ts
├── index.ts
├── package.json
├── tsconfig.json
├── tslint.json
└── webpack.config.js
```

#### 编辑业务文件
index.ts
```
import { LOG } from "./inc";
LOG("i am a function from inc.ts");
```

inc.ts
```
export const LOG = console.log.bind(console);
```

index.html
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Typescript & Webpack</title>
  </head>
  <body></body>
</html>

```

#### 打包文件
1) 安装本地依赖包
```
npm install ts-loader html-webpack-plugin --save-dev
npm install typescript webpack --save-dev
```
- [为什么NPM全局安装过的包还要在本地安装一次](http://www.cnblogs.com/PeunZhang/p/5629329.html)
- [怎么把打好的包动态添加到HTML文件中](https://webpack.js.org/plugins/html-webpack-plugin/)
- [Webpack怎么处理Typescript文件](https://github.com/TypeStrong/ts-loader)

2) 修改webpack.config.js
```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './index.ts',
  output: {
    //The output directory as an absolute path.
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  resolve: {
      // Add '.ts' and '.tsx' as a resolvable extension.
      extensions: ['.ts', '.tsx', '.js']
  },
  module: {
    loaders: [
      { test: /\.tsx?$/, loader: 'ts-loader', exclude: /(node_modules)/, }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
        template: path.resolve(path.resolve(__dirname), 'index.html'),
        inject: 'body'
    }),
  ]
}
```
上面的配置指定了输入, 输出, 需要处理的后缀类型, loader, plugin
- [上面配置看不懂的点我](https://webpack.js.org/configuration/)

3) 编译并执行
```
webpack
cd dist && open index.html
```
运行webpack, 在当前目录下生成一个dist文件夹(在上一步output.path中配置), 并将我们的输出文件写在这个目录下.
```
.
├── dist
│   ├── bundle.js
│   └── index.html
├── inc.ts
├── index.html
├── index.ts
├── node_modules
├── package.json
├── tsconfig.json
├── tslint.json
└── webpack.config.js
```
在浏览器打开dist/index.html, 观察console的输出
```
i am a function from inc.ts
```

#### 调试
每次修改代码都得重新编译, 实在太麻烦, 下面提供两种解决方案.
1) 启用webpack的watch模式, 监听文件的变化, 并自动进行编译.
```
webpack -w
```
2) webpack-dev-server
在webpack.config.js中plugins后继续添加devServer的配置如下, 和第一种方式相比, webpack-dev-server要更加强大, 提供了一个web server并且可以live-reload
```
devServer: {
    contentBase: './dist',
},
```
上述的配置告诉webpack-dev-server在./dist目录启动一个服务localhost:8080, 地址和端口当然都是可配置的.
试一下在命令行运行
```
webpack-dev-server --open
```
浏览器自动打开并且访问localhost:8080, 然后修改你的ts文件, 在浏览器的console会看到以下信息, 然后浏览器就自动刷新了. 怎么样, 超赞的.
```
[WDS] App updated. Recompiling...
14:42:11.346 bundle.js:3710 [WDS] App updated. Recompiling...
14:42:11.720 bundle.js:3710 [WDS] App updated. Reloading...
```
- [查看所有的devServer配置](https://webpack.js.org/configuration/dev-server/)

#### 代码检查
前端很常见各种linter, 帮助我们检查代码的一些风格, 潜在问题或错误, Typescript也不例外, 还记得在之前我们全局安装了tslint吗? 现在我们需要在本地安装一个tslint-loader
```
npm install --save-dev tslint-loader
```
修改webpack.config.js, 在module.rules中添加一条, 注意看`enfore`属性设置为了pre, 相当于1.x版本的preLoaders
```
{ test: /\.tsx?$/, loader: 'tslint-loader', enforce: 'pre', exclude: /(node_modules)/, },
```
修改完后, 再次启动服务. 看到命令行有一些warning,
```
WARNING in ./inc.ts
[2, 62]: block is empty
[1, 5]: Identifier 'DEBUG' is never reassigned; use 'const' instead of 'let'.
```
- [tslint有哪些规则可以配置?](https://palantir.github.io/tslint/rules/)

#### 单元测试
额, 又是一大块内容, 之前写过一篇[搭建基于Karma和Jasmine的前端单元测试](http://www.jianshu.com/p/a7ffb564734e),
1) 安装相关依赖
```
npm install karma-cli -g
npm install karma jasmine-core karma-jasmine karma-chrome-launcher karma-sourcemap-loader karma-webpack karma-coverage-istanbul-reporter istanbul-instrumenter-loader @types/jasmine --save-dev
```
我擦嘞, 这么多东西? 是的, 我也不想这样.
和之前相比, 多了[karma-coverage-istanbul-reporter](https://github.com/mattlewis92/karma-coverage-istanbul-reporter) [istanbul-instrumenter-loader](https://github.com/webpack-contrib/istanbul-instrumenter-loader) 用来生成测试报告. [@types/jasmine](https://github.com/DefinitelyTyped/DefinitelyTyped) jasmine的类型定义, 修改tsconfig.json
```
"compilerOptions": {
        "types": [
            "jasmine"
        ]
    },
```

2) 配置karma
```
karma init
```
上面命令帮我们在项目根目录生成karma.conf.js, 修改其内容如下
```
module.exports = function(config) {
    config.set({
        basePath: '',
        frameworks: ['jasmine'],
        files: [ 'spec.bundle.ts' ],
        plugins: [ 'karma-webpack', 'karma-sourcemap-loader', 'karma-chrome-launcher', 'karma-coverage-istanbul-reporter', 'karma-jasmine' ],
        exclude: [],
        preprocessors: {
            'spec.bundle.ts': ['webpack', 'sourcemap']
        },
        mime: { 'text/x-typescript': ['ts', 'tsx'] },
        reporters: ['progress', 'coverage-istanbul'],
        port: 9876,
        colors: true,
        logLevel: config.LOG_INFO,
        autoWatch: true,
        browsers: ['Chrome'],
        singleRun: false,
        concurrency: Infinity,
        coverageIstanbulReporter: {
            reports: ['html', 'text-summary'],
            dir: './coverage',
            fixWebpackSourcePaths: true,
            skipFilesWithNoCoverage: true,
            'report-config': {
                html: {
                    subdir: 'html'
                }
            }
        },
        webpack: {
            resolve: {
                extensions: ['.ts', '.tsx', '.js']
            },
            devtool: 'inline-source-map',
            module: {
                rules: [
                    { test: /\.tsx?$/, loader: 'ts-loader', exclude: /(node_modules)/, },
                    { test: /\.tsx?$/, loader: 'istanbul-instrumenter-loader', enforce: 'post', exclude: [/(node_modules)/, /\.spec\.ts$/] },
                ]
            }
        }
    })
}
```
- [ng test ends with "Executed 0 of 0 ERROR"](https://github.com/angular/angular-cli/issues/2838#issuecomment-265782400)

3) 编写测试文件
创建一个spec.bundle.ts作为我们的测试入口文件, 并把它添加到files配置中去
spec.bundle.ts
```
declare function require(moduleName: string): any;
let testsContext = (require as any).context("./test", true, /\.spec\.ts$/);
testsContext.keys().forEach(testsContext);
```
- [require context](https://webpack.github.io/docs/context.html)
创建test文件夹
```
mkdir test
cd test
touch inc.spec.ts
```
inc.spec.ts
```
import { LOG } from "../inc";

describe('inc.ts', () => {
  it('LOG should be defined', () => {
    expect(LOG).toBeDefined();
    expect(toString.apply(LOG)).toEqual('[object Function]');
  });
});
```
这一步完成后, 我们的目录结构为
```
.
├── dist
│   ├── bundle.js
│   └── index.html
├── inc.ts
├── index.html
├── index.ts
├── karma.conf.js
├── package.json
├── spec.bundle.ts
├── test
│   └── inc.spec.ts
├── tsconfig.json
├── tslint.json
└── webpack.config.js
```
4) 运行测试
```
karma start
```
Chrome弹出新窗口, 命令行控制台输出如下.
```
Chrome 60.0.3112 (Mac OS X 10.12.5): Executed 1 of 1 SUCCESS (0.046 secs / 0.037 secs)

=============================== Coverage summary ===============================
Statements   : 100% ( 4/4 )
Branches     : 100% ( 0/0 )
Functions    : 100% ( 0/0 )
Lines        : 100% ( 4/4 )
================================================================================
```
根目录下生成了coverage目录, 我们的测试报告就在这里. 直接进入目录打开index.html即可看到测试的具体覆盖率情况. 超赞的
```
.
├── coverage
│   └── html
│       ├── base.css
│       ├── inc.ts.html
│       ├── index.html
│       ├── prettify.css
│       ├── prettify.js
│       ├── sort-arrow-sprite.png
│       ├── sorter.js
│       └── spec.bundle.ts.html

```

![image.png](http://upload-images.jianshu.io/upload_images/2261833-2b52c7572715f00a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 简介
[Jasmine](http://jasmine.github.io/2.0/introduction.html)是一个*behavior-driven development* ( 行为驱动开发 ) 测试框架, 不依赖于任何其他JavaScript框架, 不依赖DOM, 并且有很简洁的语法让你能够很轻松的编写单元测试. 本文并不会介绍Jasmine的语法, 重点是成功的搭建一个测试环境.

[Karma](https://karma-runner.github.io/0.13/index.html)是Testacular的新名字, 在2012年google开源了Testacular，2013年Testacular改名为Karma.
Karma是一个基于Node.js的JavaScript测试执行过程管理工具( Test Runner ). 该工具可用于测试所有主流Web浏览器, 也可集成到CI ( Continuous integration ) 工具, 也可和其他代码编辑器一起使用. 这个测试工具的一个强大特性就是, 它可以监控(Watch)文件的变化, 然后自行执行.

接下来的内容需要读者对 [Node.js](https://nodejs.org/en/) 和 [NPM](https://www.npmjs.com/) 有一个基本了解, 否则你可能搞不清楚我们究竟在干什么.
#### 安装

为了能成功运行Jasmine单元测试, 请安装以下NPM Package. 它们并不都是必须的, 但为了完整性, 我会把所有要使用的包列在这里. 

安装[karma-cli]
> Commandline Interface. You will need to do this if you want to run Karma on Windows from the command line. 

``
npm install karma-cli -g
``

安装[Karma](https://karma-runner.github.io/latest/intro/installation.html)
> Spectacular Test Runner for JavaScript.

``
npm install karma --save-dev
``

安装[karma-jasmine](https://www.npmjs.com/package/karma-jasmine)
> A Karma plugin - adapter for Jasmine testing framework.

``
npm install karma-jasmine --save-dev
``

安装[karma-chrome-launcher](https://www.npmjs.com/package/karma-chrome-launcher)
> Launcher for Google Chrome and Google Chrome Canary.

``
npm install karma-chrome-launcher --save-dev
``

安装[jasmine-core](https://www.npmjs.com/package/jasmine-core)
> Official packaging of Jasmine's core files for use by Node.js projects.Note: Since karma-jasmine 0.3.0 the jasmine library is no longer bundled with karma-jasmine and you have to install it on your own. 

``
npm install jasmine-core --save-dev
``

安装[karma-coverage](https://www.npmjs.com/package/karma-coverage)
> A Karma plugin. Generate code coverage.

``
npm install karma-coverage
``
#### 配置
karma通过[karma.config.js](https://karma-runner.github.io/0.13/config/configuration-file.html)进行配置, 这个文件具体的配置项可以点击链接进入官网, 有很详细的介绍. 但我们通常不需要把所有的配置都弄清楚, 这并不代表它们不重要, 只是熟悉一些常用的配置就足以满足我们的需要. 

我们可以通过`karma init`自动生成一个配置文件.整个过程完全是傻瓜式的操作.
```
C:\Users\Administrator>karma init

Which testing framework do you want to use ?
Press tab to list possible options. Enter to move to the next question.
> jasmine

Do you want to use Require.js ?
This will add Require.js plugin.
Press tab to list possible options. Enter to move to the next question.
> no

Do you want to capture any browsers automatically ?
Press tab to list possible options. Enter empty string to move to the next question.
> Chrome
>

What is the location of your source and test files ?
You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
Enter empty string to move to the next question.
>

Should any of the files included by the previous patterns be excluded ?
You can use glob patterns, eg. "**/*.swp".
Enter empty string to move to the next question.
>

Do you want Karma to watch all the files and run the tests on change ?
Press tab to list possible options.
> yes


Config file generated at "C:\Users\Administrator\karma.conf.js".
```


#### 运行karma
如果上面的步骤你执行的很顺利, 接下来试着在命令行运行`karma start`
```
C:\Users\Administrator>karma start
20 06 2016 08:59:21.910:WARN [karma]: No captured browser, open http://localhost:9876/
20 06 2016 08:59:21.925:INFO [karma]: Karma v0.13.22 server started at http://localhost:9876/
20 06 2016 08:59:21.941:INFO [launcher]: Starting browser Chrome
20 06 2016 08:59:23.282:INFO [Chrome 51.0.2704 (Windows 10 0.0.0)]: Connected on socket /#pJl8pFn3FBKacmLnAAAA with id 58410301
Chrome 51.0.2704 (Windows 10 0.0.0): Executed 0 of 0 ERROR (0.004 secs / 0 secs)
```
如果在命令行看到如下提示, 并且成功的打开了Chrome浏览器, 那么恭喜你, 你成功的运行了karma. 但是很明显你可以看到我们并没有写任何一个单元测试. `Executed 0 of 0 ERROR (0.004 secs / 0 secs)`. 所以接下来我们要做的就是开始写单元测试.
#### 写一个简单的单元测试
假设我们有一个js/main.js需要测试, 它包含了两个简单的函数. 
```
function reverse(str) {
    return str.split('').reverse().join('');
}

function isInteger(num) {
    if (typeof num !== "number") return false;
    var pattern = /^[1-9]\d*$/g;
    return pattern.test(num);
}

```
我们创建一个test/mainSpec.js文件*注意文件的路径需要和配置文件对应*
```
describe("main.js", function() {

    it("reverse", function() {
        // Assert (verify the result)
        expect(reverse('abcd')).toEqual('dcba');
        expect(reverse('asdf')).toEqual('fdsa');
    });

    it("isInteger", function() {
        expect(true).toEqual(isInteger(20));
        expect(false).toEqual(isInteger("20"));
        expect(false).toEqual(isInteger(0));
    })


});
```
至于如何来写一个jasmine单元测试, 不是这篇文章的内容. 大家可以移步[jasmine](http://jasmine.github.io/2.0/introduction.html)官网学习.
它的四要素就是`describe, it, expect, toBe`
再运行`karma start` 试试
```
C:\Users\Administrator>karma start
20 06 2016 09:10:18.766:WARN [karma]: No captured browser, open http://localhost:9876/
20 06 2016 09:10:18.782:WARN [karma]: Port 9876 in use
20 06 2016 09:10:18.782:INFO [karma]: Karma v0.13.22 server started at http://localhost:9877/
20 06 2016 09:10:18.782:INFO [launcher]: Starting browser Chrome
20 06 2016 09:10:20.109:INFO [Chrome 51.0.2704 (Windows 10 0.0.0)]: Connected on socket /#OoyWShGkmQDvgjqvAAAA with id 84751591
Chrome 51.0.2704 (Windows 10 0.0.0): Executed 2 of 2 SUCCESS (0.013 secs / 0.004 secs)
```
已经成功的执行了测试, karma会自动watch我们的js文件, 如果文件发生变化, 它会立刻做出响应.
#### 生成测试报告
还记得我们之前安装的`karma-coverage`吗? 它是karma的一个插件, 提供了测试代码覆盖率的支持.
修改我们的karma.conf.js.

```
// preprocess matching files before serving them to the browser
// available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
// source files, that you wanna generate coverage for
// do not include tests or libraries
// (these files will be instrumented by Istanbul)
preprocessors: {
	'js/*.js': ['coverage']
},

// optionally, configure the reporter
coverageReporter: {
	type: 'html',
	dir:  'coverage/'
},

// test results reporter to use
// possible values: 'dots', 'progress'
// available reporters: https://npmjs.org/browse/keyword/karma-reporter
// coverage reporter generates the coverage
reporters: ['progress','coverage'],
```
涉及三个配置信息, 两个是必须的, 一个是可选的.
1. preprocessors 预处理器配置
这里配置哪些文件需要统计测试覆盖率, 例如, 如果你的所有代码文件都在 js文件夹中, 你就需要如下配置. 注意不要包含你所依赖的库，测试文件等等
2.  reports 报告配置
在配置文件中包含下面的信息来激活覆盖率报告器.
3. coverageReporter 报告选项, 默认格式如下
type 是一个字符串值，其他取值可以进入官网查阅. 通常我们就用默认就好了.
dir 则用来配置报告的输出目录。如果是一个相对路径的话，将相对与 basePath 参数。
如果类型是 text 或者 text-summary，你可以配置 file 参数来指定保存的文件名。
```
coverageReporter = {
  type : 'text',
  dir : 'coverage/',
  file : 'coverage.txt'
}
```
如果没有文件名，就会输出到控制台。

#### 集成到webstorm
到了上一步, 我们的工作就完成了, 你已经可以开始在项目中使用karma和jasmine, 但是为了方便我把它们集成到了webstorm. 这并不复杂. 
进入run菜单下面的edit Configrations. 点击+, 选择karma. 配置好相应的路径.
然后再karma.conf.js上右键选择 `run karma.conf.js`
参考文章
- [https://blog.jetbrains.com/webstorm/2013/10/running-javascript-tests-with-karma-in-webstorm-7/](https://blog.jetbrains.com/webstorm/2013/10/running-javascript-tests-with-karma-in-webstorm-7/)
- [http://confluence.jetbrains.com/display/WI/Running+JavaScript+tests+with+Karma#RunningJavaScripttestswithKarma-KarmaintegrationinWebStorm](http://confluence.jetbrains.com/display/WI/Running+JavaScript+tests+with+Karma#RunningJavaScripttestswithKarma-KarmaintegrationinWebStorm)

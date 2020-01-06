# 什么是process.env.NODE_ENV

在node中, 有全局变量process表示的是当前的node进程. process.env包含着关于系统环境的信息. 但是process.env中并不存在NODE_ENV这个东西. NODE_ENV是用户一个自定义的变量, 在webpack中它的用途是判断生产环境或开发环境的依据的.

# 如何设置环境变量

## windows

    // node中常用的到的环境变量是NODE_ENV, 首先查看是否存在 
    set NODE_ENV 

    // 如果不存在则添加环境变量 
    set NODE_ENV=production 

    // 环境变量追加值 set 变量名=%变量名%;变量内容 
    set path=%path%;C:\web;C:\Tools 

    // 某些时候需要删除环境变量 
    set NODE_ENV=

## linux

    // node中常用的到的环境变量是NODE_ENV, 首先查看是否存在
    echo $NODE_ENV

    // 如果不存在则添加环境变量
    export NODE_ENV=production

    // 环境变量追加值
    export path=$path:/home/download:/usr/local/

    // 某些时候需要删除环境变量
    unset NODE_ENV

    // 某些时候需要显示所有的环境变量
    env

# 理解DefinePlugin

[https://webpack.js.org/plugins/define-plugin/](https://webpack.js.org/plugins/define-plugin/)

DefinePlugin允许我们创建全局变量, 可以在编译时进行设置. 这就是DefinePlugin的基本功能.

    module.exports = {
        mode: "production",
        // using mode: "production" attaches the following configuration:
        plugins: [
            new webpack.DefinePlugin({
                "process.env.NODE_ENV": JSON.stringify("production")
            }),
        ]
    }

通常在设置了`mode`属性以后, 会自动帮我们设置`process.env.NODE_ENV`, 比如上面的例子.

注意因为直接文本替换, 所给的属性值必须包括引号, 要这么做JSON.stringify("production")或'"production"' **(注意：有单引号包裹)**.

在编译时解析意味着你在代码中使用的process.env.NODE_ENV,将被替换成production这个值.

    console.log(process.env.NODE_ENV);
    if(process.env.NODE_ENV === 'production') {
        console.log('this is production!')
    }

    -----------------------------------------------------

    console.log("production");
    if(true) {
        console.log("this is production!")
    }

# 如何设置环境变量

还是以上面的例子来说, 如果需要区分当前打包环境是dev还是prod, 一种方式是针对两种环境分别编写webpack的配置文件, 比如`webpack.dev.js` `webpack.prod.js`, 然后分别设置

    module.exports = {
        mode: "development / production",
        // using mode: "production" attaches the following configuration:
        plugins: [
            new webpack.DefinePlugin({
                "process.env.NODE_ENV": JSON.stringify("development / production")
            }),
        ]
    }

但更方便的做法是从环境变量获取process.env.NODE_ENV的值.

    module.exports = {
        plugins: [
            new webpack.DefinePlugin({
                "process.env.NODE_ENV": JSON.stringify(process.env.NODE_ENV)
            }),
        ]
    }

文章开头已经说了NODE_ENV实际上是一个用户自定义的环境变量, 最常见的设置改变量的方式是采用 **[corss-env](https://github.com/kentcdodds/cross-env)**, 它提供了一种跨平台的方式来设置环境变量.

    "scripts": {
      "dev": "cross-env NODE_ENV=development webpack-dev-server",
      "build": "cross-env NODE_ENV=production webpack",
    }

# 如何更好的管理环境变量

如果我们需要设置多个环境变量的时候, 显然cross-env还是有些麻烦, 最重要的是这些变量的值会被提交的代码仓库上去, 如果存在API_KEY, PASSWORD这类的敏感信息, 会造成一些安全性的风险. 这里推荐两个非常好用的工具

*   [dotenv-webpack](https://github.com/mrsteele/dotenv-webpack)
*   [env2](https://github.com/dwyl/learn-environment-variables)

这两个工具可以从外部的`.env`配置文件中加载数据并且设置到`process.env`对象上, 而我们可以把.env添加到.gitignore中避免泄露敏感信息.

    // ./.env.production
    PASSWORD=TEST
    API_KEY=TEST

    // webpack.config.js
    module.exports = {
      plugins: [
        new Dotenv({
          path: './.env.production',
        }),
      ],
    };

.env文件在webpack_dev_server启动之后是只读的, 如果想让修改生效则需重启Webpack_dev_server

另外全局变量如果引起eslint的问题, 可以在`.eslintrc`中添加 `{ "globals": { "GLOBAL_VIRABLE": true }, }`

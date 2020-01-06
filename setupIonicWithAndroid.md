做完两个Ionic项目后, 真的是踩坑无数. 开发Android, 光配置环境就弄得你不要不要的. 内心一个崩溃接一个崩溃, 各种墙、 下载不了、 速度慢. 

### NodeJs 和 NPM环境
[Node](https://nodejs.org/en/)官网下载直接安装即可. 
建议使用[国内镜像](http://npm.taobao.org/)

### Ionic Cli 和CORDOVA
```
npm install -g ionic cordova
```

### Android Platform
这一步虽然在[Crodova Doc](https://cordova.apache.org/docs/en/latest/guide/platforms/android/)上有说明, 但是在国内, 老铁门都懂的. 各种墙.

#### Java环境配置.
- 下载JDK安装
- 环境变量新建JAVA_HOME, 所填的路径为JDK所在路径, 例如 「D:\Program Files\Java\jdk1.8.0_91」
- 环境变量新建或编辑CLASSPATH, 追加.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
- 环境变量编辑path, 追加%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

#### 安装Android SDK
- 官网sdk下载不了，进入[开源镜像站](http://mirrors.neusoft.edu.cn/android/repository/)进行下载, 选择tools_r***-windows.zip版本, 
- 解压文件后找到tools文件夹打开android.bat程序，进入主界面，菜单栏依次选择「Tools」、「Options...」，弹出『Android SDK Manager - Settings』窗口
- 在『Android SDK Manager - Settings』窗口中，在「HTTP Proxy Server」和「HTTP Proxy Port」输入框内填入mirrors.neusoft.edu.cn和80，并且选中「Force https://... sources to be fetched using http://...」复选框。设置完成后单击「Close」按钮关闭『Android SDK Manager - Settings』窗口返回到主界面；
- 依次选择「Packages」、「Reload」。(由于某些网络接入商进行了劫持，会弹出用户认证界面无法使用，和本镜像服务器配置无关。)
- 在packages列表里选择合适的SDK Platform安装
- 环境变量新建ANDROID_HOME, 所填的路径就是解压tools_r***-windows文件夹所在的路径, 例如 「D:\Program Files\tools_r25.2.5-windows」
- 环境变量编辑path, 追加%ANDROID_HOME%\tools;%ANDROID_HOME%\tools\bin;%ANDROID_HOME%\platform-tools;%ANDROID_HOME%;

### Gradle配置
- 在[gradle.org](https://gradle.org/install)提供了几种安装方法, 我是直接选择「Install manually > Binary-only」, 如果有版本问题, 在[releases](https://gradle.org/releases)页面选择3.3版本
- 解压到本地目录(建议新建一个Gradle目录, 然后解压, 以便存放不同的版本, 另外压缩包不要删, 后边还会用) 
- 环境变量编辑path, 追加上一步解压的目录, 例如「D:\Program Files\Gradle\gradle-3.3\bin;」
- 在windows用户目录下找到.gradle, 编辑init.gradle, [原因以及详情](http://www.jianshu.com/p/4a90743e771a)
```
allprojects {
    repositories {
         maven {
             name "aliyunRepo"
             url "http://maven.aliyun.com/nexus/content/groups/public"
         }
    }
}
```

### 第一个项目
- 新开命令行, 输入ionic start myIonic tabs --skip-deps
- 运行```cnpm install```
- 运行```ionic cordova platform add android```
- 打开myIonic\platforms\android\gradle, 复制「gradle-3.3-bin.zip」过来, [原因以及详情](http://blog.csdn.net/bangrenzhuce/article/details/51990525)
- 编辑项目package.json, 在scripts下添加一个
```
"gradlerun": "set CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL=../gradle-3.3-bin.zip&&ionic cordova run android",
"gradlerundebug": "set CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL=../gradle-3.3-bin.zip&&ionic cordova run android -lcs"
```
- 连接手机, 手机开启开发者模式
- 运行
```
cnpm run gradle
```
如果一切顺利的话, 应该可以在手机上跑起应用了.

### 可能遇到的问题
- [Gradle提示，You have not accepted the license agreements of the following SDK components: Android SDK Build-Tools 24.0.2](http://majing.io/questions/804)

### 调试
```
cnpm run gradlerundebug
```
确保手机和电脑在同一个局域网, 防火墙设置好.
- adb logcat | find "xx", xx是包名
- [http://www.genuitec.com/products/gapdebug/](http://www.genuitec.com/products/gapdebug/)
- [adb logcat chromium:D SystemWebViewClient:D *:S](https://stackoverflow.com/questions/23385839/cordova-console-log-not-working)
- [https://developer.android.com/studio/command-line/logcat.html](https://developer.android.com/studio/command-line/logcat.html)

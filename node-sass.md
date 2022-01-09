# 参考

[node-sass 安装失败的各种坑](https://www.jianshu.com/p/92afe92db99f)

# node-sass

​	Node-sass是一个库，它将Node.js绑定到LibSass（流行样式表预处理器Sass的C版本）。它允许用户以令人难以置信的速度将.scss文件本地编译为css，并通过连接中间件自动编译。

# Sass

​	Sass是一种是采用**Ruby** 语言编写的一款 CSS 预处理语言，可以解释或编译成层叠样式表（CSS）。

# 安装node-sass

- 直接运行 npm i -g node-sass 可能会失败。

- 主要是windows平台缺少编译环境，
   1、先运行： npm install -g node-gyp
   2、然后运行：运行 npm install --global --production windows-build-tools 可以自动安装跨平台的编译器：gym注：第二句执行下载好msi文件卡着不动不安装 ， 手动去对应的目录底下安装一下。

- 删除之前安装失败的包

  ```
  npm uninstall node-sass
  ```

- 重新安装

  ```
  npm install node-sass
  ```

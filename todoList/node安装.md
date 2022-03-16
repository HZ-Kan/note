2022-03-16

# 下载

[官网](https://nodejs.org/zh-cn/)

# 安装

直接下一步

# 配置

配置自定义的全局模块安装目录，在node.js安装目录下新建两个文件夹 node_global和node_cache，然后在cmd命令下执行如下两个命令：

```
npm config set prefix "D:\Program Files\nodejs\node_global"

npm config set cache "D:\Program Files\nodejs\node_cache"
```

执行成功。然后在环境变量 -> 系统变量中新建一个变量名为 “NODE_PATH”， 值为“D:\Program Files\nodejs\node_modules”

最后编辑用户变量里的Path，将相应npm的路径改为：D:\Program Files\nodejs\node_global

# 参考

[node.js 安装详细步骤教程](https://blog.csdn.net/antma/article/details/86104068)

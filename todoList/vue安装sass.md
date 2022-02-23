安装方法

```powershell
npm install node-sass -D
npm install sass-loader -D
```

报错

```powershell
# 没有安装sass-loader
**Can't resolve 'sass-loader'**
# sass-loader版本过高 默认安装版本是11.0.1
**TypeError: this.getOptions is not a function**   
```

报错解决方案

```powershell
npm install sass-loader@7.3.1    卸载你之前安装的node-loader(mac卸载报红命令行前加sudo即可)
npm install sass-loader@10.1.1  这个版本也可
npm install node-sass@4.14.1    (降低sass-loader版本还是报错，有可能node-sass版本是5+)
```

改完啦别忘了重新运行一下！！！
温馨提示：如果你的node-sass版本太低不兼容node版本10,node-sass4.9.0支持node版本10,node-sass4.3.5最高支持node8



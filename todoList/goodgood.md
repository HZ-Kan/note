# 根据日期获取本周周一

```
new Date(date.getFullYear(), date.getMonth(), date.getDate() + 1 - (date.getDay() || 7))
```

# [后端一次给你10万条数据，如何优雅展示，到底考察我什么?](https://juejin.cn/post/7031923575044964389)

# 关于[margin](https://so.csdn.net/so/search?q=margin&spm=1001.2101.3001.7020)-right失效的问题：

**example**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .container{
            width: 500px;
            height: 100px;
            margin: 0 auto;
            background-color: black;
        }
        .sss{
            width: 400px;
            height: 100px;
            background-color: violet;
            margin-right: -100px;            
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="sss"></div>
    </div>
</body>
</html>
```

- **默认状态下的块级元素右边距是无效的** 
  - 设置float（除了none以外的值）、display (inline-block，inline-flex，inline-grid，inline-table，inline-box，table)、position（absolute，fixed）之后会生效。


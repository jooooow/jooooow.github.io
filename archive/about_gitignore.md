---
title: "关于gitignore的用法"
---

# 简介

在使用git时有时我们希望忽略一些文件不追踪/不上传到远程库，此时可以使用.gitignore。

在.git同级目录下创建.gitignore文件，里面填入想要忽略的文件/目录。

在执行git add .的时候便会根据.gitignore文件中的内容将这些文件不放入cache便不会被提交。

# 语法

+ 无斜杠：文件
+ 只在开头有斜杠：用以禁止递归匹配
+ 只在结尾有斜杠：用以匹配目录
+ 感叹号：标识取反(即不需要忽略)
+ #：注释
+ 可用glob

# 注意点
```开头无斜杠则默认递归匹配```

# 例子
假设存在以下目录结构：  

```bash
├── .git
├── .gitignore
├── README.md
├── img
│   └── 1.png
├── server.py
├── static
│   ├── css
│   │   └── index.css
│   ├── img
│   │   ├── 1.png
│   │   └── 2.png
│   └── js
│       └── index.js
└── templates
    └── index.html
```

1. 忽略所有img目录  

```img/```  

2. 忽略server.py  

```server.py```  

3. 忽略所有img下的1.png文件    

```img/1.png```  

4. 只忽略根目录下的img目录    

```/img```  

5. 忽略/static/img下的所有文件除了1.png  

```/static/img/*<br>!/static/img/1.png```  

6. 一度被add后的对象再修改.gitignore后不会生效，需要删除cache重新add  

```
git add .
git commit -m 'bug fixed'
git push
```
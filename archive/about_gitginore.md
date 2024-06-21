<h4>简介</h4>
<p>在使用git时有时我们希望忽略一些文件不追踪/不上传到远程库，此时可以使用.gitignore。<br>
在.git同级目录下创建.gitignore文件，里面填入想要忽略的文件/目录。<br>
在执行git add .的时候便会根据.gitignore文件中的内容将这些文件不放入cache便不会被提交。</p>
<h4>语法</h4>
<p>无斜杠：文件<br>
只在开头有斜杠：用以禁止递归匹配<br>
只在结尾有斜杠：用以匹配目录<br>
感叹号：标识取反(即不需要忽略)<br>
#：注释<br>
可用glob</p>
<h4>注意点</h4>
<p>开头无斜杠则默认递归匹配</p>
<h4>例子</h4>
假设存在以下目录结构：
<pre><code class="Bash">.
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
</code></pre>
<h5>1)忽略所有img目录：</h5>
<p>img/</p>
<h5>2)忽略server.py：</h5>
<p>server.py</p>
<h5>3)忽略所有img下的1.png文件：</h5>
<p>img/1.png</p>
<h5>4)只忽略根目录下的img目录：</h5>
<p>/img</p>
<h5>5)忽略/static/img下的所有文件除了1.png：</h5>
<p>/static/img/*<br>!/static/img/1.png</p>
<h5>6)一度被add后的对象再修改.gitignore后不会生效，需要删除cache重新add：</h5>
<p><pre><code class='Bash'>git rm -r --cached .
git add .
git commit -m 'bug fixed'
git push</code></p></p>
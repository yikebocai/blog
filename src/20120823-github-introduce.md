GitHub入门
> *原文写于 2012-8-23*

<a href="http://git-scm.com" target="_blank">Git</a>是目前最流行的版本管理工具，而<a href="https://github.com" target="_blank">GitHub</a>是基于Git协议的一个代码托管站点，很多著名的开源程序都在上面。其实GitHub不仅仅是个代码托管站点，也可以做为一个在线文档写作平台，或者<a href="http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html" target="_blank">免费搭建自己的Blog站点</a>。 

虽然之前也写过开源软件ictclas4j，但那已是往事很久都没有去维护了，我现在使用的GitHub的主要目的是为了托管自己写的一些小程序，之前也写过很多都弄丢了，而GitHub是个很好的代码托管站点，也许有机会也会再写一些开源软件。 

**1.注册帐号** 
到<a href="https://github.com/plans" target="_blank">GitHub主页</a>上注册一个新的免费帐号，注意免费帐号的内容都必做是公开的，如果要用私有帐号，必做花钱购买。 
![github_signin](https://f.cloud.github.com/assets/2130097/267054/521968c6-8e52-11e2-835a-c159f6476d08.jpg)



**2.创建仓库** 
![github_respo](https://f.cloud.github.com/assets/2130097/267051/3bbd3774-8e52-11e2-8974-a9b235588f7b.jpg)

 helloword项目的主页如下所示，包括项目介绍，项目地址，项目源代码等 
![github_hellworld](https://f.cloud.github.com/assets/2130097/267056/7b56da20-8e52-11e2-87b4-68d95473f0c4.jpg)


**3.设置SSH访问权限** 
如果想在本地提交自己的代码，必段在帐号设定里增加公匙，公匙可以从/home/xxx/.ssh/id_rsa.pub文件中获取，没有这个文件的话需要执行如下命令生成： 
```
ssh-keygen -t rsa
```
![github_ssh](https://f.cloud.github.com/assets/2130097/267058/892e7d56-8e52-11e2-9ddc-2ede142a1faf.jpg)


**4.checkout代码本地** 
```bash
#把代码checkout到本地，和svn co类似
git clone git@github.com:yikebocai/helloword.git helloword
```

**5.修改一个文件并提交到服务器** 
```bash
#在README.md文件中增加一行内容，然后提交 
git add README.md
#windows下一定要用双引号，否则会报error: pathspec &#039;commit&#039;&#039; 
#did not match any file(s) known to git.
git commit -m &#039;add one line&#039;
git push origin master 
#如果没有设置SSH免登录，需要输入GitHub的用户名和密码才能提交代码成功
```

**6.查看变更** 
查看helloword项目主页，可以发布README.md文件已被修改 

**参考** 
<a href="http://www.worldhello.net/gotgithub/index.html" target="_blank">GoGitHub</a>，写的非常好的GitHub使用教程

 
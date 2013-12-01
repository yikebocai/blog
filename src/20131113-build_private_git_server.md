搭建私有Git服务器

Tag:Git

>在大公司时，服务器部署、代码和文档管理等都有专职的工程师负责，你只管使用就行了，有问题他们会帮你搞定，但创业团队钱少人寡资源有限，一切都要自己动手，源代码管理就是其中的一项。

关于Git的入门介绍，严重推荐廖雪峰写的[Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)，浅显易懂，非常适合入门者。话说当初学习Git时，老是搞不清楚一些原理，看这个之后才有恍然大悟的感觉。其次是推荐[Pro Git](http://ikandou.com/book/7188000/),记得最初学习Git时就是读的这本。

[GitHub](https://github.com)本来是非常理想的代码托管网站，但建私有库是不免费的，有各种不同的[收费方案](https://github.com/pricing)，对于一个10+的开发团队来说每月至少25美刀以上，也是一笔不少的开支，本能自己动手丰衣足食的原则，还是自己搭建一套吧，搞技术的就是这点好，不用求人。

[GitLab](http://gitlab.org/)是一个完全模仿GitHub而开发的一个开源私人代码托管应用，但是经过我多次尝试，都没有安装成功，所依赖的东西实在太多了，有一个组件安装不成功就完蛋了，这个时候觉得Mac下的dmg和Windows下的exe安装方式实在太他妈让人喜爱了。

源代码托管最主要是解决我们创业小公司源代码的版本控制和管理，刚开始并不需要提供Web服务，在线查看源代码做代码比较等功能，因此安装不上只好先暂时放弃。至于搭建一个私有的Git服务器，实在是非常简单，只要安装好Git后，创建一个仓库就行。

但前面的Git教程有一点有误导，比如你在/home/git/repo下生成一个仓库叫test.git，远程访问时应该是这样：`git clone git@103.21.140.1:repo/test.git`  ,repo前面不能加斜杠，如果按上文那样会报如下错误：

```
git clone git@103.21.140.1:/repo/test.git
Cloning into 'test'...
fatal: '/repo/test.git' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

为了给不同的人分配不同的权限，方便管理，可以使用[Gitolite]()，可以控制到每个仓库的每个分支甚至目录，还可以通过正规表达式做规则匹配，以达到更灵活的控制需求。用户的公钥做为一个文件放到keydirs目录下，文件名必须是上述设置的用户名，比如:xinbo.pub，admin的权限也是通过gitolite-admin这个库里的文件来实现的，因此管理员必须有该仓库的读写权限。

如果想只让用户提交到分支，而不能提交到master，可以在`conf/gitolite.conf`文件加加如下内容: 

```
 repo test
          - master = xinbo
          RW         = xinbo
```



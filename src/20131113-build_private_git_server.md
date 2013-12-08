搭建私有Git服务器

Tag:Git

>在大公司时，服务器部署、代码和文档管理等都有专职的工程师负责，你只管使用就行了，有问题他们会帮你搞定，但创业团队钱少人寡资源有限，一切都要自己动手，源代码管理就是其中的一项。

关于Git的入门介绍，严重推荐廖雪峰写的[Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)，浅显易懂，非常适合入门者。话说当初学习Git时，老是搞不清楚一些原理，看这个之后才有恍然大悟的感觉。其次是推荐[Pro Git](http://git-scm.com/book/zh),记得最初学习Git时就是读的这本。

[GitHub](https://github.com)本来是非常理想的代码托管网站，但建私有库是不免费的，有各种不同的[收费方案](https://github.com/pricing)，对于一个10+的开发团队来说每月至少25美刀以上，也是一笔不少的开支，本能自己动手丰衣足食的原则，还是自己搭建一套吧，搞技术的就是这点好，不用求人。

[GitLab](http://gitlab.org/)是一个完全模仿GitHub而开发的一个开源私人代码托管应用，但是经过我多次尝试，都没有安装成功，所依赖的东西实在太多了，有一个组件安装不成功就完蛋了，这个时候觉得Mac下的dmg和Windows下的exe安装方式实在太他妈让人喜爱了。

源代码托管最主要是解决我们创业小公司源代码的版本控制和管理，刚开始并不需要提供Web服务，在线查看源代码做代码比较等功能，因此安装不上只好先暂时放弃。至于搭建一个私有的Git服务器，实在是非常简单，只要安装好Git后，创建一个仓库就行。


## Git的安装
在CentOS6.4上使用`yum install git`安装的git版本比较老，是1.7.1的，如果想使用最新的1.8.5版本的代码，就需要自己手动编辑源代码安装 ，首先安装一些需要的开发包：

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
```

然后从github上获取最新的git代码 
```bash
$ git clone https://github.com/git/git
$ cd git
$ make configure
$ ./configure --prefix=/usr/local
$ make all
$ make install
```

然后退出重新登陆，执行`git --version`查看安装的最新版本号。

## Git服务器搭建

在root下创建git帐号，并切换到git用户下，创建一个新库,参考这篇文章
```bash
  $ useradd git
  $ su - git
  $ mkdire repo
  $ cd repo
  $ git init --bare test.git  
```
  然后在客户端就可以这样使用 `git clone git@host:repo/test.git`把代码拉到本地。在参考廖雪峰的文章时，怎么也clone不成功，后来发现参考文章里有一点问题，比如你在/home/git/repo下生成一个仓库叫test.git，远程访问时应该是这样：git clone git@103.21.140.1:repo/test.git  ,repo前面不能加斜杠，如果按上文那样会报如下错误：

```
git clone git@103.21.140.1:/repo/test.git
Cloning into 'test'…
fatal: '/repo/test.git' does not appear to be a git repository
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```
## Git权限管理

为了给不同的人分配不同的权限，方便管理，可以使用[Gitolite](https://github.com/sitaramc/gitolite/)，可以控制到每个仓库的每个分支甚至目录，还可以通过正规表达式做规则匹配，以达到更灵活的控制需求。用户的公钥做为一个文件放到keydirs目录下，文件名必须是上述设置的用户名，比如:xinbo.pub，admin的权限也是通过gitolite-admin这个库里的文件来实现的，因此管理员必须有该仓库的读写权限。

首先是安装[Gitolite](https://github.com/sitaramc/gitolite/)，在服务器git帐号$HOME目录下创建公钥文件，比如：xinbo.pub，将本地.ssh/id_rsa.pub里的内容复制到这个文件中

```bash
  $ git clone git://github.com/sitaramc/gitolite
  $ mkdir -p $HOME/bin
  $ gitolite/install -to $HOME/bin
  $ $HOME/bin/gitolite setup -pk xinbo.pub
```
  如果在执行install时出现`Can't locate Time/HiRes.pm in @INC …`错误，需要用root帐号执行`yum install perl-Time-HiRes`安装perl的这个组件。

安装完毕后，Gitolite会首先创建一个gitolite-admin的仓库，这个是用来进行权限管理，前面增加的xinbo.pub是为了有权限修改该仓库里的配置文件。然后把该仓库clone到本地，如果有需要权限修改或增加新的仓库，直接修改`conf/gitolite.conf`文件即可，如果增加了一个新库的配置，push时Gitolite会自动创建一个祼库，非常的方便。

如果有新人加入，需要在keydirs下增加一个pub文件，把他的公匙文件考到该目录下，并命名为比如说xinbo.pub，然后把xinbo这个帐号在conf/gitolite.conf做相应的权限分配，比如如果想只让用户提交到分支，而不能提交到master，可以在`conf/gitolite.conf`文件加加如下内容: 

```
 repo test
          - master = xinbo
          RW         = xinbo
```

## 迁移Git库

原来Git部署到了一个VPS服务器上，这个服务器同时还部署了公司的官网，最近官网频繁受到攻击而被服务商暂停使用，因此需要将Git仓库迁移到另外一个服务器上，这时Git分布式的优势就体现出来了，你完全不用担心Git服务器被攻击或损坏后代码丢失，只要有一份完整的代码拷贝，你就可以随时搭建出新的Git服务器。

* 在新服务器创建一个目录repo，然后进入repo目录并初始化一个裸仓库 `git init --bare test.git`
* 修改本地工作目录，比如test/.git/config文件，将remote url修改成新的仓库地址，比如:git@newhost:repo/test.git ，或者直接执行命令`git remote set-url origin git@newhost:repo/test.git`进行修改
* 将本地代码推送到新的仓库 `git push origin master`
* 从新仓库clone下这个库，可以用tree命令查看代码结构验证是否已推送成功 `git clone git@newhost:repo/test.git`
* 如果使用了Gitolite-admin，会更方便一点。首先
开发新人学习指南

>创业公司招人很难，招有经验的人难上加难，因此往往会退而求次，找一些刚毕业没多久有潜力的新人，但新人因为经历不多往往不知道自己要学什么，根据自己这么多年的经验教训，总结了一个开发新人指南，希望能给他们一些参考。


工欲善其事，必先利其器。要做一个好的开发，必先要有得以应手的工具。Windows虽然进入门槛很低，但使用它做开发平台越久会越有一种无力感，它本来也就不是面向程序员的，有这种感觉也正常。随着Linux在服务端的大行其道，使用Linux做开发的也越来越多，尤其喜欢Linux下强大的shell脚本，让我的工作更加简单便捷和自动化。Mac OS X本身也保留了Unix的精髓，是UI和命令行的完美结合，如果有钱也上Mac，没钱就用Linux，Windows还是工作娱乐时完吧，作为一名搞技术的专业人士，咱们怎么着也得专业一点，不是吗？这篇文章《[好程序员不用windows作开发环境](http://jianshu.io/p/3006d1cdd8ff)》说出了我的心声，《[为什么国外的程序员爱用Mac？](http://www.vpsee.com/2009/06/why-programmers-love-mac/)》这篇文章解释了使用Mac的人其实不是为了装逼，它确实有非常多独道之处。


## Linux
* 首先强烈推荐《[鸟哥的Linux私房菜](http://linux.vbird.org/)》，看完这个你玩转Linux基本没有问题了 
* 重点学习档案和目录管理，shell，vim，正则表达式，iptables等
* 全面了解bash请读《[Bash Guide for Beginners](http://tille.garrels.be/training/bash/)》

## Web
* Http协议是学习web编程的基础，很多做了多年web编程的人都不了解最基础的HTTP协议，只能一辈子做个最低级的码农，浅显一点的推荐这篇文章《[深入理解HTTP协议](http://wenku.baidu.com/view/b05bc7df5022aaea998f0feb.html)》，深入学习的话推荐《[HTTP权威指南](http://book.douban.com/subject/10746113/)》
* css是做高端、大气、上档次的前端页面的必备武器，css的盒模型概念是基础，推荐这个视频教程《[CSS样式](http://itercast.com/library/3/course/45)》
* Javascript是使用前端动态交互的重要手段，推荐阮一峰写的《[Javascript标准参考教程](http://javascript.ruanyifeng.com/)》，jquery虽然是非常不错的工具，但先了解基础的东西能更好的发挥工具的作用
* 上述的技术都是一个叫浏览器的工具来实现的，想了解它的工作原理可以参考这篇中文《[浏览器渲染原理简介](http://coolshell.cn/articles/9666.html)》和这篇英文《[How Browser Work](http://taligarsiel.com/Projects/howbrowserswork1.htm)》
* 访问一个网站都是通过域名来访问的，而服务器对外的标识只有IP，DNS在其中承担了中介的作用，使得我们更方便的记住网站的地址，DNS的工作原理参考这篇《[互联网域名解析系统DNS的工作原理及相关服务配置](http://guodayong.blog.51cto.com/263451/1173678)》
* 模型引擎，现在的网站纯静态的已经很少了，JSP这种把html、js、java代码进行杂交的技术用的也不多了，MVC是当前的主流，了解一种模型引擎确有必要，推荐Apache的[Velocity](http://velocity.apache.org/engine/devel/user-guide.html)
* [JSON](http://www.json.org/json-zh.html)，是非常流程的数据传输方式，实际工作中出场的机会非常之多，甚至还有基于此的[RPC](http://en.wikipedia.org/wiki/Remote_procedure_call)框架[JsonRPC](http://en.wikipedia.org/wiki/JSON-RPC)
* [Firebug](http://getfirebug.com/)，Firefox的一个前端调试插件，深入了页面元素的html、css定义的必备工具，调试Js也非常方便
* [HttpFox](https://addons.mozilla.org/zh-cn/firefox/addon/httpfox/)，另外一款FireFox插件，可以很方便地知道一个页面在加载时访问哪些URL，对理解页面的实现很有帮助
* 作为目前最流程的前端框架，[Bootstrap](http://www.bootcss.com/)可以帮我们快速搭建漂亮专业的网站
* 高性能Web网站需要的技术还有很多，比如负载均衡、反向代理、Cache等，这本《[构建高性能Web站点](http://book.douban.com/subject/3924175/)》介绍的比较全面
 
## Java
* 作为一个Java程序员，这门语言是我们吃饭的家伙，需要掌握的知识很多，但最基础的是抽象和继承、接口实现、网络和IO、并发和多线程等，推荐经典的《[Java编程思想](http://book.douban.com/subject/1101158/)》
* 并发和多线程也是Java语言的一大优势，在实际应用中使用广泛，深入理解他的实现机制，推荐《J[ava并发编程实战](http://book.douban.com/subject/10484692/)》，还有一个[并发编程网站](http://ifeve.com/)上也很非常多的干货
* 做Web开发当然少不了要了解Servlet，所有的Web框架都基于此，不要做一个只会依赖框架的程序员，看看这篇《[初学Java Web开发，请远离各种框架，从Servlet开始](http://www.oschina.net/question/12_52027)》，顺便看看这篇《[Servlet工作原理](http://www.ibm.com/developerworks/cn/java/j-lo-servlet/)》，深入一点看这篇《[Java Servelt技术简介](http://www.ibm.com/developerworks/cn/education/java/j-intserv/index.html)》
* JVM作为Java以及越来越多动态语言的运行平台，了解它的基本原理有助于我们更好的地排查问题优化程序，推荐周志明的《[深入理解Java虚拟机](http://book.douban.com/subject/6522893/)》

## 数据库
* 数据库是使用场景最多的基础系统，了解一些数据库的基本原则，对于我们日常开发大有裨益，比如赶集网首席DBA总结的《[MySQL数据库开发的三十六军规](http://ishare.iask.sina.com.cn/f/20250459.html)》
* 深入了解索引的实现原理，有助于我们创建更合理的索引，或者对SQL进行性能优化,这篇《[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)》介绍的很清楚
* MySQL是目前最流程的开源数据库，之前阿里同事简朝阳写的《[MySQL性能调优和架构设计](http://book.douban.com/subject/3729677/)》是一本很不错的深入了解MySQL实现原理的入门书

## 其它
* [Startup News](http://news.dbanotes.net/)，小道君模仿[Hacker News](https://news.ycombinator.com/) 搭建的一个社会化技术新闻网站，是我获取最新技术咨询的首选站点，文章质量非常高
* [Github](https://github.com/),目前业界最流行的代码托管网站，基本上现在数的着的知名开源框架都在上面，是学习开源技术参与开源活动的不二之选
* [Sublime Text 2](http://www.sublimetext.com/)，一个强大的万能文本编辑器，并且跨平台，浏览和编写web文件比较方便
* [Evernote](https://evernote.com/)，几乎支持目前所有主流的平台，是最早推出的云笔记软件，推荐使用国内版印象笔记，之前用国际版的有点不太稳定
* [Markdown](http://wowubuntu.com/markdown/)，一种非常简洁的写作格式，让你专注内容而不是展现，GitHub上的文档默认都是这种格式，并对源代码语法高亮做了扩展
* [Pocket](http://getpocket.com/)，一个跨平台的稍后阅读的软件，可以让你把看到的好文章先保存下来，等有时间了慢慢看
* [Hostsplus](http://yaniswang.com/frontend/2012/08/07/hostsplus/),一个跨平台的Hosts管理软件，很方便让我们在多个环境中自由切换，一个编辑页面搞定一切
* 源代码比较工具，Mac下推荐[VisualDiffer](http://visualdiffer.com/)，Linux推荐[Meld](http://meldmerge.org/)，Windows下没有特别好用的，[WinMerge](http://winmerge.org/)还行
* 数据库客户库，MySQL推荐官方的[MySQL Workbench](http://www.mysql.com/products/workbench/)，跨平台是一大特点，Oracle推荐[SQL Handler](http://www.onlinedown.net/soft/91179.htm)，Java写的也是跨平台，并且不需要安装Oracle客户端，数据库相关信息配置进去就能使用。
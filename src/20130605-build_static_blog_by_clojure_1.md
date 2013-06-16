用Clojure搭建自己的静态博客（一）

Tag:Clojure,Blog,Github

>自己动手用Clojure搭建一个完全属于自己的博客系统，按自己喜欢的风格进行排版，本身不提供文章编辑功能，自动同步GitHub上的文章，在自己的博客站点展示，充分利用GitHub的优势。

## 缘由

去年七月份重新开始写博客，注册了域名`yikebocai.com`，买了一个虚拟空间，用WordPress搭建自己的独立博客。用了一段时间后，发现WordPress的垃圾评论特别多，后来安装了垃圾评论过滤插件才有好转，页面不好定制因为我几乎不懂PHP，几年前学的那点东西早丢到姥姥家了。文章格式排版尤其是源代码的引入不太方便，后来订阅`玉伯`的[WTP](https://github.com/lifesinger/lifesinger.github.com/issues?state=open)时发现他在用GitHub Issues写博客，非常简单方便又带有评论功能，最重要的是我大爱Markdown的书写格式和Github的排版，清新简洁优雅，于是人肉搬家搬到了GitHub上。

但用了一段时间后，发现Issues有几个问题，第一个是不能像Repository一样对内容进行变更记录，第二个是无法引用Repository里的文章，每次我还得人工去复制一遍到Issues里，因为我喜欢用Mou在本地写文章，然后Push到GitHub源代码仓库里，三是不方便搜索，包括站内和站外的，四是不方便分享。正好学了有一段时间的Clojure，想实战一下，就搭建个自己的博客来练手了，可以充分按自己的喜好来搭建。另外对近来大火的[Bootstrap](https://github.com/twitter/bootstrap)框架也想学习一下，团队内一个系统就用的这个框架，个人兴趣和工作也能兼顾。

GitHub本身是一个非常优秀的代码托管网站，其实它不仅仅可以做代码托管，很多人已开始用它做文章的托管，可以像代码一样记录每一次的变更，完全不用担心你的信息会丢失或搞不清楚。因此，在搭建自己的博客时，我就在想能不能直接读取GitHub上的文章，博客系统只用负责做展示就行，完全不用关心文章的编辑，因为我更喜欢在本地用`Mou`写好文章然后Push到GitHub上存储。

## 需求

博客要尽量简单，要充分利用其它网站的功能，但需要满足我的以下几个核心需求：

* **完全静态**。我的博客站点的生成，完全依赖于Github上的静态文章，不需要依赖后续的配置，比如：文章的标题，标签，发表时间等都能从文章里取到，这样在写好文章后只需要推到Github仓库里就完事了，不用再后续到博客站点上增加额外的修改。 
* **支持Markdown**。托管在Github的文章本来就是Markdown格式，但Web网站只支持Html，需要自动做Markdown到Html的格式转换。
* **源代码语法高亮**。作为一个码农，写文章时经常要引用到源代码，语法高亮绝对是刚需，另外我还特别需要支持Clojure的语法高亮功能，现在咱可能Clojure的发烧友。
* **支持评论和分享**。虽然写文章更多是为了记录自己的思考过程和结果，但如果只是封闭在自己的世界中不和同行交流，写文章的益处只能得到一半，正所谓`三个行必有我师`。如果碰巧有人看了觉得还不错，分享给其它的人更加好了。
* **支持文本搜索**。文章写多了，想要找到某一篇文章，搜索也是基本需求。
* **支持RSS订阅**。RSS也是博客的标配功能，万一有人觉得我文章写的不错想订阅，咱也得支持呀。
* **文章自动同步**。指定Github上仓库的URL和本地目录，能够通过手动触发或者定时器触发，自动同步文章。Keep it simple，换句话说，一切要傻瓜化。
* **系统配置管理**。能够自己实现用户名密码、同步URL和目录、博客标题等信息的配置，别让我再去修改源代码才能实现这些功能。

## 选型

需求清楚了，接下来就要甩开膀子大干一场了。不过先别急，Clojure世界已有非常多不错的[开源框架](http://www.clojure-toolbox.com/)，虽然想从头到尾写一个博客系统，但也没有必要把所有的东西都自己去实现了，就像你自己想建个房子，不需要自己做门窗做家具吧，有现成买来用就行，只要符合自己的要求。

Clojure世界里大名鼎鼎Web应用框架首推[Ring](https://github.com/ring-clojure/ring)了，目前是Clojure世界里Web编程的事实标准。它参考了Python的[WSGI](http://wsgi.readthedocs.org/en/latest/)和Ruby的[Rack](http://rack.github.io/)，定义和实现了Web应用的一些基本接口。它只依赖了Clojure和Servlet等一些非常基础的包，对底层的HTTP协议进行了封装，提供了基础的web组件，方便上层的Web框架、Web Server的实现。像上层的Web框架都是基于它来实现的，目前Ring已发展成了一个系列产品，最核心的是ring-core包。

[lib-noir](https://github.com/noir-clojure/lib-noir)是基于Ring的一个工具集，是从Web框架[Noir]()独立出来的一个基础组件，目前已和Noir没什么关系了，可以支持其它Web框架，像Compojure。它实现诸如有状态和Session和Cookie，静态资源文件管理，输入校验，内容缓存，路由过滤和重定向等功能。像路由过滤功能，有点像阿里自己的[Webx]()里的`valve`，可以在Request请求的处理过程中，加入自己特殊的功能，比如侧栏中展示的最近文章，是在每个页面都需要的，可以在这里做查询并放到Response里，完全在每个具体的页面里都去实现一遍。

再往上开发主要用到的是[Compojure](https://github.com/weavejester/compojure)，它主要实现了Web路由功能，比如当URI是`/`时，应该执行哪些代码，是`/blog`应该执行哪些代码，可以很方便地把不同的URI映射到我们的不同处理模块上。

为了能在页面上实现动态内容的加载，引入了一个模板引擎[Clabango](https://github.com/danlarkin/clabango)，功能比[Velocity](http://velocity.apache.org/)是简陋多了，但基本可以满足要求了。

一直做Java后端的开发，对前端技术知之甚少，虽然自诩审美还行，但要让我做个清新简洁的页面出来估计也是赶鸭子上架，好在有了现成的框架[Bootstrap](https://github.com/twitter/bootstrap)，可以让我们这些前端菜鸟也能快速做出非常专业漂亮的页面。同时，Bootstrap能够很好地支持多终端的显示，这些移动时代非常有吸引力。

另外，为了方便创建应用程序，引入了另外一个Web框架[Luminus](http://www.luminusweb.net/)，它可以方便地帮我生成基本的Web代码结构，启动应用程序，打成独立的Jar包或War包。

最后，虽然我写的是一个静态博客程序，但还是需要保存一些基本的配置信息，这就需求用到数据库，但我又不想实现像MySQL这些**重量级**的数据库，正好Luminus示例中也用到了[H2](http://www.h2database.com)这个轻量级的内存数据库，反正要保存的数据很少，之前在公司里做系统也用到过，就它了。访问数据库，可以使用Clojure提供的jdbc包，但我更喜欢Clojure风格的数据层框架[Korma](http://sqlkorma.com/)，可以使用Clojure的风格方便地进行数据库的操作。

主要的技术框架选型完毕，还有一些具体的实现细节需要解决方案，实现一个简单的博客程序，也TMD不容易呀，涉及到很多的技术细节。

首先，为支持我喜欢的Markdown格式，需要一个MD转HTML的组件，好在网上已在达人实现了叫[markdown-clj](https://github.com/yogthos/markdown-clj)，并且能够把[Github Flavored Markdown](https://help.github.com/articles/github-flavored-markdown)支持的源代码格式转成[SyntaxHightlighter](http://alexgorbatchev.com/SyntaxHighlighter/ )风格的`<pre>`标签，为后面做源代码语法高亮奠定了基础。SyntaxHightlighter默认是不支持Clojure的，需要自己下载插件[shClojureBrusher](https://github.com/sattvik/sh-clojure)。

评论和分享我不想自己实现了，一是复杂要考虑注册存储和反垃圾，二是已有非常不错的社会化评论和分享工具，国内的有[加网](http://www.jiathis.com/ )提供的社会化分享以及旗下的社会化评论[友言](http://www.uyan.cc/)，支持新浪微博、腾讯微博、人人网等的帐号登录，只要加几行Js代码就行了，还提供后面的分析统计功能。国外的有[Disqus](http://disqus.com/)，虽然我更喜欢它的风格，符合我一贯的审美简洁清爽，但考虑到咱的读者还没扩展到国外，还是选国内的吧。

为了实现同步功能，服务器上需要支持[Git](http://git-scm.com/)（需要梯子），以便可以从Github上自动Pull最新的文章，装个Git客户端就行了。另外，手动同步总是比较麻烦，需要定时器的功能，在[Clojure工具箱](http://www.clojure-toolbox.com/)里找到了类似Crontab的。指定Github上仓库的URL和本地目录，能够通过手动触发或者定时器触发，自动同步文章。对于定时器功能，我选了[Quartzite](http://clojurequartz.info/)。

搜索框架[Clojure工具箱](http://www.clojure-toolbox.com/)提供了好几个，但基于[Lucene](http://lucene.apache.org/)的[Clucy](https://github.com/weavejester/clucy)被我一眼看中，熟悉的东西总是让我比较相信，和现实世界里的道理一模一样，呵呵。

至此，技术选型全部完毕，上述的框架除了参考clojure邮件组里一位同学的[实现](https://github.com/baoliang/clojure-blog)外，也在Github看了这些框架的Star和Fork情况，尽量挑选比较流行的框架，有了问题也方便找答案。另外，这些框架绝大部分在Github上都有源代码，想深入了解的可以去阅读源代码。

接下来，会讲一讲用Clojure搭建静态博客系统的具体过程以及VPS购买和应用部署。




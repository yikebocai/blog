用Clojure搭建自己的静态博客（一）

>自己动手用Clojure搭建一个完全属于自己的博客系统，按自己喜欢的风格进行排版，本身不提供文章编辑功能，自动同步GitHub上的文章，在自己的博客站点展示，充分利用GitHub的优势。

去年七月份重新开始写博客，注册了域名`yikebocai.com`，买了一个虚拟空间，用WordPress搭建自己的独立博客。用了一段时间后，发现WordPress的垃圾评论特别多，后来安装了垃圾评论过滤插件才有好转，页面不好定制因为我不懂PHP，文章格式排版尤其是源代码的引入不太方便，后来发现有人用GitHub Issues写博客挻简单的，于是人肉搬家搬到了GitHub上。我非常喜欢GitHub的简约风格和Markdown的书写方式。

但用了一段时间后，发现Issues有几个问题，第一个是不能像Repository一样对内容进行变更记录，第二个是无法引用Repository里的文章，我还得人工去复制一遍，因为我喜欢用Mou在本地写文章，然后Push到GitHub源代码仓库里，三是不方便搜索，包括站内和站外的。正好学了有一段时间的Clojure，想实战一下，就搭建个自己的博客来练手了，可以充分按自己的喜好来搭建。

GitHub本身是一个非常优秀的代码托管网站，其实它不仅仅可以做代码托管，很多人已开始用它做文章的托管，可以像代码一样记录每一次的变更，完全不用担心你的信息会丢失或搞不清楚。因此，在搭建自己的博客时，我就在想能不能直接读取GitHub上的文章，博客系统只用负责做展示就行，完全不用关心文章的编辑，因为我更喜欢在本地用`Mou`写好文章然后Push到GitHub上存储。

博客要尽量简单，要充分利用其它网站的功能，但需要满足我的以下几个核心功能：

* **可以自动同步GitHub上的文章**。对于自动同步功能，可以通过shell脚本来实现，另外需要的定时功能，我选了[Quartzite](http://clojurequartz.info/)来实现定时器调度。
* **支持我自己约定的文章标题和发布时间**。对于发表时间，我约定了GitHub上的文章名格式为`yyyymmdd-xxx_yyy.md`，前面为发表时间，后面为英文标题，文章内容的第一行约定为中文标题。
* **支持Markdown格式的文章展示**。使用了[markdown-clj](https://github.com/yogthos/markdown-clj)把markdown格式的文章转换成Html文档。
* **支持GitHub风格的源代码高亮展示和Clojure源代码**。用markdown-clj转成html后，GitHub风格的源代码会转成[SyntaxHightlighter](http://alexgorbatchev.com/SyntaxHighlighter/ )风格的`<pre>`标签，引入该组件可以进行常用代码的高亮，但Clojure需要单独安装自己的[shClojureBrusher](https://github.com/sattvik/sh-clojure)。
* **能够支持社会化网络的分享和评估**。分享选用了[加网](http://www.jiathis.com/ )的分享功能，评论功能选了该网站旗下的[友言](http://www.uyan.cc/)
* **支持文本搜索便于查询历史内容**。选用了[Clucy](https://github.com/weavejester/clucy)，这个是基于Lucene的一个Clojure包装。
* **能够提供简单的系统配置方便管理**。实现用户名密码、同步URL、博客标题等信息的配置，采用了[H2](http://www.h2database.com)这个轻量级的可可持久化的内存数据库。

Web底层框架选择了[Ring](https://github.com/ring-clojure/ring)以及[lib-noir](https://github.com/noir-clojure/lib-noir)，上面的路由框架采用了[Compojure](https://github.com/weavejester/compojure)，Web框架使用了[Luminus](http://www.luminusweb.net/)，它可以方便地帮我生成基本的Web代码结构，模块引擎采用了[Clabango](https://github.com/danlarkin/clabango)，有一点点类似于之前公司常用的Webx，数据库层框架使用了[Korma](http://sqlkorma.com/)，可以使用Clojure的风格方便地进行数据库的操作。



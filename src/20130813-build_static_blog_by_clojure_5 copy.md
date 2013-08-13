用Clojure搭建自己的静态博客（五）

>前面几章基本把这个静态博客系统的核心功能都讲过了，这篇文章讲一下锦上添花的功能，比如：定时任务、RSS订阅、搜索等功能，虽然这些功能没有也没有大的影响，但要想成为高端大气的博客系统，这些功能又是必不可少的。

## 定时器

虽然在同步页面可以实现手动点击来同步文章，但这毕竟不是很方便，每次用客户端`Mou`写完文章push到GitHub上之后，我都希望能自动同步到我的博客站点，因此就需要用到了定时器功能。熟悉Linux的同学肯定都用到`crontab`功能，我记得09年刚到阿里的时候，很多定时器都是有运维写`crontab`来实现调度的，后来平台技术部开发了一个定时器平台叫`belfry`，可以很方便地定义和修改定时运行的时间，也有相应的日志记录和报警功能。不过，在这个博客系统里我不想用最简单的`crontab`，因为不方便管理也不方便触发我的同步程序，也不会使用`belfry`这样重量级的定时器平台，而采用了[quartz](http://quartz-scheduler.org/)的clojure实现版本[quartzite](https://github.com/michaelklishin/quartzite)。当然，你也可以利用Java本身的`TimerTask`来实现，不过有免费的好工具，咱就别重复造轮子了。

因为写文章最多也就一天一两篇，只需要每天或每12小时更新一次就可以了，因此在定时同步配置中，并不支持其它复杂的类似`crontab`的表达式，只有一个设置值就是同步周期并且是以小时为单位，每隔N小时运行一次。

首先，定义一个job，读取同步的URL和目录，然后调用同步函数进行数据同步：

```clojure
(defjob myjob [ctx]
  (let [path (config/get-value "path")
        url (config/get-value "url")]
    (do
      (timbre/info "Sync github blogs ...")
      (sync/sync-blog path url))))
```

接着定义一个调度器，在调度器中指定调度类型和周期、触发器、执行的job等信息：

```clojure
(defn myschedule
  [& m]
  (qs/initialize)
  (qs/start)
  (let [job (j/build
              (j/of-type myjob)
              (j/with-identity (j/key "jobs.noop.1")))
        tk (t/key "triggers.1")
        period (java.lang.Integer/parseInt (config/get-value "period"))
        trigger (t/build
                  (t/with-identity tk)
                  (t/start-now)
                  (t/with-schedule (schedule
                                     (repeat-forever)
                                     ;(with-repeat-count 10)
                                     (with-interval-in-hours period))))]
    ;; submit for execution
    (qs/schedule job trigger)
    ;; and immediately unschedule the trigger
    ;(qs/unschedule-job tk)
    ))
```

最后，在应用启动时调用`myschedule`把定时器启动起来，这需要在`handler`里的`init`函数中增加对它的调用：

```clojure
(defn init
  "init will be called once when
   app is deployed as a servlet on
   an app server such as Tomcat
   put any initialization code here"
  []

  (quartz/myschedule)
  (timbre/info "myapp quartzite start")
  )
```

## RSS订阅

自从微博、微信等工具火了之后，貌似使用RSS进行阅读的人越来越少了，就连`Google Reader`这样当年的大作都关闭大吉了，不过博客站点没有RSS订阅总觉得好像少了点什么，因此还是加上吧。

RSS功能其实说白了也简单，就是把文章按照协议生成一个XML文件，其中一个站点的RSS叫作一个`channel`，这个频道里可以定义博客标题、网站链接、描述以及版权等信息，然后是文章的列表，叫作`item`，里面包含文章标题、发布日期以及文章内容，内容可以你的摘要让用户点击链接后跳到你的博客页面阅读全文，也可以是全文能在RSS阅读器看到全部内容。格式如下所示：

```xml
<rss version="2.0">
<channel>
<generator>clj-rss</generator>
<title>一棵波菜</title>
<link>http://xinbo.me</link>
<description>bocai's blog</description>
<copyright>yikebocai@gmail.com</copyright>
<item>
<description>
写文章有时候很有激情总想写点什么出来，有时候就懒的动手即使有主题在脑中构思了很久，这一段时间的状态就是后者，这个系列后面按计划应该还有至少几篇，比如配置管理，搜… …
</description>
<pubDate>Thu, 11 Sep 3913 00:00:00 +0800</pubDate>
<link>
http://xinbo.me/blog?p=20130811-build_static_blog_by_clojure_4.md
</link>
<title>用Clojure搭建自己的静态博客（四）</title>
</item>
<item>
<description>
D0(9.27)：杭州~台北 D1(9.28)：台北~宜兰（70公里），新店、小格头、坪林公园、北宜公路、礁溪、宜兰 D2(9.29)：宜兰~和平（78公里），… …
</description>
<pubDate>Sun, 07 Sep 3913 00:00:00 +0800</pubDate>
<link>
http://xinbo.me/blog?p=20130807-taiwan_riding_plan.md
</link>
<title>国庆环台湾骑行计划</title>
</item>
</channel>
</rss>
```

把生成的xml内容输出到页面即可，不过要注意的一点是生成web页面默认的`content-type`是`text/html`，用来标识这个返回结果是一个网页，如果使用默认的方式，你会发现展示的是全部xml文件的网页，而不是浏览器已经支持的xml文件形式，因此需要将它设置成`application/xml`，所以在路由中我们不再需要渲染页面，而是指定类型后直接把原始结果输出：

```clojure
(defroutes home-routes
  (GET "/feed" [] (resp/content-type "application/xml; charset=utf-8" (feed/create-feeds)))
  )
```

## 搜索

Java开源世界中最具盛名的搜索工具是`Lucene`，既然Clojure能直接引用Java代码，那么使用Lucene也毫无压力，不过有人对它做了包装，以Clojure习惯的方式提供出来，叫[clucy](https://github.com/weavejester/clucy)。我们都知道，想要搜索之前，必须先建索引，而建索引的方式有两种，一种是直接放在内存中，一种是放在硬盘上，我们的文章又不多因此完全可以把索引放在内存中，每次有更新全部重建即可。

```clojure
(def index (clucy/memory-index))

(defn build-index [name title content]
  (do
    (timbre/info (str "build index: " title))
    (clucy/delete index {:name name})
    (clucy/delete index {:title title})
    (clucy/delete index {:content content})
    (clucy/add index {:name name
                      :title title
                      :content content})))

(defn search [keyword]
  (do
    (timbre/debug (str "search for :" keyword))
    (clucy/search index keyword 20)))
```

在同步文章时重新构建索引：

```clojure
(defn- sync-index [path]
  (let [bloglist (blog/load-blog-list (str path "/src/"))
        len (count bloglist)]
    (dotimes [x len]
      (let [blg (nth bloglist x)
            name (:name blg)
            title (:title blg)
            content (:content (blog/load-blog-content name))]
        (search/build-index name title content)))))
```
 
## 后记
 
 上述功能可以查看[源代码](https://github.com/yikebocai/blogapp/tree/master/myapp)中的quartzite.clj、feed.clj、search.clj以及sync.clj，当然还有handler.clj。也可以访问我用该系统搭建的[一棵波菜](http://yikebocai.com)。
 


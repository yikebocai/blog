用Clojure搭建自己的静态博客（三）

Tag:Clojure,Blog
>最近去城西上班，每天来回两三个小时，每天回到家确实感觉有点累懒得动笔。另外最近买的书也在阅读中，《Clojure编程》已囫囵吞枣读完了，《Javascript高级编程》读了个头，《Unix编程艺术》也刚刚读了个开头，越正是翻到哪里越得有意思就就看会儿，也挺好。

前面介绍了搭建静态博客所需要的框架和技术，以及首页的实现，本章讲讲最核心的功能，博客下文的展示。

## 正文展示
展示博客正文首先要加载博客文件的内容，并解析其中的发布时间、博客标题、关键词以及正文，下面的函数传入博客文件名称，然后得到上述的字段。

```clojure
;;load blog content by blog name
(defn load-blog-content [name]
  (let [blog (first (db/find-blog-by-name name))
        filename  (:name blog)
        postdate0 (first (clojure.string/split filename #"-"))
        path (config/get-src-path)
        blogpath (str path filename)
        tags (read-tags blogpath) 
        pageview (if (nil? (:pageview blog)) 0 (:pageview blog))]
    (do 
      (db/update-blog-stat (:id blog)  (+ 1 pageview) 0 0)
      (conj {} 
      {:postdate (format-postdate postdate0)}
      {:title (read-title (str path filename))}
      {:tags tags}
      {:hastag (if (> (count tags) 0) true)}
      {:content (read-content blogpath)})
      )
    )
  )
```
这里大量用到了`let`这个功能，它的意思有像Java代码中的部署变量，把一个表达式绑定到一个变量上，然后可以在下面进行引用，避免写重复的代码。`(db/find-blog-by-name name)`是调用了数据查询的一个功能，从数据库表中按名称得到一个结果集，然后使用`first`取第一个结果。数据库中读取的结果集都是用Map来表示的，因此第二步可以使用`(:name blog)`来得到key为name的value值，这里取Map的值和Java中略微不同更简洁，key值前加上冒号即可。发布日期是从文件名中解析出来的，用到了字符串分隔的函数，注意到最后一个表达式`#"-"`，是一个匿名函数，里面是一个正则表达式，比Java中String类里的分隔函数更为强大，比如可以这样来使用：

```
user=> (clojure.string/split "ns1h2e3re" #"\d")
["ns" "h" "e" "re"]
```

最后一个变量pageview使用了`if`函数，后面紧跟着是一个布尔表达式，如果表达式成立返回结果为默认值0，否则从`blog`这个Map取实际的值，有点类似Java中的`pageview=xxx==null?0:xxx`。

完成变量的绑定后，下面执行两个表达式，一个是更新数据库中的阅读次数，另一个是取得要展示的博客正文的各个属性字段。顺序执行表达式使用了`do`函数，它会依次执行后面的表达式，并返回最后一个表达式的值。`conj`是把多个Map进行合并，它其实是把后面的值都添加到第一个集合中，也可以用于List、Set等。`cons`也有连接集合和值的意思，但不同的是`conj`是把后面的值放到前面的集合中，而`cons`是把后面的集合和前面的值连接起来，比如：

```
user=> (cons 1 [2 3 4])
(1 2 3 4)
user=> (conj 1 [2 3 4])
ClassCastException java.lang.Long cannot be cast to clojure.lang.IPersistentCollection  clojure.core/conj (core.clj:83)
```

加载正文内容时，也引用到了一些其它函数，比如下面这个是对发布日期进行格式化：

```clojure
(defn format-postdate [postdate]
  (str 
    (.substring postdate 0 4) "/" 
    (.substring postdate 4 6) "/" 
    (.substring postdate 6)))
 ```
 读取标题和Tag：
 
 ```clojure
 (defn read-title [filename]
	(with-open [rdr (reader filename)]
		(first (line-seq rdr))))
		
;;second line
;;Tag:java,jvm
(defn read-tags [filename]
  (with-open [rdr (reader filename)]
    (let [tagline (nth (line-seq rdr) 2)]
      (if (.startsWith (.toLowerCase tagline) "tag")
        (.split (.substring tagline 4) ",")))))
 ```
 
 这里用到了`with-open`，它是简化了Java中的`try-catch-finally`异常处理结构，保证打开IO流并使用完毕后肯定进行关闭。
 
 
因为博客正文是从Github上同步过来的，文件的原始格式是`Markdown`的，因此首先需要把它转换成能在网页上正常显示的`Html`格式，这就用到了[markdown-clj](https://github.com/yogthos/markdown-clj)这个工具，它负责把Markdown格式的内容翻译成Html的内容。另外，因为文章标题和Tag也是在同一文件中的，所以在展示时把它去除，单独展示。

```clojure
;remove the title and tags
(defn read-content [filename]
  (let [html (util/md->html  filename) 
    offset (+ (.indexOf html "</p>") 4)
    substr (.substring html offset) 
    offset2 (+ (.indexOf substr "</p>" 4))
    substr2 (if (> offset2 4) (.substring substr offset2))]
    (if (.startsWith (.toLowerCase substr) "<p>tag")
      (str substr2)
      (str substr))))
 ```

主要功能已编写完毕，剩下的就是处理路由信息，当接收到指定的URI时调用该函数展示文章内容：

```clojure
(defn blog-page [p]
  (layout/render
    "blog.html" {:blog (blog/load-blog-content p)}))

(defroutes home-routes
  (GET "/blog" [p] (blog-page p)))
```

## 数据库访问

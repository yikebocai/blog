使用WordPress搭建自己的Blog站点

1. 首先注册一个域名，比如<a href="http://www.net.cn" target="_blank">万网</a>现在正在搞活动，com和net域名只需要55块钱一年，时间长的话更便宜 

2. 申请一个主机空间，最好是国外的，不用备案，说到这里可能很多有过经历的人可能忍不住要骂通信管理局这帮滚蛋了，吃人饭不干人事。我找的是<a href="http://www.bloghost.cn" target="_blank">BlogHost</a>，也是看到别人的博客用的这个才知道的，服务器托管在美国，支持php和MySQL,一年100块钱，600M空间，每月12G流量，足够用了 

3. 然后要设置DNS域名解析，进入万网后台管理，把申请的主机IP和域名绑定，过一两个小时就生效了 

4. 当前最流行的博客软件就是<a href="http://cn.wordpress.org" target="_blank">Wordpress</a>，模板很多插件很多，到英文官方站点被墙了(想不明白为啥被墙，它只不过是套工具软件而已)，可以到国内站点下载，最新版本是3.4.1 

5. 将wordpress压缩包通过FTP客户端比如<a href="http://filezilla-project.org/" target="_blank">FileZilla</a>,或者直接通过后台提供的文件管理功能上传，然后解压缩到public_html目录下,注意要把内容直接放到这个目录下 

6. 在浏览器输入http://xxx.com/readme.html ,按照教程安装即可。wordpress的内容都是存在数据库中，安装前要先在主机上创建一个数据库，安装是在数据库连接配置页面填上去就可以了，测试连通就好了 7.安装好之后可以设置自己的站点名称，建立文章的类别，设置自己喜欢的主题和插件，对码农来说肯定少不了放源代码，可以下载代码高亮插件WP SyntaxHighlight,支持大部分编程语言
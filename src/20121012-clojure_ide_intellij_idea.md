clojure IDE开发环境Itellij IDEA介绍

想用clojure开发大型程序，找一个趁手的开发利器是必不可少的，就像Java的开发利器Eclipse，首先想到也是它，支持各种编程语言，只要下载相应的插件就可以了。但使用了一下并不很爽，并且所说REPL容易假死，因此又去试用了一下Linux下的神器<a href="http://www.gnu.org/software/emacs/" target="_blank">Emacs</a>，但一开始的使用体验就让我很不爽，已经习惯了VIM的操作方式，Emacs的操作都要同时按CTRL或ALT键让我很难受，并且还要记各种命令，安装各种插件，烦的要死，也放弃了。最后找到了评价颇高的<a href="http://www.jetbrains.com/idea/" target="_blank">IntelliJ IDEA</a>+<a href="http://plugins.intellij.net/plugin/?idea_ce&pluginId=4050" target="_blank">La Clojure</a>的组合，简单使用了还是很不错的。虽然使用很简单，但对第一次使用IDEA开发的人来说，还是有点不熟悉，并且最新的La Clojure版本0.5.7和IDEA不兼容（试用了社区版11.1.3、10.5.4，终极版11.0.2都不行），有必要介绍介绍一下，避免大家和我一样走弯路。 

我是在Windows下配置开发环境的，首先去IDEA的官方网站下载免费的最新<a href="http://download.jetbrains.com/idea/ideaIC-11.1.3.exe " target="_blank">社区版本11.1.3</a>(ULTIMATE版本只能合作30天，社区版本功能已经很强大了），按提示安装。然后下载<a href="http://plugins.intellij.net/plugin/?idea_ce&pluginId=4050" target="_blank">La Clojure 0.4.216</a>版本，解压缩到IDEA安装目录的plugins目录下，然后重新启动IDEA，可以到File->Settings->Plugins下查看是否安装成功，也可以直接到这里点击“Install Plugin from disk"选择La Clojure的zip文件安装或者点击”Browse resposities"按钮搜索La Clojure直接从线上下载安装，然后重新启动。
![ 7](https://f.cloud.github.com/assets/2130097/267678/448e5738-8eb2-11e2-932b-004e0bbaab0c.png)


然后再来创建一个Clojure工程，亲自体验一下IDEA的强大 
![1](https://f.cloud.github.com/assets/2130097/267684/b4470598-8eb2-11e2-97bd-26e3b39a83a4.PNG)
![ 2](https://f.cloud.github.com/assets/2130097/267686/c7409592-8eb2-11e2-9e0d-04b0541d097d.png)
![ 3](https://f.cloud.github.com/assets/2130097/267687/cece8fee-8eb2-11e2-9e07-210605707e16.png)

如果是第一次创建Clojure工程，上面一步之后多出一步会让你先配置一下JDK的路径，之后将不会再出现。IDEA默认clojure的最高版本是1.3.0，选择Download会自动从线下载，但目前clojure已经到1.4.0版本了，所以可以自己指定使用最新的版本。
![ 4](https://f.cloud.github.com/assets/2130097/267692/eb4b76d2-8eb2-11e2-9457-b44ff051062e.png)

和Java工程类似的使用环境，一切都很自然没有太大学习成本，按快捷键CTRL+SHIFT+F10或者点击菜单Tool->Start Clojure Console可以调出REPL，方便命令行操作，IDEA特别强大的地方在于可以像Java一样自动提示和补全代码，REPL里也有这个功能，现在的人越来越懒的记太多东西了，这个功能超赞：）
![ 5](https://f.cloud.github.com/assets/2130097/267691/da39220e-8eb2-11e2-9e39-d7394185f19d.png)
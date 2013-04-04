>做为一名攻城狮，Mac操作系统下的终端软件`Terminal`不可不用，而Mac下的终端并不像Linux下那样，可以默认高亮，全部是一种颜色，尤其是VIM里面无法忍受，除非您练就一双火眼金睛，于万千行代码中一眼分辨出自己关注的重点。不过，这难不倒咱们做技术的人，Mac本身也是类Unix操作系统，Linux能干的，它也能干，不过得自己花点功夫而已。

Vim是我最常用的文本编辑器，但并不会自动进行语法高亮，通过`vim`命令打开文件后，需要输入`:syntax on`，显式地指示vim进行语法高亮才行。但每次这样未免太麻烦了，一个简单的方法是在用户目录下创建一个`.vimrc`的环境配置文件，里面添加上一行内容即可：
```
syntax on
```
有些文档中说要把这个文件的权限设置成`777`的可读写模式，我在本机上测试了一下，并不需要如此，建立一个普通文件即可，再次打开一个shell脚本文件，熟悉的感觉回来了，显示如下：
![ 13-4-4_ 9_32-10](https://f.cloud.github.com/assets/2130097/339273/457046dc-9d32-11e2-86f5-cc209eb123c6.jpg)

但如果想要在使用`ls`这些Linux命令时，也要高亮，就需要花费点周折了。首先需要安装一个类似Ubuntu下面`apt-get`命令的终端安装工具[homebrew](http://mxcl.github.com/homebrew/index_zh-cn.html)，这个工具是用ruby写的，Mac本身已自带的有，我的机器的版本是：
```sh
zmbp:~ zxb$ ruby --version
ruby 1.8.7 (2012-02-08 patchlevel 358) [universal-darwin12.0]
```
安装`homebrew`，只要在终端里执行下面一行代码即可：
```sh
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```
安装虽然最终显示成功了，但是细心的话你会注意到，中间的一个`Warning`，提示缺少一个命令行工具，如果不安装这个命令行工具你在下面进行软件安装时会发现安装失败。
```
zmbp:~ zxb$ ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
==> This script will install:
/usr/local/bin/brew
/usr/local/Library/...
/usr/local/share/man/man1/brew.1

Press ENTER to continue or any other key to abort
==> /usr/bin/sudo /bin/mkdir /usr/local
Password:
==> /usr/bin/sudo /bin/chmod g+rwx /usr/local
==> /usr/bin/sudo /usr/bin/chgrp admin /usr/local
==> Downloading and Installing Homebrew...
Warning: Install the "Command Line Tools for Xcode": http://connect.apple.com
==> Installation successful!
You should run `brew doctor' *before* you install anything.
Now type: brew help
```

其中[Xcode](https://zh.wikipedia.org/zh-cn/Xcode)是苹果公司提供的一套完整的IDE环境,可以用来开发OS X或iOS下的软件，但是它非常庞大有一个多G，如果你今后没有这方面的打算用不着下载这么庞大的一个软件，只用下载[Command Line Tools for Xcode](https://developer.apple.com/downloads/index.action)即可，只有一百多M。
![ 13-4-4_ 10_36](https://f.cloud.github.com/assets/2130097/339543/e44acc54-9d38-11e2-9b89-ef819ee625cb.jpg)

安装完毕之后，`homebrew`就可以正常工作，我们先和做一次更新，保持和服务器同步，它的软件源应该是通过`Git`来同步到同地的：
```
zmbp:~ zxb$ brew update
Initialized empty Git repository in /usr/local/.git/
remote: Counting objects: 107514, done.
remote: Compressing objects: 100% (44571/44571), done.
remote: Total 107514 (delta 76368), reused 90280 (delta 62012)
Receiving objects: 100% (107514/107514), 16.24 MiB | 514 KiB/s, done.
Resolving deltas: 100% (76368/76368), done.
From https://github.com/mxcl/homebrew
 * [new branch]      gh-pages   -> origin/gh-pages
 * [new branch]      go         -> origin/go
 * [new branch]      master     -> origin/master
 * [new branch]      superwip   -> origin/superwip
HEAD is now at 0a273da tmux: patch for Snow Leopard
Already up-to-date.
```

完成上面的工作后，才真正可以开始安装终端高亮的组件`coreutils`了，不过这个需要`xz`这个解压缩工具，感觉下载速度挺快但编译安装的过程有点慢要等几分钟，安装命令如下：
```sh
zmbp:~ zxb$ brew install xz coreutils
==> Downloading http://tukaani.org/xz/xz-5.0.4.tar.bz2
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/xz/5.0.4
==> make install
🍺  /usr/local/Cellar/xz/5.0.4: 58 files, 1.5M, built in 21 seconds
==> Downloading http://ftpmirror.gnu.org/coreutils/coreutils-8.21.tar.xz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/coreutils/8.21 --program-prefix=g
==> make install
==> Caveats
All commands have been installed with the prefix 'g'.

If you really need to use these commands with their normal names, you
can add a "gnubin" directory to your PATH from your bashrc like:

    PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"

Additionally, you can access their man pages with normal names if you add
the "gnuman" directory to your MANPATH from your bashrc as well:

    MANPATH="/usr/local/opt/coreutils/libexec/gnuman:$MANPATH"

==> Summary
🍺  /usr/local/Cellar/coreutils/8.21: 210 files, 9.6M, built in 80 second
```

然后生成颜色自定义文件：
```
gdircolors --print-database > ~/.dir_colors
```
并在`.bash_profile`文件中加入以下配置信息：
```sh
if brew list | grep coreutils > /dev/null ; then
  PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
  alias ls='ls -F --show-control-chars --color=auto'
  eval `gdircolors -b $HOME/.dir_colors`
fi
```
执行`source .bash_profile`命令使这些配置生效，然后测试一下，终于可以看到和Ubuntu等Linux操作系统类似的结果了，刀磨好开始努力砍柴吧！
![ 13-4-4_ 11_17-5](https://f.cloud.github.com/assets/2130097/339666/03c66d2e-9d3c-11e2-91ae-bd1dc095c11d.jpg)

**参考**
1.[让Mac OS X的终端多姿多彩](http://linfan.info/blog/2012/02/27/colorful-terminal-in-mac/)

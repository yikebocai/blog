版本管理工具Git入门

> *原文写于 2012-8-21*

1.首先在Linux环境下安装Git，Ubuntu下很简单 
```bash
simbo@desktop:~$sudo apt-get install git
simbo@desktop:~$sudo apt-get install gitk ##gui环境
``` 

2.设置全局用户名和邮件地址 

```bash
simbo@desktop:~/study/git$ git config --global user.name "bocai"
simbo@desktop:~/study/git$ git config --global user.email "xxx@gmail.com"
``` 

3.创建一个本地目录做测试 

```bash
simbo@desktop:~/study/git$ mkdir gittest 
simbo@desktop:~/study/git$ cd gittest/
``` 

4.进行本地仓库初始化 

```bash
simbo@desktop:~/study/git/gittest$ git init
Initialized empty Git repository in /home/simbo/study/git/gittest/.git/
simbo@desktop:~/study/git/gittest$ git add .
simbo@desktop:~/study/git/gittest$ git commit
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
``` 

5.git的基本使用 

```bash
simbo@desktop:~/study/git/gittest$ vi helloworld   ##创建一个新文件，加一行内容
simbo@desktop:~/study/git/gittest$ git add helloworld  ##添加这个文件到仓库
simbo@desktop:~/study/git/gittest$ git commit      ##提交 
simbo@desktop:~/study/git/gittest$ vi helloworld  ##再增加一行内容
simbo@desktop:~/study/git/gittest$ git diff  ##查看变更,类似svn status
diff --git a/helloworld b/helloworld
index bda66a8..73a026a 100644
--- a/helloworld
+++ b/helloworld
@@ -1 +1,2 @@
Hello,World
+second 

//查看状态,类似svn status
simbo@desktop:~/study/git/gittest$ git status 
# On branch master
# Changed but not updated:
#   (use "git add &lt;file&gt;..." to update what will be committed)
#   (use "git checkout -- &lt;file&gt;..." to discard changes in working directory)
#
#     modified:   helloworld
#
no changes added to commit (use "git add" and/or "git commit -a")

 ##提交更新
simbo@desktop:~/study/git/gittest$ git add helloworld 
simbo@desktop:~/study/git/gittest$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD &lt;file&gt;..." to unstage)
#
#     modified:   helloworld
#
 simbo@desktop:~/study/git/gittest$ git log
commit c6828ed2024f208d2e4d480736a446edf66f32dd
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 11:55:14 2012 +0800

    haaaa
``` 

6.分支管理 

```bash
##创建新分支
simbo@desktop:~/study/git/gittest$ git branch temp1
simbo@desktop:~/study/git/gittest$ git branch
* master
  temp1

##切换到新分支
simbo@desktop:~/study/git/gittest$ git checkout temp1
Switched to branch &#039;temp1&#039;
simbo@desktop:~/study/git/gittest$ git branch
  master
* temp1

##在helloworld文件中增加一行内容并提交 
simbo@desktop:~/study/git/gittest$ vi helloworld
simbo@desktop:~/study/git/gittest$ git commit -a
[temp1 a54c05f] my temp1 branch work
1 files changed, 1 insertions(+), 0 deletions(-)
simbo@desktop:~/study/git/gittest$ git status
# On branch temp1
nothing to commit (working directory clean)
simbo@desktop:~/study/git/gittest$ git log
commit a54c05f75262475bf2389c9662410f8029d962b4
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 13:36:21 2012 +0800

    my temp1 branch work

commit 1a712b4683d48474867473b90f1d2a059d874f9c
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:34:47 2012 +0800

    llllll

commit 27446b28ff6aafd4361d689512ba90035c513c74
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:33:31 2012 +0800

    eeeee

##回到master分支,再在helloword文件中增加一行内容
simbo@desktop:~/study/git/gittest$ git checkout master
Switched to branch &#039;master&#039;
simbo@desktop:~/study/git/gittest$ git log
commit 1a712b4683d48474867473b90f1d2a059d874f9c
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:34:47 2012 +0800

    llllll

commit 27446b28ff6aafd4361d689512ba90035c513c74
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:33:31 2012 +0800

    eeeee

commit c6828ed2024f208d2e4d480736a446edf66f32dd
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 11:55:14 2012 +0800

    haaaa
simbo@desktop:~/study/git/gittest$ vi helloworld
simbo@desktop:~/study/git/gittest$ git diff
diff --git a/helloworld b/helloworld
index f4058f8..69cab0f 100644
--- a/helloworld
+++ b/helloworld
@@ -1,3 +1,4 @@
Hello,World
second
dddd
+my master branch work
simbo@desktop:~/study/git/gittest$ git commit -a
[master 37c3ec4] my master
1 files changed, 1 insertions(+), 0 deletions(-)
simbo@desktop:~/study/git/gittest$ git log
commit 37c3ec418682f178ea44704a9b89a1c6e38a0f74
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 13:37:55 2012 +0800

    my master

commit 1a712b4683d48474867473b90f1d2a059d874f9c
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:34:47 2012 +0800

    llllll

commit 27446b28ff6aafd4361d689512ba90035c513c74
Author: bocai &lt;xxx@gmail.com&gt;
Date:   Tue Aug 21 12:33:31 2012 +0800

    eeeee

##对两个分支进行合并
simbo@desktop:~/study/git/gittest$ git merge temp1
Auto-merging helloworld
CONFLICT (content): Merge conflict in helloworld
Automatic merge failed; fix conflicts and then commit the result.
simbo@desktop:~/study/git/gittest$ vi helloworld  

Hello,World
second
dddd
&lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD
my master branch work
=======
temp1 branch
&gt;&gt;&gt;&gt;&gt;&gt;&gt; temp1
simbo@desktop:~/study/git/gittest$ git commit -a
[master 9626ced] Merge branch &#039;temp1&#039;

##删除临时分支 
simbo@desktop:~/study/git/gittest$ git branch -d temp1
Deleted branch temp1 (was a54c05f).
simbo@desktop:~/study/git/gittest$ git branch
* master
```

## 参考
1.<a href="http://www.docin.com/p-46661785.html" target="_blank">看日记学Git</a>
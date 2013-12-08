在CentOS 6.4上安装Oracle 11g

>之前在阿里的时候，公司的技术人员有明确的分工，有专职的运维、DBA、配置管理员等，你需要的机器、数据库、代码库、文档库等都有专门的人员给准备，去创业公司了一切都得自己动手。因为项目需要，要在Linux上安装好久没碰过的Oracle。

## 环境准备

创业公司条件艰苦，我们只能拿一机台式机来做开发服务器，并且还没有显示器，不过这难不倒我们，使用远程桌面来控制也是一样的，而Linux上非VNC莫属了。

首先在服务端安装VNCServer，用于远程桌面控制

 * 执行`yum install tigervnc-server tigervnc`安装软件
 * 执行`/etc/init.d/vncserver restart`重启使之生效
 * 如果启动时报“Starting VNC server: no displays configured                [FAILED]”，修改/etc/sysconfig/vncservers:
 
 ```
VNCSERVERS=“1:admin"
VNCSERVERARGS[1]="-geometry 1024x768 -alwaysshared -depth 24"
 ```
 
 * 执行`vncpasswd`命令，修改admin 帐号通过vnc客户访问的密码 
 * 然后在客户端安装好vnc viewer后进行连接。
 * 你会发现使用vnc viewer连接时可能还是连不上，是因为iptables没有把vnc的连接端口开放，需要修改iptables，放开vnc server的端口，vi /etc/sysconfig/iptables，添加如下几行信息：
 
 ```
     -A INPUT -m state --state NEW -m tcp -p tcp --dport 5801 -j ACCEPT
     -A INPUT -m state --state NEW -m tcp -p tcp --dport 5901 -j ACCEPT
     -A INPUT -m state --state NEW -m tcp -p tcp --dport 6001 -j ACCEPT
```
 * 在vnc viewer输入192.168.1.xxx:1就可以连上了

## 安装Oracle 11g
 * 首先执行`rpm -qa|grep xxx `Oracle依赖的类库是否已安装完整，没有的要手动安装,除了[compat-libstdc++](http://fr.rpmfind.net//linux/RPM/centos/6.4/x86_64/Packages/compat-libstdc++-33-3.2.3-69.el6.i686.html)要下载rpm包安装外，其它的都可以使用yum install安装
 
 ```
 binutils-2.17.50.0.6
compat-libstdc++-33-3.2.3
elfutils-libelf-0.125
elfutils-libelf-devel-0.125
elfutils-libelf-devel-static-0.125
gcc-4.1.2
gcc-c++-4.1.2
glibc-2.5-24
glibc-common-2.5
glibc-devel-2.5
glibc-headers-2.5
kernel-headers-2.6.18
ksh-20060214
libaio-0.3.106
libaio-devel-0.3.106 
libgcc-4.1.2
libgomp-4.1.2
libstdc++-4.1.2 
libstdc++-devel-4.1.2
make-3.81
sysstat-7.0.2
unixODBC-2.2.11
unixODBC-devel-2.2.11
```

* 创建组和用户

```
[root@fmdev1 ~]# groupadd oinstall
[root@fmdev1 ~]# groupadd dba
[root@fmdev1 ~]# useradd -g oinstall -G dba oracle
[root@fmdev1 ~]# passwd oracle 
```

* 修改内核参数,在`/etc/sysctl.conf`后增加如下崆,并通过`/sbin/sysctl -p` 重启生效

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.shmmax = 536870912 
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586 
```

* 修改用户限制，vi /etc/security/limits.conf

```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536 
```

* 修改用户验证选项，vi /etc/pam.d/login，增加一行

```
session    required     pam_limits.so
```

* 修改用户配置文件,vi /etc/profile，增加下面内容

```bash
if [ $USER = "oracle" ]; then
  if [ $SHELL = "/bin/ksh" ]; then
     ulimit -p 16384
     ulimit -n 65536
  else
     ulimit -u 16384 -n 65536
  fi
fi 
```

* 创建安装目录

```
[root@fmdev1 ~]# mkdir -p /u01/oraInventory
[root@fmdev1 ~]# chown -R oracle:oinstall /u01
[root@fmdev1 ~]# chmod -R 775 /u01
```

* 修改用户配置，在oracle帐号下 vi ~/.bash_profile，增加

```
export ORACLE_BASE=/u01
export ORACLE_HOME=$ORACLE_BASE/oracle
export ORACLE_SID=fmora001
export ORACLE_UNQNAME=$ORACLE_SID 
export PATH=$ORACLE_HOME/bin:$PATH:$HOME/bin
```

* 设置使用oracle用户安装时显示图形界面。先用能显示图形界面的帐号登录，然后在Terminal里执行`xhost +` ，授权所有其它的用户都能连接X Window,然后`su - oracle`转换到oracle帐号下，执行`export DISPLAY=:1`设置图形界面在该用户下显示。DISPLAY环境变量的格式是hostname:displaynumber.screennumber,其中hostname指的是运行X server的机器名或IP，省略的话表在在本机运行.screennumber几乎都是0，displaynumber一般是0，如果使用了VNCServer而不是直接外接显示器，这个值是1。Xclient使用tcp连接时该值为X Server启动的端口号减去6000,因为VNCServer使用了6001端口，因此使用VNCViewer操作时要配成1。

* 启动oracle。不知道是什么原因，服务器是配置的静态IP，但安装Oracle后他读取到的IP另外一个莫名其妙的IP，listener.ora和tnsnames.ora里的主机IP都不正确，因此需要编辑/u01/oracle/network/admin/listener.ora和tnsnames.ora，将IP修改为实际的IP地址，否则无法正常使用。执行 $ORACLE_HOME/bin/dbstart 可以启动Oracle服务,或者以sysdba角色进行sqlplus，执行startup启动

* 编辑`/u01/oracle/network/admin/listener.ora`文件，oracle安装之后默认是没有侦听器相关的内容，你用Oracle SQL Handler进行连接时会发现提示找不到sid，其实不是sid不存在，而没有加入到LISTENER里面，它在接收到外部请求时不知道有这个SID的存在，从而就会报错。增加之后使用lnsrctl start,lnsrctl stop 进行启动和关闭。

```
SID_LIST_LISTENER =
    (SID_DESC =
      (GLOBAL_DBNAME = oracle11g)
      (ORACLE_HOME = /u01/oracle)
      (SID_NAME = fmora001)
    )
  ) 
```

* 开机自动启动Oracle服务和侦听器，首先无修改`/etc/oratab`配置文件，否则会报错：
`cat: /etc/oratab: No such file or directory`，从而 无法使用dbstart进行启动

```
# $ORACLE_SID:$ORACLE_HOME:<N|Y>
fmora001:/u01/oracle:Y
```

* 然后增加开机启动文件`/etc/init.d/oradb`，内容如下

```bash
#!/bin/bash
# chkconfig: 2345 90 10
export ORACLE_BASE=/u01
export ORACLE_HOME=$ORACLE_BASE/oracle
export ORACLE_SID=fmora001
export ORACLE_UNQNAME=$ORACLE_SID
export ORACLE_HOME_LISTNER=$ORACLE_HOME
export PATH=$ORACLE_HOME/bin:$PATH

ORCL_OWN="oracle"
# if the executables do not exist -- display error
if [ ! -f $ORACLE_HOME/bin/dbstart -o ! -d $ORACLE_HOME ]
then
   echo "Oracle startup: cannot start"
   exit 1
fi
# depending on parameter -- start, stop, restart
# of the instance and listener or usage display
case "$1" in
start)
# Oracle listener and instance startup
echo -n "Starting Oracle Instance ... "
su - $ORCL_OWN -c "$ORACLE_HOME/bin/dbstart"
echo -n "Starting Oracle Listeners ... : "
su - $ORCL_OWN -c "$ORACLE_HOME/bin/lsnrctl start"
touch /var/lock/subsys/oradb
#su - $ORCL_OWN -c "$ORACLE_HOME/bin/emctl start dbconsole"
echo "OK"
;;
stop)
# Oracle listener and instance shutdown
echo -n "Shutdown Oracle Listeners ... "
#su - $ORCL_OWN -c "$ORACLE_HOME/bin/emctl stop dbconsole"
su - $ORCL_OWN -c "$ORACLE_HOME/bin/lsnrctl stop”
echo -n "Shutdown Oracle Instance ... "
#su - $ORCL_OWN -c "$ORACLE_HOME/bin/emctl stop dbconsole"
su - $ORCL_OWN -c "$ORACLE_HOME/bin/dbshut"
rm -f /var/lock/subsys/oradb
echo "OK"
;;
reload|restart)
$0 stop
$1 start
;;
*)
echo "Usage: 'basename $0' start|stop|restart|reload"
exit 1
esac
exit 0  
```

* 使用`service oradb restart`进行重启
* 同样的，也需要在iptables里放开1521端口，否则外面是无法连上Oracle的

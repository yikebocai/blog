JVM监控分析工具之jps、jstat和jinfo

>日常工作中难免会遇到一些Java的异难杂症，尤其是线上环境，不可能让你去debug，而jdk
自带的一些小工具能让你快速定位问题所在，这些工具都是基于命令行的，远程连接服务器在终端下就可以操作，可以说十分方便。

## jps
jps命令类似Linux下的ps，只不过显示的是虚拟机的进程号，后面介绍到的jstat等命令都需要用到这个虚拟机进程号
<table>
   <tr>
       <td><b>选项</b></td>
       <td><b>作用</b></td>
   </tr>
    <tr>
       <td>-q </td>
       <td>只输出LVMID，省略主类的名称 </td>
   </tr>
   <tr>
       <td>-m </td>
       <td> 输出虚拟机进程启动时传给主类main()函数的参数</td>
   </tr>
   <tr>
       <td>-l </td>
       <td>输出主类的全名，如果进程执行的是Jar包，输出Jar的路径 </td>
   </tr>
   <tr>
       <td>-v </td>
       <td>输出虚拟机进程启动时的JVM参数</td>
   </tr>
</table>

通过一段源代码来测试一下这几个参数的含义
```java
package org.bocai.jvm.monitor;


/**
 * @author yikebocai@gmail.com
 * @since 2013-3-25
 * 
 */
public class Testjps {

	public static void main(String[] args) throws InterruptedException {
		
		if (args != null && args.length == 1) {
			System.out.println("Hello," + args[0] + "!");
		}

		while (true) {
			Thread.sleep(1000);
		}
	}

}

```
在Eclipse里运行Testjps程序，然后在命令行下可以看到jps的执行结果：
```bash
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ jps
3596 Jps
920 Testjps
4056

xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ jps -q
920
5544
4056
```

`-m`参数是会显示传给main函数的参数，为了测试这个参数，我们在命令行下执行Testjps，并传入一个任意字符串参数。
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ java org/bocai/jvm/monitor/Testjps world
Hello,world!

xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ jps -m
3944 Jps -m
4600 Testjps world
4056
```

`-l`参数会输出主类的全名，如下所示
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ jps -l
4912 sun.tools.jps.Jps
4600 org/bocai/jvm/monitor/Testjps
4056
```

如果执行的jar包，则输出jar的路径。为了测试这一情况，我们先对class文件进行打包。首先先创建一个MANIFEST文件，用来指定jar包运行时的主类是什么，文件名可以任意取，内容如下：
```
Main-Class: org.bocai.jvm.monitor.Testjps

```
然后用指定的MNIFEST文件和Java源文件目录来打包：
```
jar cvfm mymanifest .
```
最后运行jar：
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ java -jar Testjps.jar  jps
Hello,jps!

xinbo.zhangxb@ALI-031884N /d/work/jvm/gc/target/classes (master)
$ jps -l
288 sun.tools.jps.Jps
4056
2060 Testjps.jar
```

## jstat
jstat(JVM Statistics Monitoring Tool)是用来监控JVM虚拟机运行时的各种状态信息，比如GC情况、类装载情况等。jstat的使用方法如下：
```
jstat -gc 1s 10
```
其中第一个可选参数如下所示，第二个参数表示输出内容的间隔时间，第三个参数表示输出的总次数 
<table>
<tr><td>选项</td><td>作用</td></tr>
<tr><td>-class</td><td>监控类加载、卸载数量、总空间及类加载所耗费的时间</td></tr>
<tr><td>-compiler</td><td>输出JIT编译器编译过的方法、耗时等信息</td></tr>
<tr><td>-gc</td><td>监视Java堆状况，包括Eden区、两个Survivor区、老年代、永久代的容量、已用空间、GC时间合计等信息</td></tr>
<tr><td>-gccapacity</td><td>同-gc，额外输出堆中各个区使用到的最大和最小空间</td></tr>
<tr><td>-gccause</td><td>同-gc，额外输出导致上一次GC的原因</td></tr>
<tr><td>-gcnew</td><td>监视新生代GC情况</td></tr>
<tr><td>-gcnewcapacity</td><td>同-gcnew，额外输出新生代各区使用到的最大和最小空间</td></tr>
<tr><td>-gcold</td><td>监视老年代GC情况</td></tr>
<tr><td>-gcoldcapacity</td><td>类似-gcnewcapactiy</td></tr>
<tr><td>-gcpermcapacity</td><td>输出永久代使用到的最大和最小空间</td></tr>
<tr><td>-gcutil</td><td>同-gc，主要关注已使用空间的百分比</td></tr>
<tr><td>-printcompilation</td><td>输出已被JIT编译的方法</td></tr>
</table>

下面先运行一个简单的程序：
```java
package org.bocai.jvm.monitor;

/**
 * VM Args:-Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails
 * 
 * @author yikebocai@gmail.com
 * @since 2013-3-26
 * 
 */
public class Testjstat {
	private static final int _1MB = 1024 * 1024;

	public static void main(String[] args) throws InterruptedException {
		while (true) {
			byte[] alloc1 = new byte[2 * _1MB];
			byte[] alloc2 = new byte[2 * _1MB];
			byte[] alloc3 = new byte[1 * _1MB];

			// first Minor GC
			byte[] alloc4 = new byte[4 * _1MB];
			byte[] alloc5 = new byte[3 * _1MB];

			// second Minor GC
			byte[] alloc6 = new byte[4 * _1MB];

			alloc1 = null;
			alloc2 = null;
			// first Full GC
			byte[] alloc7 = new byte[2 * _1MB];

			Thread.sleep(100);
		}

	}

}

```
***
### -class
先查看类加载情况：
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc (master)
$ jstat -class 2124
Loaded  Bytes  Unloaded  Bytes     Time
    14    16.4        0     0.0       0.03
```
其中几列所代表的含义如下：
<table> 
<tr><td><b>Column</b></td><td><b>Description</b></td></tr>
<tr><td>Loaded</td><td>加载的Class文件数目</td></tr>
<tr><td>Bytes</td><td>加载的字节数，KB为单位</td></tr>
<tr><td>Unloaded</td><td>卸载的Class文件数目</td></tr>
<tr><td>Bytes</td><td>卸载的字节数</td></tr>
<tr><td>Time</td><td>Class文件加载和卸载花费的时间</td></tr>
</table>
*****
### -compiler
JIT编译情况:
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc (master)
$ jstat -compiler 2124
Compiled Failed Invalid   Time   FailedType FailedMethod
       4      0       0     0.00          0
```
其中几列所代表的含义如下：
<table> 
<tr><td><b>Column</b></td><td><b>Description</b></td></tr>
<tr><td>Compiled</td><td>执行的编译任务个数</td></tr>
<tr><td>Failed</td><td>编译失败的个数</td></tr>
<tr><td>Invalid</td><td>无效的编译任务个数</td></tr>
<tr><td>Time</td><td>编译任务花费的时间</td></tr>
<tr><td>FailedType</td><td>最后一个编译失败的编译类型</td></tr> 
<tr><td>FailedMethod</td><td>最后一个编译失败的Class名称和方法</td></tr> 
</table>
***
### -gc 
监控Java堆状况:
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc (master)
$ jstat -gc 2124
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT
1024.0 1024.0  0.0    0.0    8192.0   4096.0   10240.0     8327.7   12288.0 363.9    4302    5.927 6452    81.285   87.212
```
其中几列所代表的含义如下：
<table> 
<tr><td><b>Column</b></td><td><b>Description</b></td></tr>
<tr><td>S0C</td><td>Survivor0的大小，单位为KB，下同</td></tr>
<tr><td>S1C</td><td>Survivor1的大小</td></tr>
<tr><td>S0U</td><td>Survivor0的使用大小</td></tr>
<tr><td>S1U</td><td>Survivor1的使用大小</td></tr>
<tr><td>EC</td><td>Eden区的大小</td></tr> 
<tr><td>EU</td><td>Eden区的使用大小</td></tr> 
<tr><td>OC</td><td>Old区的大小</td></tr>
<tr><td>OU</td><td>Old区的使用大小</td></tr>
<tr><td>PC</td><td>Perm区的大小</td></tr>
<tr><td>PU</td><td>Perm区的使用大小</td></tr>
<tr><td>YGC</td><td>Young GC次数</td></tr> 
<tr><td>YGCT</td><td>YGC花费时间，单位为秒</td></tr> 
<tr><td>FGC</td><td>Full GC次数</td></tr>
<tr><td>FGCT</td><td>FGC花费时间</td></tr>
<tr><td>GCT</td><td>GC总时间</td></tr>
</table>
***
### -gcutil
监控Java堆的百分比情况，其中`-t`在第一列输出JVM启动到当前的时间单位为秒，`-hn`表示每隔几行输出一行标题：
```
xinbo.zhangxb@ALI-031884N /d/work/jvm/gc (master)
$ jstat -gcutil -t -h5 4488 1s 10
Timestamp         S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
           18.2   0.00   0.00  75.00  81.33   2.96    248    0.337   371    4.227    4.564
           19.2   0.00   0.00  75.00  81.33   2.96    262    0.355   392    4.458    4.813
           20.2   0.00   0.00  75.00  81.33   2.96    276    0.372   413    4.680    5.052
           21.2   0.00   0.00  75.00  81.33   2.96    290    0.389   434    4.909    5.298
           22.2   0.00   0.00   0.00  51.33   2.96    306    0.410   457    5.141    5.551
Timestamp         S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
           23.2   0.00   0.00   0.00  51.33   2.96    320    0.428   478    5.377    5.805
           24.2   0.00   0.00  87.50  91.33   2.96    334    0.446   500    5.630    6.075
           25.2   0.00   0.00  75.00  81.33   2.96    348    0.464   521    5.856    6.320
           26.2   0.00   0.00  50.00  81.33   2.96    362    0.482   542    6.081    6.564
           27.2   0.00   0.00  75.00  81.33   2.96    376    0.500   563    6.311    6.811
```
其中几列所代表的含义如下：
<table> 
<tr><td><b>Column</b></td><td><b>Description</b></td></tr>
<tr><td>S0</td><td>Survivor0使用的百分比，单位为%</td></tr>
<tr><td>S1</td><td>Survivor1使用的百分比</td></tr>
<tr><td>E</td><td>Eden区使用的百分比</td></tr>  
<tr><td>O</td><td>Old区使用的百分比</td></tr> 
<tr><td>P</td><td>Perm区使用的百分比</td></tr> 
<tr><td>YGC</td><td>Young GC次数</td></tr> 
<tr><td>YGCT</td><td>YGC花费时间，单位为秒</td></tr> 
<tr><td>FGC</td><td>Full GC次数</td></tr>
<tr><td>FGCT</td><td>FGC花费时间</td></tr>
<tr><td>GCT</td><td>GC总时间</td></tr>
</table>

其它的几个参数说明请[参见这里](http://docs.oracle.com/javase/1.5.0/docs/tooldocs/share/jstat.html)

## jinfo
jinfo(Configuration Info for Java)用来实时查看和调整虚拟机的各项参数，格式如下：
```
jinfo [option] pid
```
其中option选项主要包括：
<table> 
<tr><td><b>Option</b></td><td><b>Description</b></td></tr>
<tr><td><i>no option</i></td><td>相当于同时使用-sysprops和-flags参数</td></tr>
<tr><td>-flag name</td><td>打印某个指定的虚拟机参数值</td></tr>
<tr><td>-flag [+/-]name</td><td>enable或disbale某个JVM参数</td></tr>  
<tr><td>-fag name=value</td><td>设定指定的参数值</td></tr> 
<tr><td>-fags</td><td>打印所有的JVM参数</td></tr> 
<tr><td>-sysprops</td><td>打印所有系统属性信息</td></tr> 
</table>

举例如下：
```
[7080@icbu-qa-006 bin]$ jinfo -flag PrintGCDetails 2041
-XX:+PrintGCDetails

[7080@icbu-qa-006 bin]$ jinfo -flag -PrintGCDetails 2041
[7080@icbu-qa-006 bin]$ jinfo -flag PrintGCDetails 2041
-XX:-PrintGCDetails

[7080@icbu-qa-006 bin]$ jinfo -flags 2041
Attaching to process ID 2041, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11

-Dprogram.name=run.sh -Xms1024m -Xmx1024m -XX:PermSize=256m -XX:SurvivorRatio=2 -XX:+UseParallelGC -Dtrace.flag=true -Dtrace.output.dir=/home/7080/work/intl-risk/deploy/trace/ -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=7050,server=y,suspend=n -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -DdisableIntlRMIStatTask=true -Dcom.alibaba.dragoon.KV_TASK_KEY=com.alibaba.intl.risk.comsat.RiskPaymentPerformanceTask -Dcommons.log.query.size.limit=100000 -Djava.endorsed.dirs=/usr/alibaba/jboss/lib/endorsed

7080@icbu-qa-006 ~]$ jinfo -sysprops 2041
Attaching to process ID 2041, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11
java.vendor = Sun Microsystems Inc.
sun.java.launcher = SUN_STANDARD
catalina.base = /home/7080/work/intl-risk/deploy/../.myjboss
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
catalina.useNaming = false
os.name = Linux
...
```

但不知道为什么无法设置某个参数的值：
```
[7080@icbu-qa-006 bin]$ jinfo -flag SurvivorRatio=5 2041
Exception in thread "main" java.io.IOException: Command failed in target VM
	at sun.tools.attach.LinuxVirtualMachine.execute(LinuxVirtualMachine.java:200)
	at sun.tools.attach.HotSpotVirtualMachine.executeCommand(HotSpotVirtualMachine.java:195)
	at sun.tools.attach.HotSpotVirtualMachine.setFlag(HotSpotVirtualMachine.java:172)
	at sun.tools.jinfo.JInfo.flag(JInfo.java:105)
	at sun.tools.jinfo.JInfo.main(JInfo.java:58)
```

其它更多的用法请[参见这里](http://docs.oracle.com/javase/6/docs/technotes/tools/share/jinfo.html)

**参考**
1.[JDK Development Tools](http://docs.oracle.com/javase/6/docs/technotes/tools/)
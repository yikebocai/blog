JVM监控分析工具之jmap、jhat和jstack

>接上篇[JVM监控分析工具之jps、jstat和jinfo](https://github.com/yikebocai/blog/issues/30)，继续介绍下面几个常用工具

## jmap
jmap(Memeory Map for Java)命令主要用于成堆快照，或者查看堆内生成的对象实例。这个命令全部是用的查看heap内的信息或使用情况，参照jstack的命名，感觉叫jheap会更直接一点：）。生成堆快照文件除了jmap命令外，常用的还有-XX:+HeapDumpOnOutOfMemoryError参数，在OOM异常时生成，甚至通过-XX:HeapDumpOnCtrlBreak参数使用CTRL+Break键生成。32位虚拟机下jmap命令格式为：
```
jmap [option] vmid
```
如果64位虚拟机下命令格式为：
```
jmap -J-d64 [option] vmid
```
`option`的主要参数选项：
<table> 
<tr><td><b>Option</b></td><td><b>Description</b></td></tr>
<tr><td>-dump</td><td>生成堆快照。格式为：-dump:[live,]format=b,file=<filename>，其中live参数说明只是dump出存活的对象</td></tr>
<tr><td>-heap</td><td>显示堆详细信息，如使用哪种回收器、参数配置、分代状况等。只在Linux/Solaris平台有效</td></tr>
<tr><td>-histo</td><td>显示堆中对象的情况，包括类、实例数量和合计容量</td></tr>  
<tr><td>-finalizerinfo</td><td>显示在F-Queue中等待Finalizer线程执行finalize方法的对象。只在Linux/Solaris平台有效</td></tr> 
<tr><td>-permstat</td><td>以ClassLoader为统计口径显示永久代内存状态。只在Linux/Solaris平台有效</td></tr> 
<tr><td>-F</td><td>强制生成堆快照。只在Linux/Solaris平台有效</td></tr> 
</table>
使用`-dump`生成堆快照，可以使用jhat来启动一个内置浏览器来查看，但一般情况使用[MAT(Eclipse Memory Analyzer Tool)](http://eclipse.org/mat/)在本地分析更方便：
```
[7080@icbu-qa-006 intl-risk]$ jmap -dump:live,format=b,file=dump.bin 2041
Dumping heap to /home/7080/work/intl-risk/dump.bin ...
Heap dump file created
```
使用`-heap`打印堆使用情况：
```
[7080@icbu-qa-006 ~]$ jmap -heap 2041
Attaching to process ID 2041, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 1073741824 (1024.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 2
   PermSize         = 268435456 (256.0MB)
   MaxPermSize      = 268435456 (256.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 351076352 (334.8125MB)
   used     = 129532184 (123.5315170288086MB)
   free     = 221544168 (211.2809829711914MB)
   36.895730305412314% used
From Space:
   capacity = 3276800 (3.125MB)
   used     = 0 (0.0MB)
   free     = 3276800 (3.125MB)
   0.0% used
To Space:
   capacity = 3407872 (3.25MB)
   used     = 0 (0.0MB)
   free     = 3407872 (3.25MB)
   0.0% used
PS Old Generation
   capacity = 715849728 (682.6875MB)
   used     = 243342936 (232.06990814208984MB)
   free     = 472506792 (450.61759185791016MB)
   33.99357804882759% used
PS Perm Generation
   capacity = 268435456 (256.0MB)
   used     = 132895480 (126.73900604248047MB)
   free     = 135539976 (129.26099395751953MB)
   49.50742423534393% used
[7080@icbu-qa-006 ~]$ jmap -heap 2041
Attaching to process ID 2041, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 1073741824 (1024.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 2
   PermSize         = 268435456 (256.0MB)
   MaxPermSize      = 268435456 (256.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 351076352 (334.8125MB)
   used     = 166729928 (159.00605010986328MB)
   free     = 184346424 (175.80644989013672MB)
   47.49107339476969% used
From Space:
   capacity = 3276800 (3.125MB)
   used     = 0 (0.0MB)
   free     = 3276800 (3.125MB)
   0.0% used
To Space:
   capacity = 3407872 (3.25MB)
   used     = 0 (0.0MB)
   free     = 3407872 (3.25MB)
   0.0% used
PS Old Generation
   capacity = 715849728 (682.6875MB)
   used     = 243342936 (232.06990814208984MB)
   free     = 472506792 (450.61759185791016MB)
   33.99357804882759% used
PS Perm Generation
   capacity = 268435456 (256.0MB)
   used     = 132895952 (126.73945617675781MB)
   free     = 135539504 (129.2605438232422MB)
   49.50760006904602% used
```

使用`-histo`打印类和对象使用情况，分析内存不足分析哪些对象占用过多是非常有用：
```
 num     #instances         #bytes  class name
----------------------------------------------
   1:        976429       61598952  [C
   2:       1014749       32471968  java.lang.String
   3:        117790       31030592  [I
   4:        941548       30129536  java.util.HashMap$Entry
   5:        195992       29365832  <constMethodKlass>
   6:        195992       26666416  <methodKlass>
   7:        188663       20477160  [Ljava.util.HashMap$Entry;
   8:         17040       19159896  <constantPoolKlass>
......
7649:             1             16  sun.reflect.GeneratedSerializationConstructorAccessor1358
Total       6364828      405048144
```

使用`-permstat`查看永久代内存状态，这个过程会比较慢：
```
[7080@icbu-qa-006 intl-risk]$ jmap -permstat 2041
Attaching to process ID 2041, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11
45152 intern Strings occupying 4494952 bytes.
finding class loader instances .. 
Finding object size using Printezis bits and skipping over...
Finding object size using Printezis bits and skipping over...
.....
done.
computing per loader stat ..done.
please wait.. computing liveness..............................................liveness analysis may be inaccurate ...
class_loader	classes	bytes	parent_loader	alive?	type

<bootstrap>	2771	17192496	  null  	live	<internal>
0x00000000c1918f98	1	1968	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
0x00000000ca5c8b08	1	1984	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
0x00000000c4070e98	7	27872	0x00000000c0007d58	dead	com/thoughtworks/xstream/core/util/CompositeClassLoader@0x00000000b434c868
0x00000000c6895d18	1	1968	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
0x00000000c9e68780	1	1968	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
0x00000000c445d288	1	3144	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
0x00000000c9e55528	1	3144	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0
......
0x00000000c0924ab8	23	166728	0x00000000c0007d10	dead	org/jboss/mx/util/MBeanProxyExt$2@0x00000000b0869818
0x00000000c0a53830	0	0	0x00000000c0007d58	live	java/net/URLClassLoader@0x00000000b01d3770
0x00000000c6cfac30	1	3144	0x00000000c02b6f78	dead	sun/reflect/DelegatingClassLoader@0x00000000b00685b0

total = 907	21288	138310208	    N/A    	alive=104, dead=803	    N/A  
```

## jhat
JDK提供jhat（JVM Heap Analysis Tool）来解析jmap生成的堆快照文件，执行jhat命令后会启动一个小型的HTTP服务，通过http://host:7000来查看解析后的堆快照文件，不过这个命令很少用，一般工作过程中也不允许在服务器启动这样一个HTTP服务，通常是scp到本地使用Eclipse MAT来分析。

## jstack
jstack（Stack Trace for Java）用于生成虚拟机当前时刻的线程快照，常用来查找线程死锁的问题。
```
jstack [option] vmid
```
option参数如下：
<table> 
<tr><td><b>Option</b></td><td><b>Description</b></td></tr>
<tr><td>-l</td><td>显示锁的额外信息</td></tr>
<tr><td>-m</td><td>如果调用到本地方法的话，可以显示C/C++的堆栈</td></tr>
<tr><td>-F</td><td>强制生成线程栈信息</td></tr> 
</table>

打印栈信息：
```
[7080@icbu-qa-006 intl-risk]$ jstack 2041
2013-03-31 06:39:10
Full thread dump Java HotSpot(TM) 64-Bit Server VM (20.0-b11 mixed mode):

"Thread-9702" daemon prio=10 tid=0x00002aaab5d89800 nid=0x321 in Object.wait() [0x000000005876c000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000f78cd908> (a EDU.oswego.cs.dl.util.concurrent.LinkedNode)
        at EDU.oswego.cs.dl.util.concurrent.SynchronousChannel.poll(SynchronousChannel.java:353)
        - locked <0x00000000f78cd908> (a EDU.oswego.cs.dl.util.concurrent.LinkedNode)
        at EDU.oswego.cs.dl.util.concurrent.PooledExecutor.getTask(PooledExecutor.java:723)
        at EDU.oswego.cs.dl.util.concurrent.PooledExecutor$Worker.run(PooledExecutor.java:747)
        at java.lang.Thread.run(Thread.java:662)

"/127.0.0.1:50796 -> /127.0.0.1:15777 session read" daemon prio=10 tid=0x00002aaab3a84800 nid=0x7845 runnable [0x0000000059b80000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.read(SocketInputStream.java:129)
        at java.net.SocketInputStream.read(SocketInputStream.java:182)
        at java.io.DataInputStream.readShort(DataInputStream.java:295)
        at com.alibaba.dragoon.common.protocol.transport.socket.SocketSessionImpl.readMessage(SocketSessionImpl.java:87)
        at com.alibaba.dragoon.common.protocol.transport.socket.SocketSessionImpl.access$000(SocketSessionImpl.java:32)
        at com.alibaba.dragoon.common.protocol.transport.socket.SocketSessionImpl$1.run(SocketSessionImpl.java:213)

```

**参考**
1.[JDK Development Tools](http://docs.oracle.com/javase/6/docs/technotes/tools/)

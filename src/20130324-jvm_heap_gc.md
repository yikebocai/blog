Java堆内存分配和回收实践

Java运行期的内存结构包括`堆(Heap)`、`栈(VM Stack)`、`方法区(Method Aread)`、`本地方法栈(Native Method Stack)`、`程序计数器(Pragram Counter Register)`这几部分，其中和我们日常编程最密切的就是`堆(Heap)`了。

`堆(Heap)`又分为三部分，`Young(New)`、`Tenured(Old)`、`Perm`，对应的中文名叫法分别为`新生代`、`老年代`、`永久代`，而新生代又可以为分`Eden`、`From Survivor`、`To Survivor`，更详细的内存结构说明和GC回收算法请参考这篇文章《[JDK5.0中JVM堆模型、GC垃圾收集详细解析](http://blog.csdn.net/sfdev/article/details/4483442)》。

为了更深入地理解内存回收机制，下面的代码是我参照《[深入理解JAVA虚拟机-JVM高级特性与最佳实践](http://book.douban.com/subject/6522893/)》提供的例子而写，分别触发Minor GC和Full GC。

```Java
package org.bocai.jvm.gc;

/**
 * VM Params:-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails -XX:MaxTenuringThreshold=1 -XX:+PrintTenuringDistribution
 * @author yikebocai@gmail.com
 * @since 2013-3-20
 * 
 */
public class GCDemo {

	private static final int _1MB=1024*1024;
	
	public static void main(String[] args) {
		testMinorGC();

	}

	/**
	 * Minor GC Demo
	 */
	private static void testMinorGC() {
		byte[] alloc1=new byte[2*_1MB];
		byte[] alloc2=new byte[2*_1MB];
		byte[] alloc3=new byte[1*_1MB];
		
		//first Minor GC,5M Eden->Old
		byte[] alloc4=new byte[4*_1MB];
		
		byte[] alloc5=new byte[_1MB/4];
		
		//second Minor GC,alloc5:Eden->Surivior
		byte[] alloc6=new byte[4*_1MB];
		
		//if MaxTenuringThreshold>2 ,then move alloc5 into old,but whatever alloc5 is moved into old,why? 
		alloc4=null;
		byte[] alloc7=new byte[4*_1MB];
		//System.out.println(alloc5.length);
	}
	
	
	/**
	 * Full GC Demo
	 */
	private static void testFullGC() {
		byte[] alloc1=new byte[2*_1MB];
		byte[] alloc2=new byte[2*_1MB];
		byte[] alloc3=new byte[1*_1MB];
		
		//first Minor GC
		byte[] alloc4=new byte[4*_1MB];
		byte[] alloc5=new byte[3*_1MB];
		
		//second Minor GC
		byte[] alloc6=new byte[4*_1MB];
		
		alloc1=null;
		alloc2=null;
		//first Full GC
		byte[] alloc7=new byte[2*_1MB];
	}
	

}

```

上面的例子很简单，我的困惑来自于Surivivor的对象处理机制，如果`MaxTenuringThreshold`大于1,在分配alloc7对象时,alloc5对象只经历了一次Minor GC,还没有达到MaxTenuringThreshold设定的值,为何放在Survivor区的alloc5对象就被移到了Old区,并且不管这个参数的值设为多少,只要经历alloc7对象的分配,都会被移到Old区,如下所示:
 

```html
[GC [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     150912 bytes,     150912 total
: 5463K->147K(9216K), 0.0705580 secs] 5463K->5267K(19456K), 0.0706022 secs] [Times: user=0.00 sys=0.08, real=0.07 secs] 
[GC [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     262160 bytes,     262160 total
: 4586K->256K(9216K), 0.0106228 secs] 9706K->9619K(19456K), 0.0106669 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[GC [DefNew: 4352K->4352K(9216K), 0.0000259 secs][Tenured: 9363K->9619K(10240K), 0.4481241 secs] 13715K->9619K(19456K), [Perm : 369K->369K(12288K)], 0.4482521 secs] [Times: user=0.00 sys=0.03, real=0.45 secs] 
Heap
 def new generation   total 9216K, used 4259K [0x322a0000, 0x32ca0000, 0x32ca0000)
  eden space 8192K,  52% used [0x322a0000, 0x326c8fe0, 0x32aa0000)
  from space 1024K,   0%  used [0x32aa0000, 0x32aa0000, 0x32ba0000)
  to   space 1024K,   0% used [0x32ba0000, 0x32ba0000, 0x32ca0000)
 tenured generation   total 10240K, used 9619K [0x32ca0000, 0x336a0000, 0x336a0000)
   the space 10240K,  93% used [0x32ca0000, 0x33604dd0, 0x33604e00, 0x336a0000)
 compacting perm gen  total 12288K, used 369K [0x336a0000, 0x342a0000, 0x376a0000)
   the space 12288K,   3% used [0x336a0000, 0x336fc718, 0x336fc800, 0x342a0000)
    ro space 10240K,  51% used [0x376a0000, 0x37bccf58, 0x37bcd000, 0x380a0000)
    rw space 12288K,  54% used [0x380a0000, 0x38738f50, 0x38739000, 0x38ca0000)

```

我们注意到`from space 1024K,0%`,Survivor区的对象被移走了,把alloc7对象分配那行注释掉,再看一下:
```
[GC [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     150912 bytes,     150912 total
: 5463K->147K(9216K), 0.0067181 secs] 5463K->5267K(19456K), 0.0067671 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     262160 bytes,     262160 total
: 4586K->256K(9216K), 0.0081092 secs] 9706K->9619K(19456K), 0.0081469 secs] [Times: user=0.00 sys=0.02, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4515K [0x322a0000, 0x32ca0000, 0x32ca0000)
  eden space 8192K,  52% used [0x322a0000, 0x326c8fe0, 0x32aa0000)
  from space 1024K,  25% used [0x32aa0000, 0x32ae0010, 0x32ba0000)
  to   space 1024K,   0% used [0x32ba0000, 0x32ba0000, 0x32ca0000)
 tenured generation   total 10240K, used 9363K [0x32ca0000, 0x336a0000, 0x336a0000)
   the space 10240K,  91% used [0x32ca0000, 0x335c4dc0, 0x335c4e00, 0x336a0000)
 compacting perm gen  total 12288K, used 369K [0x336a0000, 0x342a0000, 0x376a0000)
   the space 12288K,   3% used [0x336a0000, 0x336fc700, 0x336fc800, 0x342a0000)
    ro space 10240K,  51% used [0x376a0000, 0x37bccf58, 0x37bcd000, 0x380a0000)
    rw space 12288K,  54% used [0x380a0000, 0x38738f50, 0x38739000, 0x38ca0000)

```

` from space 1024K,  25% used`里有alloc5对象,符合我们的预期,上面的情况一直没弄明白是什么原因.
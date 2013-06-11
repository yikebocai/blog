深入理解Java字符串连接

网上很多Java面试题都会有问到String字符串连接的性能，大家也都知道`+`号的性能远远差于StringBuffer或StringBuilder的`append`的性能，给出的答案是因为每次都要创建新的String对象，而StringBuffer或StringBuilder是不用创建的，其实不仅仅如此，如果比较一下字节码指令的话很容易弄明白。

先看一个简单的测试程序：
```java
public class TestString {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		testPlus();
		testStringBuilder();
	}

	public static void testPlus() {
		long start = System.currentTimeMillis();

		String str = "";
		for (int i = 0; i < 10000; i++) {
			str += i;
		}

		long end = System.currentTimeMillis();
		System.out.println("testPlus spend time:" + (end - start));
	}

	public static void testStringBuilder() {
		long start = System.currentTimeMillis();

		StringBuilder sb = new StringBuilder();
		for (int i = 0; i < 10000; i++) {
			sb.append(i);
		}

		long end = System.currentTimeMillis();
		System.out.println("testStringBuilder spend time:" + (end - start));
	}

	
	
}

```

运行结果如下，不出所料，StringBuilder要快的多：
```
testPlus spend time:492
testStringBuilder spend time:2
```

为什么StringBuilder会快这么多呢，我们用javap把Class文件反编译成字节码指令：
```shell
javap -v TestString
```

去除常量池，只看两个方法翻译成字节码的部分发现，`+`操作符其实在经过编译之后也会转成StringBuilder的`append`来做字符串连接，但是对比字节码之后发现，前者循环多了StringBuilder对象的创建以外额外三条指令，而后者要简洁很多，因此速度要快的多，如下图所示：
![testString](https://f.cloud.github.com/assets/2130097/276114/5f1aabd6-90a3-11e2-81e5-a08e7e69677f.png)
![testStringBuilder](https://f.cloud.github.com/assets/2130097/276117/65a57d00-90a3-11e2-8370-4c38fac0e0c4.png)

**后记**
发现学习JVM之后，之前不是很明白的基本概念，通过字节码的方式很容易就能理解，底层技术有可能永远在工作中用不到，但对于我们更深入地掌握日常工作中用的技能还是有很大帮助。

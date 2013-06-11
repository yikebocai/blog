Java Class文件解析

花了约10个晚上的时间，终于完成了第一版的Java文件反编译，不过离真正的反编译器还差很远，目前只做到了类似`javap -v`的功能，但Class文件的文件结构解析工作已基本完工，后面的工作就是做测试了，保证反编译的健壮性，发现反编译复杂的长文件时还会报错。另外就是还原源代码，真正做到反编译器的功能。但如果是仅仅从学习Class文件结构的角度出发的话，还原源代码的意义并不是很大。

Class文件结构最权威的还是《[The Java® Virtual Machine Specification](http://docs.oracle.com/javase/specs/jvms/se7/html/index.html)》这个官方文档，不过因为内容太多并且是英文版的读起来比较费劲，我参考的主要是周志明的《[深入理解Java虚拟机：JVM高级特性与最佳实践](http://book.douban.com/subject/6522893/)》以及很早之前国内翻译的第一本关于JVM的专著《[深入Java虚拟机（第二版）](http://book.douban.com/subject/1138768/)》，如果有不清楚的地方或者有些新增加的内容比如StackMapTable，我会再参考虚拟机规范的官方文档。

**参考**
[Java版反编译源代码](https://github.com/yikebocai/jvm/tree/master/decompiler/java)
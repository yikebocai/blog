【算法】去除字符串中多余的空格

经典算法书《<a href="http://book.douban.com/subject/2708786/" target="_blank">Algorithms in java</a>》中给出的方案是循环比对字符串中的每个字符，如果当前字符不为空或前一个字符不为空就继续，否则就忽略掉这个字符，利用clojure的特性，可以更简单点。该算法中用到如下几个函数，我们来分别了解一下它的含义。

**split**:进行字符串切分，可以指定分隔的字符 
**blank**:判断一个字符串是否为nil、空字符或者只包含空字符 
**not**：取反 
**filter**：对集合进行过滤，过滤时可以指定条件，以#开头 
**let**：可以绑定若干表达式到声明的变量上（可以理解成变量初始化），然后执行后面的表达式，绑定表达式都放在[]里面，后面可以跟多个表达式，但最后一个是该语句的返回结果 
**join**：把集合中的元素组合成一个字符串，可以指定组合时的分隔符 
**use**：相当于clojure中的reqire和refer,和java中的import一样，可以把非默认命名空间的包引用进来，并且可以不用全路径就可以使用 

```clojure
(use  'clojure.string  )

;;去除字符串中多余的空格
(defn remove-redundant-space [str]
  ;按空格分隔并过滤空字符串
  (let [x (filter #(not (blank? %)) (split str #" "))]
    (println x )
    x )  )

(println (join " "  ;将字符串序列按空格分隔并合成一个字符串
           (remove-redundant-space
             "  i ride bike   to longjing   mountain   ")
           )  )

```

 输出结果： 
```
i ride bike to longjing mountain
```
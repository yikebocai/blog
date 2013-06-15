使用Clojure进行文件操作

Tag:Clojure

学习Clojure也有一段时间了，感觉虽然对Clojure有了点认识，但除了在REPL里做些简单的测试外，真要想在实际中使用好像还无从下手，正好IntelliJ IDEA环境也搭建好了，拿来练练手。想了想，还是从最常用的文件操作来实现，毕竟日常工作中有很多文本的处理，用clojure实现应该比java简单很多。 
```clojure
(require '[clojure.java.io :as io])

;;追加写入文件
(defn appendFile [fileName content]
  (binding [*out* (java.io.FileWriter. fileName true)]
    (println content)
    (flush )))

(appendFile "my.log" "hello,bob")
(appendFile "my.log" "hello,zhang")

;;按行读取文件中的内容，并放到List中
(defn read-file-into-list [fileName]
  (with-open [rdr (io/reader fileName)]
     (doall (line-seq rdr)) ))

(println (read-file-into-list "my.log"))

;;打印文件中的每一行内容
(defn printFile [fileName]
  (with-open [rdr (io/reader fileName)]
    (doseq [line (line-seq rdr)]
      (println line))))

(println "---------")
(printFile "my.log")
```

输出结果： 
```
(hello,bob hello,zhang)
---------
hello,bob
hello,zhang
```
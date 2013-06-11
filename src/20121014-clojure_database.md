使用Clojure进行数据库操作

数据库是应用程序中使用最频繁的东东，clojure中简化了数据库的操作，几行代码就可以实现对数据库的操作。

首先，我们要下载clojure的一个数据级组件，实现了对jdbc的封装，叫<a href="https://github.com/clojure/java.jdbc" target="_blank">clojure/java.jdbc</a>，以前的名称叫作clojure.contrib.sql，因为没有直接打包好的jar包，因此需求我们自己Git下载进行编译。如果没有安装Git，首先下载<a href="http://msysgit.github.com/" target="_blank">Git fow Windows</a>并安装Git，然后在命令行下运行Git下载源代码： 
```
git clone git://github.com/clojure/java.jdbc.git
```
然后切换到java.jdbc目录，执行mvn install进行编译打包,如果没有安装maven，请先<a href="http://www.apache.org/dyn/closer.cgi/maven/maven-2/2.2.1/binaries/apache-maven-2.2.1-bin.zip" target="_blank">安装maven</a>。执行mvn命令后，会在target目录下生成编译好的jar包，将之拷贝到D:\work\lib目录下，同时也会自动下载依赖的相应jar，切换到C:\Users\Administrator\.m2\repository\mysql\mysql-connector-java\5.1.6,将Mysql驱动也拷贝到D:\work\lib目录下。回到IDEA环境中，打开File->Project Structure->Lib,将D:\work\lib目录添加到CLASSPATH中，如下图所示： 
![ 1](https://f.cloud.github.com/assets/2130097/267703/ff7b665c-8eb3-11e2-8992-b6532f7936a0.png)

下面是进行<a href="http://www.mysql.com/downloads/" target="_blank">MySQL</a>数据库操作的实例： 
```clojure
(use 'clojure.java.jdbc)

;设置端口和数据库名称的值
(let [db-host "localhost"
      db-port 3306 ; 3306
      db-name "risk"]

  ; 定义Msql连接属性
  (def db {:classname "com.mysql.jdbc.Driver"
           :subprotocol "mysql"
           :subname (str "//" db-host ":" db-port "/" db-name)
           :user "root"
           :password "hello1234"})

  ;数据库插入
  (with-connection db
    (insert-records :test
      {:id 10 :name "Tom"}
      {:id 11 :name "Jack"}))

  ;数据库查询
  (with-connection db ; 创建一个连接
    (with-query-results rs ["select * from test"] ;执行一条SQL
      (doseq [row rs]      ;遍历ResultSet，打印结果
        (println  (row :id) ":" (row  :name))))))   

```

输入结果： 
```
10 : Tom
11 : Jack
```

 [1]: http://yikebocai.com/wp-content/uploads/2012/10/图像-1.png
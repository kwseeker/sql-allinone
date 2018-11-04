### MySQL体系结构与存储引擎

数据库实例是由数据库后台进程/线程以及一个共享内存区组成。共享内存可以被运行的后台进程/线程所共享。需要注意的是，数据库实例才是真正用来操作数据库文件的。  

![MySQL体系结构](https://upload-images.jianshu.io/upload_images/2300270-d7347b38064ce886.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/578/format/webp)

1）最上层的连接服务，用于不同语言与SQL的交互。    
```
可以通过【show variables like '%connections%'】命令查看MySQL实例的最大连接数和单个用户的最大连接数。  
```
2）第二层MySQL Server，大多数MySQL的核心服务功能都在这一层，包括查询解析、分析、优化、缓存，所有跨存储引擎的功能都集中在这一层实现。  

+ Management Serveices & Utilities：系统管理和控制工具   
  备份和恢复的安全性，复制，集群，管理，配置，迁移和元数据。
+ Connection Pool：连接池  
  进行身份验证、线程重用，连接限制，检查内存，数据缓存；管理用户的连接，线程处理等需要缓存的需求。
+ SQL Interface：SQL 接口  
  进行 DML、DDL，存储过程、视图、触发器等操作和管理；用户通过 SQL 命令来查询所需结果。
+ Parser：解析器   
  查询翻译对象的特权；SQL 命令传递到解析器的时候会被解析器验证和解析。
+ Optimizer：查询优化器  
+ Cache 和 Buffer：查询缓存  
  全局和引擎特定的缓存和缓冲区  

3）第三层为存储引擎层，存储引擎负责MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。  
```
可以通过【SHOW ENGINES】命令查看各个存储引擎信息。
```

![](http://imgs.kwseeker.top/MySQLD%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E4%B8%8E%E7%BB%84%E6%88%90%E6%A8%A1%E5%9D%97.png)

### 客户端操作数据库执行流程
![](https://upload-images.jianshu.io/upload_images/2300270-35a11c289d778b20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/609/format/webp)

在 MySQL Server 中首先有一个 Cache，用来缓存 SQL 查询语句的查询结果，如果缓存命中，则直接返回结果；如果缓存没有命中，则对 SQL 语句进行语法分析，预处理和查询优化，最终得到该查询的执行计划，之后，该执行计划被送往数据库引擎，得到最终查询到的数据。数据库引擎并不会严格按照执行计划进行执行，而是会根据自身的架构和特性进行一些调整，以提高执行效率。

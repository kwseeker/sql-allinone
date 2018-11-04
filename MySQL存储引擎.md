## MySQL存储引擎简介

MySQL支持使用多种存储引擎，但是最常用的是InnoDB和MyISAM。

两者各有优缺和适用范围：  
![](http://imgs.kwseeker.top/%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E.jpg)

#### 锁和事务（InnoDB）

参考：《MySQL技术内幕InnoDB存储引擎》 第6、7章  



+ **很重要的用于测试的命令**
  ```
  set autocommit = 0;   -- 关闭自动提交，只对当前连接有效
  commit;               -- 手动提交，如果有多条sql,则执行这个命令一起提交
  ```

+ **锁的使用与并发场景下的测试**
  这里以InnoDB的锁讲解：参考《MySQL技术内幕InnoDB存储引擎》  

  分类：    
  排他锁(X锁)/共享锁(S锁)（sql语句不指定锁的话默认是没有锁的，则不会被其他锁阻塞）  
  行锁/表锁/页锁(MySQL特有)  
    MyISAM使用表级锁，InnoDB使用行级锁和表级锁  

  行锁和页锁可能出现死锁（为什么？）  

  测试  

  行共享锁测试  
  ```
  -- 连接1
  use employess;      --0
  desc departments;   --0
  set autocommit = 0; --0
  select * from departments where dept_no = 'd006' lock in share mode;  --2
  commit;             --5
  -- 连接2
  use employess;      --1
  desc departments;   --1
  select * from departments where dept_no = 'd006' lock in share mode;  --3 不会被阻塞，共享锁之间不互斥；
  select * from departments where dept_no = 'd006' for update;          --4 被阻塞,直到执行5
  ```

  行排他锁测试
  ```
  -- 连接1
  use employess;      --0 使用官方数据库
  desc departments;   --0 两列数据 dept_no是PRI dept_name是UNI
  set autocommit = 0; --0 关闭自动提交
  select * from departments where dept_no = 'd006' for update;  --2 排他锁，因为dept_no有索引，这里是行锁
  commit;             --3 提交
  -- 连接2
  use employees;      -- 1
  desc departments;   -- 1
  select * from departments where dept_no = 'd004' for update;  --2 不会被阻塞因为现在加的行排他锁,只有d006已经被加锁
  select * from departments where dept_no = 'd006' for update;  --2 会被阻塞，直到执行3，这里继续执行
  -- 超时的话： ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
  ```

  表共享锁
  ```
  -- 连接1
  use employees;      --0 数据库
  desc employees;     --0 数据表
  set autocommit = 0; --0
  select * from employees where first_name = 'Georgi' lock in share mode;   --2 表共享锁，因为first_name没有索引会加表锁
  -- 连接2 
  use employees;      -- 1
  desc departments;   -- 1
  select * from employees where first_name = 'Elvia' for update;    --3 会被阻塞（共享锁和排他锁互斥）
  ```

+ **活锁与死锁，死锁的解除**

  - **活锁**
    事务T1封锁数据对象R, 事务T2也请求封锁R, 于是T2等待；接着T3也请求封锁R； 当T1释放锁后，系统批准了T3的请求，后面T4也请求封锁R, 系统等T3释放锁后又批准了T4的请求；这样循环一直轮不到T2获取锁，这种情况叫做活锁。

    解决方法：
    采用先来先服务的队列策略。队列式申请。

  - **死锁**

+ **事务的使用与并发场景下的测试**
  - 事务的特点（ACID）
    Atomicity Consistency Isolation Durability  
    事务是恢复和并发控制的基本单位（以事务为单位回滚，并发时同步的最小单位是事务）
  - 事务的并发调度问题
    1）脏读  
      A事务读取B事务尚未提交的更改数据   
    2）不可重复读  
      A事务读取了B事务已经提交的更改数据  
    3）幻读  
      A事务读取了B事务已经提交的新增数据  
    4）丢失更新  
      A事务撤销时，把已提交的B事务的数据覆盖掉  
    5）覆盖更新
      A事务提交时，把已提交的B事务的数据覆盖掉
    
  - 事务的三级封锁协议

  - 事务四种隔离级别
    1）
    2）可重复读
    3）


#### 键

+ **InnoDB的几种键的区别**
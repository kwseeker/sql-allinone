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
  show engine innodb status\G;
  ```

+ **锁的使用与并发场景下的测试**
  这里以InnoDB的锁讲解：参考《MySQL技术内幕InnoDB存储引擎》  

  分类：    
  排他锁(X锁)/共享锁(S锁) [select语句不指定锁的话默认是没有锁的，且不会被其他锁阻塞, update/insert/delete InnoDB会自动添加排他锁]  
  意向共享锁（IS）/意向排他锁（IX）[意向锁是表级别的锁，用于在一个事务中揭示下一行将被请求的锁的类型，InnoDB自动处理，代码无法控制也无需控制]  
  间隙锁 [也是InnoDB自动控制的，对范围内数据加共享锁或排他锁时，会对这个范围不存在的记录也添加锁，为了解决幻读问题]  

  MVCC(Multi Version Concurrency Controll) 多版本并发控制，是实现非锁定读的一种方式；
  读取的行如果正在被执行delete、update操作并不会等待行上锁的释放，而是直接读取行的快照数据。

  行锁/表锁/页锁(MySQL特有)  
    MyISAM使用表级锁，InnoDB使用行级锁和表级锁也支持页级锁。  
    ```
    show status like 'innodb_row_lock%';
    ```
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

+ **乐观锁和悲观锁**  
  乐观锁和悲观锁并不是数据库中实现的锁机制，是需要我们自己实现的。

  乐观锁：  
  乐观锁对每一个数据维护一个版本号，每次读取时把版本号读取出来，更新后版本号+1；然后更新时将读取版本号作为条件，如果有其他实例更新了，
  版本号会变化，导致本次更新失败。  

  悲观锁：  


+ **活锁与死锁，死锁的解除**

  - **活锁**  
    事务T1封锁数据对象R, 事务T2也请求封锁R, 于是T2等待；接着T3也请求封锁R； 当T1释放锁后，系统批准了T3的请求，后面T4也请求封锁R, 系统等T3释放锁后又批准了T4的请求；这样循环一直轮不到T2获取锁，这种情况叫做活锁。

    解决方法：
    采用先来先服务的队列策略。队列式申请。

  - **死锁**

+ **事务的使用与并发场景下的测试**  
  事务的并发问题和隔离级别从隔离的实现（基于X/S锁）上很好理解。

  - 事务的特点（ACID） 
    Atomicity Consistency Isolation Durability  
    事务是恢复和并发控制的基本单位（以事务为单位回滚，并发时同步的最小单位是事务）

  - 事务的并发调度问题
    1）脏读（未提交读）  
      脏数据：指缓冲池中被修改的数据还没有被提交；
      A事务读取B事务尚未提交的更改数据   

    测试
    ```
    -- 事务A
    set session transaction_isolation='READ-UNCOMMITTED';   --设置为未提交读隔离级别，可能发生脏读
    show session variables like 'transaction_isolation'；
    begin；
    select * from departments；
    -- 这里执行事务B update
    select * from departments;      -- 这里就是属于脏读
    -- 这里执行事务B rollback；

    -- 事务B
    set session transaction_isolation='READ-UNCOMMITTED';
    show session variables like 'transaction_isolation'；
    begin；
    update departments set dept_name='SelfDefine11' where dpet_no='d010'；
    rollback; 
    ```

    2）不可重复读（使用RC解决脏读）  
      一个事务中多次读同一数据，事务还没结束时，另一个事务修改同一数据。

      测试
      ```
      -- 事务A
      set session transaction_isolation='READ-COMMITTED';
      begin;
      select * from departments;  -- 1
      -- 执行事务B update
      select * from departments； -- 2， 结果总是update提交后的结果
      -- 提交事务B commit
      -- 执行事务C并commit；
      select * from departments； -- 3， 2、3对比可能发生不可重复读

      -- 事务B
      set session transaction_isolation='READ-COMMITTED';
      begin;
      update departments set dept_name='SelfDefine21' where dept_no='d011';
      commit;

      --事务C
      set session transaction_isolation='READ-COMMITTED';
      begin;
      update departments set dept_name='SelfDefine31' where dept_no='d012';
      commit；
      ```

    3）幻读（使用RR解决脏读和不可重复读）  
      A事务第二次select读取了B事务已经提交的新增数据，不可重复读重点在于update和delete，幻读的重点在于insert。
      
      测试
      ```
      -- 事务A
      begin；
      select * from departments； --1
      -- 执行事务B update
      select * from departments； --2 
      -- 执行事务B commit
      -- 执行事务C commit
      select * from departments； --3 

      -- 事务B
      begin;
      update departments set dept_name='SelfDefine2' 
      where dept_no='d011';
      commit;

      -- 事务C
      begin；
      insert into departments values（‘d012’， ‘SelfDefine3’）；
      commit；
      ```

    4）丢失更新  
      A事务撤销时，把已提交的B事务的数据覆盖掉  
    5）覆盖更新
      A事务提交时，把已提交的B事务的数据覆盖掉
    
  - 事务的三级封锁协议

  - 事务四种隔离级别  
    1）未提交读（RU）  
      允许读取其他事物未提交的更改数据；  
      无法解决上述任何并发问题。  
      原理：事务在读数据的时候并未对数据加锁，修改数据的时候只对数据增加行级共享锁。  

    2）提交读（RC）  
      大多数数据库使用这个隔离级别；  
      只允许读取到事务提交的数据，可以解决脏读（未提交读）问题。  
      原理：事务对当前被读取的数据加行级共享锁（当读到时才加锁），一旦读完该行，立即释放该行级共享锁；更新某数据的瞬间（就是发生更新的瞬间），必须先对其加 行级排他锁（写锁），直到事务结束才释放。  
      
      A|B
      ---|---
      begin | begin
      select1(row_S,查完立即释放) | /
       / | update(row_X)
      select2(row_S,查完立即释放) | /
      / | commit(释放row_X)
      select3(row_S,查完立即释放) | /
      commit | / 

      由上图可知，实际执行过程是select1 -> update -> select2 -> select3，select2读取的必定是update提交之后的数据，不存在脏读问题；由于select2 select3之间还有间隙，很可能在2，3之间发生对该行的修改操作，出现不可重复读问题。  

    3）可重复读（RR）  
      MySQL默认隔离级别；  
      可解决脏读和不可重复读；  
      原理：事务在读取某数据的瞬间（就是开始读取的瞬间），必须先对其加行级共享锁，直到事务结束才释放；更新某数据的瞬间（就是发生更新的瞬间），必须先对其加行级排他锁，直到事务结束才释放。   

      A|B
      ---|---
      begin | begin
      select1(row_S) | /
       / | update(row_X)
      select2(row_S) | /
      / | commit（释放row_X）
      select3(row_S） | /
      commit(释放row_S)| / 

      这里三个查询语句是一起释放锁的，select1 -> select2 -> select3 -> update, 不存在脏读、不可重复读问题。  
      如果没有1的话，update -> select2 -> select3, 2\3之间没有间隙（3个查询三位一体），查询结果是相同的。

    4）可串行化  
      可解决脏读、不可重复读、幻读；  
      原理：事务在读取数据时，必须先对其加表级共享锁 ，直到事务结束才释放；更新数据时，必须先对其加 表级排他锁 ，直到事务结束才释放。  

#### 键

+ **InnoDB的几种键的区别**
### 备份

MySQL表存储方式：
备份需要注意的事项： 备份的一致性，

MySQL数据库备份的方法：  

按备份内容分为：  
+ 完全备份  
    - mysqldump  
        ```
        mysqldump -uroot -p --opt employees > employees-bkup.sql
        ```
    
    
+ 增量备份

+ 日志备份

按备份时服务器是否运行：  
+ 热备

+ 冷备

+ 温备



### 恢复


### 主从复制


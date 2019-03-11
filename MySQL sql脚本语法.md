### MySQL sql脚本语法

+ 变量的定义与赋值（DECLARE, SET, @）

    参考：  
    [13.7.4.1 SET Syntax for Variable Assignment]https://dev.mysql.com/doc/refman/5.7/en/set-variable.html
    [MySQL入门 SQL语言之十八：系统变量(全局变量、会话变量)，自定义变量(用户变量、局部变量)的使用]https://blog.csdn.net/qq_34626097/article/details/86528466

    会话(Session) 是和连接(Connection)是同时建立的，两者是对同一件事情不同层次的描述。简单讲，连接(Connection)是物理上的客户端同服务器的通信链路，会话(Session)是逻辑上的用户同服务器的通信交互。

    DECLARE 用来声明局部变量，作用于 BEGIN...END 之间。
    ```
    DECLARE variable_name datatype(size) DEFAULT default_value;
    示例：
    DECLARE x, y INT DEFAULT 0; //可以同时声明多个
    ```

    SET 用来为变量赋值,如为上面的x赋值。
    ```
    SET x = 10;
    ```

    @ 与 SET 一起用来指定用户变量，作用于当前会话（连接）；
    ```
    SET @my_name = 'Arvin';
    ```

    GLOBAL 与 SET 一起用来指定系统全局变量，作用于所有用户所有连接，
    相应的 SESSION/LOCAL 与 SET 一起用来指定系统会话变量，作用于当前会话（连接），系统为每个会话维护了一组配置。
    注意系统全局变量和系统会话变量的变量名都是系统定义的，SET GLOBAL my_name = 'Arvin'; 这种是会报“ERROR 1193 (HY000): Unknown system variable 'my_name'”的。
    ```
    set GLOBAL 变量名 = 值;
    set @@GLOBAL.变量名 = 值;   //和上一行等价
    set @@session.变量名 = 值
    -- 下面的写法意义相同
    SET SESSION sql_mode = 'TRADITIONAL';
    SET LOCAL sql_mode = 'TRADITIONAL';
    SET @@SESSION.sql_mode = 'TRADITIONAL';
    SET @@LOCAL.sql_mode = 'TRADITIONAL';
    SET @@sql_mode = 'TRADITIONAL';
    SET sql_mode = 'TRADITIONAL';   --默认为SESSION
    ```

+ 数据类型

    - 实现“数组”操作

        SQL中没有数组，但是可以使用字符串及相关函数间接实现数组的操作；
        ```
        declare awards varchar default '空奖,开星光商品奖励池,流星雨商品奖励池';      --定义“数组”
        declare award_count int default 0;
        declare i int default 0;
        declare award_name varchar default '';
        set award_count = length(awards) - length(replace(awards,',','')) + 1;  --获取“数组”长度
        while i <= award_count do
            select SUBSTRING_INDEX(SUBSTRING_INDEX(awards, ',', i), ',', -1) into award_name;   --遍历“数组”
        end while;
        ```

+ 特殊字符

    - 字符串连接符（||）

        可用于SQL脚本中字符串连接以及换行。
        
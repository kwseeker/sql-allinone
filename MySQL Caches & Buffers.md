## MySQL缓存和缓存池

#### MySQL缓存机制  
  每种存储引擎都支持缓存机制；InnoDB的缓存机制和MyISAM  
  select类型sql命令分发解析执行之前会先去查询缓存模块。  

  + 使用查询缓存的好处

    查询缓存从MySQL5.7.20开始被弃用，MySQL8.0中被删除。

    如下查询，第一次查询0.44s，后面重复查询耗时0.3s左右。
    ```
    select dept_name, first_name 
    from (departments d left join dept_emp de on d.dept_no = de.dept_no) 
      left join employees e on de.emp_no = e.emp_no 
    where last_name = "Facello";
    ```

  + LRU算法  
    缓存池的设计原理，使用了LRU算法。LRU也常用于web服务器的缓存。  
    ![](https://img.mukewang.com/5a9f4f6e0001ca0405250475.png)  

#### 缓存池的设置

  + 查看缓存配置参数
    ```
    mysql> show variables like '%query_cache%';
    +------------------------------+---------+·
    | Variable_name                | Value   |
    +------------------------------+---------+
    | have_query_cache             | YES     |
    | query_cache_limit            | 1048576 |
    | query_cache_min_res_unit     | 4096    |
    | query_cache_size             | 1048576 |
    | query_cache_type             | OFF     |
    | query_cache_wlock_invalidate | OFF     |
    +------------------------------+---------+
    6 rows in set, 1 warning (0.53 sec)
    ```
### MySQL系统变量

系统变量分为GLOBAL和SESSION（也可以写作LOCAL）变量；前者作用于整个服务器，后者作用于当前连接会话。
默认使用SESSION。
```
set GLOBAL sort_buffer_size=value；
set @@global.sort_buffer_size=value;

select @@global.sort_buffer_size;
show GLOBAL variables like 'sort_buffer_size';

set SESSION sort_buffer_size=value;
set @@session.sort_buffer_size=value;
set sort_buffer_size=value;

select @@sort_buffer_size;
select @@session.sort_buffer_size;
show session variables like 'sort_buffer_size';
```


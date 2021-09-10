# MySQL查询
json类型的like查询
```
select * from dynamic where content_json->'$.content' like '%商城%'
```
状态查询
```

show variables like '%innodb_flush_log_at%';-- 查询innodb引擎刷盘方式

SHOW VARIABLES LIKE 'datadir';--查询数据目录

SHOW GLOBAL STATUS like 'Innodb_page_size';--查询默认页大小 默认值16k

 show global variables like '%page%';
 
 SHOW VARIABLES LIKE 'innodb_file_per_table'
 
 SHOW VARIABLES LIKE 'innodb_autoextend_increment';
 
 SHOW VARIABLES LIKE 'innodb_log_buffer_size';-- 查询日志缓冲池大小
 
 show engine innodb status;-- 查询引擎状态
 
 SHOW VARIABLES LIKE 'innodb_adaptive_hash_index';
 
 SHOW VARIABLES LIKE 'innodb_adaptive_hash_index_parts';
 
 optimize table test; -- 优化页分裂 碎片等问题，重量级操作
```
慢查询
```
 SHOW VARIABLES LIKE '%long_query_time%';
 
 SHOW VARIABLES LIKE '%log_output%';
 
 SHOW VARIABLES LIKE '%slow%';
 
 show processlist;
 
 kill 1357916
 
 force index() -- 指定本次查询强制使用哪个索引，因为MySQL优化器的选择并不是最优的索引
 explain select  CustName,count(1) c from WorkOrder force index(ix_date) where CreateDate>'2016-5-1' and CreateDate<'2017-1-1' group by CustName
 having c>100 order by c desc;

 ignore index() -- 强制Mysql在查询时，不使用某索引
 explain select  CustName,count(1) c from WorkOrder  ignore index(ix_date) 
where CreateDate>'2016-5-1' and CreateDate<'2017-1-1' group by CustName having c>100 order by c desc;
```

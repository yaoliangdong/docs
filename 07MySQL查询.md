# MySQL查询
json类型的like查询
```
select * from dynamic where content_json->'$.content' like '%商城%'
```


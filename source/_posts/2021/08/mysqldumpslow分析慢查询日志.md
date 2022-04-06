---
title: mysqldumpslow分析慢查询日志
comments: true
tags: mysql
categories: mysql
abbrlink: 19102
date: 2021-08-21 16:16:02
translate_title: mysqldumpslow-query-log
---
按照平均查询输出5行慢查询记录
```shell
mysqldumpslow -s at -t 5 /phpstudy/data/slowquery.log
```
- -s   排序方式，可选值有c（记录次数）、t（查询时间）、l（锁定时间）、r（返回记录）、a（平均）
- -t    显示的记录数Spawn failed解决方式
- -g   后面跟正则表达式（如 left join），不区分大小写。
- -r   正序排序，即从小到大排序。
- -d  调试 debug
- -v   查看版本

按照平均查询时间排序且只显示含有left join的记录
```shell
mysqldumpslow -s at -g 'left join' /phpstudy/data/slowquery.log
```

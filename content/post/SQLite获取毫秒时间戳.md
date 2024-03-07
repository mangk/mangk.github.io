+++
title = 'SQLite获取毫秒时间戳'
date = 2024-03-07T11:34:24+08:00
draft = false
+++

# 先说结论
SQLite 获取`毫秒级时间戳`可以通过拼接的方式：
```SQL
-- 通过 SQLite 内置函数拼凑毫秒时间戳
SELECT (strftime('%s', 'now') * 1000 + substr(strftime('%f', 'now'),4,3)) AS milliseconds; -->结果：1709783374677
```
---

## 唠叨一下我在干什么
最近在开发中使用到了 SQLite 来讲数据保存到本地，并有条件的将数据同步到云端。所以我设计了表结果如下：
- 服务端表
```SQL
CREATE TABLE `book` (
  `id` int NOT NULL AUTO_INCREMENT,
  `local_id` bigint NOT NULL COMMENT '用户本地数据表id',
  `uid` int NOT NULL COMMENT '所属用户',
  other columns...
  PRIMARY KEY (`id`),
  UNIQUE KEY `user_local_data_idx` (`local_id`,`uid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
- 客户端 SQLite 表
```SQL
CREATE TABLE book	(
    id TIMESTAMP PRIMARY KEY,
    other columns...
);
```
计划本地使用`毫秒级时间戳`来作为 id 存储数据，同步到云端时，采用用户 id 和 本地数据 id 两个字段作为唯一键来存储数据。结果就遇到了 SQLite 存储`毫秒级时间戳`的问题。

正常情况下，我们在 SQLite 获取时间可以使用 `date`，`time`，`datetime`，`strftime`，`datetime`这些函数。
```SQL
SELECT date('now');     -->结果：2018-05-05
SELECT time('now');     -->结果：06:55:38
SELECT datetime('now'); -->结果：2018-05-05 06:55:38
SELECT strftime('%Y-%m-%d %H:%M:%S', 'now');    -->结果：2018-05-05 06:55:38

select strftime('%s','now');    -->结果：1525502284
select strftime('%s','2018-05-05');     -->结果：1525478400

select datetime(1525502284, 'unixepoch', 'localtime');  -->结果：2018-05-05 14:38:04
```
但是没有一个能获取到`毫秒级时间戳`。

最终我发现，可以通过字符串截取的方式先获取到毫秒时间，再和时间戳进行拼接来直接获取到`毫秒级时间戳`。下面是结论：
```SQL
-- 通过 SQLite 内置函数拼凑毫秒时间戳
SELECT (strftime('%s', 'now') * 1000+substr(strftime('%f', 'now'),4,4)) AS milliseconds; -->结果：1709783374677
```
上面我们成功在 SQLite 中获取到了`毫秒级时间戳`。
如果想让数据插入时自动填充，可以这样来写 Create 语句：
```SQL
CREATE TABLE book	(
    id TIMESTAMP PRIMARY KEY DEFAULT ((strftime('%s', 'now') * 1000 + substr(strftime('%f', 'now'),4,4))),
    other columns,
);
```
## 一些格式化参数，在 strftime() 方法中使用
|替换|描述|
|:---|:---|
|%d|一月中的第几天，01-31|
|%f|带小数部分的秒，SS.SSS|
|%H|小时，00-23|
|%j|一年中的第几天，001-366|
|%J|儒略日数，DDDD.DDDD|
|%m|月，01-12|
|%M|分，00-59|
|%s|从 1970-01-01 算起的秒数|
|%S|秒，00-59|
|%w|一周中的第几天，0-6 (0 is Sunday)|
|%W|一年中的第几周，01-53|
|%Y|年，YYYY|
|%%|% symbol|

例如：
```SQL
SELECT strftime('%Y-%m-%d %H:%M:%f'); -->结果：2021-08-06 06:35:00.333
```
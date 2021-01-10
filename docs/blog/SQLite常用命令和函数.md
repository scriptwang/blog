SQLite在某些场景下还是很有用处，比如最近在用Shell脚本+SQLlite做简单的业务处理，详细使用场景请看
http://1024.services/index.php/archives/46/
- sqlite 官网 https://www.sqlite.org/index.html
（暂时更新这些，当业务中用到了更为复杂的逻辑在更新）

# 进入命令行
```
sqlite3 test.db

# 退出sqlite
.quit
```

# 查看表/表结构
```
# 查看所有表
.table 

# 打开表头
.header on

# 查看所有表结构
select * from sqlite_master where type="table";

# 查看某张表结构
select * from sqlite_master where type="table" and name="emperors";

# 查看某张表结构也可以这样
.schema emperors 

```

# 时间相关
注意sqlite里面没有时间数据格式，只有字符串，处理时间都是通过函数在处理字符串
```
# 获取的是格林威治时间
select datetime();
select CURRENT_TIMESTAMP;

# 这个一般常用，获取的是当前本地时间
select datetime(CURRENT_TIMESTAMP,'localtime');
select datetime(datetime(),'localtime');

# 将字符串时间转换成时间对象datetime
select datetime('2019-07-28 10:00:00');

# 获取时间差（默认单位为天）
select Cast ((JulianDay('now','localtime') - JulianDay('2019-07-28 10:00:00') ) as double );
# 当然也可以换成小时和分钟
select Cast ((JulianDay('now','localtime') - JulianDay('2019-07-28 10:00:00') ) * 24  as double );
# 换成分钟
select Cast ((JulianDay('now','localtime') - JulianDay('2019-07-28 10:00:00') ) * 24 * 60  as INTEGER );

# 格式化时间字符串
select strftime('%Y年%m月%d日%H点%M分%S秒', datetime('2019-07-28 10:00:00'))

# 时间在原来的基础上加减
SELECT datetime('2020-06-08 00:00:00', '+1 day') as plus_one_day; -- 当然加2天就是+2 day
SELECT datetime('2020-06-08 00:00:00', '-1 day') as minus_one_day; -- 当然减2天就是-2 day
SELECT datetime('2020-06-08 00:00:00', '+1 year') as plus_one_year; -- 当然加2年就是+2 year，减2年就是-2 year 
SELECT datetime('2020-06-08 00:00:00', '+1 month') as plus_one_month; -- 当然加2月就是+2 month，减2月就是-2 month

SELECT datetime('2020-06-08 00:00:00', '+1 hour') as plus_one_hour; -- 当然加2小时就是+2 hour，减2小时就是-2 hour
SELECT datetime('2020-06-08 00:00:00', '+1 minute') as plus_one_minute; -- 当然加2分钟就是+2 minute，减2分钟就是-2 minute 
SELECT datetime('2020-06-08 00:00:00', '+1 second') as plus_one_second; -- 当然加2秒就是+2 second，减2秒就是-2 second 

 
```

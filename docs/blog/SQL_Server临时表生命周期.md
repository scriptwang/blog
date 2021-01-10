# #tmp 与 ##tmp
SQL Server可这样创建临时表
```
SELECT * INTO #tmp FROM table1; 
```
也可以这样创建临时表
```
SELECT * INTO ##tmp FROM table1; 
```
这两者的区别在于生命周期的不同：

```#tmp```的生命周期在于当前连接，连接关闭则该表自动被删除

```##tmp```的生命周期在于所有连接，在当前连接和其他访问过它的连接都断开时失效

所以，在java中，如果当需要多个连接访问临时表时应该用```##tmp```而非```#tmp```！

# 临时表是否需要显示删除
不用临时表的时候最好显示删除掉，因为关闭连接/连接池的时候SQL Server才帮我们清除临时表了，但事实上连接池只有当程序重启的时候才清掉。

不显示清楚掉则可能导致tempdb占用硬盘空间非常大！

# 参考
- [SQL Server中的临时表是否需要显式删除？](https://blog.csdn.net/yenange/article/details/78685231)
- [sql server临时表的生命周期](https://blog.csdn.net/wd1624348916/article/details/52566174)



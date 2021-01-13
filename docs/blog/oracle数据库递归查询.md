# Oracle数据库递归查询
最近在做一个树状编码管理系统，其中用到了oracle的树状递归查询（关键字：SELECT ... WHERE... START WITH ... CONNECT BY PRIOR）
以后开发树状菜单、树状评论、树状文件结构，只要和树扯得上关系的都可以应用，前提是你用的是oracle数据库

#### 数据准备
```sql
-- 建表
CREATE TABLE TB (
	ID NUMBER(10) NOT NULL, --主键
	PID NUMBER(10) , --父id
	NAME VARCHAR(128) --名称
)

-- 插数据
-- 一级节点
INSERT INTO TB VALUES (1,0,'查询');
INSERT INTO TB VALUES (2,0,'咨询');
INSERT INTO TB VALUES (3,0,'办理');

-- 二级节点
INSERT INTO TB VALUES (4,1,'余额查询');
INSERT INTO TB VALUES (5,1,'话费查询');
INSERT INTO TB VALUES (6,1,'城市查询');
INSERT INTO TB VALUES (7,1,'租房查询');
INSERT INTO TB VALUES (8,1,'公交查询');
INSERT INTO TB VALUES (9,1,'地铁查询');
INSERT INTO TB VALUES (10,2,'疑问咨询');
INSERT INTO TB VALUES (11,2,'报障咨询');
INSERT INTO TB VALUES (12,2,'话费咨询');
INSERT INTO TB VALUES (13,2,'余额咨询');
INSERT INTO TB VALUES (14,2,'活动咨询');
INSERT INTO TB VALUES (15,3,'公交办理');
INSERT INTO TB VALUES (16,3,'地铁办理');
INSERT INTO TB VALUES (17,3,'银行办理');

-- 三级节点
INSERT INTO TB VALUES (18,7,'一手房东查询');
INSERT INTO TB VALUES (19,7,'二手房东查询');
INSERT INTO TB VALUES (20,7,'三手房东查询');
INSERT INTO TB VALUES (21,7,'中介房东查询');
```
以上数据PID为0的表示根节点，根节点可以有多个，根节点的PID最好不要用NULL，此时会引起全表扫描。
按照层级关系展示如下：
```sql
ID |父ID|层级|名称               |
---|----|----|-------------------|
1  |0   |1   |--查询             |
4  |1   |2   |----余额查询       |
5  |1   |2   |----话费查询       |
6  |1   |2   |----城市查询       |
7  |1   |2   |----租房查询       |
18 |7   |3   |------一手房东查询 |
19 |7   |3   |------二手房东查询 |
20 |7   |3   |------三手房东查询 |
21 |7   |3   |------中介房东查询 |
8  |1   |2   |----公交查询       |
9  |1   |2   |----地铁查询       |
2  |0   |1   |--咨询             |
10 |2   |2   |----疑问咨询       |
11 |2   |2   |----报障咨询       |
12 |2   |2   |----话费咨询       |
13 |2   |2   |----余额咨询       |
14 |2   |2   |----活动咨询       |
3  |0   |1   |--办理             |
15 |3   |2   |----公交办理       |
16 |3   |2   |----地铁办理       |
17 |3   |2   |----银行办理       |
```

#### 查找根节点
```sql
SELECT * FROM TB WHERE PID = 0;

ID |PID |NAME |
---|----|-----|
3  |0   |办理 |
1  |0   |查询 |
2  |0   |咨询 |
```

#### 查找某节点一级子节点
查询ID为1的节点的儿子
```sql
SELECT * FROM TB WHERE PID = 1;

ID |PID |NAME     |
---|----|---------|
4  |1   |余额查询 |
5  |1   |话费查询 |
6  |1   |城市查询 |
7  |1   |租房查询 |
8  |1   |公交查询 |
9  |1   |地铁查询 |
```

#### 查询某节点直系父节点
```sql
SELECT * FROM TB C,TB P WHERE C.PID=P.ID AND C.ID=20;

ID |PID |NAME         |ID |PID |NAME     |
---|----|-------------|---|----|---------|
20 |7   |三手房东查询 |7  |1   |租房查询 |
```

#### 查询某节点所有兄弟节点
查询ID为6的节点的所有亲兄弟节点
```sql
SELECT * FROM TB A WHERE EXISTS (SELECT ID FROM TB B WHERE A.PID=B.PID AND B.ID=6);

ID |PID |NAME     |
---|----|---------|
9  |1   |地铁查询 |
8  |1   |公交查询 |
7  |1   |租房查询 |
6  |1   |城市查询 |
5  |1   |话费查询 |
4  |1   |余额查询 |
```


#### 查找某节点所有子节点（自顶向下的树状）
从ID为1的节点开始，查询所有属于它的子节点，包括儿子，儿子的儿子，儿子的儿子的儿子，儿子的儿子的儿子....无限个儿子
```sql
SELECT * FROM TB START WITH ID = 1 CONNECT BY PRIOR ID=PID;

ID |PID |NAME        |
---|----|------------|
1  |0   |查询        |
4  |1   |余额查询    |
5  |1   |话费查询    |
6  |1   |城市查询    |
7  |1   |租房查询    |
18 |7   |一手房东查询| 
19 |7   |二手房东查询| 
20 |7   |三手房东查询| 
21 |7   |中介房东查询| 
8  |1   |公交查询    |
9  |1   |地铁查询    |
```

当然，你也可以加WHERE条件，不要名称中含有房东的儿子节点
```sql
SELECT * FROM TB WHERE NAME NOT LIKE '%房东%' START WITH ID = 1 CONNECT BY PRIOR ID=PID;

ID |PID |NAME     |
---|----|---------|
1  |0   |查询     |
4  |1   |余额查询 |
5  |1   |话费查询 |
6  |1   |城市查询 |
7  |1   |租房查询 |
8  |1   |公交查询 |
9  |1   |地铁查询 |
```

甚至可以指定多个根节点
```sql
SELECT * FROM TB START WITH ID IN (1,3) CONNECT BY PRIOR ID=PID;

ID |PID |NAME         |
---|----|-------------|
1  |0   |查询         |
4  |1   |余额查询     |
5  |1   |话费查询     |
6  |1   |城市查询     |
7  |1   |租房查询     |
18 |7   |一手房东查询 |
19 |7   |二手房东查询 |
20 |7   |三手房东查询 |
21 |7   |中介房东查询 |
8  |1   |公交查询     |
9  |1   |地铁查询     |
3  |0   |办理         |
15 |3   |公交办理     |
16 |3   |地铁办理     |
17 |3   |银行办理     |
```

#### 查找某节点所有父节点（自下向上的树状）
这个和上面```查找某节点所有子节点（自顶向下的树状）```的唯一区别就是```PID```和```ID```的位置交换了，上面能用的WHERE和IN这里也能用，不再赘述
```sql
SELECT * FROM TB START WITH ID = 18 CONNECT BY PRIOR PID=ID;

ID |PID |NAME         |
---|----|-------------|
18 |7   |一手房东查询 |
7  |1   |租房查询     |
1  |0   |查询         |
```

#### 查询同一层级的所有节点
不管节点是属于哪个根节点的，只要在同一层级都可以查询出来，和```查询某节点所有兄弟节点```的不同之处在于，前者是查询亲兄弟，后者是所有兄弟，不管是亲兄弟，堂兄弟，表兄弟
用临时表保存层级信息（LEAF），然后从临时表中查询传入ID的层级，最后查询所有在同一层级的节点
```sql
WITH TMP AS( SELECT A.*, LEVEL LEAF 
	FROM TB A START WITH A.PID = 0 CONNECT BY PRIOR A.ID = A.PID) 
SELECT	* FROM TMP 
	WHERE LEAF = ( SELECT LEAF FROM TMP WHERE ID = 7);
	
ID |PID |NAME     |LEAF |
---|----|---------|-----|
4  |1   |余额查询 |2    |
5  |1   |话费查询 |2    |
6  |1   |城市查询 |2    |
7  |1   |租房查询 |2    |
8  |1   |公交查询 |2    |
9  |1   |地铁查询 |2    |
10 |2   |疑问咨询 |2    |
11 |2   |报障咨询 |2    |
12 |2   |话费咨询 |2    |
13 |2   |余额咨询 |2    |
14 |2   |活动咨询 |2    |
15 |3   |公交办理 |2    |
16 |3   |地铁办理 |2    |
17 |3   |银行办理 |2    |
```
可以看到，上面的所有节点都是2层级，本文最开始的层级关系展示图可以用下面的SQL查询出来，LPAD函数可适当调整
```sql
WITH TMP AS( SELECT A.*, LEVEL LEAF 
	FROM TB A START WITH A.PID = 0 CONNECT BY PRIOR A.ID = A.PID) 
SELECT ID,PID AS 父ID,LEAF AS 层级,LPAD(NAME,LEAF * 6,'-') AS 名称
 	FROM TMP
```
#### 查询每个节点所属的分类（正向递归嵌套反向递归）
此查询在对每个节点进行分类和其他表联查的时候比较有用
```sql
SELECT A.ID,A.NAME,
-- 在正向递归的基础上反向（自下向上）递归，找到每个子节点PID为0的那个父节点（大分类）的ID
(SELECT B.ID   FROM TB B WHERE B.PID = 0 START WITH B.ID = A.ID CONNECT BY PRIOR B.PID = B.ID) AS RID,
-- 在正向递归的基础上反向（自下向上）递归，找到每个子节点PID为0的那个父节点（大分类）的名称 
(SELECT B.NAME FROM TB B WHERE B.PID = 0 START WITH B.ID = A.ID CONNECT BY PRIOR B.PID = B.ID) AS RNAME
-- 正向（自顶向下）递归，A.PID != 0排除了最外层的大类分类
FROM TB A WHERE A.PID != 0 START WITH A.PID = 0 CONNECT BY PRIOR A.ID = A.PID
ID |NAME         |RID |RNAME|
---|-------------|----|-----|
4  |余额查询     |1   |查询 |
5  |话费查询     |1   |查询 |
6  |城市查询     |1   |查询 |
7  |租房查询     |1   |查询 |
18 |一手房东查询 |1   |查询 |
19 |二手房东查询 |1   |查询 |
20 |三手房东查询 |1   |查询 |
21 |中介房东查询 |1   |查询 |
8  |公交查询     |1   |查询 |
9  |地铁查询     |1   |查询 |
10 |疑问咨询     |2   |咨询 |
11 |报障咨询     |2   |咨询 |
12 |话费咨询     |2   |咨询 |
13 |余额咨询     |2   |咨询 |
14 |活动咨询     |2   |咨询 |
15 |公交办理     |3   |办理 |
16 |地铁办理     |3   |办理 |
17 |银行办理     |3   |办理 |

```

#### 其他查询
- 自顶向下路径查询

```sql
SELECT SYS_CONNECT_BY_PATH (NAME, '/') AS PATH FROM TB 
	WHERE ID = 18 START WITH PID = 0 CONNECT BY PRIOR ID=PID;
PATH                        |
----------------------------|
/查询/租房查询/一手房东查询 |
```


- 自下向上路径查询
注意和自顶向下的在效率上的区别，引用别人的一句话
>在这里我又不得不放个牢骚了。oracle只提供了一个sys_connect_by_path函数，却忘了字符串的连接的顺序。在上面的例子中，第一个sql是从根节点开始遍历，而第二个sql是直接找到当前节点，从效率上来说已经是千差万别，更关键的是第一个sql只能选择一个节点，而第二个sql却是遍历出了一颗树来。再次ps一下。https://www.cnblogs.com/linjiqin/p/3152674.html

```sql
SELECT SYS_CONNECT_BY_PATH (NAME, '/') AS PATH FROM TB
	START WITH ID = 18 CONNECT BY PRIOR PID=ID;

PATH                        |
----------------------------|
/一手房东查询               |
/一手房东查询/租房查询      |
/一手房东查询/租房查询/查询 |
```

- 查询树状始终显示根节点

```sql
SELECT CONNECT_BY_ROOT NAME,TB.* FROM TB START WITH ID = 7 CONNECT BY PRIOR ID = PID;

CONNECT_BY_ROOTNAME     |ID |PID |NAME         |
------------------------|---|----|-------------|
租房查询                |7  |1   |租房查询     |
租房查询                |18 |7   |一手房东查询 |
租房查询                |19 |7   |二手房东查询 |
租房查询                |20 |7   |三手房东查询 |
租房查询                |21 |7   |中介房东查询 |

```

- 动态查询是否是叶子节点，是叶子节点表示该节点没有儿子了，否则有儿子，ORACLE自带的CONNECT_BY_ISLEAF能动态显示是否叶子节点，1是0否
  
```sql
SELECT CONNECT_BY_ISLEAF AS IS_LEAF,TB.* 
	FROM TB START WITH ID = 1 CONNECT BY PRIOR ID = PID;
	
IS_LEAF |ID |PID |NAME         |
--------|---|----|-------------|
0       |1  |0   |查询         |
1       |4  |1   |余额查询     |
1       |5  |1   |话费查询     |
1       |6  |1   |城市查询     |
0       |7  |1   |租房查询     |
1       |18 |7   |一手房东查询 |
1       |19 |7   |二手房东查询 |
1       |20 |7   |三手房东查询 |
1       |21 |7   |中介房东查询 |
1       |8  |1   |公交查询     |
1       |9  |1   |地铁查询     |
```

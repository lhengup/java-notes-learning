## sql中查找两个表的并集，差集和交集

### union形成并集

UNION可以对两个或多个结果集进行连接，形成“并集”。
　　子结果集所有的记录组合在一起形成新的结果集。

1、限定条件
　　要是用UNION来连接结果集，有4个限定条件。
　　(1)、子结果集要具有相同的结构。
　　(2)、字结果集的列数必须相同。
　　(3)、子结果集对应的数据类型必须可以兼容。
　　(4)、每个子结果集不能包含order by和compute子句。

2、语法形式
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;

### except形成差集

EXCEPT可以对两个或多个结果集进行连接，形成“差集”。
　　返回左边结果集合中已经有的记录，而右边结果集中没有的记录。

1、限定条件
　　要是用EXCEPT来连接结果集，有4个限定条件。
　　(1)、子结果集要具有相同的结构。
　　(2)、字结果集的列数必须相同。
　　(3)、子结果集对应的数据类型必须可以兼容。
　　(4)、每个子结果集不能包含order by和compute子句。

2、语法形式
SELECT column_name(s) FROM table1
EXCEPT
SELECT column_name(s) FROM table2;

### inner join形成交集

INNER JOIN可以对两个或多个结果集进行连接，形成“交集”。
　　返回左边结果集和右边结果集中都有的记录。

1、限定条件
　　要是用INNER JOIN来连接结果集，有4个限定条件。
　　(1)、子结果集要具有相同的结构。
　　(2)、字结果集的列数必须相同。
　　(3)、子结果集对应的数据类型必须可以兼容。
　　(4)、每个子结果集不能包含order by和compute子句。

2、语法形式
SELECT column_name(s) FROM table1
INNER JOIN table2
ON table2.column_name(s) = table1.column_name(s);

或

SELECT column_name(s) FROM table1
JOIN table2
ON table2.column_name(s) = table1.column_name(s);





## sql中的内连接（等值连接），外连接（左连接，右连接，全外连接）

基本定义：
　　left join （左连接）：返回包括左表中的所有记录和右表中连接字段相等的记录。
　　right join （右连接）：返回包括右表中的所有记录和左表中连接字段相等的记录。
　　inner join （等值连接或者叫内连接）：只返回两个表中连接字段相等的行。
　　full join （全外连接）：返回左右表中所有的记录和左右表中连接字段相等的记录。

>  注：在sql中l外连接包括左连接（left join ）和右连接（right join），全外连接（full join），等值连接（inner join）又叫内连接。





## 数据库范式

**·第一范式（1NF）：属性不可分。**

**第二范式（2NF）：符合1NF，并且，非主属性完全依赖于码。**

**第三范式（3NF）：符合2NF，并且，消除传递依赖**

**BC范式（BCNF）：符合3NF，并且，主属性不依赖于主属性**

> **四种范式之间存在如下关系：
>
> 这里主要区别3NF和BCNF，一句话就是**3NF是要满足不存在非主属性对候选码的传递函数依赖，BCNF是要满足不存在任一属性（包含非主属性和主属性）对候选码的传递函数依赖**。
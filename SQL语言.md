区别DDL、DML所完成功能的差异
掌握SQL的各种数据定义语句（表及完整性、视图）
SQL语言表达查询
查询语句中各子句执行的逻辑顺序
ORDER BY子句相关知识（多个属性、空值、升序降序等）
去重复
空值处理（is null，除此以外不满足任何匹配条件）
模糊查询（通配符的使用:%和_）  
掌握查询的基本结构、分组聚集运算、分组的选择、不相关子查询（IN或NOT IN）等 
带存在量词的相关子查询不考查
SQL数据操纵语言
理解视图的本质和特点
命名的查询语句，虚拟关系；动态变化；行列子集视图（更新限制）  
视图的优点

# SQL语言分类

|分类|功能|操作语句|
|--|--|--|
|数据定义语言DDL|提供定义/修改/删除域/关系/视图/索引/数据库的命令|CREATE，ALTER，DROP|
|数据操纵语言DML|提供对数据库增删改查的能力|SELCT，INSERT，UPDATE，DELETE|
|数据控制语言DCL|提供管理权限的能力|GRANT，REVOKE，...|

# DDL数据定义语言

**SQL提供几种对象的定义**
* 域(数据类型)【内置的有char(n)，varchar(n)，int，float，date，time，...】
* 关系
* 视图
* 索引
* 数据库

||创建|修改|删除|
|--|--|--|--|
|域|Create Domain||Drop Domain|
|关系|Create Table|Alter Table|Drop Table|
|视图|Create View||Drop View|
|索引|Create Index||Drop Index|
|数据库|Create Database||Drop Database|

**举例：定义关系**
```sql
create  table   SC (   　
    SNO  VARCHAR(4) not null, 
　　CNO  VARCHAR(4) not null,
　　RESULT　INT,
　　primary  key  (SNO, CNO),
　　foreign  key  (SNO)  references  S (SNO),
　　foreign  key  (CNO)  references  C (CNO),
　　check  ( (RESULT  IS  NULL) OR (RESULT  BETWEEN  0  AND  100) )
　)
```

**举例：修改关系**
```sql
alter table S (add DEPT VARCHAR(20) not null)
alter table SC (modify RESULT SMALLINT)
```

# DML数据操作语言

## 查询

### 简单查询

查询操作最核心的就是select，from，where

**对应于关系代数**
select相当于投影运算（严谨的说，应该是select distinct等价于投影），from相当于笛卡尔积运算，where相当于选择运算
* **语义次序**：From (笛卡儿积) → Where (选择) → Select (投影)
* 可以通过关系代数，按照对应关系转换成sql语句

***

**select子句**
* **select默认保留重复元组，即select all**。如果想要对结果去重，需要在select后加`distinct`关键字。如 `select distinct A,B,C`，表示最终结果是对ABC组合进行去重操作后的结果。
* 属性列表中，可以**用表达式来构造的新属性**，相当于广义投影。比如 `select 2020-出生日期`
* **用`as`重命名**。比如`select 姓名 as 学生姓名, 2023-出生日期 as 年龄`
* **同名属性必须加关系名前缀**。比如`select R.姓名, S.姓名, S.性别 From R, S`

***

**where子句**

**条件中可能出现的运算符**
* **比较运算符**：<，>，=，!=（也写作<>），<=，>=
* **逻辑运算符**：and，or，not
* **范围运算**(下界<上界)
  * between 下界 and 上界
  * not between 下界 and 上界
* **集合运算**：
  * in 集合；
  * not in 集合
* **匹配运算**：like 匹配串；not like 匹配串
  * 百分号%：匹配任一子串
  * 下划线_：匹配任意一个字符
\是转义字符
例1：姓名 like ‘陈%’  【匹配：以“陈”开头的任意字符串】
例2：姓名 like ‘陈_’  【匹配：以“陈”开头，之后只有一个字符，且任意】
例3：内容 like ‘%100\%正确%’   【匹配：`任意字符串`100%正确`任意字符串`】

**特别注意：因为选择运算(Where子句)在投影运算(Select子句)之前。所以Where子句中不能使用Select子句中更名后的新属性名**

错误的
```sql
Select  姓名, 2020－年龄 as 出生年份　
From    S　
Where  出生年份<1987
```

正确的
```sql
Select  姓名, 2020－年龄 as 出生年份　
From    S　
Where  2020－年龄<1987
```

***

**from子句**

**from子句中关系的重命名**：旧关系名 [as] 新关系名
等价于更名运算。一个关系在From子句中多次出现时，从第二次开始必须重命名。关系重命名后，**Select和Where子句中的前缀是使用新的关系名**

比如
```sql
Select S.姓名　
From R, R  as  S
Where R.课程=“物理” and R.姓名=“王红” 
and S.课程=“物理” and R.成绩 < S.成绩
```

### 集合运算

提前声明
>虽然SQL标准中支持Intersect和Except，但并非所有DBMS产品都提供这两种运算符。对于没有提供的DBMS，若要实现差运算或交运算，应使用exist/in和not exist/not in形式的嵌套子查询

多个Select查询语句的结果可用集合运算进行交/并/差，**语法如下**

```sql
SQL查询1
Union/Intersect/Except [all]
SQL查询2
Union/Intersect/Except [all]
SQL查询3
……  
```

* Union: 并运算;  Intersect: 交运算; Except: 差运算。
* 参与集合运算的表的**表结构要相同**
* select不会自动去重，但**集合运算会自动去重**。如果不想去重，需要加上参数all：`union all`，`intersect all`，`except all`
  
**重复元组的处理举例**：两个查询A和B，如果一个元组在A中出现a次，在B中出现b次，那么这个元组：

```sql
A union B           中出现1次
A intersect B       中出现1次
A except B          中出现1次(a>b)或0次(a<=b)
A union all B       中出现a+b次
A intersect all B   中出现Min(a, b)次
A except all B      中出现a－b次(a>b)或0次(a<=b)
```

### 聚集函数

涉及到的子句：聚集函数，group by子句，having子句，order by子句

总览
```sql
Select A1 , A2 , … , An　　　　　　
From R1 , R2 , … , Rm
[Where 元组限定条件P]
[Group By 属性1, 属性2, … 　　
[Having 分组限定条件Q]]　
[Order By 属性1 [asc｜desc], 属性2 [asc｜desc], … ]
```

**语义次序**：From (笛卡儿积) → [ Where (选择) ] → [ Group By (分组) ] → [ Having (限定分组) ] → [ Select (投影, 或统计) ] → [ Order By (结果排序) ] 

**Where子句**中的条件用于**限定元组**，施加在单个元组上，不满足条件的元组将不会参与到下一步的分组运算(Group By子句)或投影运算(Select 子句)等

**Having子句**中的条件用于**限定Group By子句产生的各个分组**，施加在整个分组上不满足条件的分组，将不会参与下一步的统计运算(Select子句)

***

**group by 子句**

Group By子句的作用：对Where子句筛选出的元组，按指定的分组属性进行分组【**分组属性相等的元组分为一组**】
同时Select 子句的作用发生变化：对分组进行统计，每个分组在结果中被统计为一个元组

**格式**
```sql
Select A1 , A2 , … , An　　　　　
From R1 , R2 , … , Rm　　　　　　　    
[Where P]
Group By 属性1, 属性2, … 　　
```

**select子句出现的属性只能是**：
① group by 中出现的属性，即分组属性
② 聚集函数
③ 由①和②组成的表达式

**举例**
```sql
//查询每门课程的平均和最高成绩
Select 课程, Avg(成绩) as 平均成绩, Max(成绩) as 最高成绩　
From R
Group By 课程 
```

***

**having子句**

Having子句**只能配合Group By子句**使用，而**不能单独出现**
Having子句作用：在分组后，筛选满足条件Q的分组。**不满足having条件的gruop by分组将被丢弃**

**格式**(出现在group by子句后面)
```sql
Select A1 , A2 , … , An　　　　　　　
From R1,R2,…,Rm　　　　　　　   
Where P (元组限定条件)
Group By 属性1, 属性2, … 　　
Having Q (分组限定条件) 　　
```

**having子句中出现的属性只能是**：
① group by 中出现的属性，即分组属性
② 聚集函数(任意属性)

**举例**

```sql
//查询平均成绩大于85的学生姓名
Select 姓名
From R
Group By 姓名 
Having Avg(成绩) > 85
```

***

**order by 子句**

Order By子句的作用：在Select子句得出结果后，**对结果进行排序**
先按属性1的值，升序(`asc`)或降序(`desc`)排列，**缺省是升序`asc`**；属性1的值相同时，再按属性2值升序或降序排列…

**举例**
```sql
//从高到低列出物理课程的成绩，再按如果同分，按学号升序排序
Select 学号, 姓名, 成绩
From S
Where 课程 = “物理” 
Order By 成绩 desc, 学号 asc
```

### 空值处理

空值null代表值未知/值不存在

**判断空值**： `A is [not] null`，不能用=或!=（结果为unknown）

**空值参与运算**
* **null参与算数运算**：结果都是null
* **null参与比较运算**：结果是unknown
* **null参与布尔运算，会被当成unknown，优先级如下**
  * and：false > unknown > true
  * or：true > unknown > false
  * not：not unknown 结果还是unknown
* **null参与聚集运算**：除了count(*)之外，其余聚集函数都忽略null


### 连接关系

From子句中的两个相邻关系之间
* 逗号分隔 —— 表示做笛卡尔积; 
* 连接操作 —— 表示将它们连接成一个新的关系

连接操作由连接类型和连接条件组成

**连接类型**
* `inner join` : 内连接。结果不包含失配元组(失配元组指的是因不满足连接条件，无法和其它元组相连接的元组)
* `left (outer) join`: 左外连接。结果包含左边关系的失配元组
* `right (outer) join`: 右外连接。结果包含右边关系的失配元组
* `full (outer) join`: 全外连接。结果包含两边关系的失配元组
* `cross join`: 笛卡儿积 例子
* `union join` 

**连接条件**
* `natural`: 连接两边元组条件为同名属性相等（自然连接）
* `using (属性1, 属性2, …)`: 类似自然连接，但是只限于列出的的属性相等
* `on 条件`: 按照指定的条件连接两边元组

**例子（就是看用法，意思不管）**
```sql
Select Emp.姓名, Dept.名称 as 部门
From Emp inner join Dept on Emp.部门号＝Dept.部门号

Select Emp.姓名, Dept.名称 as 部门
From Emp inner join Dept using (部门号)

Select Emp.姓名, Dept.名称 as 部门
From Emp natural inner join Dept

Select Emp.姓名, Dept.名称 as 部门
From Emp left outer join Dept on Emp.部门号=Dept.部门号

Select Emp.姓名, Dept.名称 as 部门
From Emp natural right outer join Dept

Select Emp.姓名, Dept.名称 as 部门
From Emp full outer join Dept using (部门号)
```

### 嵌套子查询

#### where中嵌套子查询

where子句中用子查询构造条件

***
用法一：**检查集合成员资格**

*集合差except不支持的话，就可以用not in代替（用法三的not exists也可以代替）*

**格式**：`字段 in/ not in 集合`
```sql
A [not] in (子查询)
```

**要求**
A可以是一个属性，也可以是一个属性集，但必须和子查询结果集的属性集一致

**结果**
当A是子查询结果中的一个值时，结果为真。否则为假

**举例**: 查询没有选修数学的学生姓名
```sql
Select 姓名
From R
Where 姓名 not in (Select 姓名 From RWhere 课程=“数学” ) 
```

***

用法二：**集合比较**

**格式**：`字段 比较运算符 som/all/any 集合`
```sql
A (比较运算, >, <, =,…)  some|all (子查询)
```

**要求**
A可以是一个属性，也可以是一个属性集，但必须和子查询结果集的属性集一致

**结果**
* some：有一个满足就为true
* all：全部满足才为true

**特别说明**
* `=some`等价于`in`，`<>some`不等价于`not in`
* `<>all`等价于`not in`，`=all`不等价于`in`

**举例**：查询物理课哪个学生的成绩最高
```sql
Select 姓名, 成绩 as 最高物理成绩
From R
Where 课程＝“物理” and 成绩 >= all (Select 成绩 From R Where 课程=“物理”) 
```

***

用法三：**判断查询结果是否为空**

**格式**
```sql
[not] exists (子查询)
```

**要点**
* 子查询可包含多个属性(因为它不必和某个值做比较)
* 允许**相关子查询**，可引用外层查询关系的属性(最好标明关系前缀)。这一点对于in、some、all操作来说也是适用的

**结果**
当子查询的结果不为空（包含任何记录）时，exists的结果为真。否则为假

**举例**：查询哪个学生没有选修任何课程，列出姓名和年龄
```sql
//使用not in
Select 姓名, 年龄
From S
Where 姓名 not in (Select 姓名 From R) 

//转换成使用not exists
Select 姓名, 年龄
From S
Where not exists (Select * From R Where R.姓名=S.姓名) 
```

***

#### from中嵌套子查询

在From子句中，允许用**子查询构造新的关系**，称为派生关系。新**关系必须命名**，其属性也可以重命名

**派生关系的作用**：允许用户在上一个查询的基础上，进一步查询，从而“一步步”地完成复杂的查询。

**格式**
```sql
From … , (SQL 子查询) as 关系名… , 
From … , (SQL子查询) as 关系名(属性1, 属性2…) , 
```

**举例**：查询每门课程的最高成绩和相应学生姓名
```sql
Select 课程, 姓名, 最高成绩 　
From R, (Select  课程, Max(成绩) as 最高成绩 From  R Group By 课程 ) as S
Where R.成绩=S.最高成绩
and R.课程=S.课程
```

## 增删改

***
**insert子句插入**

* 如果没有列出插入的属性，默认values()里要一一对应表中所有的列
* 所列值的个数必须和属性的个数相等，且一一对应
* 对没有指定的属性填入缺省值（Create Table时定义），没有缺省值时填入空值

**插入一个元组**
```sql
Insert Into 关系 [(属性1, 属性2, …)] Values (值1, 值2 , … )
```

**插入多个元组（1个sql查询的结果）**
```sql
/*查询结果的各个属性赋值给前面的属性，
所以要一一对应：类型相容，属性名可不同*/
Insert Into 关系 [(属性1, 属性2, …)] Select ……
```

举例：添加所有学生选修数学课程的信息
```sql
Insert Into R(姓名, 课程) 
Select  姓名，课程  From S, C Where 课程=“数学”
```

***
**delete子句删除**

在关系中找到满足条件的元组，并删除之
如果没有Where子句，表示删除关系的全部元组(保留结构)
一次只能删除一个关系中的元组

**格式**
```sql
Delete From 关系 [Where  条件]
```

举例：删除平均分不及格的学生的选修信息
```sql
Delete From R Where 姓名 in
(select 姓名 from R 
group by 姓名 
having avg(成绩) < 60)
```

***
**update子句更新**

在关系中找到满足条件的元组，然后更新：表达式1的值赋予属性1；表达式2的值赋予属性2……
没有Where子句时，则对关系的全部元组都要更新

**格式**
```sql
Update 关系 Set 属性1 = 表达式1
[,属性2 = 表达式2]
……
[Where  条件]
```

举例：在原有的学生关系S里面增加"选修课程数"属性，然后填充正确的数值。

```sql
Update S set 选修课程数 = 
(select count(课程) from R where R.姓名=S.姓名 )
```

# 视图

## 视图的概念

**视图的本质**：有名字的查询，而非一个表
一个视图总是对应一个select查询，且有唯一的名字（视图名）
查询中使用到的关系，称为基础关系。视图就是定义在基础关系上的。

视图的表象：**“虚拟”关系**
用户访问视图时，看到的是一个`基础关系+查询`所派生 (计算) 出来的“虚拟”关系
* 关系（的元组）是实际存在的，即是存储在数据库中的
* 视图（的元组）并不实际存在，而是通过查询“算”出来。每一次访问视图，都要通过查询“算”一次，以获得最新的内容。【数据库存的是视图的定义，而非其执行结果】
* 在增删改查操作中都可以使用视图，就像使用一个真正的关系一样。

**注意**
* 当基础关系发生变化后，再去访问视图，看到的虚拟关系也会发生相应的变化。
* 用户**对视图的查询**，系统在执行时必须**转化为对基础关系的查询**。
* 用户**对视图的修改**，系统在执行时必须**转化为对基础关系的修改**。

## 视图优点

**简化用户的操作**
当经常进行某一复杂查询时，可将其固定的部分定义为一个视图，再用这个视图来重写查询，从而得到简化

**对于同一数据，不同用户可以从不同角度观察**
一个关系中，不同的用户可能关注不同的部分(不同的行/列)，我们把这些部分定义成不同的视图，分配给不同用户。
表面上看，这些视图是内容不一样的“关系”。实际上，它们源自相同的基础关系，也不占用实际的存储空间。

**增强安全性**
“知必所需”，用视图来限制用户数据的访问范围。
例如，在为用户定义视图时，通过选择和投影操作限制用户只能访问数据库关系中的某些行和某些列

**提供模式（逻辑模式）/外模式（子模式）映射和逻辑独立性**
关系属于模式（逻辑模式），视图属于外模式（子模式）。
在关系上定义视图相当于定义了模式/外模式映射。用户/应用程序看到的应该是子模式（视图）
当关系发生变化，例如增加了新字段时，视图不一定变化。也就是说，逻辑模式发生变化时，子模式不一定变化；子模式不变化时，用户程序也不变化。这就提供了数据逻辑独立性

## 定义视图

**创建视图**
```sql
create view 视图名 [( 属性名 {, 属性名} )] as (查询语句) [with check option]
```
* 视图的属性可以显式命名，也可缺省取查询结果中的属性名
* `with check option`: 当对视图进行插入，更新时，要检查新元组是否满足视图对应查询的条件。不满足则无法超过操作。
* 定义视图时，可以在相应查询中使用其它视图

举例
```sql
//对视图Class1插入2班的学生是不能成功的
create view Class1(sno, name, sex, class) as 
(Select 学号, 姓名, 性别, 班级 From S Where 班级 = "1班") 
with check option

//定义视图时，可以在相应查询中使用其它视图
Create View Class1_male as   
Select 学号, 姓名 From Class1 Where 性别=‘男’
```

**删除视图**
```sql
drop view 视图名
```

## 查询视图

对视图的查询，语法上和对关系的查询是一样的，但是系统在查询时作的工作不一样

涉及视图的查询，系统必须 “展开” 视图，既用对应的定义（查询语句）代替视图本身，如果仍有视图，则继续展开，直到查询语句中没有视图。

## 修改视图

对视图的增删改查和基础关系相同。即在Insert、Delete、Update语句中，直接使用视图（代替关系）即可。

但是**视图的修改是受限的**：视图是一个不存在的“虚拟”关系，对视图的修改要转化为对基础关系的修改。这种转化不一定成功，导致并非所有视图都能被修改。

**修改失败的情况**
* **情况一：信息缺失**：如果**视图定义不包括基础关系的主码**，则在对视图的修改转化为对基础关系的修改时，往往会因为缺乏主码值而无法完成。

* **情况二：信息歧义**
  * 当一个**视图是来自对多个基础关系的查询**时，无法决定将对视图的修改转化为对哪个基础关系的修改
  * 当视图的一个元组是“来自”关系里面的多个元组时【比如count(*)】，同样修改无法转化

**无法修改的情况总结为**
>（查询的）Select 子句不包括主码
Select 子句中用聚集函数或算术表达式构造的新属性
Select 子句存在distinct关键字
From 子句中有多个关系
有Group By子句

**同时满足以下条件，视图才是可修改的**
* Select 子句中包含主码
* Select 子句中只出现原有属性，而没有用聚集函数或算术表达式构造的新属性
* Select 子句中无distinct关键字
* From 子句中只有一个关系
* 无Group By子句

## 物化视图

**物化视图**
某些数据库系统允许将某一时刻视图的内容（对应查询的结果关系）保存起来，得到一个“快照” ，称为物化视图。

**创建物化视图**
```sql
create materialized view 物化视图名 as (查询语句)
```

**一般视图 vs 物化视图**
`一般视图`是访问的时候才用查询算出来，是一个“虚拟”的关系。**并不真正地占用存储空间**。
* 优点：我们看到的**视图永远是“最新”**的，反映基础关系的最近变化。因此不**用特意考虑视图与基础关系间的“同步”**
* 缺点：每次使用视图，都要**重新“计算”一遍**，付出相应的开销

`物化视图`是将某一时刻对应的查询结果保存起来。**需要占用存储空间**。
* 优点：访问物化视图时，**不用再“算”了**，直接取出预先存储好的数据即可，提高查询效率
* 缺点：**基础关系变化以后，必须修改物化视图以同步反映这一变化**。当然，延长多久来做这种修改，取决与你可以容忍物化视图的内容有多“过时”

**物化视图的应用场合**
* 频繁地对视图进行查询，并且基础关系很少变化，所以视图的内容也很少变化。即**读的频率远远高于写**。用物化视图可以减少计算量
* 或者即使基础关系变化了，也**允许“看到”的视图内容有一定的延迟**。















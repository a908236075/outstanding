# Mysql知识点

## 基本概念

1. ### 何时创建索引

   1. 主键自动简历唯一索引.
   2. 频繁作为查询条件的字段应该创建索引.
   3. 查询与其他表关联的字段,外键关联建立索引.
   4. 频繁更新的字段不适合创建索引.
   5. Where条件里用不到的字段不创建索引.

2. ### 常用函数

   1. DISTINCT  将重复的数据去掉.

   2. IF(expr1,expr2,expr3)  expr1是true,则if返回expr2,否则返回expr3.

   3. IFNUll(expr1,expr2)  如果expr1不为NUll返回expr1,否则返回expr2

      1. SELECT IFNULL(NULL,10);  返回值10.

   4. INSTR(str,substr)  返回在str中第一次出现substr的位置,如果找不到返回0

      1. SELECT INSTR('football','ball');  返回 5

   5. FLOOR(数值) 取整函数

      1. SELECT FLOOR(78.99);  78

   6. cast()类型转换函数  支持的转换的类型有binary,char,date,time,datetime,signed,unsigned(无符号值即为非负数).

   7. group_concat() 和并列函数 例如一个学生有多门成绩,可以合并到一起显示,可以自定义分隔符和进行排序.

      1. SELECT score.`学号`,GROUP_CONCAT(score.`成绩`) FROM score GROUP BY score.`学号`;

   8. ROUND() 四舍五入函数

   9. TRUNCATE(x,y)  返回数值x保留小数点后y位的值,不会进行四舍五入.

   10. Left(str,length),Right(str,length),substring(str,pos,length).截取字符串.

   11. CASE WHEN函数   END 
   
       1. CASE   
          WHEN [condititonal test 1] THEN [result1] 
       WHEN [condititonal test 2] THEN [result2] 
          ELSE [result3] 
   
          END

3. ### Sql语句进阶

   1. SELECT CASE pref_name
      WHEN '德岛' THEN '四国'
      WHEN '香川' THEN '四国'
      WHEN '爱媛' THEN '四国'
      ELSE '其他' END AS **district**,
      SUM(population)
      FROM PopTbl
      GROUP BY **district**;
      使用case when 的别名,进行分组,防止一处更改而忘了另一处.
   2. 约束 主键约束  唯一约束  check约束的 ,CONSTRAINT(外部连接约束) 一般会在建表的时候进行检验.
   3. 内连接和外连接的区别,如果通过价格排序table1.price<table2.price,使用内连接会把第一的价格过滤掉.
   4. 为什么判断null值是 is null 但不是 = null ,这是因为，NULL 既不是值也不是变量。NULL 只是一个表示“**没**
      **有值**”的标记，**而比较谓词只适用于值**。因此，对并非值的 NULL 使
      用比较谓词本来就是没有意义的
   5. 真值 unknown 和作为 NULL 的一种的UNKNOWN （未知）是不同的东西。前者是明确的布尔型的真值，后者既不是值也不是变量.
   6. All  是许多and 连接的省略的写法.有时候可以用极值函数(MAX MIN) 代替ALL,但是如果没有符合极值的条件,则没有返回值,然而我们想要返回第一个表的所有值.
   7. COUNT(*) 包含NULL值,COUNT(列名) 不包含空值.
   8. WHERE 子句用来调查集合元素的性质(行元素的值)，而 HAVING 子句用来调查集合本身的性质(一般会分组查询的时候使用)。
   9. 关联子查询和自连接在很多时候都是等价的.
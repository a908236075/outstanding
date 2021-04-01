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
   3. 内连接和外连接的区别,如果通过价格排序table1.price<table2.price**,使用内连接会把第一的价格过滤掉.**
   4. 为什么判断null值是 is null 但不是 = null ,这是因为，NULL 既不是值也不是变量。NULL 只是一个表示“**没**
      **有值**”的标记，**而比较谓词只适用于值**。因此，对并非值的 NULL 使
      用比较谓词本来就是没有意义的
   5. 真值 unknown 和作为 NULL 的一种的UNKNOWN （未知）是不同的东西。前者是明确的布尔型的真值，后者既不是值也不是变量.
   6. In和exists可以等价互换,但是not in 和not Exists不能等价互换,原因是由null的存在,如果age not in(12,13,null),这个永远返回null,而exists会返回结果**,exist是经过处理的,只会返回true或者false**,永远不会返回unknown.
   7. **All  是许多and 连接的省略的写法**.有时候可以用极值函数(MAX MIN) 代替ALL,但是如果没有符合极值的条件,则没有返回值,然而我们想要的正确结果是第一个表的所有值.
      1. ALL 谓词：他的年龄比在东京住的所有学生都小 
         极值函数：他的年龄比在东京住的年龄最小的学生还要小 
   8. COUNT(*) 包含NULL值,COUNT(列名) 不包含空值.
   9. **WHERE 子句用来调查集合元素的性质(行元素的值)，而 HAVING 子句用来调查集合本身的性质(一般会分组查询的时候使用)**。
   10. 关联子查询和自连接在很多时候都是等价的.
   
4. ### Sql 练习语句

   1. ~~~sql
      SELECT score.`学号`,COALESCE(NULL,score.`成绩`) AS good FROM score;
      SELECT 1,2,3 CROSS JOIN 2;
      CASE sex
      	WHEN 1 THEN '男' ,
      	WHEN 2 THEN '女'
      	ELSE '其它'
      	END;
      #按市统计男女数
      SELECT CASE WHEN sex=1 THEN '男' ELSE '女' END,SUM(sex) FROM TABLE GROUP BY sex;
      SELECT pre_name,SUM(CASE WHEN sex =1 THEN pop ELSE 0 END) AS '男',SUM(CASE WHEN sex =2 THEN pop ELSE 0 END) AS '女' FROM TABLE GROUP BY pre_name,
      ## 课程考试表
      SELECT course_name FROM coursemaster WHERE course_id IN()
      SELECT 
        course_id,
        CASE MONTH
          WHEN '200706' 
          THEN '6月'
          WHEN '200707' 
          THEN '7月'
          WHEN '200708' 
          THEN '8月' ELSE '其它' END AS TIME
          FROM  opencourse;
        #正确的解法
        SELECT course_name ,CASE WHEN course_id IN (SELECT course_id  FROM opencourse WHERE MONTH ='200706') THEN 'o' ELSE '×' END AS '6月' FROM coursemaster;
        SELECT A.course_name,CASE WHEN EXISTS (SELECT course_id  FROM opencourse B  WHERE  B.MONTH ='200706' AND A.course_id =B.course_id ) THEN 'o' ELSE '×' END AS '6月' FROM coursemaster A;
        ##只参加一个社团的学生
        SELECT std_id,MAX(clbu_id) FROM studentclub GROUP BY std_id HAVING COUNT(*)=1;
        ##参加了多个社团的
        SELECT std_id FROM studentclub WHERE main_club_flg ='Y';
        ## 查询出所有学生参加的主社团
        SELECT std_id ,CASE WHEN COUNT(*)=1 THEN MAX(clbu_id) ELSE MAX(CASE WHEN main_club_flg='Y' THEN clbu_id ELSE NULL END ) END AS 'mainclub' FROM studentclub GROUP BY std_id;
      ## greatest 列中最大的值
      SELECT GREATEST(X,Y,z) FROM greaters;
      SELECT CASE WHEN (CASE WHEN X<Y THEN Y ELSE X END) <z THEN z ELSE (CASE WHEN X<Y THEN Y ELSE X END)END AS greater FROM greaters;
      ##根据价格对products排序 跳过相同排名和不跳过相同排名
      SELECT P1.name,P1.price,(SELECT COUNT( DISTINCT P2.price) FROM products P2 WHERE P2.price >P1.price)+1 AS ranks  FROM products P1 ORDER BY ranks;
      SELECT COUNT(price) FROM products;
      SELECT P1.name, P2.name FROM products P1 LEFT OUTER JOIN  products P2 ON P2.price >P1.price;
      ~~~

   2. 
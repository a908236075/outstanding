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
   8. **COUNT(*) 包含NULL值,COUNT(列名) 不包含空值.**
   9. **WHERE 子句用来调查集合元素的性质(行元素的值)，而 HAVING 子句用来调查集合本身的性质(一般会分组查询的时候使用)**。
   10. 关联子查询和自连接在很多时候都是等价的.
   11. group by 分组后会形成新的视图,通常不能沿用原表的索引.所以使用where代替Having可是是性能更好.
   12. !=和not in 不能用到索引.

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


## 	MVCC

1. MVCC:多版本并发控制.
2. 隐藏字段:分别是事务id,**回滚段的指针**和主键.
3. undo_log:事务**未提交**,每次更改值都会产生一条日志,格式与隐藏字段一致.
4. MVCC只作用在RC和RR的事务的隔离级别,它们涉及到快照读.当select发生的时候,会生成一个readView读视图,注意与活跃的事物ID有关.读视图的结构是分别是:
   1. **creator_trx_id**:创建这个Read View的事务ID.只有在对表的记录做改动时,才会分配事务ID,否则在一个只读的事务id值默认为0.
   2. **trx_ids:**表示在生成ReadView时当前系统中**活跃的**读写事务的**事物id列表**
   3. **up_limit_id**:**活跃事物**中最小的事务ID.
   4. **low_limit_id:**表示生成ReadView时系统中应该分配给下一个事务的id值.low_limit_id是**系统的最大事物的ID**,不是活跃的事务ID.
5. ReadView的规则
   1. 有了这个ReadView，这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见。
      **trx_id是从查询数据的事务id进行获取,如果不满足需要从undo_log日志中获取**.如果被访问版本的trx_id属性值与ReadView中的 creator_trx_id 值相同，意味着当前事务在访问
      它自己修改过的记录，所以该版本可以被当前事务访问。
   2. 如果被访问版本的trx_id属性值小于ReadView中的 up_limit_id 值，表明生成该版本的事务在当前
      事务生成ReadView前已经提交，所以该版本可以被当前事务访问。
   3. 如果被访问版本的trx_id属性值大于或等于ReadView中的 low_limit_id 值，表明生成该版本的事
      务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。
   4. 如果被访问版本的trx_id属性值在ReadView的 up_limit_id 和 low_limit_id 之间，那就需要判
      断一下trx_id属性值是不是在 trx_ids 列表中。
   5. 如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问。
      如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问
6. RR隔离级别是事务之后**第一个** SELECT 语句生成ReadView,RC是事务是**每条**  SELECT 语句都会生成ReadView.所以RR解决了不可重复读和幻读的问题.
7. 快照读:在快照中进行读取,当前读:读取当前最新的值.

## 事务日志

1. 事务的**隔离性**由 锁机制 实现。而事务的**原子性、一致性和持久性**由事务的 redo 日志和undo 日志来保证。

   - REDO LOG 称为 重做日志 ，提供再写入操作，恢复提交事务修改的页操作，用来保证事务的持
     久性。(内存中的数据进行修改后,记录在了REDO日志中,如果数据库崩溃,可以通过日志进行恢复)
   - UNDO LOG 称为 回滚日志 ，回滚行记录到某个特定版本，用来保证事务的原子性、一致性。


## 索引

1. B+Tree结构

![](\picture\mysql\B+Tree.png)

2. B-Tree
   1. 
3. 
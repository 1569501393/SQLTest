[TOC]
# SQL面试50题

## 参考资料
> [SQL面试必会50题](https://zhuanlan.zhihu.com/p/43289968)

> [常见的SQL面试题：经典50题](https://zhuanlan.zhihu.com/p/38354000)

> [SQL经典50题——刷题必备](https://zhuanlan.zhihu.com/p/52530057)

> [SQL面试经典50题](https://www.jianshu.com/p/3f27a6dced16)

> [SQL数据库面试题以及答案（50例题）](https://www.cnblogs.com/lhx0827/p/9567199.html)


## 解题

### -- 总表

``` sql
CREATE VIEW v_student_score AS
SELECT student.*, score.s_score, course.c_id, course.c_name, teacher.t_id, teacher.t_name
FROM student -- 学生为基本表
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON score.c_id = course.c_id
LEFT JOIN teacher ON course.t_id = teacher.t_id
ORDER BY student.s_id ASC;

DROP TABLE if EXISTS student_score ;
CREATE table student_score 
AS 
SELECT student.*, score.s_score, course.c_id, course.c_name, teacher.t_id, teacher.t_name
FROM student -- 学生为基本表
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON score.c_id = course.c_id
LEFT JOIN teacher ON course.t_id = teacher.t_id
ORDER BY student.s_id ASC;

SELECT * FROM student_score;

```

s_id | s_name| s_birth| s_sex| s_score| c_id| c_name| t_id| t_name
---|---|---|---|---|---|---|---|---
01 | 赵雷 | 1990-01-01 | 男 | 80 | 01 | 语文 | 02 | 李四
01 | 赵雷 | 1990-01-01 | 男 | 90 | 02 | 数学 | 01 | 张三
01 | 赵雷 | 1990-01-01 | 男 | 99 | 03 | 英语 | 03 | 王五
02 | 钱电 | 1990-12-21 | 男 | 80 | 03 | 英语 | 03 | 王五
02 | 钱电 | 1990-12-21 | 男 | 60 | 02 | 数学 | 01 | 张三
02 | 钱电 | 1990-12-21 | 男 | 70 | 01 | 语文 | 02 | 李四
03 | 孙风 | 1990-05-20 | 男 | 80 | 01 | 语文 | 02 | 李四
03 | 孙风 | 1990-05-20 | 男 | 80 | 02 | 数学 | 01 | 张三
03 | 孙风 | 1990-05-20 | 男 | 80 | 03 | 英语 | 03 | 王五
04 | 李云 | 1990-08-06 | 男 | 20 | 03 | 英语 | 03 | 王五
04 | 李云 | 1990-08-06 | 男 | 30 | 02 | 数学 | 01 | 张三
04 | 李云 | 1990-08-06 | 男 | 50 | 01 | 语文 | 02 | 李四
05 | 周梅 | 1991-12-01 | 女 | 76 | 01 | 语文 | 02 | 李四
05 | 周梅 | 1991-12-01 | 女 | 87 | 02 | 数学 | 01 | 张三
06 | 吴兰 | 1992-03-01 | 女 | 31 | 01 | 语文 | 02 | 李四
06 | 吴兰 | 1992-03-01 | 女 | 34 | 03 | 英语 | 03 | 王五
07 | 郑竹 | 1989-07-01 | 女 | 89 | 02 | 数学 | 01 | 张三
07 | 郑竹 | 1989-07-01 | 女 | 98 | 03 | 英语 | 03 | 王五
08 | 王菊 | 1990-01-20 | 女 | 



### -- 1、查询“001”课程比“002”课程成绩高的所有学生的学号

```sql
-- 思路：学生表左外连接成绩表
EXPLAIN
SELECT * 
FROM student stu
LEFT JOIN score s1 ON stu.s_id = s1.s_id AND s1.c_id = 1
LEFT JOIN score s2 ON stu.s_id = s2.s_id AND s2.c_id = 2
WHERE s1.s_score > s2.s_score
ORDER BY stu.s_id DESC;

-- 其它解法
EXPLAIN
-- select a.s_id as s_id,score1,score2 from
select * from
(select s_id, s_score as score1 from score where c_id='01') a
inner join
(select s_id, s_score as score2 from score where c_id='02') b
on a.s_id=b.s_id
where score1>score2;


-- 存在没有成绩的课程
SELECT * 
FROM student stu
LEFT JOIN score s1 ON stu.s_id = s1.s_id AND s1.c_id = 1
LEFT JOIN score s2 ON stu.s_id = s2.s_id AND s2.c_id = 2
WHERE (IF(s1.s_score, s1.s_score, 0)) > (IF(s2.s_score, s2.s_score, 0))
ORDER BY stu.s_id DESC;
```


### -- 2、查询平均成绩大于60分的同学的学号和平均成绩 

```sql
-- 成绩表 AVG([DISTINCT] expr)
SELECT s_id, AVG(s_score) AS avg_score from score
GROUP BY s_id
HAVING  avg_score > 60;

SELECT *  from score
GROUP BY s_id
HAVING  AVG(s_score)   > 60; -- 直接在having中计算平均成绩

SELECT student.*,score.s_id, AVG(score.s_score) AS avg_score 
from score
LEFT JOIN student ON student.s_id = score.s_id
GROUP BY score.s_id
HAVING  avg_score > 60;
```


### -- 3、查询所有同学的学号、姓名、选课数、总成绩

```sql
SELECT student.s_id, student.s_name, COUNT(*), SUM(score.s_score)
FROM student
LEFT JOIN score ON student.s_id = score.s_id
GROUP BY student.s_id;
```


### -- 4、查询姓‘李’的老师的个数：

```sql
SELECT * 
FROM teacher
WHERE t_name LIKE '李%';

SELECT COUNT(*)
FROM teacher
WHERE t_name LIKE '李%';
```


### -- 5、查询没有学过“叶平”老师可的同学的学号、姓名： 

```sql
EXPLAIN
SELECT * 
FROM student 
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON course.c_id = score.c_id
LEFT JOIN teacher ON course.t_id= teacher.t_id
WHERE teacher.t_name != '张三'; 

EXPLAIN
SELECT * FROM student
WHERE student.s_id NOT IN (
SELECT student.s_id 
FROM student 
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON course.c_id = score.c_id
LEFT JOIN teacher ON course.t_id= teacher.t_id
WHERE teacher.t_name = '张三'
);

EXPLAIN
SELECT * FROM student
WHERE NOT EXISTS (
SELECT *
FROM score
LEFT JOIN course ON course.c_id = score.c_id
LEFT JOIN teacher ON course.t_id= teacher.t_id
WHERE teacher.t_name = '张三' AND student.s_id = score.s_id
);
```


### -- 6、查询学过“叶平”老师所教的所有课的同学的学号、姓名： 

```sql
SELECT * 
FROM student 
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON course.c_id = score.c_id
LEFT JOIN teacher ON course.t_id= teacher.t_id
WHERE teacher.t_name = '张三';  
```


### -- 7、查询学过“011”并且也学过编号“002”课程的同学的学号、姓名：

```sql
SELECT * FROM score 
WHERE c_id IN ('01', '02')
ORDER BY s_id ASC;

SELECT * FROM student
INNER JOIN score s1 ON student.s_id = s1.s_id AND s1.c_id = '01'
INNER JOIN score s2 ON student.s_id = s2.s_id AND s2.c_id = '02'
ORDER BY s1.s_id ASC;
```


### -- 8、查询课程编号“002”的成绩比课程编号“001”课程低的所有同学的学号、姓名：

```sql
SELECT * 
FROM student stu
LEFT JOIN score s1 ON stu.s_id = s1.s_id AND s1.c_id = '01'
LEFT JOIN score s2 ON stu.s_id = s2.s_id AND s2.c_id = '02'
WHERE s1.s_score < s2.s_score
ORDER BY stu.s_id DESC;
```


### -- 9、查询所有课程成绩小于60的同学的学号、姓名：

```sql
SELECT * FROM score;

SELECT * 
FROM student
WHERE NOT EXISTS (
SELECT * FROM score WHERE s_score >= 60 AND s_id = student.s_id
)
```


### -- 10、查询没有学全所有课的同学的学号、姓名：

```sql
SELECT * FROM course;

SELECT * FROM student
LEFT JOIN score ON student.s_id = score.s_id
GROUP BY student.s_id
HAVING COUNT(*) >= 3
ORDER BY student.s_id ASC;
```


### -- 11、查询至少有一门课与学号为“1001”同学所学相同的同学的学号和姓名：

```sql
SELECT * FROM score WHERE s_id = '01';

SELECT * FROM student,score
WHERE student.s_id = score.s_id AND score.c_id IN (
  SELECT c_id FROM score WHERE s_id = '01'
)
GROUP BY student.s_id
ORDER BY student.s_id ASC;
```


### -- 12、查询至少学过学号为“001”同学所有一门课的其他同学学号和姓名；

```sql
SELECT * FROM student,score
WHERE student.s_id = score.s_id AND student.s_id != '01' AND score.c_id IN (
  SELECT c_id FROM score WHERE s_id = '01'
)
GROUP BY student.s_id
ORDER BY student.s_id ASC;
```


### -- 13、把“SC”表中“叶平”老师教的课的成绩都更改为此课程的平均成绩(重点：构建中间表)：

```sql 
SELECT * FROM score 
WHERE c_id = '02';

-- [Err] 1093 - You can't specify target table 'score' for update in FROM clause
-- 解决方案：自己构建一张带有平均成绩的新表

-- 正解
update score s 
join (
select c_id,AVG(s_score) s_score from score where c_id in (
select c.c_id from course c join teacher t on c.t_id=t.t_id where t.t_name='张三'
) group by c_id
) t on s.c_id=t.c_id 
set s.s_score=t.s_score;


-- 构建中间表
UPDATE score s, (SELECT AVG(s_score) AS s_score_new, s.c_id FROM score s, teacher, course
                  WHERE s.c_id = course.c_id AND teacher.t_id = course.t_id AND t_name = '张三' 
                  GROUP BY s.c_id
                ) s2 
SET s.s_score = s2.s_score_new 
WHERE s.c_id = s2.c_id;


SELECT * FROM teacher WHERE t_name = '张三';
SELECT * FROM teacher, course WHERE teacher.t_id = course.t_id AND t_name = '张三';

UPDATE score
SET score.s_score = (
SELECT AVG(s2.s_score)
FROM score s2, teacher, course WHERE s2.c_id = course.c_id AND teacher.t_id = course.t_id AND t_name = '张三');

UPDATE score
SET score.s_score = (SELECT (
SELECT AVG(s2.s_score)
FROM score s2, teacher, course WHERE s2.c_id = course.c_id AND teacher.t_id = course.t_id AND t_name = '张三') AS avg);
[Err] 1093 - You can't specify target table 'score' for update in FROM clause
```


### -- 14、查询和“1002”号的同学学习的课程完全相同的其他同学学号和姓名：

```sql
SELECT s.s_id, COUNT(*) FROM score s 
RIGHT  JOIN (SELECT * FROM score WHERE s_id = '02') AS s2
ON s.c_id = s2.c_id
WHERE s.s_id != '02'
GROUP BY s.s_id 
HAVING COUNT(*) = (SELECT COUNT(*) FROM score WHERE s_id = '02');
```


### -- 15、删除学习“叶平”老师课的SC表记录：

```sql
SELECT * FROM teacher WHERE t_name = '张三';

-- 查询'张三'教授课程
SELECT * FROM teacher t, course c WHERE t_name = '张三' AND t.t_id = c.t_id;

SELECT * FROM score s
WHERE s.c_id IN (
  SELECT c.c_id FROM teacher t, course c WHERE t_name = '张三' AND t.t_id = c.t_id
)

DELETE FROM score 
WHERE c_id IN (
  SELECT c.c_id FROM teacher t, course c WHERE t_name = '张三' AND t.t_id = c.t_id
)
```



### -- 16、向SC表中插入一些记录，这些记录要求符合以下条件：没有上过编号“003”课程的同学学号、002号课的平均成绩：

```sql
SELECT * FROM score;

-- 没有上过编号“003”课程的同学学号 03同学很积极，都上满了
SELECT * FROM score 
WHERE c_id NOT IN (
  SELECT c_id FROM score WHERE s_id = '05'
)
```


### -- 17、按平均成绩从高到低显示所有学生的“数据库”、“企业管理”、“英语”三门的课程成绩，按如下形式显示：学生ID，数据库，企业管理，英语，有效课程数，有效平均分(待思考：按学号分组)：

```sql
SELECT score.s_id, 
AVG(score.s_score) AS avg_score, 
(SELECT s1.s_score FROM score AS s1 WHERE score.s_id = s1.s_id AND s1.c_id = '01') AS c1, 
(SELECT s1.s_score FROM score AS s1 WHERE score.s_id = s1.s_id AND s1.c_id = '02') AS c2, 
(SELECT s1.s_score FROM score AS s1 WHERE score.s_id = s1.s_id AND s1.c_id = '03') AS c3
FROM score
GROUP BY score.s_id
ORDER BY avg_score DESC;
```


### -- 18、查询各科成绩最高和最低的分： 以如下的形式显示：课程ID，最高分，最低分

```sql
SELECT c_id, MAX(s_score), MIN(s_score) FROM score
GROUP BY c_id;
```


### -- 19、按各科平均成绩从低到高和及格率的百分数从高到低顺序(重点：sum if)： 

```sql
SELECT c_id, AVG(s_score) AS avg_score, (SUM(IF(s_score >= 60, 1, 0)) / COUNT(*)) AS rate_good, 
ROUND((SUM(IF(s_score >= 60, 1, 0)) / COUNT(*)), 2) AS rate_good_round
FROM score
GROUP BY c_id
ORDER BY avg_score ASC, rate_good DESC;

01  64.5000 0.6667  0.67
03  68.5000 0.6667  0.67
02  72.6667 0.8333  0.83
```


### 20、查询如下课程平均成绩和及格率的百分数(用”1行”显示): 企业管理（001），马克思（002），OO&UML （003），数据库（004）：

```sql
按课程分组统计
SELECT score.c_id, COUNT(*) cnt, 
AVG(score.s_score) AS avg_score, 
(SUM(IF((score.s_score >= 60 ), 1, 0))) / COUNT(*)  AS rate
FROM score
GROUP BY score.c_id
ORDER BY avg_score DESC;
```


### -- 21、查询不同老师所教不同课程平均分从高到低显示：

```sql
SELECT t.t_id, c.c_id, AVG(s_score) AS avg_score FROM teacher t
LEFT JOIN course c ON t.t_id = c.t_id
LEFT JOIN score s ON c.c_id = s.c_id
GROUP BY t.t_id, c.c_id
ORDER BY avg_score DESC;

01  02  72.6667
03  03  68.5000
02  01  64.5000
```


### -- 22、查询如下课程成绩第3名到第6名的学生成绩单：企业管理(001)，马克思(002)，UML(003)，数据库(004)(待思考：除了用union连接外)：

```sql
-- UNION 连接
(SELECT * FROM score
WHERE c_id IN ('01')
ORDER BY s_score ASC 
LIMIT 2, 4)
UNION ALL
(SELECT * FROM score
WHERE c_id IN ('02')
ORDER BY s_score ASC 
LIMIT 2, 4);
```


### -- 23、统计下列各科成绩，各分数段人数：课程ID，课程名称，[100-85],[85-70],[70-60],[ 小于60] ：

```sql
SELECT 
c_id, COUNT(1) AS cnt,
SUM(IF((s_score >= 85 AND s_score <= 100), 1, 0)) AS s_score85_100,
SUM(IF((s_score >= 70 AND s_score < 85), 1, 0)) AS s_score70_85,
SUM(IF((s_score >= 60 AND s_score < 70), 1, 0)) AS s_score60_70,
SUM(IF((s_score < 60), 1, 0)) AS s_score60
FROM score
GROUP BY c_id;
```


### -- 24、查询学生平均成绩及其名次（重点：使用变量）：

```sql
待思考
SELECT s1.s_id, AVG(s1.s_score) AS avg_score, 
(SELECT sum(1) FROM score AS s2 WHERE s1.s_id = s2.s_id GROUP BY s2.s_id HAVING AVG(s2.s_score) <= AVG(s1.s_score) ) AS rank
FROM score AS s1
GROUP BY s1.s_id
ORDER BY avg_score DESC;

<!--使用变量-->
SET @rank = 0;
SELECT *, (@rank := @rank + 1) AS rank
FROM (
    SELECT s_id,AVG(score.s_score) from score
    GROUP BY s_id
    ORDER BY AVG(score.s_score) DESC
) t;

07  93.5000 1
01  89.6667 2
05  81.5000 3
03  80.0000 4
02  70.0000 5
04  33.3333 6
06  32.5000 7
```


### -- 25、查询各科成绩前三名的记录（不考虑成绩并列情况）(待思考：子查询作为条件)

```sql
SELECT * 
FROM score AS s1
WHERE (
    SELECT COUNT(*) FROM score AS s2
    WHERE s1.c_id = s2.c_id AND s2.s_score > s1.s_score
) < 3
ORDER BY s1.c_id ASC, s1.s_score DESC;
```


### -- 26、查询每门课程被选修的学生数：

```sql
SELECT c_id, COUNT(DISTINCT s_id)
FROM score
GROUP BY c_id;
```



### -- 27、查询出只选修一门课程的全部学生的学号和姓名：

```sql
SELECT student.s_id, student.s_name FROM student 
LEFT JOIN score 
ON student.s_id = score.s_id
GROUP BY student.s_id
HAVING COUNT(DISTINCT c_id) = 1;
```



### -- 28、查询男生、女生人数：

```sql
SELECT s_sex, COUNT(*) FROM student
GROUP BY s_sex;
```


### -- 29、查询姓“张”的学生名单：

```sql
SELECT * FROM student WHERE s_name LIKE '赵%';
```


### -- 30、查询同名同姓的学生名单，并统计同名人数：

```sql 
SELECT s_name, COUNT(*) cnt FROM student 
GROUP BY s_name
HAVING cnt > 1;

```


### -- 31、1981年出生的学生名单（注：student表中sage列的类型是datetime）:

```sql 
SELECT * FROM student
WHERE s_birth > '1990';
```


### -- 32、查询平均成绩大于85的所有学生的学号、姓名和平均成绩：

```
SELECT student.s_id, student.s_name, AVG(s_score) AS avg_score
FROM student
LEFT JOIN score ON student.s_id = score.s_id
GROUP BY student.s_id
HAVING avg_score > 85;

```


### -- 33、查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列：

```sql
SELECT c_id, AVG(s_score) AS avg_score 
FROM score
GROUP BY c_id
ORDER BY avg_score ASC, c_id DESC;
```


### -- 34、查询课程名称为“数据库”，且分数低于60的学生名字和分数：

```sql
在关联课程表和学生表
SELECT * FROM score
WHERE s_score < 60
AND c_id = '01';
```


### -- 35、查询所有学生的选课情况：

```sql
SELECT * FROM student
LEFT JOIN score ON student.s_id = score.s_id;
```

### -- 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数：

```sql 
关联学生表和成绩表
SELECT s_id, c_id, s_score
FROM score
WHERE s_score > 70
GROUP BY s_id, c_id;
```


### -- 37、查询不及格的课程，并按课程号从大到小的排列：

```
SELECT * 
FROM score
WHERE s_score < 60
ORDER BY c_id DESC;
```


### -- 38、查询课程编号为“003”且课程成绩在80分以上的学生的学号和姓名：

```
SELECT * 
FROM score
WHERE c_id = '03' AND s_score > 80;
```


### -- 39、求选了课程的学生人数：

```sql 
SELECT COUNT(DISTINCT(student.s_id))
FROM student
LEFT JOIN score ON student.s_id = score.s_id
WHERE score.s_score IS NOT NULL;
```


### -- 40、查询选修“叶平”老师所授课程的学生中，成绩最高的学生姓名及其成绩：

```sql
SELECT * FROM score
WHERE s_score = (
SELECT MAX(s_score)
FROM score
WHERE c_id = '02');
```


### -- 41、查询各个课程及相应的选修人数：

```sql
SELECT c_id, COUNT(*)
FROM score
GROUP BY c_id;
```


### -- 42、查询不同课程成绩相同的学生和学号、课程号、学生成绩：

```sql
SELECT * FROM score;

SELECT * 
FROM score s1
LEFT JOIN score s2 ON s1.s_id = s2.s_id
WHERE s1.s_score = s2.s_score AND s1.c_id != s2.c_id;

-- 需要分组统计，否则会重复
EXPLAIN
SELECT * 
FROM score s1
LEFT JOIN score s2 ON s1.s_id = s2.s_id
LEFT JOIN student ON s1.s_id = student.s_id
WHERE s1.s_score = s2.s_score AND s1.c_id != s2.c_id
GROUP BY s1.c_id;

EXPLAIN
SELECT  DISTINCT(s1.c_id), s1.s_id, student.s_name, s1.s_score
FROM score s1
LEFT JOIN score s2 ON s1.s_id = s2.s_id
LEFT JOIN student ON s1.s_id = student.s_id
WHERE s1.s_score = s2.s_score AND s1.c_id != s2.c_id;
```


### 43、查询每门课程成绩最好的前两名(待思考：UNION， 两表关联)：

```sql
SELECT * FROM score
WHERE c_id = '01'
ORDER BY s_score DESC
LIMIT 0, 2;

正解
SELECT s1.* FROM score s1 WHERE
(
SELECT COUNT(1) FROM score s2 WHERE
s1.c_id=s2.c_id AND s2.s_score>=s1.s_score
) <= 2
ORDER BY s1.c_id,s1.s_score DESC;

(SELECT * FROM score
WHERE c_id = '01'
ORDER BY s_score DESC
LIMIT 0, 2
)
UNION 
(SELECT * FROM score
WHERE c_id = '02'
ORDER BY s_score DESC
LIMIT 0, 2
)
```


### -- 44、统计每门课程的学生选修人数(超过10人的课程才统计)。要求输出课程号和选修人数，查询结果按人数降序排序，若人数相同，按课程号升序排序

```sql
SELECT c_id, COUNT(*) cnt
FROM score
GROUP BY c_id
HAVING cnt > 3
ORDER BY cnt DESC, c_id ASC;
```


### -- 45、检索至少选修两门课程的学生学号：

```sql
SELECT s_id
FROM score
GROUP BY s_id
HAVING COUNT(s_id) > 2;
```


### -- 46、查询全部学生选修的课程和课程号和课程名：

```sql
SELECT *
FROM student
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON score.c_id = course.c_id; 
```


### -- 47、查询没学过”叶平”老师讲授的任一门课程的学生姓名：

```sql
SELECT * FROM student
WHERE NOT EXISTS (
    SELECT course.c_id 
    FROM teacher
    LEFT JOIN course ON teacher.t_id = course.t_id
    LEFT JOIN score ON course.c_id = score.c_id
    WHERE teacher.t_name = '张三' AND student.s_id = score.s_id
);
```



### -- 48、查询两门以上不及格课程的同学的学号以及其平均成绩：

```sql
SELECT s_id, AVG(s_score) AS avg_score
FROM score
WHERE EXISTS (
    SELECT * FROM score s1
    WHERE s1.s_score <60 AND s1.s_id = score.s_id
    GROUP BY s1.s_id
)
GROUP BY s_id
ORDER BY avg_score DESC;
```


### -- 49、检索“004”课程分数小于60，按分数降序排列的同学学号：

```sql
SELECT * FROM score
WHERE c_id = '01' AND s_score < 60
ORDER BY s_id DESC;
```


### -- 50、删除“002”同学的“001”课程的成绩：

```sql
SELECT * FROM score
WHERE s_id = '02' AND c_id = '01';

DELETE score
WHERE s_id = '02' AND c_id = '01';
```


## 建表

```sql
/*
Navicat MySQL Data Transfer

Source Server         : localhost
Source Server Version : 50553
Source Host           : 127.0.0.1:3306
Source Database       : test50

Target Server Type    : MYSQL
Target Server Version : 50553
File Encoding         : 65001

Date: 2019-09-22 22:21:47
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for course
-- ----------------------------
DROP TABLE IF EXISTS `course`;
CREATE TABLE `course` (
  `c_id` varchar(20) NOT NULL,
  `c_name` varchar(20) NOT NULL DEFAULT '' COMMENT '课程名',
  `t_id` varchar(20) NOT NULL COMMENT '教师主键',
  PRIMARY KEY (`c_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='课程表';

-- ----------------------------
-- Records of course
-- ----------------------------
INSERT INTO `course` VALUES ('01', '语文', '02');
INSERT INTO `course` VALUES ('02', '数学', '01');
INSERT INTO `course` VALUES ('03', '英语', '03');

-- ----------------------------
-- Table structure for score
-- ----------------------------
DROP TABLE IF EXISTS `score`;
CREATE TABLE `score` (
  `s_id` varchar(20) NOT NULL DEFAULT '' COMMENT '学号',
  `c_id` varchar(20) NOT NULL DEFAULT '' COMMENT '课程号',
  `s_score` int(3) NOT NULL DEFAULT '0' COMMENT '成绩',
  PRIMARY KEY (`s_id`,`c_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='成绩表';

-- ----------------------------
-- Records of score
-- ----------------------------
INSERT INTO `score` VALUES ('01', '01', '80');
INSERT INTO `score` VALUES ('01', '02', '90');
INSERT INTO `score` VALUES ('01', '03', '99');
INSERT INTO `score` VALUES ('02', '01', '70');
INSERT INTO `score` VALUES ('02', '02', '60');
INSERT INTO `score` VALUES ('02', '03', '80');
INSERT INTO `score` VALUES ('03', '01', '80');
INSERT INTO `score` VALUES ('03', '02', '80');
INSERT INTO `score` VALUES ('03', '03', '80');
INSERT INTO `score` VALUES ('04', '01', '50');
INSERT INTO `score` VALUES ('04', '02', '30');
INSERT INTO `score` VALUES ('04', '03', '20');
INSERT INTO `score` VALUES ('05', '01', '76');
INSERT INTO `score` VALUES ('05', '02', '87');
INSERT INTO `score` VALUES ('06', '01', '31');
INSERT INTO `score` VALUES ('06', '03', '34');
INSERT INTO `score` VALUES ('07', '02', '89');
INSERT INTO `score` VALUES ('07', '03', '98');

-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
  `s_id` varchar(20) NOT NULL,
  `s_name` varchar(20) NOT NULL DEFAULT '',
  `s_birth` varchar(20) NOT NULL DEFAULT '',
  `s_sex` varchar(10) NOT NULL DEFAULT '',
  PRIMARY KEY (`s_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='学生表';

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('01', '赵雷', '1990-01-01', '男');
INSERT INTO `student` VALUES ('02', '钱电', '1990-12-21', '男');
INSERT INTO `student` VALUES ('03', '孙风', '1990-05-20', '男');
INSERT INTO `student` VALUES ('04', '李云', '1990-08-06', '男');
INSERT INTO `student` VALUES ('05', '周梅', '1991-12-01', '女');
INSERT INTO `student` VALUES ('06', '吴兰', '1992-03-01', '女');
INSERT INTO `student` VALUES ('07', '郑竹', '1989-07-01', '女');
INSERT INTO `student` VALUES ('08', '王菊', '1990-01-20', '女');

-- ----------------------------
-- Table structure for student_score
-- ----------------------------
DROP TABLE IF EXISTS `student_score`;
CREATE TABLE `student_score` (
  `s_id` varchar(20) NOT NULL,
  `s_name` varchar(20) NOT NULL DEFAULT '',
  `s_birth` varchar(20) NOT NULL DEFAULT '',
  `s_sex` varchar(10) NOT NULL DEFAULT '',
  `s_score` int(3) DEFAULT '0' COMMENT '成绩',
  `c_id` varchar(20),
  `c_name` varchar(20) DEFAULT '' COMMENT '课程名',
  `t_id` varchar(20) COMMENT '教师主键',
  `t_name` varchar(20) DEFAULT '' COMMENT '教师名'
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of student_score
-- ----------------------------
INSERT INTO `student_score` VALUES ('01', '赵雷', '1990-01-01', '男', '80', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('01', '赵雷', '1990-01-01', '男', '90', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('01', '赵雷', '1990-01-01', '男', '99', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('02', '钱电', '1990-12-21', '男', '70', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('02', '钱电', '1990-12-21', '男', '60', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('02', '钱电', '1990-12-21', '男', '80', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('03', '孙风', '1990-05-20', '男', '80', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('03', '孙风', '1990-05-20', '男', '80', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('03', '孙风', '1990-05-20', '男', '80', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('04', '李云', '1990-08-06', '男', '50', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('04', '李云', '1990-08-06', '男', '30', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('04', '李云', '1990-08-06', '男', '20', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('05', '周梅', '1991-12-01', '女', '76', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('05', '周梅', '1991-12-01', '女', '87', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('06', '吴兰', '1992-03-01', '女', '31', '01', '语文', '02', '李四');
INSERT INTO `student_score` VALUES ('06', '吴兰', '1992-03-01', '女', '34', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('07', '郑竹', '1989-07-01', '女', '89', '02', '数学', '01', '张三');
INSERT INTO `student_score` VALUES ('07', '郑竹', '1989-07-01', '女', '98', '03', '英语', '03', '王五');
INSERT INTO `student_score` VALUES ('08', '王菊', '1990-01-20', '女', null, null, null, null, null);

-- ----------------------------
-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher` (
  `t_id` varchar(20) NOT NULL COMMENT '教师主键',
  `t_name` varchar(20) NOT NULL DEFAULT '' COMMENT '教师名',
  PRIMARY KEY (`t_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='教师表';

-- ----------------------------
-- Records of teacher
-- ----------------------------
INSERT INTO `teacher` VALUES ('01', '张三');
INSERT INTO `teacher` VALUES ('02', '李四');
INSERT INTO `teacher` VALUES ('03', '王五');

-- ----------------------------
-- Table structure for test2
-- ----------------------------
DROP TABLE IF EXISTS `test2`;
CREATE TABLE `test2` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `remark` varchar(255) NOT NULL DEFAULT '' COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of test2
-- ----------------------------

-- ----------------------------
-- Table structure for test_gbk
-- ----------------------------
DROP TABLE IF EXISTS `test_gbk`;
CREATE TABLE `test_gbk` (
  `id` int(11) NOT NULL DEFAULT '0' COMMENT '主键',
  `remark` varchar(255) NOT NULL DEFAULT '' COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='cmd字符集测试';

-- ----------------------------
-- Records of test_gbk
-- ----------------------------

-- ----------------------------
-- View structure for v_student_score
-- ----------------------------
DROP VIEW IF EXISTS `v_student_score`;
CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`localhost`  VIEW `v_student_score` AS SELECT student.*, score.s_score, course.c_id, course.c_name, teacher.t_id, teacher.t_name
FROM student -- 学生为基本表
LEFT JOIN score ON student.s_id = score.s_id
LEFT JOIN course ON score.c_id = course.c_id
LEFT JOIN teacher ON course.t_id = teacher.t_id ;

```


# Mysql 练习题


**我使用的Mysql版本是5.7.19。答案可能会因版本会有少许出入**。

## 练习数据
数据表
--1.学生表
Student(SId,Sname,Sage,Ssex) 

--SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

--2.课程表 
Course(CId,Cname,TId) 
--CId --课程编号,Cname 课程名称,TId 教师编号

--3.教师表 
Teacher(TId,Tname)
 --TId 教师编号,Tname 教师姓名

--4.成绩表 
SC(SId,CId,score)
 --SId 学生编号,CId 课程编号,score 分数

创建测试数据

学生表 Student
```sql
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2017-12-30' , '女');
insert into Student values('12' , '赵六' , '2017-01-01' , '女');
insert into Student values('13' , '孙七' , '2018-01-01' , '女');
```
科目表 Course
```sql
create table Course(CId varchar(10),Cname varchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```
教师表 Teacher
```sql
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```
成绩表 SC
```sql
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```
## 练习题目
1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
1.1 查询同时存在" 01 "课程和" 02 "课程的情况
1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
1.3 查询不存在" 01 "课程但存在" 02 "课程的情况
2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
3. 查询在 SC 表存在成绩的学生信息
4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )
4.1 查有成绩的学生信息
5. 查询「李」姓老师的数量 
6. 查询学过「张三」老师授课的同学的信息 
7. 查询没有学全所有课程的同学的信息 
8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息 
9. 查询和" 01 "号的同学学习的课程   完全相同的其他同学的信息 
10. 查询没学过"张三"老师讲授的任一门课程的学生姓名 
11. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 
12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
14. 查询各科成绩最高分、最低分和平均分：
    以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
    及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
    要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次
16.  查询学生的总成绩，并进行排名，总分重复时保留名次空缺
16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
18. 查询各科成绩前三名的记录
19. 查询每门课程被选修的学生数 
20. 查询出只选修两门课程的学生学号和姓名 
21. 查询男生、女生人数
22. 查询名字中含有「风」字的学生信息
23. 查询同名同性学生名单，并统计同名人数
24. 查询 1990 年出生的学生名单
25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩 
27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数 
28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
30. 查询不及格的课程
31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
32. 求每门课程的学生人数 
33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 
36. 查询每门功成绩最好的前两名
37. 统计每门课程的学生选修人数（超过 5 人的课程才统计）。
38. 检索至少选修两门课程的学生学号 
39. 查询选修了全部课程的学生信息
40. 查询各学生的年龄，只按年份来算 
41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
42. 查询本周过生日的学生
43. 查询下周过生日的学生
44. 查询本月过生日的学生
45. 查询下月过生日的学生



## 参考答案

1.查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
```sql
select *
from (select SId ,score from sc where sc.CId='01')as t1 , (select SId ,score from sc where sc.CId='02') as t2
where t1.SId=t2.SId
and   t1.score>t2.score
```

1.1 查询同时存在" 01 "课程和" 02 "课程的情况
```sql
select *
from (select SId ,score from sc where sc.CId='01')as t1 , (select SId ,score from sc where sc.CId='02') as t2
where t1.SId=t2.SId
```
1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
```sql
select *
from (select SId ,score from sc where sc.CId='01')as t1 left join (select SId ,score from sc where sc.CId='02') as t2
on t1.SId=t2.SId
```
1.3 查询不存在" 01 "课程但存在" 02 "课程的情况
```sql
select *
from sc
where sc.SId not in (select SId from sc where sc.CId='01')
and  sc.CId='02'
```
2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
```sql
select student.*,t1.avgscore
from student inner JOIN(
select sc.SId ,AVG(sc.score)as avgscore
from sc 
GROUP BY sc.SId
HAVING AVG(sc.score)>=60)as t1 on student.SId=t1.SId 
```
3. 查询在 SC 表存在成绩的学生信息
```sql
select DISTINCT student.*
from student ,sc
where student.SId=sc.SId
```

```
select student.* from student,
(select SId from sc group by sc.SId)as t1
where student.SId=t1.SId;
```

4.查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为null)  

```sql 
select student.SId,student.Sname,t1.sumscore,t1.coursecount
from student ,(
select SC.SId,sum(sc.score)as sumscore ,count(sc.CId) as coursecount
from sc 
GROUP BY sc.SId) as t1
where student.SId =t1.SId
```
```sql
select student.SId,student.Sname,t1.count_num,t1.score_sum from student left join
(select sc.SId,count(sc.CId)as count_num,sum(sc.score)as score_sum from sc group by sc.SId)as t1 on student.SId=t1.SId
```

4.1 查有成绩的学生信息

```sql
select *
from student
where EXISTS(select * from sc where student.SId=sc.SId)
```
```sql
select * from student 
where student.SId in(select sc.SId from sc);  --使用in代替exists
```


5. 查询「李」姓老师的数量 
 ```sql
select count(*)
from teacher
where teacher.Tname like '李%
 ```
6. 查询学过「张三」老师授课的同学的信息 
```sql
select student.*
from teacher  ,course  ,student,sc
where teacher.Tname='张三'
and   teacher.TId=course.TId
and   course.CId=sc.CId
and   sc.SId=student.SId
```
```sql
 select * from student  
 where SId in  
 (select SId from sc where CId in
  (select CId from course where TId in
   (select TId from teacher where Tname='张三')));
```


7. 查询没有学全所有课程的同学的信息
- 解法1
```sql
select student.*
from sc ,student
where sc.SId=student.SId
GROUP BY sc.SId
Having count(*)<(select count(*) from course)
```
```sql
select student.* 
from student left join sc 
on student.SId=sc.SId 
group by student.SId 
having count(*) <(select count(*) from course);
```

但这种解法得出来的结果不包括什么课都没选的同学。**

- 解法2
```sql
select DISTINCT student.*
from 
(select student.SId,course.CId
from student,course ) as t1 LEFT JOIN (SELECT sc.SId,sc.CId from sc)as t2 on t1.SId=t2.SId and t1.CId=t2.CId,student
where t2.SId is null
and   t1.SId=student.SId
```
**利用笛卡尔积可以把什么课都没选的同学查询出来**

8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
```sql 
select DISTINCT student.*
from  sc ,student
where sc.CId in (select CId from sc where sc.SId='01')
and   sc.SId=student.SId
```

```sql
select * from student 
where SId in 
(select SId from SC where CId in (select CId from SC where SId='01'));
```

9.查询和" 01 "号的同学学习的课程完全相同的其他同学的信息 

```sql
select DISTINCT student.*
from (
select student.SId,t.CId
from student ,(select sc.CId from sc where sc.SId='01') as t) as t1 LEFT JOIN sc on t1.SId=sc.SId and t1.CId=sc.CId,student
where sc.SId is null 
and   t1.SId=student.SId
```

10.查询没学过"张三"老师讲授的任一门课程的学生姓名 
```sql
select *
from student 
where student.SId not in 
(
select student.SId
from student left join sc on student.SId=sc.SId 
where EXISTS 
(select *
from teacher ,course
where teacher.Tname='张三'
and   teacher.TId=course.TId
and 	course.CId=sc.CId))
```
```sql
select * from student 
where SId not in 
(select SId from sc where CId in(select CId from course where TId in (select TId from teacher where Tname='张三')));
```

11.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 
```sql
select student.SId,student.Sname,avg(sc.score)
from student ,sc
where student.SId=sc.SId
and   sc.score<60
GROUP BY sc.SId
HAVING count(*)>=2
```

12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
```sql
select student.*, sc.score from student, sc
where student.sid = sc.sid
and sc.score < 60
and cid = "01"
ORDER BY sc.score DESC;
```
```sql
+------+--------+---------------------+------+-------+
| SId  | Sname  | Sage                | Ssex | score |
+------+--------+---------------------+------+-------+
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |  50.0 |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |  31.0 |
+------+--------+---------------------+------+-------+
2 rows in set (0.00 sec)
```

13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
```sql
select sc.SId,sc.CId,sc.score,t1.avgscore 
from  sc left join (select sc.SId,avg(sc.score) as avgscore from sc GROUP BY sc.SId) as t1 on sc.SId =t1.SId 
ORDER BY t1.avgscore DESC;
```
或者

```sql
select *  
from sc left join (select sid,avg(score) as avgscore from sc group by sid)r 
on sc.sid = r.sid
order by avgscore desc;
```

```
+------+------+-------+------+-----------+
| SId  | CId  | score | sid  | avgscore  |
+------+------+-------+------+-----------+
| 09   | 03   | 100.0 | 09   | 100.00000 |
| 07   | 03   |  98.0 | 07   |  93.50000 |
| 07   | 02   |  89.0 | 07   |  93.50000 |
| 01   | 01   |  80.0 | 01   |  89.66667 |
| 01   | 03   |  99.0 | 01   |  89.66667 |
| 01   | 02   |  90.0 | 01   |  89.66667 |
| 05   | 01   |  76.0 | 05   |  81.50000 |
| 05   | 02   |  87.0 | 05   |  81.50000 |
| 03   | 02   |  80.0 | 03   |  80.00000 |
| 03   | 01   |  80.0 | 03   |  80.00000 |
| 03   | 03   |  80.0 | 03   |  80.00000 |
| 02   | 02   |  60.0 | 02   |  70.00000 |
| 02   | 01   |  70.0 | 02   |  70.00000 |
| 02   | 03   |  80.0 | 02   |  70.00000 |
| 04   | 01   |  50.0 | 04   |  33.33333 |
| 04   | 03   |  20.0 | 04   |  33.33333 |
| 04   | 02   |  30.0 | 04   |  33.33333 |
| 06   | 01   |  31.0 | 06   |  32.50000 |
| 06   | 03   |  34.0 | 06   |  32.50000 |
+------+------+-------+------+-----------+
19 rows in set (0.00 sec)
```


14. 查询各科成绩最高分、最低分和平均分：
    以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
    及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
    要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
```sql
select sc.CId ,max(sc.score)as 最高分,min(sc.score)as 最低分,AVG(sc.score)as 平均分,count(*)as 选修人数,sum(case when sc.score>=60 then 1 else 0 end )/count(*)as 及格率,sum(case when sc.score>=70 and sc.score<80 then 1 else 0 end )/count(*)as 中等率,sum(case when sc.score>=80 and sc.score<90 and sc.score<80 then 1 else 0 end )/count(*)as 优良率,sum(case when sc.score>=90 then 1 else 0 end )/count(*)as 优秀率 
from sc
GROUP BY sc.CId
ORDER BY count(*)DESC,sc.CId asc
```

15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
```sql
select sc.CId ,@curRank:=@curRank+1 as rank,sc.score
from (select @curRank:=0) as t ,sc
ORDER BY sc.score desc
```

15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次
```sql 
select sc.CId , case when @fontscore=score then @curRank when @fontscore:=score then @curRank:=@curRank+1  end as rank,sc.score
from (select @curRank:=0 ,@fontage:=null) as t ,sc
ORDER BY sc.score desc
```

16.  查询学生的总成绩，并进行排名，总分重复时保留名次空缺
```sql
select t1.*,@currank:= @currank+1 as rank
from (select sc.SId, sum(score)
from sc
GROUP BY sc.SId 
ORDER BY sum(score) desc) as t1,(select @currank:=0) as t
```

16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
```sql
select t1.*, case when @fontscore=t1.sumscore then @currank  when @fontscore:=t1.sumscore  then @currank:=@currank+1  end as rank
from (select sc.SId, sum(score) as sumscore
from sc
GROUP BY sc.SId 
ORDER BY sum(score) desc) as t1,(select @currank:=0,@fontscore:=null) as t
```

17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
```sql 
select course.CId,course.Cname,t1.*
from course LEFT JOIN (
select sc.CId,CONCAT(sum(case when sc.score>=85 and sc.score<=100 then 1 else 0 end )/count(*)*100,'%') as '[85-100]',
CONCAT(sum(case when sc.score>=70 and sc.score<85 then 1 else 0 end )/count(*)*100,'%') as '[70-85)',
CONCAT(sum(case when sc.score>=60 and sc.score<70 then 1 else 0 end )/count(*)*100,'%') as '[60-70)',
CONCAT(sum(case when sc.score>=0 and sc.score<60 then 1 else 0 end )/count(*)*100,'%') as '[0-60)'
from sc
GROUP BY sc.CId) as t1 on course.CId=t1.CId
```

18. 查询各科成绩前三名的记录

思路：前三名转化为若大于此成绩的数量少于3即为前三名。
```sql
select *
from sc  
where  (select count(*) from sc as a where sc.CId =a.CId and  sc.score <a.score )<3
ORDER BY CId asc,sc.score desc
```

19. 查询每门课程被选修的学生数 
```
select cid, count(*) from sc 
group by cid;
```

```
+------+------------+
| CId  | count(CID) |
+------+------------+
| 01   |          6 |
| 02   |          6 |
| 03   |          7 |
+------+------------+
3 rows in set (0.00 sec)
```

20. 查询出只选修两门课程的学生学号和姓名
```sql
--联合
select student.SId,student.Sname
from sc,student
where student.SId=sc.SId  
GROUP BY sc.SId
HAVING count(*)=2
```
```sql
-- 嵌套
select sid,sname 
from student 
where sid in 
(select sid from sc group by sid having count(*)=2);
```

```
+------+--------+
| sid  | sname  |
+------+--------+
| 05   | 周梅   |
| 06   | 吴兰   |
| 07   | 郑竹   |
+------+--------+
3 rows in set (0.00 sec)
```

21.查询男生、女生人数

```sql
select ssex, count(*) from student
group by ssex;
```
```
+------+----------+
| ssex | count(*) |
+------+----------+
| 女   |        8 |
| 男   |        4 |
+------+----------+
2 rows in set (0.00 sec)
```

22. 查询名字中含有「风」字的学生信息
```sql
select *
from student 
where Sname like '%风%'
```

```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
+------+--------+---------------------+------+
1 row in set (0.00 sec)
```

23.查询同名同性学生名单，并统计同名人数

```sql 
select * from student 
where sname in (select sname from student group by sname having count(*)>1);
```

```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 10   | 李四   | 2017-12-25 00:00:00 | 女   |
| 11   | 李四   | 2017-12-30 00:00:00 | 女   |
+------+--------+---------------------+------+
2 rows in set (0.00 sec)
```

```sql
select *,count(*)同名人数 from student  
where sname in (select sname from student group by sname having count(*)>1);
```

```
+------+--------+---------------------+------+--------------+
| SId  | Sname  | Sage                | Ssex | 同名人数     |
+------+--------+---------------------+------+--------------+
| 10   | 李四   | 2017-12-25 00:00:00 | 女   |            2 |
+------+--------+---------------------+------+--------------+
1 row in set (0.00 sec)
```

24.查询 1990 年出生的学生名单

```sql
select *
from student
where YEAR(student.Sage)=1990
```

```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
+------+--------+---------------------+------+
4 rows in set (0.00 sec)
```

25.查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

```sql
select sc.cid, course.cname, AVG(SC.SCORE) as 平均成绩 from sc, course
where sc.cid = course.cid
group by sc.cid 
order by 平均成绩 desc,cid asc;
```

26.查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩 
```sql
select student.SId,student.Sname,t1.avgscore
from student INNER JOIN (select sc.SId ,AVG(sc.score) as avgscore from sc GROUP BY sc.SId HAVING AVG(sc.score)>85) as t1 on 
student.SId=t1.SId
```
```sql
-- 我的解法
select student.sid,student.sname,avg(sc.score)as avg 
from sc,student 
where student.sid=sc.sid 
group by student.sid 
having avg>=85;
```

```
+------+--------+-----------+
| sid  | sname  | avg       |
+------+--------+-----------+
| 01   | 赵雷   |  89.66667 |
| 07   | 郑竹   |  93.50000 |
| 09   | 张三   | 100.00000 |
+------+--------+-----------+
3 rows in set (0.00 sec)
```

27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
```sql
select student.Sname ,t1.score
from student INNER JOIN  (select sc.SId,sc.score 
from sc,course
where sc.CId=course.CId
and   course.Cname='数学'
and   sc.score<60)as t1 on student.SId=t1.SId 
```
```sql
-- 我的
select student.sname,sc.score 
from student,sc,course 
where course.cname='数学' 
and sc.score<60
and course.cid=sc.cid 
and sc.sid=student.sid ;

```

```
+--------+-------+
| sname  | score |
+--------+-------+
| 李云   |  30.0 |
+--------+-------+
1 row in set (0.00 sec)
```

28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
```sql
select student.SId,sc.CId,sc.score 
from Student  left join sc  
on student.SId=sc.SId 
```
```sql
-- 我的
select student.sid,sname,sc.cid,cname,score 
from  (student left join sc  on student.sid=sc.sid )left join course on sc.cid=course.cid;
```

```
+------+--------+------+--------+-------+
| sid  | sname  | cid  | cname  | score |
+------+--------+------+--------+-------+
| 01   | 赵雷   | 01   | 语文   |  80.0 |
| 02   | 钱电   | 01   | 语文   |  70.0 |
| 03   | 孙风   | 01   | 语文   |  80.0 |
| 04   | 李云   | 01   | 语文   |  50.0 |
| 05   | 周梅   | 01   | 语文   |  76.0 |
| 06   | 吴兰   | 01   | 语文   |  31.0 |
| 01   | 赵雷   | 02   | 数学   |  90.0 |
| 02   | 钱电   | 02   | 数学   |  60.0 |
| 03   | 孙风   | 02   | 数学   |  80.0 |
| 04   | 李云   | 02   | 数学   |  30.0 |
| 05   | 周梅   | 02   | 数学   |  87.0 |
| 07   | 郑竹   | 02   | 数学   |  89.0 |
| 01   | 赵雷   | 03   | 英语   |  99.0 |
| 02   | 钱电   | 03   | 英语   |  80.0 |
| 03   | 孙风   | 03   | 英语   |  80.0 |
| 04   | 李云   | 03   | 英语   |  20.0 |
| 06   | 吴兰   | 03   | 英语   |  34.0 |
| 07   | 郑竹   | 03   | 英语   |  98.0 |
| 09   | 张三   | 03   | 英语   | 100.0 |
| 10   | 李四   | NULL | NULL   |  NULL |
| 11   | 李四   | NULL | NULL   |  NULL |
| 12   | 赵六   | NULL | NULL   |  NULL |
| 13   | 孙七   | NULL | NULL   |  NULL |
+------+--------+------+--------+-------+
23 rows in set (0.00 sec)
```


29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
```sql
select student.Sname,course.Cname,sc.score
from student , sc  ,course
where sc.score>=70
and  student.SId=sc.SId
and sc.CId=course.CId
```

```
+--------+--------+-------+
| sname  | cname  | score |
+--------+--------+-------+
| 赵雷   | 语文   |  80.0 |
| 赵雷   | 数学   |  90.0 |
| 赵雷   | 英语   |  99.0 |
| 钱电   | 英语   |  80.0 |
| 孙风   | 语文   |  80.0 |
| 孙风   | 数学   |  80.0 |
| 孙风   | 英语   |  80.0 |
| 周梅   | 语文   |  76.0 |
| 周梅   | 数学   |  87.0 |
| 郑竹   | 数学   |  89.0 |
| 郑竹   | 英语   |  98.0 |
| 张三   | 英语   | 100.0 |
+--------+--------+-------+
12 rows in set (0.00 sec)
```

30.查询存在不及格的课程

可以用group by 来取唯一，也可以用distinct

```sql
select DISTINCT sc.CId
from sc
where sc.score <60
```

```sql
select course.cname,sc.cid ,count(*)as 不及格总数 
from sc,course 
where sc.score<60 
and sc.cid=course.cid 
group by course.cname;
```

```
+--------+------+-----------------+
| cname  | cid  | 不及格总数      |
+--------+------+-----------------+
| 数学   | 02   |               1 |
| 英语   | 03   |               2 |
| 语文   | 01   |               2 |
+--------+------+-----------------+
3 rows in set (0.00 sec)
```

31.查询课程编号为 01 且课程成绩在 70 分以上的学生的学号和姓名

```sql
select student.sid,student.sname,sc.score 
from student,sc 
where sc.cid='01' 
and sc.score>70 
and student.sid=sc.sid;
```

```
+------+--------+-------+
| sid  | sname  | score |
+------+--------+-------+
| 01   | 赵雷   |  80.0 |
| 03   | 孙风   |  80.0 |
| 05   | 周梅   |  76.0 |
+------+--------+-------+
3 rows in set (0.00 sec)
```

32. 求每门课程的学生人数 
```sql
select cid,count(*)as 学生人数 
from sc 
group by cid;
```
```
+------+--------------+
| cid  | 学生人数     |
+------+--------------+
| 01   |            6 |
| 02   |            6 |
| 03   |            7 |
+------+--------------+
3 rows in set (0.00 sec)
```

33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
```sql
select student.*,sc.score 
from student,sc,course,teacher 
where teacher.tname='张三' 
and teacher.tid=course.tid 
and course.cid=sc.cid   
and sc.sid=student.sid  
order by score desc 
limit 1;
```
34.成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
为了验证，先修改数据。这样张三老师教的02号课就有两个学生同时获得90的最高分了。

```sql
UPDATE sc SET score=90
where sid = "07"
and cid ="02";
```



```sql
select student.*, sc.score, sc.cid from student, teacher, course,sc 
where teacher.tid = course.tid
and sc.sid = student.sid
and sc.cid = course.cid
and teacher.tname = "张三"
and sc.score = (
    select Max(sc.score) 
    from sc,student, teacher, course
    where teacher.tid = course.tid
    and sc.sid = student.sid
    and sc.cid = course.cid
    and teacher.tname = "张三"
);
```
```
+------+--------+---------------------+------+-------+------+
| SId  | Sname  | Sage                | Ssex | score | cid  |
+------+--------+---------------------+------+-------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |  90.0 | 02   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |  90.0 | 02   |
+------+--------+---------------------+------+-------+------+
2 rows in set (0.00 sec)
```

35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

```sql
select *
from sc as t1
where exists(select * from sc as t2 where t1.SId=t2.SId and t1.CId!=t2.CId and t1.score =t2.score )
```

36.查询每门功成绩最好的前两名
```sql
select *
from sc as t1
where (select count(*) from sc as t2 where t1.CId=t2.CId and t2.score >t1.score)<2
ORDER BY t1.CId
```

37.统计每门课程的学生选修人数（超过 5 人的课程才统计）
```sql 
select sc.CId as 课程编号,count(*) as 选修人数
from sc 
GROUP BY sc.CId
HAVING count(*)>5
```

38.检索至少选修两门课程的学生学号 
```sql
select DISTINCT t1.SId
from sc as t1 
where (select count(* )from sc where t1.SId=sc.SId)>=3
```


39. 查询选修了全部课程的学生信息
```sql
select student.*
from sc ,student 
where sc.SId=student.SId
GROUP BY sc.SId
HAVING count(*) = (select DISTINCT count(*) from course )
```

40.查询各学生的年龄，只按年份来算 
```sql
select student.SId as 学生编号,student.Sname  as  学生姓名,TIMESTAMPDIFF(YEAR,student.Sage,CURDATE()) as 学生年龄
from student
```

41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
```Sql
select student.SId as 学生编号,student.Sname  as  学生姓名,TIMESTAMPDIFF(YEAR,student.Sage,CURDATE()) as 学生年龄
from student
```

42.查询本周过生日的学生
```sql
select * from student 
where weekofyear(sage)=weekofyear(curdate());
```
```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
+------+--------+---------------------+------+
1 row in set (0.01 sec)
```

43. 查询下周过生日的学生
```sql
select * from student 
where weekofyear(sage)=weekofyear(curdate())+1;
```

44.查询本月过生日的学生（目前是8月）

```sql
select * from student 
where month(sage)=month(curdate());
```

```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
+------+--------+---------------------+------+
1 row in set (0.00 sec)
```

45.查询上个月过生日的学生（目前是8月）

```sql 
select * from student 
where month(sage)=month(curdate())-1;
```

```
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
1 row in set (0.00 sec)
```



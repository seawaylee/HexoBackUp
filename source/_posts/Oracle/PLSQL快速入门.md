---
title: 数据库编程入门(一)-PL/SQL快速入门 
date: 2017-03-10 00:47:43
tags: [Oracle]
---


## 1.什么是PL/SQL
### 1.1 PL/SQL含义
Procedure Language / SQL 是Oracle对过程化语言的扩展，针对CRUD的过程处理语句，使得SQL语句具有过程处理能力。
### 1.2 为什么要学习PL/SQL
- 比如要给员工按职位不同增加不同的工资，虽然可以用java等编程语言操作数据库进行实现，但是效率远不如使用Oralce原生的变成语言实现。
- 为了学习存储过程和触发器打基础
- Procedure Language / SQL 是Oracle对过程化语言的扩展，针对CRUD的过程处理语句，使得SQL语句具有过程处理能力。

<!--more-->

## 2.最简单的PL/SQL程序
我用的是Oracle11g,客户端连接工具是SQLDeveloper。

```sql
set serveroutput on; — 打开输出开关(默认是关闭的)
declare — 说明部分 (变量，光标，例外)
begin
   -- 程序体
    DBMS_OUTPUT.PUT_LINE('Hello World');
end;
/ -- 表示此段程序执行结束
```

## 3.使用desc查看表结构、视图结构、包结构
![查看如何使用DBMS_OUTPUT.PUTLINE()](http://img.blog.csdn.net/20161122121857763)
## 4.基本数据类型

```sql
char,varchar2,date,number,boolean,long

declare
pnumber number(10,2);
pname varchar2(200);
pdate date;
begin
  pnumber := 1;
  DBMS_OUTPUT.PUT_LINE(pnumber);
  pname := 'NikoBelic';
  DBMS_OUTPUT.PUT_LINE(pname);
  pdate := sysdate;
  DBMS_OUTPUT.PUT_LINE(pdate);
  pdate := pdate + 1;
  DBMS_OUTPUT.PUT_LINE(pdate);
end;
/
```

![这里写图片描述](http://img.blog.csdn.net/20161122122032788)
## 5.if else then的使用

```sql
/*
  判断用户从键盘输入的数字
  1.如何使用if语句
  2.接收一个键盘输入（字符串）
*/

-- 接受一个键盘输入
-- num:地址值，含义是：在改地址上保存了输入的值
accept num PROMPT '请输入一个数字'

declare
  -- 定义变量保存用户输入的数字
  pnum number := &num;
begin
  if pnum = 0 then dbms_output.put_line('您输入的是0');
    elsif pnum = 1 then dbms_output.put_line('您输入的是1');
    elsif pnum = 2 then dbms_output.put_line('您输入的是2');
    else dbms_output.put_line('其他数字');
  end if;
end;
/
```
## 6.循环的使用
### 6.1 while loop 循环

```sql
declare
num number := 1;
begin
  while num <= 10 loop
    dbms_output.put_line(num);
    num := num + 1;
  end loop;
end;
```

### 6.2 loop循环  

```sql
declare
  num number := 1;
begin
  loop
    exit when num > 10;
    dbms_output.put_line(num);
    num := num + 1;
  end loop;
end;
```

### 6.3 for loop循环

```sql
declare
  num number;

begin
  for i in 1..10 loop
    dbms_output.put_line(i);
  end loop;
end;
```
## 7.游标/光标

### 7.1什么是游标
游标可以理解为一个指向数据集合的指针
cursor c_user is select * from tb_user;  -- c_user就是指向user表中所有数据的集合

### 7.2游标的属性
- found: 光标有值
- notfound:光标无
- isopen:光标是否打开
- rowcount:光标影响的行数

### 7.3查看和和修改游标限制
游标数的限制：默认情况下，oracle数据库只允许在同一个会话中打开300个游标。
查看和修改游标限制数:

```commandline
     sqlplus sys/password@192.168.1.100:1521/orcl as sysdba;
     show user;
     show parameter cursor;
```
![这里写图片描述](http://img.blog.csdn.net/20161122122755979)

```commandline
     alter system set open_cursors=400 scope=both;
     -- scope的取值有三个:
          both           配置文件和当前实例都更改 
          memory     只更改当前实例，不更改配置文件
          spfile          只更改参数文件，不更改当前实例，需要重启数据库
```

### 7.4 游标应用示例
示例1

```sql
declare
  -- 定义一个光标
  cursor c_user is select name,salary from tb_user;
  -- 为光标定义对应的变量
  uname TB_USER.NAME%type;
  usalary TB_USER.SALARY%type;
  user tb_user%rowtype;
begin
  -- 打开光标
  open c_user;
  loop
    -- 取一条记录
    fetch c_user into uname,usalary;
    exit when c_user%notfound; -- 光标的属性found和notfound
    dbms_output.put_line(uname || '的薪资是' || usalary);
  end loop;
  -- 关闭光标
  close c_user;
end;
/
```
![这里写图片描述](http://img.blog.csdn.net/20161122123029010)
示例2

```sql
declare
  -- 定义一个光标
  cursor c_user is select * from tb_user;
  -- 为光标定义对应的变量
  uname TB_USER.NAME%type;
  usalary TB_USER.SALARY%type;
  user tb_user%rowtype;
begin
  -- 打开光标
  open c_user;
  loop
    -- 取一条记录
    fetch c_user into user;
    exit when c_user%notfound; -- 光标的属性found和notfound
    dbms_output.put_line(user.name || '的薪资是' || user.salary);
  end loop;
  -- 关闭光标
  close c_user;
end;
/
```

![这里写图片描述](http://img.blog.csdn.net/20161122123038651)

员工涨工资程序

```sql
DECLARE
CURSOR CUR_USER IS SELECT * FROM TB_USER;
USER TB_USER%ROWTYPE;
BEGIN
  OPEN CUR_USER;
  LOOP
    FETCH CUR_USER INTO USER;
    EXIT WHEN CUR_USER%NOTFOUND;
    IF USER.JOB = 'CEO'
      THEN UPDATE TB_USER SET SALARY = SALARY + 100000 WHERE NO = USER.NO;
    ELSIF USER.JOB = 'MANAGER'
      THEN UPDATE TB_USER SET SALARY = SALARY + 50000 WHERE NO = USER.NO;
    ELSE
      UPDATE TB_USER SET SALARY = SALARY + 10000 WHERE NO = USER.NO;
    END IF;
  END LOOP;
  CLOSE CUR_USER;
  COMMIT;
END;
```
(此图有误，但是结果是正确的，懒得再截图了)
![这里写图片描述](http://img.blog.csdn.net/20161122123120640)
## 8.系统例外(异常)
### 8.1 NO_DATA_FOUND

```sql
DECLARE
CURSOR CUR_USER(NUM NUMBER) IS SELECT * FROM TB_USER WHERE NO > NUM;
USER TB_USER%ROWTYPE;

BEGIN
OPEN CUR_USER(2);
LOOP
  FETCH CUR_USER INTO USER;
  EXIT WHEN CUR_USER%NOTFOUND;
  DBMS_OUTPUT.PUT_LINE(USER.NAME);
END LOOP;
CLOSE CUR_USER;
END;

```
![这里写图片描述](http://img.blog.csdn.net/20161122123444426)
### 8.2 TOO_MANY_ROWS
```sql
DECLARE
UNAME TB_USER.NAME%TYPE;

BEGIN
  SELECT NAME INTO UNAME FROM TB_USER WHERE NO > 1;
  EXCEPTION
    WHEN TOO_MANY_ROWS THEN DBMS_OUTPUT.PUT_LINE('TOO FUCKING MANY ROWS!!!!!!');
END;
```
![这里写图片描述](http://img.blog.csdn.net/20161122123509176)

### 8.3 ZERO_DIVIDE
```sql
DECLARE
PNUM NUMBER;

BEGIN
  PNUM := 1 / 0;
  EXCEPTION
    WHEN ZERO_DIVIDE THEN DBMS_OUTPUT.PUT_LINE('FUCKING ZERO CANNOT BE DIVIDE!!!!!');
                          DBMS_OUTPUT.PUT_LINE('UNDERSTAND????SILLY BOY!!!');
    WHEN OTHERS THEN DBMS_OUTPUT.PUT_LINE('OTHER EXCEPTIONS...');
END;
```

### 8.4 VALUE_ER
```sql
DECLARE
UNO NUMBER;

BEGIN
  UNO := 'ABC';
  EXCEPTION
    WHEN VALUE_ERROR THEN DBMS_OUTPUT.PUT_LINE('FUCKING DATA TRANSFER ERROR!!!!');
END; 8.5自定义异常
```
```sql
DECLARE
CURSOR CUR_USER IS SELECT * FROM TB_USER WHERE NO = 100;
USER TB_USER%ROWTYPE;
USER_NOT_FOUND EXCEPTION;
BEGIN
  OPEN CUR_USER;
    FETCH CUR_USER INTO USER;
    IF CUR_USER%NOTFOUND THEN RAISE USER_NOT_FOUND;
    END IF;
    EXCEPTION
      WHEN USER_NOT_FOUND THEN DBMS_OUTPUT.PUT_LINE('THERE IS NO FUCKING USER DATA!!!');
      WHEN OTHERS THEN DBMS_OUTPUT.PUT_LINE('OTHER EXCEPTION');  
  CLOSE CUR_USER; -- 关闭光标这里由于出现了异常所以不会被执行，但是oracle自动启动pomon(process monitor)来自动回收cursor
END;
```
![这里写图片描述](http://img.blog.csdn.net/20161122123525200)

## 9.案例
9.1按照入职年份统计员工数量

```sql
DECLARE
CURSOR CUR_DATE IS SELECT TO_CHAR(HIREDATE,'YYYY') FROM TB_USER;
UDATE VARCHAR2(4);
COUNT10 NUMBER := 0;
COUNT11 NUMBER := 0;
COUNT12 NUMBER := 0;
COUNT13 NUMBER := 0;
COUNT14 NUMBER := 0;
COUNT15 NUMBER := 0;
COUNT16 NUMBER := 0;

BEGIN
  OPEN CUR_DATE;
    LOOP
      FETCH CUR_DATE INTO UDATE;
      EXIT WHEN CUR_DATE%NOTFOUND;
      IF UDATE = '2016' THEN COUNT16 := COUNT16 + 1;
      ELSIF UDATE = '2010' THEN COUNT10 := COUNT10 + 1;
      ELSIF UDATE = '2011' THEN COUNT11 := COUNT11 + 1;
      ELSIF UDATE = '2012' THEN COUNT12 := COUNT12 + 1;
      ELSIF UDATE = '2013' THEN COUNT13 := COUNT13 + 1;
      ELSIF UDATE = '2014' THEN COUNT14 := COUNT14 + 1;
      ELSIF UDATE = '2015' THEN COUNT15 := COUNT15 + 1;
      END IF;
    END LOOP;
  CLOSE CUR_DATE;
  DBMS_OUTPUT.PUT_LINE('TOTAL:'||(COUNT10 + COUNT11 + COUNT12 + COUNT13 + COUNT14 + COUNT15 + COUNT16));
  DBMS_OUTPUT.PUT_LINE('2011:'||COUNT11);
END;
```
![这里写图片描述](http://img.blog.csdn.net/20161122123853478)
### 9.2员工涨工资
从薪资最低的员工开始，每人涨10%的工资，要求工资总数不能超过50000

```sql
DECLARE
CURSOR CUR_USER IS SELECT NO,SALARY FROM TB_USER ORDER BY SALARY;
UNO NUMBER;
USALARY NUMBER;
TOTALMONEY NUMBER;
TOTALCOUNT NUMBER := 0;
INCREAMENTMONEY NUMBER;
TMP NUMBER;

BEGIN
  OPEN CUR_USER;
  SELECT SUM(SALARY) INTO TOTALMONEY FROM TB_USER;
  LOOP
    FETCH CUR_USER INTO UNO,USALARY;
    EXIT WHEN CUR_USER%NOTFOUND;
    INCREAMENTMONEY := USALARY * 0.1;
    TMP := TOTALMONEY + INCREAMENTMONEY;
    EXIT WHEN TMP >= 50000;
    TOTALMONEY := TOTALMONEY + INCREAMENTMONEY;
    TOTALCOUNT := TOTALCOUNT + 1;
    UPDATE TB_USER SET SALARY = SALARY + INCREAMENTMONEY WHERE NO = UNO;
  END LOOP;
  CLOSE CUR_USER;
  DBMS_OUTPUT.PUT_LINE('TOTALCOUNT:'||TOTALCOUNT||'      TOTALMONEY:'||TOTALMONEY);
  COMMIT;
END;
```
![这里写图片描述](http://img.blog.csdn.net/20161122123928026)
### 9.3 按部门统计员工工资区间和部门总工资
创建部门表

```sql
create  table tb_dept(
deptno number,
name varchar(200),
location varchar(200)
);
```

插入数据

```sql
insert into TB_DEPT(DEPTNO,NAME,LOCATION) values(10,'会计部门','北京');
insert into TB_DEPT(DEPTNO,NAME,LOCATION) values(20,'人力部门','上海');
insert into TB_DEPT(DEPTNO,NAME,LOCATION) values(30,'研发部门','广州');
insert into TB_DEPT(DEPTNO,NAME,LOCATION) values(40,'销售部门','深圳');
```

创建统计结果表

```sql
create table tb_msg
(
  deptno number,
  count1 number,
  count2 number,
  count3 number,
  saltotal number
);
```

```sql
DECLARE
CURSOR CUR_DEPT IS SELECT DEPTNO FROM TB_DEPT;
CURSOR CUR_USER(DNO NUMBER) IS SELECT * FROM TB_USER WHERE DEPTNO = DNO;
DEPTNO TB_DEPT.DEPTNO%TYPE;
USER TB_USER%ROWTYPE;
COUNT1 NUMBER := 0;
COUNT2 NUMBER := 0;
COUNT3 NUMBER := 0;
DEPT_TOTAL_SALARY NUMBER := 0;

BEGIN
  OPEN CUR_DEPT;


  LOOP
    FETCH CUR_DEPT INTO DEPTNO;
    EXIT WHEN CUR_DEPT%NOTFOUND;
    COUNT1 := 0;
    COUNT2 := 0;
    COUNT3 := 0;
    DEPT_TOTAL_SALARY := 0;
    OPEN CUR_USER(DEPTNO);
    LOOP

      FETCH CUR_USER INTO USER;
      EXIT WHEN CUR_USER%NOTFOUND;
      DEPT_TOTAL_SALARY := DEPT_TOTAL_SALARY + USER.SALARY;

      IF USER.SALARY <= 3000 THEN COUNT1 := COUNT1 + 1;
      ELSIF USER.SALARY <= 6000 THEN COUNT2 := COUNT2 + 1;
      ELSE COUNT3 := COUNT3 + 1;
      END IF;
    END LOOP;
    CLOSE CUR_USER;
    DBMS_OUTPUT.PUT_LINE('DEPTNO='||DEPTNO||' DEPT_TOTAL_SALARY='||DEPT_TOTAL_SALARY ||'  COUNT1='||COUNT1||'   COUNT2='||COUNT2||'   COUNT3='||COUNT3);
    INSERT INTO TB_MSG(DEPTNO,COUNT1,COUNT2,COUNT3,SALTOTAL) VALUES(DEPTNO,COUNT1,COUNT2,COUNT3,NVL(DEPT_TOTAL_SALARY,0));
  END LOOP;


  CLOSE CUR_DEPT;
  COMMIT; 
END;
```
![这里写图片描述](http://img.blog.csdn.net/20161122124011608)

### 9.4按系名分段统计成绩(<60,60~85,>85) 大学物理课程各个分数段的学生人数，以及各系学生的平均成绩。

各个表的创建和数据插入sql

```sql
drop table sc;
drop table course;
drop table student;
drop table teacher;
drop table dep;


CREATE TABLE DEP
       (DNO NUMBER(2),
        DNAME VARCHAR2(30),
        DIRECTOR NUMBER(4),
        TEL   VARCHAR2(8));


CREATE TABLE TEACHER
       (TNO NUMBER(4),
        TNAME VARCHAR2(10),
        TITLE VARCHAR2(20),
        HIREDATE DATE,
        SAL NUMBER(7,2),
        BONUS  NUMBER(7,2),
    MGR NUMBER(4),             
        DEPTNO NUMBER(2));


CREATE TABLE student
       (sno NUMBER(6),
        sname VARCHAR2(8),
        sex VARCHAR2(2),
        birth   DATE,
        passwd  VARCHAR2(8),
        dno  NUMBER(2));
        
CREATE TABLE course
       (cno VARCHAR2(8),
        cname VARCHAR2(20),
        credit NUMBER(1),
        ctime  NUMBER(2),
        quota  NUMBER(3));
        
CREATE TABLE sc
       (sno NUMBER(6),
        cno  VARCHAR2(8),
        grade NUMBER(3));        
        
alter table dep add (constraint pk_deptno primary key(dno));
alter table dep add(constraint dno_number_check check(dno>=10 and dno<=50));
alter table dep modify(tel default 62795032);
alter table student add (constraint pk_sno primary key(sno));
alter table student add(constraint sex_check check(sex='男' or sex='女'));
alter table student modify(birth default sysdate);
alter table course add (constraint pk_cno primary key(cno));
alter table sc add (constraint pk_key primary key(cno,sno));
alter table teacher add (constraint pk_tno primary key(tno));
alter table sc add (FOREIGN KEY(cno) REFERENCES course(cno));
alter table sc add (FOREIGN KEY(sno) REFERENCES student(sno));
alter table student add (FOREIGN KEY(dno) REFERENCES dep(dno));
alter table teacher add (FOREIGN KEY(deptno) REFERENCES dep(dno));  


INSERT INTO DEP VALUES (10, '计算机系', 9469 , '62785234');
INSERT INTO DEP VALUES (20,'自动化系', 9581 , '62775234');
INSERT INTO DEP VALUES (30,'无线电系', 9791 , '62778932');
INSERT INTO DEP VALUES (40,'信息管理系', 9611, '62785520');
INSERT INTO DEP VALUES (50,'微纳电子系', 2031, '62797686');




INSERT INTO TEACHER VALUES(9468,'CHARLES','PROFESSOR','17-12月-2004',8000,1000,NULL,10);
INSERT INTO TEACHER VALUES(9469,'SMITH','PROFESSOR','17-12月-2004',5000,1000 ,9468,10);
INSERT INTO TEACHER VALUES(9470,'ALLEN','ASSOCIATE PROFESSOR', '20-2月-2003',4200,500,9469,10);
INSERT INTO TEACHER VALUES(9471,'WARD','LECTURER', '22-2月-2004',3000,300,9469,10);
INSERT INTO TEACHER VALUES(9581,'JONES','PROFESSOR ', '2-4月-2003',6500,1000,9468,20);
INSERT INTO TEACHER VALUES(9582,'MARTIN','ASSOCIATE PROFESSOR ','28-9月-2005',4000,800,9581,20);
INSERT INTO TEACHER VALUES(9583,'BLAKE','LECTURER ','1-5月-2006',3000,300,9581,20);
INSERT INTO TEACHER VALUES(9791,'CLARK','PROFESSO', '9-6月-2003',5500,NULL,9468,30);
INSERT INTO TEACHER VALUES(9792,'SCOTT','ASSOCIATE PROFESSOR ','09-12月-2004',4500,NULL,9791,30);
INSERT INTO TEACHER VALUES(9793,'BAGGY','LECTURER','17-11月-2004',3000,NULL,9791,30);
INSERT INTO TEACHER VALUES(9611,'TURNER','PROFESSOR ','8-9月-2005',6000,1000,9468,40);
INSERT INTO TEACHER VALUES(9612,'ADAMS','ASSOCIATE PROFESSO','12-1月-2004',4800,800,9611,40);
INSERT INTO TEACHER VALUES(9613,'JAMES','LECTURER','3-12月-2006',2800,200,9611,40);
INSERT INTO TEACHER VALUES(2031,'FORD','PROFESSOR','3-12月-2005',5500,NULL,9468,50);
INSERT INTO TEACHER VALUES(2032,'MILLER','ASSOCIATE PROFESSO','23-1月-2005',4300,NULL,2031,50);
INSERT INTO TEACHER VALUES(2033,'MIGEAL','LECTURER','23-1月-2006',2900,NULL,2031,50);
INSERT INTO TEACHER VALUES(2034,'PEGGY', 'LECTURER', '23-1月-2007',2500,NULL,2031,50);




insert into student(birth,sno,sname,sex,passwd,dno) values('01-8月 -10',1,'John','男','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('02-8月 -10',2,'Jacob','男','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('03-8月 -10',3,'Michael','男','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('04-8月 -10',4,'Joshua','男','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('05-8月 -10',5,'Ethan','男','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('06-8月 -10',6,'Matthew','男','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('07-8月 -10',7,'Daniel','男','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('08-8月 -10',8,'Chris','男','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('09-8月 -10',9,'Andrew','男','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('10-8月 -10',10,'Anthony','男','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('11-8月 -10',11,'William','男','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('12-8月 -10',12,'Joseph','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('13-8月 -10',13,'Alex','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('14-8月 -10',14,'David','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('15-8月 -10',15,'Ryan','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('16-8月 -10',16,'Noah','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('17-8月 -10',17,'James','男','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('18-8月 -10',18,'Nicholas','男','123456',50);
insert into student(birth,sno,sname,sex,passwd,dno) values('19-8月 -10',19,'Tyler','男','123456',50);
insert into student(birth,sno,sname,sex,passwd,dno) values('20-8月 -10',20,'Logan','男','123456',50);
insert into student(birth,sno,sname,sex,passwd,dno) values('21-8月 -10',21,'Emily','女','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('22-8月 -10',22,'Emma','女','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('23-8月 -10',23,'Madis','女','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('24-8月 -10',24,'Isabe','女','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('25-8月 -10',25,'Ava','女','123456',10);
insert into student(birth,sno,sname,sex,passwd,dno) values('26-8月 -10',26,'Abigail','女','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('27-8月 -10',27,'Olivia','女','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('28-8月 -10',28,'Hannah','女','123456',20);
insert into student(birth,sno,sname,sex,passwd,dno) values('29-8月 -10',29,'Sophia','女','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('30-8月 -10',30,'Samant','女','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('31-8月 -10',31,'Elizab','女','123456',30);
insert into student(birth,sno,sname,sex,passwd,dno) values('01-7月 -10',32,'Ashley','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('02-7月 -10',33,'Mia','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('11-8月 -10',34,'Alexis','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('12-8月 -10',35,'Sarah','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('13-8月 -10',36,'Natalie','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('14-8月 -10',37,'Grace','女','123456',40);
insert into student(birth,sno,sname,sex,passwd,dno) values('15-8月 -10',38,'Chloe','女','123456',50);
insert into student(birth,sno,sname,sex,passwd,dno) values('16-8月 -10',39,'Alyssa','女','123456',50);
insert into student(birth,sno,sname,sex,passwd,dno) values('17-8月 -10',40,'Brianna','女','123456',50);         




insert into course values('c001','数据结构',3,10,100);
insert into course values('c002','Java语言',2,20,100);
insert into course values('c003','数字电路',3,30,100);
insert into course values('c004','模拟电路',3,40,100);
insert into course values('c005','信号与系统',4,50,100);
insert into course values('c006','C语言',3,60,100);
insert into course values('c007','高等数学',5,70,100);
insert into course values('c008','自动原理',1,80,100);
insert into course values('c009','数理方程',3,90,100);
insert into course values('c010','大学语文',2,61,100);
insert into course values('c011','机械制图',3,52,100);
insert into course values('c012','微机原理',3,43,100);
insert into course values('c013','通信基础',3,74,100);
insert into course values('c014','计算机原理',5,35,100);
insert into course values('c015','数据库',3,86,100);
insert into course values('c016','编译原理',2,97,100);
insert into course values('c017','大学物理',2,38,100);
insert into course values('c018','统计基础',4,50,100);
insert into course values('c019','线性代数',4,70,100);
insert into course values('c020','Linux基础',3,60,100);






insert into sc values(6,'c002',60);
insert into sc values(6,'c015',60);
insert into sc values(6,'c010',61);
insert into sc values(27,'c010',65);
insert into sc values(6,'c001',60);
insert into sc values(6,'c011',61);
insert into sc values(6,'c018',70);
insert into sc values(8,'c007',65);
insert into sc values(27,'c020',65);
insert into sc values(27,'c015',65);       
insert into sc values(26,'c015',55);   
insert into sc values(25,'c015',59);      
insert into sc values(1,'c017',65);
insert into sc values(2,'c017',66);
insert into sc values(3,'c017',67);
insert into sc values(4,'c017',68);
insert into sc values(5,'c017',69);
insert into sc values(6,'c017',70);
insert into sc values(7,'c017',71);
insert into sc values(8,'c017',72);
insert into sc values(9,'c017',73);
insert into sc values(10,'c017',74);
insert into sc values(11,'c017',75);
insert into sc values(12,'c017',76);
insert into sc values(13,'c017',77);
insert into sc values(14,'c017',78);
insert into sc values(15,'c017',79);
insert into sc values(16,'c017',80);
insert into sc values(17,'c017',81);
insert into sc values(18,'c017',82);
insert into sc values(19,'c017',83);
insert into sc values(20,'c017',84);
insert into sc values(21,'c017',85);
insert into sc values(22,'c017',86);
insert into sc values(23,'c017',87);
insert into sc values(24,'c017',88);
insert into sc values(25,'c017',89);
insert into sc values(26,'c017',90);
insert into sc values(27,'c017',89);
insert into sc values(28,'c017',88);
insert into sc values(29,'c017',87);
insert into sc values(30,'c017',86);
insert into sc values(31,'c017',85);
insert into sc values(32,'c017',84);
insert into sc values(33,'c017',83);
insert into sc values(34,'c017',82);
insert into sc values(35,'c017',81);
insert into sc values(36,'c017',80);
insert into sc values(37,'c017',79);
insert into sc values(38,'c017',78);
insert into sc values(39,'c017',77);
insert into sc values(40,'c017',76);
commit;
```

PL/SQL程序

```sql
DECLARE
CURSOR CUR_COURSE IS SELECT * FROM COURSE;
COS COURSE%ROWTYPE;

CURSOR CUR_DEPT IS SELECT DNO,DNAME FROM DEP;
DEPART_NO DEP.DNO%TYPE;
DEPART_DNAME DEP.DNAME%TYPE;

--CURSOR CUR_SC(DEPART_NO DEP.DNO%TYPE,COURSE_NO COURSE.CNO%TYPE) IS SELECT * FROM SC WHERE SNO IN (SELECT SNO FROM STUDENT WHERE DNO = DEPART_NO) AND CNO = COURSE_NO;
CURSOR CUR_SC(DEPART_NO DEP.DNO%TYPE,COURSE_NAME COURSE.CNAME%TYPE) IS SELECT * FROM SC WHERE CNO=(SELECT CNO FROM COURSE WHERE CNAME = COURSE_NAME)
                                                                                        AND   SNO IN (SELECT SNO FROM STUDENT WHERE DNO = DEPART_NO);
S_C SC%ROWTYPE;

COUNT1 NUMBER := 0;
COUNT2 NUMBER := 0;
COUNT3 NUMBER := 0;
AVGGRADE NUMBER := 0;
COURSE_DEPT_TOTAL_GRADE NUMBER := 0;
BEGIN
OPEN CUR_COURSE;
LOOP
  FETCH CUR_COURSE INTO COS;
  EXIT WHEN CUR_COURSE%NOTFOUND;

  OPEN CUR_DEPT;
  LOOP
    FETCH CUR_DEPT INTO DEPART_NO,DEPART_DNAME;
    EXIT WHEN CUR_DEPT%NOTFOUND;

    OPEN CUR_SC(DEPART_NO,COS.CNAME);
    COUNT1 := 0;
    COUNT2 := 0;
    COUNT3 := 0;
    COURSE_DEPT_TOTAL_GRADE := 0;
    AVGGRADE := 0;
    SELECT AVG(GRADE) INTO AVGGRADE FROM SC WHERE CNO = (SELECT CNO FROM COURSE WHERE CNAME = COS.CNAME) AND SNO IN (SELECT SNO FROM STUDENT WHERE DNO = DEPART_NO);
    LOOP
      FETCH CUR_SC INTO S_C;
      EXIT WHEN CUR_SC%NOTFOUND;
      IF S_C.GRADE < 60 THEN COUNT1 := COUNT1 + 1;
      ELSIF S_C.GRADE <85 THEN COUNT2 := COUNT2 + 1;
      ELSE COUNT3 := COUNT3 + 1;
      END IF;
      COURSE_DEPT_TOTAL_GRADE := COURSE_DEPT_TOTAL_GRADE + S_C.GRADE;
    END LOOP;
    CLOSE CUR_SC;
--    IF COUNT1 + COUNT2 + COUNT3 = 0 THEN AVGGRADE := 0;
--    ELSE AVGGRADE := COURSE_DEPT_TOTAL_GRADE/(COUNT1+COUNT2+COUNT3);
--    END IF;
--    INSERT INTO TB_SCORREMSG(CNAME,DNAME,COUNT1,COUNT2,COUNT3,AVGSCORE) VALUES(COS.CNAME,DEPART_DNAME,COUNT1,COUNT2,COUNT3,AVGGRADE);
    DBMS_OUTPUT.PUT_LINE('课程：'||COS.CNAME||'  系名：'||DEPART_DNAME||'   低于60分：'||COUNT1||'  60~85分：'||COUNT2||'  85分以上:'||COUNT3||'  系总分 ：'||COURSE_DEPT_TOTAL_GRADE||'   平均分:'||AVGGRADE);

  END LOOP;
  CLOSE CUR_DEPT;
END LOOP;

CLOSE CUR_COURSE;
COMMIT;
END;
```

![这里写图片描述](http://img.blog.csdn.net/20161122124151155)
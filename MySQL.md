---
title: Basic MySQL Syntax
date: 2022-01-31 15:07:40
tags: Technology
categories: Technology
---

## Basic

### Show
```SQL
show databases;
```
show 를 입력해서 데이터베이스 목록을 볼 수 있습니다

### Use
```SQL
use members;
```
use 를 입력해서 members 라는 데이터베이스를 선택할 수 있습니다

### Desc
```SQL
desc members;
```
desc 를 입력해서 members 라는 테이블의 구조를 볼 수 있습니다

## Create , Drop

### 데이터베이스 생성
```SQL
create database db_name;

create database members;
```

### 테이블 생성
```SQL
create table name (column_name) [data_type];

create table members(seq int, name char(20), email(50));
```
데이터베이스는 뒤에 아무것도 나오지 않지만 테이블은 뒤에 컬럼이 나오게됩니다
data_type 에 대한 자료형은 숫자형 , 문자형 , 날짜형이 있습니다

### 데이터베이스 / 테이블 삭제
```SQL
drop [database / table] [name];

drop table members;
```

## Insert , Select , Update , Delete

### 데이터 삽입
```SQL
insert into [table_name] [column] values(data);

insert into members(seq, name, email) values(1, 'admin', 'admin@gmail.com');
insert into members(seq, name, email) values(2, 'moon', 'moon@gmail.com');
insert into members(seq, name, email) values(3, 'test', 'test@gmail.com');
insert into members(seq, name, email) values(4, 'guest', 'guest@gmail.com');
insert into members(seq, name, email) values(5, 'hi', 'hi@gmail.com');
```

### 데이터 조회
```SQL
select [column] from [table] where [condition];

select * from members;
select * from members where seq=1;
```
특정 컬럼을 보고 싶을때 where 구문을 입력합니다

### 데이터 수정
```SQL
update [table] set [column]=[data] where [condition];

update members set name='test1' where seq=3;
```

### 데이터 삭제
```SQL
delete from [table] where [condition];

delete from members where seq=1;
```

## 연산자

### 산술 연산자
<img width="468" alt="스크린샷 2022-01-31 오후 3 31 12" src="https://user-images.githubusercontent.com/94741698/151748882-94d3d349-13f8-484e-bae2-e8c81d10fa99.png">

```SQL
select 1+4;
select 1+4*2;
select * from members where seq=2-1;
```

### 비교 연산자
<img width="301" alt="스크린샷 2022-01-31 오후 3 34 16" src="https://user-images.githubusercontent.com/94741698/151749149-e1246c9f-1156-415c-b6bd-c5e9670321f4.png">

```SQL
select * from members where seq=1;
select * from members where seq > 2;
select * from members where seq >= 2;
select * from members where seq < 3;
select * from members where seq <> 3;
select * from members where seq != 3;
```

### 논리 연산자
<img width="381" alt="스크린샷 2022-01-31 오후 3 36 26" src="https://user-images.githubusercontent.com/94741698/151749319-532ff50b-dfe0-45b4-bb87-3dd3930e8d92.png">

```SQL
select * from members where not seq=1>2;
select * from members where seq=1 or seq=5;
select * from members where seq=1 and name='admin';
```

### 비트 논리 연산자
<img width="499" alt="스크린샷 2022-01-31 오후 3 39 37" src="https://user-images.githubusercontent.com/94741698/151749612-8f68b6db-d8df-468f-83a0-5fa77db07728.png">

```SQL
1 xor 2 = True

select 1&1;
select 2&1;
select 3&1;
select 5&3;
select 5|2;
```

### 연결 연산자
<img width="432" alt="스크린샷 2022-01-31 오후 3 48 55" src="https://user-images.githubusercontent.com/94741698/151750464-8cc605fe-9f3b-4bff-aea9-bbc61a9c2e33.png">

```SQL
select 'te' 'st'
select * from members where name='ad' 'min';
select concat('Jung', 'Moon', 'Yong');
```

### In 연산자
```SQL
[컬럼 / 값] in (값 1, 값 2);

select name, email from members where name in ('admin', 'guest');
select name, email from members where name not in ('admin', 'guest')
```

### LIKE 연산자
<img width="298" alt="스크린샷 2022-01-31 오후 3 53 26" src="https://user-images.githubusercontent.com/94741698/151750868-1f4aa885-fd93-4d0d-b999-406ab26187db.png">

```SQL
select * from members where name like '%admin%';
select * from members where name like 'ad_in';
select * from members where name like '%@gmail.com';
```

## 함수

### 문자열 함수
<img width="311" alt="스크린샷 2022-01-31 오후 3 58 12" src="https://user-images.githubusercontent.com/94741698/151751297-5eb56855-bf31-4516-aa8b-27925dd44645.png">

```SQL
select substring('test',1,1);
select substring('test',2,1);
select substring('test',3,1);
select substring('test',4,1);

select substr('test',1,1);
select substr('test',2,1);
select substr('test',3,1);
select substr('test',4,1);

select mid('test',1,1);
select mid('test',2,1);
select mid('test',3,1);
select mid('test',4,1);
```

### 아스키 코드 변환 함수
<img width="360" alt="스크린샷 2022-01-31 오후 4 00 09" src="https://user-images.githubusercontent.com/94741698/151751481-4c9eb6f8-ab2a-410d-ac7c-3250b45f65fa.png">

```SQL
select ascii('a')
select bin(ascii('a'));

select ascii(substring('test',1,1));
select ascii(substring('test',2,1));
select ascii(substring('test',3,1));
select ascii(substring('test',4,1));

select char(97);
select concat(char(97), char(100), char(109), char(105), char(110));
```

### COUNT 함수
```SQL
select count(column) from [table];

select count(*) from members;
```

### 길이 함수
<img width="231" alt="스크린샷 2022-01-31 오후 4 04 15" src="https://user-images.githubusercontent.com/94741698/151751895-ce029c7c-0de3-424f-a6b6-651a74c35ddb.png">

```SQL
select length('test');
select name, length(email) from members;
```

### 조건문
<img width="263" alt="스크린샷 2022-01-31 오후 4 07 08" src="https://user-images.githubusercontent.com/94741698/151752189-59b6cfae-7f4f-4f20-a51a-14d5cbc0fdbb.png">

```SQL
case when [condition] then [true] else [false] end;

select if(1=1, 1, 2);
select case when 1=1 then 1 else 2 end;

select * from members where seq=(case when 1=1 then 1 else 2 end);
```

## 서브쿼리
최초의 시작된 select 절을 메인쿼리라고 하고 메인쿼리안에 또 다른 하나의 select 절을 서브쿼리라고 합니다

```SQL
select column 1, column 2 from table : 메인쿼리
(select column from table where column=[value]) : 서브쿼리
select column 1, column 2 from table where [column] = (select column from table where column=[value])
```

### 스칼라 서브쿼리
```SQL
select (SubQuery) from (SubQuery)a where [column] = (SubQuery)

select name, (select version()) from members;
select name, (select email from members where seq=a.seq) from members a;
```

select 절에 위치한 서브쿼리는 하나의 레코드와 하나의 컬럼만 반환할 수 있습니다

### 인라인 뷰
```SQL
select (SubQuery) from (SubQuery)a where [column] = (SubQuery)

select * from (select * from members)a;
```

위 예시에서는 a 라는 가상 테이블이 됩니다
서브쿼리가 실행이되고 결과가 반환이 되는데 결과 반환된 테이블 형태가 a 라는 가상 테이블이 형성이 됩니다

from 절에 위치한 서브쿼리는 뷰는 하나 이상 테이블들을 기반으로 만들어진 테이블로써 물리적인 데이터를 저장하고 있지 않고 논리적으로 구현되어 있는 가상 테이블이라고 합니다

### 일반 서브쿼리
```SQL
select (SubQuery) from (SubQuery)a where [column] = (SubQuery)

select * from members where seq=(select 1);
select * from members where seq=(select seq from members where id='guest');
```

where 절에서 사용되는 서브쿼리는 연산자에 따라서 반환되는 레코드 개수와 컬럼 수가 다르게 됩니다

### 서브쿼리 종류
```SQL
select name, email from member where id=(select id from bbs where idx=192)

select name, email from members where id in(select id from bbs)
```

단일행 서브쿼리는 하나의 레코드만 반환해야 하는 서브쿼리입니다

in 연산자안에 서브쿼리가 사용가능합니다
in 연산자를 통해 id 라는 컬럼과 select id from bbs 에 반환되는 다수의 레코드와 비교를 하게됩니다

## ORDER BY 절을 이용한 레코드 정렬
```SQL
select column 1, column 2 from table order by column [asc / desc];

select * from members order by 1;
select * from members order by 2;
select * from members order by 3;

select * from members order by seq asc;
select * from members order by seq desc;
select * from members order by seq, name;
```

## 레코드 출력 개수 제한
<img width="229" alt="스크린샷 2022-01-31 오후 4 22 34" src="https://user-images.githubusercontent.com/94741698/151753722-3a0aee20-f7a0-43e2-afaa-fa2adcca4e36.png">

```SQL
select column 1, column 2 from table limit [row_count];
select column 1, column 2 from table limit [offset], [row_count];

select * from members limit 1;
select * from members limit 0,1;
```

## GROUP BY
![SQL](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a821c669-fe02-4d67-8db7-88228eced336/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220131%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220131T154112Z&X-Amz-Expires=86400&X-Amz-Signature=5fef674b78ae35cd7d4594aabea2eefb59c15ce2756242ab8536544a633609a6&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

```SQL
select name from test group by name;
select name, count(name), sum(quantity) from test group by name;
select name, count(name) from test group by name having count(name) = 1;
```

## UNION
```SQL
select id from users union select password from users;
```

## Problem
1. users 테이블의 id 가 `admin` 인 레코드를 출력하라
`flag`: select * from users where id='admin';
2. users 테이블의 id 가 `admin` 인 레코드의 이메일을 출력하라
`flag`: select email from users where id='admin';
3. users 테이블의 id 가 `admin` 가 아닌 레코드의 password, email 를 출력하라
`flag`: select password, email from id != 'admin';
4. users 테이블의 id 가 `admin` 또는 `guest` 인 레코드를 출력하라
`flag`: select * from users where id in ('admin', 'guest');
5. users 테이블의 id 가 `admin` 또는 `guest` 가 아닌 레코드를 출력하라
`flag`: select * from users where id not in ('admin', 'guest');
6. users 테이블의 no 가 2 이상 5 미만인 레코드를 출력하라
`flag`: select * from users where no >= 2 and no < 5;
7. users 테이블의 no 가 1 또는 2 또는 3 인 레코드를 출력하라
`flag`: select * from users where no in (1, 2, 3);
8. users 테이블의 id 가 `ad` 로 시작하는 레코드를 출력하라
`flag`: select * from users where id like 'ad%';
9. users 테이블의 email 이 `@gmail.com` 를 주소로 사용하는 레코드를 출력하라
`flag`: select * from users where email like '%@gmail.com';
10. users 테이블의 id 레코드를 출력하라 단 `name : 사용자명` 형식으로 출력하라
`flag`: select concat('name : ', id) from users;
11. users 테이블의 id 값의 앞 3 문자만 출력 후 뒤에는 `###` 가 붙도록 출력하라
`flag`: select concat(substring(id,1,3), '###') from users;
12. users 테이블의 레코드를 no 컬럼 기준으로 내림차순 으로 정렬 후 2 개의 레코드를 출력하라
`flag`: select * from users order by no desc limit 2;

## 메타 데이터


* * *

## Reference
Reference : https://www.inflearn.com/course/SQL-%EC%9D%B8%EC%A0%9D%EC%85%98-%EA%B3%B5%EA%B2%A9-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95/dashboard
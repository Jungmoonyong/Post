---
title: Dreamhack Relational DBMS
date: 2022-02-19 23:37:52
tags: Dreamhack
categoeis: Dreamhack
---

## Summary
- 데이터베이스 : 데이터가 저장되는 공간
- DBMS : 데이터베이스를 관리하는 어플리케이션
- RDBMS : 테이블 형태로 저장하는 관계형 DBMS
- SQL : RDBMS 와 상호작용할 때 사용되는 언어

한번쯤 다이어리에 일정과 메모를 적고 학교에서 들은 강의를 노트에 기록해 본 경험이 있을겁니다
새로운 일정이 생기면 다이어리를 펼쳐 날짜에 맞게 기록을 하고 , 일정이 변경되거나 취소된다면 기록한 일정을 수정하거나 삭제할 수 있습니다

일상 생활에서 생기는 다양한 정보를 기록하기 위헤 다이어리와 노트를 사용하듯이 , 컴퓨터도 정보를 기록하기 위해 데이터베이스를 사용합니다

그리고 데이터베이스를 관리하는 애플리케이션을 DataBase Management System 이라고 합니다

현대 웹 서비스는 다양한 기능을 제공합니다
예를 들어 드림핵에 회원가입을 하고 , 로그인해 포럼에 글을 쓸 수 있습니다

웹 서버는 데이터베이스를 사용하여 이용자의 요청에 맞게 회원 정보와 포럼 글을 저장하고 조회 , 수정합니다

다수의 사람들이 데이터를 생성하고 , 참조하고 , 수정하는 웹 서버의 특수한 환경에 가장 적합한 자료구조로 채택된 것이 데이터베이스이며 , 최근의 웹 서비스에는 거의 필수적으로 포함되어야 합니다

### 데이터베이스 관리 시스템
웹 서비스는 데이터베이스에 정보를 저장하고 , 이를 관리하기 위해 DataBase Management System 을 사용합니다

DBMS 는 데이터베이스에 새로운 정보를 기록하거나 , 기록된 내용을 수정 , 삭제하는 역할을 합니다

DBMS 는 다수의 사람이 동시에 데이터베이스에 접근할 수 있고 , 웹 서비스의 검색 기능과 같이 복잡한 요구사항을 만족하는 데이터를 조회할 수 있다는 특징이 있습니다

DBMS 는 크게 관계형과 비관계형을 기준으로 분류하며 , 다양한 종류의 DBMS 가 존재합니다
각 종류별 대표적인 DBMS 는 우측 표에 정리되어 있습니다

두 DBMS 의 가장 큰 차이로 관계형은 행과 열의 집합인 테이블 형식으로 데이터를 저장하고 , 비관계형은 테이블 형식이 아닌 키 - 값 형태로 값을 저장합니다

- Relational ( 관계형 ) : MySQL , MariaDB , PostgreSQL , SQLite
- Non - Relational ( 비관계형 ) : MongoDB , CouchDB , Redis

## Relational DBMS
Relational DataBase Management System 는 1970 년에 Codds 가 12 가지 규칙을 정의하여 생성한 데이터베이스 모델입니다

RDBMS 는 행 ( Row ) 과 열 ( Column ) 의 집합으로 구성된 테이블의 묶음 형식으로 데이터를 관리하고 , 테이블 형식의 데이터를 조작할 수 있는 관계 연산자를 제공합니다

Codds 는 12 가지 규칙을 정의했지만 실제로 구현된 RDBMS 들은 12 가지 규칙을 모두 따르지는 않았고 , 최소한의 조건으로 앞의 두 조건을 만족하는 DBMS 를 RDBMS 라고 부르게 되었습니다

RDBMS 에서 관계 연산자는 Structured Query Language 라는 쿼리 언어를 사용하고 , 쿼리를 통해 테이블 형식의 데이터를 조작합니다

학교에서 사용하는 정보들을 사용해 RDBMS 의 예시를 들어보겠습니다

학교에 여러 정보를 담은 테이블이 있습니다

예를 들어 학생의 정보를 담은 학생 명부 , 출석부 , 성젹표 , 생활기록부 등이 테이블 형태로 관리됩니다
그리고 각 테이블의 정보를 사용할 때에는 학번 을 참조해 사용합니다

### 학생 명부
- 학번 ( 문자열 ) : 20211234 , 20211235
- 이름 ( 문자열 ) : 드림이 , 오리
- 생년월일 ( 날짜 ) : 2020.04.01 , 2016.01.01
- 번호 ( 문자열 ) : 010-1337-1337 , 010-8864-1337

### 드림과목 출석부
- 학번 ( 문자열 ) : 20211234 , 20211235
- 04/05(ENUM) : 출석 , 지각
- 04/12(ENUM) : 공결 , 결석
- 04/19(ENUM) : 출석 , 출석

## SQL
Structured Query Language 는 RDBMS 의 데이터를 정의하고 질의 , 수정 들을 하기 위해 고안된 언어입니다

SQL 은 구조화된 형태를 가지는 언어로 어플리케이션이 DBMS 와 상호작용할 때 사용됩니다

SQL 은 사용 목적과 행위에 따라 다양한 구조가 존재하며 대표적으로 아래와 같이 구분됩니다

- DDL ( Data Definition Language ) : 데이터를 정의하기 위한 언어입니다
데이터를 저장하기 위한 스키마 , 데이터베이스의 생성 / 수정 / 삭제 등의 행위를 수행합니다
- DML ( Data Manipulation Language ) : 데이터를 조작하기 위한 언어입니다
실제 데이터베이스 내에 존재하는 데이터에 대해 조회 / 저장 / 수정 / 삭제 등의 행위를 수행합니다
- DCL ( Data Control Language ) : 데이터베이스의 접근 권한 등의 설정을 하기 위한 언어입니다
데이터베이스 내에 이용자의 권한을 부여하기 위한 GRANT 와 권한을 박탈하는 REVOKE 가 대표적입니다

## DDL
웹 어플리케이션은 SQL 을 사용해서 DBMS 와 상호작용을 하며 데이터를 관리합니다
RDBMS 에서 사용하는 기본적인 구조는 데이터베이스 -> 테이블 -> 데이터 구조 입니다

데이터를 다루기 위해 데이터베이스와 테이블을 생성해야 하며 , DDL 을 사용해야 합니다
DDL 의 CREATE 명령을 사용해 새로운 데이터베이스 또는 테이블을 생성할 수 있습니다

### 데이터베이스 생성
Dreamhack 이라는 데이터베이스를 생성하는 쿼리문입니다

```SQL
CREATE DATABASE Dremahack;
```

### 테이블 생성
앞서 생성한 데이터베이스에 Board 테이블을 생성하는 쿼리문입니다

```SQL
USE Dreamhack;
# Borad 이름의 테이블 생성
CREATE TABLE Board(
	idx INT AUTO_INCREMENT,
	boardTitle VARCHAR(100) NOT NULL,
	boardContent VARCHAR(2000) NOT NULL,
	PRIMARY KEY(idx)
)
```

## DML
생성된 테이블에 데이터를 추가하기 위해 DML 을 사용합니다
다음은 새로운 데이터를 생성하는 INSERT , 데이터를 조회하는 SELECT , 그리고 데이터를 수정하는 UPDATE 의 예시입니다

### 테이블 데이터 생성
Board 테이블에 데이터를 삽입하는 쿼리문입니다

```SQL
INSERT INTO 
  Board(boardTitle, boardContent, createdDate) 
Values(
  'Hello', 
  'World !',
  Now()
);
```

### 테이블 데이터 조회
Board 테이블의 데이터를 조회하는 쿼리문입니다

```SQL
SELECT 
  boardTitle, boardContent
FROM
  Board
Where
  idx=1;
```

### 테이블 데이터 변경
Board 테이블의 컬럼 값을 변경하는 쿼리문입니다

```SQL
UPDATE Board SET boardContent='DreamHack!' 
  Where idx=1;
```

## Quiz
> 1. Redis
2. DBL
3. SELECT

* * * 

## Reference
Reference : https://dreamhack.io
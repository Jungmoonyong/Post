---
title: Dreamhack SQL DML
date: 2022-02-22 23:22:57
tags: Dreamhack
categories: Dreamhack
---

## Summary
기초 과정에서 알아보았듯이 SQL에서는 Data Definition Language (DDL), Data Manipulation Language (DML), Data Control Language (DCL) 총 세 가지의 질의 언어를 제공합니다

이번 강의에서 알아볼 Data Manipulation Language (DML)은 데이터베이스에서 데이터를 조회, 추가, 삭제, 수정을 수행하는 구문입니다

우리가 웹 서비스에서 이용하는 대다수의 기능은 DML을 통해 처리한다 해도 과언이 아닐 정도로 가장 많이 사용됩니다 DML 구문을 사용하는 목적과 형태를 알고 있다면 SQL Injection 공격을 보다 쉽고 빠르게 이해할 수 있습니다

그렇다면 각각의 구문을 사용 예시와 함께 알아보도록 하겠습니다

## SELECT
SELECT 는 데이터를 조회하는 구문입니다

Figure1 은 MYSQL 공식 페이지에서 제공하는 SELECT 구문의 예시입니다

Figure1 SELECT 구문

```SQL
# mysql SELECT Statement https://dev.mysql.com/doc/refman/8.0/en/select.html
SELECT
    select_expr [, select_expr] ...
FROM table_references
WHERE where_condition
[GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
[ORDER BY {col_name | expr | position} [ASC | DESC], ... [WITH ROLLUP]]
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

다음은 SELECT 구문의 예시를 자세히 설명한 표입니다
SELECT 구문에서 특정 조건을 맍족하는 데이터를 찾기 위한 where , 조회한 결과를 정렬하거나 특정 부문만을 확인하기 위한 order by , limit 등이 있습니다

- SELECT : 해당 문자열을 시작으로 , 조회하기 위한 표현식과 컬럼들에 대해 정의합니다
- FROM 데이터를 조회할 테이블의 이름입니다
- WHERE : 조회할 데이터의 조건입니다
- ORDER BY : 조회한 결과를 원하는 컬럼 기준으로 정렬합니다
- LIMIT : 조회한 결과에서 행의 갯수와 오프셋을 지정합니다

### SELECT 사용 예시
Figure2는 SELECT 구문을 사용한 예시입니다

쿼리를 살펴보면 먼저 FROM 절을 사용해 board 테이블의 uid, title, boardcontent 데이터를 검색합니다

그리고 WHERE 절을 사용해 boardcontent 데이터에 abc 문자가 포함되어 있는지 찾습니다

이렇게 찾은 데이터를 ORDER BY 절을 통해 uid를 기준으로 내림차순 정렬한 후, 5 개의 행을 결과로 반환합니다

Figure2 SELECT 구문 사용 예시

```SQL
SELECT
    uid, title, boardcontent
FROM boards
WHERE boardcontent like '%abc%'
ORDER BY uid DESC
LIMIT 5 
```

## INSERT
INSERT 는 데이터를 추가하는 구문입니다

Figure3 은 MYSQL 공식 페이지에서 제공하는 INSERT 구문의 예시입니다

Figure3 INSERT 구문

```SQL
# mysql INSERT Statement https://dev.mysql.com/doc/refman/8.0/en/insert.html
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    { {VALUES | VALUE} (value_list) [, (value_list)] ...
      |
      VALUES row_constructor_list
    }
```

다음은 INSERT 구문의 예시를 자세히 설명한 표입니다

INSERT 구문에서 데이터를 추가할 테이블과 컬럼을 정의하는 INTO 추가할 데이터를 정의하는 VALUES 등이 있습니다

- INSERT : 해당 문자열을 시작으로 추가할 테이블과 데이터를 정의합니다
- INTO : 데이터를 추가할 테이블의 이름과 컬럼을 정의합니다
- VALUES : INTO 절에서 정의한 테이블의 컬럼에 명시한 데이터들을 추가합니다

### INSERT 사용 예시
Figure4,5 는 INSERT 구문을 사용한 예시입니다

먼저 Figure4 의 쿼리를 살펴보면 INTO 절을 사용해 추가할 테이블과 컬럼을 각각 board와 title, boardcontent로 명시합니다

그리고 VALUES 절을 통해 INTO 절에서 명시한 테이블의 각각의 컬럼에 title 1,  content 1 과 title 2, content 2 데이터를 추가합니다

이처럼 쿼리 한 줄로 두 개 이상의 데이터를 추가할 수 있습니다

Figure4 INSERT 구문 사용 예시

```SQL
INSERT 
    INTO boards (title, boardcontent)
    VALUES ('title 1', 'content 1'), ('title 2', 'content 2');
```

Figure5와 같이 서브 쿼리를 통해 다른 테이블에 존재하는 데이터를 추가할 수 있습니다

쿼리를 살펴보면 이전과 같이 추가할 테이블과 컬럼을 각각 board와 title, boardcontent로 명시합니다

그리고 데이터를 추가하되 boardcontent에는 SELECT 구문을 실행해 users 테이블에 uid 컬럼의 값이 admin 인 행을 찾고, 해당하는 행의 upw 데이터를 추가합니다

이처럼 서브쿼리를 사용하면 다른 테이블에 있는 데이터를 그대로 가져와 삽입할 수 있습니다

Figure5 INSERT 구문 응용 예시

```SQL
INSERT 
    INTO boards (title, boardcontent)
    VALUES ('title 1', (select upw from users where uid='admin'));
```

## UPDATE
UPDATE 는 데이터를 수정하는 구문입니다

Figure6 은 MYSQL 공식 페이지에서 제공하는 UPDATE 구문의 예시입니다

Figure6 INSERT 구문

```SQL
# mysql UPDATE https://dev.mysql.com/doc/refman/8.0/en/update.html
UPDATE [LOW_PRIORITY] [IGNORE] table_references
    SET assignment_list
    [WHERE where_condition]
```

다음은 UPDATE 구문의 예시를 자세히 설명한 표입니다

UPDATE 구문에서 수정할 컬럼과 데이터를 정의하는 SET, 수정할 데이터의 조건을 정의하는 WHERE 등이 있습니다

- UPDATE : 해당 문자열을 시작으로 수정할 테이블을 정의합니다
- SET : 수정할 컬럼과 데이터를 정의합니다
- WHERE : 수정할 행의 조건을 정의합니다

### UPDATE 사용 예시
Figure7 은 UPDATE 구문을 사용한 예시입니다

쿼리를 살펴보면 boards 테이블의 boardcontent 컬럼을 “update content 2” 문자열로 수정합니다. 수정될 데이터의 조건은 title 컬럼의 값이 “title 1”인 행인 것을 WHERE 절을 통해 알 수 있습니다

Figure7 UPDATE 구문 사용 예시

```SQL
UPDATE boards
    SET boardcontent = "update content 2"
    WHERE title = 'title 1';
```

## DELETE
DELETE는 데이터를 삭제하는 구문입니다

Figure8은 MYSQL 공식 페이지에서 제공하는 DELETE 구문의 예시입니다

Figure8 DELETE 구문

```SQL
# mysql DELETE https://dev.mysql.com/doc/refman/8.0/en/delete.html
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias]
    [PARTITION (partition_name [, partition_name] ...)]
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]
```

다음은 DELETE 구문의 예시를 자세히 설명한 표입니다

DELETE 구문에서 삭제할 테이블을 정의하는 FROM, 삭제할 데이터의 조건을 정의하는 WHERE 절이 있습니다

- DELETE : 해당 문자열을 시작으로 이후에 삭제할 테이블을 정의합니다
- FROM : 삭제할 테이블을 정의합니다
- WHERE : 삭제할 행의 조건을 정의합니다

## SQL Injection 모듈 실습 3
SQL Injection 취약점이 존재하는 모듈입니다

주어진 목표를 확인하고 이를 만족하는 쿼리를 실행해보시기 바랍니다

게시판 서비스 입니다.
SQL Injection으로 admin 테이블에 있는 비밀번호를 찾으세요

```
SQL Injection으로 Insert 되는 값을 제어할 수 있습니다
e.g.
 - id: ", 1234)--
   insert into board (uid, text) values ("", 1234)--", "");
 - pw: "), (1,2)--
   insert into board (uid, text) values ("", ""), (1,2)--");
```

```
Insert 쿼리의 VALUE 값에서 서브 쿼리를 사용할 수 있습니다.
서브 쿼리를 통해 admin 테이블에 값을 가져올 수 있습니다
e.g.
 - id: ", (select upw from admin))--
   insert into board (uid, text) values ("", (select upw from admin))--", "");

apple
```

## SQL DML Quiz
> 1. C
2. A
3. C

* * * 

## Reference
Reference : https://dreamhack.io

---
title: MySQL Metadata
date: 2022-02-05 23:43:46
tags: Technology
categories: Technology
---

## 메타 데이터
데이터의 종류가 너무 많아질 경우 이 또한 목록화 시킬 필요가 생기는데 이러한 요구에 맞춰 생겨난게 바로 메타 데이터입니다
메타 데이터는 데이터베이스와 같이 빅데이터를 다뤄야 하는 경우에 효율적인 정보처리를 위해 만든 것으로 쉽게 말하자면 정보에 대한 정보입니다

MySQL 은 메타 데이터들을 종류별로 묶어 테이블을 만들었고 이 테이블을 모아 데이터베이스를 만들었는데 그 데이터베이스가 information_schema 입니다

```SQL
@@version
```
MySQL 버전 정보를 알 수 있습니다

```SQL
select user();
select system_user();
select current_user;
```
current_user 은 MySQL 서버에 접속한 유저목록을 출력합니다

table_schema : Database name
table_name : Table name
column_name : Column name

```SQL
show databases;
select schema_name from information_schema.schemata;
```
위 구문을 통해 MySQL 서버에 어떤 데이터베이스들이 있는 알 수 있습니다
show databases; 와 같은 내용을 출력합니다

애플리케이션에서 데이터를 처리할 때 select 문은 써도 show 문은 쓰질 않으니 그 동안에는 전체 데이터베이스 목록 만큼은 쉽게 알 수 없다고 생각하고 있었을 지 모릅니다

그러나 애플리케이션에 select 문을 쓰는 지점에서 취약점이 있다면 위와 같은 방법으로 희생자 서버에서 어떠한 데이터베이스들이 있는지 조회할 수 있게 됩니다

```SQL
show tables;
use database;
select table_schema, table_name from information_schema.tables where table_schema='데이터베이스 명';
```
위 구문을 통해 어떤 테이블이 있는지 알 수 있습니다
user 데이터베이스 명; 와 show tables; 와 같은 내용을 출력합니다

```SQL
select database()
```
애플리케이션이 사용하는 데이터베이스를 보려면 다음과 같은 구문을 사용할 수 있습니다

```SQL
select table_name, column_name from information_schema.columns where table_schema='데이터베이스 명';
```
위 구문을 통해 어떤 컬럼이 있는지 알 수 있습니다

```SQL
select column_name from information_schema.columns where table_name='테이블 명';
```
만약 특정 테이블에 있는 컬럼들만 알려면 위 구문을 통해 알 수 있습니다

```SQL
select grantee, privilege_type, is_grantable from information_schema.user_privileges;
```
위 구문을 통해 어떤 사용자가 어떤 권한을 가지는지 알 수 있습니다

## Reference
Reference : https://poqw.tistory.com/24
# PostgreSQL Note
## sub query 적용 비교
postgresql에서 필터를 적용한 쿼리를 실행할 때 서브 쿼리가 포함된 경우가 더 빠를까? 보통은 서브 쿼리가 들어가면 속도가 더 느려진다.  
아래 예시처럼 실제 시간 차이가 나는지 간단하게 확인해보았다. 현재 테이블 크기는 table1의 경우 약 4.2G로 다수의 `entity_id` 20년치 데이터가 들어있다.  
추가적으로 Query Plan을 함께 확인하여 쿼리 동작이 어떻게 이루어지는지 확인하였다.

[ 예시 ]
- table1, table2의 `entity_id`라는 칼럼을 기준으로 inner join
- 조인 대상을 서브쿼리로 적용하느냐에 따라 구분하여 실행

### join with sub query
```sql
SELECT base_date, table2.unique_id, agg_count FROM table1 
INNER JOIN ( select entity_id, unique_id from table2 
		  where entity_id in ('xxxx0', 'xxxx1', 'xxxx2', 'xxxx3', 'xxxx4', 'xxxx5', 'xxxx6', 'xxxx7', 'xxxx8', 'xxxx9', 'xxxx10', 'xxxx11', 'xxxx12', 'xxxx13', 'xxxx14', 'xxxx15', 'xxxx16', 'xxxx17', 'xxxx18', 'xxxx19', 'xxxx20', 'xxxx21', 'xxxx22', 'xxxx23', 'xxxx24', 'xxxx25', 'xxxx26', 'xxxx27', 'xxxx28', 'xxxx29', 'xxxx30', 'xxxx31', 'xxxx32', 'xxxx33', 'xxxx34', 'xxxx35', 'xxxx36', 'xxxx37', 'xxxx38', 'xxxx39', 'xxxx40', 'xxxx41', 'xxxx42', 'xxxx43', 'xxxx44', 'xxxx45', 'xxxx46', 'xxxx47', 'xxxx48', 'xxxx49', 'xxxx50', 'xxxx51', 'xxxx52', 'xxxx53', 'xxxx54') ) table2
ON table1.entity_id = table2.entity_id 

/*
"Gather  (cost=600025.39..792357.91 rows=1257562 width=22)"
"  Workers Planned: 2"
"  ->  Parallel Hash Join  (cost=599025.39..665601.71 rows=523984 width=22)"
"        Hash Cond: (table2.entity_id = table1.entity_id)"
"        ->  Parallel Seq Scan on table2  (cost=0.00..12118.46 rows=337 width=17)"
"              Filter: (entity_id = ANY ('{xxxx0, xxxx1, xxxx2, xxxx3, xxxx4, xxxx5, xxxx6, xxxx7, xxxx8, xxxx9, xxxx10, xxxx11, xxxx12, xxxx13, xxxx14, xxxx15, xxxx16, xxxx17, xxxx18, xxxx19, xxxx20, xxxx21, xxxx22, xxxx23, xxxx24, xxxx25, xxxx26, xxxx27, xxxx28, xxxx29, xxxx30, xxxx31, xxxx32, xxxx33, xxxx34, xxxx35, xxxx36, xxxx37, xxxx38, xxxx39, xxxx40, xxxx41, xxxx42, xxxx43, xxxx44, xxxx45, xxxx46, xxxx47, xxxx48, xxxx49, xxxx50, xxxx51, xxxx52, xxxx53, xxxx54}'::bpchar[]))"
"        ->  Parallel Hash  (cost=433834.17..433834.17 rows=8997617 width=19)"
*/
```
```
Successfully run. Total query runtime: 19 secs 638 msec.
1472265 rows affected.
```
### join without sub query
```sql
SELECT base_date, table2.unique_id, agg_count FROM table1 
INNER JOIN table2 
ON table1.entity_id = table2.entity_id 
 WHERE table1.entity_id in ('xxxx0', 'xxxx1', 'xxxx2', 'xxxx3', 'xxxx4', 'xxxx5', 'xxxx6', 'xxxx7', 'xxxx8', 'xxxx9', 'xxxx10', 'xxxx11', 'xxxx12', 'xxxx13', 'xxxx14', 'xxxx15', 'xxxx16', 'xxxx17', 'xxxx18', 'xxxx19', 'xxxx20', 'xxxx21', 'xxxx22', 'xxxx23', 'xxxx24', 'xxxx25', 'xxxx26', 'xxxx27', 'xxxx28', 'xxxx29', 'xxxx30', 'xxxx31', 'xxxx32', 'xxxx33', 'xxxx34', 'xxxx35', 'xxxx36', 'xxxx37', 'xxxx38', 'xxxx39', 'xxxx40', 'xxxx41', 'xxxx42', 'xxxx43', 'xxxx44', 'xxxx45', 'xxxx46', 'xxxx47', 'xxxx48', 'xxxx49', 'xxxx50', 'xxxx51', 'xxxx52', 'xxxx53', 'xxxx54')

/*
"Gather  (cost=6680.84..358967.87 rows=74263 width=22)"
"  Workers Planned: 6"
"  ->  Parallel Hash Join  (cost=5680.84..350541.57 rows=12377 width=22)"
"        Hash Cond: (table1.entity_id = table2.entity_id)"
"        ->  Parallel Index Scan using table1_pkey on table1  (cost=0.56..344723.63 rows=1565 width=19)"
"              Index Cond: ((base_date >= '2000-01-01 00:00:00'::timestamp without time zone) AND (base_date <= '2005-06-30 00:00:00'::timestamp without time zone))"
"              Filter: (entity_id = ANY ('{xxxx0, xxxx1, xxxx2, xxxx3, xxxx4, xxxx5, xxxx6, xxxx7, xxxx8, xxxx9, xxxx10, xxxx11, xxxx12, xxxx13, xxxx14, xxxx15, xxxx16, xxxx17, xxxx18, xxxx19, xxxx20, xxxx21, xxxx22, xxxx23, xxxx24, xxxx25, xxxx26, xxxx27, xxxx28, xxxx29, xxxx30, xxxx31, xxxx32, xxxx33, xxxx34, xxxx35, xxxx36, xxxx37, xxxx38, xxxx39, xxxx40, xxxx41, xxxx42, xxxx43, xxxx44, xxxx45, xxxx46, xxxx47, xxxx48, xxxx49, xxxx50, xxxx51, xxxx52, xxxx53, xxxx54}'::bpchar[]))"
"        ->  Parallel Hash  (cost=4249.57..4249.57 rows=114457 width=17)"
"              ->  Parallel Seq Scan on table2  (cost=0.00..4249.57 rows=114457 width=17)"
*/
```
```
Successfully run. Total query runtime: 12 secs 407 msec.
1472265 rows affected.
```

요약하면..
- sub query 없이 join하는게 더 빠름
- filter의 역할이 중요하게 작동함( 전체 entity_id 18377개인데 55개( 0.3% )에 해당하는 경우만 추출하도록 하여 Index Scan이 효과적으로 적용됨 )
- filter의 역할이 미미해질 경우( 전체 테이블에서 큰 비율로 데이터를 로드해야 하는 경우,..) Seq Scan이 적용될 것(Parallel Seq Scan 인 경우 프로세스를 여러 개 띄워 병렬로 쿼리를 수행하나 CPU 사용량이 급증할 수 있음)

결과적으로 왠만하면 서브 쿼리는 적용하지 않는게 나을 것 같다..!

## NUMERIC data type
*[출처] https://www.geeksforgeeks.org/postgresql-numeric-data-type/*

postgresql에서  `NUMERIC` type이 지원된다. syntax는 아래와 같다. 

```sql
NUMERIC(precision, scale)

/*
Precision: 전체 숫자 길이
Scale: fraction( 소숫점 )의 길이
*
```
예를 들어,

```sql
CREATE TABLE IF NOT EXISTS products (
    id serial PRIMARY KEY,
    name VARCHAR NOT NULL,
    price NUMERIC (5, 2)
);
```
테이블을 생성하면, 아래와 같은 데이터를 삽입할 때
```sql
INSERT INTO products (name, price)
VALUES
    ('Phone', 100.2157), 
    ('Tablet', 300.2149);
```
[ 결과 ]
```sql
id | name  | price
-------------------
 1 | Phone | 100.22
 2 | Table | 300.21
```
전체 길이 5에서 소숫점 2만큼만 반영된다.

---
## inner join vs. pandas.merge(how='inner')
Postgresql 서버에서 inner join을 적용하는게 나을지, python에서 데이터를 메모리에 올려 `pandas.merge`로 적용하는게 나을지 실험해보았다. 데이터를 전구간으로 한번에 쿼리 요청하면 프로세스가 죽어있는 경우가 발생하여 연도별로 나눠서 받았다.

결과를 정리하면, 
- 기존에 병합이 필요한 부분은 `pd.merge`를 적용하였는데 약 1~2 GB 데이터 받는데 대략 4060초 소요되었음. 
- `pd.merge` 대신 PostgreSQL에서 대신 쿼리로 작업을 수행하여 `inner join`을 적용하였는데 9172초로 대략 2배 더 발생했음. 


## 도커로 설치
```console
$ sudo docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD={password} -d postgres

enable to find image 'postgres:latest' locally
latest: Pulling from library/postgres
b4d181a07f80: Pull complete 
46ca1d02c28c: Pull complete 
a756866b5565: Pull complete 
36c49e539e90: Pull complete 
664019fbcaff: Pull complete 
727aeee9c480: Pull complete 
796589e6b223: Pull complete 
add1501eead6: Pull complete 
fdad1da42790: Pull complete 
8c60ea65a035: Pull complete 
ccdfdf5ee2b1: Pull complete 
a3e1e8e2882e: Pull complete 
a6032b436e45: Pull complete 
Digest: sha256:2b87b5bb55589540f598df6ec5855e5c15dd13628230a689d46492c1d433c4df
Status: Downloaded newer image for postgres:latest
3f80d6777eeb31ace20bab432fde8735c2681a8c362576e06ed32473a75e1c6d

$ sudo docer ps -a

CONTAINER ID   IMAGE      COMMAND                  CREATED              STATUS              PORTS                                       NAMES
3f80d6777eeb   postgres   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
```
컨테이너에 접속한다.
```console
$ docker exec -it postgres /bin/bash
```
접속하려는 서버 정보를 명시한다.
```console
root@3f80d6777eeb:/ psql -U postgres
root@3f80d6777eeb:/ psql -h {host_name} -p 5432 -U postgres -d {db_name} 
```

해당 컨테이너를 다시 시작하려면
```console
$ docker run postgres 
```

## 성능 분석 - pgbench
pgbench는 PostgreSQL에서 제공하는 Benchmark 툴이다. `SELECT`, `INSERT`, `UPDATE` 등 명령어를 조합해서 시뮬레이션하고 이를 초당 Transaction 횟수로 성능을 평가한다.
서버가 설치된 하드웨어 및 OS 환경에서 성능을 측정하며 postgresql.conf의 환경 변수 설정에 사용될 수 있다.

### pgbench 사용
1. 성능 측정을 위한 임시 DB 인스턴스 생성 : 임시 Table을 쉽게 정리하기 위해 DB instance를 생성한다.
```console
$ psql -h {host} -U postgres postgres
$ CREATE DATABASE pgbenchtest OWNER postgres;
$ pgbenchtest postgres
```
2. DB instance `pgbenchtest`에 테이블 생성
```console 
$ pgbench -h {host} -p 5432 -U postgres -i pgbenchtest
```

접속하여 `\dt`로 실행하면 생성된 테이블을 확인할 수 있다.
테이블 크기를 키우려면 `-s`로 tuple수를 배수만큼 늘린다. 아래의 경우 10배만큼 증가한다.
```console
$ pgbench -h {host} -p 5432 -U postgres -i -s 10 pgbenchtest
```
3. 성능 테스트
벤치마킹 옵션을 적용하여 성능 테스트를 진행한다. 대표적으로
- `-c`: DB에 접속하는 가상의 client수
- `-j`: client를 thread 몇 개로 동작할 것인지(`-c`에서 설정한 값 이상이어야 함)
- `-t`: 시뮬레이션할 transaction 수

8개의 client, 각 client가 10회의 transaction을 수행할 때(4개의 thread로 분산)
```console
$ pgbench -h {host} -p 5432 -U postgres -c 8 -j 4 -t 10 pgbenchtest
```

튜닝 시에 postgresql.conf 인자 변경 및 pgbench 수행을 반복해야 하는 번거로움이 있다. 여러 변수를 테스트하려면 테스트할 시나리오로 자동화 테스트를 진행해야 할 것 같다.

*[참고]*  https://browndwarf.tistory.com/52

## Lock 파악하기
스크랩 : https://medium.com/29cm/db-postgresql-lock-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-57d37ebe057

## Connection Pool deadlock 현상

PostgreSQL 서버의 최대 connection 갯수는 크게 설정되지 않고 client에서 무거운 transaction으로 인한 *connection pool deadlock*이 발생하기 쉽다. 
초창기에 무거운 transaction으로 인해 pool에 deadlock이 걸려 서비스 장애가 일어날 수 있다. pool size를 늘리고 무거운 transaction은 한번에 처리할 수 있도록 최대 갯수를 늘려 deadlock을 방지할 수 있으나 서비스 응답시간과 connection 사용량에 문제가 생긴다. 

해결 방안 제시 : https://medium.com/@hyeonjay.kim/postgresql-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%BF%BC%EB%A6%AC-%EC%BB%A4%EB%84%A5%EC%85%98-%ED%92%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%9D%91%EB%8B%B5%EC%8B%9C%EA%B0%84-%EC%B5%9C%EC%A0%81%ED%99%94%ED%95%98%EA%B8%B0-917352a2a19a

*[참고]* 
- https://sondahum.tistory.com/21  
- https://blog.lael.be/post/3056

## WAL in PostgreSQL
[visit related](https://github.com/yujin-dev/CS-base/blob/master/Database.md)
PostgreSQL에서는 모든 데이터의 read & write는 보통의 file 처리로 이루어진다.

*[출처] https://habr.com/en/company/postgrespro/blog/491730/*


## Postgresql Trigger
Trigger은 INSERT, UPDATE, DELETE, TRUNCATE 와 같은 database event에 대해 특정 함수를 실행하도록 한다.
trigger은 특정 테이블과 연관되어 실행된다. 

### Types of Trigger
- Row Level Trigger : `FOR EACH ROW`로 100열을 업데이트한다면 UPDATE trigger 함수가 각 열마다 100번 호출된다.
- Statement Level Trigger : `FOR EACH STATEMENT`로 각 statement마다 한번씩 trigger 함수가 호출된다.

### Creating a Trigger
```sql
CREATE [ CONSTRAINT ] TRIGGER name { BEFORE | AFTER | INSTEAD OF } { event [ OR ... ] }

ON table_name

[ FROM referenced_table_name ]

[ NOT DEFERRABLE | [ DEFERRABLE ] [ INITIALLY IMMEDIATE | INITIALLY DEFERRED ] ]

[ REFERENCING { { OLD | NEW } TABLE [ AS ] transition_relation_name } [ ... ] ]

[ FOR [ EACH ] { ROW | STATEMENT } ]

[ WHEN ( condition ) ]

EXECUTE { FUNCTION | PROCEDURE } function_name ( arguments )

/*
where event can be one of:
INSERT
UPDATE [ OF column_name [, ... ] ]
DELETE
TRUNCATE
*/
```

#### INSERT event trigger
`INSERT`로 새로운 record가 추가되면 INSERT event trigger가 호출된다.

[ example ]  
```sql
CREATE OR REPLACE FUNCTION trigger_ex_func()
RETURNS trigger AS
$$
BEGIN
    INSERT INTO "Employee_Audit" ( "EmployeeId", "LastName", "FirstName","UserName" ,"EmpAdditionTime")
    VALUES(NEW."EmployeeId",NEW."LastName",NEW."FirstName",current_user,current_date);
RETURN NEW;
END;
$$ 
LANGUAGE 'plpgsql';

CREATE TRIGGER trigger_insert
    AFTER INSERT
    ON "Employee"
    FOR EACH ROW
    EXECUTE PROCEDURE trigger_ex_func()
```

#### UPDATE event trigger
[ example ]  
```sql
CREATE TRIGGER trigger_update
    BEFORE UPDATE
    ON "Employee"
    FOR EACH ROW
    EXECUTE PROCEDURE trigger_update_func()
```

#### DELETE event trigger
[ example ]  
```sql
CREATE TRIGGER trigger_delete
    AFTER DELETE
    ON "Employee"
    FOR EACH ROW
    EXECUTE PROCEDURE trigger_delete_func()
```


### Dropping a Trigger 
```sql
DROP TRIGGER trigger_insert on "Employee";
```

### Tips
- TRIGGER에 대한 권한이 있어야 생성 가능하다.
- `pg_trigger` 테이블에서 존재하는 trigger를 파악할 수 있다.

*[출처] https://www.enterprisedb.com/postgres-tutorials/everything-you-need-know-about-postgresql-triggers*  
*[참고] https://www.postgresql.org/docs/9.1/sql-createtrigger.html*

## Stored Functions

### Aggregate Functions
aggregations을 위한 함수. 

[ example ]  
조건에 따른 counts를 수행하는 경우 
```sql
CREATE OR REPLACE FUNCTION countif_add(current_count int, expression bool)
RETURNS int AS
$BODY$

    SELECT CASE expression
    WHEN true THEN
    current_count + 1
    ELSE
    current_count
    END;
$BODY$
LANGUAGE SQL IMMUTABLE;


CREATE AGGREGATE count_if (boolean)
(
  sfunc = countif_add,
  stype = int,
  initcond = 0
);

```
```sql
SELECT count_if(age < 30)
FROM users;
```
처럼 필터를 적용한 aggregate로 유용하게 사용할 수 있다.

### Trigger Functions
trigger 함수는 trigger만은 반환한다.

[ example ]  
table에 변화가 있었을 경우 변경된 rows를 기록한다.
```sql
CREATE FUNCTION log_user_change() RETURNS trigger
LANGUAGE plpgsql AS
$$
BEGIN
  IF NEW <> OLD THEN
    INSERT INTO user_change_log(entry_time, entry) VALUES (now(), NEW);
  END IF;
  RETURN NEW;
END;
$$;

CREATE TRIGGER user_changes
  AFTER UPDATE ON users
  FOR EACH ROW
  EXECUTE PROCEDURE log_user_change();
```

이외에도 아래 2가지의 함수를 사용하는 trigger가 존재한다.
- constraint trigger 
- event trigger 

### Functional Stability
함수 안정성과 관련하여 아래와 같이 세팅할 수 있다.
- `IMMUTABLE`: 동일한 input에 대해 결과가 절대 바뀌지 않는다.
- `STABLE`: 동일한 input에 대해 결과가 바뀔 수 있다.
- `VOLATILE`: 동일한 input에 대해 결과가 바뀌고, DB에도 변화를 줄 수 있다.

*[출처] https://www.enterprisedb.com/postgres-tutorials/everything-you-need-know-about-postgres-stored-procedures-and-functions*


### Trigger Procedures
https://www.postgresql.org/docs/9.2/plpgsql-trigger.html

- NEW : RECORD 타입으로 INSERT/UPDATE, row-level trigger에서 사용
- OLD : RECORD 타입으로 UPDATE/DELETE, row-level trigger에서 사용 


## Backup
### `pg_dump`
하나의 데이터베이스만 백업을 대상으로 한다. 
데이터 압축, 분할 및 커스텀 설정이 가능하다.
```console
$ pg_dump -Fc -d mydb -U myuser -f /tmp/db.dump
```

파일 포멧에 따라 확장자는 다르게 설정하는 것이 권장된다.
```console
$ pg_dump -Fp -d mydb -f /tmp/db.sql
$ pg_dump -Fc -d mydb -f /tmp/db.dump  # custom format file
$ pg_dump -Ft -d mydb -f /tmp/db.tar # tar format file 
$ pg_dump -Fd -d mydb -f /tmp/dumpdir #  directory format file
```

### Backup LargeSize
용량을 줄여 dump를 만들 수 있으나 DB상에 로드가 증가할 수 있다.

```console
$ pg_dump {dabase_name} | gzip > {filename}.gz
```

### `pg_dumpall`
데이터베이스 전체를 백업할 때 사용한다. role이나 tablespace 백업 이 가능하다.
```console
$ pg_dumpall > {sqlname}.sql
$ pg_dumpall -h {hostname} -p {port} -U {user} --file={filename}.sql
Password: 
Password: 
Password: 
...
```
매 데이터베이스를 백업할 때 인증을 요구한다. 

### `restore`
- 텍스트가 아닌 덤프 파일을 대상으로 한다. 
- `pg_dump`로 백업한 경우 적용한다.( `pg_dumpall`은 불가 )

### `psql` 
- 텍스트 덤프 파일을 대상으로 한다.
- `pg_dump`, `pg_dumpall`로 백업한 경우 적용한다. 
```console
$ psql -h {host} -p {port} -U username -f backupfile
```
*[출처]*
- https://m.blog.naver.com/anytimedebug/221222479261*
- https://www.tecmint.com/backup-and-restore-postgresql-database/*

*[참고]*
- https://www.postgresql.org/docs/13/app-pg-dumpall.html
- https://www.enterprisedb.com/postgresql-database-backup-recovery-what-works-wal-pitr
- https://bylee5.tistory.com/73
- https://www.postgresql.org/docs/9.1/backup.html*


## Lock

### Read lock — AccessShareLock
```sql
A1018 > begin; select * from item;
 id |  name               | selected
----+---------------------+----------
  1 | Mary    |     0
(1 rows)

B1010 > begin; select * from item;
 id |  name               | selected
----+---------------------+----------
  1 | Mary    |     0
(1 rows)
```

`Begin`을 통해 이후의 쿼리문이 Transaction으로 묶여서 `commit` 또는 `rollback`을 할 때까지 DB에 반영되지 않는다.

postgresql에서는 `pg_catalog` 스키마에서 메타 정보를 관리하는데 `pg_locks`에서 현재 transaction에 있는 lock 정보를 제공한다.
```sql
Monitor> select locktype, relation::regclass, mode, transactionid tid, pid, granted from pg_catalog.pg_locks where not pid=pg_backend_pid();

locktype    | relation  |      mode       | tid | pid  | granted
------------+-----------+-----------------+-----+------+---------
 relation   | item_pk   | AccessShareLock |     | 1010 | t
 relation   | item      | AccessShareLock |     | 1010 | t
 virtualxid |           | ExclusiveLock   |     | 1010 | t
 relation   | item_pk   | AccessShareLock |     | 1018 | t
 relation   | item      | AccessShareLock |     | 1018 | t
 virtualxid |           | ExclusiveLock   |     | 1018 | t
(6 rows)
```
`AccessShareLock`은 Read Lock으로 `select`문으로 잡히는데 granted = t로 해당 요청이 승인되었음을 알 수 있다.
모든 트랜젝션은 `virtualxid`에 `ExclusiveLock`을 잡고 있다.

### Write Lock — RowExclusiveLock
A1018가 B1010보다 먼저 item을 선택한 상황이다.

```sql
A1018 > update item set selected=selected+1 where id=1;
UPDATE 1
B1010> select * from item;
 id |  name               | selected
----+---------------------+----------
  1 | Mary    |     1
(1 rows)
```

```sql
locktype       |relation|       mode       | tid  | pid  |ranted
---------------+---------+-----------------+------+------+-------
 relation      | item_pk | AccessShareLock  |      | 1010 | t
 relation      | item    | AccessShareLock  |      | 1010 | t
 virtualxid    |         | ExclusiveLock    |      | 1010 | t    
 relation      | item_pk | AccessShareLock  |      | 1018 | t
 relation      | item    | AccessShareLock  |      | 1018 | t
 virtualxid    |         | ExclusiveLock    |      | 1018 | t
 -------------------------------------------------------------
 relation      | item_pk | RowExclusiveLock |      | 1018 | t
 relation      | item    | RowExclusiveLock |      | 1018 | t
 transactionid |         | ExclusiveLock    | 5659 | 1018 | t
(9 rows)
```
`RowExclusiveLock`은 Write Lock으로 `update`, `delete`, `insert`문에서 잡힌다. 배타적인 잠금으로 데이터를 수정할 때 
다른 사람이 동시에 바꿀 수 없도록 쓰기 잠금을 걸고 끝나면 해제한다.

마지막의 `transactionid |         | ExclusiveLock    | 5659 | 1018 | t` 은 DB 상태를 변경하려는 모든 트랜젝션에 부여되는 id로 트랜젝션이 종료될 때까지 유지된다.

### Race Condition — ShareLock
A1018와 B1010가 동시에 item을 선택한 상황이다.
```sql
B1010> update item set selected=selected+1 where id=1;
```

```sql
locktype       | relation  |       mode       | tid  | pid  |granted
---------------+-----------+------------------+------+------+-------
 relation      | item_pk   | AccessShareLock  |      | 1010 | t
 relation      | item_pk   | RowExclusiveLock |      | 1010 | t
 relation      | item      | AccessShareLock  |      | 1010 | t
 relation      | item_pk   | AccessShareLock  |      | 1018 | t
 relation      | item_pk   | RowExclusiveLock |      | 1018 | t
 relation      | item      | AccessShareLock  |      | 1018 | t
 relation      | item      | RowExclusiveLock |      | 1018 | t
 virtualxid    |           | ExclusiveLock    |      | 1018 | t
 transactionid |           | ExclusiveLock    | 5659 | 1018 | t
 relation      | item      | RowExclusiveLock |      | 1010 | t
 tuple         | item      | ExclusiveLock    |      | 1010 | t
 virtualxid    |           | ExclusiveLock    |      | 1010 | t
 transactionid |           | ExclusiveLock    | 5660 | 1010 | t
 transactionid |           | ShareLock        | 5659 | 1010 | f
(14 rows)
```

`ShareLock`은 동시에 데이터를 변경할 때 먼저 lock을 잡은 transactionid에 공유를 요청하는 lock이다. A가 먼저 ExclusiveLock을 잡아 ShareLock과 충돌되어 lock이 승인되지 않았다.
이에 따라 ExclusiveLock을 해제할 때까지 B의 요청은 대기된다.

여기서 `pg_stat_activity`을 확인하면
```sql
Monitor> SELECT query,state,pid FROM pg_catalog.pg_stat_activity;
                   query                      |    state          | pid
----------------------------------------------+-------------------+----
update item set selected=selected+1 where id=1|active             |1010
update item set selected=selected+1 where id=1|idle in transaction|1018
(2 rows)
```
A1018의 쿼리는 실행이 완료되어 트랜젝션이 끝나길 기다리기 떄문에 `idle in transaction`이고, 
B1010는 A1018의 트랜젝션이 완료되길 기다리기에 `active` 상태이다.

이 때 A1018가 `commit`이나 `rollback`을 하게 되면
```sql
A1018> commit;
COMMIT
B1010>
UPDATE 1
B1010> select * from item;
 id |  name               | selected
----+---------------------+----------
  1 | tears of goddess    |     1
(1 rows)
```
A1018의 트랜젝션이 종료되면 B1010의 대기 중인 쿼리가 실행된다.

```sql
locktype       | relation  |       mode     | tid  | pid  | granted
------------- -+-----------+------------------+------+------+-------
 virtualxid    |           | ExclusiveLock    |      | 1010 | t
 relation      | item_pk   | AccessShareLock  |      | 1010 | t
 relation      | item      | AccessShareLock  |      | 1010 | t
 relation      | item_pk   | RowExclusiveLock |      | 1010 | t
 relation      | item      | RowExclusiveLock |      | 1010 | t
 transactionid |           | ExclusiveLock    | 5660 | 1010 | t
(6 rows)
```

### Explicit Locking
- Table Lock - AccessExclusiveLock
테이블 전체 lock

- RowLock — RowShareLock
`SELECT FOR UPDATE`으로 쓰기 잠금을 걸이 row변경을 막음

*[출처] https://medium.com/29cm/db-postgresql-lock-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-57d37ebe057*


## The Internals of PostgreSQL

> 참고 : https://www.interdb.jp/pg/pgsql01.html

### Architecture
PostgreSQL은 client-server 구조이다. Database 파일에 직접 접근하지 않고 Server에 요청하여 데이터를 받는 방식이다.  
server는 각 요청마다 하나의 프로세스를 실행시킨다. threading을 사용하지 않는다. Instance는 server에서 생성한 프로세스 그룹으로 Shared Memory에서 클라이언트 요청을 처리한다. 
Postmaster라는 단일 프로세스에서 Instance가 시작된다. Instance는 configuration files를 로드하고, Shared Memory를 할당하여 Background Writer, Checkpointer, WAL Writer, WAL Archiver, Autovacuum, Statistics Collector, Logger같은 작업을 실행한다. 

![](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b0/PostgreSQL%27s_Internal_Architecture.svg/1350px-PostgreSQL%27s_Internal_Architecture.svg.png)

client에서 요청한 SELECT, UPDATE 같은 I/O-activities는 디스크에서 바로 처리되지 않는다. file page에 대한 **Shared Memory, 즉 캐시에서 동작**한다.

*dirty page*가 생기면 (file page 내용이 바뀌면) 디스크에 write된다.  
1. *WAL Writer* process ON : 우선 *WAL record( 바뀌기 전과 후에 대한 차이점 )*가 생성되어 Shared Memory에서 저장된다. `COMMIT`동안 WAL Writer process는 WAL file에 기록을 추가한다. **모든 dirty page에서 생성된 WAL record는 Memory에서 디스크로 이전된다.**
2. *Background Writer* process, *Checkpointer* process ON : Memory의 *dirty buffers*를 파일로 변환한다. Checkpointer process는 기존의 모든 dirty buffer, WAL record, Checkpoint record를 디스크에 쓰고(flush) `Checkpoint`를 생성한다. `Checkpoint`는 Checkpoint 이전의 모든 dirty page를 heap, index files에 동기화하는 트랜잭션 과정에서의 포인트를 의미한다.  

*Autovacuum* process는 heap과 index files에서 더이상 사용하지 않는 오래된 record를 `finally deleted`로 표시한다. 해당 records가 차지하고 있던 공간을 확보하게 된다.







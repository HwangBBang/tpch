# TPC-H Benchmark on MySQL 


e [TPC Website (tpc.org)](http://tpc.org/tpc_documents_current_versions/current_specifications5.asp).
The generated items are SQL compliant and can be ported to all major relational databases. While the system can generate data for major databases, support for MYSQL and MariaDB shall benefit from further documentation.
This work is based on [

Implementation of TPC-H schema into MySQL 

[Visit the Downloads page of TPC and download the latest version of TPC-H](http://tpc.org/tpc_documents_current_versions/current_specifications5.asp)  


dbgen은 TPC-H를 위한 텍스트 파일을 생성해 주는 프로그램

이 텍스트 파일을 MySQL의 LOAD DATA 구문을 이용하여 테이블에 로딩 한다 


__dbgen 확인해주기__

```
$ time ./dbgen
TPC-H Population Generator (Version 2.16.0)
Copyright Transaction Processing Performance Council 1994 - 2010
 
real    0m39.142s
user    0m38.165s
sys     0m0.903s
```

Make a copy of the dummy makefile  
```
cp makefile.suite makefile
```  
makfile 변경 
```
sudo vim makefile
```  
 
CC, DATABASE, MACHINE and WORKLOAD 변경

```
################
## CHANGE NAME OF ANSI COMPILER HERE
################
CC      = gcc
# Current values for DATABASE are: INFORMIX, DB2, TDAT (Teradata)
#                                  SQLSERVER, SYBASE, ORACLE, VECTORWISE
# Current values for MACHINE are:  ATT, DOS, HP, IBM, ICL, MVS, 
#                                  SGI, SUN, U2200, VMS, LINUX, WIN32 
# Current values for WORKLOAD are:  TPCH
DATABASE= ORACLE
MACHINE = LINUX
WORKLOAD = TPCH
#
```  

Inside the DBgen folder, run the make command.   
make 는 makefile을 자동으로 찾아 컴파일함 

```
make
```

컴파일 후 DATA 구문을 이용하여  scale factor = 10 으로 테이블에 로딩

```
$ ./dbgen -s 10
```
-s 1 인 경우 8개 의 테이블 860만 개 튜플이 생성된다. 
-s 10 인 경우는 8개의 테이블에 8600만개 튜플이 생성될 것이다 .

잘 생성되었나 확인해보자 

```
$ wc -l *.tbl


sf -1 일때 
hwangbbang@ids-srv4:~/tpch-mysql/dbgen $ wc -l *.tbl
    150000 customer.tbl (고객 테이블용 데이터)
   6001215 lineitem.tbl
        25 nation.tbl
   1500000 orders.tbl (주문 테이블용 데이터)
    800000 partsupp.tbl
    200000 part.tbl
         5 region.tbl
     10000 supplier.tbl
   8661245 total


sf -10 일때 
hwangbbang@ids-srv4:~/tpch-mysql/dbgen$ wc -l *.tbl
    1500000 customer.tbl
   59986052 lineitem.tbl
         25 nation.tbl
   15000000 orders.tbl
    8000000 partsupp.tbl
    2000000 part.tbl
          5 region.tbl
     100000 supplier.tbl
   86586082 total

```


이제 primary key와 foreign key를 설정해줘야한다 .. 



1. MySQL 서버를 동작

```
./mysql-8.0.24/build/bin/mysqld --defaults-file=./mysql-8.0.24/my.cnf
```
1-1 MySQL 접속

```
mysql -u root -p --local-infile (mysql에서 현재디렉토리에 있는 dbgen로컬파일을 사용한다.)
$ mysql> CREATE DATABASE IDS_TPCH;
$ mysql> USE IDS_TPCH;
```

2. TPC-H 로드하기

2-0. 사용법

```
./tpch-mysql/dbgen/dbgen -h
```

2-1. 데이터베이스 생성하기(데이터베이스 이름: IDS_TPCH)

```
mysqladmin create IDS_TPCH -uroot -p1234
```

2-2. 테이블 생성하기

```
mysql IDS_TPCH < ./tpch-mysql/dbgen/dss.ddl -uroot -p1234
```

2-3. 로드하기  

```
mysql IDS_TPCH < ./tpch-mysql/dbgen/dss.ri -uroot -p1234
```

2-4. 로드하기  
sf 10 (10 GB / 실험을 위한 값임)   

```
./tpch-mysql/dbgen/dbgen -s 10
```  
-> .tbl 생성

3. 쿼리 생성 

` export DSS_QUERY=PATH_TO_QUERIES_FOLDER ` /.bashrc 에 추가 


```
for i in {1..22}; do ./qgen $i > query-$i.sql; done
```
4. 쿼리 실행 

```
for i in {1..22}; do mysql IDS_TPCH < ./tpch-mysql/dbgen/queries/query-$i.sql -uroot -p1234
```


### 출처 

https://www.tpc.org/ 

https://velog.io/@shinyehwan/TPC-H 

https://sjp38.github.io/ko/post/tpch-on-mariadb/

https://ycseo.tistory.com/21 


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


1. MySQL 서버를 동작

```
./mysql-8.0.24/build/bin/mysqld --defaults-file=./mysql-8.0.24/my.cnf
```
1-1 MySQL 접속 후 데이터베이스 생성 

```
mysql -u root -p --local-infile (mysql에서 현재디렉토리에 있는 dbgen로컬파일을 사용한다.)
$ mysql> CREATE DATABASE IDS_TPCH;
$ mysql> USE IDS_TPCH;
```

2. 테이블 생성 
```
CREATE TABLE NATION  ( N_NATIONKEY  INTEGER NOT NULL,
                            N_NAME       CHAR(25) NOT NULL,
                            N_REGIONKEY  INTEGER NOT NULL,
                            N_COMMENT    VARCHAR(152));

CREATE TABLE REGION  ( R_REGIONKEY  INTEGER NOT NULL,
                            R_NAME       CHAR(25) NOT NULL,
                            R_COMMENT    VARCHAR(152));

CREATE TABLE PART  ( P_PARTKEY     INTEGER NOT NULL,
                          P_NAME        VARCHAR(55) NOT NULL,
                          P_MFGR        CHAR(25) NOT NULL,
                          P_BRAND       CHAR(10) NOT NULL,
                          P_TYPE        VARCHAR(25) NOT NULL,
                          P_SIZE        INTEGER NOT NULL,
                          P_CONTAINER   CHAR(10) NOT NULL,
                          P_RETAILPRICE DECIMAL(15,2) NOT NULL,
                          P_COMMENT     VARCHAR(23) NOT NULL );

CREATE TABLE SUPPLIER ( S_SUPPKEY     INTEGER NOT NULL,
                             S_NAME        CHAR(25) NOT NULL,
                             S_ADDRESS     VARCHAR(40) NOT NULL,
                             S_NATIONKEY   INTEGER NOT NULL,
                             S_PHONE       CHAR(15) NOT NULL,
                             S_ACCTBAL     DECIMAL(15,2) NOT NULL,
                             S_COMMENT     VARCHAR(101) NOT NULL);

CREATE TABLE PARTSUPP ( PS_PARTKEY     INTEGER NOT NULL,
                             PS_SUPPKEY     INTEGER NOT NULL,
                             PS_AVAILQTY    INTEGER NOT NULL,
                             PS_SUPPLYCOST  DECIMAL(15,2)  NOT NULL,
                             PS_COMMENT     VARCHAR(199) NOT NULL );

CREATE TABLE CUSTOMER ( C_CUSTKEY     INTEGER NOT NULL,
                             C_NAME        VARCHAR(25) NOT NULL,
                             C_ADDRESS     VARCHAR(40) NOT NULL,
                             C_NATIONKEY   INTEGER NOT NULL,
                             C_PHONE       CHAR(15) NOT NULL,
                             C_ACCTBAL     DECIMAL(15,2)   NOT NULL,
                             C_MKTSEGMENT  CHAR(10) NOT NULL,
                             C_COMMENT     VARCHAR(117) NOT NULL);

CREATE TABLE ORDERS  ( O_ORDERKEY       INTEGER NOT NULL,
                           O_CUSTKEY        INTEGER NOT NULL,
                           O_ORDERSTATUS    CHAR(1) NOT NULL,
                           O_TOTALPRICE     DECIMAL(15,2) NOT NULL,
                           O_ORDERDATE      DATE NOT NULL,
                           O_ORDERPRIORITY  CHAR(15) NOT NULL,  
                           O_CLERK          CHAR(15) NOT NULL, 
                           O_SHIPPRIORITY   INTEGER NOT NULL,
                           O_COMMENT        VARCHAR(79) NOT NULL);

CREATE TABLE LINEITEM ( L_ORDERKEY    INTEGER NOT NULL,
                             L_PARTKEY     INTEGER NOT NULL,
                             L_SUPPKEY     INTEGER NOT NULL,
                             L_LINENUMBER  INTEGER NOT NULL,
                             L_QUANTITY    DECIMAL(15,2) NOT NULL,
                             L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
                             L_DISCOUNT    DECIMAL(15,2) NOT NULL,
                             L_TAX         DECIMAL(15,2) NOT NULL,
                             L_RETURNFLAG  CHAR(1) NOT NULL,
                             L_LINESTATUS  CHAR(1) NOT NULL,
                             L_SHIPDATE    DATE NOT NULL,
                             L_COMMITDATE  DATE NOT NULL,
                             L_RECEIPTDATE DATE NOT NULL,
                             L_SHIPINSTRUCT CHAR(25) NOT NULL,
                             L_SHIPMODE     CHAR(10) NOT NULL,
                             L_COMMENT      VARCHAR(44) NOT NULL);
```


3.  클라이언트의 로컬 파일을 MySQL 서버로 직접 로드 제어 열기 
```
mysql> show global variables like 'local_infile';
mysql> set global local_infile=true;
```

로컬에서 로드 
```
LOAD DATA LOCAL INFILE 'customer.tbl' INTO TABLE CUSTOMER FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'orders.tbl' INTO TABLE ORDERS FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'lineitem.tbl' INTO TABLE LINEITEM FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'nation.tbl' INTO TABLE NATION FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'partsupp.tbl' INTO TABLE PARTSUPP FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'part.tbl' INTO TABLE PART FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'region.tbl' INTO TABLE REGION FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'supplier.tbl' INTO TABLE SUPPLIER FIELDS TERMINATED BY '|';
```

4. primary key와 foreign key를 설정
```
ALTER TABLE REGION ADD PRIMARY KEY (R_REGIONKEY);

ALTER TABLE NATION ADD PRIMARY KEY (N_NATIONKEY);

ALTER TABLE NATION
ADD FOREIGN KEY NATION_FK1 (N_REGIONKEY) references REGION(R_REGIONKEY);

ALTER TABLE PART ADD PRIMARY KEY (P_PARTKEY);

ALTER TABLE SUPPLIER ADD PRIMARY KEY (S_SUPPKEY);

ALTER TABLE SUPPLIER ADD PRIMARY KEY (S_SUPPKEY);

ALTER TABLE ORDERS ADD PRIMARY KEY (O_ORDERKEY);

ALTER TABLE SUPPLIER
ADD FOREIGN KEY SUPPLIER_FK1 (S_NATIONKEY) references NATION(N_NATIONKEY);

ALTER TABLE PARTSUPP ADD PRIMARY KEY (PS_PARTKEY,PS_SUPPKEY);

ALTER TABLE CUSTOMER ADD PRIMARY KEY (C_CUSTKEY);

ALTER TABLE CUSTOMER ADD FOREIGN KEY CUSTOMER_FK1 (C_NATIONKEY) references NATION(N_NATIONKEY);

ALTER TABLE LINEITEM ADD PRIMARY KEY (L_ORDERKEY,L_LINENUMBER);

ALTER TABLE PARTSUPP
ADD FOREIGN KEY PARTSUPP_FK1 (PS_SUPPKEY) references SUPPLIER(S_SUPPKEY);

ALTER TABLE PARTSUPP
ADD FOREIGN KEY PARTSUPP_FK2 (PS_PARTKEY) references PART(P_PARTKEY);

ALTER TABLE ORDERS
ADD FOREIGN KEY ORDERS_FK1 (O_CUSTKEY) references CUSTOMER(C_CUSTKEY);

ALTER TABLE LINEITEM
ADD FOREIGN KEY LINEITEM_FK1 (L_ORDERKEY)  references ORDERS(O_ORDERKEY);

ALTER TABLE LINEITEM
ADD FOREIGN KEY LINEITEM_FK2 (L_PARTKEY,L_SUPPKEY) references PARTSUPP(PS_PARTKEY, PS_SUPPKEY);

```


5. 확인해보기 

```
mysql> NATION;
mysql> SELECT COUNT(*) FROM CUSTOMER;
mysql> SELECT * FROM ORDERS LIMIT 10;
```


4. 쿼리 실행

mysqsl –u 유저 –p 데이터베이스명 < 파일명.sql

```
$ cd /home/hwangbbang/tpch-mysql/dbgen/queries
$ mysql –u root –p IDS_TPCH < 1.sql 
```

```
for i in {1..22}; do ./qgen $i > query-$i.sql; done
```


### 출처 

https://www.tpc.org/ 

https://velog.io/@shinyehwan/TPC-H 

https://sjp38.github.io/ko/post/tpch-on-mariadb/

https://ycseo.tistory.com/21 


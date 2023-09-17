# TPC-H Benchmark helper for MySQL and MariaDB


The scripts hosted below are for implementing the TPC-H database, sample data and queries to MySQL and MariaDB Databases under Linux.
TPC-H is a benchmark for Decision support made available by the Transaction Processing Performance Council (TPC). 
ALL the necessary specifications and documentation for setting up the TPC-H database, generating data and queries are available on the [TPC Website (tpc.org)](http://tpc.org/tpc_documents_current_versions/current_specifications5.asp).
The generated items are SQL compliant and can be ported to all major relational databases. While the system can generate data for major databases, support for MYSQL and MariaDB shall benefit from further documentation.
This work is based on [
Catarina Ribeiro's port to MySQL ](https://github.com/catarinaribeir0/queries-tpch-dbgen-mysql). The previous work dates back to 2016, which used version 2.16 of TPC-H and was meant mainly for Windows-based machines.Because Linux systems are more strict with case sensitivity of characters, the existing SQL queries available do not work under Linux.
This work reviewed the SQL queries and created a script to make the task easier. The script is only a helper to create the empty database structure,primary and foreign keys and import the generated data into a database.  All credits to the original author and the TPC team for making these tools available. Please consult the official documentation of TPC-H version 3.0.0 (published 18 February 2021).


Implementation of TPC-H schema into MySQL and MariaDB. 

[Visit the Downloads page of TPC and download the latest version of TPC-H](http://tpc.org/tpc_documents_current_versions/current_specifications5.asp)  

Move the zipped folder to /tmp of the target Linux server. The downloaded file usually ends with *-tpc-h-tool.zip
```
mv *-tpc-h-tool.zip /tmp/ && cd /tmp
``` 
Unzip the downloaded file

```
unzip *-tpc-h-tool.zip
``` 

Navigate through the command line to DBGEN folder  
```
cd /tmp/TPC-H_Tools_v*/dbgen/
```  

Type make -v and gcc -v on shell to detect if necessary tools are installed.

Install ‘make’ and ‘gcc’ if not available. If both are not present, the command below shall help in ubuntu
```
sudo apt install make && sudo apt install gcc -y
```  

Make a copy of the dummy makefile  
```
cp makefile.suite makefile
```  

Still, in the dbgen folder edit the makefile with the command below to use Nano or alternative text editor.
```
sudo nano makefile
```  
 
Find the values CC, DATABASE, MACHINE and WORKLOAD and change them as follows
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
...
```  

Quit Nano or the text editor of choice by saving the changes.


Inside the DBgen folder, run the make command.  
```
make
```
  
1. MySQL 서버를 동작

```
./mysql-8.0.24/build/bin/mysqld --defaults-file=./mysql-8.0.24/my.cnf
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






---
title: CentOS 7 환경에 MySQL 8.x 버전 설치
tags: [devops, mysql]
comments: true
categories: devops
header:
  teaser: "/assets/images/mysql/mysql_logo.png"
---


## 1. 사전 준비 과정

<br/>

#### 1-1. 기존 MySQL 패키지 제거

만약, 기존에 MySQL이 설치되어있다면, MySQL v8.x 설치하기 이전에, 이전 버전의 MySQL을 삭제해 주어야 합니다.

```sh
# yum remove mysql-community-common mysql-community-libs mysql-community-client mysql-community-server
# yum -y remove mysql57-community-release-el7-11.noarch
```

<br/>

#### 1-2. 기존 데이터 디렉토리 제거

기존에 MySQL이 설치되어있다면, 기존에 참조하고 있는 디렉토리 또한 함께 제거합니다.<br/>

이 때 디렉토리 정보는 MySQL 설정이 저장되는 /etc/my.cnf 파일을 참고하면 됩니다.

```sh
# rm -rf /var/data/mysql/* /var/run/mysqld/* /var/lib/mysql/* /var/log/mysql/*
```

<br/>

#### 1-3. mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar 다운로드

https://dev.mysql.com/downloads/mysql/ 에서 OS 버전에 맞는 RPM Bundle 패키지를 다운로드 받습니다.

![mysql_download_package](/assets/images/mysql/mysql_download_package.png)

<br/>

그리고, No thanks, just start my download. 의 링크를 복사합니다.

<br/>

![mysql_download_package_link](/assets/images/mysql/mysql_download_package_link.png)

<br/>

복사된 링크를 사용하여, mysql rpm bundle 패키지를 다운로드 받습니다.

```sh
# mkdir -p /root/download/mysql
# cd /root/download/mysql
# curl -O -L https://dev.ysql.com/get/Downloads/MySQL-8.0/mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar
# tar xvf mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar
```

<br/>

<br/>

## 2. MySQL v8.x 설치



#### 2-1. MySQL v8.x 설치

```sh
# cd /root/download/mysql
# rpm -Uvh mysql-community-common-8.0.21-1.el7.x86_64.rpm
# rpm -Uvh mysql-community-libs-8.0.21-1.el7.x86_64.rpm
# rpm -Uvh mysql-community-client-8.0.21-1.el7.x86_64.rpm
# rpm -Uvh mysql-community-server-8.0.21-1.el7.x86_64.rpm
```

<br/>



#### 2-2. MySQL 버전확인

```sh
# mysqld --version
/usr/sbin/mysqld  Ver 8.0.21 for Linux on x86_64 (MySQL Community Server - GPL)
```

<br/>

#### 2-3. MySQL 관련 디렉토리 생성 및 권한수정

```sh
# rm -rf /var/data/mysql/* /var/run/mysqld/* /var/lib/mysql/* /var/log/mysql/*
# mkdir -p /var/log/mysql /var/data/mysql
# chown -R mysql:mysql /var/log/mysql /var/data/mysql /var/lib/mysql /var/run/mysqld
```

<br/>

#### 2-4. MySQL 설정 (/etc/my.cnf 파일 편집)

```ini
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
[client]
port		= 3306
socket		= /var/lib/mysql/mysql.sock

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket		= /var/lib/mysql/mysql.sock
nice		= 0

[mysql]
default-character-set = utf8

[mysqld]
#
# * Basic Settings
#
user                = mysql
default-time-zone   ='+9:00'
pid-file            = /var/run/mysqld/mysqld.pid
socket              = /var/lib/mysql/mysql.sock
port                = 3306
basedir             = /usr
datadir             = /var/data/mysql
tmpdir              = /tmp
lc-messages-dir     = /usr/share/mysql-8.0
skip-external-locking

# character set
init_connect="SET collation_connection = utf8_general_ci"
init_connect="SET NAMES utf8"

#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address        = 0.0.0.0

#
# * Fine Tuning
#
max_allowed_packet  = 16M
thread_stack        = 192K
thread_cache_size   = 8


#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
general_log_file    = /var/log/mysql/mysql.log
general_log         = 1
#
# Error log - should be very few entries.
#
log-error = /var/log/mysql/error.log

# Logging
slow_query_log      = 1
slow_query_log_file = /var/log/mysql/mysql_slow.log

#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
server-id        = 1
log_bin          = /var/log/mysql/mysql-bin.log
max_binlog_size  = 100M
```

<br/>

#### 2-5. 데이터베이스 초기화

```sh
# sudo -u mysql mysqld --initialize
```

<br/>

#### 2-6. MySQL 초기 패스워드 확인

```sh
# grep password /var/log/mysql/error.log
2020-07-23T03:53:12.218652Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: <FOaqqvVq94+
```



<br/>

<br/>

## 3. MySQL 실행 및 중지

#### 3-1. MySQL 실행

```sh
# systemctl start mysqld
```

<br/>

#### 3-2. MySQL 중지

```sh
# systemctl stop mysqld
```

<br/>

<br/>

## 4. MySQL root 패스워드 변경



MySQL v8.x 설치한 이후, 최초 접속시에는 반드시 root 계정의 패스워드 및 접근권한을 변경해 주어야 합니다.

<br/>

#### 4-1. root 계정 생성 및 패스워드 변경

```sh
# mysql -u root -p
Enter password: 

... 이하 생략 ...
mysql> SELECT VERSION();
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

mysql> CREATE USER 'root'@'%' IDENTIFIED BY '1111';

mysql> SELECT VERSION();
+-----------+
| VERSION() |
+-----------+
| 8.0.21    |
+-----------+
1 row in set (0.00 sec)

mysql> quit
Bye
```

<br/>

#### 4-2. root 접근 권한 설정

```sh
# mysql -u root -p
Enter password: 
... 이하 생략 ...

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```



<br/>

<br/>

## 5. MySQL Client (Heidisql) 

외부에서 MySQL Client (Heidisql) 을 활용하여 접속이 되는지 확인합니다. 

![mysql_client_heidisql](/assets/images/mysql/mysql_client_heidisql.png)
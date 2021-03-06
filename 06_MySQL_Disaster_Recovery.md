# 数据库灾难恢复步骤

[TOC]

## 适用场景

表 frm、ibd文件正常情况下使用frm文件解析表结构恢复数据

## 模拟故障

```shell
# 模拟ibd文件损坏

[root@localhost data]# mv ibdata1 ibdata1.bac
# 查看日志
2019-08-05T08:51:52.466645Z 0 [Note] InnoDB: The first innodb_system data file 'ibdata1' did not exist. A new tablespace will be created!
2019-08-05T08:51:52.466848Z 0 [ERROR] InnoDB: redo log file './ib_logfile0' exists. Creating system tablespace with existing redo log files is not recommended. Please delete all redo log files before creating new system tablespace.
2019-08-05T08:51:52.466874Z 0 [ERROR] InnoDB: InnoDB Database creation was aborted with error Generic error. You may need to delete the ibdata1 file before trying to start up again.
2019-08-05T08:51:53.072929Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-08-05T08:51:53.073328Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-08-05T08:51:53.073390Z 0 [ERROR] Failed to initialize plugins.
2019-08-05T08:51:53.073450Z 0 [ERROR] Aborting

2019-08-05T08:51:53.073485Z 0 [Note] Binlog end
2019-08-05T08:51:53.073848Z 0 [Note] Shutting down plugin 'MyISAM'
2019-08-05T08:51:53.075598Z 0 [Note] /usr/local/mysql/bin/mysqld: Shutdown complete

```

## 恢复步骤

```shell
1 将数据目录重命名
[root@localhost mysql]# mv data/ data.bac

2 创建新的数据目录
[root@localhost mysql]# chown mysql. -R data
3 重新初始化 
/alidata/mysql/bin/mysqld --initialize-insecure --datadir=/alidata/mysql/data/ --basedir=/alidata/mysql/ --user=mysql

4 下载工具包：https://downloads.mysql.com/archives/utilities/
5 解压
[root@localhost install]# tar -xf mysql-utilities-1.6.5.tar.gz
[root@localhost install]# cd mysql-utilities-1.6.5
6 安装
python ./setup.py build
python ./setup.py install
7 解析表结构
mysqlfrm --server=用户名:密码@ip:3306 /alidata/mysql/data.bac2/test1/t9.frm --port=3434 --user=mysql --diagnostic
或mysqlfrm --basedir=/alidata/mysql/ /alidata/mysql/data.bac2/test1/t9.frm --port=3434 --user=mysql --diagnostic
WARNING: Using a password on the command line interface can be insecure.
# WARNING The --port option is not used in the --diagnostic mode.
# WARNING: The --user option is only used for the default mode.
# Source on 192.168.140.6: ... connected.
# CAUTION: The diagnostic mode is a best-effort parse of the .frm file. As such, it may not identify all of the components of the table correctly. This is especially true for damaged files. It will also not read the default values for the columns and the resulting statement may not be syntactically correct.
# Reading .frm file for /alidata/mysql/data.bac2/test1/t9.frm:
# The .frm file is a TABLE.
# CREATE TABLE Statement:

CREATE TABLE `test1`.`t9` (
  `id` int(11) NOT NULL, 
  `name` int(11) DEFAULT NULL, 
  `name1` char(10) COLLATE `latin1_swedish_ci` DEFAULT NULL, 
  `name2` char(10) COLLATE `latin1_swedish_ci` DEFAULT NULL, 
PRIMARY KEY `PRIMARY` (`id`),
KEY `index1` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

#...done.

# 以上的用户名最好重新建一个授权所有ip访问的临时用户名，以防止mysqlfrm权限不够

8 重新创建数据库test1
9 将解析出来的表结构语句导入数据库
[root@localhost test1]# mysql -uroot -pwaqb1314 test1 <t9.sql

10 分离表空间
ALTER TABLE t9 DISCARD TABLESPACE;

11 将原先的表的ibd文件copy到现有数据目录

12 授权数据目录
chown mysql. /alidata/mysql/data/t9.ibd

13 重新加载表空间
ALTER TABLE t9 DISCARD TABLESPACE;

至此数据库恢复完成，视图解析的话只能解析出一条select语句
解析表结构的方式可以使用脚本循环获取

```


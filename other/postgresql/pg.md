brew install postgresql

initdb /usr/local/var/postgres

createdb

psql

\l

CREATE USER postgres WITH PASSWORD 'XXXXXX';

DROP DATABASE postgres;

CREATE DATABASE postgres OWNER postgres;

GRANT ALL PRIVILEGES ON DATABASE postgres to postgres;

ALTER ROLE postgres CREATEDB;

psql -U [user] -d [database] -h [host] -p [port]
user：当前mac用户
database：用户同名数据库
主机：localhost
端口号：5432，postgresql的默认端口是5432

\password：设置当前登录用户的密码
\h：查看SQL命令的解释，比如\h select。
\?：查看psql命令列表。
\l：列出所有数据库。
\c [database_name]：连接其他数据库。
\d：列出当前数据库的所有表格。
\d [table_name]：列出某一张表格的结构。
\du：列出所有用户。
\e：打开文本编辑器。
\conninfo：列出当前数据库和连接的信息。
\password [user]: 修改用户密码
\q：退出

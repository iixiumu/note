CREATE USER 'username'@'host' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO username;
flush privileges;

GRANT privileges ON databasename.tablename TO 'username'@'host'
grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码';

SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');

DROP USER 'username'@'host';

show grants for dog@localhost;